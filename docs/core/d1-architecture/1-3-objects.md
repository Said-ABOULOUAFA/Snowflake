# 1.3 — Objets Snowflake (Hiérarchie & Types)

> **Domaine D1 — 31% du COF-C03**

## Types de tables ⭐

| Type | Durée de vie | Time Travel | Fail-safe | Coût stockage |
|---|---|---|---|---|
| **Permanente** | Indéfinie | 0–90 jours | 7 jours | Standard |
| **Temporaire** | Session uniquement | 0–1 jour | ❌ | Réduit |
| **Transitoire** | Indéfinie | 0–1 jour | ❌ | Réduit |
| **Apache Iceberg™** | Indéfinie | Limité | ❌ | Externe |
| **Externe** | Indéfinie | ❌ | ❌ | Externe |
| **Dynamique** | Indéfinie | ✅ | ✅ | Standard |

### Quand utiliser chaque type ?

```sql
-- Table permanente (défaut — OLAP)
CREATE TABLE ventes (id INT, montant FLOAT, date_vente DATE);

-- Table temporaire (calculs intermédiaires dans une session)
CREATE TEMPORARY TABLE tmp_calculs AS
SELECT * FROM ventes WHERE YEAR(date_vente) = 2024;

-- Table transitoire (staging, pas de Fail-safe → économies)
CREATE TRANSIENT TABLE staging_raw (data VARIANT)
  DATA_RETENTION_TIME_IN_DAYS = 0;

-- Table Iceberg (data lake externe)
CREATE ICEBERG TABLE ext_produits (id BIGINT, nom STRING)
  CATALOG = 'SNOWFLAKE'
  EXTERNAL_VOLUME = 'vol_s3'
  BASE_LOCATION = 'produits/';

-- Table hybride (OLTP + OLAP — Unistore)
CREATE HYBRID TABLE commandes (
    id INT PRIMARY KEY,
    client_id INT NOT NULL,
    statut VARCHAR(50),
    INDEX idx_client (client_id)
);
```

!!! danger "Piège exam fréquent"
    - Tables **temporaires et transitoires** → **PAS de Fail-safe**
    - Seules les tables **permanentes** ont le Fail-safe (7 jours)
    - Tables **hybrides** = seules à supporter le verrouillage de lignes et les contraintes référentielles physiques

---

## Types de vues ⭐

| Type | Stockage | DDL visible | Partage | Performances |
|---|---|---|---|---|
| **Standard** | Aucun | ✅ Oui | ❌ Non recommandé | Recalcul à chaque appel |
| **Matérialisée** | Physique | ✅ Oui | ❌ | Résultats pré-calculés |
| **Sécurisée** | Aucun | ❌ Masqué | ✅ Recommandée | Recalcul à chaque appel |

```sql
-- Vue standard
CREATE VIEW vue_ventes_2024 AS
SELECT * FROM ventes WHERE YEAR(date_vente) = 2024;

-- Vue matérialisée (Enterprise+)
CREATE MATERIALIZED VIEW mv_resume AS
SELECT region, SUM(montant) AS total
FROM ventes GROUP BY region;

-- Vue sécurisée (pour partage — DDL masqué)
CREATE SECURE VIEW vue_clients_pub AS
SELECT id, prenom, ville FROM clients WHERE statut = 'actif';

-- Vue matérialisée + sécurisée
CREATE SECURE MATERIALIZED VIEW smv_stats AS
SELECT region, COUNT(*) AS nb FROM clients GROUP BY region;
```

!!! warning "Vue sécurisée = obligatoire pour les Data Shares"
    Quand tu partages des données via un Share, utilise toujours des **vues sécurisées** — sinon le consommateur peut voir la définition SQL.

---

## Stages ⭐

Zones de transit pour les fichiers avant/après chargement.

| Type | Syntaxe | Description |
|---|---|---|
| Stage interne utilisateur | `@~` | Automatique pour chaque user |
| Stage interne table | `@%ma_table` | Automatique pour chaque table |
| Stage interne nommé | `@mon_stage` | Créé manuellement |
| Stage externe S3 | `@stage_s3` | Pointe vers bucket S3 |
| Stage externe Azure | `@stage_azure` | Pointe vers Azure Blob |
| Stage externe GCS | `@stage_gcs` | Pointe vers GCS |

```sql
-- Stage interne nommé
CREATE STAGE mon_stage COMMENT = 'Stage CSV quotidien';

-- Stage externe avec Storage Integration (sécurisé)
CREATE STAGE stage_s3
  URL = 's3://mon-bucket/data/'
  STORAGE_INTEGRATION = si_aws;

-- Lister les fichiers
LIST @mon_stage;
LIST @mon_stage PATTERN = '.*\.csv';

-- Supprimer des fichiers
REMOVE @mon_stage/ancien.csv;
```

---

## Sequences

Génèrent des valeurs numériques uniques et séquentielles.

```sql
CREATE SEQUENCE seq_commande START = 1 INCREMENT = 1;

-- Utiliser dans une table
CREATE TABLE commandes (
    id INT DEFAULT seq_commande.NEXTVAL,
    client VARCHAR(100)
);

SELECT seq_commande.NEXTVAL;  -- prochain numéro
SELECT seq_commande.CURRVAL;  -- valeur courante
```

---

## File Formats ⭐

```sql
-- CSV
CREATE FILE FORMAT fmt_csv
  TYPE = 'CSV' FIELD_DELIMITER = ','
  SKIP_HEADER = 1 NULL_IF = ('NULL','') COMPRESSION = 'AUTO';

-- JSON
CREATE FILE FORMAT fmt_json
  TYPE = 'JSON' STRIP_OUTER_ARRAY = TRUE COMPRESSION = 'AUTO';

-- Parquet
CREATE FILE FORMAT fmt_parquet
  TYPE = 'PARQUET' COMPRESSION = 'SNAPPY';
```

| Format | Type de données |
|---|---|
| CSV, TSV | Structurées |
| JSON, Avro, ORC, Parquet, XML | Semi-structurées |
