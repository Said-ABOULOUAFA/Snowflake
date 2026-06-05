# 4.1 Monitorer les données (tagging, classification, lineage)

> **Domain 4.0 — Data Governance (14%)**

## Object Tagging

```sql
CREATE TAG cost_center;
ALTER TABLE ventes SET TAG cost_center = 'finance';

-- Retrouver les objets par tag
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.TAG_REFERENCES
WHERE tag_name = 'COST_CENTER';
```

## Data Classification

Identifie automatiquement les colonnes sensibles (PII) et propose des catégories (`SEMANTIC`/`PRIVACY`).

```sql
CALL SYSTEM$CLASSIFY('mydb.myschema.clients', {'auto_tag': true});
SELECT * FROM TABLE(INFORMATION_SCHEMA.CLASSIFY('clients'));
```

## Access History & Lineage

```sql
-- Qui a lu/écrit quoi (audit d'accès)
SELECT query_id, user_name, direct_objects_accessed, objects_modified
FROM SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY
ORDER BY query_start_time DESC;
```

| Vue | Usage |
|---|---|
| `ACCESS_HISTORY` | Lineage colonne-à-colonne, audit RGPD |
| `OBJECT_DEPENDENCIES` | Dépendances entre objets |
| `TAG_REFERENCES` | Cartographie des tags |

!!! danger "Piège exam"
    `ACCESS_HISTORY` fournit le **data lineage** (colonnes lues `base_objects_accessed` vs modifiées `objects_modified`) — c'est la source pour tracer la propagation des données sensibles. Disponible en édition **Enterprise+**.

📎 *Réf. : `docs.snowflake.com/en/user-guide/access-history`*
