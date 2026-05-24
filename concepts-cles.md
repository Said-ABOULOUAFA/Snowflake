# COPY INTO & Snowpipe

## COPY INTO \<table\> — Chargement batch ⭐

```sql
-- Depuis un stage interne
COPY INTO ma_table
FROM @mon_stage/fichier.csv
FILE_FORMAT = (FORMAT_NAME = 'fmt_csv');

-- Depuis S3 avec pattern
COPY INTO ma_table
FROM @stage_s3
FILE_FORMAT = (TYPE = 'PARQUET')
PATTERN = '.*ventes_2024.*\.parquet';

-- Options importantes
COPY INTO ma_table FROM @mon_stage
  FILE_FORMAT = (TYPE = 'CSV' SKIP_HEADER = 1)
  ON_ERROR    = 'CONTINUE'   -- ABORT_STATEMENT (défaut) | CONTINUE | SKIP_FILE
  PURGE       = TRUE          -- supprime les fichiers après chargement réussi
  FORCE       = FALSE;        -- TRUE = recharge même les fichiers déjà chargés
```

### Comportement ON_ERROR

| Option | Comportement |
|---|---|
| `ABORT_STATEMENT` | Annule tout le chargement à la première erreur (défaut) |
| `CONTINUE` | Ignore les lignes en erreur, continue |
| `SKIP_FILE` | Ignore le fichier entier s'il contient des erreurs |
| `SKIP_FILE_n` | Ignore le fichier si + de n erreurs |
| `SKIP_FILE_n%` | Ignore le fichier si + de n% d'erreurs |

---

## Historique de chargement

```sql
-- Fichiers chargés dans les 14 derniers jours (métadonnées conservées 64 jours)
SELECT *
FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME => 'MA_TABLE',
  START_TIME => DATEADD('day', -14, CURRENT_TIMESTAMP())
));
```

!!! info "Déduplication automatique"
    Snowflake garde en mémoire les fichiers déjà chargés pendant **64 jours**.
    Si tu recharges le même fichier → ignoré automatiquement (sauf `FORCE = TRUE`).

---

## Snowpipe — Chargement continu ⭐

Charge automatiquement les fichiers **dès qu'ils arrivent** dans un stage.

```sql
-- Créer un pipe
CREATE PIPE mon_pipe
  AUTO_INGEST = TRUE          -- via notifications S3/Azure/GCS
  COMMENT     = 'Pipe ventes'
AS
COPY INTO ventes
FROM @stage_s3/ventes/
FILE_FORMAT = (FORMAT_NAME = fmt_json);

-- Voir le statut
SELECT SYSTEM$PIPE_STATUS('mon_pipe');

-- Actualiser manuellement (si pas AUTO_INGEST)
ALTER PIPE mon_pipe REFRESH;
```

---

## Comparatif COPY INTO vs Snowpipe vs Snowpipe Streaming ⭐

| | COPY INTO | Snowpipe | Snowpipe Streaming |
|---|---|---|---|
| **Déclenchement** | Manuel / planifié | Automatique (nouveaux fichiers) | Continu (SDK/API) |
| **Latence** | Minutes | Secondes–minutes | < 1 seconde |
| **Granularité** | Fichiers | Fichiers | Lignes |
| **Warehouse requis** | ✅ Oui | ❌ Non (serverless) | ❌ Non (serverless) |
| **Coût** | Crédits warehouse | Crédits Snowpipe | Crédits Snowpipe |
| **Cas d'usage** | Batch quotidien/horaire | Event-driven (S3 events) | Kafka, IoT, streaming |

!!! danger "Question fréquente"
    Snowpipe utilise un **warehouse serverless géré par Snowflake**, pas ton propre warehouse.
