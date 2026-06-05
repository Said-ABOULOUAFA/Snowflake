# 5.3 Marketplace & listings

> **Domain 5.0 — Data Collaboration (10%)**

## Snowflake Marketplace ⭐

Place de marché de **données** et d'**applications** entre comptes Snowflake.

| Type de listing | Visibilité |
|---|---|
| **Private listing** | Ciblé sur des comptes/organisations précis |
| **Public listing** | Visible sur la Marketplace publique |

- Repose sur le **Secure Data Sharing** (pas de copie de données).
- Le consommateur monte le share comme une base et requête normalement.

## Native Apps ⭐

Le **Snowflake Native App Framework** permet de distribuer une **application** (logique + données + UI Streamlit) qui s'exécute **dans le compte du consommateur**.

| Concept | Description |
|---|---|
| **Application package** | Paquet versionné publié par le provider |
| **Listing** | Distribution via Marketplace (privé/public) |
| **Exécution** | Code et données dans le compte **du consommateur** |

!!! tip "Différence clé"
    - **Data Sharing / Marketplace** → partage de **données**.
    - **Native Apps** → distribution d'une **application** (avec logique métier) exécutée chez le consommateur.

## Reader accounts (rappel)

Pour un consommateur **sans** compte Snowflake : le provider crée un **reader account** (qu'il gère et facture).

📎 *Réf. : `docs.snowflake.com/en/user-guide/data-marketplace`*
