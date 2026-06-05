# 3.5 — Persister les résultats des DataFrames

> **Domaine D3 Transformations — 35% du SPS-C01**

## Créer des vues ⭐

```python
# Vue standard
df.create_or_replace_view("ma_base.mon_schema.ma_vue")
df.create_or_replace_temp_view("ma_vue_temp")  # session uniquement

# Vue depuis SQL
session.sql("""
    CREATE OR REPLACE VIEW ma_vue AS
    SELECT region, SUM(montant) AS total FROM ventes GROUP BY region
""")
```

## Sauvegarder en table Snowflake ⭐

```python
# Mode overwrite (recrée la table)
df.write.mode("overwrite").save_as_table("ma_table")

# Mode append (ajoute les données)
df.write.mode("append").save_as_table("ma_table")

# Mode error_if_exists (défaut)
df.write.mode("errorifexists").save_as_table("ma_table")

# Mode ignore
df.write.mode("ignore").save_as_table("ma_table")

# Options avancées
df.write \
  .mode("overwrite") \
  .option("columnOrder", "name") \
  .save_as_table("ma_table", table_type="transient")  # ou "temporary"

# Créer une table directement
df.create_or_replace_table("ma_nouvelle_table")
```

## Sauvegarder en fichiers sur stage ⭐

```python
from snowflake.snowpark.functions import col

# Export en Parquet
df.write.mode("overwrite") \
  .copy_into_location(
      "@mon_stage/exports/",
      file_format_type="parquet",
      header=True
  )

# Export en CSV avec options
df.write.copy_into_location(
    "@mon_stage/exports/ventes_",
    file_format_type="csv",
    format_type_options={"COMPRESSION": "GZIP", "HEADER": True},
    overwrite=True,
    single=False,   # False = plusieurs fichiers (recommandé)
    max_file_size=104857600  # 100 MB
)
```

## cache_result() ⭐

Matérialise le DataFrame dans une table temporaire pour réutilisation.

```python
# Sans cache : chaque utilisation de df recalcule la requête
df_lourd = session.table("large_table").filter(col("status") == "ACTIVE")

# Avec cache : calculé UNE fois, réutilisé plusieurs fois
df_cache = df_lourd.cache_result()

count1 = df_cache.count()           # utilise le cache
avg    = df_cache.agg(avg("montant")).collect()  # utilise le cache
df_cache.write.save_as_table("result")           # utilise le cache

# Le cache est une table temporaire → nettoyé à la fin de session
```

!!! tip "Quand utiliser cache_result() ?"
    Quand tu utilises le même DataFrame lourd **plusieurs fois** dans la même session.
    C'est l'équivalent Snowpark du Result Cache (mais pour un DataFrame spécifique).
