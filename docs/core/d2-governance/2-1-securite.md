# 2.1 Modèle de sécurité & principes (RBAC)

> **Domain 2.0 — Account Management & Data Governance (20%)**

## RBAC — hiérarchie des rôles système ⭐

```
ACCOUNTADMIN
   ├── SYSADMIN ──── (rôles fonctionnels personnalisés)
   ├── SECURITYADMIN ──── USERADMIN
   └── PUBLIC (accordé à tous)
```

| Rôle | Pouvoirs |
|---|---|
| **ACCOUNTADMIN** | Tout : compte, facturation, réplication. **MFA obligatoire**, jamais pour l'usage quotidien |
| **SYSADMIN** | Crée/gère warehouses, bases, schémas, tables |
| **SECURITYADMIN** | Gère rôles et users ; GRANT sur tous objets |
| **USERADMIN** | Crée users et rôles uniquement |
| **PUBLIC** | Accès minimal, accordé automatiquement |

!!! danger "Piège exam"
    `ACCOUNTADMIN` ne doit **jamais** servir au travail courant. Créer des rôles fonctionnels (moindre privilège).

## DAC & rôles

- **RBAC** : privilèges accordés aux rôles, rôles accordés aux users/autres rôles.
- **DAC** (Discretionary Access Control) : chaque objet a un **owner** qui peut accorder l'accès.
- **Account roles** vs **Database roles** vs **Custom roles** vs **Secondary roles** (cumul de rôles actifs).

```sql
GRANT USAGE ON WAREHOUSE wh_bi TO ROLE analyst;
GRANT USAGE ON DATABASE sales TO ROLE analyst;
GRANT USAGE ON SCHEMA sales.public TO ROLE analyst;
GRANT SELECT ON ALL TABLES IN SCHEMA sales.public TO ROLE analyst;
GRANT ROLE analyst TO USER said;
GRANT ROLE analyst TO ROLE data_scientist;   -- héritage
```

## Authentification

| Méthode | Note |
|---|---|
| **MFA** | Recommandé/obligatoire pour ACCOUNTADMIN |
| **Federated / SSO** | SAML 2.0 |
| **OAuth** | Snowflake OAuth / External OAuth |
| **Key-pair** | Authentification par clé (services, drivers) |

## Network Policies

```sql
CREATE NETWORK POLICY np_corp
  ALLOWED_IP_LIST = ('192.168.1.0/24')
  BLOCKED_IP_LIST = ('192.168.1.99');
ALTER ACCOUNT SET NETWORK_POLICY = np_corp;
```

- **Logging & tracing** : `ACCESS_HISTORY`, `LOGIN_HISTORY`, event tables.

📎 *Réf. : `docs.snowflake.com/en/user-guide/security-access-control-overview`*
