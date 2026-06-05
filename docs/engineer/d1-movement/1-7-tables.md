# 1.7 — Types de tables et opérations

> **Domaine D1 Data Movement — 28% du DEA-C02**

## Tables externes ⭐

```sql
CREATE EXTERNAL TABLE ext_ventes (
    date_vente DATE   AS (VALUE:c1::DATE),
    region     STRING AS (VALUE:c2::STRING),
    montant    FLOAT  AS (VALUE:c3::FLOAT)
)
LOCATION = @stage_s3/ventes/
FILE_FORMAT = (TYPE = 'PARQUET')
AUTO_REFRESH = TRUE;

ALTER EXTERNAL TABLE ext_ventes REFRESH;
```

## Tables Iceberg ⭐

```sql
CREATE ICEBERG TABLE ice_commandes (id BIGINT, total DOUBLE)
  CATALOG       = 'SNOWFLAKE'
  EXTERNAL_VOLUME = 'vol_s3'
  BASE_LOCATION = 'commandes/';
```

## Tables hybrides ⭐

```sql
CREATE HYBRID TABLE transactions (
    txn_id     INT PRIMARY KEY,
    account_id INT NOT NULL,
    amount     DECIMAL(15,2),
    status     VARCHAR(20) DEFAULT 'PENDING',
    INDEX idx_account (account_id)
);
```

## Horizon Catalog ⭐

```sql
CREATE CATALOG INTEGRATION glue_catalog
  CATALOG_SOURCE    = GLUE
  CATALOG_NAMESPACE = 'mon_namespace'
  GLUE_AWS_ROLE_ARN = 'arn:aws:iam::123:role/sf-glue'
  GLUE_CATALOG_ID   = '123456789'
  GLUE_REGION       = 'eu-west-1'
  ENABLED = TRUE;
```

## Schema Evolution ⭐

```sql
CREATE TABLE ventes ENABLE_SCHEMA_EVOLUTION = TRUE;

COPY INTO ventes FROM @stage
  FILE_FORMAT = (FORMAT_NAME = fmt_parquet)
  MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE;
```

## Unload ⭐

```sql
COPY INTO @stage/exports/
FROM (SELECT * FROM ventes WHERE YEAR(date_vente) = 2024)
FILE_FORMAT = (TYPE='PARQUET' COMPRESSION='SNAPPY')
OVERWRITE = TRUE MAX_FILE_SIZE = 209715200;
```
