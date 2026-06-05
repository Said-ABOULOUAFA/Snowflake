# 5.3 — Snowflake Marketplace & Listings

> **Domaine D5 — 10% du COF-C03**

## Snowflake Marketplace ⭐

Place de marché pour **partager, acheter ou accéder** à des données.

```sql
-- Listing privé (partage B2B ciblé)
CREATE LISTING mon_listing_prive
  FOR SHARE mon_partage AS $$
    title: 'Données Météo France'
    description: 'Données météo historiques'
  $$;

ALTER LISTING mon_listing_prive ADD ACCOUNTS = ('compte_partenaire');

-- Voir les listings disponibles
SHOW LISTINGS;
```

| Type de listing | Visibilité |
|---|---|
| **Privé** | Comptes invités uniquement |
| **Public** | Tout le Marketplace |

## Native Apps Framework ⭐

Créer et distribuer des **applications Snowflake** via le Marketplace.

```sql
CREATE APPLICATION PACKAGE mon_app_package;

ALTER APPLICATION PACKAGE mon_app_package
  ADD VERSION v1_0 USING @mon_stage/app/;

-- Consommateur
CREATE APPLICATION mon_app FROM APPLICATION PACKAGE mon_app_package
  USING VERSION v1_0;
```

## Data Clean Rooms ⭐

Analyser des données croisées **sans exposer les données brutes**.

- Chaque partie voit uniquement les **résultats agrégés**
- Les données brutes ne sont jamais exposées
- **Differential privacy** disponible pour masquer les petits groupes
- Utilisé pour : marketing croisé, analyses partenaires, compliance

## Réplication Snowgrid ⭐

Pour partager entre régions ou clouds différents.

```sql
ALTER DATABASE ma_db ENABLE REPLICATION TO ACCOUNTS aws.eu-west-1.compte_cible;
CREATE DATABASE ma_db_replica AS REPLICA OF source_account.ma_db;
ALTER DATABASE ma_db_replica REFRESH;
ALTER DATABASE ma_db_replica PRIMARY;  -- failover
```
