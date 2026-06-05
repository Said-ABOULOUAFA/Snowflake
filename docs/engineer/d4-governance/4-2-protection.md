# 4.2 Protection des données (masking, row-level, clean rooms)

> **Domain 4.0 — Data Governance (14%)**

## Dynamic Data Masking

```sql
CREATE MASKING POLICY email_mask AS (val STRING) RETURNS STRING ->
  CASE WHEN CURRENT_ROLE() IN ('ADMIN') THEN val
       ELSE REGEXP_REPLACE(val, '.+@', '****@') END;

ALTER TABLE clients MODIFY COLUMN email
  SET MASKING POLICY email_mask;
```

## Row Access Policy (sécurité au niveau ligne)

```sql
CREATE ROW ACCESS POLICY rap_region AS (region STRING) RETURNS BOOLEAN ->
  EXISTS (SELECT 1 FROM mapping_roles
          WHERE role = CURRENT_ROLE() AND region_autorisee = region);

ALTER TABLE ventes ADD ROW ACCESS POLICY rap_region ON (region);
```

| Mécanisme | Granularité | Cas d'usage |
|---|---|---|
| **Masking policy** | Colonne | Masquer PII selon le rôle |
| **Row access policy** | Ligne | Cloisonner par région/tenant |
| **External tokenization** | Colonne | Tokeniser via fonction externe |
| **Tag-based masking** | Colonne (via tag) | Appliquer à grande échelle |

## Data Clean Rooms

Permettent à deux parties de **croiser des données sans se les exposer** (analytique sur intersection, jamais d'accès aux lignes brutes du partenaire).

!!! danger "Piège exam"
    **Masking** = transforme la *valeur affichée* (colonne). **Row access policy** = filtre les *lignes visibles*. Le **tag-based masking** applique une policy à toutes les colonnes portant un tag donné → gouvernance à l'échelle.

!!! tip
    Les policies sont évaluées à l'exécution selon `CURRENT_ROLE()` / `CURRENT_USER()` ; tester avec `USE ROLE` pour valider.

📎 *Réf. : `docs.snowflake.com/en/user-guide/security-column-ddm-intro`*
