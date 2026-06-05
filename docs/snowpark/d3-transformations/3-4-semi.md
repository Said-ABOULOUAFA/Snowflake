# 3.4 — Données semi-structurées dans les DataFrames

> **Domaine D3 Transformations — 35% du SPS-C01**

## Traverser les données semi-structurées ⭐

```python
from snowflake.snowpark.functions import col, get, get_path, parse_json, to_variant

# Accès à un champ JSON avec col() et notation crochets
df = session.table("events_json")

# Accéder aux champs imbriqués
df.select(
    col("data")["user"].alias("utilisateur"),
    col("data")["address"]["city"].alias("ville"),
    col("data")["tags"][0].alias("premier_tag")
)

# get() : accès par clé (équivalent à [])
from snowflake.snowpark.functions import get
df.select(get(col("data"), lit("user")).alias("user"))

# get_path() : chemin imbriqué
from snowflake.snowpark.functions import get_path
df.select(get_path(col("data"), lit("address.city")).alias("ville"))
```

## Cast dans les données semi-structurées ⭐

```python
# Convertir explicitement le type
df.select(
    col("data")["id"].cast("integer").alias("id"),
    col("data")["amount"].cast("float").alias("montant"),
    col("data")["date"].cast("date").alias("date_vente"),
    col("data")["active"].cast("boolean").alias("actif")
)

# Try cast (ne plante pas si échec)
from snowflake.snowpark.functions import try_cast
df.with_column("montant_num", try_cast(col("data")["amount"], "float"))
```

## Aplatir un tableau d'objets en lignes ⭐

```python
from snowflake.snowpark.functions import flatten

# JSON: {"commande_id": 1, "lignes": [{"produit": "A", "qte": 2}, {"produit": "B", "qte": 1}]}

df_flat = (df.join_table_function(
    flatten(col("data")["lignes"]).alias("seq", "key", "path", "idx", "value", "this")
))

df_result = df_flat.select(
    col("data")["commande_id"].cast("integer").alias("commande_id"),
    col("value")["produit"].cast("string").alias("produit"),
    col("value")["qte"].cast("integer").alias("quantite")
)
```

## Charger des données semi-structurées ⭐

```python
# Depuis un fichier JSON
df_json = session.read \
    .option("STRIP_OUTER_ARRAY", True) \
    .json("@mon_stage/data.json")

# Depuis un fichier Parquet (natif semi-structuré)
df_parquet = session.read.parquet("@mon_stage/data.parquet")

# Transformer semi → structuré
df_structure = df_json.select(
    col("$1")["id"].cast("int").alias("id"),
    col("$1")["nom"].cast("string").alias("nom"),
    col("$1")["montant"].cast("float").alias("montant")
)

# Transformer structuré → semi-structuré (OBJECT_CONSTRUCT)
from snowflake.snowpark.functions import object_construct, array_construct
df.select(
    object_construct(
        lit("id"), col("id"),
        lit("nom"), col("nom"),
        lit("montant"), col("montant")
    ).alias("json_ligne")
)
```
