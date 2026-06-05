# 2.3 UDFs, UDTFs & vectorisées

> **Domain 2.0 — Snowpark API for Python (30%)**

## UDF scalaire

```python
from snowflake.snowpark.functions import udf
from snowflake.snowpark.types import IntegerType, StringType

@udf(name="categorie", is_permanent=True, stage_location="@stg", replace=True,
     packages=["snowflake-snowpark-python"])
def categorie(montant: float) -> str:
    return "Premium" if montant > 1000 else "Standard"
```

## UDF vectorisée (pandas) ⭐

```python
import pandas as pd
from snowflake.snowpark.functions import pandas_udf
from snowflake.snowpark.types import PandasSeriesType, FloatType

@pandas_udf(name="tva", is_permanent=True, stage_location="@stg", replace=True)
def tva(s: pd.Series) -> pd.Series:
    return s * 1.2
```

## UDTF

```python
from snowflake.snowpark.functions import udtf
from snowflake.snowpark.types import StructType, StructField, StringType

class Split:
    def process(self, txt: str):
        for part in txt.split(","):
            yield (part.strip(),)

session.udtf.register(Split, output_schema=StructType([StructField("item", StringType())]),
    name="split_udtf", replace=True, is_permanent=True, stage_location="@stg")
```

| Type | Décorateur / register | Sortie |
|---|---|---|
| Scalaire | `@udf` | 1 valeur / ligne |
| Vectorisée | `@pandas_udf` | 1 valeur / ligne (batch pandas) |
| UDTF | `session.udtf.register` (classe `process`) | N lignes |
| UDAF | `session.udaf.register` (`accumulate`/`merge`/`finish`) | 1 / groupe |

!!! danger "Piège exam"
    La **vectorisée** (`pandas_udf`) traite des Series → bien plus rapide sur gros volumes. Une UDF `is_permanent=True` requiert un `stage_location`. Déclarer les `packages` nécessaires (résolus via Anaconda). La UDTF implémente `process()` et **yield** des tuples.

📎 *Réf. : `docs.snowflake.com/en/developer-guide/snowpark/python/creating-udfs`*
