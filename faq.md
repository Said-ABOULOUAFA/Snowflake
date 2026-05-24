# Performance Optimization (22%)

## Query Profile — Diagnostics ⭐

Accessible dans Snowsight : **Requête → Query Profile**

### Signaux d'alerte

| Signal | Cause probable | Solution |
|---|---|---|
| **Bytes scanned élevé** | Pas de pruning | Ajouter clustering key |
| **Spillage to local disk** | Warehouse trop petit | Augmenter la taille |
| **Spillage to remote** | Warehouse beaucoup trop petit | Augmenter significativement |
| **Exploding joins** | Produit cartésien | Revoir la condition JOIN |
| **% Partitions scanned = 100%** | Aucun pruning | Filtrer sur colonnes clusterisées |
| **Queue time élevé** | Trop de requêtes simultanées | Multi-cluster ou warehouse séparé |

```sql
-- Voir les requêtes lentes
SELECT query_id, query_text, execution_time/1000 AS sec,
       bytes_scanned, partitions_scanned, partitions_total,
       ROUND(100*partitions_scanned/partitions_total, 1) AS pct_scanned
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
  AND execution_status = 'SUCCESS'
ORDER BY execution_time DESC
LIMIT 20;
```

---

## Warehouse Sizing ⭐

### Règle générale

| Problème | Solution |
|---|---|
| Requête lente, spillage | **Augmenter la taille** du warehouse (scale UP) |
| Trop de requêtes en queue | **Multi-cluster** (scale OUT) |
| Coûts trop élevés | **Diminuer la taille**, activer auto-suspend |

```sql
-- Modifier la taille à chaud (les requêtes en cours continuent)
ALTER WAREHOUSE wh_etl SET WAREHOUSE_SIZE = 'LARGE';

-- Surveiller le queuing
SELECT warehouse_name, queued_load, query_count
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY
WHERE start_time >= DATEADD('hour', -1, CURRENT_TIMESTAMP());
```

### Multi-cluster warehouse

```sql
CREATE WAREHOUSE wh_bi_multiclusters
  WAREHOUSE_SIZE   = 'MEDIUM'
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 3
  SCALING_POLICY   = 'STANDARD'   -- réactif
  AUTO_SUSPEND     = 120
  AUTO_RESUME      = TRUE;
```

!!! info "STANDARD vs ECONOMY"
    - **STANDARD** : ajoute un cluster dès qu'une requête est en queue → priorité performance
    - **ECONOMY** : attend d'avoir assez de requêtes en queue → priorité coûts

---

## Matérialisation & Search Optimization

### Vues matérialisées

```sql
-- Résultats pré-calculés et stockés physiquement
CREATE MATERIALIZED VIEW mv_ventes_mensuelles AS
SELECT DATE_TRUNC('month', date_vente) AS mois,
       region, SUM(montant) AS total
FROM ventes
GROUP BY 1, 2;

-- Snowflake maintient automatiquement la vue à jour
-- Coût: stockage + crédits de maintenance
```

### Search Optimization Service

Pour accélérer les requêtes de **recherche ponctuelle** (lookups par valeur exacte).

```sql
-- Activer sur une table
ALTER TABLE clients ADD SEARCH OPTIMIZATION;

-- Vérifier le statut
SHOW TABLES LIKE 'clients';
-- Colonne: search_optimization = ON

-- Coût: crédits de maintenance + stockage des index
```

!!! warning "Search Optimization vs Clustering"
    - **Clustering** → requêtes analytiques avec filtres sur plages (range)
    - **Search Optimization** → requêtes de lookup ponctuel (equality, IN, LIKE)

---

## Réplication & Business Continuity

```sql
-- Voir les objets répliqués et leur statut
SELECT name, replication_factor, primary_snapshot_timestamp
FROM SNOWFLAKE.ACCOUNT_USAGE.REPLICATION_GROUP_HISTORY;

-- Failover vers le replica
ALTER DATABASE ma_db_replica PRIMARY;
```
