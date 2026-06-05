# 1.1 — Architecture Snowflake

> **Domaine D1 — 31% du COF-C03**

## Architecture hybride ⭐

Snowflake combine deux architectures classiques :

```
┌──────────────────────────────────────────────────┐
│              CLOUD SERVICES LAYER                │
│  Authentification · Métadonnées · Optimiseur     │
│  Sécurité · Conformité · Transactions             │
├─────────────┬──────────────┬─────────────────────┤
│  Warehouse  │  Warehouse   │   Warehouse N        │
│     VW1     │     VW2      │   (indépendants)     │
│   COMPUTE   │   COMPUTE    │   COMPUTE LAYER      │
├─────────────┴──────────────┴─────────────────────┤
│            DATABASE STORAGE LAYER                │
│   Micro-partitions · Colonnes · Compressé        │
│   AWS S3 · Azure Blob · Google Cloud Storage     │
└──────────────────────────────────────────────────┘
```

| Architecture | Caractéristique | Avantage Snowflake |
|---|---|---|
| **Disque partagé** | Stockage central unique | Simplicité de gestion, cohérence |
| **Sans partage (MPP)** | Calcul indépendant par nœud | Performances, isolation |
| **Hybride Snowflake** | Les deux combinés | Scalabilité + simplicité |

---

## Couche 1 — Database Storage Layer

- Données stockées en **format colonnaire compressé**
- Découpées automatiquement en **micro-partitions** (50–500 MB non compressé)
- Hébergées sur le cloud public (AWS S3, Azure Blob Storage, GCS)
- Gestion 100% automatique — invisible pour l'utilisateur
- Supporte données **structurées, semi-structurées et non structurées**

---

## Couche 2 — Compute Layer (Entrepôts virtuels)

- Clusters de ressources de calcul **totalement indépendants**
- Aucun partage de ressources entre warehouses → **aucune contention**
- Exécutent : SQL, Snowpark (Python/Java/Scala), Spark via Snowpark
- Configurables : taille, auto-suspend, auto-resume, multi-cluster

!!! danger "Question fréquente"
    Deux warehouses sur le même compte Snowflake **ne s'impactent pas mutuellement**. Chacun a ses propres ressources isolées.

---

## Couche 3 — Cloud Services Layer

- **Authentification & autorisation** (RBAC, SSO, MFA, OAuth)
- **Catalogue de métadonnées** (informations sur tous les objets)
- **Parsing & optimisation des requêtes** (plan d'exécution)
- **Gestion des transactions** (ACID)
- **Conformité réglementaire** (HIPAA, PCI-DSS selon l'édition)

!!! tip "Cloud Services = cerveau de Snowflake"
    Cette couche fonctionne **en permanence sans warehouse**. C'est pourquoi les requêtes sur les métadonnées (COUNT(*), MIN/MAX) peuvent utiliser le **metadata cache** sans warehouse.

---

## Snowflake = SaaS uniquement ⭐

!!! danger "Piège exam — très fréquent"
    Snowflake **ne peut PAS** s'installer on-premise ou dans un data center privé.
    Il tourne **uniquement** sur cloud public : **AWS, Azure ou GCP**.

---

## Éditions Snowflake ⭐

| Fonctionnalité | Standard | Enterprise | Business Critical | Virtual Private |
|---|---|---|---|---|
| **Time Travel max** | 1 jour | **90 jours** | 90 jours | 90 jours |
| **Multi-cluster warehouse** | ❌ | ✅ | ✅ | ✅ |
| **Dynamic Data Masking** | ❌ | ✅ | ✅ | ✅ |
| **Row Access Policies** | ❌ | ✅ | ✅ | ✅ |
| **Tri-secret key** | ❌ | ❌ | ✅ | ✅ |
| **AWS PrivateLink** | ❌ | ❌ | ✅ | ✅ |
| **HIPAA / PCI-DSS** | ❌ | ❌ | ✅ | ✅ |
| **Environnement isolé** | ❌ | ❌ | ❌ | ✅ |

!!! danger "Questions très fréquentes"
    - Dynamic Data Masking → **Enterprise minimum**
    - Time Travel 90 jours → **Enterprise minimum**
    - Clé gérée par le client (tri-secret) → **Business Critical**
    - Conformité HIPAA / données de santé → **Business Critical**
    - Environnement physiquement isolé → **Virtual Private**
