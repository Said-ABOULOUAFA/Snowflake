# 1.6 Data sharing & solutions de consommation

> **Domain 1.0 — Data Movement (28%)**

## Share ou clone ? ⭐

| Besoin | Choix |
|---|---|
| Donner accès live, sans copie, à un autre compte | **Data Share** |
| Créer un environnement isolé modifiable (dev/test) | **Zero-Copy Clone** |

## Implémenter un share

```sql
CREATE SHARE ventes_share;
GRANT USAGE ON DATABASE sales TO SHARE ventes_share;
GRANT USAGE ON SCHEMA sales.public TO SHARE ventes_share;
GRANT SELECT ON TABLE sales.public.ventes TO SHARE ventes_share;
ALTER SHARE ventes_share ADD ACCOUNTS = consumer_acct;
```

- **Auto-fulfillment** : réplication automatique du listing vers d'autres régions/clouds pour les consommateurs distants.
- **Row-level filtering** : limiter les lignes visibles par consommateur (secure views / row access policies).

## Consommation & data apps

| Outil | Usage |
|---|---|
| **Secure views** | Exposer une vue filtrée plutôt que la table brute |
| **Marketplace / listing** | Distribution publique ou privée |
| **Streamlit** | Dashboards interactifs, apps self-service de consommation |

```sql
CREATE SECURE VIEW v_ventes_client AS
  SELECT * FROM ventes WHERE client_id = CURRENT_USER();   -- ex. filtrage
GRANT SELECT ON VIEW v_ventes_client TO SHARE ventes_share;
```

!!! warning "Lecture seule"
    Le consommateur ne peut **pas** modifier ni re-partager les objets reçus.

📎 *Réf. : `docs.snowflake.com/en/user-guide/data-sharing-provider`*
