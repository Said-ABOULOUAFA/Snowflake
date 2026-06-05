# 4.2 — Optimisation des requêtes

> **Domaine D4 — 21% du COF-C03**

## Query Acceleration Service (QAS) ⭐

Accélère automatiquement les **parties lentes** d'une requête avec des ressources serverless.

```sql
ALTER WAREHOUSE wh_analytics
  SET ENABLE_QUERY_ACCELERATION = TRUE
      QUERY_ACCELERATION_MAX_SCALE_FACTOR = 8;

-- Requêtes éligibles au QAS
SELECT query_id, eligible_query_acceleration_time
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_ACCELERATION_ELIGIBLE
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY eligible_query_acceleration_time DESC;
```

!!! info "QAS vs Multi-cluster"
    - **QAS** = accélère une **requête individuelle** (scale up automatique)
    - **Multi-cluster** = gère la **concurrence** (beaucoup de requêtes simultanées)

## Search Optimization Service ⭐

Pour les requêtes de **lookup ponctuel** (equality, IN, LIKE).

```sql
ALTER TABLE clients ADD SEARCH OPTIMIZATION ON EQUALITY(email, telephone);
ALTER TABLE clients ADD SEARCH OPTIMIZATION;  -- toutes les colonnes
SHOW TABLES LIKE 'clients';  -- colonne: search_optimization
```

!!! warning "Search Optimization vs Clustering"
    - **Clustering** → range filters (`BETWEEN`, `>`, `<`)
    - **Search Optimization** → equality lookups (`=`, `IN`, `LIKE`)

## Clustering Keys ⭐

Pour les très grandes tables (multi-TB) avec patterns de requêtes répétitifs.

```sql
ALTER TABLE ventes CLUSTER BY (date_vente, region);
ALTER TABLE ventes CLUSTER BY (DATE_TRUNC('month', date_vente));
SELECT SYSTEM$CLUSTERING_INFORMATION('ventes', '(date_vente)');
ALTER TABLE ventes RESUME RECLUSTER;
ALTER TABLE ventes SUSPEND RECLUSTER;
```

## Materialized Views ⭐

Résultats pré-calculés et stockés physiquement (Enterprise+).

```sql
CREATE MATERIALIZED VIEW mv_ventes_region AS
SELECT region, DATE_TRUNC('month', date_vente) AS mois,
       SUM(montant) AS total, COUNT(*) AS nb
FROM ventes GROUP BY 1, 2;
```

!!! warning "Limitations vue matérialisée"
    Ne supporte PAS : `HAVING`, `JOIN`, sous-requêtes, `LIMIT`, `ORDER BY`.
