# 5.5 — Données non structurées & Cortex

> **Domaine D5 Data Transformation — 25% du DEA-C02**

## Types d'URLs ⭐

| Fonction | Type d'URL | Durée |
|---|---|---|
| `BUILD_SCOPED_FILE_URL` | Temporaire sécurisée | Configurable |
| `BUILD_STAGE_FILE_URL` | Permanente | N/A |

```sql
-- URL temporaire sécurisée
SELECT BUILD_SCOPED_FILE_URL(@stage_docs, 'rapport.pdf') AS url_tmp;

-- URL permanente
SELECT BUILD_STAGE_FILE_URL(@stage_docs, 'rapport.pdf') AS url_perm;
```

## Directory Tables ⭐

```sql
CREATE STAGE stage_docs URL='s3://bucket/docs/'
  DIRECTORY = (ENABLE=TRUE AUTO_REFRESH=TRUE);
ALTER STAGE stage_docs REFRESH;

SELECT relative_path, size, last_modified,
       BUILD_SCOPED_FILE_URL(@stage_docs, relative_path) AS url
FROM DIRECTORY(@stage_docs)
WHERE relative_path LIKE '%.pdf';
```

## Snowflake Cortex pour données non structurées ⭐

```sql
-- Catégorisation automatique
SELECT id,
       SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
           description,
           ['Urgent', 'Normal', 'Info']
       ) AS priorite
FROM tickets_support;

-- Extraction de données depuis texte/images
SELECT SNOWFLAKE.CORTEX.COMPLETE(
    'mistral-7b',
    CONCAT('Extrais le montant et la date de cette facture : ', texte_facture)
) AS donnees_extraites FROM factures;

-- Analyse sémantique
SELECT SNOWFLAKE.CORTEX.SENTIMENT(avis) AS score_sentiment FROM avis_clients;

-- Résumer des documents longs
SELECT id, SNOWFLAKE.CORTEX.SUMMARIZE(contenu) AS resume FROM documents;

-- Traduction
SELECT SNOWFLAKE.CORTEX.TRANSLATE(texte, 'fr', 'en') AS traduction FROM rapports;

-- Pipelines text analytics
SELECT id,
       SNOWFLAKE.CORTEX.EMBED_TEXT_768('snowflake-arctic-embed-m', texte) AS embedding
FROM articles;
```

## Semantic Views ⭐

```sql
-- Vue sémantique pour le langage naturel (Cortex Analyst)
CREATE SEMANTIC VIEW sv_ventes
  TABLES (ventes AS v, clients AS c)
  RELATIONSHIPS (v.client_id = c.id)
  FACTS (v.montant)
  DIMENSIONS (c.region, c.segment, v.date_vente);

-- Utiliser avec Cortex Analyst
SELECT SNOWFLAKE.CORTEX.COMPLETE(
    'mistral-7b',
    'Quel est le total des ventes par région en 2024 ?',
    {'semantic_view': 'sv_ventes'}
);
```

## Cortex LLM Cost Management ⭐

```sql
-- Surveiller les coûts Cortex
SELECT function_name, SUM(token_credits) AS credits_total
FROM SNOWFLAKE.ACCOUNT_USAGE.CORTEX_FUNCTION_HISTORY
WHERE start_time >= DATEADD('day', -30, CURRENT_TIMESTAMP())
GROUP BY 1 ORDER BY 2 DESC;
```
