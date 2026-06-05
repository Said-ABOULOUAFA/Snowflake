# 2.2 — Data Governance

> **Domaine D2 — 20% du COF-C03**

## Dynamic Data Masking ⭐ (Enterprise+)

Masque les données sensibles selon le rôle — les données en base ne changent PAS.

```sql
CREATE MASKING POLICY masque_email
  AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() IN ('DBA', 'COMPLIANCE') THEN val
    WHEN CURRENT_ROLE() = 'ANALYST'              THEN '***@***.***'
    ELSE '** CONFIDENTIEL **'
  END;

ALTER TABLE clients MODIFY COLUMN email SET MASKING POLICY masque_email;
ALTER TABLE clients MODIFY COLUMN email UNSET MASKING POLICY;
```

!!! danger "Points clés exam"
    - Nécessite **Enterprise minimum**
    - Les données en base **ne changent pas** — seul l'affichage est masqué
    - La politique s'applique à la **colonne**, pas à la table

## Row Access Policies ⭐ (Enterprise+)

Filtre les lignes selon le rôle — les lignes non autorisées sont **invisibles**.

```sql
CREATE ROW ACCESS POLICY rap_region
  AS (region STRING) RETURNS BOOLEAN ->
  CASE
    WHEN CURRENT_ROLE() = 'ADMIN' THEN TRUE
    ELSE region = CURRENT_ROLE()
  END;

ALTER TABLE ventes ADD ROW ACCESS POLICY rap_region ON (region);
```

## Object Tagging ⭐

```sql
CREATE TAG sensibilite ALLOWED_VALUES 'PII', 'Confidentiel', 'Public';
ALTER TABLE clients MODIFY COLUMN email SET TAG sensibilite = 'PII';

SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.TAG_REFERENCES
WHERE TAG_NAME = 'SENSIBILITE' AND TAG_VALUE = 'PII';
```

## Privacy Policies ⭐

```sql
CREATE PRIVACY POLICY pp_gdpr AS () RETURNS PRIVACY_BUDGET ->
  PRIVACY_BUDGET(BUDGET => 10.0);
ALTER TABLE clients MODIFY COLUMN email SET PRIVACY POLICY pp_gdpr;
```

## Trust Center ⭐

!!! danger "Question COF-C03 officielle"
    **Q : Un administrateur veut évaluer son compte contre les recommandations de sécurité. Quelle fonctionnalité utiliser ?**
    **R : Trust Center** (réponse A dans les sample questions officielles)

Tableau de bord de sécurité dans Snowsight :
- Détecte configurations risquées (utilisateurs sans MFA, etc.)
- Recommandations de sécurité automatiques
- Accessible par ACCOUNTADMIN et SECURITYADMIN

## Encryption Key Management ⭐

| Édition | Gestion des clés |
|---|---|
| Standard/Enterprise | Snowflake gère les clés |
| **Business Critical** | **Tri-secret key** (clé client via HSM) |
| Virtual Private | Environnement isolé |

## Alerts & Notifications ⭐

```sql
CREATE NOTIFICATION INTEGRATION notif_email TYPE = EMAIL ENABLED = TRUE;

CREATE ALERT alerte_pipeline
  WAREHOUSE = wh_monitoring
  SCHEDULE  = '5 MINUTES'
  IF (EXISTS (
    SELECT 1 FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
      SCHEDULED_TIME_RANGE_START => DATEADD('minute', -5, CURRENT_TIMESTAMP())
    ))
    WHERE state = 'FAILED'
  ))
  THEN CALL SYSTEM$SEND_EMAIL('notif_email','said@exemple.com','Échec pipeline','Details...');

ALTER ALERT alerte_pipeline RESUME;
```

!!! tip "Resource Monitors ≠ crédits supplémentaires"
    Les resource monitors **ne consomment PAS** de crédits — ils surveillent uniquement.
    (Question officielle COF-C03 sample Q2 : réponse C)

## Data Replication & Failover ⭐

```sql
ALTER DATABASE ma_db ENABLE REPLICATION TO ACCOUNTS aws.eu-west-1.compte_dr;
CREATE DATABASE ma_db_replica AS REPLICA OF source.aws.eu-west-1.ma_db;
ALTER DATABASE ma_db_replica REFRESH;
ALTER DATABASE ma_db_replica PRIMARY;  -- failover
```

## Data Lineage ⭐

Traçabilité automatique dans Snowsight → Governance → Data Lineage.

```sql
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.OBJECT_DEPENDENCIES
WHERE referenced_object_name = 'VENTES';
```
