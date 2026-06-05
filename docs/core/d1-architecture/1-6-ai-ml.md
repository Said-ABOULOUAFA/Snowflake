# 1.6 — AI/ML & Développement d'applications

> **Domaine D1 — 31% du COF-C03 — NOUVEAU dans C03 ⭐**

## Snowflake Cortex ⭐

Suite de fonctions AI/ML directement utilisables en SQL, sans infrastructure ML externe.

### AI SQL Functions

```sql
-- Résumer un texte
SELECT SNOWFLAKE.CORTEX.SUMMARIZE(description) AS resume FROM produits;

-- Traduire
SELECT SNOWFLAKE.CORTEX.TRANSLATE(commentaire, 'fr', 'en') AS traduction FROM avis;

-- Analyser le sentiment (-1 négatif → +1 positif)
SELECT SNOWFLAKE.CORTEX.SENTIMENT(avis) AS score FROM feedbacks;

-- Complétion LLM
SELECT SNOWFLAKE.CORTEX.COMPLETE(
    'mistral-7b',
    'Résume ce contrat en 3 points : ' || texte
) AS resume FROM contrats;

-- Classifier un texte
SELECT SNOWFLAKE.CORTEX.CLASSIFY_TEXT(
    description,
    ['positif', 'négatif', 'neutre']
) AS categorie FROM avis;

-- Extraire une réponse depuis un texte
SELECT SNOWFLAKE.CORTEX.EXTRACT_ANSWER(texte, 'Quel est le prix ?') AS reponse FROM docs;
```

### Modèles LLM disponibles

| Modèle | Caractéristique |
|---|---|
| `snowflake-arctic` | Modèle Snowflake natif, optimisé coût |
| `mistral-7b` | Rapide, usage général |
| `llama3-8b` | Open source Meta |
| `llama3-70b` | Plus puissant |
| `mixtral-8x7b` | Bon équilibre perf/coût |

### Cortex Search

Recherche **sémantique vectorielle** sur tes données textuelles.

```sql
CREATE CORTEX SEARCH SERVICE recherche_docs
  ON ma_table COLUMNS (titre, contenu)
  WAREHOUSE = wh_cortex
  TARGET_LAG = '1 hour';
```

### Cortex Analyst

Permet aux utilisateurs de poser des questions en **langage naturel** — Cortex génère le SQL automatiquement.

---

## Snowflake Notebooks ⭐

Environnement interactif (style Jupyter) directement dans Snowsight.

```python
# Python dans un Notebook Snowflake
import snowflake.snowpark.functions as F

# Accès direct à la session Snowflake (pas besoin de configurer)
df = session.table("ventes")
df.filter(F.col("region") == "EMEA") \
  .group_by("date_vente") \
  .agg(F.sum("montant").alias("total")) \
  .show()
```

**Fonctionnalités :**
- Supporte **Python, SQL et Markdown** dans le même notebook
- Exécution sur le warehouse Snowflake (pas de serveur local)
- Partage et collaboration via Snowsight
- Idéal pour : exploration, ML, pipelines d'ingestion

!!! info "Default warehouse for Notebooks"
    Cette fonctionnalité n'est **pas encore testée** à l'examen COF-C03 (pas globalement GA au moment de la rédaction du study guide).

---

## Streamlit in Snowflake ⭐

Créer des **applications web interactives** directement dans Snowflake, sans infrastructure externe.

```python
# Application Streamlit dans Snowflake
import streamlit as st
from snowflake.snowpark.context import get_active_session

session = get_active_session()

st.title("Dashboard Ventes")
region = st.selectbox("Région", ["EMEA", "AMER", "APAC"])

df = session.sql(f"""
    SELECT DATE_TRUNC('month', date_vente) AS mois, SUM(montant) AS total
    FROM ventes WHERE region = '{region}'
    GROUP BY 1 ORDER BY 1
""").to_pandas()

st.line_chart(df.set_index('MOIS'))
```

**Avantages :**
- Données restent dans Snowflake (sécurité maximale)
- Pas de serveur à gérer
- Hébergé et partageable via Snowsight
- Idéal pour : dashboards self-service, accès aux données

---

## Snowpark ⭐

Écrire du code Python/Java/Scala qui s'exécute **directement dans Snowflake**.

```python
from snowflake.snowpark.functions import col, sum as sum_

# Connexion
session = Session.builder.configs({...}).create()

# Transformation Snowpark = SQL exécuté dans Snowflake
df = session.table("ventes")
df.filter(col("region") == "EMEA") \
  .group_by("region") \
  .agg(sum_("montant").alias("total")) \
  .write.mode("overwrite").save_as_table("ventes_emea")
```

**Langages supportés :** Python, Java, Scala

---

## Snowflake ML ⭐

Fonctions ML natives dans SQL (sans Python requis).

```sql
-- Prévision de séries temporelles
CREATE SNOWFLAKE.ML.FORECAST modele_prev (
    INPUT_DATA => SYSTEM$REFERENCE('VIEW', 'ventes_historique'),
    TIMESTAMP_COLNAME => 'date_vente',
    TARGET_COLNAME => 'montant'
);
CALL modele_prev!FORECAST(FORECASTING_PERIODS => 30);

-- Détection d'anomalies
CREATE SNOWFLAKE.ML.ANOMALY_DETECTION modele_anom (
    INPUT_DATA => SYSTEM$REFERENCE('VIEW', 'metriques'),
    TIMESTAMP_COLNAME => 'ts',
    TARGET_COLNAME => 'valeur'
);
```
