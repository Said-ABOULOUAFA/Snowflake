# 1.1 Architecture Snowpark

> **Domain 1.0 — Snowpark Concepts (15%)**

![Architecture Snowpark](../../assets/snowpark-pushdown.svg)

Snowpark est une bibliothèque (Python, Java, Scala) qui permet d'écrire des transformations dans un langage de programmation, **exécutées dans le moteur Snowflake** via *pushdown* SQL.

## Deux plans d'exécution ⭐

| Plan | S'exécute | Exemples |
|---|---|---|
| **Client-side** | Sur la machine / le driver | Construction du plan logique, `print` |
| **Server-side** | Dans le warehouse Snowflake | DataFrame ops, UDF, sproc, `collect` |

## Évaluation paresseuse (lazy)

```python
df = session.table("ventes").filter(col("montant") > 100)   # rien n'est exécuté
df.show()                                                    # ACTION → exécution SQL
```

| Transformations (lazy) | Actions (déclenchent) |
|---|---|
| `select`, `filter`, `join`, `group_by`, `with_column` | `show`, `collect`, `count`, `save_as_table`, `to_pandas` |

!!! danger "Piège exam"
    Snowpark **ne transfère pas** les données vers le client : les transformations sont compilées en SQL et *pushed down*. Seules les **actions** ramènent un résultat (et seulement ce qui est demandé). C'est la différence majeure avec pandas classique (qui charge tout en mémoire locale).

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/python/index`*
