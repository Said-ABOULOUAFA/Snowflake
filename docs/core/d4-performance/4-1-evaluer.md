# 4.1 Évaluer la performance des requêtes

> **Domain 4.0 — Performance, Querying & Transformation (21%)**

## Query Profile ⭐

![Lecture d'un Query Profile](../../assets/query-profile.svg)

Accessible dans Snowsight : **Query History → Query Profile**.

| Signal | Cause probable | Solution |
|---|---|---|
| **Bytes spilled to local** | Warehouse trop petit | Scale UP |
| **Bytes spilled to remote** | Warehouse très trop petit | Scale UP fortement |
| **Partitions scanned = 100%** | Aucun pruning | Clustering / filtre |
| **Exploding join** | Produit cartésien | Revoir condition JOIN |
| **Queued (queue time)** | Trop de requêtes simultanées | Multi-cluster |

```sql
SELECT query_id, query_text, execution_time/1000 AS sec,
       bytes_scanned, partitions_scanned, partitions_total,
       ROUND(100*partitions_scanned/NULLIF(partitions_total,0),1) AS pct
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
  AND execution_status = 'SUCCESS'
ORDER BY execution_time DESC LIMIT 20;
```

## Vues ACCOUNT_USAGE utiles

- `QUERY_HISTORY` / **query attribution** : qui consomme quoi.
- `WAREHOUSE_LOAD_HISTORY` : `queued_load` pour repérer le besoin de multi-cluster.

!!! tip "Workload management"
    Regrouper les workloads similaires sur un même warehouse, isoler les charges hétérogènes. Une requête lente n'est pas toujours un problème de SQL : souvent **sizing** ou **clustering**.

📎 *Réf. : `docs.snowflake.com/en/user-guide/ui-query-profile`*
