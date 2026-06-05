# 3.2 Analyser les micro-partitions

> **Domain 3.0 — Storage & Data Protection (14%)**

## Principe

![Micro-partitions](../../assets/micro-partitions.svg)

Snowflake partitionne **automatiquement** chaque table en **micro-partitions** (~50–500 Mo non compressés, stockage colonne). Chaque micro-partition conserve des **métadonnées** : min/max par colonne, nombre de valeurs distinctes, etc. → c'est ce qui permet le **pruning**.

## Évaluer le clustering

```sql
SELECT SYSTEM$CLUSTERING_INFORMATION('ventes', '(date_vente)');
```

Champs clés du retour :

| Champ | Bon signe | Mauvais signe |
|---|---|---|
| `average_overlaps` | Faible | Élevé (partitions qui se chevauchent) |
| `average_depth` | Proche de 1 | Élevé → mauvais pruning |
| `partition_depth_histogram` | Concentré sur 1–2 | Étalé |

!!! danger "Piège exam"
    Un **average_depth élevé** = mauvais clustering = beaucoup de partitions scannées inutilement. Les micro-partitions sont **immuables** : un UPDATE/DELETE réécrit des partitions entières (important pour comprendre le coût des DML et le Time Travel).

!!! tip
    Comparer `partitions scanned` vs `partitions total` dans le Query Profile pour mesurer concrètement l'efficacité du pruning.

📎 *Réf. : `docs.snowflake.com/en/user-guide/tables-clustering-micropartitions`*
