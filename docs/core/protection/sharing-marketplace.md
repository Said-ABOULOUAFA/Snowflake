# Data Sharing & Marketplace

## Secure Data Sharing ⭐

Partager des données **sans copier** — le consommateur accède directement aux données du fournisseur.

```sql
-- FOURNISSEUR : créer un partage
CREATE SHARE partage_ventes;
GRANT USAGE ON DATABASE sales_db TO SHARE partage_ventes;
GRANT USAGE ON SCHEMA sales_db.public TO SHARE partage_ventes;
GRANT SELECT ON TABLE sales_db.public.ventes TO SHARE partage_ventes;

-- Ajouter un compte consommateur
ALTER SHARE partage_ventes ADD ACCOUNTS = 'abc12345.eu-west-1.aws';

-- CONSOMMATEUR : créer une base depuis le partage
CREATE DATABASE ventes_partagees FROM SHARE fournisseur.partage_ventes;
```

---

## Caractéristiques clés ⭐

| Caractéristique | Détail |
|---|---|
| **Copie des données** | ❌ Aucune — accès direct |
| **Accès consommateur** | Lecture seule uniquement |
| **Coût calcul** | Payé par le **consommateur** |
| **Coût stockage** | Payé par le **fournisseur** |
| **Latence** | Temps réel |
| **Cross-cloud** | ✅ Via Snowgrid (réplication) |

!!! danger "Piège exam"
    Le consommateur d'un partage **ne peut pas** accorder des privilèges sur les objets partagés à d'autres rôles/utilisateurs.

---

## Objets partageables

✅ Tables
✅ Vues sécurisées (Secure Views)
✅ Vues matérialisées sécurisées
✅ UDFs sécurisées
❌ Tables externes
❌ Stages

---

## Secure Views — Partager avec filtrage

```sql
-- Créer une vue sécurisée (le DDL est masqué au consommateur)
CREATE SECURE VIEW vue_ventes_publiques AS
SELECT id, date_vente, montant, region
FROM ventes
WHERE statut = 'PUBLIÉ';  -- filtre appliqué avant partage

-- Partager la vue, pas la table source
GRANT SELECT ON VIEW vue_ventes_publiques TO SHARE mon_partage;
```

---

## Snowflake Marketplace

- Place de marché pour **partager ou acheter** des données
- **Annonces privées** : partage B2B entre comptes spécifiques
- **Annonces publiques** : visibles sur le Marketplace
- **Data Clean Rooms** : analyses sur données croisées sans accès direct

---

## Réplication inter-régions (Snowgrid)

Pour partager entre régions ou clouds différents, il faut d'abord répliquer :

```sql
-- Activer la réplication sur une base
ALTER DATABASE ma_db ENABLE REPLICATION TO ACCOUNTS aws.eu-west-1.compte_cible;

-- Créer un replica sur le compte cible
CREATE DATABASE ma_db_replica AS REPLICA OF source_account.ma_db;

-- Actualiser le replica
ALTER DATABASE ma_db_replica REFRESH;
```
