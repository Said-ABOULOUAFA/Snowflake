# Streams & Tasks

## Streams — Capture des changements (CDC) ⭐

Un stream **capture les modifications** (INSERT, UPDATE, DELETE) sur une table source.

```sql
-- Créer un stream sur une table
CREATE STREAM stream_commandes ON TABLE commandes;

-- Lire les changements
SELECT *,
       METADATA$ACTION,        -- INSERT ou DELETE
       METADATA$ISUPDATE,      -- TRUE si c'est une mise à jour
       METADATA$ROW_ID         -- identifiant unique de ligne
FROM stream_commandes;
```

### Colonnes système des streams

| Colonne | Valeur | Signification |
|---|---|---|
| `METADATA$ACTION` | `INSERT` | Nouvelle ligne ou ligne après UPDATE |
| `METADATA$ACTION` | `DELETE` | Ligne supprimée ou ligne avant UPDATE |
| `METADATA$ISUPDATE` | `TRUE` | La ligne fait partie d'un UPDATE |

!!! info "Un UPDATE = 1 DELETE + 1 INSERT"
    Snowflake représente un UPDATE comme une suppression de l'ancienne ligne + insertion de la nouvelle.

### Types de streams

| Type | Capture |
|---|---|
| **Standard** (défaut) | INSERT, UPDATE, DELETE |
| **Append-only** | INSERT uniquement (plus performant) |
| **Insert-only** | INSERT uniquement (pour tables externes) |

```sql
-- Stream append-only (plus léger)
CREATE STREAM stream_logs ON TABLE logs APPEND_ONLY = TRUE;
```

---

## Tasks — Planification automatique ⭐

Une task **exécute du SQL** selon un planning ou quand déclenché.

```sql
-- Task planifiée toutes les heures
CREATE TASK task_traitement_commandes
  WAREHOUSE = wh_etl
  SCHEDULE  = 'USING CRON 0 * * * * UTC'  -- toutes les heures
AS
INSERT INTO commandes_traitees
SELECT * FROM stream_commandes WHERE METADATA$ACTION = 'INSERT';

-- Activer la task (désactivée par défaut !)
ALTER TASK task_traitement_commandes RESUME;

-- Task serverless (pas de warehouse requis)
CREATE TASK task_serverless
  SCHEDULE        = '5 MINUTE'
  USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE = 'SMALL'
AS
MERGE INTO dest USING source ON dest.id = source.id ...;
```

!!! danger "Piège exam"
    Les tasks sont **désactivées par défaut** après création. Toujours faire `ALTER TASK ... RESUME`.

---

## Stream + Task — Pattern CDC classique ⭐

```sql
-- 1. Stream sur la table source
CREATE STREAM stream_src ON TABLE source_table;

-- 2. Task qui consomme le stream
CREATE TASK task_sync
  WAREHOUSE = wh_etl
  SCHEDULE  = '1 MINUTE'
  WHEN SYSTEM$STREAM_HAS_DATA('stream_src')  -- s'exécute seulement si données
AS
INSERT INTO destination
SELECT col1, col2 FROM stream_src
WHERE METADATA$ACTION = 'INSERT';

ALTER TASK task_sync RESUME;
```

!!! tip "SYSTEM$STREAM_HAS_DATA"
    Utilise cette condition dans `WHEN` pour éviter d'exécuter la task inutilement quand le stream est vide.

---

## DAG de tasks (arborescence)

```sql
-- Task parent (déclencheur)
CREATE TASK task_parent SCHEDULE = '1 HOUR' AS SELECT 1;

-- Tasks enfants (déclenchées par le parent)
CREATE TASK task_enfant_1 AFTER task_parent AS INSERT INTO ...;
CREATE TASK task_enfant_2 AFTER task_parent AS UPDATE ...;

-- Activer dans l'ordre : enfants d'abord, parent en dernier
ALTER TASK task_enfant_1 RESUME;
ALTER TASK task_enfant_2 RESUME;
ALTER TASK task_parent RESUME;
```
