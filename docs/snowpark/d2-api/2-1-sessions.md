# 2.1 — Sessions & Authentification

> **Domaine D2 API Python — 30% du SPS-C01**

## Account Identifiers ⭐

```python
# Format des account identifiers
# <account_name>.<region>.<cloud>
# Exemples :
# myaccount.eu-west-1.aws
# myaccount.westeurope.azure

# Vérifier dans Snowflake
session.sql("SELECT CURRENT_ACCOUNT()").collect()
session.sql("SELECT CURRENT_ORGANIZATION_NAME()").collect()
```

## Méthodes d'authentification ⭐

```python
# 1. Login/Password
params = {"account": "mon_compte.eu-west-1", "user": "user", "password": "pwd"}

# 2. Key-pair authentication (recommandée en prod)
params = {
    "account":     "mon_compte.eu-west-1",
    "user":        "service_account",
    "private_key": private_key_object,  # chargé depuis fichier .p8
}

# 3. SSO/Browser (développement interactif)
params = {
    "account":       "mon_compte.eu-west-1",
    "user":          "mon_user",
    "authenticator": "externalbrowser"
}

# 4. Snowflake CLI / .env file
# Utiliser connexion_name dans snowflake.toml
session = Session.builder.config("connection_name", "ma_connexion").create()

# 5. MFA caching (réduire les alertes MFA)
# ALTER ACCOUNT SET ALLOW_CLIENT_MFA_CACHING = TRUE;
```

## Session Methods ⭐

```python
# Attributs de session
print(session.get_current_account())
print(session.get_current_role())
print(session.get_current_warehouse())
print(session.get_current_database())
print(session.get_current_schema())

# Changer le contexte
session.use_role("analyst")
session.use_warehouse("wh_large")

# Exécuter du SQL
rows = session.sql("SELECT CURRENT_VERSION()").collect()
df   = session.sql("SELECT * FROM ma_table WHERE id = 1")

# Ajouter des imports pour les UDFs
session.add_import("@mon_stage/ma_lib.zip")
session.add_packages("pandas", "numpy")

# Fermer la session
session.close()
```

## AsyncJob ⭐

```python
# Exécution asynchrone (non-bloquante)
job = session.sql("SELECT COUNT(*) FROM large_table").collect_nowait()

# Vérifier si terminé
while not job.is_done():
    print("En cours...")
    import time; time.sleep(1)

# Récupérer le résultat
result = job.result()

# Ou bloquer jusqu'à la fin
result = session.sql("SELECT ...").collect(block=True)   # défaut
result = session.sql("SELECT ...").collect(block=False)  # async
```
