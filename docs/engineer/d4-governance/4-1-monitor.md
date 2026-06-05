# 4.1 — Monitoring des données

> **Domaine D4 Data Governance — 14% du DEA-C02**

## Object Tagging et Classification ⭐

```sql
-- Créer des tags
CREATE TAG sensibilite ALLOWED_VALUES 'PII', 'Confidentiel', 'Public';
CREATE TAG domaine ALLOWED_VALUES 'Finance', 'RH', 'Marketing';

-- Appliquer les tags
ALTER TABLE clients MODIFY COLUMN email SET TAG sensibilite = 'PII';
ALTER TABLE clients SET TAG domaine = 'RH';
ALTER SCHEMA finance SET TAG domaine = 'Finance';

-- Classification automatique (Snowflake Horizon)
-- Détecte EMAIL, PHONE, SSN, CREDIT_CARD automatiquement
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.OBJECT_TAG_POLICY_COVERAGE;

-- Voir tous les objets taggés PII
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.TAG_REFERENCES
WHERE TAG_NAME = 'SENSIBILITE' AND TAG_VALUE = 'PII';
```

## Data Lineage ⭐

```sql
-- Dépendances des objets
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.OBJECT_DEPENDENCIES
WHERE referenced_object_name = 'VENTES';

-- Lineage via Access History
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY
WHERE ARRAY_CONTAINS('VENTES'::VARIANT, direct_objects_accessed::VARIANT)
ORDER BY query_start_time DESC LIMIT 20;
```

## Data Quality Monitoring ⭐

```sql
-- Data Metric Functions (DMF)
CREATE DATA METRIC FUNCTION dmf_check_email(
    arg_t TABLE(email TEXT)
) RETURNS NUMBER AS $$
    SELECT COUNT_IF(NOT email LIKE '%@%.%') FROM arg_t
$$;

-- Associer à la table
ALTER TABLE clients ADD DATA METRIC FUNCTION dmf_check_email
  ON (email) SCHEDULE = '1 HOUR';

-- Résultats
SELECT * FROM SNOWFLAKE.LOCAL.DATA_QUALITY_MONITORING_RESULTS
WHERE metric_name = 'DMF_CHECK_EMAIL'
ORDER BY measurement_time DESC LIMIT 10;
```
