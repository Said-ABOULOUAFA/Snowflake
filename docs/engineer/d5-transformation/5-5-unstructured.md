# 5.5 Données non structurées & Cortex

> **Domain 5.0 — Data Transformation (25%)**

## Fichiers non structurés (stages + URLs)

```sql
-- Stage interne avec directory table
CREATE STAGE docs DIRECTORY = (ENABLE = TRUE);

-- Lister les fichiers
SELECT * FROM DIRECTORY(@docs);

-- Générer des URLs d'accès
SELECT GET_PRESIGNED_URL(@docs, relative_path) FROM DIRECTORY(@docs);
```

| Type d'URL | Usage |
|---|---|
| **Scoped URL** | Accès temporaire encodé (sûr dans une app) |
| **File URL** | Permanent, soumis aux privilèges |
| **Pre-signed URL** | Accès direct temporaire (sans login Snowflake) |

## Snowflake Cortex (LLM & ML) ⭐

```sql
-- Fonctions LLM serverless
SELECT SNOWFLAKE.CORTEX.SUMMARIZE(transcription)         AS resume,
       SNOWFLAKE.CORTEX.SENTIMENT(avis)                  AS sentiment,
       SNOWFLAKE.CORTEX.TRANSLATE(texte, 'fr', 'en')     AS traduction,
       SNOWFLAKE.CORTEX.COMPLETE('mistral-large', prompt) AS reponse
FROM documents;
```

| Fonction Cortex | Rôle |
|---|---|
| `COMPLETE` | Génération via LLM hébergé |
| `SUMMARIZE` | Résumé |
| `SENTIMENT` | Score de sentiment |
| `TRANSLATE` | Traduction |
| `EXTRACT_ANSWER` | Q/R sur texte |
| `EMBED_TEXT_*` | Vecteurs d'embedding |

!!! danger "Piège exam"
    Les fonctions **Cortex** sont **serverless** (pas de warehouse à dimensionner pour le modèle, facturation à l'usage). `COMPLETE` prend un nom de modèle + un prompt. Pour le traitement de fichiers (PDF, images), utiliser les **directory tables** + fonctions de parsing.

📎 *Réf. : `docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions`*
