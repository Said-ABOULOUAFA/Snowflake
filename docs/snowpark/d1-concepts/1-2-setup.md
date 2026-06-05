# 1.2 -- Installation et Setup Snowpark

> **Domaine D1 -- 15% du SPS-C01**

## Installation STAR

```bash
# Version recommandee
pip install snowflake-snowpark-python==1.22.1

# Verifier la version installee
python -c "import snowflake.snowpark; print(snowflake.snowpark.__version__)"
```

## Environnements de developpement

| Environnement | Description |
|---|---|
| **Snowflake Notebooks** | Natif dans Snowsight -- pas d'installation locale |
| **Jupyter Notebooks** | Python local avec connexion Snowflake |
| **VS Code** | Extension Snowflake + snowpark-python |
| **IntelliJ/PyCharm** | Plugin Snowflake |
| **Outils tiers** | Tout IDE supportant Python |

## Creer une Session Snowpark STAR

```python
from snowflake.snowpark import Session

# Methode 1 : dictionnaire de connexion
connection_params = {
    "account":   "mon_compte.eu-west-1",
    "user":      "mon_user",
    "password":  "mon_password",
    "warehouse": "wh_dev",
    "database":  "ma_db",
    "schema":    "public",
    "role":      "analyst"
}
session = Session.builder.configs(connection_params).create()

# Methode 2 : Key-pair authentication (recommandee pour CI/CD)
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.serialization import load_pem_private_key

with open("rsa_key.p8", "rb") as key_file:
    private_key = load_pem_private_key(key_file.read(), password=None,
                                        backend=default_backend())

session = Session.builder.configs({
    "account":     "mon_compte.eu-west-1",
    "user":        "mon_service_account",
    "private_key": private_key,
    "warehouse":   "wh_dev",
    "database":    "ma_db",
    "schema":      "public"
}).create()

# Methode 3 : Snowflake CLI ou .env
# Utiliser snowflake.toml ou variables d'environnement
```

!!! danger "Question officielle SPS-C01"
    **Q : Comment reduire les alertes MFA constantes pour les utilisateurs d'une app Snowpark ?**
    **R : Mettre ALLOW_CLIENT_MFA_CACHING = TRUE** (reponse B officielle)

## SessionBuilder et methodes de Session STAR

```python
# Methodes utiles de Session
session.get_current_role()       # role actif
session.get_current_warehouse()  # warehouse actif
session.get_current_database()   # base active
session.get_current_schema()     # schema actif

# Modifier le contexte
session.use_role("analyst")
session.use_warehouse("wh_large")
session.use_database("autre_db")
session.use_schema("autre_schema")

# Executer du SQL brut
result = session.sql("SELECT CURRENT_VERSION()").collect()

# AsyncJob (execution asynchrone)
job = session.sql("SELECT COUNT(*) FROM large_table").collect_nowait()
result = job.result()  # attend la completion
```

## Environment Python recommande

```bash
# Creer un environnement virtuel dedie
python -m venv snowpark_env
source snowpark_env/bin/activate  # Linux/Mac
snowpark_env\\Scripts\\activate   # Windows

# Installer les dependances
pip install snowflake-snowpark-python pandas numpy scikit-learn
```
