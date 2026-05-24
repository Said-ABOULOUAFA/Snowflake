# Clustering & Pruning avancé

## Rappel : Micro-partitions et pruning

Snowflake stocke les **min/max** de chaque colonne dans chaque micro-partition. Lors d'un filtre, seules les micro-partitions potentiellement concernées sont lues.

```
Table ventes (10 TB)
├── Micro-partition 1 : date 2024-01-01 → 2024-01-15  ← lu si filtre janv.
├── Micro-partition 2 : date 2024-01-16 → 2024-01-31  ← lu si filtre janv.
├── Micro-partition 3 : date 2024-02-01 → 2024-02-28  ← ignoré si filtre janv.
└── ...
```

---

## Clustering Keys ⭐

Pour les **très grandes tables** (multi-TB) où l'ordre naturel d'insertion ne correspond pas aux patterns de requêtes.

```sql
-- Ajouter une clé de clustering
ALTER TABLE ventes CLUSTER BY (date_vente);

-- Clustering sur plusieurs colonnes
ALTER TABLE ventes CLUSTER BY (date_vente, region);

-- Clustering sur une expression
ALTER TABLE ventes CLUSTER BY (DATE_TRUNC('month', date_vente));

-- Voir le statut du clustering
SELECT SYSTEM$CLUSTERING_INFORMATION('ventes', '(date_vente)');
-- Retourne: average_depth, average_overlaps, total_partition_count

-- Supprimer le clustering
ALTER TABLE ventes DROP CLUSTERING KEY;
```

---

## Clustering Depth & Overlaps ⭐

| Métrique | Signification | Valeur idéale |
|---|---|---|
| **Average Depth** | Nombre moyen de micro-partitions qui se chevauchent | **Proche de 1** |
| **Average Overlaps** | Chevauchement moyen des valeurs | **Proche de 0** |
| **Partition Count** | Nombre total de micro-partitions | — |

```sql
-- Évaluer si le clustering serait bénéfique
SELECT SYSTEM$CLUSTERING_DEPTH('ma_grande_table');

-- Résultat : 1.0 → déjà bien clusterisé, pas besoin de clé explicite
-- Résultat : 5.8 → beaucoup de chevauchement, clustering recommandé
```

!!! danger "Piège exam"
    Une **clustering depth proche de 1** = excellent clustering.
    Une valeur **élevée** = mauvais clustering = pruning inefficace.

---

## Automatic Clustering ⭐

Snowflake reclusters automatiquement les tables (service continu facturé en crédits).

```sql
-- Activer le reclustering automatique
ALTER TABLE ventes RESUME RECLUSTER;

-- Suspendre (pour contrôler les coûts)
ALTER TABLE ventes SUSPEND RECLUSTER;

-- Voir les statistiques de reclustering
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.AUTOMATIC_CLUSTERING_HISTORY
WHERE start_time >= DATEADD('day', -7, CURRENT_TIMESTAMP())
  AND table_name = 'VENTES';
```

---

## Quand utiliser un Clustering Key ? ⭐

**✅ Utiliser si :**
- Table > plusieurs TB
- Requêtes filtrent toujours sur les mêmes colonnes
- Données insérées dans un ordre aléatoire (ex: UUID comme clé)
- Clustering depth > 3-4

**❌ Ne pas utiliser si :**
- Table petite ou moyenne (< quelques centaines de GB)
- Pas de filtre répétitif dans les requêtes
- Les données sont déjà naturellement ordonnées (ex: dates chronologiques)

---

## Clustering vs Search Optimization ⭐

| | Clustering Key | Search Optimization |
|---|---|---|
| **Type de requête** | Range filters, agrégations | Equality, LIKE, IN |
| **Exemple** | `WHERE date BETWEEN ... AND ...` | `WHERE id = 'ABC123'` |
| **Mécanisme** | Réorganise les micro-partitions | Index de recherche |
| **Coût** | Crédits de reclustering | Crédits de maintenance + stockage |
| **Tables** | Très grandes tables | Tables avec lookups fréquents |

```sql
-- Search Optimization pour lookups ponctuels
ALTER TABLE clients ADD SEARCH OPTIMIZATION ON EQUALITY(email, telephone);

-- Vérifier le statut
SHOW TABLES LIKE 'clients';
-- Colonne: search_optimization
```

---

## Analyser le pruning dans Query Profile

```sql
-- Lancer une requête
SELECT SUM(montant) FROM ventes WHERE date_vente = '2024-06-15';

-- Dans Snowsight → Query Profile :
-- "Partitions scanned" vs "Partitions total"
-- Exemple: 12 / 10,000 = 99.88% éliminé = excellent pruning ✅
-- Exemple: 9,800 / 10,000 = 2% éliminé = mauvais pruning ❌ → envisager clustering
```
