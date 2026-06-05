# 3.1 Chargement & déchargement de données

> **Domain 3.0 — Data Loading, Unloading & Connectivity (18%)**

## Stages ⭐

| Type de stage | Description |
|---|---|
| **User stage** (`@~`) | Personnel à chaque utilisateur |
| **Table stage** (`@%table`) | Lié à une table |
| **Named internal stage** (`@stage`) | Objet réutilisable, partageable |
| **External stage** | Pointe vers S3 / Azure Blob / GCS via storage integration |

```sql
CREATE STAGE mon_stage
  FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY='"')
  DIRECTORY = (ENABLE = TRUE);            -- directory table

-- External stage
CREATE STAGE ext_stage
  URL = 's3://mon-bucket/data/'
  STORAGE_INTEGRATION = s3_int
  FILE_FORMAT = mon_format;
```

## File formats

```sql
CREATE FILE FORMAT ff_json TYPE = JSON STRIP_OUTER_ARRAY = TRUE;
CREATE FILE FORMAT ff_parquet TYPE = PARQUET;
```

Formats supportés : **CSV, JSON, Avro, ORC, Parquet, XML**.

## COPY INTO (chargement) ⭐

```sql
COPY INTO ventes
FROM @mon_stage/ventes/
FILE_FORMAT = (FORMAT_NAME = 'ff_csv')
ON_ERROR = 'CONTINUE'           -- ABORT_STATEMENT (défaut) | CONTINUE | SKIP_FILE
PURGE = FALSE;
```

| `ON_ERROR` | Effet |
|---|---|
| `ABORT_STATEMENT` | Annule tout le chargement (défaut) |
| `CONTINUE` | Charge les lignes valides, ignore les mauvaises |
| `SKIP_FILE` | Saute le fichier entier en cas d'erreur |

!!! tip "Idempotence"
    COPY garde la trace des fichiers déjà chargés (métadonnées de load, 64 jours). Reload forcé : `FORCE = TRUE`. Valider sans charger : `VALIDATION_MODE = 'RETURN_ERRORS'`.

## Déchargement (unload)

```sql
COPY INTO @mon_stage/export/
FROM (SELECT * FROM ventes WHERE annee = 2024)
FILE_FORMAT = (TYPE = PARQUET)
HEADER = TRUE
MAX_FILE_SIZE = 100000000
SINGLE = FALSE;
```

- Chiffrement côté serveur (SSE) ou côté client.
- Directory tables : exposer les fichiers d'un stage comme une table.

📎 *Réf. : `docs.snowflake.com/en/user-guide/data-load-overview`*
