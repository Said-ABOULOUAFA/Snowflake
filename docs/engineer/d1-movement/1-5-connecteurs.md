# 1.5 Connecteurs & intégrations

> **Domain 1.0 — Data Movement (28%)**

## Connecteurs à connaître ⭐

| Connecteur | Usage |
|---|---|
| **Kafka** | Streaming d'événements → Snowflake (s'appuie sur Snowpipe / Snowpipe Streaming) |
| **Spark** | Lire/écrire Snowflake depuis Spark ; pushdown possible |
| **Python** | `snowflake-connector-python` (DB-API), base de Snowpark |
| **Native connectors** | Connecteurs managés (Snowsight) vers sources tierces |

```python
# Connecteur Python
import snowflake.connector
con = snowflake.connector.connect(
    account='xy12345', user='said', password='***',
    warehouse='wh_etl', database='db', schema='public')
cur = con.cursor()
cur.execute("SELECT COUNT(*) FROM ventes")
print(cur.fetchone())
```

## Kafka connector — points clés

- Modes : **Snowpipe** (fichiers) ou **Snowpipe Streaming** (lignes, latence plus faible).
- Mappe topics → tables ; gère offsets et schéma.

!!! tip "Choix d'ingestion"
    Batch volumineux → COPY. Fichiers en continu → Snowpipe. Flux d'événements ligne-à-ligne faible latence → **Snowpipe Streaming / Kafka**.

📎 *Réf. : `docs.snowflake.com/en/user-guide/kafka-connector-overview`*
