# 2.1 — Troubleshoot les requêtes sous-performantes

> **Domaine D2 Performance — 19% du DEA-C02**

## Identifier les requêtes lentes ⭐

```sql
SELECT query_id, query_text,
       execution_time/1000        AS sec,
       bytes_scanned/1e9          AS gb_scanned,
       partitions_scanned,
       partitions_total,
       ROUND(100*partitions_scanned/NULLIF(partitions_total,0),1) AS pct_scan,
       bytes_spilled_to_local_storage/1e9  AS spill_local_gb,
       bytes_spilled_to_remote_storage/1e9 AS spill_remote_gb
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
  AND execution_status = 'SUCCESS'
ORDER BY execution_time DESC LIMIT 20;
```

## Télémétrie & Root Cause ⭐

| Signal Query Profile | Cause racine | Solution |
|---|---|---|
| Spill to local disk | Warehouse trop petit | Scale UP |
| Spill to remote storage | Warehouse beaucoup trop petit | Scale UP significatif |
| Partitions scanned = 100% | Aucun pruning | Clustering key |
| Exploding joins | Produit cartésien | Revoir la condition JOIN |
| Queue time élevé | Trop de concurrence | Multi-cluster |
| Cloud Services > 10% | Trop de métadonnées | Simplifier les requêtes |

## Augmenter l'efficacité ⭐

```sql
-- Analyser le plan d'exécution
SELECT SYSTEM$EXPLAIN_PLAN_JSON($$
    SELECT r.region, SUM(v.montant)
    FROM ventes v JOIN regions r ON v.region_id = r.id
    GROUP BY r.region
$$);

-- Voir les statistiques de clustering
SELECT SYSTEM$CLUSTERING_INFORMATION('ventes', '(date_vente)');

-- Surveillance du queuing
SELECT warehouse_name, queued_load, running
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY
WHERE start_time >= DATEADD('hour', -1, CURRENT_TIMESTAMP())
ORDER BY queued_load DESC;
```
