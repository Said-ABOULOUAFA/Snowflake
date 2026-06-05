# 3.1 Filtrer & manipuler

> **Domain 3.0 — Data Transformations (35%)**

```python
from snowflake.snowpark.functions import col, lit, upper, when

df = session.table("ventes")

# Filtrer
df.filter((col("montant") > 100) & (col("region") == "EMEA"))
df.filter(col("statut").isin("PAYE", "EXPEDIE"))
df.filter(col("email").is_not_null())

# Sélectionner / dériver
df.select(col("id"), upper(col("region")).alias("region"))
df.with_column("net", col("montant") * lit(0.8))
df.with_columns(["a", "b"], [col("x")+1, col("y")*2])

# Renommer / supprimer / trier
df.rename(col("montant"), "ca").drop("col_inutile").sort(col("ca").desc())
```

| Besoin | Méthode |
|---|---|
| Filtrer | `filter` / `where` |
| Ajouter colonne | `with_column(s)` |
| Sélectionner | `select` |
| Trier | `sort` / `order_by` |
| Dédupliquer | `distinct`, `drop_duplicates` |
| Limiter | `limit` |

!!! danger "Piège exam"
    Les conditions composées utilisent `&` (ET), `|` (OU), `~` (NON) avec **parenthèses obligatoires** autour de chaque comparaison. `col("x") == "a"` (pas `=`). `isin(...)` remplace plusieurs OR.

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/python/working-with-dataframes`*
