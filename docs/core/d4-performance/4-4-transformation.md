# 4.4 Techniques de transformation des données

> **Domain 4.0 — Performance, Querying & Transformation (21%)**

## Types de données

| Type | Exemple |
|---|---|
| **Structuré** | Tables relationnelles classiques |
| **Semi-structuré** | JSON, Avro, Parquet, ORC, XML → type **VARIANT** |
| **Non structuré** | PDF, images, audio (via directory tables / file URLs) |

## Semi-structuré ⭐

```sql
-- Accès aux champs JSON dans une colonne VARIANT
SELECT v:client.nom::STRING       AS nom,
       v:montant::NUMBER          AS montant,
       f.value:produit::STRING    AS produit
FROM commandes,
     LATERAL FLATTEN(input => v:lignes) f;
```

- `:` accès attribut, `[]` index, `::` cast, `FLATTEN` aplatit les tableaux.

## Fonctions d'agrégation & fenêtre

```sql
-- Agrégats
SELECT region, SUM(montant), AVG(montant), COUNT(DISTINCT client_id)
FROM ventes GROUP BY region;

-- Window functions
SELECT vente_id, region, montant,
       ROW_NUMBER() OVER (PARTITION BY region ORDER BY montant DESC) AS rang,
       SUM(montant) OVER (PARTITION BY region) AS total_region,
       LAG(montant) OVER (PARTITION BY region ORDER BY date_vente) AS prec
FROM ventes;
```

!!! tip "SQL pour l'optimisation"
    Filtrer tôt, éviter `SELECT *`, exploiter le pruning (filtrer sur colonnes clusterisées), préférer les agrégations natives.

📎 *Réf. : `docs.snowflake.com/en/user-guide/querying-semistructured`*
