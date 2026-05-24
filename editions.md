# Cache & Optimisation des requêtes

## Les 3 niveaux de cache ⭐

| Cache | Niveau | Durée | Invalidation |
|---|---|---|---|
| **Result Cache** | Services Cloud | 24h | Si données source changent |
| **Local Disk Cache** | Warehouse (SSD) | Vie du warehouse | Suspension du warehouse |
| **Remote Disk Cache** | Stockage cloud | Permanent | Jamais (c'est le stockage) |

---

## Result Cache (Cache de résultats)

!!! danger "Question très fréquente"
    Le Result Cache **ne consomme aucun crédit** de calcul — le warehouse n'est même pas sollicité.

Conditions pour utiliser le Result Cache :
- La **requête SQL est identique** (même texte exact)
- Les **données source n'ont pas changé** depuis le dernier calcul
- Les **paramètres de session** sont identiques
- Moins de **24 heures** se sont écoulées

```sql
-- Vérifier si une requête utilise le cache
-- Dans le Query Profile → chercher "Query Result Reuse"

-- Désactiver le cache (pour tests de performance)
ALTER SESSION SET USE_CACHED_RESULT = FALSE;
```

---

## Local Disk Cache (Warehouse Cache)

- Snowflake garde en cache sur SSD les micro-partitions récemment lues
- **Perdu si le warehouse est suspendu ou redimensionné**
- C'est pourquoi suspendre/reprendre un warehouse peut ralentir les premières requêtes

!!! tip "Astuce"
    Pour des requêtes répétitives (dashboards BI), évite de suspendre le warehouse entre les sessions.

---

## Query Profile

Outil d'analyse des requêtes dans Snowsight.

```
Snowsight → Requête → Query Profile
```

Éléments à surveiller :
- **Bytes scanned** → si très élevé → envisager clustering
- **Spillage to disk/remote** → warehouse trop petit → augmenter la taille
- **Exploding joins** → produit cartésien → revoir la requête
- **Pruning** → % de micro-partitions éliminées → plus c'est élevé, mieux c'est

---

## Optimisation SQL — Bonnes pratiques

```sql
-- ✅ Filtrer tôt, sur colonnes clusterisées
SELECT montant FROM ventes
WHERE date_vente BETWEEN '2024-01-01' AND '2024-12-31'
  AND region = 'EMEA';

-- ✅ Utiliser des CTEs pour lisibilité et réutilisation
WITH ventes_2024 AS (
    SELECT * FROM ventes WHERE YEAR(date_vente) = 2024
)
SELECT region, SUM(montant) FROM ventes_2024 GROUP BY region;

-- ❌ Éviter SELECT * en production
-- ❌ Éviter les fonctions sur colonnes filtrées (empêche le pruning)
-- MAL: WHERE YEAR(date_vente) = 2024
-- BIEN: WHERE date_vente BETWEEN '2024-01-01' AND '2024-12-31'
```
