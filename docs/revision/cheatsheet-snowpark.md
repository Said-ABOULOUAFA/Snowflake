# Cheat Sheet SPS-C01 — Snowpark

## Prérequis
- COF-C03 recommandé
- 1+ an Snowpark en entreprise

## Poids des domaines

| Domaine | % |
|---|---|
| D1 Concepts | 15% |
| D2 API Python | 30% |
| D3 Transformations | **35%** |
| D4 Performance | 20% |

## Fondamentaux

```python
# Lazy evaluation = PAS d'exécution tant qu'il n'y a pas d'action
# Actions : .show(), .collect(), .count(), .write, .save_as_table()

# Premier paramètre stored procedure = TOUJOURS Session
def ma_proc(session: Session, param: str) -> str: ...

# Snowpark-Optimized = ML TRAINING uniquement
CREATE WAREHOUSE wh_ml WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'

# cache_result() = matérialiser un DataFrame pour réutilisation
df_cache = df.cache_result()

# UDF vectorisée = pandas Series → 10-100x plus rapide que scalaire
@pandas_udf
def ma_udf(s: pd.Series) -> pd.Series: ...

# MFA Caching = réduire les prompts MFA
ALTER ACCOUNT SET ALLOW_CLIENT_MFA_CACHING = TRUE;
```

## API Snowpark essentiels

```python
# Créer DataFrame
session.table("ma_table")
session.sql("SELECT ...")
session.create_dataframe([(1,"Alice")])

# Transformations
df.filter(col("x") > 10)
df.select("col1", col("col2").alias("new_name"))
df.with_column("new", col("a") + col("b"))
df.group_by("region").agg(sum_("montant").alias("total"))

# Window
from snowflake.snowpark import Window
w = Window.partition_by("region").order_by(col("date"))
df.with_column("cumul", sum_(col("montant")).over(w))

# Persist
df.write.mode("overwrite").save_as_table("ma_table")
df.write.copy_into_location("@stage/path/")
df.create_or_replace_view("ma_vue")

# DML
session.table("t").update({"col": lit("val")}, condition)
session.table("t").delete(condition)
session.table("t").merge(source, condition, [when_matched(), when_not_matched()])
```

## Questions pièges SPS-C01

```
❗ group_by("id").agg(sum("qty")) = syntaxe correcte
❗ Premier param proc = Session (pas DataFrame, pas String)
❗ UDF sans IMPORTS → GRANT USAGE possible
❗ UDF sans IMPORTS → PAS de session object accessible
❗ MFA Caching → ALLOW_CLIENT_MFA_CACHING = TRUE
❗ Snowpark-Optimized → ML Training (pas inference)
❗ SnowflakeFile.open() → lire fichiers dans UDFs
❗ AsyncJob → exécution non-bloquante
❗ explain() → voir le SQL généré
```
