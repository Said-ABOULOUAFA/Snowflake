# 5.7 — Snowpark pour les transformations DEA-C02

> **Domaine D5 Data Transformation — 25% du DEA-C02**

## Architecture Snowpark ⭐

```
Code Python (local ou Notebook)
        │ Lazy Evaluation
        ▼
Plan logique (DataFrame API)
        │ Compile en SQL
        ▼
Moteur Snowflake (dans le warehouse)
        │ Résultat
        ▼
Table / Stage / Retour Python
```

## Requêtes et filtres ⭐

```python
from snowflake.snowpark.functions import col, year, month, upper, trim

df = session.table("ventes")

# Filtres
df.filter(col("region") == "EMEA")
df.filter((col("montant") > 100) & (year(col("date_vente")) == 2024))
df.filter(col("client_id").isin([1, 2, 3]))
df.where(col("statut").isNotNull())

# Sélection et transformation
df.select("id", "region", col("montant").alias("ca"))
df.with_column("nom_maj", upper(trim(col("nom"))))
df.with_columns(
    ["annee", "mois"],
    [year(col("date_vente")), month(col("date_vente"))]
)
```

## Agrégations ⭐

```python
from snowflake.snowpark.functions import sum as sum_, avg, count, max as max_

# GROUP BY + AGG
df.group_by("region").agg(
    sum_("montant").alias("total"),
    avg("montant").alias("moyenne"),
    count("*").alias("nb"),
    max_("montant").alias("max_vente")
)

# Plusieurs colonnes
df.group_by("region", "annee").agg(sum_("montant").alias("total"))
```

## Manipuler les DataFrames ⭐

```python
# Jointures
df_v.join(df_c, df_v["client_id"] == df_c["id"], "left")

# Union
df_2023.union_all(df_2024)

# Window functions
from snowflake.snowpark import Window
w = Window.partition_by("region").order_by(col("date_vente"))
df.with_column("cumul", sum_(col("montant")).over(w))

# Écriture
df.write.mode("overwrite").save_as_table("ma_table")
df.write.mode("append").save_as_table("ma_table")

# Export stage
df.write.copy_into_location("@mon_stage/exports/",
    file_format_type="parquet", header=True, overwrite=True)
```
