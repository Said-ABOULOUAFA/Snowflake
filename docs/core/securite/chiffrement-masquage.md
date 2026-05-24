# Chiffrement & Masquage dynamique

## Chiffrement par défaut

Snowflake chiffre **toutes les données** au repos et en transit, automatiquement.

| Édition | Gestion des clés |
|---|---|
| Standard / Enterprise | Snowflake gère les clés (Snowflake-managed key) |
| **Business Critical** | **Tri-secret key** : clé partagée entre Snowflake + client (HSM) |
| **Virtual Private** | Environnement complètement isolé |

!!! danger "Exam"
    Si une question parle de **"clé gérée par le client"** ou **"Tri-secret"** → réponse = **Business Critical ou Virtual Private**.

---

## Dynamic Data Masking ⭐ (Enterprise+)

Masque automatiquement les données sensibles selon le rôle de l'utilisateur.

```sql
-- 1. Créer la politique de masquage
CREATE MASKING POLICY masque_email
  AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() IN ('ADMIN', 'DBA') THEN val          -- valeur réelle
    WHEN CURRENT_ROLE() IN ('ANALYST') THEN '***@***.***'     -- masqué
    ELSE '** CONFIDENTIEL **'
  END;

-- 2. Appliquer sur une colonne
ALTER TABLE clients MODIFY COLUMN email
  SET MASKING POLICY masque_email;

-- 3. Supprimer la politique
ALTER TABLE clients MODIFY COLUMN email UNSET MASKING POLICY;
```

!!! tip "Points clés"
    - La politique s'applique **à la colonne**, pas à la table
    - Les données **en base ne changent pas** — seul le résultat retourné est masqué
    - Nécessite édition **Enterprise minimum**

---

## Row Access Policies ⭐ (Enterprise+)

Filtre les lignes accessibles selon le rôle ou l'utilisateur.

```sql
-- Politique : chaque région ne voit que ses données
CREATE ROW ACCESS POLICY rap_region
  AS (region_col STRING) RETURNS BOOLEAN ->
  CASE
    WHEN CURRENT_ROLE() = 'ADMIN' THEN TRUE
    ELSE region_col = CURRENT_USER()  -- ou via une table de mapping
  END;

-- Appliquer sur une table
ALTER TABLE ventes ADD ROW ACCESS POLICY rap_region ON (region);
```

!!! warning "Différence clé"
    - **Dynamic Data Masking** → masque des **colonnes** (valeurs visibles mais obfusquées)
    - **Row Access Policy** → filtre des **lignes** (lignes invisibles si condition FALSE)

---

## Object Tagging

Permet de taguer les colonnes sensibles pour la gouvernance.

```sql
-- Créer un tag
CREATE TAG pii_tag ALLOWED_VALUES 'PII', 'Confidentiel', 'Public';

-- Appliquer
ALTER TABLE clients MODIFY COLUMN email SET TAG pii_tag = 'PII';
ALTER TABLE clients MODIFY COLUMN nom SET TAG pii_tag = 'PII';
```
