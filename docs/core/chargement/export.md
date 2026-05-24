# Export & Unload de données

## COPY INTO \<location\> ⭐

Exporte des données depuis Snowflake vers un stage ou un stockage cloud.

```sql
-- Exporter vers un stage interne
COPY INTO @mon_stage/exports/ventes_2024_
FROM (SELECT * FROM ventes WHERE YEAR(date_vente) = 2024)
FILE_FORMAT     = (TYPE = 'CSV' HEADER = TRUE COMPRESSION = 'GZIP')
OVERWRITE       = TRUE
MAX_FILE_SIZE   = 104857600;   -- 100 MB par fichier

-- Exporter vers S3 directement
COPY INTO 's3://mon-bucket/exports/ventes/'
FROM ventes
STORAGE_INTEGRATION = si_aws
FILE_FORMAT = (TYPE = 'PARQUET' COMPRESSION = 'SNAPPY')
OVERWRITE   = TRUE;

-- Exporter une requête complexe
COPY INTO @stage_azure/rapports/mensuel_
FROM (
    SELECT region,
           DATE_TRUNC('month', date_vente) AS mois,
           SUM(montant) AS total
    FROM ventes
    GROUP BY 1, 2
)
FILE_FORMAT = (TYPE = 'CSV' HEADER = TRUE);
```

---

## Options d'export

| Option | Description | Défaut |
|---|---|---|
| `OVERWRITE` | Écrase les fichiers existants | `FALSE` |
| `SINGLE` | Exporte en un seul fichier | `FALSE` |
| `MAX_FILE_SIZE` | Taille max par fichier (bytes) | 16 MB |
| `HEADER` | Ajoute les noms de colonnes (CSV) | `FALSE` |
| `INCLUDE_QUERY_ID` | Ajoute le query_id au nom du fichier | `FALSE` |

```sql
-- Fichier unique (attention aux gros volumes !)
COPY INTO @mon_stage/export_unique
FROM ma_table
FILE_FORMAT = (TYPE = 'CSV' HEADER = TRUE)
SINGLE = TRUE;
```

---

## Nommage des fichiers exportés

Snowflake génère automatiquement les noms de fichiers :
```
@mon_stage/ventes_0_0_0.csv.gz
@mon_stage/ventes_0_0_1.csv.gz
@mon_stage/ventes_0_0_2.csv.gz
```

Le préfixe que tu donnes est utilisé comme base :
```sql
-- Préfixe "ventes_2024_" → ventes_2024_0_0_0.csv.gz
COPY INTO @mon_stage/ventes_2024_ FROM ventes ...
```

---

## Télécharger depuis un stage (SnowSQL CLI)

```bash
# Télécharger tous les fichiers d'un stage
GET @mon_stage/exports/ file:///chemin/local/exports/;

# Télécharger avec pattern
GET @mon_stage/exports/ file:///chemin/local/ PATTERN='.*2024.*';
```

---

## Formats d'export recommandés

| Format | Avantage | Cas d'usage |
|---|---|---|
| **Parquet + Snappy** | Compressé, rapide | Réingestion, data lake |
| **CSV + GZIP** | Universel | Excel, outils BI |
| **JSON** | Semi-structuré | APIs, NoSQL |
| **ORC** | Hadoop/Spark | Big data ecosystème |

!!! tip "Bonne pratique"
    Utilise `PARQUET` pour réingérer dans Snowflake ou un autre outil analytique — jusqu'à 10x plus compact que CSV.
