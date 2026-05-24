# Rôles & Privilèges (RBAC)

## Hiérarchie des rôles système ⭐

```
ACCOUNTADMIN
    └── SYSADMIN
            └── (rôles personnalisés)
    └── SECURITYADMIN
            └── USERADMIN
    └── PUBLIC (tous les utilisateurs)
```

| Rôle | Pouvoirs clés |
|---|---|
| **ACCOUNTADMIN** | Tout. Gestion compte, facturation, réplication. À utiliser avec MFA obligatoire. |
| **SYSADMIN** | Crée et gère warehouses, bases de données, schémas, tables. |
| **SECURITYADMIN** | Gère les rôles et les utilisateurs. Peut faire GRANT sur tous les objets. |
| **USERADMIN** | Crée et gère les utilisateurs et rôles uniquement. |
| **PUBLIC** | Accordé automatiquement à tous. Accès minimal. |

!!! danger "Piège exam fréquent"
    `ACCOUNTADMIN` ne doit **jamais** être utilisé pour le travail quotidien. Créer des rôles fonctionnels séparés.

---

## Grants — Syntaxe à connaître

```sql
-- Donner un privilège sur un objet
GRANT SELECT ON TABLE ma_table TO ROLE analyst;
GRANT USAGE ON WAREHOUSE wh_bi TO ROLE analyst;
GRANT USAGE ON DATABASE ma_db TO ROLE analyst;
GRANT USAGE ON SCHEMA ma_db.public TO ROLE analyst;

-- Donner un rôle à un utilisateur
GRANT ROLE analyst TO USER said;

-- Donner un rôle à un autre rôle (héritage)
GRANT ROLE analyst TO ROLE data_scientist;

-- Révoquer
REVOKE SELECT ON TABLE ma_table FROM ROLE analyst;
```

---

## Privilèges sur objets clonés ⭐

!!! warning "Question très fréquente"
    Quand tu **clones** un objet :
    - Les **privilèges de l'objet source** ne sont **PAS** copiés sur le clone
    - Le propriétaire du clone = le rôle qui a exécuté le CLONE
    - Les **privilèges accordés sur les sous-objets** (ex: tables dans un schéma cloné) sont copiés

```sql
-- Clone d'une base de données
CREATE DATABASE ma_db_dev CLONE ma_db_prod;
-- ⚠️ Les GRANTs sur ma_db_prod ne s'appliquent PAS à ma_db_dev
```

---

## Privilèges sur objets partagés

!!! warning "Question très fréquente"
    - Les objets d'un **partage (Share)** sont en **lecture seule**
    - Le consommateur **ne peut pas** accorder des privilèges à d'autres sur les objets partagés
    - Le fournisseur contrôle entièrement l'accès

---

## Principe du moindre privilège

```sql
-- Bonne pratique : rôle fonctionnel
CREATE ROLE analyst_ventes;
GRANT USAGE ON DATABASE sales_db TO ROLE analyst_ventes;
GRANT USAGE ON SCHEMA sales_db.public TO ROLE analyst_ventes;
GRANT SELECT ON ALL TABLES IN SCHEMA sales_db.public TO ROLE analyst_ventes;
GRANT USAGE ON WAREHOUSE wh_bi TO ROLE analyst_ventes;

-- Assigner à un utilisateur
GRANT ROLE analyst_ventes TO USER said;
```
