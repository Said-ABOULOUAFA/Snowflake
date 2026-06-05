# Connecteurs & Intégrations

## Vue d'ensemble ⭐

| Connecteur | Usage | Protocole |
|---|---|---|
| **JDBC Driver** | Java, outils BI (Tableau, Power BI) | JDBC |
| **ODBC Driver** | Applications Windows, Excel | ODBC |
| **Python Connector** | Scripts Python, notebooks | Python |
| **Node.js Driver** | Applications JavaScript | Node.js |
| **Snowflake CLI** | Terminal, scripts shell | CLI |
| **Kafka Connector** | Streaming depuis Kafka | Kafka |
| **Spark Connector** | Apache Spark | Spark |
| **dbt** | Transformations SQL | Python |

---

## Storage Integration ⭐

Permet à Snowflake d'accéder à un stockage cloud **sans clés en dur**.

```sql
-- AWS S3
CREATE STORAGE INTEGRATION si_aws
  TYPE                      = EXTERNAL_STAGE
  STORAGE_PROVIDER          = 'S3'
  ENABLED                   = TRUE
  STORAGE_AWS_ROLE_ARN      = 'arn:aws:iam::123456789:role/snowflake-role'
  STORAGE_ALLOWED_LOCATIONS = ('s3://mon-bucket/data/', 's3://mon-bucket-2/');

-- Récupérer les infos pour configurer AWS IAM
DESC INTEGRATION si_aws;
-- → STORAGE_AWS_IAM_USER_ARN et STORAGE_AWS_EXTERNAL_ID à mettre dans AWS

-- Azure
CREATE STORAGE INTEGRATION si_azure
  TYPE                      = EXTERNAL_STAGE
  STORAGE_PROVIDER          = 'AZURE'
  ENABLED                   = TRUE
  AZURE_TENANT_ID           = 'mon-tenant-id'
  STORAGE_ALLOWED_LOCATIONS = ('azure://moncompte.blob.core.windows.net/conteneur/');

-- GCS
CREATE STORAGE INTEGRATION si_gcs
  TYPE                      = EXTERNAL_STAGE
  STORAGE_PROVIDER          = 'GCS'
  ENABLED                   = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('gcs://mon-bucket/');
```

---

## API Integration ⭐

Pour les **External Functions** et **Git repositories**.

```sql
-- API Gateway AWS (pour External Functions)
CREATE API INTEGRATION api_lambda
  API_PROVIDER          = aws_api_gateway
  API_AWS_ROLE_ARN      = 'arn:aws:iam::123456789:role/snowflake-api-role'
  API_ALLOWED_PREFIXES  = ('https://abc123.execute-api.eu-west-1.amazonaws.com/prod/')
  ENABLED               = TRUE;

-- GitHub (pour Git Integration)
CREATE API INTEGRATION git_github
  API_PROVIDER          = git_https_api
  API_ALLOWED_PREFIXES  = ('https://github.com/Said-ABOULOUAFA/')
  ENABLED               = TRUE;
```

---

## Kafka Connector ⭐

```properties
# Configuration du connecteur Kafka → Snowflake
name=snowflake-sink
connector.class=com.snowflake.kafka.connector.SnowflakeSinkConnector
tasks.max=4

# Topics Kafka → Tables Snowflake
topics=commandes,events,logs
snowflake.topic2table.map=commandes:tbl_commandes,events:tbl_events

# Authentification Snowflake
snowflake.url.name=mon_compte.eu-west-1.snowflakecomputing.com
snowflake.user.name=kafka_user
snowflake.private.key=MIIEvg...

# Base et schéma cibles
snowflake.database.name=MA_DB
snowflake.schema.name=STREAMING

# Mode d'ingestion
snowflake.ingestion.method=SNOWPIPE_STREAMING  # ou SNOWPIPE

# Buffering (pour SNOWPIPE mode)
buffer.count.records=10000
buffer.flush.time=60
buffer.size.bytes=5000000
```

!!! info "SNOWPIPE vs SNOWPIPE_STREAMING"
    - **SNOWPIPE** : bufferise en fichiers → latence secondes/minutes
    - **SNOWPIPE_STREAMING** : direct row-level → latence < 1 seconde

---

## Spark Connector

```python
# Lire depuis Snowflake dans Spark
df = spark.read \
    .format("snowflake") \
    .options(**sfOptions) \
    .option("dbtable", "MA_TABLE") \
    .load()

# Écrire dans Snowflake depuis Spark
df.write \
    .format("snowflake") \
    .options(**sfOptions) \
    .option("dbtable", "MA_TABLE_DEST") \
    .mode("append") \
    .save()
```

!!! tip "Snowpark vs Spark Connector"
    - **Snowpark** → exécute le code directement dans Snowflake (recommandé)
    - **Spark Connector** → exécute dans Spark, lit/écrit depuis Snowflake (si Spark déjà en place)

---

## Python Connector

```python
import snowflake.connector

conn = snowflake.connector.connect(
    account   = 'mon_compte.eu-west-1',
    user      = 'mon_user',
    password  = 'mon_password',
    warehouse = 'wh_dev',
    database  = 'ma_db',
    schema    = 'public'
)

cur = conn.cursor()
cur.execute("SELECT COUNT(*) FROM ventes")
print(cur.fetchone())

# Avec paramètres (évite les injections SQL)
cur.execute(
    "SELECT * FROM ventes WHERE region = %s AND montant > %s",
    ('EMEA', 1000)
)
rows = cur.fetchall()

conn.close()
```
