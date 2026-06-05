# 4.1 Warehouses Snowpark-Optimized

> **Domain 4.0 — Performance Optimization (20%)**

## Pourquoi un warehouse spécialisé ?

Les **Snowpark-Optimized Warehouses** offrent **16× plus de mémoire** et plus de cache local par nœud → indispensables pour les charges Python gourmandes : ML training, UDF lourdes, grands modèles en mémoire.

```sql
CREATE WAREHOUSE wh_ml
  WAREHOUSE_SIZE = 'MEDIUM'
  WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'
  AUTO_SUSPEND = 60 AUTO_RESUME = TRUE;
```

| Type | Mémoire / nœud | Cas d'usage |
|---|---|---|
| **STANDARD** | Standard | SQL, DataFrame ops classiques |
| **SNOWPARK-OPTIMIZED** | ~16× plus | ML, UDF/sproc mémoire-intensives |

!!! danger "Piège exam"
    Si une UDF/sproc Python échoue avec une **erreur mémoire** (out of memory) ou fait du *spilling* massif, la réponse attendue est : passer à un **Snowpark-Optimized Warehouse** (et/ou augmenter la taille). Pour la simple **concurrence** (beaucoup de requêtes), c'est le **multi-cluster** qui répond, pas le type Snowpark-Optimized.

📎 *Réf. : `docs.snowflake.com/en/user-guide/warehouses-snowpark-optimized`*
