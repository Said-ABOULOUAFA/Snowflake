# 4.2 Optimiser & troubleshoot Snowpark

> **Domain 4.0 — Performance Optimization (20%)**

## Bonnes pratiques ⭐

| Pratique | Bénéfice |
|---|---|
| Maximiser le **pushdown** (rester server-side) | Pas de transfert de données |
| Limiter `collect`/`to_pandas` aux **petits résultats** | Évite la saturation mémoire client |
| **Filtrer tôt** (`filter` avant `join`) | Réduit le volume traité |
| `cache_result()` sur DataFrame réutilisé | Évite le recalcul |
| **UDF vectorisée** (`pandas_udf`) | Traitement par batch |
| Snowpark-Optimized WH pour ML | Plus de mémoire |

## Diagnostiquer

```python
# Voir le SQL généré (sans exécuter)
print(df.queries)            # plan / requêtes SQL pushed down
df.explain()                 # plan d'exécution
```

```sql
-- Côté Snowflake : Query Profile + history
SELECT query_id, execution_time, bytes_spilled_to_remote_storage
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE query_text ILIKE '%SNOWPARK%' ORDER BY start_time DESC;
```

!!! danger "Piège exam"
    `df.explain()` / `df.queries` montrent le **SQL pushed down** — utile pour vérifier qu'une opération reste server-side. Le **spilling remote** dans le Query Profile = manque de mémoire → agrandir / Snowpark-Optimized. Ramener trop de données via `collect()` est l'anti-pattern n°1.

!!! tip
    Réutiliser une **Session** plutôt que d'en recréer une par appel. Regrouper les actions pour limiter les allers-retours.

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/python/index`*
