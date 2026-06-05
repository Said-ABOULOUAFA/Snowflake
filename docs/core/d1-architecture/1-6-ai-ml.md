# 1.6 AI/ML & fonctionnalités de développement

> **Domain 1.0 — Architecture (31%)** — *renforcé dans COF-C03*

## Snowflake Cortex ⭐

Suite IA managée, accessible **en SQL** (pas besoin d'apprendre les API bas niveau pour l'examen).

| Fonction | Rôle |
|---|---|
| **AI SQL functions** | `COMPLETE`, `SUMMARIZE`, `TRANSLATE`, `SENTIMENT`, `EXTRACT_ANSWER`, `CLASSIFY_TEXT` |
| **Cortex Search** | Recherche sémantique / hybride (RAG) sur des documents |
| **Cortex Analyst** | Interrogation des données en langage naturel (texte → SQL) |

```sql
-- Exemples d'AI SQL functions
SELECT SNOWFLAKE.CORTEX.SENTIMENT(commentaire) FROM avis;
SELECT SNOWFLAKE.CORTEX.SUMMARIZE(texte) FROM articles;
SELECT SNOWFLAKE.CORTEX.COMPLETE('llama3.1-70b', 'Résume : ' || texte) FROM docs;
```

## Snowflake ML

- Entraînement et inférence **dans** Snowflake (ML functions, model registry).
- Pas de déplacement des données vers un environnement externe.

## Développement applicatif

| Outil | Description |
|---|---|
| **Snowflake Notebooks** | Cellules SQL + Python, intégrées à Snowsight |
| **Streamlit in Snowflake** | Apps de données interactives hébergées dans Snowflake |
| **Snowpark** | API DataFrame Python/Java/Scala, exécution pushdown (voir SPS-C01) |

!!! tip "Pour l'examen COF-C03"
    Savoir **ce que fait** chaque service Cortex et **quand l'utiliser**. Pas besoin de connaître les signatures d'API en détail.

📎 *Réf. : `docs.snowflake.com/en/guides-overview-ai-features`*
