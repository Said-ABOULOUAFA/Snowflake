# 2.5 — Procédures stockées Snowpark

> **Domaine D2 API Python — 30% du SPS-C01**

## Créer des procédures stockées ⭐

```python
from snowflake.snowpark import Session

# La PREMIERE PARAMETRE doit TOUJOURS être une Session ⭐
def pipeline_nettoyage(session: Session, table_src: str, table_dst: str) -> str:
    from snowflake.snowpark.functions import col, upper, trim

    df = session.table(table_src)
    df_clean = (df
        .filter(col("id").is_not_null())
        .with_column("nom", upper(trim(col("nom"))))
        .distinct()
    )
    nb = df_clean.count()
    df_clean.write.mode("overwrite").save_as_table(table_dst)
    return f"OK : {nb} lignes"

# Enregistrer
session.sproc.register(
    pipeline_nettoyage,
    name="pipeline_nettoyage",
    packages=["snowflake-snowpark-python"],
    is_permanent=True,
    stage_location="@mon_stage",
    replace=True,
    execute_as="caller"   # ou "owner"
)
```

!!! danger "Question officielle SPS-C01"
    **Q : Que doit considérer un Snowpark Specialist pour définir une procédure Python ?**
    **R : Le PREMIER paramètre doit être un objet Session** (réponse B officielle)

## Caller vs Owner Rights ⭐

| | EXECUTE AS CALLER | EXECUTE AS OWNER |
|---|---|---|
| **Droits** | Droits de l'appelant | Droits du propriétaire |
| **Sécurité** | Appelant doit avoir accès | Peut accéder à des objets privés |
| **Cas d'usage** | Opérations sur données utilisateur | Procédures admin système |

```python
session.sproc.register(
    ma_proc,
    execute_as="caller",   # l'appelant paie les droits
    # ou execute_as="owner"  # droits du propriétaire de la procédure
)
```

## Modules Python avec les procédures ⭐

```python
# Utiliser des packages Anaconda
session.sproc.register(
    ma_proc,
    packages=["snowflake-snowpark-python", "pandas", "numpy", "scikit-learn"]
)

# Rendre des modules disponibles (code custom)
session.add_import("@mon_stage/utils.py")
session.add_import("@mon_stage/config.json")

session.sproc.register(
    ma_proc,
    imports=["@mon_stage/utils.py"],  # disponible dans la procédure
    packages=["snowflake-snowpark-python"]
)
```

## DAG de Tasks exécutant des Stored Procedures ⭐

```python
# Python API pour créer des tasks
from snowflake.snowpark.functions import col

# Task parent (déclencheur)
session.sql("""
    CREATE OR REPLACE TASK task_root
    WAREHOUSE = wh_etl
    SCHEDULE  = 'USING CRON 0 2 * * * UTC'
    AS CALL pipeline_nettoyage('raw_table', 'clean_table')
""").collect()

# Task enfant
session.sql("""
    CREATE OR REPLACE TASK task_child
    AFTER task_root
    AS CALL pipeline_agregation('clean_table', 'summary_table')
""").collect()

# Activer (toujours désactivé par défaut !)
session.sql("ALTER TASK task_child RESUME").collect()
session.sql("ALTER TASK task_root RESUME").collect()
```
