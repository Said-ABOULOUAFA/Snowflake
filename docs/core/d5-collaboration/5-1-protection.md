# 5.1 — Protection des données (CDP)

> **Domaine D5 — 10% du COF-C03**

## Continuous Data Protection (CDP) ⭐

```
Données modifiées/supprimées
        │
        ▼
┌──────────────────────────┐
│      TIME TRAVEL         │  ← SQL : AT | BEFORE | UNDROP
│  0–90 jours (config)     │     Accessible par l'utilisateur
└──────────────────────────┘
        │ Expire
        ▼
┌──────────────────────────┐
│       FAIL-SAFE          │  ← Support Snowflake UNIQUEMENT
│    7 jours (fixe)        │     Non accessible par SQL
└──────────────────────────┘
```

## Time Travel ⭐

```sql
-- Interroger à un instant passé
SELECT * FROM ventes AT (TIMESTAMP => '2024-06-01 08:00:00'::TIMESTAMP_TZ);
SELECT * FROM ventes AT (OFFSET => -3600);               -- il y a 1h
SELECT * FROM ventes BEFORE (STATEMENT => 'query_id');   -- avant un statement

-- Restaurer
UNDROP TABLE ventes;
UNDROP SCHEMA mon_schema;
UNDROP DATABASE ma_db;

-- Cloner à un instant passé
CREATE TABLE ventes_backup CLONE ventes
  AT (TIMESTAMP => '2024-06-01 00:00:00'::TIMESTAMP_TZ);
```

| Édition | Time Travel max |
|---|---|
| Standard | **1 jour** |
| Enterprise+ | **90 jours** |

```sql
ALTER TABLE ma_table SET DATA_RETENTION_TIME_IN_DAYS = 30;
```

## Fail-safe ⭐

| Caractéristique | Valeur |
|---|---|
| Durée | **7 jours** — fixe, non configurable |
| Accès | **Support Snowflake UNIQUEMENT** |
| Tables concernées | **Permanentes uniquement** |
| Tables temporaires | ❌ Pas de Fail-safe |
| Tables transitoires | ❌ Pas de Fail-safe |

!!! danger "Distinction critique — très fréquente"
    - **Time Travel** = configurable, **accessible par l'utilisateur via SQL**
    - **Fail-safe** = fixe 7j, **accessible uniquement par Snowflake Support**

## Clonage (Zero-Copy Clone) ⭐

```sql
CREATE TABLE clients_dev CLONE clients_prod;
CREATE SCHEMA schema_dev CLONE schema_prod;
CREATE DATABASE db_dev CLONE db_prod;

-- Clone à un instant passé
CREATE TABLE clients_hier CLONE clients AT (OFFSET => -86400);
```

!!! danger "Privilèges sur clones — question fréquente"
    - Les **GRANTs sur l'objet source** ne sont **PAS copiés** sur le clone
    - Le propriétaire du clone = le rôle qui a exécuté CREATE CLONE
    - Exception : les GRANTs sur les **sous-objets** d'un schéma cloné **sont** copiés

### Objets clonables vs non-clonables

| Clonable | Non-clonable |
|---|---|
| Tables, schémas, bases | Streams ❌ |
| Stages internes | Tasks ❌ |
| File formats, séquences | Pipes ❌ |
