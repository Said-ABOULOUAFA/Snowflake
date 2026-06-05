# Troubleshooting — Ingestion de données

## Erreurs COPY INTO fréquentes ⭐

| Erreur | Cause | Solution |
|---|---|---|
| `Number of columns mismatch` | Fichier ≠ structure table | Vérifier le schéma ou utiliser `COPY INTO` avec mapping |
| `NULL value not allowed` | Colonne NOT NULL mais valeur manquante | Ajouter `NULL_IF` dans le file format |
| `Date format not recognized` | Format date incorrect | Ajouter `DATE_FORMAT` dans le file format |
| `Stage not found` | Stage supprimé ou mal nommé | Recréer le stage |
| `File not found` | Chemin incorrect | Vérifier avec `LIST @mon_stage` |
| `Invalid UTF-8` | Encodage incorrect | Ajouter `ENCODING = 'LATIN1'` dans le file format |

---

## Diagnostiquer les erreurs COPY INTO ⭐

```sql
-- 1. Voir les erreurs sans charger (mode validation)
COPY INTO ma_table FROM @mon_stage
  FILE_FORMAT = (FORMAT_NAME = fmt_csv)
  VALIDATION_MODE = 'RETURN_ERRORS';

-- 2. Voir les N premières lignes sans charger
COPY INTO ma_table FROM @mon_stage
  FILE_FORMAT = (FORMAT_NAME = fmt_csv)
  VALIDATION_MODE = 'RETURN_10_ROWS';

-- 3. Historique des chargements (64 jours)
SELECT *
FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME  => 'MA_TABLE',
    START_TIME  => DATEADD('day', -7, CURRENT_TIMESTAMP())
))
WHERE STATUS != 'Loaded'
ORDER BY LAST_LOAD_TIME DESC;

-- 4. Voir les erreurs du dernier chargement
SELECT * FROM TABLE(VALIDATE(ma_table, JOB_ID => '_last'));
```

---

## Diagnostiquer Snowpipe ⭐

```sql
-- Statut du pipe
SELECT PARSE_JSON(SYSTEM$PIPE_STATUS('mon_pipe'));
-- Champs importants: executionState, pendingFileCount, notificationChannelName

-- Valider les chargements Snowpipe
SELECT *
FROM TABLE(VALIDATE_PIPE_LOAD(
    PIPE_NAME  => 'MON_PIPE',
    START_TIME => DATEADD('hour', -1, CURRENT_TIMESTAMP())
));

-- Historique Snowpipe
SELECT *
FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME  => 'MA_TABLE',
    START_TIME  => DATEADD('hour', -6, CURRENT_TIMESTAMP())
))
WHERE PIPE_NAME IS NOT NULL
ORDER BY LAST_LOAD_TIME DESC;

-- Rafraîchir manuellement un pipe (si fichiers manqués)
ALTER PIPE mon_pipe REFRESH;

-- Recréer les notifications (si SQS désynchronisé)
SELECT SYSTEM$PIPE_FORCE_RESUME('mon_pipe');
```

---

## Diagnostiquer Snowpipe Streaming

```sql
-- Voir les channels actifs
SELECT *
FROM TABLE(INFORMATION_SCHEMA.SNOWPIPE_STREAMING_CLIENT_HISTORY(
    START_TIME => DATEADD('hour', -1, CURRENT_TIMESTAMP())
));

-- Vérifier les erreurs par channel
SELECT channel_name, error_count, rows_inserted
FROM TABLE(INFORMATION_SCHEMA.SNOWPIPE_STREAMING_CLIENT_HISTORY(
    START_TIME => DATEADD('day', -1, CURRENT_TIMESTAMP())
))
WHERE error_count > 0;
```

---

## Problèmes courants & solutions

### Fichiers rechargés indésirables
```sql
-- Voir si un fichier a déjà été chargé
SELECT stage_location, file_name, last_load_time, status
FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
    TABLE_NAME => 'MA_TABLE',
    START_TIME => DATEADD('day', -64, CURRENT_TIMESTAMP())
))
WHERE file_name LIKE '%mon_fichier%';

-- Forcer le rechargement (si > 64 jours ou FORCE requis)
COPY INTO ma_table FROM @mon_stage/mon_fichier.csv
  FILE_FORMAT = (FORMAT_NAME = fmt_csv)
  FORCE = TRUE;
```

### Stage externe inaccessible
```sql
-- Tester la connexion au stage externe
LIST @mon_stage_s3;
-- Si erreur → vérifier Storage Integration et IAM AWS

-- Vérifier l'intégration de stockage
DESC INTEGRATION si_aws;
SHOW INTEGRATIONS;
```

### Données JSON malformées
```sql
-- Charger en VARIANT d'abord, puis transformer
COPY INTO staging_raw (data)
FROM (SELECT $1 FROM @mon_stage)
FILE_FORMAT = (TYPE = 'JSON' STRIP_OUTER_ARRAY = FALSE);

-- Identifier les lignes problématiques
SELECT data, TRY_PARSE_JSON(data::STRING) AS parsed
FROM staging_raw
WHERE TRY_PARSE_JSON(data::STRING) IS NULL;
```

---

## Checklist troubleshooting ⭐

```
❌ Erreur de chargement ?
  → VALIDATION_MODE = 'RETURN_ERRORS' pour voir sans charger

❌ Fichier non détecté par Snowpipe ?
  → LIST @mon_stage → vérifier que le fichier existe
  → SYSTEM$PIPE_STATUS → vérifier que AUTO_INGEST est actif
  → Vérifier les notifications SQS/Event Grid/GCS Pub/Sub

❌ Doublons dans les données ?
  → COPY_HISTORY → vérifier si fichier déjà chargé (64j)
  → Si > 64j → utiliser MERGE ou déduplication

❌ Performance lente ?
  → Trop petits fichiers → regrouper en fichiers de 100-250 MB
  → Trop gros fichiers → laisser Snowflake paralléliser (auto)
  → Vérifier taille du warehouse (augmenter si nécessaire)
```
