# 1.3 Hiérarchie & types d'objets

> **Domain 1.0 — Architecture (31%)**

## Hiérarchie des objets ⭐

```
ORGANIZATION
   └── ACCOUNT
         ├── Objets compte : USER, ROLE, WAREHOUSE, RESOURCE MONITOR,
         │                   INTEGRATION, NETWORK POLICY, SHARE
         └── DATABASE
                └── SCHEMA
                      ├── TABLE / VIEW / MATERIALIZED VIEW
                      ├── STAGE / FILE FORMAT / PIPE
                      ├── STREAM / TASK / DYNAMIC TABLE
                      ├── FUNCTION (UDF/UDTF/UDAF) / PROCEDURE
                      ├── SEQUENCE / ML MODEL / APPLICATION
                      └── POLICIES (masking, row access, ...)
```

| Niveau | Exemples d'objets |
|---|---|
| **Organisation** | Comptes, réplication, organisation-level roles |
| **Compte** | Users, rôles, warehouses, resource monitors, intégrations, shares |
| **Base de données** | Schémas |
| **Schéma** | Tables, vues, stages, file formats, pipes, streams, tasks, dynamic tables, UDFs, procédures, séquences, modèles ML, apps |

## Types de tables ⭐

| Type | Time Travel | Fail-safe | Persistance |
|---|---|---|---|
| **Permanent** | 0–90 j | ✅ 7 j | Permanente |
| **Transient** | 0–1 j | ❌ | Permanente (pas de Fail-safe) |
| **Temporary** | 0–1 j | ❌ | **Session uniquement** |
| **External** | — | — | Données hors Snowflake (lecture) |
| **Iceberg** | selon config | — | Format ouvert Apache Iceberg |
| **Dynamic** | — | — | Rafraîchie automatiquement (TARGET_LAG) |

!!! danger "Piège exam"
    Tables **TEMPORARY** et **TRANSIENT** → **pas de Fail-safe**. Idéales pour le staging afin de réduire les coûts CDP.

## Types de vues

| Vue | Description |
|---|---|
| **Standard** | Définition logique, recalculée à chaque requête |
| **Materialized** | Résultat **stocké physiquement** et maintenu auto (Enterprise+) |
| **Secure** | Masque la définition et empêche certaines optimisations qui pourraient fuiter des données |

## Variables de session & contexte

- **Hiérarchie des paramètres** : compte → user → session → objet.
- **Précédence** : le niveau le plus spécifique l'emporte.

📎 *Réf. : `docs.snowflake.com/en/user-guide/tables-storage-considerations`*
