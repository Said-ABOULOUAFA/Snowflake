# Cortex LLM, Semi-structuré avancé & Workflows

## Snowflake Cortex LLM ⭐

### Fonctions AI dans les pipelines

```sql
-- Catégorisation automatique des tickets support
UPDATE tickets
SET categorie = SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
    description,
    ['facturation', 'technique', 'livraison', 'retour', 'autre']
):label::STRING
WHERE categorie IS NULL;

-- Extraction d'informations depuis des emails
SELECT
    id,
    SNOWFLAKE.CORTEX.COMPLETE(
        'mistral-7b',
        'Extrais le montant, la date et le fournisseur de cette facture en JSON. Réponds UNIQUEMENT avec le JSON: ' || texte
    ) AS donnees_extraites
FROM factures_brutes;

-- Pipeline de résumé automatique
INSERT INTO articles_resumes
SELECT
    id,
    titre,
    SNOWFLAKE.CORTEX.SUMMARIZE(contenu) AS resume,
    SNOWFLAKE.CORTEX.SENTIMENT(contenu) AS sentiment,
    CURRENT_TIMESTAMP()                 AS traite_le
FROM articles_nouveaux
WHERE traite = FALSE;
```

### Gestion des coûts Cortex

```sql
-- Voir la consommation Cortex
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.CORTEX_USAGE_HISTORY
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY credits_used DESC;

-- Estimer le coût avant d'exécuter
-- Les tokens sont facturés selon le modèle utilisé
-- mistral-7b < llama3-8b < llama3-70b (en coût)
```

---

## Semi-structuré avancé ⭐

### FLATTEN avec tableaux imbriqués

```sql
-- JSON complexe :
-- {"commande": {"id": 1, "lignes": [{"produit": "A", "qte": 2}, {"produit": "B", "qte": 1}]}}

SELECT
    c.data:commande:id::INT          AS commande_id,
    f.value:produit::STRING          AS produit,
    f.value:qte::INT                 AS quantite
FROM commandes c,
LATERAL FLATTEN(INPUT => c.data:commande:lignes) f;
```

### PARSE_JSON vs OBJECT_CONSTRUCT

```sql
-- Parser une string JSON
SELECT PARSE_JSON('{"nom": "Alice", "age": 30}') AS obj;

-- Construire un JSON
SELECT OBJECT_CONSTRUCT(
    'nom',    prenom || ' ' || nom,
    'email',  email,
    'region', region
) AS json_client
FROM clients;

-- Aplatir vers colonnes (LATERAL FLATTEN sur objet)
SELECT f.key, f.value
FROM TABLE(FLATTEN(input => PARSE_JSON('{"a":1,"b":2,"c":3}'))) f;
```

### Semi-structuré → Structuré

```sql
-- Transformer JSON en table structurée
CREATE TABLE clients_structure AS
SELECT
    data:id::INT              AS id,
    data:nom::STRING          AS nom,
    data:adresse:ville::STRING AS ville,
    data:adresse:pays::STRING  AS pays,
    data:tags[0]::STRING       AS tag_principal
FROM clients_json;
```

### Structuré → Semi-structuré

```sql
-- Créer un JSON depuis des colonnes
SELECT OBJECT_CONSTRUCT(
    'id',      id,
    'client',  OBJECT_CONSTRUCT('nom', nom, 'email', email),
    'montant', montant,
    'produits', ARRAY_CONSTRUCT(produit1, produit2)
) AS commande_json
FROM commandes;
```

---

## Git Integration ⭐

Synchroniser du code SQL/Python depuis un repo Git directement dans Snowflake.

```sql
-- Créer l'intégration API avec GitHub
CREATE API INTEGRATION git_api_github
  API_PROVIDER       = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/Said-ABOULOUAFA/')
  ENABLED            = TRUE;

-- Créer le repository Git dans Snowflake
CREATE GIT REPOSITORY mon_repo
  ORIGIN         = 'https://github.com/Said-ABOULOUAFA/Snowflake.git'
  API_INTEGRATION = git_api_github;

-- Rafraîchir depuis le repo
ALTER GIT REPOSITORY mon_repo FETCH;

-- Voir les fichiers du repo
SHOW GIT BRANCHES IN GIT REPOSITORY mon_repo;
LS @mon_repo/branches/main/;

-- Exécuter un script depuis Git
EXECUTE IMMEDIATE FROM @mon_repo/branches/main/scripts/init.sql;
```

---

## dbt Projects dans Snowflake ⭐

dbt (data build tool) s'intègre nativement avec Snowflake.

```yaml
# profiles.yml — connexion Snowflake
mon_projet:
  target: prod
  outputs:
    prod:
      type: snowflake
      account: mon_compte
      user: dbt_user
      private_key_path: /path/to/key.pem
      database: MA_DB
      warehouse: wh_dbt
      schema: dbt_prod
      threads: 8
```

```sql
-- Modèle dbt (fichier .sql)
-- models/ventes_resumees.sql
{{ config(materialized='incremental', unique_key='id') }}

SELECT
    {{ dbt_utils.surrogate_key(['date_vente', 'region']) }} AS id,
    date_vente,
    region,
    SUM(montant) AS total
FROM {{ source('raw', 'ventes') }}
{% if is_incremental() %}
    WHERE date_vente > (SELECT MAX(date_vente) FROM {{ this }})
{% endif %}
GROUP BY 1, 2, 3
```

### Types de matérialisation dbt

| Type | Description | Equivalent Snowflake |
|---|---|---|
| `table` | Recrée à chaque run | `CREATE OR REPLACE TABLE` |
| `view` | Vue SQL | `CREATE OR REPLACE VIEW` |
| `incremental` | Ajoute seulement les nouvelles données | `MERGE INTO` ou `INSERT` |
| `ephemeral` | CTE temporaire | `WITH ... AS (...)` |
| `dynamic_table` | Table dynamique Snowflake | `CREATE DYNAMIC TABLE` |

---

## Snowflake Notebooks dans les pipelines ⭐

```python
# Notebook Snowflake — pipeline d'ingestion
import snowflake.snowpark.functions as F
from snowflake.snowpark.context import get_active_session

session = get_active_session()

# Étape 1 : Charger les données brutes
df_raw = session.table("staging.commandes_raw")
print(f"Lignes brutes: {df_raw.count()}")

# Étape 2 : Nettoyer
df_clean = (df_raw
    .filter(F.col("id").is_not_null())
    .filter(F.col("montant") > 0)
    .with_column("date_normalise", F.to_date(F.col("date_str"), "DD/MM/YYYY"))
    .drop("date_str")
    .distinct()
)
print(f"Lignes après nettoyage: {df_clean.count()}")

# Étape 3 : Sauvegarder
df_clean.write.mode("append").save_as_table("prod.commandes")
print("Données chargées avec succès ✓")
```
