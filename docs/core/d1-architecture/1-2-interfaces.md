# 1.2 — Interfaces & Outils Snowflake

> **Domaine D1 — 31% du COF-C03**

## Snowsight — Interface Web ⭐

Interface principale de Snowflake, accessible via navigateur.

**Fonctionnalités clés :**
- Éditeur SQL avec autocomplétion et historique des requêtes
- **Query Profile** — analyse visuelle des plans d'exécution
- **Worksheets** — espaces de travail collaboratifs
- **Dashboards** — visualisations intégrées
- **Data Governance** — Data Lineage, Trust Center
- **Monitoring** — Task History, Pipe Status, Warehouse usage
- **Snowflake Marketplace** — accès aux données partagées

```sql
-- Changer de rôle dans Snowsight (ou SQL)
USE ROLE analyst;
USE WAREHOUSE wh_bi;
USE DATABASE ma_db;
USE SCHEMA public;
```

---

## Snowflake CLI ⭐

Interface en ligne de commande pour automatiser et scripter.

```bash
# Installation
pip install snowflake-cli-labs

# Connexion
snow connection add

# Exécuter une requête
snow sql -q "SELECT CURRENT_VERSION()"

# Déployer une fonction Snowpark
snow snowpark deploy

# Uploader des fichiers (stage)
snow stage copy ./data.csv @mon_stage/
```

!!! info "Usages typiques du CLI"
    - CI/CD pipelines
    - Déploiement de Snowpark (UDFs, procédures)
    - Scripts d'administration automatisés
    - Migrations de schémas

---

## IDE Integrations

Snowflake s'intègre avec les principaux environnements de développement :

| Outil | Type d'intégration |
|---|---|
| **Visual Studio Code** | Extension Snowflake + Snowpark support |
| **IntelliJ / PyCharm** | Connecteur JDBC |
| **Jupyter Notebooks** | Python Connector + Snowpark |
| **Snowflake Notebooks** | Natif dans Snowsight |

### VS Code — Extension Snowflake

```json
// .vscode/settings.json
{
  "snowflake.account": "mon_compte.eu-west-1",
  "snowflake.username": "mon_user",
  "snowflake.authenticator": "externalbrowser"
}
```

---

## Hiérarchie des objets Snowflake ⭐

```
Organisation
    └── Compte (Account)
            ├── Utilisateurs & Rôles
            ├── Warehouses
            └── Bases de données (Database)
                    └── Schémas (Schema)
                            ├── Tables
                            ├── Vues
                            ├── Stages
                            ├── File Formats
                            ├── Pipes
                            ├── Streams
                            ├── Tasks
                            ├── UDFs
                            ├── Procédures stockées
                            ├── Sequences
                            ├── Shares
                            ├── ML Models
                            └── Applications
```

### Session & Variables de contexte ⭐

```sql
-- Voir les paramètres actifs
SELECT CURRENT_ROLE(), CURRENT_WAREHOUSE(), 
       CURRENT_DATABASE(), CURRENT_SCHEMA();

-- Paramètres de session
SHOW PARAMETERS;
SHOW PARAMETERS LIKE 'TIMEZONE';

-- Modifier un paramètre de session
ALTER SESSION SET TIMEZONE = 'Europe/Paris';
ALTER SESSION SET QUERY_TAG = 'mon_pipeline_etl';

-- Hiérarchie de précédence des paramètres
-- Session > Utilisateur > Rôle > Compte
```

!!! tip "Précédence des paramètres"
    Un paramètre défini au niveau **Session** écrase celui défini au niveau **Utilisateur**, qui écrase **Rôle**, qui écrase **Compte**. C'est la hiérarchie officielle COF-C03.
