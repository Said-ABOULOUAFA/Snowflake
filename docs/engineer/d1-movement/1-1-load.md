# 1.1 Charger des données dans Snowflake

> **Domain 1.0 — Data Movement (28%)**

## Considérations de chargement

| Levier | Bonne pratique |
|---|---|
| **Taille des fichiers** | Cibler **100–250 Mo compressés** par fichier (parallélisme optimal) |
| **Nombre de fichiers** | Plusieurs fichiers > un énorme fichier (un fichier = un thread) |
| **Warehouse** | Taille adaptée au nb de fichiers ; éviter de surdimensionner |
| **Format** | Colonnaire (Parquet) pour analytique ; compresser (gzip) le CSV |

## COPY INTO — features & impacts

```sql
COPY INTO ventes
FROM @stage/ventes/
FILE_FORMAT = (FORMAT_NAME='ff_csv')
ON_ERROR = 'CONTINUE'
MATCH_BY_COLUMN_NAME = 'CASE_INSENSITIVE'   -- mapping par nom de colonne
PURGE = TRUE;                               -- supprime les fichiers après load
```

| Option | Impact |
|---|---|
| `ON_ERROR` | `ABORT_STATEMENT` (défaut), `CONTINUE`, `SKIP_FILE`, `SKIP_FILE_n` |
| `MATCH_BY_COLUMN_NAME` | Charge par nom de colonne (utile semi-structuré) |
| `FORCE` | Recharge des fichiers déjà chargés |
| `VALIDATION_MODE` | `RETURN_ERRORS` / `RETURN_n_ROWS` : valide sans charger |
| `PURGE` | Supprime les fichiers du stage après succès |

!!! tip "Métadonnées de load"
    Snowflake mémorise les fichiers chargés (~64 j) → évite les doublons. `LOAD_HISTORY` / `COPY_HISTORY` pour auditer.

📎 *Réf. : `docs.snowflake.com/en/user-guide/data-load-considerations`*
