# 3.1 — Data Recovery : Time Travel, Fail-safe, Réplication

> **Domaine D3 Storage — 14% du DEA-C02**

## Time Travel avancé ⭐

```sql
-- Récupérer des données supprimées par erreur
SELECT * FROM ventes AT (TIMESTAMP => '2024-06-01 00:00:00'::TIMESTAMP_TZ);
SELECT * FROM ventes AT (OFFSET => -3600);           -- il y a 1 heure
SELECT * FROM ventes BEFORE (STATEMENT => 'query_id'); -- avant un statement

-- Configurer par objet
ALTER TABLE ventes SET DATA_RETENTION_TIME_IN_DAYS = 30;
ALTER SCHEMA finance SET DATA_RETENTION_TIME_IN_DAYS = 10;
ALTER DATABASE ma_db SET DATA_RETENTION_TIME_IN_DAYS = 1;

-- Question officielle DEA-C02 :
-- Solution la plus économique : ALTER SCHEMA (pas DATABASE ni ACCOUNT)
ALTER SCHEMA SALES SET DATA_RETENTION_TIME_IN_DAYS = 10;

-- Impact des Streams sur Time Travel
-- Un stream doit être consommé AVANT expiration du Time Travel
-- Sinon le stream devient STALE (invalide)
SHOW STREAMS;
-- → colonne "stale" indique si le stream est expiré
```

## Fail-safe ⭐

| Propriété | Valeur |
|---|---|
| Durée | **7 jours** — fixe, non configurable |
| Accès | **Support Snowflake UNIQUEMENT** |
| Tables concernées | **Permanentes uniquement** |
| Après Time Travel | Commence à J+Time_Travel_period |

## Réplication cross-region & cross-cloud ⭐

```sql
-- Activer la réplication
ALTER DATABASE ma_db ENABLE REPLICATION TO ACCOUNTS aws.us-east-1.compte_dr;
ALTER DATABASE ma_db ENABLE REPLICATION TO ACCOUNTS azure.westeurope.compte_dr;

-- Côté secondaire : créer le replica
CREATE DATABASE ma_db_replica AS REPLICA OF source.aws.eu-west-1.ma_db;

-- Rafraîchir
ALTER DATABASE ma_db_replica REFRESH;

-- Failover (promouvoir le secondaire en primaire)
ALTER DATABASE ma_db_replica PRIMARY;
ALTER DATABASE ma_db_replica FAILOVER;

-- Réplication de stages, pipes, historique de chargement
ALTER DATABASE ma_db ENABLE REPLICATION
  TO ACCOUNTS aws.us-east-1.dr
  INCLUDE FAILOVER GROUPS;
```
