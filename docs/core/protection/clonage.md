# Clonage (Zero-Copy Clone)

## Concept ⭐

Le clonage Snowflake est **instantané et gratuit** en espace disque initial.

!!! info "Zero-Copy Clone"
    - Le clone **partage les micro-partitions** de la source → 0 octet copié au départ
    - Les modifications futures sur le clone ou la source créent **de nouvelles micro-partitions** indépendantes
    - Coût de stockage = seulement les données modifiées depuis le clone

---

## Syntaxe

```sql
-- Cloner une table
CREATE TABLE ma_table_dev CLONE ma_table_prod;

-- Cloner un schéma entier (clone toutes les tables du schéma)
CREATE SCHEMA schema_dev CLONE schema_prod;

-- Cloner une base de données entière
CREATE DATABASE db_dev CLONE db_prod;

-- Cloner à un instant passé (Time Travel + Clone)
CREATE TABLE ma_table_hier CLONE ma_table
  AT (OFFSET => -86400);  -- il y a 24h
```

---

## Objets clonables

| Objet | Clonable |
|---|---|
| Table | ✅ |
| Schéma | ✅ |
| Base de données | ✅ |
| Stage interne | ✅ |
| File format | ✅ |
| Séquence | ✅ |
| Stream | ❌ |
| Pipe | ❌ |
| Task | ❌ |

---

## Privilèges sur les clones ⭐

!!! danger "Question très fréquente"
    | Situation | Comportement |
    |---|---|
    | Privilèges sur l'objet source | **NON copiés** sur le clone |
    | Propriétaire du clone | Le rôle qui a exécuté CREATE CLONE |
    | Privilèges sur les sous-objets d'un schéma cloné | **Copiés** (ex: GRANTs sur les tables) |

```sql
-- Après ce clone, il faut réaccorder les droits
CREATE DATABASE db_dev CLONE db_prod;
-- Les GRANTs sur db_prod NE s'appliquent PAS à db_dev
GRANT USAGE ON DATABASE db_dev TO ROLE developer;
```

---

## Cas d'usage typiques

- **Environnements de dev/test** : clone prod → dev sans coût de stockage
- **Sauvegarde avant migration** : clone avant un gros UPDATE/DELETE
- **Sandbox analytics** : donner un clone de la prod aux data scientists
