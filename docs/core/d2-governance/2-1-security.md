# 2.1 — Modèle de sécurité & RBAC

> **Domaine D2 — 20% du COF-C03**

## RBAC vs DAC ⭐

| Modèle | Signification | Principe |
|---|---|---|
| **RBAC** | Role-Based Access Control | Les droits sont attribués aux **rôles**, les rôles aux utilisateurs |
| **DAC** | Discretionary Access Control | Le **propriétaire** d'un objet peut accorder des droits dessus |

!!! danger "Question fréquente COF-C03"
    **Q : Quel modèle permet au propriétaire d'un objet d'accorder l'accès à cet objet ?**
    **R : DAC (Discretionary Access Control)** → réponse A dans les sample questions officielles

Snowflake utilise **les deux** en combinaison :
- RBAC = organisation hiérarchique des rôles
- DAC = les propriétaires peuvent faire GRANT sur leurs objets

---

## Hiérarchie des rôles système ⭐

```
ORGADMIN
    │
ACCOUNTADMIN ◄── rôle le plus puissant
    ├── SYSADMIN
    │       └── (rôles fonctionnels)
    ├── SECURITYADMIN
    │       └── USERADMIN
    └── PUBLIC (tous les utilisateurs)
```

| Rôle | Pouvoirs clés |
|---|---|
| **ACCOUNTADMIN** | Tout : compte, facturation, réplication, sécurité |
| **SYSADMIN** | Créer/gérer warehouses, bases, schémas, tables |
| **SECURITYADMIN** | Gérer rôles, utilisateurs, GRANT sur tous objets |
| **USERADMIN** | Créer/gérer utilisateurs et rôles uniquement |
| **PUBLIC** | Accordé automatiquement à tous, accès minimal |

!!! danger "Bonne pratique — exam"
    `ACCOUNTADMIN` ne doit **jamais** être utilisé pour le travail quotidien. Toujours créer des **rôles fonctionnels** dédiés.

---

## Rôles fonctionnels ⭐

### Account Roles (rôles de compte)

```sql
-- Créer un rôle fonctionnel
CREATE ROLE analyst_ventes;

-- Accorder des privilèges
GRANT USAGE ON DATABASE sales_db TO ROLE analyst_ventes;
GRANT USAGE ON SCHEMA sales_db.public TO ROLE analyst_ventes;
GRANT SELECT ON ALL TABLES IN SCHEMA sales_db.public TO ROLE analyst_ventes;
GRANT USAGE ON WAREHOUSE wh_bi TO ROLE analyst_ventes;

-- Accorder à un utilisateur
GRANT ROLE analyst_ventes TO USER said;

-- Héritage de rôle
GRANT ROLE analyst_ventes TO ROLE senior_analyst;
```

### Database Roles (nouveau COF-C03) ⭐

Rôles scopés à une base de données — idéaux pour le Data Sharing.

```sql
-- Créer un rôle de base de données
CREATE DATABASE ROLE ma_db.role_lecture;

-- Accorder des privilèges
GRANT SELECT ON ALL TABLES IN SCHEMA ma_db.public
  TO DATABASE ROLE ma_db.role_lecture;

-- Accorder le rôle DB à un rôle de compte
GRANT DATABASE ROLE ma_db.role_lecture TO ROLE analyst;
```

---

## Secondary Roles ⭐

Permettent d'activer **plusieurs rôles simultanément** dans une session.

```sql
ALTER SESSION SET SECONDARY_ROLES = ALL;   -- tous les rôles de l'utilisateur
ALTER SESSION SET SECONDARY_ROLES = NONE;  -- rôle principal uniquement

SELECT CURRENT_ROLE(), CURRENT_SECONDARY_ROLES();
```

---

## Authentification ⭐

| Méthode | Description | Sécurité |
|---|---|---|
| **Login/Password** | Par défaut | Faible |
| **MFA** | Multi-Factor Authentication (Duo) | Bonne |
| **Federated Auth / SSO** | SAML 2.0 (Okta, Azure AD) | Très bonne |
| **OAuth** | Pour apps tierces (Tableau, Power BI) | Bonne |
| **Key-pair** | RSA (scripts, CI/CD, services) | Très bonne |

```sql
-- Assigner une clé RSA publique à un utilisateur
ALTER USER mon_service SET RSA_PUBLIC_KEY = 'MIIBIj...';

-- Rotation de clé (zéro downtime)
ALTER USER mon_service SET RSA_PUBLIC_KEY_2 = 'NOUVELLE_CLE...';
-- Après validation :
ALTER USER mon_service UNSET RSA_PUBLIC_KEY;
```

---

## Network Policies ⭐

```sql
-- Créer une politique réseau
CREATE NETWORK POLICY np_bureau
  ALLOWED_IP_LIST = ('203.0.113.0/24', '10.0.0.10')
  BLOCKED_IP_LIST = ('198.51.100.99');

-- Appliquer au compte
ALTER ACCOUNT SET NETWORK_POLICY = np_bureau;

-- Appliquer à un utilisateur (priorité sur le compte)
ALTER USER said SET NETWORK_POLICY = np_bureau;
```

!!! warning "Priorité : utilisateur > compte"
    La politique définie sur un **utilisateur** est prioritaire sur celle du **compte**.

---

## Securable Object Hierarchy

La hiérarchie des objets sécurisables suit l'ordre : **Organisation → Compte → Base → Schéma → Objet**.

```sql
-- Syntaxe générale d'un GRANT
GRANT <privilege> ON <object_type> <object_name> TO ROLE <role_name>;

-- Exemples
GRANT CREATE TABLE ON SCHEMA ma_db.public TO ROLE developer;
GRANT OPERATE ON WAREHOUSE wh_etl TO ROLE etl_role;
GRANT MONITOR ON WAREHOUSE wh_etl TO ROLE etl_role;

-- Voir les droits accordés
SHOW GRANTS TO ROLE analyst_ventes;
SHOW GRANTS ON TABLE ma_table;
```

---

## Account Identifiers ⭐

```sql
-- Format standard
-- <account_name>.<region>.<cloud>
-- Exemple : myaccount.eu-west-1.aws

SELECT CURRENT_ACCOUNT();         -- identifiant du compte
SELECT CURRENT_ORGANIZATION_NAME(); -- organisation
SELECT CURRENT_REGION();           -- région cloud
```

---

## Logging & Tracing ⭐

```sql
-- Activer les logs sur un objet
ALTER PROCEDURE ma_proc SET LOG_LEVEL = DEBUG;
ALTER FUNCTION ma_func SET TRACE_LEVEL = ALWAYS;

-- Créer une table d'événements
CREATE EVENT TABLE ma_db.public.my_event_table;
ALTER ACCOUNT SET EVENT_TABLE = ma_db.public.my_event_table;

-- Consulter les logs
SELECT * FROM ma_db.public.my_event_table
WHERE record_type = 'LOG'
ORDER BY timestamp DESC;
```
