# Pipelines avancés — Data Transformation (30%)

## Tables dynamiques ⭐

Définissent une transformation **déclarative** — Snowflake gère le rafraîchissement automatiquement.

```sql
-- Créer une table dynamique
CREATE DYNAMIC TABLE ventes_resumees
  TARGET_LAG    = '1 hour'          -- fraîcheur cible des données
  WAREHOUSE     = wh_etl
  COMMENT       = 'Ventes agrégées par jour et région'
AS
SELECT
    DATE_TRUNC('day', date_vente)   AS jour,
    region,
    SUM(montant)                    AS total_montant,
    COUNT(*)                        AS nb_commandes
FROM ventes_raw
GROUP BY 1, 2;

-- Forcer un rafraîchissement manuel
ALTER DYNAMIC TABLE ventes_resumees REFRESH;

-- Voir le statut
SHOW DYNAMIC TABLES;
SELECT * FROM INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY
WHERE NAME = 'VENTES_RESUMEES';
```

### TARGET_LAG

| Valeur | Signification |
|---|---|
| `'1 minute'` | Les données ont au max 1 minute de retard |
| `'1 hour'` | Rafraîchissement toutes les heures max |
| `DOWNSTREAM` | Se synchronise avec les tables dynamiques en aval |

!!! tip "Tables dynamiques vs Streams+Tasks"
    - **Tables dynamiques** = approche déclarative, Snowflake optimise automatiquement
    - **Streams + Tasks** = approche impérative, tu contrôles la logique exacte

---

## Procédures stockées ⭐

```sql
-- JavaScript
CREATE PROCEDURE inserer_si_nouveau(p_id INT, p_nom STRING)
RETURNS STRING
LANGUAGE JAVASCRIPT
AS $$
  var stmt = snowflake.createStatement({
    sqlText: 'SELECT COUNT(*) AS cnt FROM clients WHERE id = ?',
    binds: [P_ID]
  });
  var result = stmt.execute();
  result.next();
  if (result.getColumnValue('CNT') === 0) {
    snowflake.execute({
      sqlText: 'INSERT INTO clients VALUES (?, ?)',
      binds: [P_ID, P_NOM]
    });
    return 'Inséré';
  }
  return 'Déjà existant';
$$;

-- Python (Snowpark)
CREATE PROCEDURE normaliser_donnees(table_name STRING)
RETURNS STRING
LANGUAGE PYTHON
RUNTIME_VERSION = '3.11'
PACKAGES = ('snowflake-snowpark-python', 'pandas')
HANDLER = 'run'
AS $$
import pandas as pd
from snowflake.snowpark.functions import col

def run(session, table_name):
    df = session.table(table_name)
    df_norm = df.with_column('montant_norm',
                             (col('montant') - df.agg({'montant': 'avg'}).collect()[0][0]))
    df_norm.write.mode('overwrite').save_as_table(f'{table_name}_normalized')
    return f'Table {table_name}_normalized créée'
$$;
```

---

## UDFs — Fonctions personnalisées

```sql
-- UDF SQL
CREATE FUNCTION calcul_tva(montant FLOAT, taux FLOAT)
RETURNS FLOAT
AS $$ montant * taux / 100 $$;

-- UDF Python
CREATE FUNCTION parse_json_field(json_str STRING, field STRING)
RETURNS STRING
LANGUAGE PYTHON
RUNTIME_VERSION = '3.11'
HANDLER = 'extract_field'
AS $$
import json

def extract_field(json_str, field):
    try:
        return str(json.loads(json_str).get(field, ''))
    except:
        return None
$$;

-- UDF tabulaire (UDTF) — retourne plusieurs lignes
CREATE FUNCTION split_tags(tags STRING)
RETURNS TABLE (tag STRING)
LANGUAGE PYTHON
RUNTIME_VERSION = '3.11'
HANDLER = 'SplitTags'
AS $$
class SplitTags:
    def process(self, tags):
        for tag in tags.split(','):
            yield (tag.strip(),)
$$;
```

---

## MERGE — Upsert ⭐

```sql
-- Pattern MERGE classique (SCD Type 1)
MERGE INTO clients_dim AS tgt
USING clients_stage AS src
  ON tgt.id = src.id
WHEN MATCHED AND (tgt.nom <> src.nom OR tgt.email <> src.email) THEN
  UPDATE SET
    tgt.nom   = src.nom,
    tgt.email = src.email,
    tgt.updated_at = CURRENT_TIMESTAMP()
WHEN NOT MATCHED THEN
  INSERT (id, nom, email, created_at)
  VALUES (src.id, src.nom, src.email, CURRENT_TIMESTAMP());
```

---

## Fonctions de fenêtre ⭐

```sql
-- Classement, total cumulé, LAG/LEAD
SELECT
    date_vente,
    region,
    montant,
    SUM(montant)   OVER (PARTITION BY region ORDER BY date_vente
                         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumul,
    LAG(montant, 1) OVER (PARTITION BY region ORDER BY date_vente)          AS prev_montant,
    RANK()          OVER (PARTITION BY region ORDER BY montant DESC)         AS rang
FROM ventes;
```
