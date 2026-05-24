# ❓ Questions fréquentes aux examens

## Architecture

??? question "Quelle est la différence entre architecture à disque partagé et sans partage ?"
    - **Disque partagé** : stockage central partagé par tous les nœuds → simplicité de gestion
    - **Sans partage** : chaque nœud a ses propres ressources → performance MPP
    - Snowflake **combine les deux** : stockage central + calcul MPP indépendant

??? question "Peut-on installer Snowflake on-premise ?"
    **Non.** Snowflake est uniquement disponible sur cloud public (AWS, Azure, GCP).

??? question "Deux warehouses sur le même compte impactent-ils leurs performances respectives ?"
    **Non.** Chaque warehouse est **totalement indépendant**. Aucune ressource partagée.

---

## Tables & Stockage

??? question "Quelle table choisir pour des charges OLTP (transactionnelles) dans Snowflake ?"
    **Tables Hybrides** — elles supportent le verrouillage de lignes, les contraintes référentielles et les accès par index (faible latence).

??? question "Quelle est la différence entre table temporaire et table transitoire ?"
    - **Temporaire** : disparaît à la fin de la session. Time Travel max 1 jour. Pas de Fail-safe.
    - **Transitoire** : persiste après la session. Time Travel max 1 jour. Pas de Fail-safe.
    Les deux permettent d'économiser les coûts Fail-safe.

??? question "Une table clonée hérite-t-elle des privilèges de la source ?"
    **Non.** Les privilèges sur l'objet source ne sont PAS copiés. Le propriétaire du clone est le rôle qui a exécuté CREATE CLONE.

---

## Time Travel & Fail-safe

??? question "Quelle est la durée du Fail-safe ?"
    **7 jours**, fixe et non configurable. Accessible uniquement par le support Snowflake (pas via SQL).

??? question "Quelle édition permet 90 jours de Time Travel ?"
    **Enterprise** (et supérieur). L'édition Standard est limitée à **1 jour**.

??? question "UNDROP fonctionne-t-il toujours ?"
    Uniquement si l'objet est encore dans la fenêtre **Time Travel**. Après expiration du Time Travel, impossible.

---

## Chargement

??? question "Quelle est la différence entre Snowpipe et COPY INTO ?"
    - **COPY INTO** : chargement batch, manuel ou planifié, utilise ton warehouse
    - **Snowpipe** : chargement automatique dès arrivée d'un fichier, serverless (warehouse Snowflake)

??? question "Snowpipe Streaming nécessite-t-il un warehouse ?"
    **Non.** Snowpipe Streaming est **serverless** et ingère les données au niveau ligne.

??? question "Comment éviter de recharger un fichier déjà chargé ?"
    Snowflake garde un historique des fichiers chargés pendant **64 jours**. Utiliser `FORCE = FALSE` (défaut). Au-delà de 64 jours, la déduplication ne fonctionne plus.

---

## Sécurité

??? question "Quelle édition minimum pour Dynamic Data Masking ?"
    **Enterprise**.

??? question "Un consommateur d'un partage peut-il re-partager les données ?"
    **Non.** Les données partagées sont en **lecture seule** et ne peuvent pas être re-partagées.

??? question "Qui paie les crédits de calcul quand un consommateur interroge un partage ?"
    **Le consommateur** paie son propre warehouse. Le fournisseur paie le stockage.

---

## Performance

??? question "Qu'est-ce que la clustering depth et quelle valeur est bonne ?"
    La clustering depth mesure le chevauchement moyen des micro-partitions. Une valeur **proche de 1** indique un excellent clustering. Plus elle est élevée, moins le pruning est efficace.

??? question "Le Result Cache consomme-t-il des crédits warehouse ?"
    **Non.** Le Result Cache est géré par la couche Services Cloud. Aucun warehouse n'est activé.

??? question "Que se passe-t-il si un warehouse est suspendu fréquemment ?"
    Le **Local Disk Cache** (cache des micro-partitions sur SSD) est perdu à chaque suspension. Les premières requêtes après reprise seront plus lentes.
