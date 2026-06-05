# 5.4 — Données semi-structurées

> **Domaine D5 Data Transformation — 25% du DEA-C02**

## Traverser les données semi-structurées ⭐

```sql
-- Accès JSON avec notation ':'
SELECT data:user::STRING         AS utilisateur,
       data:address:city::STRING AS ville,
       data:tags[0]::STRING      AS premier_tag
FROM events;

-- FLATTEN : aplatir un tableau
SELECT e.id,
       f.value::STRING AS tag
FROM events e,
LATERAL FLATTEN(INPUT => e.data:tags) f;

-- FLATTEN avec tous les champs
SELECT seq, key, path, index, value, this
FROM TABLE(FLATTEN(INPUT => PARSE_JSON('{"a":1,"b":[1,2,3]}')));
```

## Transformer semi → structuré ⭐

```sql
CREATE TABLE ventes_clean AS
SELECT data:id::INTEGER        AS id,
       data:client::STRING     AS client,
       data:montant::FLOAT     AS montant,
       data:date::DATE         AS date_vente,
       ARRAY_SIZE(data:lignes) AS nb_lignes
FROM ventes_raw;
```

## Transformer structuré → semi-structuré ⭐

```sql
-- OBJECT_CONSTRUCT
SELECT OBJECT_CONSTRUCT(
    'id', id,
    'client', client,
    'montant', montant,
    'date', date_vente::STRING
) AS json_ligne FROM ventes;

-- ARRAY_CONSTRUCT
SELECT id, ARRAY_CONSTRUCT(produit1, produit2, produit3) AS produits FROM commandes;

-- OBJECT_AGG : construire un objet depuis des lignes
SELECT OBJECT_AGG(cle, valeur::VARIANT) AS json_config
FROM configuration;

-- ARRAY_AGG : construire un tableau depuis des lignes
SELECT client_id, ARRAY_AGG(produit_id) AS achats
FROM commandes GROUP BY client_id;
```

## TRY_PARSE_JSON ⭐

```sql
-- Ne plante PAS sur JSON invalide → retourne NULL
SELECT TRY_PARSE_JSON('{"valid": true}');   -- → {"valid": true}
SELECT TRY_PARSE_JSON('invalid json {');     -- → NULL (pas d'erreur)
SELECT TRY_PARSE_JSON(NULL);                 -- → NULL

-- vs PARSE_JSON → PLANTE sur JSON invalide
SELECT PARSE_JSON('invalid');  -- → ERROR
```
