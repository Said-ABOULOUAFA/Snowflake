# 3.3 — Connecteurs & Intégrations

> **Domaine D3 — 18% du COF-C03**

## Drivers Snowflake

| Driver | Usage |
|---|---|
| **JDBC** | Java, BI (Tableau, Power BI) |
| **ODBC** | Windows, Excel |
| **Python Connector** | Scripts Python |
| **Node.js Driver** | Applications JS |
| **Snowflake CLI** | Terminal, CI/CD |

## Storage Integration ⭐

Permet à Snowflake d'accéder au stockage cloud **sans clés en dur**.

```sql
-- AWS S3
CREATE STORAGE INTEGRATION si_aws
  TYPE = EXTERNAL_STAGE STORAGE_PROVIDER = 'S3' ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123:role/snowflake-role'
  STORAGE_ALLOWED_LOCATIONS = ('s3://mon-bucket/');

DESC INTEGRATION si_aws;  -- récupérer ARN IAM et External ID

-- Azure
CREATE STORAGE INTEGRATION si_azure
  TYPE = EXTERNAL_STAGE STORAGE_PROVIDER = 'AZURE' ENABLED = TRUE
  AZURE_TENANT_ID = 'mon-tenant-id'
  STORAGE_ALLOWED_LOCATIONS = ('azure://compte.blob.core.windows.net/ctn/');

-- GCS
CREATE STORAGE INTEGRATION si_gcs
  TYPE = EXTERNAL_STAGE STORAGE_PROVIDER = 'GCS' ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('gcs://mon-bucket/');
```

## API Integration ⭐

```sql
-- Pour External Functions (AWS Lambda)
CREATE API INTEGRATION api_lambda
  API_PROVIDER = aws_api_gateway
  API_AWS_ROLE_ARN = 'arn:aws:iam::123:role/sf-api-role'
  API_ALLOWED_PREFIXES = ('https://abc.execute-api.eu-west-1.amazonaws.com/prod/')
  ENABLED = TRUE;

-- Pour Git repositories
CREATE API INTEGRATION git_github
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/Said-ABOULOUAFA/')
  ENABLED = TRUE;
```

## Git Integration ⭐

```sql
CREATE GIT REPOSITORY mon_repo
  ORIGIN = 'https://github.com/Said-ABOULOUAFA/Snowflake.git'
  API_INTEGRATION = git_github;

ALTER GIT REPOSITORY mon_repo FETCH;
LS @mon_repo/branches/main/;
SHOW GIT BRANCHES IN GIT REPOSITORY mon_repo;
EXECUTE IMMEDIATE FROM @mon_repo/branches/main/scripts/init.sql;
```

## Connecteurs natifs

| Connecteur | Description |
|---|---|
| **Kafka Connector** | Ingestion depuis Apache Kafka |
| **Spark Connector** | Lecture/écriture depuis Apache Spark |
| **Python Connector** | Accès programmatique Python |
| **Native connectors** | Connecteurs Snowflake natifs (partage direct) |
