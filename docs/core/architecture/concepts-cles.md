# Architecture Snowflake — Concepts clés

## Architecture hybride

Snowflake combine deux architectures classiques :

| Architecture | Caractéristique |
|---|---|
| **Disque partagé** | Stockage central unique, accessible par tous les nœuds |
| **Sans partage (MPP)** | Chaque nœud calcule indépendamment |

> L'hybridation offre la **simplicité** du disque partagé + les **performances** du MPP.

---

## Les 3 couches — À connaître par cœur ⭐

```
┌──────────────────────────────────────────┐
│           SERVICES CLOUD                 │
│  Sécurité · Métadonnées · Optimiseur     │
│  Authentification · Conformité           │
├─────────────┬────────────┬───────────────┤
│  Entrepôt 1 │ Entrepôt 2 │  Entrepôt N   │  ← CALCUL (MPP)
│  (indépendant de chaque autre)           │
├─────────────┴────────────┴───────────────┤
│              STOCKAGE                    │
│  Micro-partitions · Format colonnaire    │
│  Compressé · Centralisé                  │
└──────────────────────────────────────────┘
```

!!! warning "Question fréquente"
    Les entrepôts virtuels **ne partagent aucune ressource** entre eux → aucun impact de performance entre warehouses.

---

## Couche Stockage

- Format **colonnaire compressé** (optimisé pour l'analytique)
- Données découpées en **micro-partitions** (50–500 MB avant compression)
- Gestion 100% automatique par Snowflake
- Stocké sur cloud public (AWS S3, Azure Blob, GCS)

## Couche Calcul

- **Entrepôt virtuel** = cluster de ressources de calcul
- Indépendant des autres entrepôts → pas de contention
- Exécute : SQL, Snowpark (Python/Java/Scala), Spark via Snowpark Connect
- **Auto-suspend** et **auto-resume** configurables

## Couche Services Cloud

- Sécurité & authentification
- Catalogue des métadonnées (SNOWFLAKE DB)
- Parsing & optimisation des requêtes
- Gestion des transactions
- Conformité réglementaire

---

## Snowflake est un SaaS — pas installable on-premise

!!! danger "Piège exam"
    Snowflake **ne peut pas** s'installer sur des serveurs privés ou on-premise. Il tourne uniquement sur AWS, Azure ou GCP.
