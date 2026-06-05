# 1.1 -- Architecture Snowpark

> **Domaine D1 -- 15% du SPS-C01**

## Concept fondamental -- Lazy Evaluation STAR

Snowpark utilise l'evaluation paresseuse : les transformations sont des plans logiques,
pas des executions immediates.

```python
# Ces lignes ne calculent RIEN -- elles construisent un plan
df = session.table("ventes")
df_filtre = df.filter(col("region") == "EMEA")
df_resume = df_filtre.group_by("region").agg(sum_("montant").alias("total"))

# L'execution reelle se produit ici (action)
df_resume.show()           # collecte et affiche
df_resume.collect()        # collecte en Python
df_resume.count()          # compte les lignes
df_resume.write.mode("overwrite").save_as_table("result")  # ecrit en table
```

!!! danger "Question officielle SPS-C01"
    La LAZY EVALUATION signifie que les transformations DataFrames ne sont executees
    que lors d'une ACTION (show, collect, count, write, save_as_table...).

## Objets cles Snowpark STAR

| Objet | Role |
|---|---|
| **Session** | Point d'entree -- connexion a Snowflake |
| **DataFrame** | Representation d'une table/requete (lazy) |
| **Column** | Reference a une colonne du DataFrame |
| **UDF** | Fonction scalaire definie par l'utilisateur |
| **UDTF** | Fonction tabulaire (retourne plusieurs lignes) |
| **Stored Procedure** | Procedure stockee Snowpark |
| **File Operations** | Lecture/ecriture de fichiers sur stages |

## Bibliotheques disponibles STAR

### Anaconda Repository (packages Python dans Snowflake)

```python
# Specifier les packages Anaconda dans UDFs/procedures
@udf(packages=["pandas", "numpy", "scikit-learn"])
def ma_fonction(x: float) -> float:
    import numpy as np
    return float(np.sqrt(x))
```

### Packages tiers non-Anaconda

```python
# Importer un package custom depuis un stage
session.add_import("@mon_stage/mon_package.zip")

@udf(imports=["@mon_stage/mon_package.zip"])
def ma_udf_custom(x: str) -> str:
    from mon_package import ma_fonction
    return ma_fonction(x)
```

## Client-Side vs Server-Side STAR

| | Client-Side | Server-Side |
|---|---|---|
| **Execution** | Machine locale (Python) | Dans Snowflake (warehouse) |
| **Donnees** | Rapatriees localement | Restent dans Snowflake |
| **Performance** | Limitee par le reseau | Moteur Snowflake |
| **Cas d'usage** | Visualisation, debug | Transformations, ML |
| **Methodes** | collect(), to_pandas() | save_as_table(), write |

```python
# Client-side : donnees rapatriees en Python
df_local = df.collect()           # liste de Row objects
pdf = df.to_pandas()              # pandas DataFrame

# Server-side : execution dans Snowflake
df.write.mode("overwrite").save_as_table("ma_table")  # reste dans SF
df.create_or_replace_view("ma_vue")                    # reste dans SF
```
