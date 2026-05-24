# Dynamic Data Masking & Column-Level Security

## Dynamic Data Masking ⭐

Masque automatiquement les données sensibles **selon le rôle** de l'utilisateur. Les données en base ne changent pas.

```sql
-- 1. Créer la politique de masquage
CREATE MASKING POLICY masque_email
  AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() IN ('DBA', 'COMPLIANCE') THEN val
    WHEN CURRENT_ROLE() IN ('ANALYST')           THEN '***@***.***'
    ELSE '** CONFIDENTIEL **'
  END;

-- 2. Appliquer sur une colonne
ALTER TABLE clients MODIFY COLUMN email
  SET MASKING POLICY masque_email;

-- 3. Vérifier
SHOW MASKING POLICIES;
DESC MASKING POLICY masque_email;

-- 4. Supprimer
ALTER TABLE clients MODIFY COLUMN email
  UNSET MASKING POLICY;

DROP MASKING POLICY masque_email;
```

---

## Masquage conditionnel (avec colonne de référence) ⭐

```sql
-- Masquer selon la valeur d'une autre colonne
CREATE MASKING POLICY masque_salaire_dept
  AS (salaire FLOAT, departement STRING) RETURNS FLOAT ->
  CASE
    WHEN CURRENT_ROLE() = 'HR'                  THEN salaire
    WHEN departement = CURRENT_USER()            THEN salaire  -- manager voit son dépt
    ELSE -1
  END;

-- Appliquer avec colonne de référence
ALTER TABLE employes MODIFY COLUMN salaire
  SET MASKING POLICY masque_salaire_dept
  USING (salaire, departement);
```

---

## Fonctions utiles dans les politiques

```sql
-- Fonctions contextuelles disponibles
CURRENT_ROLE()         -- rôle actif
CURRENT_USER()         -- utilisateur connecté
CURRENT_ACCOUNT()      -- compte Snowflake
IS_ROLE_IN_SESSION()   -- vérifie si un rôle est actif

-- Exemple avec IS_ROLE_IN_SESSION
CREATE MASKING POLICY masque_pii
  AS (val STRING) RETURNS STRING ->
  CASE
    WHEN IS_ROLE_IN_SESSION('DATA_ADMIN') THEN val
    ELSE SHA2(val)  -- hash irréversible
  END;
```

---

## Voir les politiques appliquées

```sql
-- Voir toutes les colonnes masquées
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.POLICY_REFERENCES
WHERE POLICY_KIND = 'MASKING_POLICY'
ORDER BY ref_column_name;

-- Voir l'historique des accès aux données masquées
SELECT user_name, query_text, objects_accessed
FROM SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY
WHERE query_start_time >= DATEADD('day', -1, CURRENT_TIMESTAMP())
LIMIT 100;
```

---

## Points clés exam ⭐

!!! danger "À retenir absolument"
    - Dynamic Data Masking nécessite **édition Enterprise minimum**
    - Les données **en base ne changent pas** — seul le résultat retourné est masqué
    - La politique s'applique **à la colonne**, pas à la table entière
    - `CURRENT_ROLE()` dans la politique = rôle **actif dans la session**
    - Si l'utilisateur a plusieurs rôles, c'est le rôle **actif** qui compte

!!! warning "Masking vs Projection Policy vs Row Access"
    | | Masking | Projection | Row Access |
    |---|---|---|---|
    | **Action** | Obfusque la valeur | Bloque l'accès à la colonne | Filtre les lignes |
    | **Résultat** | `***@***.***` | Erreur si tentative d'accès | Lignes invisibles |
    | **Édition** | Enterprise+ | Enterprise+ | Enterprise+ |
