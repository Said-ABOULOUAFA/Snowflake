# 1.2 Sessions & DataFrames

> **Domain 1.0 — Snowpark Concepts (15%)**

## Créer une Session

```python
from snowflake.snowpark import Session

session = Session.builder.configs({
    "account": "mon_compte", "user": "u", "password": "***",
    "warehouse": "wh_dev", "database": "ma_db", "schema": "public"
}).create()
```

!!! tip "Dans Snowflake (sproc/Notebook)"
    Quand le code s'exécute déjà *dans* Snowflake, on récupère la session active : `session = get_active_session()` (pas d'identifiants à fournir).

## Construire des DataFrames

```python
df1 = session.table("ventes")                      # depuis une table
df2 = session.sql("SELECT * FROM ventes WHERE x>0") # depuis du SQL
df3 = session.create_dataframe([[1,"a"],[2,"b"]], schema=["id","val"])
df4 = session.read.schema(mon_schema).csv("@stage/data/")
```

## Opérations de base

```python
df.select("region", "montant")
df.filter(col("montant") > 100)
df.with_column("net", col("montant") * 0.8)
df.drop("col_inutile")
df.rename(col("montant"), "ca")
df.distinct()
df.limit(10)
```

!!! danger "Piège exam"
    `session.table()` et `session.sql()` renvoient un **DataFrame lazy**. `create_dataframe()` sert à matérialiser des données locales côté serveur. Une Session encapsule la connexion + le contexte (warehouse/db/schema).

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/python/working-with-dataframes`*
