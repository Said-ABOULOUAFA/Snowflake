# 1.5 — Stockage : Micro-partitions, Tables & Vues

> **Domaine D1 — 31% du COF-C03**

## Micro-partitions ⭐

Unité fondamentale de stockage dans Snowflake.

```
Table ventes (multi-TB)
├── Micro-partition 1  │ date: 2024-01-01→2024-01-10 │ region: EMEA, AMER │ min_montant: 10 │ max_montant: 9800
├── Micro-partition 2  │ date: 2024-01-11→2024-01-20 │ region: EMEA       │ min_montant: 5  │ max_montant: 7500
├── Micro-partition 3  │ date: 2024-01-21→2024-01-31 │ region: APAC, AMER │ min_montant: 20 │ max_montant: 12000
└── ...
```

| Caractéristique | Valeur |
|---|---|
| Taille | 50–500 MB (non compressé) |
| Format | Colonnaire |
| Gestion | 100% automatique |
| Contenu | Statistiques min/max par colonne |
| Compression | Automatique par Snowflake |

### Pruning automatique ⭐

Snowflake stocke les **min/max** de chaque colonne dans chaque micro-partition. Lors d'une requête avec filtre, seules les micro-partitions concernées sont lues.

```sql
-- Snowflake ne lit QUE les micro-partitions avec date dans [01-01 → 01-31]
SELECT * FROM ventes WHERE date_vente BETWEEN '2024-01-01' AND '2024-01-31';

-- Voir le pruning dans le Query Profile
-- → "Partitions scanned" vs "Partitions total"
-- Exemple: 15 / 10,000 = excellent pruning ✅

-- ❌ Mauvais : fonction sur la colonne filtrée → empêche le pruning !
SELECT * FROM ventes WHERE YEAR(date_vente) = 2024;

-- ✅ Bon : filtre sur valeur directe
SELECT * FROM ventes WHERE date_vente BETWEEN '2024-01-01' AND '2024-12-31';
```

---

## Data Clustering ⭐

Pour les **très grandes tables** (multi-TB) où l'ordre naturel d'insertion ne correspond pas aux patterns de requêtes.

```sql
-- Ajouter une clé de clustering
ALTER TABLE ventes CLUSTER BY (date_vente, region);

-- Clustering sur expression
ALTER TABLE ventes CLUSTER BY (DATE_TRUNC('month', date_vente));

-- Vérifier l'état du clustering
SELECT SYSTEM$CLUSTERING_INFORMATION('ventes', '(date_vente)');
-- → average_depth (idéal: proche de 1)
-- → average_overlaps (idéal: proche de 0)

-- Activer/suspendre le reclustering automatique
ALTER TABLE ventes RESUME RECLUSTER;
ALTER TABLE ventes SUSPEND RECLUSTER;
```

!!! danger "Clustering Depth"
    - Valeur **proche de 1** = excellent clustering
    - Valeur **élevée** = mauvais clustering → pruning inefficace

---

## Types de tables — Récapitulatif ⭐

```sql
-- Permanente (défaut)
CREATE TABLE clients (id INT, nom STRING, email STRING);

-- Temporaire (disparaît à la fin de session)
CREATE TEMPORARY TABLE tmp_analyse AS SELECT ...;

-- Transitoire (pas de Fail-safe → économies sur staging)
CREATE TRANSIENT TABLE stg_ventes (data VARIANT)
  DATA_RETENTION_TIME_IN_DAYS = 0;

-- Iceberg (data lake externe)
CREATE ICEBERG TABLE ice_commandes (id BIGINT, total DOUBLE)
  CATALOG = 'SNOWFLAKE'
  EXTERNAL_VOLUME = 'vol_aws'
  BASE_LOCATION = 'commandes/';

-- Externe (lecture seule sur fichiers)
CREATE EXTERNAL TABLE ext_produits (
    nom STRING AS (VALUE:c1::STRING),
    prix FLOAT AS (VALUE:c2::FLOAT)
)
LOCATION = @stage_s3/produits/
FILE_FORMAT = (TYPE = 'PARQUET');

-- Dynamique (rafraîchissement automatique)
CREATE DYNAMIC TABLE dt_resume
  TARGET_LAG = '1 hour'
  WAREHOUSE = wh_etl
AS SELECT region, SUM(montant) AS total FROM ventes GROUP BY region;

-- Hybride (OLTP + OLAP, index, verrouillage)
CREATE HYBRID TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    status VARCHAR(20),
    INDEX idx_customer (customer_id)
);
```

---

## Types de vues — Récapitulatif ⭐

```sql
-- Vue standard
CREATE VIEW v_ventes AS SELECT * FROM ventes WHERE annee = 2024;

-- Vue matérialisée (résultats pré-calculés, Enterprise+)
CREATE MATERIALIZED VIEW mv_ventes_region AS
SELECT region, SUM(montant) AS total FROM ventes GROUP BY region;
-- ⚠️ Ne supporte pas : HAVING, JOIN, sous-requêtes, LIMIT, ORDER BY

-- Vue sécurisée (DDL masqué, obligatoire pour les Shares)
CREATE SECURE VIEW sv_clients AS
SELECT id, nom FROM clients WHERE statut = 'actif';

-- Vue sécurisée + matérialisée
CREATE SECURE MATERIALIZED VIEW smv_kpi AS
SELECT region, COUNT(*) AS nb_clients FROM clients GROUP BY region;
```

!!! warning "Vue matérialisée — limitations importantes"
    Ne supporte **pas** : `HAVING`, `JOIN`, sous-requêtes, `LIMIT`, `ORDER BY`.
    Coût : crédits de maintenance + stockage des résultats.
    Nécessite **Enterprise minimum**.
