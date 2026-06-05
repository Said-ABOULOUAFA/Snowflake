# 5.4 Données semi-structurées (VARIANT, JSON)

> **Domain 5.0 — Data Transformation (25%)**

## Type VARIANT & parsing

```sql
-- Charger du JSON dans une colonne VARIANT
CREATE TABLE events (v VARIANT);
COPY INTO events FROM @stage FILE_FORMAT = (TYPE = JSON);

-- Accès par notation pointée / bracket
SELECT v:user.id::INTEGER       AS user_id,
       v:user.name::STRING      AS nom,
       v:tags[0]::STRING        AS premier_tag
FROM events;
```

## FLATTEN — déplier les tableaux ⭐

```sql
SELECT e.v:id::INT AS event_id, f.value::STRING AS tag
FROM events e,
LATERAL FLATTEN(input => e.v:tags) f;
```

| Fonction | Rôle |
|---|---|
| `PARSE_JSON` | String → VARIANT |
| `FLATTEN` | Déplie tableaux/objets en lignes |
| `OBJECT_CONSTRUCT` | Lignes → objet JSON |
| `ARRAY_AGG` | Lignes → tableau |
| `GET_PATH` / `:` | Naviguer dans la structure |

!!! danger "Piège exam"
    `LATERAL FLATTEN(input => col:chemin)` est LA technique pour transformer un tableau JSON en lignes. `OUTER => TRUE` conserve les lignes sans éléments (équivalent LEFT JOIN). Le **cast** (`::INT`, `::STRING`) est nécessaire car VARIANT est non typé.

!!! tip
    Snowflake stocke le VARIANT en **colonnaire optimisé** : interroger un sous-champ ne lit que ce champ → performant même sur gros JSON.

📎 *Réf. : `docs.snowflake.com/en/user-guide/semistructured-concepts`*
