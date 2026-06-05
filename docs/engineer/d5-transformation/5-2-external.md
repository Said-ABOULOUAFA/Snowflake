# 5.2 External functions

> **Domain 5.0 — Data Transformation (25%)**

Appelle une **API externe** (AWS Lambda, Azure Functions, GCP) depuis SQL.

```sql
CREATE API INTEGRATION api_scoring
  API_PROVIDER = aws_api_gateway
  API_AWS_ROLE_ARN = 'arn:aws:iam::123456789:role/snowflake-role'
  API_ALLOWED_PREFIXES = ('https://abc.execute-api.eu-west-1.amazonaws.com/prod/')
  ENABLED = TRUE;

CREATE EXTERNAL FUNCTION scoring_client(client_id INTEGER)
  RETURNS VARIANT
  API_INTEGRATION = api_scoring
  AS 'https://abc.execute-api.eu-west-1.amazonaws.com/prod/score';

SELECT client_id, scoring_client(client_id) AS score FROM clients LIMIT 100;
```

!!! warning "Considérations"
    - Latence réseau plus élevée qu'une UDF native.
    - Facturation = crédits Snowflake **+** coût de l'API externe.
    - Idéal pour : modèles ML hébergés ailleurs, tokenisation, enrichissement tiers.

!!! danger "Piège exam"
    Une external function nécessite **obligatoirement** une `API INTEGRATION` (objet sécurisé) qui référence le rôle/proxy cloud. Sans elle, impossible de créer la fonction. Pour du code custom *dans* Snowflake, préférer une UDF Python (pas d'appel réseau).

📎 *Réf. : `docs.snowflake.com/en/sql-reference/external-functions`*
