# ⚙️ Cheat Sheet — DEA-C02 (Data Engineer)

> 65 questions · 115 min · score 750/1000 · 375 $ · **prérequis : SnowPro Core**

## Poids des domaines
| Domaine | % |
|---|---|
| 1.0 Data Movement | 28 |
| 2.0 Performance Optimization | 19 |
| 3.0 Storage & Data Protection | 14 |
| 4.0 Data Governance | 14 |
| 5.0 Data Transformation | 25 |

## Data Movement
- `COPY INTO`, `INFER_SCHEMA`, `MATCH_BY_COLUMN_NAME` pour Parquet/JSON.
- **Snowpipe** (auto, event notifications) vs **Snowpipe Streaming** (faible latence, rowset API).
- **Streams** (CDC) + **Tasks** (DAG) = pipelines continus.
- Troubleshoot : `COPY_HISTORY`, `VALIDATE()`, `VALIDATION_MODE`, `SYSTEM$PIPE_STATUS`.

## Performance ⭐
- **Query Profile** : repérer *spilling remote* (→ agrandir WH), *partitions scanned* (→ pruning/clustering), exploding joins.
- Multi-cluster = concurrence ; taille = puissance d'une requête.
- Leviers : Clustering keys, Search Optimization (point lookups), Materialized Views, QAS.

## Storage & protection
- Time Travel 0–90 j (Ent.), Fail-safe 7 j (non configurable).
- **Zero-copy clone** : copy-on-write, coût initial nul.
- `AT/BEFORE`, `UNDROP`, `SWAP WITH`, réplication/failover (BCDR).

## Governance
- Tags, Data Classification (`SYSTEM$CLASSIFY`), **Access History** (lineage).
- Masking policy (colonne), Row access policy (ligne), tag-based masking, clean rooms.

## Transformation ⭐
- UDF / UDTF / UDAF / **UDF vectorisée** (pandas, perf).
- External functions (API integration obligatoire).
- Stored procs (SQL/JS/Python), `EXECUTE AS OWNER|CALLER`, transactions.
- Semi-structuré : `VARIANT`, `LATERAL FLATTEN`, cast `::`.
- Cortex (serverless) : `COMPLETE`, `SUMMARIZE`, `SENTIMENT`.
- **Snowpark** : lazy, pushdown, `save_as_table`.
