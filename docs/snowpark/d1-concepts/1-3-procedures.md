# 1.3 Stored procedures & logique conditionnelle

> **Domain 1.0 — Snowpark Concepts (15%)**

## Stored procedure Snowpark Python

```python
def transformer(session, seuil: float) -> str:
    df = session.table("ventes").filter(col("montant") > seuil)
    df.write.mode("overwrite").save_as_table("ventes_filtrees")
    return f"{df.count()} lignes"

session.sproc.register(func=transformer, name="transformer",
    packages=["snowflake-snowpark-python"], is_permanent=True,
    stage_location="@mon_stage", replace=True)
# CALL transformer(100);
```

## Logique conditionnelle dans un DataFrame

```python
from snowflake.snowpark.functions import when, col, lit

df = df.with_column("segment",
    when(col("ca") > 10000, lit("Grand compte"))
    .when(col("ca") > 1000, lit("PME"))
    .otherwise(lit("TPE")))
```

## Logique Python (flux de contrôle)

```python
def pipeline(session) -> str:
    tables = ["ventes_2023", "ventes_2024"]
    total = 0
    for t in tables:                       # boucle Python côté serveur
        c = session.table(t).count()
        if c > 0:
            total += c
    return f"Total : {total}"
```

!!! danger "Piège exam"
    Dans une stored procedure Snowpark, le **flux de contrôle Python** (if/for/try) s'exécute côté serveur, mais les **opérations sur DataFrame** sont compilées en SQL. Le premier argument de la fonction est **toujours** `session`. `when().otherwise()` est l'équivalent de `CASE WHEN` côté DataFrame.

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/python/creating-sprocs`*
