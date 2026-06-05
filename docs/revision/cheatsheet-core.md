# 🎯 Cheat Sheet — COF-C03 (SnowPro Core)

> 100 questions · 115 min · score 750/1000 · 175 $ · validité 2 ans

## Poids des domaines
| Domaine | % |
|---|---|
| 1.0 AI Data Cloud & Architecture | 31 |
| 2.0 Account Management & Governance | 20 |
| 3.0 Data Loading/Unloading & Connectivity | 18 |
| 4.0 Performance, Querying & Transformation | 21 |
| 5.0 Data Collaboration | 10 |

## Architecture
- 3 couches : **Stockage** / **Compute (warehouses MPP)** / **Cloud Services**.
- Hybride : disque partagé (stockage central) **+** sans partage (compute MPP).
- 100 % cloud (AWS/Azure/GCP), **pas d'on-premise**.
- Warehouses **totalement indépendants** entre eux.

## Objets & éditions
- Hiérarchie : Organisation → Compte → Base → Schéma → Objets.
- Éditions : Standard / Enterprise (Time Travel 90 j, MV) / Business Critical (HIPAA, PrivateLink) / VPS.

## Tables & stockage
| Type | Time Travel | Fail-safe |
|---|---|---|
| PERMANENT | 0–90 j | 7 j |
| TRANSIENT | 0–1 j | aucun |
| TEMPORARY | 0–1 j (session) | aucun |
- Micro-partitions auto (~50–500 Mo), colonnaire, métadonnées min/max → **pruning**.

## Caching (3 niveaux) ⭐
1. **Result cache** (Cloud Services, 24 h, exact même requête).
2. **Local disk / warehouse cache** (données lues).
3. **Metadata cache**.

## Chargement
- **COPY INTO** = batch (ton warehouse). **Snowpipe** = continu, serverless.
- `VALIDATION_MODE`, `ON_ERROR`, `FILE FORMAT`, `PURGE`.

## Sécurité (RBAC)
- Rôles système : ORGADMIN, ACCOUNTADMIN, SECURITYADMIN, USERADMIN, SYSADMIN, PUBLIC.
- Privilèges hérités via hiérarchie de rôles ; **clé : un clone n'hérite PAS des privilèges**.

## Collaboration
- **Secure Data Sharing** : pas de copie, partage des métadonnées (même région/cloud sinon réplication).
- **Marketplace** & **Listings**, **Reader Accounts** pour non-clients.
