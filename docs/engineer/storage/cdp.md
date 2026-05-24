# CDP — Time Travel, Fail-safe & Réplication

## Continuous Data Protection (CDP) ⭐

```
Modification/Suppression
        │
        ▼
┌────────────────────────┐
│      TIME TRAVEL       │  ← SQL : AT | BEFORE | UNDROP
│  0–90 jours (config)   │     Accessible par l'utilisateur
└────────────────────────┘
        │ Expire
        ▼
┌────────────────────────┐
│       FAIL-SAFE        │  ← Support Snowflake uniquement
│    7 jours (fixe)      │     Non accessible par SQL
└────────────────────────┘
```

---

## Time Travel ⭐

```sql
-- Interroger les données à un instant passé
SELECT * FROM ventes
AT (TIMESTAMP => '2024-06-01 08:00:00'::TIMESTAMP_TZ);

-- Avant un statement précis
SELECT * FROM ventes
BEFORE (STATEMENT => '8e5d0ca9-005e-44e6-b858-a8f5b37c5726');

-- Il y a N secondes
SELECT * FROM ventes AT (OFFSET => -7200);  -- il y a 2h

-- Restaurer une table supprimée
UNDROP TABLE ventes_2022;
UNDROP SCHEMA mon_schema;
UNDROP DATABASE ma_db;

-- Cloner à un instant passé (très utile !)
CREATE TABLE ventes_backup CLONE ventes
AT (TIMESTAMP => '2024-06-01 00:00:00'::TIMESTAMP_TZ);
```

### Durées Time Travel par type de table

| Type | Durée max (Standard) | Durée max (Enterprise+) |
|---|---|---|
| **Permanente** | 1 jour | 90 jours |
| **Temporaire** | 1 jour | 1 jour |
| **Transitoire** | 1 jour | 1 jour |

```sql
-- Configurer la durée
ALTER TABLE ma_table SET DATA_RETENTION_TIME_IN_DAYS = 30;
ALTER DATABASE ma_db SET DATA_RETENTION_TIME_IN_DAYS = 7;
ALTER SCHEMA mon_schema SET DATA_RETENTION_TIME_IN_DAYS = 0;
```

---

## Fail-safe ⭐

| Caractéristique | Valeur |
|---|---|
| Durée | **7 jours** — fixe, non configurable |
| Accès | **Support Snowflake uniquement** |
| Tables concernées | **Permanentes uniquement** |
| Tables temporaires | ❌ Pas de Fail-safe |
| Tables transitoires | ❌ Pas de Fail-safe |

!!! danger "Distinction clé — très fréquente aux examens"
    - **Time Travel** = configurable (0–90j), **accessible par l'utilisateur via SQL**
    - **Fail-safe** = fixe (7j), **accessible uniquement par Snowflake Support**

---

## Optimisation des coûts CDP

```sql
-- Voir le coût de stockage CDP par table
SELECT table_name,
       ROUND(active_bytes/1e9, 2)       AS active_gb,
       ROUND(time_travel_bytes/1e9, 2)  AS time_travel_gb,
       ROUND(failsafe_bytes/1e9, 2)     AS failsafe_gb
FROM SNOWFLAKE.ACCOUNT_USAGE.TABLE_STORAGE_METRICS
WHERE deleted IS NULL
ORDER BY (time_travel_bytes + failsafe_bytes) DESC
LIMIT 20;

-- Utiliser des tables transitoires pour le staging (0 Fail-safe)
CREATE TRANSIENT TABLE staging_load AS
SELECT * FROM source_raw;
-- DATA_RETENTION = 0 par défaut → 0 coût Time Travel et Fail-safe
```
