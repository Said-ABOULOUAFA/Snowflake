# Partage avancé — Reader Accounts, Native Apps, Clean Rooms

## Reader Accounts ⭐

Permet de partager des données avec des partenaires qui **n'ont pas de compte Snowflake**.

```sql
-- Créer un reader account
CREATE MANAGED ACCOUNT lecteur_externe
  ADMIN_NAME    = 'admin_externe'
  ADMIN_PASSWORD = 'MotDePasse123!'
  TYPE          = READER;

-- Ajouter ce reader account au partage
ALTER SHARE mon_partage ADD ACCOUNTS = lecteur_externe;

-- Voir les reader accounts
SHOW MANAGED ACCOUNTS;
```

!!! warning "Points clés"
    - Le reader account est **géré et facturé** par le fournisseur
    - Le consommateur utilise les **warehouses du fournisseur** pour les requêtes
    - Limité à la **lecture seule** des données partagées
    - Différent d'un compte Snowflake normal (fonctionnalités limitées)

---

## Native Apps Framework ⭐

Permet de créer et distribuer des **applications Snowflake** via le Marketplace.

```sql
-- Créer un package d'application
CREATE APPLICATION PACKAGE mon_app_package;

-- Créer une version
ALTER APPLICATION PACKAGE mon_app_package
  ADD VERSION v1_0
  USING @mon_stage/app/;

-- Installer une application (côté consommateur)
CREATE APPLICATION mon_app
  FROM APPLICATION PACKAGE mon_app_package
  USING VERSION v1_0;
```

!!! info "Cas d'usage"
    - Distribuer une application data sur le Marketplace
    - Monétiser des données ou des algorithmes
    - L'application s'exécute dans le compte du consommateur (sécurité)

---

## Data Clean Rooms ⭐

Permet d'analyser des données **croisées entre deux entreprises** sans qu'aucune ne voie les données brutes de l'autre.

**Exemple :** Une banque et un assureur veulent analyser leurs clients communs sans partager les données personnelles.

```sql
-- Via Snowflake Native App (Clean Room)
-- Le fournisseur crée la clean room
CREATE CLEANROOM ma_clean_room;

-- Définir les analyses autorisées (templates)
ALTER CLEANROOM ma_clean_room
  ADD DIFFERENTIAL_PRIVACY_ANALYSIS overlap_analysis
  AS $$
    SELECT COUNT(*) AS clients_communs
    FROM provider_table p
    JOIN consumer_table c ON p.email_hash = c.email_hash
  $$;
```

!!! info "Principe"
    - Chaque partie voit uniquement les **résultats agrégés**
    - Les données brutes ne sont jamais exposées
    - La **differential privacy** peut être activée pour masquer les petits groupes

---

## Listings — Privé vs Public

| | Listing Privé | Listing Public |
|---|---|---|
| **Visibilité** | Comptes invités uniquement | Tout le Marketplace |
| **Accès** | Sur invitation | Libre (ou sur demande) |
| **Cas d'usage** | Partage B2B ciblé | Distribution large |

```sql
-- Créer un listing privé
CREATE LISTING mon_listing_prive
  FOR SHARE mon_partage
  AS $$
    title: 'Données Météo France'
    description: 'Données météo historiques...'
    auto_fulfillment:
      refresh_schedule: '1 MINUTE'
      refresh_type: FULL_DATABASE
  $$;

-- Ajouter des comptes autorisés
ALTER LISTING mon_listing_prive ADD ACCOUNTS = ('compte_partenaire');
```
