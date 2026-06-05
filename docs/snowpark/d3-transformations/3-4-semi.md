# 3.4 Semi-structuré dans Snowpark

> **Domain 3.0 — Data Transformations (35%)**

## Accès aux champs VARIANT

```python
from snowflake.snowpark.functions import col, parse_json, get, lit

df = session.table("events")           # colonne v: VARIANT

# Navigation + cast
df.select(
    col("v")["user"]["id"].cast("integer").alias("user_id"),
    col("v")["tags"][0].cast("string").alias("premier_tag"))
```

## FLATTEN (déplier des tableaux)

```python
from snowflake.snowpark.functions import flatten

flat = df.join_table_function(flatten(col("v")["tags"]))
flat.select(col("value").cast("string").alias("tag"))
```

| Opération | Snowpark |
|---|---|
| Parser JSON | `parse_json(col("txt"))` |
| Accès champ | `col("v")["chemin"]` |
| Cast | `.cast("integer")` / `.astype()` |
| Déplier | `flatten()` + `join_table_function` |
| Construire | `object_construct`, `array_agg` |

!!! danger "Piège exam"
    En Snowpark, on déplie un tableau VARIANT avec `flatten()` passé à `join_table_function()` (équivalent de `LATERAL FLATTEN` en SQL). L'accès par crochets `col("v")["champ"]` renvoie du VARIANT → **toujours caster** vers le type cible.

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/python/working-with-types`*
