# 4.4 — Transformation des données

> **Domaine D4 — 21% du COF-C03**

## Données structurées — SQL optimisé

```sql
-- ✅ CTE pour lisibilité et réutilisation
WITH ventes_emea AS (
    SELECT * FROM ventes WHERE region = 'EMEA' AND date_vente >= '2024-01-01'
)
SELECT region, SUM(montant) AS total FROM ventes_emea GROUP BY region;

-- ✅ Filtre sans fonction sur colonne (pruning actif)
SELECT * FROM ventes WHERE date_vente BETWEEN '2024-01-01' AND '2024-12-31';

-- ❌ Filtre avec fonction (pruning désactivé !)
SELECT * FROM ventes WHERE YEAR(date_vente) = 2024;
```

## Fonctions d'agrégation ⭐

```sql
SELECT region,
       COUNT(*)            AS nb_commandes,
       SUM(montant)        AS total,
       AVG(montant)        AS moyenne,
       MIN(montant)        AS minimum,
       MAX(montant)        AS maximum,
       STDDEV(montant)     AS ecart_type,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY montant) AS mediane
FROM ventes
GROUP BY region
HAVING COUNT(*) > 100;
```

## Fonctions de fenêtre (Window Functions) ⭐

```sql
SELECT date_vente, region, montant,
    -- Cumul courant par région
    SUM(montant) OVER (
        PARTITION BY region ORDER BY date_vente
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumul,
    -- Valeur précédente
    LAG(montant, 1)  OVER (PARTITION BY region ORDER BY date_vente) AS prev,
    -- Valeur suivante
    LEAD(montant, 1) OVER (PARTITION BY region ORDER BY date_vente) AS next,
    -- Rang
    RANK()   OVER (PARTITION BY region ORDER BY montant DESC) AS rang,
    DENSE_RANK() OVER (PARTITION BY region ORDER BY montant DESC) AS rang_dense,
    -- Pourcentage du total
    RATIO_TO_REPORT(montant) OVER (PARTITION BY region) AS pct_region
FROM ventes;
```

## Données semi-structurées ⭐

```sql
-- Accéder aux champs JSON avec :
SELECT data:user::STRING AS utilisateur,
       data:address:city::STRING AS ville,
       data:tags[0]::STRING AS premier_tag
FROM events_json;

-- FLATTEN un tableau JSON
SELECT e.id, f.value::STRING AS tag
FROM events e, LATERAL FLATTEN(INPUT => e.data:tags) f;

-- Construire du JSON
SELECT OBJECT_CONSTRUCT('nom', nom, 'email', email) AS json_client FROM clients;
SELECT ARRAY_CONSTRUCT(1, 2, 3) AS tableau;

-- TRY_PARSE_JSON (ne plante pas sur JSON invalide)
SELECT TRY_PARSE_JSON(champ_texte) AS json_parse FROM ma_table;
-- Retourne NULL si JSON invalide (contrairement à PARSE_JSON qui plante)
```

## Données non structurées ⭐

```sql
-- Types d'URLs pour les fichiers stagés
SELECT BUILD_SCOPED_FILE_URL(@stage_docs, 'rapport.pdf') AS url_temp;    -- temporaire
SELECT BUILD_STAGE_FILE_URL(@stage_docs, 'rapport.pdf') AS url_perm;      -- permanente

-- Utiliser avec UDFs Python pour traiter des fichiers
-- Cortex pour analyser du texte non structuré
SELECT SNOWFLAKE.CORTEX.SUMMARIZE(texte_pdf) FROM documents;
```
