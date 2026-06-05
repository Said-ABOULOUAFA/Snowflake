# 4.2 — Protection des données

> **Domaine D4 Data Governance — 14% du DEA-C02**

## Horizon Catalog ⭐

```sql
-- Fédérer des données depuis AWS Glue
CREATE CATALOG INTEGRATION glue_catalog
  CATALOG_SOURCE  = GLUE
  CATALOG_NAMESPACE = 'mon_namespace'
  GLUE_AWS_ROLE_ARN = 'arn:aws:iam::123:role/sf-glue'
  GLUE_CATALOG_ID   = '123456789'
  GLUE_REGION       = 'eu-west-1'
  ENABLED = TRUE;
```

## Column-Level Security ⭐

### Dynamic Data Masking

```sql
CREATE MASKING POLICY mp_email
  AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() IN ('DBA') THEN val
    ELSE REGEXP_REPLACE(val, '.+@', '***@')
  END;

ALTER TABLE clients MODIFY COLUMN email SET MASKING POLICY mp_email;
```

### External Tokenization

```sql
CREATE MASKING POLICY mp_tokenize
  AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() = 'DBA' THEN val
    ELSE vault_udf(val)  -- appel à une External Function de tokenisation
  END;
```

### Projection Policies ⭐

Empêche de voir qu'une colonne existe.

```sql
CREATE PROJECTION POLICY pp_ssn
  AS () RETURNS PROJECTION_CONSTRAINT ->
  CASE
    WHEN CURRENT_ROLE() IN ('COMPLIANCE_TEAM') THEN PROJECTION_CONSTRAINT(ALLOW => TRUE)
    ELSE PROJECTION_CONSTRAINT(ALLOW => FALSE)
  END;

ALTER TABLE employes MODIFY COLUMN ssn SET PROJECTION POLICY pp_ssn;
```

## Row-Level Security ⭐

### Row Access Policies

```sql
CREATE ROW ACCESS POLICY rap_par_region
  AS (region STRING) RETURNS BOOLEAN ->
  CASE
    WHEN CURRENT_ROLE() = 'ADMIN' THEN TRUE
    ELSE region = CURRENT_ROLE()
  END;

ALTER TABLE ventes ADD ROW ACCESS POLICY rap_par_region ON (region);
```

### Aggregation Policies

```sql
CREATE AGGREGATION POLICY ap_min_group
  AS () RETURNS AGGREGATION_CONSTRAINT ->
  CASE
    WHEN CURRENT_ROLE() = 'ANALYST' THEN AGGREGATION_CONSTRAINT(MIN_GROUP_SIZE => 5)
    ELSE NO_AGGREGATION_CONSTRAINT()
  END;

ALTER TABLE clients SET AGGREGATION POLICY ap_min_group;
```

## Data Clean Rooms ⭐

Partager des analyses sans exposer les données brutes.

```sql
-- Via l'interface Snowsight : Collaboration → Clean Rooms
-- Ou via l'API développeur :
SELECT SNOWFLAKE.DATA_CLEAN_ROOM.create_cleanroom('clean_room_partenaire', ...);
```
