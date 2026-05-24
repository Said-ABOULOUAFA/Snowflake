# Réplication & Business Continuity

## Réplication de bases de données ⭐

Copie les données vers un autre compte Snowflake (autre région ou autre cloud).

```sql
-- COMPTE SOURCE : Activer la réplication
ALTER DATABASE ma_db
  ENABLE REPLICATION TO ACCOUNTS aws.eu-west-1.compte_cible;

-- COMPTE CIBLE : Créer le replica
CREATE DATABASE ma_db_replica
  AS REPLICA OF source_account.aws.eu-west-1.ma_db;

-- COMPTE CIBLE : Rafraîchir le replica
ALTER DATABASE ma_db_replica REFRESH;

-- Voir le statut de la réplication
SELECT SYSTEM$DATABASE_REFRESH_STATUS('ma_db_replica');

-- Planifier des rafraîchissements automatiques
ALTER DATABASE ma_db_replica
  ENABLE REPLICATION
  REFRESH_SCHEDULE = '10 MINUTES';
```

---

## Réplication Groups ⭐

Réplique plusieurs bases de données + objets du compte en un seul groupe.

```sql
-- Créer un groupe de réplication (compte source)
CREATE REPLICATION GROUP rg_production
  OBJECT_TYPES = DATABASES, ROLES, USERS, WAREHOUSES, RESOURCE MONITORS
  DATABASES = ma_db, ma_db2
  ALLOWED_ACCOUNTS = aws.eu-west-1.compte_dr;

-- Créer le groupe secondaire (compte cible)
CREATE REPLICATION GROUP rg_production
  AS REPLICA OF source_account.aws.eu-west-1.rg_production;

-- Rafraîchir
ALTER REPLICATION GROUP rg_production REFRESH;
```

---

## Failover & Failback ⭐

```sql
-- FAILOVER : Promouvoir le replica en primaire (en cas de sinistre)
-- Sur le compte cible :
ALTER DATABASE ma_db_replica PRIMARY;
-- La base devient maintenant en lecture/écriture sur le compte cible

-- FAILBACK : Après récupération, retour au compte original
-- Sur le compte original (devenu replica) :
ALTER DATABASE ma_db PRIMARY;
```

!!! danger "Impact du Failover"
    - La base **originale devient un replica** (lecture seule)
    - Toutes les connexions doivent être **redirigées** vers le nouveau primaire
    - Utiliser les **Connection Policies** pour automatiser la redirection

---

## Connection Policies (Failover automatique)

```sql
-- Créer une connection policy pour le failover automatique
CREATE CONNECTION mon_endpoint
  PRIMARY;

-- Compte secondaire
CREATE CONNECTION mon_endpoint
  AS REPLICA OF source_account.mon_endpoint;

-- Failover de la connexion
ALTER CONNECTION mon_endpoint PRIMARY;
```

---

## Surveillance de la réplication

```sql
-- Historique des réplications
SELECT phase_name, start_time, end_time,
       bytes_transferred, object_count, status
FROM SNOWFLAKE.ACCOUNT_USAGE.REPLICATION_GROUP_REFRESH_HISTORY
WHERE replication_group_name = 'RG_PRODUCTION'
ORDER BY start_time DESC
LIMIT 10;

-- Coût de la réplication
SELECT start_time, credits_used
FROM SNOWFLAKE.ACCOUNT_USAGE.REPLICATION_GROUP_USAGE_HISTORY
WHERE start_time >= DATEADD('month', -1, CURRENT_TIMESTAMP());
```
