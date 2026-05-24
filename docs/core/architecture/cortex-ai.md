# Snowflake Cortex, Notebooks & Streamlit

## Snowflake Cortex ⭐ (COF-C03 : fort emphasis)

Suite de fonctions AI/ML directement dans SQL.

### Cortex AI — Fonctions SQL

```sql
-- Résumer un texte
SELECT SNOWFLAKE.CORTEX.SUMMARIZE(description) AS resume
FROM produits;

-- Traduire
SELECT SNOWFLAKE.CORTEX.TRANSLATE(commentaire, 'fr', 'en') AS en_anglais
FROM avis_clients;

-- Analyser le sentiment
SELECT SNOWFLAKE.CORTEX.SENTIMENT(avis) AS score_sentiment
FROM avis_clients;
-- Retourne un score entre -1 (négatif) et 1 (positif)

-- Extraction d'entités
SELECT SNOWFLAKE.CORTEX.EXTRACT_ANSWER(texte, 'Quel est le prix ?') AS reponse
FROM documents;

-- Classifier un texte
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
    texte,
    ['positif', 'négatif', 'neutre']
) AS categorie
FROM avis;

-- Complétion LLM (via un modèle)
SELECT SNOWFLAKE.CORTEX.COMPLETE(
    'mistral-7b',
    'Résume ce contrat en 3 points : ' || texte_contrat
) AS resume_contrat
FROM contrats;
```

### Modèles LLM disponibles dans Cortex

| Modèle | Caractéristique |
|---|---|
| `snowflake-arctic` | Modèle Snowflake natif, optimisé coût |
| `mistral-7b` | Rapide, usage général |
| `llama3-8b` | Open source Meta |
| `llama3-70b` | Plus puissant, plus coûteux |
| `mixtral-8x7b` | Bon équilibre perf/coût |

### Cortex Search

Recherche sémantique (vectorielle) sur tes données textuelles.

```sql
-- Créer un index Cortex Search
CREATE CORTEX SEARCH SERVICE recherche_produits
  ON ma_table
  COLUMNS (description, nom)
  WAREHOUSE = wh_cortex
  TARGET_LAG = '1 hour';
```

### Cortex Analyst

Permet aux utilisateurs de poser des questions en **langage naturel** sur leurs données — Cortex génère le SQL automatiquement.

---

## Snowflake Notebooks ⭐

Environnement interactif (style Jupyter) directement dans Snowsight.

```python
# Dans un Notebook Snowflake (Python)
import snowflake.snowpark.functions as F

# Accès direct à la session Snowflake
df = session.table("ventes")
df.filter(F.col("region") == "EMEA") \
  .group_by("date_vente") \
  .agg(F.sum("montant").alias("total")) \
  .show()
```

!!! info "Fonctionnalités clés"
    - Supporte Python, SQL, et Markdown dans le même notebook
    - Exécution sur le warehouse Snowflake (pas de serveur local)
    - Partage et collaboration via Snowsight
    - Idéal pour : exploration de données, ML, pipelines d'ingestion

---

## Streamlit in Snowflake ⭐

Créer des **applications web interactives** directement dans Snowflake, sans infrastructure externe.

```python
# Application Streamlit dans Snowflake
import streamlit as st
from snowflake.snowpark.context import get_active_session

session = get_active_session()

st.title("Dashboard Ventes")

# Sélecteur de région
region = st.selectbox("Région", ["EMEA", "AMER", "APAC"])

# Requête Snowflake
df = session.sql(f"""
    SELECT DATE_TRUNC('month', date_vente) AS mois,
           SUM(montant) AS total
    FROM ventes
    WHERE region = '{region}'
    GROUP BY 1 ORDER BY 1
""").to_pandas()

st.line_chart(df.set_index('MOIS'))
```

!!! tip "Avantages"
    - Données restent dans Snowflake (sécurité)
    - Pas de serveur à gérer
    - Hébergé et partageable via Snowsight
    - Idéal pour : dashboards self-service, outils d'accès aux données

---

## Snowflake ML

Fonctions ML natives dans SQL (sans Python requis) :

```sql
-- Prévision de séries temporelles
CREATE SNOWFLAKE.ML.FORECAST modele_prevision (
    INPUT_DATA  => SYSTEM$REFERENCE('VIEW', 'ventes_historique'),
    TIMESTAMP_COLNAME => 'date_vente',
    TARGET_COLNAME    => 'montant'
);

-- Appliquer la prévision
CALL modele_prevision!FORECAST(FORECASTING_PERIODS => 30);

-- Détection d'anomalies
CREATE SNOWFLAKE.ML.ANOMALY_DETECTION modele_anomalie (
    INPUT_DATA    => SYSTEM$REFERENCE('VIEW', 'metriques'),
    TIMESTAMP_COLNAME => 'ts',
    TARGET_COLNAME    => 'valeur'
);
```
