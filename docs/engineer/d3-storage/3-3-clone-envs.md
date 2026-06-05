# 3.3 Time Travel & clonage pour environnements

> **Domain 3.0 — Storage & Data Protection (14%)**

## Zero-Copy Cloning

```sql
-- Cloner instantanément une base entière (dev à partir de prod)
CREATE DATABASE dev_db CLONE prod_db;

-- Cloner une table à un instant passé (Time Travel + clone)
CREATE TABLE ventes_avant_incident CLONE ventes
  BEFORE(STATEMENT => '<query_id>');
```

| Caractéristique | Détail |
|---|---|
| **Coût initial** | Nul (partage des micro-partitions) |
| **Coût ultérieur** | Seules les modifications consomment du stockage |
| **Objets clonables** | Bases, schémas, tables, streams, tasks… |
| **Indépendance** | Clone et source évoluent séparément |

!!! danger "Piège exam"
    Le **zero-copy clone** ne duplique PAS les données au départ : il référence les mêmes micro-partitions. Le stockage supplémentaire n'apparaît qu'au fur et à mesure des **écritures divergentes** (copy-on-write). Idéal pour créer des environnements dev/test à partir de la prod sans coût immédiat.

## Combiner Time Travel + Clone

Récupération chirurgicale d'un état antérieur sans toucher à la table de prod :

```sql
CREATE TABLE recup CLONE ventes AT(OFFSET => -1800);
-- vérifier, puis SWAP si besoin
ALTER TABLE ventes SWAP WITH recup;
```

!!! tip
    `ALTER TABLE ... SWAP WITH` échange atomiquement deux tables (métadonnées) — pratique pour les déploiements blue/green.

📎 *Réf. : `docs.snowflake.com/en/user-guide/object-clone`*
