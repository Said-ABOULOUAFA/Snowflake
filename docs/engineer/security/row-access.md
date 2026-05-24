# Row Access & Projection Policies

## Row Access Policies ⭐

Filtre les lignes accessibles selon le rôle ou l'utilisateur. Les lignes non autorisées sont **invisibles**.

```sql
-- Politique simple : chaque utilisateur voit seulement sa région
CREATE ROW ACCESS POLICY rap_par_region
  AS (region STRING) RETURNS BOOLEAN ->
  CASE
    WHEN CURRENT_ROLE() = 'ADMIN' THEN TRUE          -- tout voir
    ELSE region = CURRENT_ROLE()                       -- seulement sa région
  END;

-- Appliquer sur une table
ALTER TABLE ventes ADD ROW ACCESS POLICY rap_par_region ON (region);

-- Supprimer
ALTER TABLE ventes DROP ROW ACCESS POLICY rap_par_region;
```

---

## Row Access avec table de mapping ⭐

Pattern plus flexible : une table de mapping définit qui peut voir quoi.

```sql
-- Table de mapping des accès
CREATE TABLE acces_regions (
    role_name    STRING,
    region       STRING
);

INSERT INTO acces_regions VALUES
    ('ANALYST_EMEA', 'EMEA'),
    ('ANALYST_AMER', 'AMER'),
    ('ANALYST_EMEA', 'APAC'),  -- EMEA peut aussi voir APAC
    ('ADMIN',        '%');      -- Wildcard pour admin

-- Politique basée sur la table de mapping
CREATE ROW ACCESS POLICY rap_mapping
  AS (region STRING) RETURNS BOOLEAN ->
  EXISTS (
      SELECT 1 FROM acces_regions
      WHERE role_name = CURRENT_ROLE()
        AND region LIKE acces_regions.region
  );

ALTER TABLE ventes ADD ROW ACCESS POLICY rap_mapping ON (region);
```

---

## Projection Policies ⭐

Empêche certains rôles d'accéder à une colonne — même avec `SELECT *`.

```sql
-- Créer la politique
CREATE PROJECTION POLICY pp_ssn
  AS () RETURNS PROJECTION_CONSTRAINT ->
  CASE
    WHEN CURRENT_ROLE() IN ('HR', 'COMPLIANCE') THEN
      PROJECTION_CONSTRAINT(ALLOW => TRUE)
    ELSE
      PROJECTION_CONSTRAINT(ALLOW => FALSE)  -- colonne inaccessible
  END;

-- Appliquer
ALTER TABLE employes MODIFY COLUMN numero_secu
  SET PROJECTION POLICY pp_ssn;

-- Test : rôle non autorisé → erreur
SELECT numero_secu FROM employes;
-- Error: Column 'NUMERO_SECU' is not accessible due to projection policy

-- SELECT * → colonne automatiquement exclue
SELECT * FROM employes;
-- La colonne numero_secu n'apparaît pas dans les résultats
```

---

## Aggregation Policies ⭐

Empêche les requêtes qui retournent moins de N lignes — protège contre la ré-identification.

```sql
CREATE AGGREGATION POLICY ap_min_groupe_5
  AS () RETURNS AGGREGATION_CONSTRAINT ->
  AGGREGATION_CONSTRAINT(MIN_GROUP_SIZE => 5);

ALTER TABLE donnees_sante SET AGGREGATION POLICY ap_min_groupe_5;

-- Effet :
SELECT sexe, COUNT(*) FROM donnees_sante GROUP BY sexe;
-- Si un groupe a < 5 lignes → masqué dans les résultats
```

---

## Voir les politiques appliquées

```sql
-- Toutes les politiques sur les objets
SELECT ref_entity_name, ref_column_name, policy_name, policy_kind
FROM SNOWFLAKE.ACCOUNT_USAGE.POLICY_REFERENCES
WHERE policy_kind IN ('ROW_ACCESS_POLICY', 'PROJECTION_POLICY', 'AGGREGATION_POLICY')
ORDER BY ref_entity_name;
```

---

## Récapitulatif — Quelle politique choisir ? ⭐

| Besoin | Politique |
|---|---|
| Masquer la valeur (garder la colonne visible) | **Dynamic Data Masking** |
| Bloquer complètement l'accès à une colonne | **Projection Policy** |
| Filtrer des lignes entières | **Row Access Policy** |
| Empêcher les petits groupes (confidentialité) | **Aggregation Policy** |
| Remplacer par token via service externe | **External Tokenization** |
