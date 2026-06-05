# 1.4 — Entrepôts Virtuels (Virtual Warehouses)

> **Domaine D1 — 31% du COF-C03**

## Types de warehouses ⭐

| Type | Description | Cas d'usage |
|---|---|---|
| **Standard Gen 1** | Warehouse classique | SQL général, ETL, BI |
| **Standard Gen 2** | Optimisé CPU/mémoire (nouveau) | Charges analytiques lourdes |
| **Snowpark-Optimized** | 16x plus de mémoire par nœud | ML, Snowpark Python intensif |

```sql
-- Warehouse standard
CREATE WAREHOUSE wh_bi
  WAREHOUSE_SIZE = 'MEDIUM'
  AUTO_SUSPEND = 60
  AUTO_RESUME = TRUE;

-- Warehouse Snowpark-optimisé
CREATE WAREHOUSE wh_ml
  WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'
  WAREHOUSE_SIZE = 'MEDIUM'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE;
```

---

## Tailles et crédits ⭐

| Taille | Crédits/heure | Cas d'usage |
|---|---|---|
| X-Small (XS) | 1 | Dev, tests légers |
| Small (S) | 2 | Requêtes légères, BI simple |
| Medium (M) | 4 | BI standard, ETL modéré |
| Large (L) | 8 | ETL, agrégations lourdes |
| X-Large (XL) | 16 | Gros volumes |
| 2X-Large | 32 | Data Science, ML |
| 3X-Large | 64 | Très gros volumes |
| 4X-Large | 128 | Charges extrêmes |

!!! info "Règle simple"
    Chaque taille **double** le nombre de nœuds et le coût en crédits.
    Facturation **à la seconde** (minimum 60 secondes par démarrage).

---

## Configuration selon les cas d'usage ⭐

```sql
-- Requêtes ad-hoc (analyses ponctuelles)
CREATE WAREHOUSE wh_adhoc
  WAREHOUSE_SIZE = 'SMALL'
  AUTO_SUSPEND = 60      -- suspend rapidement
  AUTO_RESUME = TRUE
  STATEMENT_TIMEOUT_IN_SECONDS = 300;

-- Chargement de données (ETL)
CREATE WAREHOUSE wh_etl
  WAREHOUSE_SIZE = 'LARGE'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE
  MAX_CONCURRENCY_LEVEL = 4;

-- BI et reporting (haute concurrence)
CREATE WAREHOUSE wh_bi
  WAREHOUSE_SIZE    = 'MEDIUM'
  MIN_CLUSTER_COUNT = 1         -- Enterprise+
  MAX_CLUSTER_COUNT = 4         -- Enterprise+
  SCALING_POLICY    = 'STANDARD'
  AUTO_SUSPEND      = 120
  AUTO_RESUME       = TRUE;
```

---

## Scaling : Scale Up vs Scale Out ⭐

| Situation | Solution | Mécanisme |
|---|---|---|
| **Requête lente, spillage** | **Scale Up** (taille +) | Augmenter la taille du warehouse |
| **Trop de requêtes en queue** | **Scale Out** (clusters +) | Multi-cluster warehouse |

### Multi-cluster Warehouse (Enterprise+)

```sql
CREATE WAREHOUSE wh_haute_concurrence
  WAREHOUSE_SIZE    = 'MEDIUM'
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 5
  SCALING_POLICY    = 'STANDARD'   -- réactif (ajoute dès qu'une requête attend)
  -- ou SCALING_POLICY = 'ECONOMY' -- économe (attend plus longtemps)
  AUTO_SUSPEND      = 120
  AUTO_RESUME       = TRUE;
```

!!! danger "Piège exam"
    - `STANDARD` : ajoute un cluster **dès qu'une requête attend** → priorité performance
    - `ECONOMY` : attend d'avoir assez de requêtes en attente → priorité coût
    - Multi-cluster nécessite **édition Enterprise minimum**

---

## Auto-Suspend & Auto-Resume ⭐

```sql
-- Modifier les paramètres à chaud
ALTER WAREHOUSE wh_bi SET AUTO_SUSPEND = 60;    -- secondes
ALTER WAREHOUSE wh_bi SET AUTO_RESUME = TRUE;

-- Suspendre/reprendre manuellement
ALTER WAREHOUSE wh_bi SUSPEND;
ALTER WAREHOUSE wh_bi RESUME;

-- Voir le statut
SHOW WAREHOUSES;
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE warehouse_name = 'WH_BI'
ORDER BY start_time DESC LIMIT 10;
```

!!! tip "Bonnes pratiques auto-suspend"
    - **Ad-hoc / interactif** : 60–120 secondes
    - **BI dashboards** : 300–600 secondes (évite de recréer le cache)
    - **ETL de nuit** : peut être suspendu entre les runs

---

## Workloads recommandés par warehouse

| Équipe | Taille | Scaling | Suspend |
|---|---|---|---|
| Data Engineers (ETL) | Large | Non | 300s |
| Analystes BI | Medium | Multi-cluster | 120s |
| Data Scientists | XL ou Snowpark-Optimized | Non | 600s |
| Développeurs / Tests | XS ou Small | Non | 60s |

!!! tip "Isoler les workloads"
    Créer des warehouses séparés par équipe ou type de charge évite qu'une requête lourde impacte les autres utilisateurs.
