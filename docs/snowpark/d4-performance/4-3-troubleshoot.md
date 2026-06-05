# 4.3 — Troubleshooting Snowpark

> **Domaine D4 Performance — 20% du SPS-C01**

## Event Tables ⭐

```sql
-- Créer une Event Table
CREATE EVENT TABLE ma_db.public.event_table;
ALTER ACCOUNT SET EVENT_TABLE = ma_db.public.event_table;

-- Activer les logs sur un objet Snowpark
ALTER FUNCTION ma_udf(STRING) SET LOG_LEVEL = INFO;
ALTER PROCEDURE mon_pipeline(STRING) SET LOG_LEVEL = DEBUG;
ALTER PROCEDURE mon_pipeline(STRING) SET TRACE_LEVEL = ALWAYS;

-- Consulter les logs
SELECT *
FROM ma_db.public.event_table
WHERE record_type = 'LOG'
  AND scope:name::STRING = 'mon_pipeline'
  AND timestamp >= DATEADD('hour', -1, CURRENT_TIMESTAMP())
ORDER BY timestamp DESC;
```

## Snowpark Python Local Testing ⭐

```python
# Tester localement sans connexion Snowflake
from snowflake.snowpark.testing import mock_session

# Créer une session locale (mock)
with mock_session() as session:
    # Créer un DataFrame depuis des données locales
    df = session.create_dataframe(
        [(1, "Alice", 100.0), (2, "Bob", 200.0)],
        schema=["id", "nom", "montant"]
    )
    
    # Tester les transformations
    result = df.filter(col("montant") > 150).collect()
    assert len(result) == 1
    assert result[0]["NOM"] == "Bob"
```

## pytest pour Snowpark ⭐

```python
# tests/test_transformations.py
import pytest
from snowflake.snowpark import Session
from snowflake.snowpark.testing import mock_session

@pytest.fixture
def session():
    with mock_session() as s:
        yield s

def test_nettoyage_noms(session):
    df = session.create_dataframe(
        [("  Alice  ",), ("BOB",), (None,)],
        schema=["nom"]
    )
    
    from mon_module import nettoyer_noms
    result = nettoyer_noms(session, df).collect()
    
    assert result[0]["NOM"] == "alice"
    assert result[1]["NOM"] == "bob"
    assert result[2]["NOM"] is None or result[2]["NOM"] == ""

# Exécuter
# pytest tests/ -v
```

## Query History — Équivalence SQL ⭐

```python
# Voir le SQL généré par Snowpark
df_complex = (session.table("ventes")
    .filter(col("region") == "EMEA")
    .group_by("region")
    .agg(sum_("montant").alias("total"))
)

# Méthode 1 : explain()
df_complex.explain()   # affiche le plan logique + SQL

# Méthode 2 : queries SQL dans QUERY_HISTORY
result = df_complex.collect()

# Voir la requête SQL générée
last_query = session.sql("""
    SELECT query_text, execution_time/1000 AS sec
    FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
    WHERE query_tag = 'mon_job_snowpark'
    ORDER BY start_time DESC LIMIT 5
""").collect()

# Méthode 3 : tag de session pour identifier les requêtes
session.query_tag = "mon_pipeline_v2"
```

## Erreurs fréquentes Snowpark ⭐

| Erreur | Cause | Solution |
|---|---|---|
| `SnowparkJoinException` | Colonnes ambiguës dans un JOIN | Utiliser `df1["col"]` vs `df2["col"]` |
| `OOM / Memory Error` | Warehouse trop petit | Scale UP ou Snowpark-Optimized |
| `Module not found` | Package non disponible | Vérifier `SHOW PACKAGES` ou stage custom |
| `Session expired` | Session inactive trop longtemps | Reconnecter ou augmenter timeout |
| `Not supported in stored procedure` | Pandas operation | Remplacer par API Snowpark |

```python
# Colonnes ambiguës dans JOIN
df1 = session.table("ventes").select(col("id").alias("vente_id"), "montant")
df2 = session.table("clients").select(col("id").alias("client_id"), "nom")

# Erreur possible avec jointure sur colonnes de même nom
# Solution : renommer avant le join
df1.join(df2, df1["client_id"] == df2["client_id"])
```
