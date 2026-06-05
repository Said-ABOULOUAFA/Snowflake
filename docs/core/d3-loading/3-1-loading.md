# 3.1 — Chargement & Export de données

> **Domaine D3 — 18% du COF-C03**

## File Formats ⭐

```sql
CREATE FILE FORMAT fmt_csv
  TYPE='CSV' FIELD_DELIMITER=',' SKIP_HEADER=1
  NULL_IF=('NULL','') EMPTY_FIELD_AS_NULL=TRUE COMPRESSION='AUTO';

CREATE FILE FORMAT fmt_json
  TYPE='JSON' STRIP_OUTER_ARRAY=TRUE COMPRESSION='AUTO';

CREATE FILE FORMAT fmt_parquet
  TYPE='PARQUET' COMPRESSION='SNAPPY';
```

| Format | Type |
|---|---|
| CSV, TSV | Structuré |
| JSON, Avro, ORC, Parquet, XML | Semi-structuré |

## Stages ⭐

```sql
-- Stage interne nommé
CREATE STAGE mon_stage;

-- Stage externe S3 avec Storage Integration
CREATE STAGE stage_s3
  URL = 's3://mon-bucket/data/'
  STORAGE_INTEGRATION = si_aws;

-- Commandes utiles
LIST @mon_stage;
LIST @mon_stage PATTERN = '.*\.csv';
REMOVE @mon_stage/ancien.csv;
```

| Type | Syntaxe |
|---|---|
| Interne utilisateur | `@~` |
| Interne table | `@%ma_table` |
| Interne nommé | `@mon_stage` |
| Externe | `@stage_s3` |

## Server-Side Encryption ⭐

```sql
CREATE STAGE stage_chiffre
  URL = 's3://bucket/'
  STORAGE_INTEGRATION = si_aws
  ENCRYPTION = (TYPE = 'AWS_SSE_KMS' KMS_KEY_ID = 'arn:aws:kms:...');
```

## COPY INTO <table> ⭐

```sql
-- Chargement basique
COPY INTO ventes FROM @mon_stage/data.csv
  FILE_FORMAT = (FORMAT_NAME = fmt_csv)
  ON_ERROR = 'ABORT_STATEMENT';

-- Transformation inline
COPY INTO ventes (date_vente, montant)
FROM (SELECT $2::DATE, $3::FLOAT FROM @mon_stage)
FILE_FORMAT = (TYPE='CSV' SKIP_HEADER=1);

-- Mode validation (ne charge PAS)
COPY INTO ventes FROM @mon_stage
  VALIDATION_MODE = 'RETURN_ERRORS';
  -- ou RETURN_10_ROWS, RETURN_ALL_ERRORS
```

## Options ON_ERROR ⭐

| Option | Comportement |
|---|---|
| `ABORT_STATEMENT` | Annule tout (défaut) |
| `CONTINUE` | Ignore les lignes en erreur, continue |
| `SKIP_FILE` | Ignore le fichier entier si erreur |
| `SKIP_FILE_n` | Ignore si + de n erreurs |
| `SKIP_FILE_n%` | Ignore si + de n% d'erreurs |

## Déduplication automatique ⭐

Snowflake garde l'historique des fichiers chargés **64 jours**. Recharger le même fichier est ignoré.

```sql
-- Forcer le rechargement
COPY INTO ventes FROM @mon_stage FORCE = TRUE;
```

## Directory Tables ⭐

```sql
CREATE STAGE stage_docs URL='s3://bucket/docs/'
  DIRECTORY = (ENABLE=TRUE AUTO_REFRESH=TRUE);
ALTER STAGE stage_docs REFRESH;

SELECT *, BUILD_SCOPED_FILE_URL(@stage_docs, relative_path) AS url_temp
FROM DIRECTORY(@stage_docs);
-- BUILD_SCOPED_FILE_URL = URL temporaire sécurisée
-- BUILD_STAGE_FILE_URL = URL permanente
```

## COPY INTO <location> — Export ⭐

```sql
COPY INTO @mon_stage/exports/ventes_
FROM (SELECT * FROM ventes WHERE YEAR(date_vente) = 2024)
FILE_FORMAT = (TYPE='CSV' HEADER=TRUE COMPRESSION='GZIP')
OVERWRITE = TRUE
MAX_FILE_SIZE = 104857600;  -- 100 MB par fichier
```

## Historique de chargement

```sql
SELECT * FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME => 'VENTES',
  START_TIME => DATEADD('day', -7, CURRENT_TIMESTAMP())
));
SELECT * FROM TABLE(VALIDATE(ventes, JOB_ID => '_last'));
```
