# 1.4 — Pipelines de données continus

> **Domaine D1 Data Movement — 28% du DEA-C02**

## Pattern Stream + Task (CDC) ⭐

```sql
-- Bronze → Silver avec Stream + Task
CREATE STREAM stream_orders ON TABLE orders_raw;

CREATE TASK task_process_orders
  WAREHOUSE = wh_etl
  SCHEDULE  = '5 MINUTE'
  WHEN SYSTEM$STREAM_HAS_DATA('stream_orders')
AS
MERGE INTO orders_silver AS tgt
USING (SELECT * FROM stream_orders WHERE METADATA$ACTION = 'INSERT') AS src
  ON tgt.id = src.id
WHEN MATCHED THEN UPDATE SET tgt.status = src.status
WHEN NOT MATCHED THEN INSERT (id, status, amount) VALUES (src.id, src.status, src.amount);

ALTER TASK task_process_orders RESUME;  -- TOUJOURS requis !
```

## Dynamic Tables ⭐

```sql
CREATE DYNAMIC TABLE dt_bronze
  TARGET_LAG = '5 minutes' WAREHOUSE = wh_etl
AS SELECT * FROM raw_stage WHERE is_valid = TRUE;

CREATE DYNAMIC TABLE dt_silver
  TARGET_LAG = DOWNSTREAM WAREHOUSE = wh_etl
AS SELECT *, UPPER(nom) AS nom_norm FROM dt_bronze;

CREATE DYNAMIC TABLE dt_gold
  TARGET_LAG = DOWNSTREAM WAREHOUSE = wh_etl
AS SELECT region, SUM(montant) AS total FROM dt_silver GROUP BY region;
```

## Snowpipe — Auto Ingest vs REST API ⭐

```sql
-- Auto Ingest (SQS/Event Grid/Pub Sub)
CREATE PIPE pipe_auto AUTO_INGEST = TRUE AS
COPY INTO ventes FROM @stage_s3/ventes/ FILE_FORMAT = (FORMAT_NAME = fmt_parquet);

-- REST API (déclenchement explicite)
CREATE PIPE pipe_api AUTO_INGEST = FALSE AS
COPY INTO ventes FROM @stage_s3/ventes/ FILE_FORMAT = (FORMAT_NAME = fmt_parquet);
-- POST https://compte.snowflakecomputing.com/v1/data/pipes/.../insertFiles
```

## Snowflake Scripting ⭐

```sql
CREATE OR REPLACE PROCEDURE run_etl(p_date STRING)
RETURNS STRING LANGUAGE SQL AS
$$
DECLARE nb_rows INTEGER;
BEGIN
    COPY INTO raw_orders FROM @stage_s3/orders/
      PATTERN = CONCAT('.*', :p_date, '.*\\.parquet');
    nb_rows := SQLROWCOUNT;
    INSERT INTO orders_clean
      SELECT * FROM raw_orders WHERE DATE(created_at) = :p_date::DATE;
    RETURN 'OK : ' || nb_rows::STRING || ' lignes';
EXCEPTION WHEN OTHER THEN
    RETURN 'ERREUR : ' || SQLERRM;
END;
$$;
CALL run_etl('2024-06-01');
```

## Openflow (à venir)

Orchestration de pipelines native dans Snowflake. **Non testé à l'examen** tant que pas globalement GA.
