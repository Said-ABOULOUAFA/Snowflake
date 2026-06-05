# 2.3 — Monitoring & Gestion des coûts

> **Domaine D2 — 20% du COF-C03**

## Resource Monitors ⭐

Surveille et contrôle la consommation de crédits.

!!! danger "Question officielle COF-C03"
    **Q : Comment les Resource Monitors affectent-ils la consommation de crédits ?**
    **R : Ils trackent la consommation SANS coût supplémentaire** (réponse C officielle)

```sql
CREATE RESOURCE MONITOR rm_mensuel
  CREDIT_QUOTA = 1000
  FREQUENCY    = MONTHLY
  START_TIMESTAMP = IMMEDIATELY
  TRIGGERS
    ON 75 PERCENT DO NOTIFY
    ON 90 PERCENT DO NOTIFY
    ON 100 PERCENT DO SUSPEND
    ON 110 PERCENT DO SUSPEND_IMMEDIATE;

ALTER WAREHOUSE wh_analytics SET RESOURCE_MONITOR = rm_mensuel;
ALTER ACCOUNT SET RESOURCE_MONITOR = rm_mensuel;
```

| Action | Comportement |
|---|---|
| `NOTIFY` | Alerte email uniquement |
| `SUSPEND` | Attend la fin des requêtes avant de suspendre |
| `SUSPEND_IMMEDIATE` | Arrête immédiatement, requêtes annulées |

## ACCOUNT_USAGE Schema ⭐

```sql
-- Requêtes les plus coûteuses (7 derniers jours)
SELECT query_text, execution_time/1000 AS sec, bytes_scanned
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
ORDER BY execution_time DESC LIMIT 20;

-- Consommation par warehouse (30 derniers jours)
SELECT warehouse_name, SUM(credits_used) AS total
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE start_time >= DATEADD('month', -1, CURRENT_TIMESTAMP())
GROUP BY 1 ORDER BY 2 DESC;

-- Calcul des crédits par warehouse
-- Crédits = Taille × Durée_heures
-- XS=1cr/h, S=2cr/h, M=4cr/h, L=8cr/h, XL=16cr/h
```

## Calcul des crédits warehouse ⭐

```
Crédits consommés = (taille_warehouse_cr_par_heure × durée_secondes) / 3600
Minimum = 60 secondes par démarrage
```

```sql
-- Exemple : warehouse Medium (4 cr/h) actif 30 min
-- Crédits = 4 × (1800/3600) = 2 crédits
```
