# 3.2 — Ingestion automatisée

> **Domaine D3 — 18% du COF-C03**

## Snowpipe ⭐

Chargement automatique dès qu'un fichier arrive dans un stage.

```sql
CREATE PIPE mon_pipe AUTO_INGEST = TRUE AS
COPY INTO ventes FROM @stage_s3/ventes/
FILE_FORMAT = (FORMAT_NAME = fmt_json)
ON_ERROR = 'CONTINUE';

SELECT SYSTEM$PIPE_STATUS('mon_pipe');
ALTER PIPE mon_pipe REFRESH;
```

!!! danger "Snowpipe = warehouse SERVERLESS"
    Snowpipe utilise un **warehouse serverless géré par Snowflake**, pas ton propre warehouse.

## Snowpipe Streaming ⭐

Ingestion ligne par ligne avec latence **< 1 seconde** via SDK Java/Python.

```java
SnowflakeStreamingIngestClient client = SnowflakeStreamingIngestClientFactory
    .builder("client").setProperties(props).build();
SnowflakeStreamingIngestChannel channel = client.openChannel(request);
Map<String,Object> row = new HashMap<>();
row.put("event_id", "evt_001");
row.put("ts", System.currentTimeMillis());
channel.insertRow(row, "offset_1");
```

## Streams ⭐

Capture les modifications (INSERT/UPDATE/DELETE) sur une table — **CDC natif**.

```sql
CREATE STREAM stream_commandes ON TABLE commandes;  -- Standard
CREATE STREAM stream_logs ON TABLE logs APPEND_ONLY = TRUE;  -- Append-only

-- Colonnes metadata
SELECT *, METADATA$ACTION,    -- INSERT ou DELETE
          METADATA$ISUPDATE,  -- TRUE si UPDATE
          METADATA$ROW_ID
FROM stream_commandes;
```

!!! info "Un UPDATE = 1 DELETE + 1 INSERT"
    Snowflake représente les mises à jour comme une suppression + insertion.

## Tasks ⭐

```sql
CREATE TASK task_sync
  WAREHOUSE = wh_etl
  SCHEDULE  = 'USING CRON 0 * * * * UTC'
  WHEN SYSTEM$STREAM_HAS_DATA('stream_commandes')
AS
INSERT INTO commandes_traitees
SELECT col1, col2 FROM stream_commandes WHERE METADATA$ACTION = 'INSERT';

ALTER TASK task_sync RESUME;  -- ⚠️ désactivée par défaut !
```

!!! danger "Piège exam majeur"
    Les tasks sont **DÉSACTIVÉES par défaut** après création.
    Toujours faire `ALTER TASK ... RESUME`.

## Dynamic Tables ⭐

Transformation déclarative — Snowflake gère le rafraîchissement automatiquement.

```sql
CREATE DYNAMIC TABLE dt_resume
  TARGET_LAG = '1 hour'
  WAREHOUSE  = wh_etl
AS SELECT region, SUM(montant) AS total FROM ventes GROUP BY region;

ALTER DYNAMIC TABLE dt_resume REFRESH;    -- forcer
ALTER DYNAMIC TABLE dt_resume SUSPEND;
ALTER DYNAMIC TABLE dt_resume RESUME;
```

| TARGET_LAG | Signification |
|---|---|
| `'1 minute'` | Données max 1 min de retard |
| `'1 hour'` | Rafraîchissement toutes les heures max |
| `DOWNSTREAM` | Se synchronise avec tables dynamiques aval |

## Comparatif ⭐

| Méthode | Latence | Warehouse | Déclenchement |
|---|---|---|---|
| COPY INTO | Minutes | Ton warehouse | Manuel/planifié |
| Snowpipe | Secondes-minutes | **Serverless** | Auto (nouveaux fichiers) |
| Snowpipe Streaming | < 1s | **Serverless** | Continu (SDK/API) |
