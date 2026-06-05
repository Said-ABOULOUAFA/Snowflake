# 3.3 Connecteurs & intégrations

> **Domain 3.0 — Data Loading, Unloading & Connectivity (18%)**

## Drivers & connecteurs

| Catégorie | Exemples |
|---|---|
| **Drivers** | JDBC, ODBC, Go, .NET, Node.js |
| **Connecteurs** | Python, **Spark**, **Kafka**, connecteurs natifs (Snowsight) |

## Intégrations ⭐

| Intégration | Rôle |
|---|---|
| **Storage Integration** | Délègue l'accès au cloud storage (S3/Blob/GCS) sans stocker de secrets |
| **API Integration** | Autorise des external functions / endpoints (API Gateway, etc.) |
| **Git Integration** | Connecte un dépôt Git (versionner SQL, notebooks, dbt) |
| **Notification Integration** | Notifications cloud (auto-ingest, alerts) |

```sql
CREATE STORAGE INTEGRATION s3_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123:role/snowflake'
  ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('s3://mon-bucket/');

CREATE GIT REPOSITORY mon_repo
  API_INTEGRATION = git_api_int
  ORIGIN = 'https://github.com/org/projet.git';
```

!!! tip "Storage integration = bonne pratique"
    Préférer une **storage integration** plutôt que des credentials en clair dans le stage : plus sûr et centralisé.

📎 *Réf. : `docs.snowflake.com/en/user-guide/data-load-overview`*
