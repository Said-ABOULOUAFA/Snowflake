# Stages & File Formats

## Types de stages ⭐

| Type | Description | Syntaxe |
|---|---|---|
| **Stage interne utilisateur** | Automatique pour chaque user | `@~` |
| **Stage interne table** | Automatique pour chaque table | `@%ma_table` |
| **Stage interne nommé** | Créé manuellement | `@mon_stage` |
| **Stage externe (S3)** | Pointe vers bucket S3 | `@mon_stage_s3` |
| **Stage externe (Azure)** | Pointe vers Azure Blob | `@mon_stage_azure` |
| **Stage externe (GCS)** | Pointe vers GCS | `@mon_stage_gcs` |

---

## Créer des stages

```sql
-- Stage interne nommé
CREATE STAGE mon_stage
  COMMENT = 'Stage pour les fichiers CSV quotidiens';

-- Stage externe S3
CREATE STAGE stage_s3
  URL = 's3://mon-bucket/data/'
  STORAGE_INTEGRATION = si_aws;  -- recommandé (pas de clés en dur)

-- Stage externe Azure
CREATE STAGE stage_azure
  URL = 'azure://moncompte.blob.core.windows.net/conteneur/'
  STORAGE_INTEGRATION = si_azure;
```

!!! tip "Storage Integration vs Credentials"
    Utilise toujours une **Storage Integration** plutôt que des clés/SAS tokens en dur — plus sécurisé et plus facile à gérer.

---

## File Formats

```sql
-- CSV
CREATE FILE FORMAT fmt_csv
  TYPE = 'CSV'
  FIELD_DELIMITER     = ','
  RECORD_DELIMITER    = '\n'
  SKIP_HEADER         = 1
  NULL_IF             = ('NULL', 'null', '')
  EMPTY_FIELD_AS_NULL = TRUE
  COMPRESSION         = 'AUTO';

-- JSON
CREATE FILE FORMAT fmt_json
  TYPE             = 'JSON'
  STRIP_OUTER_ARRAY = TRUE  -- si le fichier est un tableau JSON: [{...},{...}]
  COMPRESSION      = 'AUTO';

-- Parquet
CREATE FILE FORMAT fmt_parquet
  TYPE        = 'PARQUET'
  COMPRESSION = 'SNAPPY';
```

## Formats supportés

| Format | Structuré | Semi-structuré |
|---|---|---|
| CSV, TSV | ✅ | |
| JSON | | ✅ |
| Avro | | ✅ |
| ORC | | ✅ |
| Parquet | | ✅ |
| XML | | ✅ |

---

## Commandes utiles

```sql
-- Lister les fichiers dans un stage
LIST @mon_stage;
LIST @mon_stage PATTERN = '.*\.csv';

-- Supprimer des fichiers du stage
REMOVE @mon_stage/ancien_fichier.csv;

-- Uploader (via SnowSQL CLI uniquement)
PUT file:///chemin/local/data.csv @mon_stage AUTO_COMPRESS=TRUE;
```
