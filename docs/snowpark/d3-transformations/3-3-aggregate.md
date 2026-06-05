# 3.3 — Agrégations & Opérations sur ensembles

> **Domaine D3 Transformations — 35% du SPS-C01**

## Fonctions d'agrégation ⭐

```python
from snowflake.snowpark.functions import (
    col, sum as sum_, avg, count, min as min_, max as max_,
    stddev, median, listagg, array_agg, count_distinct
)

# Agrégation simple
df.agg(sum_("montant").alias("total"))

# Groupement
df.group_by("region").agg(
    sum_("montant").alias("total"),
    avg("montant").alias("moyenne"),
    count("*").alias("nb"),
    min_("montant").alias("minimum"),
    max_("montant").alias("maximum"),
    count_distinct("client_id").alias("nb_clients_uniques")
)

# GROUP BY multiple colonnes
df.group_by("region", "annee").agg(sum_("montant").alias("total"))

# GROUP BY avec expression
from snowflake.snowpark.functions import year
df.group_by(year(col("date_vente")).alias("annee"), "region") \
  .agg(sum_("montant").alias("total"))
```

## Window Functions ⭐

```python
from snowflake.snowpark import Window
from snowflake.snowpark.functions import (
    col, sum as sum_, rank, dense_rank, row_number,
    lag, lead, first_value, last_value,
    ratio_to_report
)

# Définir une fenêtre
window_region = Window.partition_by("region").order_by(col("date_vente"))
window_all    = Window.partition_by()  # toute la table

# Fonctions de fenêtre
df.with_columns(
    ["cumul", "rang", "precedent", "pct"],
    [
        sum_(col("montant")).over(window_region.rows_between(
            Window.UNBOUNDED_PRECEDING, Window.CURRENT_ROW
        )),
        rank().over(Window.partition_by("region").order_by(col("montant").desc())),
        lag(col("montant"), 1).over(window_region),
        ratio_to_report(col("montant")).over(Window.partition_by("region"))
    ]
)
```

## Table Functions ⭐

```python
from snowflake.snowpark.functions import table_function, flatten

# FLATTEN : dépliage de tableaux JSON
split_func = table_function("split_to_table")
df_tags = session.table("produits") \
    .join_table_function(split_func(col("tags"), lit(",")).alias("idx", "tag_value"))

# FLATTEN natif Snowpark
df_flat = df.join_table_function(
    flatten(col("data")["items"]).alias("seq", "key", "path", "index", "value", "this")
)
```

## Opérations sur ensembles ⭐

```python
# UNION
df_total = df_2023.union(df_2024)
df_total = df_2023.union_all(df_2024)  # avec doublons

# INTERSECT
df_commun = df_clients_2023.intersect(df_clients_2024)

# EXCEPT / MINUS
df_nouveaux = df_clients_2024.except_(df_clients_2023)
```
