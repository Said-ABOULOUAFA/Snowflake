# 4.1 — Évaluer les performances des requêtes

> **Domaine D4 — 21% du COF-C03**

## Query Profile ⭐

Accessible dans Snowsight : **Requête → Query Profile**

### Signaux d'alerte

| Signal | Cause | Solution |
|---|---|---|
| **Bytes spilled to local disk** | Warehouse trop petit | Augmenter la taille |
| **Bytes spilled to remote** | Warehouse beaucoup trop petit | Augmenter significativement |
| **Inefficient pruning** | Clustering inefficace | Ajouter clustering key |
| **Exploding joins** | Produit cartésien | Revoir la condition JOIN |
| **Queuing** | Trop de requêtes simultanées | Multi-cluster ou nouveau warehouse |
| **Partitions scanned = 100%** | Aucun pruning | Filtrer sur colonnes clusterisées |

```sql
-- Identifier les requêtes lentes
SELECT query_id, query_text, execution_time/1000 AS sec,
       bytes_scanned, partitions_scanned, partitions_total,
       ROUND(100*partitions_scanned/NULLIF(partitions_total,0),1) AS pct_scanned
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
  AND execution_status = 'SUCCESS'
ORDER BY execution_time DESC LIMIT 20;
```

## ACCOUNT_USAGE Views ⭐

```sql
-- Query attribution (qui consomme le plus)
SELECT user_name, role_name,
       COUNT(*) AS nb_queries,
       SUM(execution_time)/1000 AS total_sec,
       SUM(credits_used_cloud_services) AS credits
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE start_time >= DATEADD('day', -30, CURRENT_TIMESTAMP())
GROUP BY 1,2 ORDER BY 5 DESC;

-- Query history filtrée
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE query_type = 'SELECT'
  AND execution_status = 'FAILED'
  AND start_time >= DATEADD('day', -1, CURRENT_TIMESTAMP());
```

## Workload Management ⭐

Regrouper les charges similaires sur des warehouses dédiés :

```sql
-- Warehouse dédié ETL
CREATE WAREHOUSE wh_etl
  WAREHOUSE_SIZE = 'LARGE' AUTO_SUSPEND = 300 AUTO_RESUME = TRUE;

-- Warehouse dédié BI (haute concurrence)
CREATE WAREHOUSE wh_bi
  WAREHOUSE_SIZE = 'MEDIUM'
  MIN_CLUSTER_COUNT = 1 MAX_CLUSTER_COUNT = 4
  SCALING_POLICY = 'STANDARD' AUTO_SUSPEND = 120 AUTO_RESUME = TRUE;

-- Warehouse dédié Data Science
CREATE WAREHOUSE wh_ds
  WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'
  WAREHOUSE_SIZE = 'LARGE' AUTO_SUSPEND = 600 AUTO_RESUME = TRUE;
```

!!! tip "Bonne pratique"
    - Séparer ETL / BI / Data Science sur des warehouses dédiés
    - Éviter `SELECT *` en production
    - Filtrer sur des colonnes **sans fonction** pour activer le pruning
    - Utiliser des CTEs pour lisibilité et réutilisation
