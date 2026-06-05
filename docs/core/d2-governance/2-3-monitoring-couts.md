# 2.3 Monitoring & gestion des coûts

> **Domain 2.0 — Account Management & Data Governance (20%)**

## Resource Monitors ⭐

Contrôlent la **consommation de crédits** des warehouses.

```sql
CREATE RESOURCE MONITOR rm_etl WITH
  CREDIT_QUOTA = 1000
  FREQUENCY = MONTHLY
  START_TIMESTAMP = IMMEDIATELY
  TRIGGERS
    ON 75 PERCENT DO NOTIFY
    ON 90 PERCENT DO SUSPEND
    ON 100 PERCENT DO SUSPEND_IMMEDIATE;

ALTER WAREHOUSE wh_etl SET RESOURCE_MONITOR = rm_etl;
```

| Action de trigger | Effet |
|---|---|
| **NOTIFY** | Envoie une alerte |
| **SUSPEND** | Suspend après les requêtes en cours |
| **SUSPEND_IMMEDIATE** | Tue les requêtes en cours |

!!! warning "Piège exam"
    Un resource monitor peut être attaché **au compte** ou **à des warehouses**. Seul **ACCOUNTADMIN** peut créer/gérer les resource monitors au niveau compte.

## Calcul de la consommation

```sql
-- Crédits par warehouse (30 j)
SELECT warehouse_name,
       SUM(credits_used) AS credits,
       SUM(credits_used_cloud_services) AS cloud_svc
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE start_time >= DATEADD('day', -30, CURRENT_TIMESTAMP())
GROUP BY 1 ORDER BY 2 DESC;
```

- Crédits facturés **à la seconde** (min 60 s).
- **Cloud services** facturés au-delà de 10 % du calcul quotidien.

## Schéma ACCOUNT_USAGE ⭐

Vues clés (latence ~45 min – 3 h) : `QUERY_HISTORY`, `WAREHOUSE_METERING_HISTORY`, `STORAGE_USAGE`, `ACCESS_HISTORY`, `LOGIN_HISTORY`, `WAREHOUSE_LOAD_HISTORY`.

!!! tip "INFORMATION_SCHEMA vs ACCOUNT_USAGE"
    - `INFORMATION_SCHEMA` : temps réel, rétention courte, par base.
    - `ACCOUNT_USAGE` : historique long (365 j), latence, niveau compte.

📎 *Réf. : `docs.snowflake.com/en/user-guide/resource-monitors`*
