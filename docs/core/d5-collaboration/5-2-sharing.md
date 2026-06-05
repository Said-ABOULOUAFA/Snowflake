# 5.2 Data Sharing

> **Domain 5.0 — Data Collaboration (10%)**

## Secure Data Sharing ⭐

Partage **sans copie** : le consommateur lit les données du fournisseur via les **métadonnées** (aucune duplication, toujours à jour).

| Acteur | Rôle |
|---|---|
| **Provider** | Crée le share, accorde l'accès aux objets |
| **Consumer** | Compte Snowflake qui consomme le share |
| **Reader account** | Compte géré/payé par le provider pour un consommateur **sans** compte Snowflake |

```sql
CREATE SHARE ventes_share;
GRANT USAGE ON DATABASE sales TO SHARE ventes_share;
GRANT USAGE ON SCHEMA sales.public TO SHARE ventes_share;
GRANT SELECT ON TABLE sales.public.ventes TO SHARE ventes_share;
ALTER SHARE ventes_share ADD ACCOUNTS = consumer_acct;
```

!!! danger "Pièges examen"
    - Les objets partagés sont en **lecture seule** côté consommateur.
    - Le consommateur **ne peut pas re-partager** ni accorder des droits dessus.
    - Le **provider** garde le contrôle total ; révocation = effet immédiat.
    - Pas de coût de stockage côté consommateur ; il paie son **propre compute**.

## Direct shares, listings, clean rooms

- **Direct share** : partage direct compte-à-compte.
- **Sharing & resharing** : selon les politiques du listing.
- **Data Clean Rooms** : collaboration sur données sensibles sans exposer les lignes brutes.

📎 *Réf. : `docs.snowflake.com/en/user-guide/data-sharing-intro`*
