# Cheat Sheet DEA-C02 — Data Engineer

## Prérequis
- **COF-C03 actif obligatoire**
- 2+ ans expérience data engineering

## Data Movement (28%)

| Scénario | Solution |
|---|---|
| Fichiers > 64j à recharger sans dupliquer | `LOAD_UNCERTAIN_FILES = TRUE` |
| Pipeline Kafka temps réel | Kafka Connector + `SNOWPIPE_STREAMING` |
| Ingestion serverless auto | Snowpipe + AUTO_INGEST = TRUE |
| Détecter schéma Parquet | `INFER_SCHEMA()` |
| Fichiers volumineux | 100-250 MB optimal par fichier |

## Performance (19%)

| Signal Query Profile | Solution |
|---|---|
| Bytes spilled to disk | Scale UP warehouse |
| Partitions scanned = 100% | Ajouter clustering key |
| Queue time élevé | Multi-cluster warehouse |
| Cloud Services > 10% | Simplifier les requêtes |

## Storage (14%)

```
ALTER SCHEMA SALES SET DATA_RETENTION_TIME_IN_DAYS = 10;
-- Plus économique que ALTER DATABASE ou ALTER ACCOUNT
-- Car applique uniquement au schéma ciblé

UNDROP TABLE → libérer le nom avec RENAME d'abord si CREATE OR REPLACE
```

## Questions pièges DEA-C02

```
❗ Task timeout par défaut = 60 min → USER_TASK_TIMEOUT_MS
❗ TRY_PARSE_JSON → NULL sur JSON invalide (pas d'erreur)
❗ COPY INTO dédup = 64 jours
❗ Dynamic Tables → ALTER DYNAMIC TABLE SUSPEND/RESUME
❗ Hybrid Tables = PRIMARY KEY + INDEX + verrouillage ligne
❗ Horizon Catalog = fédérer données externes (Glue, Hive)
❗ Clean Rooms = partager sans exposer données brutes
❗ Projection Policies = masquer existence d'une colonne
```
