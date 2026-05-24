# Micro-partitions & Clustering

## Micro-partitions ⭐

Unité de stockage fondamentale de Snowflake.

| Caractéristique | Valeur |
|---|---|
| Taille | 50–500 MB (données non compressées) |
| Format | Colonnaire |
| Gestion | 100% automatique |
| Contenu | Statistiques min/max par colonne |

### Pruning automatique

Snowflake stocke les **valeurs min/max** de chaque colonne dans chaque micro-partition.

```sql
-- Requête avec filtre → Snowflake ne lit QUE les micro-partitions concernées
SELECT * FROM ventes WHERE date_vente = '2024-01-15';
-- Si date_vente est triée naturellement → pruning très efficace
-- Si date_vente est aléatoire → pruning peu efficace → envisager clustering
```

!!! tip "Pruning = performance gratuite"
    Plus tes données sont **naturellement triées** (ex: dates d'insertion chronologique), plus le pruning est efficace.

---

## Clustering Keys ⭐

Utilisé pour les **très grandes tables** (plusieurs TB) où le pruning naturel est insuffisant.

```sql
-- Ajouter une clé de clustering
ALTER TABLE ventes CLUSTER BY (date_vente, region);

-- Voir l'état du clustering
SELECT SYSTEM$CLUSTERING_INFORMATION('ventes', '(date_vente, region)');
```

### Clustering Depth

!!! danger "Question fréquente"
    **Clustering depth** = profondeur moyenne des micro-partitions qui se chevauchent.
    - Valeur **proche de 1** = excellent clustering
    - Valeur **élevée** = mauvais clustering → reclustering bénéfique

```sql
-- Vérifier si une table bénéficierait d'un clustering explicite
SELECT SYSTEM$CLUSTERING_DEPTH('ma_grande_table');
```

### Quand utiliser un clustering key ?

✅ Table > plusieurs TB
✅ Requêtes filtrent toujours sur les mêmes colonnes (ex: date, région)
✅ Les données sont insérées dans un ordre aléatoire

❌ Tables petites ou moyennes → clustering automatique suffit
❌ Si les requêtes n'ont pas de filtre → inutile

---

## Automatic Clustering

Snowflake reclusters automatiquement les tables avec une clé de clustering définie (service continu, facturé en crédits).

```sql
-- Activer/désactiver le reclustering automatique
ALTER TABLE ventes RESUME RECLUSTER;
ALTER TABLE ventes SUSPEND RECLUSTER;
```
