# 2.3 Monitorer les pipelines de données

> **Domain 2.0 — Performance Optimization (19%)**

## Vues ACCOUNT_USAGE & INFORMATION_SCHEMA

| Source | Latence | Rétention | Usage |
|---|---|---|---|
| `INFORMATION_SCHEMA` (fonctions table) | Temps réel | 7–14 j | Diagnostic immédiat |
| `SNOWFLAKE.ACCOUNT_USAGE` | ~45 min–3 h | 365 j | Historique & audit |

```sql
-- Tâches : historique d'exécution
SELECT name, state, scheduled_time, error_message
FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY())
ORDER BY scheduled_time DESC;

-- Snowpipe : consommation de crédits
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.PIPE_USAGE_HISTORY
ORDER BY start_time DESC;

-- Dépendances de tâches (DAG)
SELECT * FROM TABLE(INFORMATION_SCHEMA.CURRENT_TASK_GRAPHS());
```

## Surveiller un DAG de tâches

```sql
SELECT SYSTEM$TASK_DEPENDENTS_ENABLE('racine_dag');
SELECT * FROM TABLE(INFORMATION_SCHEMA.COMPLETE_TASK_GRAPHS());
```

!!! danger "Piège exam"
    Une tâche enfant ne s'exécute **que** si sa tâche racine est `RESUME` (les tâches sont créées `SUSPENDED`). Activer un DAG = reprendre depuis la racine.

!!! tip "Alerting"
    Combiner `SYSTEM$SEND_EMAIL` + une tâche planifiée, ou les **Alerts** Snowflake, pour notifier en cas d'échec ou de seuil dépassé.

📎 *Réf. : `docs.snowflake.com/en/sql-reference/account-usage`*
