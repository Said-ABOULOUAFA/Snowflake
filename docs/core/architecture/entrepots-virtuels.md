# Entrepôts Virtuels

## Tailles disponibles

| Taille | Crédit/heure | Nœuds | Cas d'usage |
|---|---|---|---|
| X-Small | 1 | 1 | Dev, tests |
| Small | 2 | 2 | Requêtes légères |
| Medium | 4 | 4 | BI standard |
| Large | 8 | 8 | ETL, agrégations |
| X-Large | 16 | 16 | Gros volumes |
| 2X-Large | 32 | 32 | Data Science |
| ... | ... | ... | Jusqu'à 6X-Large |

!!! info "Règle simple"
    Chaque taille double le nombre de nœuds et le coût en crédits.

---

## Configuration recommandée

```sql
CREATE WAREHOUSE wh_production
  WAREHOUSE_SIZE    = 'MEDIUM'
  AUTO_SUSPEND      = 60        -- secondes d'inactivité avant suspension
  AUTO_RESUME       = TRUE      -- redémarre automatiquement
  MAX_CONCURRENCY_LEVEL = 8     -- requêtes simultanées max par cluster
  COMMENT = 'Warehouse production BI';
```

---

## Warehouse Queuing

Quand trop de requêtes arrivent en même temps :

1. Le warehouse met en file d'attente les requêtes supplémentaires
2. Avec **Multi-cluster** (Enterprise+) → nouveaux clusters créés automatiquement
3. Sans Multi-cluster → les requêtes attendent

!!! warning "Piège exam"
    `MAX_CONCURRENCY_LEVEL` ne s'applique qu'**au sein d'un seul cluster**. Pour scaler au-delà, il faut le multi-cluster.

---

## Crédit vs Facturation

- Les crédits sont consommés **par seconde** (minimum 60 secondes)
- Un warehouse **suspendu** ne consomme **aucun crédit**
- Le stockage est facturé séparément des crédits de calcul

```sql
-- Voir la consommation (30 derniers jours)
SELECT warehouse_name,
       SUM(credits_used)          AS credits_calcul,
       SUM(credits_used_cloud_services) AS credits_services
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY
WHERE start_time >= DATEADD('day', -30, CURRENT_TIMESTAMP())
GROUP BY 1
ORDER BY 2 DESC;
```
