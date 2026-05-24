# Snowpark & UDFs avancés — Data Transformation (30%)

## Architecture Snowpark ⭐

Snowpark permet d'écrire du code Python/Java/Scala qui **s'exécute directement dans Snowflake** (pas de transfert de données).

```
Code Python/Java/Scala
        ↓
  Snowpark API
        ↓
  Plan logique
        ↓
  Optimiseur Snowflake
        ↓
  Exécution dans le warehouse
```

---

## Snowpark Python — DataFrames ⭐

```python
from snowflake.snowpark import Session
from snowflake.snowpark.functions import col, sum as sum_, avg, when, lit
from snowflake.snowpark.types import IntegerType, StringType

# Connexion
session = Session.builder.configs({
    "account":   "mon_compte",
    "user":      "mon_user",
    "password":  "mot_de_passe",
    "warehouse": "wh_dev",
    "database":  "ma_db",
    "schema":    "public"
}).create()

# Lire une table
df = session.table("ventes")

# Filtrer et agréger
df_resume = (df
    .filter(col("region") == "EMEA")
    .group_by("region", "date_vente")
    .agg(
        sum_("montant").alias("total"),
        avg("montant").alias("moyenne")
    )
    .sort(col("date_vente").desc())
)

# Afficher (collecte les données)
df_resume.show()

# Écrire dans une table
df_resume.write.mode("overwrite").save_as_table("ventes_resume_emea")

# Écrire en mode append
df_resume.write.mode("append").save_as_table("ventes_resume_emea")
```

### Transformations courantes

```python
# Ajouter une colonne calculée
df = df.with_column("tva", col("montant") * lit(0.2))

# Renommer
df = df.rename(col("montant"), "chiffre_affaires")

# Jointure
df_clients = session.table("clients")
df_joined = df.join(df_clients, df["client_id"] == df_clients["id"], "left")

# CASE WHEN
df = df.with_column("categorie",
    when(col("montant") > 1000, lit("Premium"))
    .when(col("montant") > 100, lit("Standard"))
    .otherwise(lit("Petit"))
)

# Déduplique
df = df.distinct()

# Union
df_total = df_2023.union(df_2024)
```

---

## Snowpark UDFs ⭐

### UDF Python Scalaire

```python
# Définir et enregistrer une UDF
from snowflake.snowpark.functions import udf
from snowflake.snowpark.types import StringType, IntegerType

@udf(name="masquer_email", is_permanent=True,
     stage_location="@mon_stage", replace=True)
def masquer_email(email: str) -> str:
    if email and '@' in email:
        parts = email.split('@')
        return parts[0][:2] + '***@' + parts[1]
    return '***'

# Utiliser en SQL
# SELECT masquer_email(email) FROM clients;
```

### UDF Vectorisée (plus performante)

```python
import pandas as pd
from snowflake.snowpark.functions import pandas_udf
from snowflake.snowpark.types import PandasSeriesType, StringType

@pandas_udf(name="normaliser_texte", is_permanent=True,
            stage_location="@mon_stage", replace=True)
def normaliser_texte(series: pd.Series) -> pd.Series:
    return series.str.lower().str.strip()
```

---

## UDAFs — User-Defined Aggregate Functions ⭐

Fonctions d'agrégation personnalisées (comme SUM, AVG mais custom).

```python
from snowflake.snowpark.functions import udaf
from snowflake.snowpark.types import IntegerType

class MedianeHandler:
    def __init__(self):
        self._values = []

    @property
    def aggregate_state(self):
        return self._values

    def accumulate(self, val):
        if val is not None:
            self._values.append(val)

    def merge(self, other_state):
        self._values.extend(other_state)

    def finish(self):
        if not self._values:
            return None
        sorted_vals = sorted(self._values)
        n = len(sorted_vals)
        mid = n // 2
        return sorted_vals[mid] if n % 2 else (sorted_vals[mid-1] + sorted_vals[mid]) // 2

session.udaf.register(MedianeHandler, name="mediane", replace=True,
                      return_type=IntegerType(), input_types=[IntegerType()])

# Utiliser en SQL
# SELECT mediane(montant) FROM ventes GROUP BY region;
```

---

## External Functions ⭐

Appelle une API externe (AWS Lambda, Azure Functions) depuis SQL.

```sql
-- Créer l'intégration API
CREATE API INTEGRATION api_scoring
  API_PROVIDER        = aws_api_gateway
  API_AWS_ROLE_ARN    = 'arn:aws:iam::123456789:role/snowflake-role'
  API_ALLOWED_PREFIXES = ('https://abc123.execute-api.eu-west-1.amazonaws.com/prod/')
  ENABLED             = TRUE;

-- Créer la fonction externe
CREATE EXTERNAL FUNCTION scoring_client(client_id INTEGER)
  RETURNS VARIANT
  API_INTEGRATION = api_scoring
  AS 'https://abc123.execute-api.eu-west-1.amazonaws.com/prod/score';

-- Utiliser en SQL
SELECT client_id,
       scoring_client(client_id) AS score
FROM clients
LIMIT 100;
```

!!! warning "Considérations"
    - Latence plus élevée qu'une UDF native (appel réseau)
    - Facturation : crédits Snowflake + coût de l'API externe
    - Idéal pour : modèles ML hébergés ailleurs, services de tokenisation

---

## Snowpark Stored Procedures ⭐

```python
# Procédure stockée Python avec Snowpark
def nettoyer_staging(session: Session, table_name: str) -> str:
    # Supprimer les doublons
    df = session.table(table_name)
    df_clean = df.distinct()
    df_clean.write.mode("overwrite").save_as_table(f"{table_name}_clean")

    # Supprimer les lignes avec valeurs nulles critiques
    df_final = session.table(f"{table_name}_clean") \
        .filter(col("id").is_not_null()) \
        .filter(col("date").is_not_null())
    df_final.write.mode("overwrite").save_as_table(f"{table_name}_clean")

    count = df_final.count()
    return f"Table nettoyée : {count} lignes conservées"

# Enregistrer la procédure
session.sproc.register(
    func           = nettoyer_staging,
    name           = "nettoyer_staging",
    packages       = ["snowflake-snowpark-python"],
    is_permanent   = True,
    stage_location = "@mon_stage",
    replace        = True
)

# Appeler en SQL
-- CALL nettoyer_staging('VENTES_RAW');
```

---

## Gestion des transactions

```sql
-- Transaction manuelle
BEGIN;
  UPDATE comptes SET solde = solde - 100 WHERE id = 1;
  UPDATE comptes SET solde = solde + 100 WHERE id = 2;
COMMIT;

-- Annuler en cas d'erreur
BEGIN;
  DELETE FROM staging WHERE date < '2023-01-01';
  -- Si erreur :
ROLLBACK;

-- Dans une procédure JavaScript
var stmt = snowflake.createStatement({sqlText: "BEGIN"});
stmt.execute();
try {
    // opérations...
    snowflake.execute({sqlText: "COMMIT"});
} catch(e) {
    snowflake.execute({sqlText: "ROLLBACK"});
    throw e;
}
```
