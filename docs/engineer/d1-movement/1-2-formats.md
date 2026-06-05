# 1.2 Ingérer des formats variés

> **Domain 1.0 — Data Movement (28%)**

## Formats requis

CSV, JSON, Avro, ORC, Parquet, XML. Type **VARIANT** pour le semi-structuré.

## INFER_SCHEMA ⭐

Détecte automatiquement le schéma de fichiers stagés (Parquet/Avro/ORC/CSV/JSON) — utile pour le design de table.

```sql
SELECT * FROM TABLE(
  INFER_SCHEMA(
    LOCATION => '@stage/parquet/',
    FILE_FORMAT => 'ff_parquet'
  ));

-- Créer une table à partir du schéma inféré
CREATE TABLE ma_table USING TEMPLATE (
  SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*))
  FROM TABLE(INFER_SCHEMA(LOCATION=>'@stage/parquet/', FILE_FORMAT=>'ff_parquet'))
);
```

## Stages, file formats & intégrations

```sql
CREATE STORAGE INTEGRATION s3_int TYPE=EXTERNAL_STAGE STORAGE_PROVIDER='S3'
  STORAGE_AWS_ROLE_ARN='arn:aws:iam::123:role/sf' ENABLED=TRUE
  STORAGE_ALLOWED_LOCATIONS=('s3://bucket/');
CREATE STAGE ext URL='s3://bucket/data/' STORAGE_INTEGRATION=s3_int FILE_FORMAT=ff_parquet;
```

| Sujet | Point clé |
|---|---|
| **Chiffrement** | Pre-signed URLs, server-side (SSE-S3/KMS), client-side |
| **Compression** | gzip/bzip2/zstd/brotli ; détection auto possible |
| **Parsing** | `STRIP_OUTER_ARRAY`, `SKIP_HEADER`, `NULL_IF`, `ERROR_ON_COLUMN_COUNT_MISMATCH` |
| **Métadonnées** | `METADATA$FILENAME`, `METADATA$FILE_ROW_NUMBER` |

```sql
-- Extraire les métadonnées de fichier pendant le COPY
COPY INTO t FROM (
  SELECT $1, METADATA$FILENAME, METADATA$FILE_ROW_NUMBER FROM @stage
);
```

Données **structurées / semi-structurées / non structurées** : toutes ingérables.

📎 *Réf. : `docs.snowflake.com/en/sql-reference/functions/infer_schema`*
