# 3.2 — Micro-partitions & Analyse du clustering

> **Domaine D3 Storage — 14% du DEA-C02**

## SYSTEM$CLUSTERING_INFORMATION ⭐

```sql
-- Analyser le clustering d'une table
SELECT SYSTEM$CLUSTERING_INFORMATION('ventes');
SELECT SYSTEM$CLUSTERING_INFORMATION('ventes', '(date_vente, region)');

-- Résultat :
-- average_depth : idéal proche de 1 (clustering parfait)
-- average_overlaps : idéal proche de 0 (pas de chevauchements)
-- partitions_scanned : % typiquement scanné avec ce clustering
-- notes : recommandations
```

## SYSTEM$CLUSTERING_DEPTH ⭐

```sql
-- Profondeur du clustering (valeur scalaire)
SELECT SYSTEM$CLUSTERING_DEPTH('ventes', '(date_vente)');
-- Retourne un nombre : proche de 1 = excellent, > 5 = à améliorer
```

## Gestion du clustering automatique ⭐

```sql
-- Ajouter une clé de clustering
ALTER TABLE ventes CLUSTER BY (date_vente, region);
ALTER TABLE ventes CLUSTER BY (DATE_TRUNC('month', date_vente), region);

-- Suspendre/reprendre le reclustering automatique
ALTER TABLE ventes SUSPEND RECLUSTER;
ALTER TABLE ventes RESUME RECLUSTER;

-- Vérifier les infos de clustering
SHOW TABLES LIKE 'ventes';
-- Colonnes : clustering_key, rows, bytes, automatic_clustering

-- Supprimer une clé de clustering
ALTER TABLE ventes DROP CLUSTERING KEY;
```

## TABLE_STORAGE_METRICS ⭐

```sql
SELECT table_name,
       active_bytes/1e9       AS active_gb,
       time_travel_bytes/1e9  AS time_travel_gb,
       failsafe_bytes/1e9     AS failsafe_gb,
       (active_bytes + time_travel_bytes + failsafe_bytes)/1e9 AS total_gb
FROM SNOWFLAKE.ACCOUNT_USAGE.TABLE_STORAGE_METRICS
WHERE active_bytes > 1073741824  -- > 1 GB
ORDER BY total_gb DESC LIMIT 20;
```

## SYSTEM$EXPLAIN_PLAN_JSON ⭐

```sql
-- Plan d'exécution pour optimisation
SELECT PARSE_JSON(
    SYSTEM$EXPLAIN_PLAN_JSON($$
        SELECT region, SUM(montant)
        FROM ventes
        WHERE date_vente BETWEEN '2024-01-01' AND '2024-12-31'
        GROUP BY region
    $$)
) AS plan;
```
