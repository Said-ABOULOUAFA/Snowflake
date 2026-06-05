# 3.2 — Nettoyer & Enrichir les données

> **Domaine D3 Transformations — 35% du SPS-C01**

## Jointures ⭐

```python
from snowflake.snowpark.functions import col

df_ventes  = session.table("ventes")
df_clients = session.table("clients")
df_regions = session.table("regions")

# INNER JOIN (défaut)
df_joined = df_ventes.join(df_clients, df_ventes["client_id"] == df_clients["id"])
df_joined = df_ventes.join(df_clients, "client_id")   # si même nom de colonne

# LEFT JOIN
df_left = df_ventes.join(df_clients, df_ventes["client_id"] == df_clients["id"], "left")

# Types de jointure : inner, left, right, full, cross, semi, anti
df_anti = df_ventes.join(df_clients, "client_id", "anti")  # lignes sans correspondance

# Jointures multiples
df_final = (df_ventes
    .join(df_clients, "client_id", "left")
    .join(df_regions, "region_id", "left")
)
```

## Gérer les valeurs manquantes ⭐

```python
from snowflake.snowpark.functions import col, lit, coalesce, iff, when

# Supprimer les lignes avec NULLs
df.dropna()                           # toutes colonnes
df.dropna(subset=["id", "email"])     # colonnes spécifiques
df.dropna(how="all")                  # seulement si TOUTES sont NULL

# Remplacer les NULLs
df.fillna({"montant": 0, "region": "INCONNU"})
df.fillna(0)                          # toutes les colonnes numériques

# COALESCE : première valeur non-NULL
df.with_column("nom_final", coalesce(col("nom"), col("prenom"), lit("Inconnu")))

# IFF : si-alors-sinon
df.with_column("categorie", iff(col("montant") > 1000, lit("Premium"), lit("Standard")))

# WHEN : CASE WHEN
df.with_column("niveau",
    when(col("montant") > 10000, lit("Platinum"))
    .when(col("montant") > 1000,  lit("Gold"))
    .when(col("montant") > 100,   lit("Silver"))
    .otherwise(lit("Bronze"))
)
```

## Échantillonnage ⭐

```python
# Échantillon aléatoire (fraction)
df_sample = df.sample(frac=0.1)    # 10% des lignes

# Échantillon par nombre de lignes
df_sample = df.sample(n=1000)

# Échantillon reproductible (avec seed)
df_sample = df.sample(frac=0.05, seed=42)
```
