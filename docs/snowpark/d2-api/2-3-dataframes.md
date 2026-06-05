# 2.3 — Créer des DataFrames Snowpark

> **Domaine D2 API Python — 30% du SPS-C01**

## Méthodes de création ⭐

```python
from snowflake.snowpark import Session
from snowflake.snowpark.types import StructType, StructField, IntegerType, StringType

# 1. Depuis une table/vue Snowflake
df_table = session.table("ma_table")
df_view  = session.table("ma_vue")

# 2. Depuis une requête SQL
df_sql = session.sql("SELECT * FROM ventes WHERE region = 'EMEA'")

# 3. Depuis une liste Python
df_list = session.create_dataframe(
    [(1, "Alice", 100), (2, "Bob", 200)],
    schema=["id", "nom", "montant"]
)

# 4. Depuis un dictionnaire
df_dict = session.create_dataframe(
    [{"id": 1, "nom": "Alice"}, {"id": 2, "nom": "Bob"}]
)

# 5. Depuis un pandas DataFrame
import pandas as pd
pdf = pd.DataFrame({"id": [1, 2], "valeur": [10.5, 20.3]})
df_pandas = session.create_dataframe(pdf)

# 6. Depuis des fichiers (JSON, CSV, Parquet, XML)
df_json    = session.read.json("@mon_stage/data.json")
df_csv     = session.read.csv("@mon_stage/data.csv")
df_parquet = session.read.parquet("@mon_stage/data.parquet")
df_xml     = session.read.xml("@mon_stage/data.xml")
```

## Schémas et types de données ⭐

```python
from snowflake.snowpark.types import (
    StructType, StructField,
    IntegerType, StringType, FloatType, DateType,
    TimestampType, BooleanType, VariantType
)

schema = StructType([
    StructField("id",         IntegerType(),   nullable=False),
    StructField("nom",        StringType(),    nullable=True),
    StructField("montant",    FloatType(),     nullable=True),
    StructField("date_vente", DateType(),      nullable=True),
    StructField("metadata",   VariantType(),   nullable=True),
])

df = session.create_dataframe([], schema=schema)

# Type hints vs Registration API
# Type hints : Python natif
def ma_udf(x: int) -> str: ...  # IntegerType → StringType auto-déduit

# Registration API : explicite
session.udf.register(func, return_type=StringType(), input_types=[IntegerType()])
```

## DataFrameReader ⭐

```python
# Configurer le reader
df = (session.read
    .option("SKIP_HEADER", 1)
    .option("FIELD_DELIMITER", ";")
    .option("NULL_IF", "NULL")
    .schema(schema)
    .csv("@mon_stage/data.csv")
)

# Parquet avec options
df = session.read.option("compression", "snappy").parquet("@stage/data.parquet")
```
