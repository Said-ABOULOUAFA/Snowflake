# 5.6 — Dev Workflows & Code Management

> **Domaine D5 Data Transformation — 25% du DEA-C02**

## Snowflake Notebooks ⭐

Environnement interactif Python/SQL dans Snowsight.

```python
# Pipeline complet dans un Notebook
import snowflake.snowpark.functions as F

# 1. Charger
df_raw = session.table("staging_ventes")
print(f"Lignes brutes: {df_raw.count()}")

# 2. Nettoyer
df_clean = (df_raw
    .filter(F.col("montant") > 0)
    .filter(F.col("client_id").isNotNull())
    .with_column("montant_eur", F.col("montant") / F.lit(1.08))
    .distinct()
)

# 3. Charger
df_clean.write.mode("append").save_as_table("ventes_propres")
print("Pipeline OK")
```

## Git Integration ⭐

```sql
CREATE API INTEGRATION git_github
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES = ('https://github.com/Said-ABOULOUAFA/')
  ENABLED = TRUE;

CREATE GIT REPOSITORY repo_snowflake
  ORIGIN = 'https://github.com/Said-ABOULOUAFA/Snowflake.git'
  API_INTEGRATION = git_github;

ALTER GIT REPOSITORY repo_snowflake FETCH;
EXECUTE IMMEDIATE FROM @repo_snowflake/branches/main/scripts/init.sql;
```

## Snowflake dbt Projects ⭐

```yaml
# profiles.yml
my_snowflake_profile:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: mon_compte.eu-west-1
      user: dbt_user
      authenticator: externalbrowser
      role: transformer
      database: analytics
      warehouse: wh_dbt
      schema: public
      threads: 4
```

```sql
-- dbt model : models/ventes_clean.sql
{{ config(
    materialized='incremental',
    unique_key='id'
) }}
SELECT * FROM {{ source('raw', 'ventes') }}
WHERE montant > 0
{% if is_incremental() %}
AND date_vente > (SELECT MAX(date_vente) FROM {{ this }})
{% endif %}
```

## Code Deployment Pipelines ⭐

```yaml
# GitHub Actions CI/CD
name: Deploy Snowpark
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: pip install snowflake-snowpark-python
      - run: |
          snow snowpark deploy \
            --account ${{ secrets.SF_ACCOUNT }} \
            --user ${{ secrets.SF_USER }} \
            --private-key-path private_key.p8
```

## Testing & Validation ⭐

```python
# pytest avec Snowpark local testing
import pytest
from snowflake.snowpark.testing import mock_session

@pytest.fixture
def session():
    with mock_session() as s:
        yield s

def test_pipeline(session):
    df = session.create_dataframe([(1, 100.0), (2, -50.0)], schema=["id", "montant"])
    from mon_module import nettoyer
    result = nettoyer(session, df).collect()
    assert len(result) == 1  # seule la ligne positive
    assert result[0]["MONTANT"] == 100.0
```
