# 2.2 — Configurer pour la performance optimale

> **Domaine D2 Performance — 19% du DEA-C02**

## Scale Out vs Scale Up ⭐

| Problème | Solution | Mécanisme |
|---|---|---|
| Requête lente / spillage | **Scale UP** | Augmenter la taille du warehouse |
| Trop de requêtes en queue | **Scale OUT** | Multi-cluster warehouse |
| Les deux | Scale UP + OUT | Multi-cluster de taille L |

## Virtual Warehouse Properties ⭐

```sql
CREATE WAREHOUSE wh_de
  WAREHOUSE_SIZE    = 'LARGE'
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 4
  SCALING_POLICY    = 'STANDARD'
  AUTO_SUSPEND      = 120
  AUTO_RESUME       = TRUE
  STATEMENT_TIMEOUT_IN_SECONDS = 3600;

-- Snowpark-Optimized pour ML/Data Science
CREATE WAREHOUSE wh_ml
  WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'
  WAREHOUSE_SIZE = 'MEDIUM';
```

## Optimisation du stockage et des coûts ⭐

```sql
-- Tables transitoires (pas de Fail-safe = économies)
CREATE TRANSIENT TABLE staging (data VARIANT)
  DATA_RETENTION_TIME_IN_DAYS = 0;

-- Tables temporaires (pas de Fail-safe, pas de Time Travel)
CREATE TEMPORARY TABLE tmp_compute AS SELECT ...;

-- Vérifier l'espace consommé par Time Travel + Fail-safe
SELECT table_name,
       active_bytes/1e9 AS active_gb,
       time_travel_bytes/1e9 AS tt_gb,
       failsafe_bytes/1e9 AS fs_gb
FROM SNOWFLAKE.ACCOUNT_USAGE.TABLE_STORAGE_METRICS
WHERE failsafe_bytes > 0
ORDER BY failsafe_bytes DESC;
```

## Data Quality Functions ⭐

```sql
-- Créer une Data Metric Function (DMF)
CREATE DATA METRIC FUNCTION dmf_null_count(
    arg_t TABLE(col1 TEXT)
) RETURNS NUMBER
AS $$
    SELECT COUNT_IF(col1 IS NULL) FROM arg_t
$$;

-- Associer à une table
ALTER TABLE ventes ADD DATA METRIC FUNCTION dmf_null_count
  ON (region)
  SCHEDULE = '15 MINUTES';

-- Voir les résultats
SELECT * FROM SNOWFLAKE.LOCAL.DATA_QUALITY_MONITORING_RESULTS
WHERE metric_name = 'DMF_NULL_COUNT'
ORDER BY measurement_time DESC;
```
