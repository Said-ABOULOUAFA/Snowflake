# 2.4 — UDFs & UDTFs en Snowpark

> **Domaine D2 API Python — 30% du SPS-C01**

## Créer et enregistrer des UDFs ⭐

```python
from snowflake.snowpark.functions import udf
from snowflake.snowpark.types import StringType, IntegerType, FloatType

# 1. Décorateur @udf (temporaire par défaut)
@udf
def doubler(x: int) -> int:
    return x * 2

# 2. UDF permanente avec stage
@udf(name="masquer_email",
     is_permanent=True,
     stage_location="@mon_stage",
     replace=True,
     packages=["snowflake-snowpark-python"])
def masquer_email(email: str) -> str:
    if email and '@' in email:
        parts = email.split('@')
        return parts[0][:2] + '***@' + parts[1]
    return '***'

# 3. Enregistrement via session.udf.register
def calculer_tva(montant: float, taux: float) -> float:
    return round(montant * taux / 100, 2)

session.udf.register(
    calculer_tva,
    name="calculer_tva",
    return_type=FloatType(),
    input_types=[FloatType(), FloatType()],
    is_permanent=True,
    stage_location="@mon_stage",
    replace=True
)

# 4. Depuis un fichier (localement ou sur stage)
session.udf.register_from_file(
    file_path="mes_fonctions.py",    # ou "@mon_stage/mes_fonctions.py"
    func_name="ma_fonction",
    name="ma_udf_from_file",
    return_type=StringType(),
    input_types=[StringType()]
)
```

## Secure UDFs ⭐

```python
# Créer une UDF sécurisée (DDL masqué)
@udf(name="hash_confidentiel",
     is_permanent=True,
     stage_location="@mon_stage",
     replace=True)
def hash_confidentiel(val: str) -> str:
    import hashlib
    return hashlib.sha256(val.encode()).hexdigest()

# Rendre sécurisée via SQL
session.sql("ALTER FUNCTION hash_confidentiel(STRING) SET SECURE").collect()

# Accorder l'accès
session.sql("GRANT USAGE ON FUNCTION hash_confidentiel(STRING) TO ROLE analyst").collect()
```

!!! danger "Question officielle SPS-C01"
    **Q : Que peut-on faire avec une Python UDF sans clause IMPORTS ?**
    **R : GRANT the USAGE privilege to a role** (réponse D officielle)
    On peut aussi : partager une vue qui appelle la UDF, mais PAS partager la UDF directement.

## Scalaire vs Vectorisée ⭐

```python
# UDF scalaire (1 ligne à la fois) — plus simple
@udf
def nettoyage_simple(texte: str) -> str:
    return texte.strip().lower() if texte else ""

# UDF vectorisée (pandas Series) — plus performante sur gros volumes
from snowflake.snowpark.functions import pandas_udf
import pandas as pd

@pandas_udf(name="nettoyage_vect",
            is_permanent=True,
            stage_location="@mon_stage",
            replace=True)
def nettoyage_vectorise(series: pd.Series) -> pd.Series:
    return series.str.strip().str.lower().fillna("")

# Vectorisée = traite des batches → 10-100x plus rapide
```

## UDTFs (User-Defined Table Functions) ⭐

```python
class SplitTagsUDTF:
    def process(self, tags_str: str):
        if tags_str:
            for tag in tags_str.split(','):
                yield (tag.strip(),)

session.udtf.register(
    SplitTagsUDTF,
    output_schema=StructType([StructField("tag", StringType())]),
    input_types=[StringType()],
    name="split_tags",
    is_permanent=True,
    stage_location="@mon_stage"
)

# Utiliser en SQL : SELECT t.tag FROM ma_table, TABLE(split_tags(tags)) t
# Utiliser en Snowpark :
from snowflake.snowpark.functions import table_function
split_tags_tf = table_function("split_tags")
df.join_table_function(split_tags_tf(col("tags")))
```

## Type hints vs Registration API ⭐

```python
# Type hints (Python natif — plus simple)
@udf
def parse_date(date_str: str) -> str:  # StringType déduit automatiquement
    from datetime import datetime
    return datetime.strptime(date_str, '%d/%m/%Y').strftime('%Y-%m-%d')

# Registration API (explicite — plus précis)
session.udf.register(
    lambda x: x.upper(),
    return_type=StringType(),
    input_types=[StringType()],  # type explicite
    name="to_upper"
)
```
