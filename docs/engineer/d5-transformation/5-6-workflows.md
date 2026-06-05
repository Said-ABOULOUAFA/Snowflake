# 5.6 Workflows & code management (Git, dbt, Notebooks)

> **Domain 5.0 — Data Transformation (25%)**

## Git integration

```sql
CREATE API INTEGRATION git_api
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/mon-org')
  ENABLED = TRUE;

CREATE GIT REPOSITORY mon_repo
  API_INTEGRATION = git_api
  ORIGIN = 'https://github.com/mon-org/mon-projet';

-- Exécuter un script versionné
EXECUTE IMMEDIATE FROM @mon_repo/branches/main/scripts/etl.sql;
```

## dbt sur Snowflake

dbt transforme via des modèles SQL versionnés (`SELECT` → tables/vues matérialisées). Snowflake exécute le `compile` + `run`. Intégration native **dbt Projects on Snowflake**.

## Snowflake Notebooks

Environnement interactif (cellules SQL + Python/Snowpark) directement dans Snowsight, idéal pour le prototypage data/ML.

| Outil | Rôle |
|---|---|
| **Git integration** | Versionner & exécuter scripts |
| **dbt** | Modélisation/transformation déclarative |
| **Notebooks** | Prototypage interactif SQL + Python |
| **Tasks** | Orchestration planifiée (DAG) |
| **Streams** | Capture des changements (CDC) |

!!! tip "Streams + Tasks = pipeline CDC"
    Un **stream** capture les changements (inserts/updates/deletes) d'une table ; une **task** les consomme périodiquement (`WHEN SYSTEM$STREAM_HAS_DATA('mon_stream')`) → pipeline incrémental automatisé.

📎 *Réf. : `docs.snowflake.com/en/developer-guide/git/git-overview`*
