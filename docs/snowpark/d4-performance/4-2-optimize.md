# 4.2 — Optimiser les performances Snowpark

> **Domaine D4 Performance — 20% du SPS-C01**

## cache_result() ⭐

```python
# Sans cache — chaque utilisation recalcule la requête
df = session.table("large_table").filter(col("status") == "ACTIVE")
count1 = df.count()              # exécution 1
avg    = df.agg(avg("val")).collect()  # exécution 2 — recalcule !

# Avec cache_result() — calculé UNE fois, réutilisé
df_cache = df.cache_result()

count1 = df_cache.count()        # utilise le cache
avg    = df_cache.agg(avg("val")).collect()  # utilise le cache
df_cache.write.save_as_table("result")       # utilise le cache
```

## Table temporaire comme cache ⭐

```python
# Alternative à cache_result() — plus de contrôle
df.write.mode("overwrite").save_as_table("tmp_result", table_type="temporary")
df_tmp = session.table("tmp_result")

# Utiliser plusieurs fois
df_tmp.count()
df_tmp.agg(avg("montant")).collect()
```

## Vectorisation ⭐

### Scalaire vs Vectorisé

```python
import pandas as pd
from snowflake.snowpark.functions import udf, pandas_udf

# Scalaire : une ligne à la fois — simple mais lent sur gros volumes
@udf(name="to_upper_scalar")
def to_upper_scalar(text: str) -> str:
    return text.upper() if text else ""

# Vectorisé : batch de lignes (pandas Series) — 10-100x plus rapide
@pandas_udf(name="to_upper_vect",
            return_type=StringType(),
            is_permanent=True,
            stage_location="@mon_stage",
            replace=True)
def to_upper_vectorized(series: pd.Series) -> pd.Series:
    return series.str.upper().fillna("")

# Pour les agrégations : vectorized UDAF
@pandas_udf(name="mean_trim",
            input_types=[FloatType()],
            return_type=FloatType(),
            func_type="aggregate")
def trimmed_mean(series: pd.Series) -> float:
    import scipy.stats
    return float(scipy.stats.trim_mean(series.dropna(), 0.05))
```

### Snowpark DataFrames vs pandas on Snowflake

| | Snowpark DataFrame | pandas on Snowflake |
|---|---|---|
| **Exécution** | Dans Snowflake (SQL) | Dans Snowflake (Python) |
| **API** | Snowpark API | pandas-like API |
| **Conversion** | `.to_pandas()` | Native |
| **Performance** | Optimisé moteur SF | Bonne sur gros volumes |
| **Cas d'usage** | Transformations SQL | Code pandas existant |

```python
# pandas on Snowflake (modin backend)
import modin.pandas as pd
import snowflake.snowpark.modin.plugin

df_pd = session.table("ventes").to_pandas_on_snowflake()
result = df_pd.groupby("region")["montant"].sum()
```

## Synchronous vs Asynchronous ⭐

```python
# Synchrone (défaut) — bloque jusqu'à la fin
result = df.collect()

# Asynchrone — ne bloque pas
job = df.collect_nowait()     # ou collect(block=False)
# ... faire autre chose ...
result = job.result()          # attendre le résultat si besoin

# Paramètre block
job = session.sql("SELECT ...").collect(block=False)
print(f"Terminé : {job.is_done()}")
```
