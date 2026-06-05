# 1.3 Troubleshoot l'ingestion

> **Domain 1.0 — Data Movement (28%)**

## Identifier les causes d'erreur

```sql
-- Historique de chargement (erreurs)
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.COPY_HISTORY
WHERE table_name = 'VENTES' AND error_count > 0
ORDER BY last_load_time DESC;

-- Valider un fichier sans charger
COPY INTO ventes FROM @stage VALIDATION_MODE = 'RETURN_ALL_ERRORS';

-- Inspecter les erreurs d'un load récent
SELECT * FROM TABLE(VALIDATE(ventes, JOB_ID => '_last'));
```

## Causes fréquentes & résolutions ⭐

| Symptôme | Cause | Résolution |
|---|---|---|
| `Number of columns mismatch` | Délimiteur / colonnes | Ajuster file format, `ERROR_ON_COLUMN_COUNT_MISMATCH=FALSE` |
| Dates mal parsées | Format de date | `DATE_FORMAT`, `TIMESTAMP_FORMAT` |
| Encodage | Caractères non UTF-8 | `ENCODING` |
| Fichiers ignorés | Déjà chargés | `FORCE=TRUE` ou vérifier load metadata |
| Lignes rejetées | Données invalides | `ON_ERROR=CONTINUE` + audit `VALIDATE()` |

!!! tip "Snowpipe"
    Pour Snowpipe : `SELECT SYSTEM$PIPE_STATUS('mon_pipe');` et `COPY_HISTORY` pour diagnostiquer les fichiers non ingérés ou les notifications manquantes.

📎 *Réf. : `docs.snowflake.com/en/sql-reference/functions/validate`*
