# 5.2 — Data Sharing

> **Domaine D5 — 10% du COF-C03**

## Secure Data Sharing ⭐

Partager des données **sans copie** — accès direct aux données du fournisseur.

```sql
-- FOURNISSEUR : créer un partage
CREATE SHARE partage_ventes;
GRANT USAGE ON DATABASE sales_db TO SHARE partage_ventes;
GRANT USAGE ON SCHEMA sales_db.public TO SHARE partage_ventes;
GRANT SELECT ON TABLE sales_db.public.ventes TO SHARE partage_ventes;
ALTER SHARE partage_ventes ADD ACCOUNTS = 'abc12345.eu-west-1.aws';

-- CONSOMMATEUR : créer une base depuis le partage
CREATE DATABASE ventes_partagees FROM SHARE fournisseur.partage_ventes;
```

## Caractéristiques clés ⭐

| Caractéristique | Valeur |
|---|---|
| Copie des données | ❌ Aucune |
| Accès consommateur | Lecture seule |
| Coût calcul | Payé par le **consommateur** |
| Coût stockage | Payé par le **fournisseur** |
| Latence | Temps réel |

!!! danger "Question fréquente"
    Le consommateur d'un partage **ne peut PAS** accorder des droits sur les objets partagés à d'autres.

## Objets partageables vs non-partageables

| Partageable | Non partageable |
|---|---|
| Tables ✅ | Tables externes ❌ |
| Vues sécurisées ✅ | Stages ❌ |
| Vues matérialisées sécurisées ✅ | |
| UDFs sécurisées ✅ | |

## Vues sécurisées — Partager avec filtrage

```sql
CREATE SECURE VIEW vue_ventes_publiques AS
SELECT id, date_vente, montant, region
FROM ventes WHERE statut = 'PUBLIÉ';

GRANT SELECT ON VIEW vue_ventes_publiques TO SHARE mon_partage;
```

## Reader Accounts ⭐

Pour les partenaires **sans compte Snowflake**.

```sql
CREATE MANAGED ACCOUNT lecteur_externe
  ADMIN_NAME = 'admin' ADMIN_PASSWORD = 'Pwd123!' TYPE = READER;
ALTER SHARE mon_partage ADD ACCOUNTS = lecteur_externe;
SHOW MANAGED ACCOUNTS;
```

!!! warning "Reader Account"
    - Géré et **facturé par le fournisseur**
    - Utilise les **warehouses du fournisseur**
    - Fonctionnalités limitées (lecture seule)
