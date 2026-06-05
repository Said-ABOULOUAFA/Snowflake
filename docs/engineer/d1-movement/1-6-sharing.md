# 1.6 — Data Sharing et consommation

> **Domaine D1 Data Movement — 28% du DEA-C02**

## Data Share vs Clone ⭐

| Critère | Data Share | Clone (Zero-Copy) |
|---|---|---|
| **Copie données** | Non | Non (initial) |
| **Modifiable** | Non (lecture seule) | Oui |
| **Coût calcul** | Consommateur | Propriétaire du clone |
| **Isolation** | Non | Oui (après clone) |
| **Cas d'usage** | Partage externe | Dev/test/backup |

```sql
-- Data Share
CREATE SHARE partage_prod;
GRANT USAGE ON DATABASE sales_db TO SHARE partage_prod;
GRANT SELECT ON TABLE sales_db.public.ventes TO SHARE partage_prod;
ALTER SHARE partage_prod ADD ACCOUNTS = 'partenaire.eu-west-1.aws';

-- Clone
CREATE DATABASE db_dev CLONE db_prod;
```

## Auto-fulfillment ⭐

```sql
CREATE LISTING mon_listing FOR SHARE mon_partage AS $$
  title: 'Données Ventes 2024'
  auto_fulfillment:
    refresh_schedule: '10 MINUTE'
$$;
```

## Row-Level Filtering dans les partages ⭐

```sql
CREATE SECURE VIEW vue_partage AS
SELECT * FROM ventes
WHERE region = (
    SELECT region FROM mapping_comptes
    WHERE snowflake_account = CURRENT_ACCOUNT()
);
GRANT SELECT ON VIEW vue_partage TO SHARE mon_partage;
```

## Streamlit pour la consommation ⭐

```python
import streamlit as st
from snowflake.snowpark.context import get_active_session

session = get_active_session()
st.title("Dashboard Ventes")

region = st.selectbox("Région", ["EMEA", "AMER", "APAC"])
df = session.sql(f"SELECT * FROM ventes WHERE region = '{region}'").to_pandas()
st.dataframe(df)
st.bar_chart(df.set_index('DATE_VENTE')['MONTANT'])
```
