# 2.3 — Monitoring des pipelines continus

> **Domaine D2 Performance — 19% du DEA-C02**

## Monitoring des Tasks ⭐

```sql
-- Historique des tasks (30 derniers jours)
SELECT *
FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
    SCHEDULED_TIME_RANGE_START => DATEADD('day', -7, CURRENT_TIMESTAMP()),
    RESULT_LIMIT => 100
))
WHERE state = 'FAILED'
ORDER BY scheduled_time DESC;

-- Via ACCOUNT_USAGE (latence 45 min)
SELECT name, state, error_code, error_message,
       scheduled_time, completed_time,
       DATEDIFF('second', scheduled_time, completed_time) AS duree_sec
FROM SNOWFLAKE.ACCOUNT_USAGE.TASK_HISTORY
WHERE state IN ('FAILED', 'TIMED_OUT')
  AND scheduled_time >= DATEADD('day', -1, CURRENT_TIMESTAMP())
ORDER BY scheduled_time DESC;
```

## Monitoring des Streams ⭐

```sql
-- Voir tous les streams du schéma
SHOW STREAMS IN SCHEMA ma_db.public;

-- Stream stale (données expirées) ?
SELECT * FROM INFORMATION_SCHEMA.STREAMS
WHERE stale = TRUE;

-- Vérifier si un stream a des données
SELECT SYSTEM$STREAM_HAS_DATA('stream_commandes');
```

## Monitoring Snowpipe Streaming ⭐

```sql
SELECT *
FROM TABLE(INFORMATION_SCHEMA.SNOWPIPE_STREAMING_CLIENT_HISTORY(
    START_TIME => DATEADD('hour', -1, CURRENT_TIMESTAMP())
))
WHERE error_count > 0;
```

## Alertes Snowflake ⭐

```sql
-- Notification integration (email)
CREATE NOTIFICATION INTEGRATION notif_pipeline
  TYPE = EMAIL ENABLED = TRUE
  ALLOWED_RECIPIENTS = ('said@exemple.com');

-- Alerte sur échec de task
CREATE ALERT alert_task_failure
  WAREHOUSE = wh_monitoring
  SCHEDULE  = '5 MINUTES'
  IF (EXISTS (
    SELECT 1 FROM TABLE(INFORMATION_SCHEMA.TASK_HISTORY(
      SCHEDULED_TIME_RANGE_START => DATEADD('minute', -5, CURRENT_TIMESTAMP())
    ))
    WHERE state = 'FAILED'
  ))
  THEN CALL SYSTEM$SEND_EMAIL(
    'notif_pipeline', 'said@exemple.com',
    'Task Failed', 'Un pipeline a échoué dans les 5 dernières minutes'
  );

ALTER ALERT alert_task_failure RESUME;
```

## Dynamic Tables Monitoring ⭐

```sql
SHOW DYNAMIC TABLES IN SCHEMA ma_db.public;

SELECT name, target_lag, scheduling_state, refresh_mode
FROM SNOWFLAKE.ACCOUNT_USAGE.DYNAMIC_TABLES
WHERE scheduling_state = 'SUSPENDED';

SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.DYNAMIC_TABLE_REFRESH_HISTORY
WHERE state = 'FAILED' ORDER BY refresh_start_time DESC LIMIT 10;
```
