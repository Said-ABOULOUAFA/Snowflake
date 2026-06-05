# Ingestion — Batch & Streaming (28%)

## Matrice de décision ⭐

```
Données arrivent en fichiers ?
    │
    ├─ Oui, manuellement/planifié ──────────► COPY INTO
    │
    ├─ Oui, automatiquement dès arrivée ───► Snowpipe (AUTO_INGEST)
    │
    └─ Non, flux continu (Kafka, IoT...) ──► Snowpipe Streaming / Kafka Connector
```

---

## COPY INTO — Options avancées

```sql
-- Chargement avec transformation inline
COPY INTO ventes (id, date_vente, montant)
FROM (
    SELECT $1::INT, $2::DATE, $3::FLOAT
    FROM @mon_stage/fichier.csv
)
FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1);

-- Charger JSON semi-structuré dans une colonne VARIANT
COPY INTO logs (data)
FROM (SELECT $1 FROM @stage_json)
FILE_FORMAT = (TYPE = 'JSON');

-- Vérifier avant de charger (VALIDATION_MODE)
COPY INTO ma_table FROM @mon_stage
  FILE_FORMAT = (FORMAT_NAME = fmt_csv)
  VALIDATION_MODE = 'RETURN_ERRORS';  -- ne charge pas, retourne les erreurs
  -- ou: RETURN_n_ROWS, RETURN_ALL_ERRORS
```

---

## Snowpipe — Architecture

```
Fichier arrive dans S3
        │
        ▼
  Notification S3 Event
        │
        ▼
   SQS Queue (AWS)
        │
        ▼
  Snowpipe (serverless)
        │
        ▼
   Table Snowflake
```

```sql
-- Pipe avec gestion d'erreur
CREATE PIPE pipe_commandes
  AUTO_INGEST    = TRUE
  ERROR_INTEGRATION = mon_error_notif  -- notifie en cas d'erreur
AS
COPY INTO commandes
FROM @stage_s3/commandes/
FILE_FORMAT = (FORMAT_NAME = fmt_json)
ON_ERROR = 'CONTINUE';

-- Diagnostiquer les erreurs Snowpipe
SELECT * FROM TABLE(VALIDATE_PIPE_LOAD(
  PIPE_NAME => 'pipe_commandes',
  START_TIME => DATEADD('hour', -1, CURRENT_TIMESTAMP())
));
```

---

## Snowpipe Streaming — Temps réel

Pour ingérer **ligne par ligne** avec latence < 1 seconde.

```java
// SDK Java — exemple simplifié
SnowflakeStreamingIngestClient client =
    SnowflakeStreamingIngestClientFactory
        .builder("mon_client")
        .setProperties(props)
        .build();

OpenChannelRequest req = OpenChannelRequest.builder("channel_1")
    .setDBName("MA_DB")
    .setSchemaName("PUBLIC")
    .setTableName("EVENTS")
    .setOnErrorOption(OpenChannelRequest.OnErrorOption.CONTINUE)
    .build();

SnowflakeStreamingIngestChannel channel = client.openChannel(req);

// Insérer une ligne
Map<String, Object> row = new HashMap<>();
row.put("event_id", "evt_001");
row.put("timestamp", System.currentTimeMillis());
row.put("payload", "{\"action\":\"click\"}");
channel.insertRow(row, "offset_1");
```

---

## Kafka Connector

Intégration native Kafka → Snowflake.

| Mode | Description |
|---|---|
| **Snowpipe mode** | Via Snowpipe (fichiers S3 intermédiaires) |
| **Streaming mode** | Via Snowpipe Streaming (direct, < 1s) |

```properties
# Configuration connector Kafka
name=snowflake-connector
connector.class=com.snowflake.kafka.connector.SnowflakeSinkConnector
tasks.max=8
topics=commandes,events
snowflake.topic2table.map=commandes:table_commandes,events:table_events
buffer.count.records=10000
buffer.flush.time=60
buffer.size.bytes=5000000
snowflake.ingestion.method=SNOWPIPE_STREAMING
```

---

## Troubleshooting ingestion ⭐

```sql
-- Historique COPY INTO (conservé 64 jours)
SELECT *
FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME  => 'MA_TABLE',
  START_TIME  => DATEADD('day', -7, CURRENT_TIMESTAMP())
))
WHERE STATUS = 'LOAD_FAILED';

-- Statut d'un pipe
SELECT PARSE_JSON(SYSTEM$PIPE_STATUS('mon_pipe'));

-- Erreurs de validation
SELECT * FROM TABLE(VALIDATE(
  ma_table,
  JOB_ID => '_last'
));
```

| Erreur fréquente | Cause | Solution |
|---|---|---|
| `Number of columns mismatch` | Fichier ≠ structure table | Vérifier le schéma |
| `NULL value not allowed` | Colonne NOT NULL mais valeur manquante | `NULL_IF` dans file format |
| `Date format not recognized` | Format date incorrect | `DATE_FORMAT` dans file format |
| `Stage not found` | Stage supprimé ou mal nommé | Recréer le stage |
