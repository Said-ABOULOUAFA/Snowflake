# 3.3 — Time Travel & Clonage pour environnements dev

> **Domaine D3 Storage — 14% du DEA-C02**

## Cloner des objets ⭐

```sql
-- Clone de table
CREATE TABLE ventes_dev CLONE ventes_prod;

-- Clone de table à un instant passé
CREATE TABLE ventes_avant_bug CLONE ventes AT (OFFSET => -3600);

-- Clone de schéma (tous les objets)
CREATE SCHEMA schema_dev CLONE schema_prod;

-- Clone de base entière
CREATE DATABASE db_dev CLONE db_prod;

-- Clone avec Time Travel
CREATE DATABASE db_backup CLONE db_prod
  AT (TIMESTAMP => '2024-06-01 00:00:00'::TIMESTAMP_TZ);
```

## Héritage des permissions ⭐

!!! danger "Question critique DEA-C02"
    Les GRANTs sur l'objet source **NE SONT PAS copiés** sur le clone.
    Exception : les GRANTs sur les **sous-objets** d'un schéma/base cloné **SONT copiés**.

```
Table clonée :
  → GRANTs sur la TABLE source = NON copiés
  → Propriétaire = rôle qui a exécuté le CLONE

Schéma cloné :
  → GRANTs sur le SCHÉMA source = NON copiés
  → GRANTs sur les TABLES du schéma = OUI copiés

Base clonée :
  → GRANTs sur la BASE source = NON copiés
  → GRANTs sur les objets internes = OUI copiés
```

## Valider avant de promouvoir ⭐

```sql
-- Pattern de déploiement sécurisé
-- 1. Cloner la production
CREATE TABLE ventes_test CLONE ventes_prod;

-- 2. Appliquer les changements dans le clone
ALTER TABLE ventes_test ADD COLUMN categorie STRING;
UPDATE ventes_test SET categorie = 'A' WHERE montant > 1000;

-- 3. Valider les résultats
SELECT categorie, COUNT(*) FROM ventes_test GROUP BY categorie;
SELECT * FROM ventes_test WHERE categorie IS NULL;

-- 4. Promouvoir en production si OK
ALTER TABLE ventes_prod ADD COLUMN categorie STRING;
UPDATE ventes_prod SET categorie = 'A' WHERE montant > 1000;

-- 5. Si erreur : rollback avec Time Travel
INSERT INTO ventes_prod
SELECT * FROM ventes_prod AT (TIMESTAMP => 'avant_deploy'::TIMESTAMP_TZ);
```
