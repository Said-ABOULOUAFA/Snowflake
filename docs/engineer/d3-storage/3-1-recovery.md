# 3.1 Data recovery (Time Travel, Fail-safe, réplication)

> **Domain 3.0 — Storage & Data Protection (14%)**

## Cycle de vie continu des données (CDP)

![Cycle de vie CDP](../../assets/cdp-lifecycle.svg)

| Phase | Durée | Accès | Récupération |
|---|---|---|---|
| **Active** | — | Lecture/écriture | — |
| **Time Travel** | 0–90 j (Enterprise) ; 0–1 j (Standard) | `AT/BEFORE`, `UNDROP` | Utilisateur |
| **Fail-safe** | 7 j (fixe, tables permanentes) | Aucun (Snowflake only) | Support Snowflake |

```sql
-- Interroger un état passé
SELECT * FROM ventes AT(OFFSET => -3600);            -- il y a 1 h
SELECT * FROM ventes BEFORE(STATEMENT => '<query_id>');
SELECT * FROM ventes AT(TIMESTAMP => '2026-06-01 09:00:00'::timestamp);

-- Restaurer un objet supprimé
UNDROP TABLE ventes;

-- Définir la rétention
ALTER TABLE ventes SET DATA_RETENTION_TIME_IN_DAYS = 30;
```

## Types de tables & protection ⭐

| Type | Time Travel | Fail-safe |
|---|---|---|
| `PERMANENT` | 0–90 j | 7 j |
| `TRANSIENT` | 0–1 j | **Aucun** |
| `TEMPORARY` | 0–1 j (session) | **Aucun** |

!!! danger "Piège exam"
    **Fail-safe n'est PAS configurable** et n'est PAS accessible par l'utilisateur (récupération via le support Snowflake uniquement, 7 jours). Les tables `TRANSIENT`/`TEMPORARY` n'ont **pas** de Fail-safe → moins de coûts de stockage mais aucune protection ultime.

## Réplication & failover

```sql
ALTER DATABASE ventes ENABLE REPLICATION TO ACCOUNTS org.compte_secondaire;
```

Réplication de bases/comptes pour la **continuité d'activité** (BCDR) entre régions/clouds ; bascule via groupes de failover.

📎 *Réf. : `docs.snowflake.com/en/user-guide/data-time-travel`*
