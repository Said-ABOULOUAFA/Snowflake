# 📋 Cheat Sheet — SnowPro Core

## Architecture — Réponses rapides

| Question | Réponse |
|---|---|
| Combien de couches ? | **3** : Stockage / Calcul / Services Cloud |
| Les warehouses partagent-ils des ressources ? | **Non**, totalement indépendants |
| Snowflake peut-il s'installer on-premise ? | **Non**, cloud public uniquement |
| Taille micro-partitions ? | **50–500 MB** (non compressé) |
| Architecture hybride = ? | **Disque partagé** + **Sans partage (MPP)** |

---

## Editions — Réponses rapides

| Fonctionnalité | Édition minimum |
|---|---|
| Time Travel 90 jours | **Enterprise** |
| Dynamic Data Masking | **Enterprise** |
| Row Access Policies | **Enterprise** |
| Multi-cluster warehouse | **Enterprise** |
| Tri-secret key | **Business Critical** |
| HIPAA / PCI-DSS | **Business Critical** |
| AWS PrivateLink | **Business Critical** |
| Environnement isolé | **Virtual Private** |

---

## Tables — Réponses rapides

| Question | Réponse |
|---|---|
| Table sans Fail-safe ? | **Temporaire** ou **Transitoire** |
| Table pour OLTP + OLAP ? | **Hybride** (Unistore) |
| Table avec verrouillage de lignes ? | **Hybride** |
| Table pour data lake externe ? | **Iceberg** |
| Time Travel max Standard ? | **1 jour** |
| Time Travel max Enterprise ? | **90 jours** |
| Fail-safe durée ? | **7 jours** (fixe, non configurable) |
| Fail-safe accessible par ? | **Support Snowflake uniquement** |

---

## Chargement — Réponses rapides

| Question | Réponse |
|---|---|
| Chargement batch manuel ? | **COPY INTO** |
| Chargement auto dès fichier disponible ? | **Snowpipe** |
| Chargement continu lignes (Kafka) ? | **Snowpipe Streaming** |
| Snowpipe utilise quel warehouse ? | **Serverless** (géré Snowflake) |
| Déduplication auto COPY INTO ? | **64 jours** |
| ON_ERROR défaut ? | **ABORT_STATEMENT** |

---

## Cache — Réponses rapides

| Question | Réponse |
|---|---|
| Cache résultats durée ? | **24 heures** |
| Cache résultats consomme des crédits ? | **Non** |
| Cache local disk perdu si ? | **Warehouse suspendu** |
| Condition cache résultats ? | Même requête + données inchangées |

---

## Sécurité — Réponses rapides

| Question | Réponse |
|---|---|
| Rôle tout-puissant ? | **ACCOUNTADMIN** |
| Rôle créer warehouses/DBs ? | **SYSADMIN** |
| Rôle gérer utilisateurs ? | **USERADMIN** |
| GRANTs copiés sur un clone ? | **Non** (propriétaire = créateur du clone) |
| Consommateur partage peut re-partager ? | **Non** |
| Accès objets partagés = ? | **Lecture seule** |

---

## Clonage — Réponses rapides

| Question | Réponse |
|---|---|
| Coût initial du clone ? | **0** (zero-copy) |
| Streams clonables ? | **Non** |
| Tasks clonables ? | **Non** |
| Pipes clonables ? | **Non** |

---

## Streams & Tasks — Réponses rapides

| Question | Réponse |
|---|---|
| Task activée par défaut ? | **Non** → `ALTER TASK ... RESUME` |
| UPDATE dans un stream = ? | **1 DELETE + 1 INSERT** |
| Stream ne consomme pas de données si ? | Non consommé dans une transaction |
| Condition d'exécution task ? | `WHEN SYSTEM$STREAM_HAS_DATA(...)` |
