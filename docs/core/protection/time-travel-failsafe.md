# Time Travel & Fail-safe (CDP)

## Continuous Data Protection (CDP) ⭐

Snowflake protège les données sur **deux niveaux** :

```
Données modifiées/supprimées
        │
        ▼
┌───────────────────────┐
│     TIME TRAVEL       │  ← Accessible par l'utilisateur
│  (0–90 jours selon    │     via SQL AT | BEFORE
│   édition + config)   │
└───────────────────────┘
        │ expire
        ▼
┌───────────────────────┐
│      FAIL-SAFE        │  ← Snowflake uniquement (support)
│      (7 jours)        │     Non accessible par SQL
│  Tables permanentes   │
└───────────────────────┘
```

---

## Time Travel ⭐

Permet d'accéder aux données **avant** une modification ou suppression.

| Édition | Durée max Time Travel |
|---|---|
| Standard | **1 jour** |
| Enterprise+ | **90 jours** |

```sql
-- Configurer la durée de Time Travel
ALTER TABLE ma_table SET DATA_RETENTION_TIME_IN_DAYS = 30;

-- Interroger les données à un instant passé
SELECT * FROM ma_table AT (TIMESTAMP => '2024-01-15 10:00:00'::TIMESTAMP);

-- Interroger avant un statement précis
SELECT * FROM ma_table BEFORE (STATEMENT => '8e5d0ca9-005e-44e6-b858-a8f5b37c5726');

-- Il y a N secondes
SELECT * FROM ma_table AT (OFFSET => -3600);  -- il y a 1 heure

-- Restaurer une table supprimée
UNDROP TABLE ma_table;
UNDROP SCHEMA mon_schema;
UNDROP DATABASE ma_db;
```

!!! tip "UNDROP"
    `UNDROP` fonctionne tant que les données sont dans la fenêtre Time Travel.

---

## Fail-safe ⭐

| Caractéristique | Valeur |
|---|---|
| Durée | **7 jours** (fixe, non configurable) |
| Accès | **Support Snowflake uniquement** |
| Applicabilité | Tables **permanentes** uniquement |
| Tables temporaires | ❌ Pas de Fail-safe |
| Tables transitoires | ❌ Pas de Fail-safe |

!!! danger "Piège exam fréquent"
    - Time Travel = **configurable**, accessible par l'utilisateur via SQL
    - Fail-safe = **fixe 7 jours**, accessible **uniquement par Snowflake Support**
    - Tables **temporaires et transitoires** → **pas de Fail-safe**

---

## Coût stockage CDP

```
Coût total stockage = données actives
                    + données Time Travel
                    + données Fail-safe
```

!!! tip "Réduire les coûts"
    Pour les tables de staging/intermédiaires, utilise des tables **transitoires** :
    - `DATA_RETENTION_TIME_IN_DAYS = 0` (pas de Time Travel)
    - Pas de Fail-safe

```sql
CREATE TRANSIENT TABLE staging_raw (
    data VARIANT
);
-- 0 coût supplémentaire CDP
```

---

## Time Travel sur différents objets

```sql
-- Cloner une table à un instant passé (très utile !)
CREATE TABLE ma_table_backup CLONE ma_table
  AT (TIMESTAMP => '2024-01-14 08:00:00'::TIMESTAMP);

-- Time Travel sur une vue (interroge la table sous-jacente)
SELECT * FROM ma_vue AT (OFFSET => -1800);
```
