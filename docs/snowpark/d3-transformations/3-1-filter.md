# 3.1 — Filtrer & Transformer les données

> **Domaine D3 Transformations — 35% du SPS-C01**

## Opérations scalaires et colonnes ⭐

```python
from snowflake.snowpark.functions import col, lit, upper, lower, trim, substring
from snowflake.snowpark.functions import to_date, to_timestamp, year, month, datediff

# Sélectionner et renommer
df = session.table("ventes")
df.select("id", "montant", "region")
df.select(col("montant").alias("chiffre_affaires"))

# Filtrer
df.filter(col("region") == "EMEA")
df.filter((col("montant") > 100) & (col("statut") == "VALIDE"))
df.filter(col("region").isin(["EMEA", "AMER"]))
df.filter(col("region").isNull())
df.filter(col("region").isNotNull())
df.filter(~col("annule"))

# Ajouter des colonnes calculées
df.with_column("montant_ttc",  col("montant") * lit(1.2))
df.with_column("nom_upper",    upper(col("nom")))
df.with_column("annee_vente",  year(col("date_vente")))

# Supprimer des colonnes
df.drop("colonne_inutile")

# Renommer
df.rename(col("montant"), "ca")

# Data type casting
df.with_column("montant", col("montant_str").cast("float"))
df.with_column("date",    col("date_str").cast("date"))
```

## Tri et limitation ⭐

```python
from snowflake.snowpark.functions import col

# Trier
df.sort(col("montant").desc())
df.sort(col("date_vente").asc(), col("montant").desc())
df.order_by(col("region"), col("montant").desc_nulls_last())

# Limiter
df.limit(100)
df.show(10)        # affiche les 10 premières
```

## Input/Output ⭐

```python
# Input : paramètres d'une procédure stockée
def ma_proc(session: Session, p_region: str, p_annee: int) -> str:
    df = session.table("ventes").filter(
        (col("region") == p_region) & (year(col("date_vente")) == p_annee)
    )
    return str(df.count())

# Output : valeur de retour
# Une procédure peut retourner STRING, INTEGER, FLOAT, VARIANT, TABLE...
def proc_table(session: Session) -> "DataFrame":
    return session.table("ventes")
```

## Extraction de données depuis un objet Row ⭐

```python
# collect() retourne une liste de Row objects
rows = df.filter(col("id") == 1).collect()

# Accéder aux données
row = rows[0]
row["nom"]         # par nom de colonne
row[0]             # par index
row.NOM            # attribut (si nom valide Python)
row.as_dict()      # convertir en dictionnaire
row.asDict()       # alias

# Itérer
for row in df.collect():
    print(row["id"], row["nom"])
```
