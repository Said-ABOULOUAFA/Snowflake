# Performance avancée — Cache, QAS, Semi-structuré

## Les 3 types de cache ⭐

| Cache | Niveau | Durée | Perdu si |
|---|---|---|---|
| **Result Cache** | Services Cloud | 24h | Données changent ou 24h écoulées |
| **Metadata Cache** | Services Cloud | Permanent | Jamais (métadonnées) |
| **Warehouse Cache** | Warehouse (SSD) | Vie du warehouse | Warehouse suspendu ou redimensionné |

!!! danger "Piège exam COF-C03"
    Il y a **3 niveaux de cache**, pas 2 ! Le **Metadata Cache** est souvent oublié.
    - **Metadata Cache** = statistiques sur les micro-partitions (min/max, count, null count) — utilisé par l'optimiseur sans activer de warehouse

```sql
-- Désactiver le result cache pour tester les vraies perfs
ALTER SESSION SET USE_CACHED_RESULT = FALSE;

-- Voir si une requête a utilisé le cache
-- Dans Query Profile → chercher "Query Result Reuse"
```

---

## Query Acceleration Service (QAS) ⭐

Accélère automatiquement les **parties lentes** d'une requête en utilisant des ressources serverless supplémentaires.

```sql
-- Activer le QAS sur un warehouse
ALTER WAREHOUSE wh_analytics SET ENABLE_QUERY_ACCELERATION = TRUE;

-- Limiter les crédits QAS (facteur multiplicateur du warehouse)
ALTER WAREHOUSE wh_analytics SET QUERY_ACCELERATION_MAX_SCALE_FACTOR = 8;
-- 8 = peut utiliser jusqu'à 8x la taille du warehouse en ressources QAS
```

!!! info "QAS vs Multi-cluster"
    | | QAS | Multi-cluster |
    |---|---|---|
    | **Problème résolu** | Requêtes individuelles lentes | Trop de requêtes simultanées |
    | **Mécanisme** | Ressources serverless supplémentaires | Nouveaux clusters |
    | **Déclenchement** | Automatique par requête | Automatique par file d'attente |

---

## Données Semi-structurées ⭐

### Type VARIANT

```sql
-- Créer une table avec VARIANT
CREATE TABLE events (
    id        NUMBER,
    data      VARIANT,
    created   TIMESTAMP
);

-- Insérer du JSON
INSERT INTO events SELECT 1, PARSE_JSON('{"user":"alice","action":"login","ip":"1.2.3.4"}'), CURRENT_TIMESTAMP();

-- Requêter avec la notation :
SELECT data:user::STRING        AS utilisateur,
       data:action::STRING      AS action,
       data:ip::STRING          AS adresse_ip
FROM events;

-- Accès imbriqué
SELECT data:address:city::STRING AS ville
FROM clients_json;

-- Notation entre crochets (si clé avec espaces ou dynamique)
SELECT data['user name']::STRING AS nom
FROM events;
```

### FLATTEN — Dépliage de tableaux

```sql
-- Table avec un tableau JSON
-- data = {"tags": ["sql", "python", "snowflake"]}

SELECT e.id,
       f.value::STRING AS tag
FROM events e,
LATERAL FLATTEN(INPUT => e.data:tags) f;
-- Retourne une ligne par élément du tableau
```

### Fonctions utiles

```sql
-- Vérifier si une clé existe
SELECT data:email IS NOT NULL AS a_email FROM clients_json;

-- Convertir VARIANT en STRING/NUMBER/DATE
SELECT data:age::INT AS age FROM clients_json;

-- Obtenir le type d'une valeur
SELECT TYPEOF(data:age) FROM clients_json;  -- 'INTEGER'

-- Construire un objet JSON
SELECT OBJECT_CONSTRUCT('nom', 'Alice', 'age', 30) AS json_obj;

-- Construire un tableau JSON
SELECT ARRAY_CONSTRUCT(1, 2, 3) AS tableau;

-- Combiner deux objets
SELECT OBJECT_INSERT(data, 'nouveau_champ', 'valeur') FROM events;
```

---

## Données Non-structurées

```sql
-- Créer un stage avec directory table activée
CREATE STAGE stage_docs
  URL = 's3://mon-bucket/documents/'
  DIRECTORY = (ENABLE = TRUE);

-- Rafraîchir le directory
ALTER STAGE stage_docs REFRESH;

-- Lister les fichiers avec métadonnées
SELECT * FROM DIRECTORY(@stage_docs);

-- Générer une URL pré-signée pour accéder à un fichier
SELECT BUILD_SCOPED_FILE_URL(@stage_docs, 'rapport.pdf');
SELECT BUILD_STAGE_FILE_URL(@stage_docs, 'rapport.pdf');
```

!!! info "Types d'URL"
    - `BUILD_SCOPED_FILE_URL` → URL temporaire sécurisée (expire)
    - `BUILD_STAGE_FILE_URL` → URL permanente (nécessite privilèges sur le stage)
