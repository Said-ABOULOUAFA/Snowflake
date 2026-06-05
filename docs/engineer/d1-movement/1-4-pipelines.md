# 1.4 Pipelines de données continus

> **Domain 1.0 — Data Movement (28%)**

![Pipeline continu Stage → Snowpipe → Stream → Task](../../assets/continuous-pipeline.svg)

## Briques

| Objet | Rôle |
|---|---|
| **Stage** | Zone d'atterrissage des fichiers |
| **Snowpipe** | Ingestion auto (fichiers) ; **Snowpipe Streaming** (lignes) |
| **Stream** | Capture des changements (CDC) sur une table |
| **Task** | Exécution planifiée / dépendante de SQL ou procédures |
| **Dynamic Table** | Transformation déclarative auto-rafraîchie (`TARGET_LAG`) |
| **Materialized View** | Agrégat maintenu auto |

## Streams + Tasks (MERGE incrémental) ⭐

```sql
CREATE STREAM s_raw ON TABLE ventes_raw;

CREATE TASK t_merge
  WAREHOUSE = wh_etl
  SCHEDULE = '5 MINUTE'
  WHEN SYSTEM$STREAM_HAS_DATA('s_raw')
AS
  MERGE INTO ventes t USING s_raw s ON t.id = s.id
  WHEN MATCHED AND s.METADATA$ACTION='DELETE' THEN DELETE
  WHEN MATCHED THEN UPDATE SET t.montant = s.montant
  WHEN NOT MATCHED THEN INSERT (id, montant) VALUES (s.id, s.montant);

ALTER TASK t_merge RESUME;
```

## Dynamic Table (alternative déclarative)

```sql
CREATE DYNAMIC TABLE dt_ventes
  TARGET_LAG = '5 minutes'
  WAREHOUSE = wh_etl
AS SELECT region, SUM(montant) total FROM ventes GROUP BY region;
```

| Stream/Task | Dynamic Table |
|---|---|
| Contrôle fin, impératif | Déclaratif, géré par Snowflake |
| MERGE explicite | `TARGET_LAG` |

!!! tip "Snowpipe Streaming vs Kafka connector"
    Snowpipe Streaming ingère des **lignes** (faible latence). Le Kafka connector peut s'appuyer dessus pour du streaming quasi temps réel.

- Pipelines aussi orchestrables via **Notebooks**, **Snowflake Scripting**, **SQL API**, **Openflow**.

📎 *Réf. : `docs.snowflake.com/en/user-guide/data-pipelines-intro`*
