# 2.1 Client-side vs server-side

> **Domain 2.0 — Snowpark API for Python (30%)**

## Où s'exécute le code ?

| Élément | Exécution | Remarque |
|---|---|---|
| Construction du plan (`select`, `filter`…) | **Client** | Lazy, pas de calcul |
| Exécution SQL générée | **Server** (warehouse) | Pushdown |
| `udf` / `sproc` enregistrées | **Server** | Dans un sandbox Python |
| `collect`, `to_pandas` | Server → **transfert client** | Ramène les données |

```python
# Server-side : tout reste dans Snowflake
df = session.table("ventes").group_by("region").agg(sum_("montant"))
df.write.save_as_table("resume")        # aucune donnée côté client

# Client-side : rapatrie les résultats
pdf = df.to_pandas()                     # transfert vers la machine cliente
rows = df.collect()                      # liste de Row côté client
```

!!! danger "Piège exam"
    Maximiser le **server-side** = performance & scalabilité (le calcul reste près des données). N'utiliser `to_pandas()`/`collect()` que sur des **résultats réduits** : ramener des millions de lignes côté client annule l'intérêt de Snowpark et sature la mémoire locale.

!!! tip
    Packages Python tiers : disponibles via le **Snowflake Anaconda channel** pour l'exécution server-side (déclarer dans `packages=[...]`).

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/python/index`*
