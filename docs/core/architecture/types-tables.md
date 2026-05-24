# Types de tables Snowflake

## Comparatif des 3 types ⭐

| | Tables Snowflake | Tables Iceberg | Tables Hybrides |
|---|---|---|---|
| **Stockage** | Interne Snowflake | Externe (S3/Azure/GCS) | Interne Snowflake |
| **Format** | Colonnaire optimisé | Format ouvert Iceberg | Row + Index |
| **Cas d'usage** | OLAP / Entrepôt | Data Lake / Lakehouse | OLTP + OLAP (Unistore) |
| **Latence** | Faible | Faible | Très faible |
| **Verrouillage lignes** | ❌ | ❌ | ✅ |
| **Contraintes intégrité** | Logiques uniquement | Logiques uniquement | ✅ Physiques |
| **Vendor lock-in** | Snowflake | ❌ Ouvert | Snowflake |

---

## Tables Snowflake

```sql
CREATE TABLE ventes (
    id          NUMBER AUTOINCREMENT PRIMARY KEY,
    date_vente  DATE,
    montant     FLOAT,
    produit     VARCHAR(100)
);
```

- Micro-partitions automatiques
- Supporte : données **structurées** et **semi-structurées** (JSON, XML, Avro…)
- Données **non structurées** via le type `FILE`

---

## Tables Apache Iceberg™

```sql
-- Iceberg avec catalogue Snowflake (Snowflake-managed)
CREATE ICEBERG TABLE catalogue_produits (
    id     BIGINT,
    nom    STRING,
    prix   DOUBLE
)
CATALOG = 'SNOWFLAKE'
EXTERNAL_VOLUME = 'mon_volume_s3'
BASE_LOCATION = 'produits/';
```

!!! info "Quand utiliser Iceberg ?"
    - Tu as déjà un data lake sur S3/Azure/GCS
    - Tu veux éviter le vendor lock-in Snowflake
    - Tu as besoin d'accès multi-moteurs (Spark, Flink, Trino...)

---

## Tables Hybrides (Unistore)

```sql
CREATE HYBRID TABLE commandes (
    id          INT PRIMARY KEY,
    client_id   INT NOT NULL,
    statut      VARCHAR(50),
    montant     FLOAT,
    INDEX idx_client (client_id)
);
```

!!! warning "Piège exam"
    Les tables hybrides sont les seules à supporter :
    - Le **verrouillage de lignes** (row locking)
    - Les **contraintes d'intégrité référentielle** physiques
    - Les **lectures/écritures aléatoires** par index

---

## Tables permanentes vs temporaires vs transitoires

| Type | Durée de vie | Time Travel | Fail-safe | Coût stockage |
|---|---|---|---|---|
| **Permanente** | Indéfinie | 0–90 jours | 7 jours | Standard |
| **Temporaire** | Session uniquement | 0–1 jour | ❌ | Réduit |
| **Transitoire** | Indéfinie | 0–1 jour | ❌ | Réduit |

```sql
-- Table temporaire (disparaît à la fin de session)
CREATE TEMPORARY TABLE temp_calculs AS
SELECT * FROM ventes WHERE date_vente = CURRENT_DATE();

-- Table transitoire (pas de Fail-safe)
CREATE TRANSIENT TABLE staging_data (
    ligne VARIANT
);
```

!!! tip "Astuce"
    Utilise les tables **transitoires** pour les données de staging/intermédiaires → économise les coûts Fail-safe.
