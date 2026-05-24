# Authentification & Réseau

## Méthodes d'authentification

| Méthode | Description |
|---|---|
| **Login/Password** | Par défaut |
| **MFA** | Multi-Factor Authentication via Duo Security |
| **SSO / SAML 2.0** | Intégration avec Okta, Azure AD, etc. |
| **Key Pair** | Clé RSA (utilisée pour Snowflake CLI, drivers) |
| **OAuth** | Pour applications tierces |

!!! tip "Bonne pratique"
    `ACCOUNTADMIN` doit **toujours** utiliser MFA.

---

## Network Policies

Restreint les connexions à Snowflake par adresse IP.

```sql
-- Créer une politique réseau
CREATE NETWORK POLICY np_bureau
  ALLOWED_IP_LIST   = ('203.0.113.0/24', '198.51.100.10')
  BLOCKED_IP_LIST   = ('198.51.100.99');

-- Appliquer au compte entier
ALTER ACCOUNT SET NETWORK_POLICY = np_bureau;

-- Appliquer à un utilisateur spécifique
ALTER USER said SET NETWORK_POLICY = np_bureau;
```

!!! warning "Priorité"
    La politique d'un **utilisateur** est prioritaire sur celle du **compte**.

---

## Private Connectivity (Business Critical+)

| Service | Cloud |
|---|---|
| **AWS PrivateLink** | AWS |
| **Azure Private Link** | Azure |
| **Google Cloud Private Service Connect** | GCP |

Permet de se connecter à Snowflake **sans passer par Internet public**.
