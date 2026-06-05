# 2.2 Fonctions de l'API Snowpark

> **Domain 2.0 — Snowpark API for Python (30%)**

## Module `functions` ⭐

```python
from snowflake.snowpark.functions import (
    col, lit, sum as sum_, avg, count, max as max_, min as min_,
    when, coalesce, to_date, datediff, upper, lower, concat,
    row_number, rank, dense_rank, parse_json, array_agg
)
```

| Catégorie | Exemples |
|---|---|
| Agrégation | `sum_`, `avg`, `count`, `max_`, `min_`, `array_agg` |
| Conditionnel | `when().otherwise()`, `coalesce`, `iff` |
| Chaînes | `upper`, `lower`, `concat`, `substring`, `trim` |
| Dates | `to_date`, `datediff`, `dateadd`, `current_date` |
| Fenêtrage | `row_number`, `rank`, `lag`, `lead` over `Window` |
| Semi-structuré | `parse_json`, `get`, `flatten` |

## Window functions

```python
from snowflake.snowpark import Window
from snowflake.snowpark.functions import row_number, col

w = Window.partition_by("region").order_by(col("montant").desc())
df = df.with_column("rang", row_number().over(w))
```

!!! danger "Piège exam"
    Attention aux collisions de noms : `sum`, `max`, `min`, `filter` existent en Python natif → on importe avec alias (`sum as sum_`). `col("x")` référence une colonne ; `lit(5)` crée une constante littérale. `when(...).otherwise(...)` = `CASE WHEN`.

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/reference/python/latest/functions`*
