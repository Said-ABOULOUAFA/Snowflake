# 1.7 Types de tables & opérations

> **Domain 1.0 — Data Movement (28%)**

## Tables externes ⭐

Lisent des données **hors Snowflake** (data lake) sans les charger.

```sql
CREATE EXTERNAL TABLE ext_logs
  LOCATION = @ext_stage/logs/
  FILE_FORMAT = (TYPE = PARQUET)
  AUTO_REFRESH = TRUE;
-- Améliorer la perf : partitions + materialized view dessus
```

## Tables Iceberg

Format **ouvert** Apache Iceberg, stockage dans ton object store, interopérable (Spark, Trino…). Catalogue interne Snowflake ou externe.

## Tables Hybrid (Unistore)

Optimisées **OLTP** (lectures/écritures ponctuelles rapides, contraintes d'unicité), pour les workloads transactionnels.

| Table | Cas d'usage |
|---|---|
| **External** | Data lake en lecture |
| **Iceberg** | Lakehouse ouvert interopérable |
| **Hybrid** | OLTP / point lookups |

## Horizon Catalog & schema evolution

- **Horizon Catalog** : fédère et gouverne des données de catalogues externes.
- **Schema evolution** : évolution automatique du schéma à l'ingestion (`ENABLE_SCHEMA_EVOLUTION = TRUE`).

```sql
ALTER TABLE ventes SET ENABLE_SCHEMA_EVOLUTION = TRUE;
```

## Unload

```sql
COPY INTO @stage/export/ FROM ventes FILE_FORMAT=(TYPE=PARQUET) HEADER=TRUE;
```

📎 *Réf. : `docs.snowflake.com/en/user-guide/tables-iceberg`*
