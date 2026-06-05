# 4.2 Optimiser la performance

> **Domain 4.0 — Performance, Querying & Transformation (21%)**

## Leviers d'optimisation ⭐

| Technique | Quand l'utiliser |
|---|---|
| **Clustering keys** | Très grandes tables, filtres récurrents sur plages |
| **Search Optimization Service** | Lookups **ponctuels** (égalité, IN, LIKE) |
| **Materialized Views** | Agrégats récurrents pré-calculés |
| **Query Acceleration Service (QAS)** | Requêtes avec scans massifs irréguliers |

```sql
-- Search Optimization (lookups exacts)
ALTER TABLE clients ADD SEARCH OPTIMIZATION;

-- Vue matérialisée (Enterprise+)
CREATE MATERIALIZED VIEW mv_ca AS
  SELECT region, DATE_TRUNC('month', d) m, SUM(montant) total
  FROM ventes GROUP BY 1,2;

-- Query Acceleration Service
ALTER WAREHOUSE wh_bi SET ENABLE_QUERY_ACCELERATION = TRUE;
```

!!! warning "Clustering vs Search Optimization"
    - **Clustering** → requêtes analytiques avec filtres de **plage** (range).
    - **Search Optimization** → **lookups ponctuels** (equality / IN / LIKE).

!!! danger "Coûts"
    Search Optimization, Materialized Views, QAS et Automatic Clustering sont des **services serverless facturés en crédits** (maintenance + stockage d'index). Toujours arbitrer perf vs coût.

📎 *Réf. : `docs.snowflake.com/en/user-guide/performance-query-optimization`*
