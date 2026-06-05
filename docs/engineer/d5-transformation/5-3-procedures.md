# 5.3 Procédures stockées & transactions

> **Domain 5.0 — Data Transformation (25%)**

## Langages disponibles ⭐

| Langage | Cas d'usage |
|---|---|
| **JavaScript** | Logique, boucles, gestion d'erreurs |
| **Python (Snowpark)** | ML, pandas, traitement avancé |
| **Java / Scala (Snowpark)** | Bases Java existantes |
| **SQL (scripting)** | Séquences de requêtes |

## EXECUTE AS CALLER vs OWNER ⭐

| | `CALLER` | `OWNER` |
|---|---|---|
| **Droits** | De l'appelant | Du propriétaire |
| **Sécurité** | Appelant doit avoir accès | Accès à des objets privés |
| **Usage** | Données de l'utilisateur | Procédures système |

## Snowpark Python stored proc

```python
def nettoyer(session, table_name: str) -> str:
    df = session.table(table_name).distinct().filter(col("id").is_not_null())
    df.write.mode("overwrite").save_as_table(f"{table_name}_clean")
    return f"{df.count()} lignes conservées"

session.sproc.register(func=nettoyer, name="nettoyer",
    packages=["snowflake-snowpark-python"], is_permanent=True,
    stage_location="@mon_stage", replace=True)
```

## Transactions

```sql
BEGIN;
  UPDATE comptes SET solde = solde - 100 WHERE id = 1;
  UPDATE comptes SET solde = solde + 100 WHERE id = 2;
COMMIT;   -- ou ROLLBACK en cas d'erreur
```

```sql
-- SQL scripting avec gestion d'exception
CREATE OR REPLACE PROCEDURE charger(src STRING, dst STRING)
RETURNS STRING LANGUAGE SQL AS $$
BEGIN
  EXECUTE IMMEDIATE 'INSERT INTO ' || dst || ' SELECT * FROM ' || src;
  RETURN 'OK : ' || SQLROWCOUNT || ' lignes';
EXCEPTION WHEN OTHER THEN
  RETURN 'ERREUR : ' || SQLERRM;
END; $$;
```

!!! danger "Piège exam"
    `EXECUTE AS OWNER` (défaut) exécute avec les droits du propriétaire — utile pour exposer une opération sans donner accès aux objets sous-jacents. `EXECUTE AS CALLER` utilise les droits de l'appelant. Une transaction non committée est **annulée** à la fin de la session.

📎 *Réf. : `docs.snowflake.com/en/developer-guide/stored-procedure/stored-procedures-overview`*
