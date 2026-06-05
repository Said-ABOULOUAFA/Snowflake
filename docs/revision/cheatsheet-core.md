# Cheat Sheet COF-C03 — SnowPro Core

## Architecture (31%)

| Élément | À retenir |
|---|---|
| 3 couches | Storage / Compute (Warehouses) / Cloud Services |
| Cloud | AWS, Azure, GCP uniquement — pas d'on-premise |
| Micro-partitions | 50-500 MB, colonnaire, min/max automatiques |
| Pruning | Filtre sans fonction sur colonne = pruning actif |
| Standard | Time Travel 1j max |
| **Enterprise+** | Time Travel 90j, Multi-cluster, DDM, RAP |
| **Business Critical** | Tri-secret key, HIPAA, PrivateLink |

## Sécurité RBAC (20%)

| Rôle | Pouvoir |
|---|---|
| ACCOUNTADMIN | Tout |
| SYSADMIN | Warehouses, bases, schémas, tables |
| SECURITYADMIN | Rôles, utilisateurs, GRANTs |
| USERADMIN | Créer users et rôles |
| PUBLIC | Tous les utilisateurs |

```
DAC = propriétaire peut accorder accès à ses objets
RBAC = droits via rôles hiérarchiques
```

## Cache (21%)

| Cache | Durée | Perdu si | Crédits |
|---|---|---|---|
| Metadata | Permanent | Jamais | Aucun |
| Result | 24h | Données changent | Aucun |
| Warehouse | Vie du WH | Warehouse suspendu | Normaux |

## Ingestion (18%)

| Méthode | Warehouse | Latence |
|---|---|---|
| COPY INTO | Ton warehouse | Minutes |
| Snowpipe | **Serverless** | Secondes |
| Snowpipe Streaming | **Serverless** | < 1s |

## Pièges fréquents

```
❗ Tasks désactivées par défaut → ALTER TASK ... RESUME
❗ Fail-safe 7j → Support Snowflake UNIQUEMENT
❗ Time Travel ≠ Fail-safe
❗ Resource Monitors = ZERO crédit supplémentaire
❗ Trust Center = évaluer sécurité du compte (pas ACCOUNT_USAGE)
❗ DAC ≠ RBAC (DAC = propriétaire, RBAC = rôles)
❗ Vue sécurisée = obligatoire pour les Shares
❗ Clone = GRANTs source NON copiés sur le clone
❗ UPDATE Stream = 1 DELETE + 1 INSERT
❗ SYSTEM$STREAM_HAS_DATA avant d'exécuter une Task
```
