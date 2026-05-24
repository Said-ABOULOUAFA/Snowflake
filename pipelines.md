# Ingestion avancée — INFER_SCHEMA, Tables Externes, Openflow

## INFER_SCHEMA ⭐

Détecte automatiquement le schéma d'un fichier stagé.

```sql
-- Détecter le schéma d'un fichier Parquet
SELECT *
FROM TABLE(
    INFER_SCHEMA(
        LOCATION     => '@mon_stage/data.parquet',
        FILE_FORMAT  => 'fmt_parquet'
    )
);
-- Retourne : COLUMN_NAME, TYPE, NULLABLE, EXPRESSION, FILENAMES

-- Créer une table directement depuis le schéma détecté
CREATE TABLE ma_table
  USING TEMPLATE (
    SELECT ARRAY_AGG(OBJECT_CONSTRUCT(*))
    FROM TABLE(
        INFER_SCHEMA(
            LOCATION    => '@mon_stage/',
            FILE_FORMAT => 'fmt_parquet'
        )
    )
  );
```

!!! tip "Formats supportés"
    Parquet, Avro, ORC, JSON, CSV (avec header)

---

## Tables Externes ⭐

Accéder à des fichiers dans un stockage cloud **sans les charger** dans Snowflake.

```sql
-- Créer une table externe sur S3
CREATE EXTERNAL TABLE ext_ventes (
    date_vente DATE          AS (VALUE:c1::DATE),
    region     STRING        AS (VALUE:c2::STRING),
    montant    FLOAT         AS (VALUE:c3::FLOAT)
)
LOCATION        = @stage_s3/ventes/
FILE_FORMAT     = (TYPE = 'PARQUET')
AUTO_REFRESH    = TRUE;      -- rafraîchit auto via notifications S3

-- Rafraîchir manuellement les métadonnées
ALTER EXTERNAL TABLE ext_ventes REFRESH;

-- Requêter comme une table normale
SELECT date_vente, SUM(montant) AS total
FROM ext_ventes
GROUP BY date_vente;
```

!!! warning "Limitations tables externes"
    - Lecture seule (pas d'INSERT/UPDATE/DELETE)
    - Performances moins bonnes que les tables Snowflake natives
    - Nécessite un stage externe (S3, Azure, GCS)
    - Pas de Time Travel ni Fail-safe

### Iceberg vs Table Externe

| | Table Iceberg | Table Externe |
|---|---|---|
| **Format** | Apache Iceberg | Parquet, CSV, JSON... |
| **Transactions ACID** | ✅ | ❌ |
| **Time Travel** | ✅ (limité) | ❌ |
| **Performances** | Meilleures | Standard |
| **Cas d'usage** | Data Lakehouse | Accès direct fichiers |

---

## Directory Tables ⭐

Métadonnées des fichiers d'un stage (nom, taille, date, chemin).

```sql
-- Activer le directory sur un stage
CREATE STAGE stage_docs
  URL = 's3://mon-bucket/docs/'
  DIRECTORY = (ENABLE = TRUE, AUTO_REFRESH = TRUE);

-- Rafraîchir manuellement
ALTER STAGE stage_docs REFRESH;

-- Lister les fichiers avec métadonnées
SELECT *
FROM DIRECTORY(@stage_docs);
-- Colonnes: RELATIVE_PATH, SIZE, LAST_MODIFIED, MD5, ETAG, FILE_URL

-- Filtrer par type de fichier
SELECT relative_path, size, last_modified
FROM DIRECTORY(@stage_docs)
WHERE relative_path LIKE '%.pdf';

-- Générer des URLs d'accès
SELECT relative_path,
       BUILD_SCOPED_FILE_URL(@stage_docs, relative_path) AS url_securisee
FROM DIRECTORY(@stage_docs);
```

---

## Openflow

Connecteur natif Snowflake pour ingérer des données depuis des sources SaaS (SharePoint, Google Drive, Salesforce, etc.) **sans code**.

!!! info "Caractéristiques"
    - Configuration via Snowsight (pas de code)
    - Gestion automatique de la planification et du rafraîchissement
    - Transformations légères intégrées
    - **Note COF-C03** : ne sera pas testé tant qu'il n'est pas GA globalement

---

## Unload — Export de données ⭐

```sql
-- Exporter vers un stage interne
COPY INTO @mon_stage/export/ventes_
FROM (SELECT * FROM ventes WHERE YEAR(date_vente) = 2024)
FILE_FORMAT     = (TYPE = 'PARQUET')
INCLUDE_QUERY_ID = TRUE;   -- ajoute le query_id au nom du fichier

-- Exporter vers S3 directement
COPY INTO 's3://mon-bucket/exports/ventes/'
FROM ventes
STORAGE_INTEGRATION = si_aws
FILE_FORMAT = (TYPE = 'CSV' COMPRESSION = 'GZIP')
OVERWRITE   = TRUE
MAX_FILE_SIZE = 104857600;  -- 100 MB par fichier

-- Télécharger (via SnowSQL CLI)
GET @mon_stage/export/ file:///chemin/local/;
```

### Options d'export utiles

| Option | Description |
|---|---|
| `OVERWRITE = TRUE` | Écrase les fichiers existants |
| `SINGLE = TRUE` | Exporte en un seul fichier |
| `MAX_FILE_SIZE` | Taille max par fichier (défaut 16 MB) |
| `HEADER = TRUE` | Ajoute les noms de colonnes (CSV) |
