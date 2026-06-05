# 2.2 Configurer pour une performance optimale

> **Domain 2.0 — Performance Optimization (19%)**

## Dimensionner le warehouse

| Levier | Effet | Quand |
|---|---|---|
| **Taille (XS→6XL)** | + mémoire & CPU par requête | Requêtes lourdes, spilling |
| **Multi-cluster (min/max)** | Scale-out concurrence | Beaucoup d'utilisateurs simultanés |
| **Scaling policy** | `STANDARD` (réactif) / `ECONOMY` (économe) | Arbitrage coût/latence |
| **AUTO_SUSPEND** | Suspend après inactivité | Maîtrise des coûts |

```sql
CREATE WAREHOUSE wh_etl
  WAREHOUSE_SIZE = 'LARGE'
  MIN_CLUSTER_COUNT = 1 MAX_CLUSTER_COUNT = 3
  SCALING_POLICY = 'STANDARD'
  AUTO_SUSPEND = 60 AUTO_RESUME = TRUE;
```

## Clustering keys

Pour les très grandes tables (multi-To) dont les filtres sont sélectifs sur une colonne :

```sql
ALTER TABLE ventes CLUSTER BY (date_vente, region);
SELECT SYSTEM$CLUSTERING_INFORMATION('ventes', '(date_vente, region)');
```

!!! warning "Coût du clustering"
    Le **reclustering automatique** consomme des crédits. À réserver aux tables volumineuses fréquemment filtrées/jointes sur la clé, et **rarement** mises à jour de façon aléatoire.

## Autres leviers

- **Search Optimization Service** : recherches ponctuelles très sélectives (point lookups) sur grandes tables.
- **Materialized views** : pré-agrégations sur sous-ensembles fréquemment requêtés.
- **Query Acceleration Service (QAS)** : déporte les scans massifs sur ressources serverless.

!!! tip
    Choix exam : *point lookup sélectif* → **Search Optimization** ; *agrégation répétée* → **Materialized View** ; *scan ponctuel énorme sur petit warehouse* → **QAS**.

📎 *Réf. : `docs.snowflake.com/en/user-guide/performance-query-optimization`*
