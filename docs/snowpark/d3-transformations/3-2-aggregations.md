# 3.2 Agrégations & jointures

> **Domain 3.0 — Data Transformations (35%)**

## Agrégations

```python
from snowflake.snowpark.functions import sum as sum_, avg, count, max as max_

df.group_by("region").agg(
    sum_("montant").alias("total"),
    avg("montant").alias("moyenne"),
    count("*").alias("nb"))

# Pivot
df.pivot("trimestre", ["Q1","Q2","Q3","Q4"]).sum("montant")
```

## Jointures ⭐

```python
clients = session.table("clients")
df.join(clients, df["client_id"] == clients["id"], "inner")
df.join(clients, df["client_id"] == clients["id"], "left")

# Éviter l'ambiguïté de colonnes
df.join(clients, df["client_id"] == clients["id"], "left",
        lsuffix="_v", rsuffix="_c")
```

| Type | Mot-clé Snowpark |
|---|---|
| INNER | `"inner"` |
| LEFT / RIGHT | `"left"` / `"right"` |
| FULL OUTER | `"outer"` |
| CROSS | `"cross"` |
| Semi / Anti | `"semi"` / `"anti"` |

## Union

```python
df_2023.union(df_2024)        # par position
df_2023.union_by_name(df_2024) # par nom de colonne
```

!!! danger "Piège exam"
    `union` aligne par **position**, `union_by_name` par **nom**. En cas de colonnes homonymes après jointure, utiliser `lsuffix`/`rsuffix` ou aliaser les DataFrames pour lever l'ambiguïté. `group_by().agg()` requiert des fonctions du module `functions`.

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/python/working-with-dataframes`*
