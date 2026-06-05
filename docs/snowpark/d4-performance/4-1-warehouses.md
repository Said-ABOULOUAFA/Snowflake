# 4.1 — Warehouses Snowpark-Optimized

> **Domaine D4 Performance — 20% du SPS-C01**

## Cas d'usage ⭐

!!! danger "Question officielle SPS-C01"
    **Q : Quel workload bénéficie le PLUS d'un Snowpark-optimized warehouse ?**
    **R : Machine learning TRAINING** (réponse A officielle)
    Pas inference, pas model registry, pas Container Services.

| Workload | Warehouse recommandé |
|---|---|
| **ML training** (modèles complexes) | **Snowpark-Optimized** |
| **Transformations Python massives** | Snowpark-Optimized |
| **Feature engineering** gros volumes | Snowpark-Optimized |
| SQL analytique standard | Standard |
| ML inference | Standard (plus économique) |
| ETL Python léger | Standard |

## Caractéristiques techniques ⭐

| Caractéristique | Standard | Snowpark-Optimized |
|---|---|---|
| Mémoire par nœud | Standard | **16x plus** |
| CPU par nœud | Standard | Standard |
| Disque SSD local | Standard | **Beaucoup plus** |
| Coût | Base | **Plus élevé** |
| Facturation | Crédits/heure | Crédits/heure + supplément |

```sql
-- Créer un Snowpark-Optimized Warehouse
CREATE WAREHOUSE wh_ml
  WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'
  WAREHOUSE_SIZE = 'MEDIUM'
  AUTO_SUSPEND   = 300
  AUTO_RESUME    = TRUE;

-- Modifier les propriétés
ALTER WAREHOUSE wh_ml SET
  WAREHOUSE_SIZE = 'LARGE'
  AUTO_SUSPEND   = 600;
```

## Scale Up vs Scale Down ⭐

```python
# Quand scale UP un Snowpark-Optimized ?
# → Mémoire insuffisante (OOM errors)
# → Spill to disk excessif
# → Temps d'entraînement ML trop long

# Quand scale DOWN ?
# → Après le ML training (passer en inference sur Standard)
# → Workload de nuit terminé

# Depuis Snowpark
session.sql("ALTER WAREHOUSE wh_ml SET WAREHOUSE_SIZE = 'LARGE'").collect()
```

## Facturation ⭐

```
Snowpark-Optimized = crédits warehouse standard + supplément mémoire
Medium Snowpark-Optimized ≈ 6 crédits/h (au lieu de 4 pour Medium Standard)
```

!!! tip "Optimisation des coûts"
    Utilise un Snowpark-Optimized uniquement pour le **training** intensif.
    Pour l'inference, un warehouse Standard est suffisant et moins cher.
