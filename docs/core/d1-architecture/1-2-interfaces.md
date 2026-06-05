# 1.2 Utiliser les interfaces & outils Snowflake

> **Domain 1.0 — Architecture (31%)**

## Interfaces principales

| Interface | Usage |
|---|---|
| **Snowsight** | Interface web par défaut : worksheets, dashboards, Query Profile, monitoring, gestion d'objets |
| **Snowflake CLI** (`snow`) | Outil ligne de commande moderne (remplace progressivement SnowSQL) pour scripts, déploiement, Snowpark |
| **SnowSQL** | Client CLI SQL historique |
| **IDE / VS Code** | Extension Snowflake pour VS Code (édition SQL, Snowpark, notebooks) |
| **Drivers / connecteurs** | JDBC, ODBC, Python, Node.js, Go, .NET, Spark, Kafka |

## Snowsight — fonctions clés à connaître

- **Worksheets** : SQL et Python (Snowpark).
- **Query Profile** : analyse visuelle de l'exécution (voir 4.1).
- **Dashboards** : visualisations à partir de worksheets.
- **Data → Databases** : explorateur d'objets.
- **Admin** : warehouses, resource monitors, coûts, utilisateurs/rôles, **Trust Center**.

```sql
-- Contexte de session (souvent testé)
SELECT CURRENT_ACCOUNT(), CURRENT_REGION(), CURRENT_USER(),
       CURRENT_ROLE(), CURRENT_WAREHOUSE(), CURRENT_DATABASE(), CURRENT_SCHEMA();
```

!!! tip "Notebooks & Streamlit"
    **Snowflake Notebooks** (cellules SQL/Python) et **Streamlit in Snowflake** permettent de développer et publier des apps de données **dans** Snowflake, sans déplacer les données.

📎 *Réf. : `docs.snowflake.com/en/user-guide/ui-snowsight`*
