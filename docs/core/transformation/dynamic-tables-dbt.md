# Tables Dynamiques & dbt

## Tables Dynamiques ⭐

Une table dynamique est une **table physique** dont le contenu est automatiquement maintenu à jour par Snowflake.

```sql
-- Créer une table dynamique simple
CREATE DYNAMIC TABLE ventes_resumees
  TARGET_LAG = '1 hour'
  WAREHOUSE  = wh_etl
AS
SELECT
    DATE_TRUNC('day', date_vente) AS jour,
    region,
    SUM(montant)                  AS total,
    COUNT(*)                      AS nb_commandes
FROM ventes_raw
GROUP BY 1, 2;

-- Forcer un rafraîchissement immédiat
ALTER DYNAMIC TABLE ventes_resumees REFRESH;

-- Suspendre / reprendre
ALTER DYNAMIC TABLE ventes_resumees SUSPEND;
ALTER DYNAMIC TABLE ventes_resumees RESUME;

-- Voir le statut
SHOW DYNAMIC TABLES;
```

---

## TARGET_LAG ⭐

Définit la **fraîcheur maximale** des données.

| Valeur | Signification |
|---|---|
| `'1 minute'` | Données ont max 1 min de retard |
| `'1 hour'` | Rafraîchissement toutes les heures max |
| `'1 day'` | Rafraîchissement quotidien max |
| `DOWNSTREAM` | Se synchronise avec les tables dynamiques en aval |

```sql
-- Chaîner des tables dynamiques
CREATE DYNAMIC TABLE dt_bronze
  TARGET_LAG = '5 minutes' WAREHOUSE = wh_etl
AS SELECT * FROM source_raw;

CREATE DYNAMIC TABLE dt_silver
  TARGET_LAG = DOWNSTREAM WAREHOUSE = wh_etl  -- attend dt_bronze
AS SELECT * FROM dt_bronze WHERE valide = TRUE;

CREATE DYNAMIC TABLE dt_gold
  TARGET_LAG = DOWNSTREAM WAREHOUSE = wh_etl  -- attend dt_silver
AS SELECT region, SUM(montant) FROM dt_silver GROUP BY 1;
```

!!! tip "DOWNSTREAM = Médaillon Architecture"
    Utilise `DOWNSTREAM` pour construire des pipelines Bronze → Silver → Gold automatiques.

---

## Tables Dynamiques vs Streams+Tasks ⭐

| | Tables Dynamiques | Streams + Tasks |
|---|---|---|
| **Approche** | Déclarative (le QUOI) | Impérative (le COMMENT) |
| **Gestion** | Snowflake optimise tout | Tu contrôles la logique |
| **Complexité** | Simple | Plus flexible |
| **MERGE/DELETE** | Automatique | À écrire manuellement |
| **Monitoring** | Refresh History intégré | TASK_HISTORY |
| **Cas d'usage** | Transformations SQL pures | Logique conditionnelle complexe |

---

## dbt avec Snowflake ⭐

dbt (data build tool) transforme les données via des modèles SQL versionnés.

### Types de matérialisation dbt

```sql
-- table : recrée à chaque run
{{ config(materialized='table') }}
SELECT * FROM {{ source('raw', 'ventes') }}

-- view : vue SQL simple
{{ config(materialized='view') }}
SELECT * FROM {{ ref('stg_ventes') }} WHERE montant > 0

-- incremental : ajoute seulement les nouvelles données
{{ config(materialized='incremental', unique_key='id') }}
SELECT * FROM {{ source('raw', 'ventes') }}
{% if is_incremental() %}
  WHERE date_vente > (SELECT MAX(date_vente) FROM {{ this }})
{% endif %}

-- dynamic_table : table dynamique Snowflake
{{ config(
    materialized='dynamic_table',
    target_lag='1 hour',
    snowflake_warehouse='wh_dbt'
) }}
SELECT region, SUM(montant) AS total
FROM {{ ref('stg_ventes') }}
GROUP BY 1
```

### Commandes dbt essentielles

```bash
# Exécuter tous les modèles
dbt run

# Exécuter un modèle spécifique
dbt run --select ventes_resumees

# Tester la qualité des données
dbt test

# Générer la documentation
dbt docs generate
dbt docs serve

# Exécuter en mode incrémental
dbt run --full-refresh  # force la recréation complète
```

### Tests dbt natifs

```yaml
# schema.yml
models:
  - name: ventes
    columns:
      - name: id
        tests:
          - unique
          - not_null
      - name: region
        tests:
          - accepted_values:
              values: ['EMEA', 'AMER', 'APAC']
      - name: client_id
        tests:
          - relationships:
              to: ref('clients')
              field: id
```
