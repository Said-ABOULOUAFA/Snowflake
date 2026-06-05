# 3.3 Actions & persistance

> **Domain 3.0 — Data Transformations (35%)**

## Actions (déclenchent l'exécution) ⭐

```python
df.show(10)                 # affiche (par défaut 10 lignes)
df.collect()                # List[Row] côté client
df.to_pandas()              # DataFrame pandas côté client
df.count()                  # nombre de lignes
df.first()                  # première Row
```

## Persistance

```python
# Écrire dans une table
df.write.mode("overwrite").save_as_table("resultat")
df.write.mode("append").save_as_table("resultat")

# Vue
df.create_or_replace_view("ma_vue")
df.create_or_replace_temp_view("ma_vue_temp")

# Décharger vers un stage
df.write.copy_into_location("@stage/export/", file_format_type="parquet")

# Matérialiser un intermédiaire réutilisé
df_cache = df.cache_result()
```

| Mode `save_as_table` | Effet |
|---|---|
| `overwrite` | Remplace la table |
| `append` | Ajoute les lignes |
| `errorifexists` | Échoue si existe |
| `ignore` | Ne fait rien si existe |

!!! danger "Piège exam"
    Tant qu'aucune **action** n'est appelée, rien ne s'exécute (lazy). `save_as_table` matérialise le résultat dans Snowflake (server-side, pas de transfert client). `cache_result()` évite de recalculer un DataFrame réutilisé plusieurs fois.

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/python/working-with-dataframes`*
