# Snowpark

## Qu'est-ce que Snowpark ? ⭐

Snowpark permet d'écrire du code **Python, Java ou Scala** qui s'exécute directement dans Snowflake — sans déplacer les données.

```
Code Python/Java/Scala  →  Snowpark API  →  Optimiseur Snowflake  →  Warehouse
```

!!! info "Avantages clés"
    - Les données **ne sortent jamais** de Snowflake → sécurité maximale
    - Même moteur d'exécution que SQL → performances optimales
    - Supporte les librairies Python populaires (pandas, scikit-learn, etc.)

---

## Snowpark Python — Bases ⭐

```python
from snowflake.snowpark import Session
from snowflake.snowpark.functions import col, sum as sum_, lit, when

# Connexion
session = Session.builder.configs({
    "account":   "mon_compte.eu-west-1",
    "user":      "mon_user",
    "password":  "mon_password",
    "warehouse": "wh_dev",
    "database":  "ma_db",
    "schema":    "public"
}).create()

# Lire une table → DataFrame (lazy, pas encore exécuté)
df = session.table("ventes")

# Filtrer
df_emea = df.filter(col("region") == "EMEA")

# Sélectionner des colonnes
df_select = df.select("id", "date_vente", "montant", "region")

# Ajouter une colonne calculée
df_tva = df.with_column("montant_ttc", col("montant") * lit(1.2))

# Agréger
df_resume = df.group_by("region").agg(
    sum_("montant").alias("total"),
)

# Afficher (déclenche l'exécution)
df_resume.show()

# Écrire dans une table
df_resume.write.mode("overwrite").save_as_table("ventes_par_region")
```

---

## DataFrames vs SQL

```python
# Snowpark DataFrame
df.filter(col("region") == "EMEA") \
  .group_by("date_vente") \
  .agg(sum_("montant").alias("total")) \
  .sort(col("date_vente"))

# Équivalent SQL
-- SELECT date_vente, SUM(montant) AS total
-- FROM ventes
-- WHERE region = 'EMEA'
-- GROUP BY date_vente
-- ORDER BY date_vente
```

---

## UDFs Snowpark ⭐

```python
# UDF Python simple
from snowflake.snowpark.functions import udf
from snowflake.snowpark.types import StringType

@udf(name="formater_telephone",
     is_permanent=True,
     stage_location="@mon_stage",
     replace=True)
def formater_telephone(tel: str) -> str:
    if tel:
        chiffres = ''.join(filter(str.isdigit, tel))
        return f"+33 {chiffres[1:2]} {chiffres[2:4]} {chiffres[4:6]} {chiffres[6:8]} {chiffres[8:10]}"
    return None

# Utiliser en SQL
# SELECT formater_telephone(telephone) FROM clients;

# Utiliser en Snowpark
df.with_column("tel_formate",
               formater_telephone(col("telephone"))).show()
```

---

## Procédures stockées Snowpark ⭐

```python
def pipeline_quotidien(session: Session, date_str: str) -> str:
    """Pipeline complet : extraction, transformation, chargement."""
    from snowflake.snowpark.functions import col, to_date, lit

    # 1. Extraire
    df_raw = session.table("staging.ventes_raw") \
        .filter(col("date_chargement") == lit(date_str))

    # 2. Transformer
    df_clean = df_raw \
        .filter(col("montant") > 0) \
        .filter(col("client_id").is_not_null()) \
        .with_column("date_vente", to_date(col("date_str"), "YYYY-MM-DD")) \
        .drop("date_str", "date_chargement")

    # 3. Charger
    nb_lignes = df_clean.count()
    df_clean.write.mode("append").save_as_table("prod.ventes")

    return f"Pipeline OK : {nb_lignes} lignes chargées pour {date_str}"

# Enregistrer
session.sproc.register(
    func           = pipeline_quotidien,
    name           = "pipeline_quotidien",
    packages       = ["snowflake-snowpark-python"],
    is_permanent   = True,
    stage_location = "@mon_stage",
    replace        = True
)
```

---

## Snowpark sur Warehouse Standard vs Optimisé

| | Warehouse Standard | Warehouse Snowpark-Optimized |
|---|---|---|
| **Mémoire par nœud** | Standard | **16x plus** |
| **Cas d'usage** | SQL, Snowpark léger | ML, traitement de gros DataFrames |
| **Coût** | Normal | Plus élevé |

```sql
-- Créer un warehouse Snowpark-optimisé
CREATE WAREHOUSE wh_ml
  WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'
  WAREHOUSE_SIZE = 'MEDIUM';
```
