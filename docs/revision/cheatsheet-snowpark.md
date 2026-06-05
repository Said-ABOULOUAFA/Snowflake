# 🐍 Cheat Sheet — SPS-C01 (Snowpark)

> 55 questions · 85 min · score 750/1000 · 225 $ · EN uniquement · **prérequis : SnowPro Core**

## Poids des domaines
| Domaine | % |
|---|---|
| 1.0 Snowpark Concepts | 15 |
| 2.0 Snowpark API for Python | 30 |
| 3.0 Data Transformations | 35 |
| 4.0 Performance Optimization | 20 |

## Concepts
- Code Python/Java/Scala **compilé en SQL** et exécuté dans le warehouse (**pushdown**).
- **Lazy** : transformations différées, exécutées à l'**action** (`show`, `collect`, `count`, `save_as_table`, `to_pandas`).
- Session = connexion + contexte ; `get_active_session()` dans Snowflake.

## API Python
- `from snowflake.snowpark.functions import col, lit, sum as sum_, when, ...` (alias pour éviter collisions).
- DataFrame : `select`, `filter`, `with_column`, `join`, `group_by().agg()`, `sort`, `distinct`, `union`/`union_by_name`.
- Window : `Window.partition_by().order_by()` + `row_number()`, `rank()`.

## UDFs
| Type | API | Sortie |
|---|---|---|
| Scalaire | `@udf` | 1 / ligne |
| Vectorisée | `@pandas_udf` | 1 / ligne (batch) |
| UDTF | `session.udtf.register` | N lignes |
| UDAF | `session.udaf.register` | 1 / groupe |
- `is_permanent=True` → `stage_location` requis ; déclarer `packages`.

## Transformations
- Conditions : `&` `|` `~` + **parenthèses** ; `col("x") == "v"`.
- Jointures : inner/left/right/outer/cross/semi/anti ; `lsuffix`/`rsuffix`.
- Semi-structuré : `col("v")["champ"].cast(...)`, `flatten()` + `join_table_function`.

## Performance ⭐
- Maximiser server-side ; `collect`/`to_pandas` sur **petits** résultats.
- `cache_result()`, filtrer tôt, `pandas_udf`.
- **Snowpark-Optimized Warehouse** (~16× mémoire) pour ML / OOM.
- Diagnostic : `df.explain()`, `df.queries`, Query Profile (spilling).
