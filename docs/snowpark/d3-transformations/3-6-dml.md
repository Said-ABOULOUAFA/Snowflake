# 3.6 — Opérations DML avec Snowpark DataFrames

> **Domaine D3 Transformations — 35% du SPS-C01**

## Insérer des données ⭐

```python
# INSERT via write (recommandé)
df_nouvelles_lignes.write.mode("append").save_as_table("ma_table")

# INSERT via SQL
session.sql("INSERT INTO ma_table SELECT * FROM staging_table").collect()
```

## Mettre à jour des données ⭐

```python
# UPDATE via la classe Table
from snowflake.snowpark.functions import col

table = session.table("clients")
table.update(
    {"statut": lit("INACTIF")},
    table["derniere_connexion"] < lit("2023-01-01").cast("date")
)

# UPDATE avec sous-requête
table_ventes = session.table("ventes")
table_clients = session.table("clients")
table_ventes.update(
    {"region_label": table_clients["region_nom"]},
    table_ventes["client_id"] == table_clients["id"],
    table_clients
)
```

## Supprimer des données ⭐

```python
table = session.table("staging_data")

# DELETE avec condition
table.delete(col("date_chargement") < lit("2023-01-01").cast("date"))

# DELETE avec sous-requête
table_staging = session.table("staging")
table_processed = session.table("processed")
table_staging.delete(
    table_staging["id"] == table_processed["staging_id"],
    table_processed
)
```

## MERGE (Upsert) ⭐

```python
from snowflake.snowpark.functions import col, when_matched, when_not_matched

table_target = session.table("clients_dim")
df_source    = session.table("clients_staging")

# MERGE : mise à jour si existe, insertion sinon
table_target.merge(
    df_source,
    table_target["id"] == df_source["id"],
    [
        when_matched().update({
            "nom":        df_source["nom"],
            "email":      df_source["email"],
            "updated_at": current_timestamp()
        }),
        when_not_matched().insert({
            "id":         df_source["id"],
            "nom":        df_source["nom"],
            "email":      df_source["email"],
            "created_at": current_timestamp()
        })
    ]
)

# Question officielle SPS-C01 Q3 :
# group_by("product_id").agg(sum("quantity")) = réponse C
```
