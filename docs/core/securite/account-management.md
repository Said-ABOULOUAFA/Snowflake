# Account Management & Gouvernance

## Resource Monitors ⭐

Contrôle la **consommation de crédits** des warehouses — déclenche des alertes ou suspend automatiquement.

```sql
-- Créer un resource monitor
CREATE RESOURCE MONITOR rm_mensuel
  CREDIT_QUOTA = 1000           -- limite mensuelle en crédits
  FREQUENCY    = MONTHLY
  START_TIMESTAMP = IMMEDIATELY
  TRIGGERS
    ON 75 PERCENT DO NOTIFY        -- alerte email à 75%
    ON 90 PERCENT DO NOTIFY        -- alerte email à 90%
    ON 100 PERCENT DO SUSPEND      -- suspend les warehouses
    ON 110 PERCENT DO SUSPEND_IMMEDIATE; -- suspension immédiate

-- Assigner à un warehouse
ALTER WAREHOUSE wh_analytics SET RESOURCE_MONITOR = rm_mensuel;

-- Assigner au compte entier
ALTER ACCOUNT SET RESOURCE_MONITOR = rm_mensuel;
```

!!! danger "Piège exam"
    - `SUSPEND` : attend la fin des requêtes en cours avant de suspendre
    - `SUSPEND_IMMEDIATE` : arrête tout immédiatement, requêtes annulées
    - Un resource monitor au niveau **compte** s'applique à **tous** les warehouses

---

## ACCOUNT_USAGE Schema ⭐

Vue d'ensemble de l'activité du compte Snowflake (latence ~45 min).

```sql
-- Requêtes les plus consommatrices (7 derniers jours)
SELECT query_text,
       execution_time / 1000        AS sec,
       credits_used_cloud_services,
       bytes_scanned
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY execution_time DESC
LIMIT 20;

-- Consommation par warehouse
SELECT warehouse_name,
       SUM(credits_used) AS total_credits
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE start_time >= DATEADD('month', -1, CURRENT_TIMESTAMP())
GROUP BY 1
ORDER BY 2 DESC;

-- Historique des accès aux données
SELECT user_name, query_text, direct_objects_accessed
FROM SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY
WHERE query_start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP());
```

---

## Rôles de base de données (Database Roles) ⭐ — Nouveau COF-C03

En plus des rôles de compte, Snowflake supporte les **rôles au niveau base de données**.

```sql
-- Créer un rôle de base de données
CREATE DATABASE ROLE ma_db.role_lecture;

-- Accorder des privilèges
GRANT SELECT ON ALL TABLES IN SCHEMA ma_db.public
  TO DATABASE ROLE ma_db.role_lecture;

-- Accorder le rôle DB à un rôle de compte
GRANT DATABASE ROLE ma_db.role_lecture TO ROLE analyst;
```

!!! info "Différence avec les rôles de compte"
    Les rôles de base de données sont **scoped à une base de données** — ils ne peuvent pas accéder à d'autres bases. Idéaux pour le partage de données et les applications natives.

---

## Secondary Roles

Permettent d'activer **plusieurs rôles simultanément** dans une session.

```sql
-- Activer tous les rôles dont dispose l'utilisateur
ALTER SESSION SET SECONDARY_ROLES = ALL;

-- Revenir au rôle principal uniquement
ALTER SESSION SET SECONDARY_ROLES = NONE;

-- Voir les rôles actifs
SELECT CURRENT_ROLE(), CURRENT_SECONDARY_ROLES();
```

---

## Trust Center

Tableau de bord de sécurité dans Snowsight pour surveiller la posture de sécurité du compte.

- Détecte les configurations risquées (ex: utilisateurs sans MFA)
- Recommandations de sécurité automatiques
- Visible par les rôles ACCOUNTADMIN et SECURITYADMIN

---

## Privacy Policies

Permettent de définir des règles de confidentialité sur les colonnes.

```sql
-- Créer une politique de confidentialité
CREATE PRIVACY POLICY pp_email
  AS () RETURNS PRIVACY_BUDGET ->
  PRIVACY_BUDGET(BUDGET => 10.0);

-- Appliquer sur une colonne
ALTER TABLE clients MODIFY COLUMN email
  SET PRIVACY POLICY pp_email;
```

---

## Data Lineage

Traçabilité automatique des données — visible dans Snowsight.

```sql
-- Voir les dépendances d'un objet
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.OBJECT_DEPENDENCIES
WHERE referenced_object_name = 'VENTES';
```

!!! tip "Snowsight"
    Dans Snowsight → Governance → Data Lineage : visualisation graphique des flux de données entre tables, vues et pipelines.

---

## Logging & Tracing

```sql
-- Activer les logs sur un objet
ALTER PROCEDURE ma_procedure SET LOG_LEVEL = DEBUG;
ALTER FUNCTION ma_fonction SET TRACE_LEVEL = ALWAYS;

-- Consulter les logs
SELECT * FROM ma_db.public.my_event_table
WHERE record_type = 'LOG'
ORDER BY timestamp DESC;
```
