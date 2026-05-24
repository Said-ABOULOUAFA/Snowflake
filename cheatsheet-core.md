# Monitoring & Performance avancée

## Query Acceleration Service (QAS) ⭐

```sql
-- Activer sur un warehouse
ALTER WAREHOUSE wh_analytics
  SET ENABLE_QUERY_ACCELERATION = TRUE
      QUERY_ACCELERATION_MAX_SCALE_FACTOR = 8;

-- Identifier les requêtes qui bénéficieraient du QAS
SELECT query_id, query_text, eligible_query_acceleration_time
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_ACCELERATION_ELIGIBLE
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY eligible_query_acceleration_time DESC
LIMIT 20;
```

!!! info "QAS vs Multi-cluster"
    - **QAS** → accélère une **requête individuelle** (scale up automatique)
    - **Multi-cluster** → gère la **concurrence** (beaucoup de requêtes simultanées)

---

## Snowpark-Optimized Warehouses

Warehouses avec plus de mémoire par nœud — idéaux pour les workloads Snowpark ML.

```sql
CREATE WAREHOUSE wh_snowpark
  WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'
  WAREHOUSE_SIZE = 'MEDIUM'
  AUTO_SUSPEND   = 300
  AUTO_RESUME    = TRUE;
```

!!! tip "Quand utiliser ?"
    - Entraînement de modèles ML avec Snowpark
    - Traitements Snowpark nécessitant beaucoup de mémoire
    - UDFs Python sur de grands volumes

---

## Monitoring des pipelines ⭐

### Monitoring des Tasks

```sql
-- Historique d'exécution des tasks
SELECT name, state, scheduled_time, completed_time, error_message
FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
    SCHEDULED_TIME_RANGE_START => DATEADD('hour', -24, CURRENT_TIMESTAMP()),
    TASK_NAME => 'MON_TASK'
))
ORDER BY scheduled_time DESC;

-- Via ACCOUNT_USAGE (latence ~45 min)
SELECT name, state, error_message, query_start_time
FROM SNOWFLAKE.ACCOUNT_USAGE.TASK_HISTORY
WHERE scheduled_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
  AND state = 'FAILED';
```

### Monitoring des Streams

```sql
-- Voir si un stream a des données non consommées
SELECT SYSTEM$STREAM_HAS_DATA('mon_stream') AS a_des_donnees;

-- Infos sur les streams
SHOW STREAMS;
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.STREAMS
WHERE deleted IS NULL;
```

### Monitoring des Dynamic Tables

```sql
-- Statut de rafraîchissement
SELECT name, state, target_lag, last_refresh_time, scheduling_state
FROM SNOWFLAKE.INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY
WHERE name = 'MA_TABLE_DYNAMIQUE'
ORDER BY last_refresh_time DESC;

-- Via Snowsight : Data → Databases → [ta table] → Refresh History
```

### Monitoring Snowpipe Streaming

```sql
-- Statut d'un channel de streaming
SELECT SYSTEM$GET_PRIVATELINK_CONFIG();

-- Voir les channels ouverts
SELECT channel_name, status, rows_inserted, error_count
FROM TABLE(INFORMATION_SCHEMA.SNOWPIPE_STREAMING_CLIENT_HISTORY(
    START_TIME => DATEADD('hour', -1, CURRENT_TIMESTAMP())
));
```

---

## Alerts & Notifications ⭐

Déclenche des actions ou notifications basées sur des conditions.

```sql
-- Créer une intégration de notification (email)
CREATE NOTIFICATION INTEGRATION notif_email
  TYPE          = EMAIL
  ENABLED       = TRUE;

-- Créer une alerte
CREATE ALERT alerte_echec_pipeline
  WAREHOUSE     = wh_monitoring
  SCHEDULE      = '5 MINUTES'
  IF (EXISTS (
      SELECT 1
      FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
          SCHEDULED_TIME_RANGE_START => DATEADD('minute', -5, CURRENT_TIMESTAMP())
      ))
      WHERE state = 'FAILED'
  ))
  THEN CALL SYSTEM$SEND_EMAIL(
      'notif_email',
      'said@exemple.com',
      'ALERTE : Echec pipeline Snowflake',
      'Une task a échoué. Vérifiez le tableau de bord Actions.'
  );

-- Activer l'alerte
ALTER ALERT alerte_echec_pipeline RESUME;
```

---

## Optimisation du stockage

```sql
-- Voir la consommation de stockage
SELECT table_name,
       ROUND(active_bytes / 1e9, 2)         AS active_gb,
       ROUND(time_travel_bytes / 1e9, 2)    AS time_travel_gb,
       ROUND(failsafe_bytes / 1e9, 2)       AS failsafe_gb,
       ROUND((active_bytes + time_travel_bytes + failsafe_bytes) / 1e9, 2) AS total_gb
FROM SNOWFLAKE.ACCOUNT_USAGE.TABLE_STORAGE_METRICS
WHERE deleted IS NULL
ORDER BY total_gb DESC
LIMIT 20;

-- Réduire le Time Travel pour économiser
ALTER TABLE ma_grande_table SET DATA_RETENTION_TIME_IN_DAYS = 1;

-- Utiliser des tables transitoires pour le staging
CREATE TRANSIENT TABLE staging_daily_load (
    data VARIANT
) DATA_RETENTION_TIME_IN_DAYS = 0;
```
