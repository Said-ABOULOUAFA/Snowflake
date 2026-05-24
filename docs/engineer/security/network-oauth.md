# Network Policies, OAuth & Authentification avancée

## Network Policies ⭐

Restreint les connexions à Snowflake par adresse IP.

```sql
-- Créer une politique réseau
CREATE NETWORK POLICY np_bureau
  ALLOWED_IP_LIST   = ('203.0.113.0/24', '198.51.100.10')
  BLOCKED_IP_LIST   = ('198.51.100.99')
  COMMENT           = 'Accès bureau uniquement';

-- Appliquer au compte entier
ALTER ACCOUNT SET NETWORK_POLICY = np_bureau;

-- Appliquer à un utilisateur spécifique
ALTER USER said SET NETWORK_POLICY = np_bureau;

-- Voir les politiques réseau
SHOW NETWORK POLICIES;

-- Supprimer
ALTER ACCOUNT UNSET NETWORK_POLICY;
```

!!! warning "Priorité"
    La politique d'un **utilisateur** est prioritaire sur celle du **compte**.

---

## Network Rules (nouvelles) ⭐

Plus flexibles que les Network Policies — supportent les ranges IP, VPC, Private Link.

```sql
-- Règle pour les IPs du bureau
CREATE NETWORK RULE nr_bureau
  TYPE       = IPV4
  VALUE_LIST = ('203.0.113.0/24', '10.0.0.0/8')
  MODE       = INGRESS;

-- Règle pour AWS VPC
CREATE NETWORK RULE nr_vpc_aws
  TYPE       = AWSVPCEID
  VALUE_LIST = ('vpce-0a1b2c3d4e5f6a7b8')
  MODE       = INGRESS;

-- Créer une politique à partir des règles
CREATE NETWORK POLICY np_avancee
  ALLOWED_NETWORK_RULE_LIST = ('nr_bureau', 'nr_vpc_aws');

ALTER ACCOUNT SET NETWORK_POLICY = np_avancee;
```

---

## OAuth ⭐

Permet aux applications tierces d'accéder à Snowflake sans partager les credentials.

```sql
-- OAuth pour Tableau / Power BI (Snowflake OAuth)
CREATE SECURITY INTEGRATION oauth_tableau
  TYPE                         = OAUTH
  OAUTH_CLIENT                 = TABLEAU_DESKTOP
  ENABLED                      = TRUE
  OAUTH_ISSUE_REFRESH_TOKENS   = TRUE
  OAUTH_REFRESH_TOKEN_VALIDITY = 86400;  -- 24h en secondes

-- OAuth personnalisé (External OAuth)
CREATE SECURITY INTEGRATION oauth_okta
  TYPE                          = EXTERNAL_OAUTH
  ENABLED                       = TRUE
  EXTERNAL_OAUTH_TYPE           = OKTA
  EXTERNAL_OAUTH_ISSUER         = 'https://mon-compte.okta.com'
  EXTERNAL_OAUTH_JWS_KEYS_URL   = 'https://mon-compte.okta.com/oauth2/v1/keys'
  EXTERNAL_OAUTH_AUDIENCE_LIST  = ('https://mon_compte.snowflakecomputing.com');
```

---

## Authentification par paire de clés ⭐

Alternative sécurisée au mot de passe — obligatoire pour les connexions programmatiques.

```bash
# Générer une paire de clés RSA
openssl genrsa 2048 | openssl pkcs8 -topk8 -inform PEM -out rsa_key.p8 -nocrypt
openssl rsa -in rsa_key.p8 -pubout -out rsa_key.pub
```

```sql
-- Assigner la clé publique à l'utilisateur
ALTER USER mon_service_account
  SET RSA_PUBLIC_KEY = 'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ...';

-- Rotation de clé (zéro downtime)
ALTER USER mon_service_account
  SET RSA_PUBLIC_KEY_2 = 'NOUVELLE_CLE_PUBLIQUE...';
-- Une fois la nouvelle clé testée :
ALTER USER mon_service_account UNSET RSA_PUBLIC_KEY;
```

```python
# Connexion avec clé privée (Python)
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.serialization import load_pem_private_key
import snowflake.connector

with open("rsa_key.p8", "rb") as key_file:
    p_key = load_pem_private_key(key_file.read(), password=None,
                                  backend=default_backend())

conn = snowflake.connector.connect(
    account    = 'mon_compte',
    user       = 'mon_service_account',
    private_key = p_key
)
```

---

## SSO / Federated Authentication ⭐

```sql
-- Configurer SAML2 avec Azure AD
CREATE SECURITY INTEGRATION sso_azure
  TYPE                          = SAML2
  ENABLED                       = TRUE
  SAML2_ISSUER                  = 'https://sts.windows.net/tenant-id/'
  SAML2_SSO_URL                 = 'https://login.microsoftonline.com/tenant-id/saml2'
  SAML2_PROVIDER                = 'CUSTOM'
  SAML2_X509_CERT               = 'MIIC...'
  SAML2_ENABLE_SP_INITIATED     = TRUE;

-- Voir les intégrations de sécurité
SHOW SECURITY INTEGRATIONS;
```

---

## Récapitulatif méthodes d'authentification ⭐

| Méthode | Cas d'usage | Sécurité |
|---|---|---|
| **Login/Password** | Utilisateurs humains (dev) | Faible |
| **MFA** | Tous les utilisateurs humains | Bonne |
| **SSO/SAML2** | Entreprises avec IdP (Okta, Azure AD) | Très bonne |
| **Key Pair** | Services, automatisation, CI/CD | Très bonne |
| **OAuth** | Applications tierces (Tableau, Power BI) | Bonne |
| **PrivateLink** | Connexion sans Internet public | Maximale |
