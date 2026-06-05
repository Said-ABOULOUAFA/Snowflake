# 4.3 Le cache (3 niveaux) ⭐

> **Domain 4.0 — Performance, Querying & Transformation (21%)**

## Les 3 niveaux de cache

| Cache | Couche | Contenu | Invalidation |
|---|---|---|---|
| **Result Cache** | Services cloud | Résultat complet d'une requête | 24 h, ou si données/contexte changent |
| **Metadata Cache** | Services cloud | Stats (min/max, counts) | Mise à jour auto |
| **Warehouse (local) Cache** | Compute (SSD) | Micro-partitions lues | Vidé quand le warehouse suspend |

```sql
-- Result cache : même requête, mêmes données → 0 crédit de calcul
SELECT COUNT(*) FROM ventes;   -- 1re fois : calcul
SELECT COUNT(*) FROM ventes;   -- 2e fois : result cache (instantané)
```

!!! danger "Pièges examen result cache"
    Le result cache est réutilisé seulement si :
    - la requête est **identique** (texte),
    - les **données sous-jacentes n'ont pas changé**,
    - le **rôle / contexte** permet le même résultat,
    - fonctions **non déterministes** (`CURRENT_TIMESTAMP()`…) → pas de réutilisation.
    Il fonctionne **même warehouse suspendu** (c'est la couche services cloud).

!!! tip "Warehouse cache"
    Suspendre un warehouse **vide** son cache local. Un `AUTO_SUSPEND` trop court peut donc dégrader la perf des requêtes répétées (relire les micro-partitions).

## Query Acceleration Service (QAS)

Délègue une partie du scan à des ressources serverless additionnelles pour les requêtes avec gros volumes ponctuels.

📎 *Réf. : `docs.snowflake.com/en/user-guide/querying-persisted-results`*
