# 1.4 Configurer les entrepôts virtuels

> **Domain 1.0 — Architecture (31%)**

## Types de warehouses

| Type | Usage |
|---|---|
| **Standard (Gen 1 / Gen 2)** | Charge SQL générale, BI, ETL |
| **Snowpark-Optimized** | Mémoire élevée par nœud → ML, UDFs lourdes, Snowpark gourmand |

## Tailles & crédits

| Taille | Crédits/h | Nœuds |
|---|---|---|
| X-Small | 1 | 1 |
| Small | 2 | 2 |
| Medium | 4 | 4 |
| Large | 8 | 8 |
| X-Large | 16 | 16 |
| … 6X-Large | jusqu'à 512 | … |

> Chaque taille **double** nœuds et crédits.

```sql
CREATE WAREHOUSE wh_prod
  WAREHOUSE_SIZE = 'MEDIUM'
  AUTO_SUSPEND   = 60       -- secondes d'inactivité
  AUTO_RESUME    = TRUE
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 3     -- multi-cluster (Enterprise+)
  SCALING_POLICY = 'STANDARD';
```

## Scale UP vs Scale OUT ⭐

| Problème | Solution |
|---|---|
| Requête lente / spilling | **Scale UP** (taille plus grande) |
| Trop de requêtes en file (queuing) | **Scale OUT** (multi-cluster) |
| Coûts trop élevés | Réduire taille + `AUTO_SUSPEND` court |

| Politique multi-cluster | Comportement |
|---|---|
| **STANDARD** | Ajoute un cluster dès qu'une requête est en file → priorité performance |
| **ECONOMY** | Attend davantage de charge avant d'ajouter → priorité coûts |

!!! warning "Piège exam"
    `MAX_CONCURRENCY_LEVEL` s'applique **au sein d'un seul cluster**. Pour aller au-delà → multi-cluster. Un warehouse **suspendu** ne consomme **aucun crédit**. Facturation **à la seconde** (minimum 60 s au démarrage).

## Bonnes pratiques

- Séparer les workloads (ETL, BI, DS) sur des warehouses distincts.
- High concurrency → multi-cluster ; requêtes complexes → taille supérieure.

📎 *Réf. : `docs.snowflake.com/en/user-guide/warehouses-overview`*
