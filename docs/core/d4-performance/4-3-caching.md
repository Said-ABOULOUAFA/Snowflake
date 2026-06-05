# 4.3 — Les 3 niveaux de cache ⭐

> **Domaine D4 — 21% du COF-C03**

## Vue d'ensemble

```
Requête SQL
     │
     ▼
[1] METADATA CACHE ── Cloud Services Layer ── Permanent
     │ Si pas en cache
     ▼
[2] RESULT CACHE ──── Cloud Services Layer ── 24h
     │ Si pas en cache
     ▼
[3] WAREHOUSE CACHE ─ Warehouse (SSD) ────── Vie du warehouse
     │ Si pas en cache
     ▼
   Lecture des micro-partitions (stockage cloud)
```

## 1. Metadata Cache ⭐

- **Niveau** : Cloud Services Layer
- **Durée** : Permanent (jamais invalidé)
- **Contenu** : Statistiques min/max des colonnes, count, null count
- **Accès** : Sans warehouse — gratuit
- **Cas d'usage** : `SELECT COUNT(*)`, `MIN()`, `MAX()` sur toute la table

```sql
-- Ces requêtes utilisent le Metadata Cache (pas de warehouse requis !)
SELECT COUNT(*) FROM ma_table;
SELECT MIN(date_vente), MAX(date_vente) FROM ventes;
```

## 2. Result Cache (Query Result Cache) ⭐

- **Niveau** : Cloud Services Layer
- **Durée** : **24 heures**
- **Invalidation** : Si les données source changent ou 24h écoulées
- **Coût** : **AUCUN crédit de calcul** — le warehouse n'est pas activé

### Conditions pour utiliser le Result Cache

1. La **requête SQL est identique** (même texte exact, case-sensitive)
2. Les **données source n'ont pas changé**
3. Les **paramètres de session** sont identiques
4. Moins de **24 heures** écoulées

```sql
-- Désactiver le Result Cache (pour tests de performance)
ALTER SESSION SET USE_CACHED_RESULT = FALSE;

-- Vérifier dans Query Profile → "Query Result Reuse"
```

!!! danger "Question fréquente exam"
    Le Result Cache **ne consomme AUCUN crédit de warehouse**. C'est géré par la couche Cloud Services.

## 3. Warehouse Cache (Local Disk Cache) ⭐

- **Niveau** : Warehouse (SSD local)
- **Durée** : Tant que le warehouse est actif
- **Invalidation** : **Warehouse suspendu ou redimensionné → cache perdu**
- **Contenu** : Micro-partitions récemment lues

```sql
-- ⚠️ Suspendre un warehouse détruit son cache local
ALTER WAREHOUSE wh_bi SUSPEND;
-- → Premier requête après reprise sera plus lente (rechargement du cache)
```

!!! tip "Astuce pour les dashboards BI"
    Si tes requêtes BI sont répétitives, évite de suspendre trop fréquemment le warehouse BI pour préserver le Warehouse Cache.

## Récapitulatif ⭐

| Cache | Niveau | Durée | Perdu si | Crédits |
|---|---|---|---|---|
| Metadata | Cloud Services | Permanent | Jamais | Aucun |
| Result | Cloud Services | 24h | Données changent | Aucun |
| Warehouse | Warehouse SSD | Vie du warehouse | Warehouse suspendu | Normaux |
