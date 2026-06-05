# 5.1 UDFs / UDTFs / UDAFs

> **Domain 5.0 — Data Transformation (25%)**

## Comparatif ⭐

| Type | Retourne | Langages | Cas d'usage |
|---|---|---|---|
| **UDF scalaire** | 1 valeur / ligne | SQL, Python, Java, JS | Transformation simple |
| **UDTF** (tabulaire) | N lignes | Python, Java, JS | Dépliage, parsing |
| **UDAF** (agrégat) | 1 valeur / groupe | Python, Java, Scala | Agrégation custom |
| **UDF vectorisée** | 1 valeur / ligne | Python (pandas) | Performance sur gros volumes |

## UDF scalaire Python

```python
from snowflake.snowpark.functions import udf
from snowflake.snowpark.types import StringType

@udf(name="masquer_email", is_permanent=True, stage_location="@mon_stage", replace=True)
def masquer_email(email: str) -> str:
    if email and '@' in email:
        p = email.split('@')
        return p[0][:2] + '***@' + p[1]
    return '***'
```

## UDF vectorisée (pandas)

```python
import pandas as pd
from snowflake.snowpark.functions import pandas_udf

@pandas_udf(name="normaliser", is_permanent=True, stage_location="@mon_stage", replace=True)
def normaliser(series: pd.Series) -> pd.Series:
    return series.str.lower().str.strip()
```

## UDTF — table function

```sql
CREATE OR REPLACE FUNCTION split_csv(val STRING, sep STRING)
RETURNS TABLE (item STRING)
AS $$ SELECT value::STRING FROM TABLE(SPLIT_TO_TABLE(val, sep)) $$;

SELECT t.item FROM ma_table, TABLE(split_csv(ma_table.tags, ',')) t;
```

## UDAF — agrégation custom (Python)

```python
class MedianeHandler:
    def __init__(self): self._v = []
    @property
    def aggregate_state(self): return self._v
    def accumulate(self, val):
        if val is not None: self._v.append(val)
    def merge(self, other): self._v.extend(other)
    def finish(self):
        if not self._v: return None
        s = sorted(self._v); n = len(s); m = n // 2
        return s[m] if n % 2 else (s[m-1] + s[m]) // 2

session.udaf.register(MedianeHandler, name="mediane", replace=True,
    return_type=IntegerType(), input_types=[IntegerType()])
```

!!! danger "Piège exam"
    **UDF vectorisée (`pandas_udf`)** = bien plus performante sur de gros volumes car elle traite des *batches* (pandas Series) au lieu d'une ligne à la fois. UDTF = plusieurs lignes en sortie (méthode `process` + `yield`). UDAF = une valeur par groupe (`accumulate`/`merge`/`finish`).

📎 *Réf. : `docs.snowflake.com/en/developer-guide/udf/udf-overview`*
