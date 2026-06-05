# 5.2 — External Functions

> **Domaine D5 Data Transformation — 25% du DEA-C02**

## External Functions ⭐

Appeler des services externes (AWS Lambda, Azure Functions) depuis SQL.

```sql
-- Créer l'API Integration
CREATE API INTEGRATION api_lambda
  API_PROVIDER = aws_api_gateway
  API_AWS_ROLE_ARN = 'arn:aws:iam::123:role/sf-api-role'
  API_ALLOWED_PREFIXES = ('https://abc.execute-api.eu-west-1.amazonaws.com/prod/')
  ENABLED = TRUE;

-- Créer la External Function
CREATE EXTERNAL FUNCTION geocoder(adresse STRING)
RETURNS VARIANT
API_INTEGRATION = api_lambda
AS 'https://abc.execute-api.eu-west-1.amazonaws.com/prod/geocode';

-- Utiliser en SQL
SELECT adresse, geocoder(adresse):latitude::FLOAT AS lat,
                geocoder(adresse):longitude::FLOAT AS lon
FROM clients;
```

## Secure External Functions ⭐

```sql
-- Rendre sécurisée (DDL masqué)
CREATE SECURE EXTERNAL FUNCTION geocoder_secure(adresse STRING)
RETURNS VARIANT
API_INTEGRATION = api_lambda
AS 'https://abc.execute-api.eu-west-1.amazonaws.com/prod/geocode';
```

## Limites des External Functions

| Limitation | Valeur |
|---|---|
| Timeout | 60 secondes par appel |
| Taille payload | 6 MB max |
| Appels simultanés | Limités par AWS API GW |
| Coût | Crédits SF + coût du service externe |
