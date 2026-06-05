# 📖 Lexique Snowflake (définitions de référence)

Glossaire complet des termes Snowflake utiles pour la préparation **et** la pratique en entreprise. Classé par thème.

---

## Architecture & plateforme

**AI Data Cloud** — Nom de la plateforme Snowflake : un service cloud unique combinant entrepôt de données, lac de données, ingénierie de données, partage et applications, avec des capacités d'IA/ML intégrées.

**Architecture à 3 couches** — Séparation physique de **Stockage** (données), **Compute** (entrepôts virtuels) et **Cloud Services** (cerveau : optimiseur, métadonnées, sécurité). Les trois s'adaptent indépendamment.

**Architecture hybride** — Combine le *disque partagé* (un stockage central unique) et le *sans-partage / MPP* (chaque nœud calcule sur sa part). Snowflake en prend le meilleur : simplicité + performance massivement parallèle.

**MPP (Massively Parallel Processing)** — Traitement réparti sur plusieurs nœuds travaillant en parallèle ; base du calcul d'un entrepôt virtuel.

**Cloud Services Layer** — Couche toujours active qui gère authentification, contrôle d'accès, optimiseur de requêtes, métadonnées, gestion des transactions et le *result cache*. Facturée seulement au-delà de 10 % des crédits compute du jour.

**SaaS** — Software as a Service. Snowflake est 100 % managé, **jamais on-premise** ; tourne sur AWS, Azure ou GCP.

**Région / Cloud** — Emplacement géographique et fournisseur cloud d'un compte. Le partage direct exige la même région/cloud ; sinon réplication.

---

## Objets & organisation

**Organisation → Compte → Base de données → Schéma → Objet** — Hiérarchie de conteneurs. Un *schéma* regroupe tables, vues, fonctions, stages, etc.

**Warehouse (entrepôt virtuel)** — Cluster de calcul à la demande qui exécute requêtes, chargements et Snowpark. Dimensionné de XS à 6XL ; indépendant des autres warehouses.

**Multi-cluster warehouse** — Warehouse pouvant démarrer plusieurs clusters pour absorber la **concurrence** (beaucoup d'utilisateurs simultanés), pas pour accélérer une requête isolée.

**Crédit** — Unité de facturation du compute. La consommation dépend de la taille du warehouse et du temps d'exécution.

**Stage** — Emplacement de fichiers pour le chargement/déchargement. *Interne* (géré par Snowflake) ou *externe* (S3/Blob/GCS). Un *named stage* est réutilisable.

**Directory table** — Couche de métadonnées sur un stage qui liste les fichiers (utile pour les données non structurées).

**Sequence** — Générateur de nombres uniques croissants.

**Stream** — Objet de **Change Data Capture (CDC)** : capture les inserts/updates/deletes d'une table source depuis le dernier offset consommé.

**Task** — Unité d'exécution planifiée (CRON ou intervalle) pouvant chaîner d'autres tasks pour former un **DAG** de pipeline.

**Dynamic Table** — Table déclarative qui se rafraîchit automatiquement selon un `TARGET_LAG` à partir d'une requête source (remplace souvent un combo Stream+Task).

---

## Stockage & données

**Micro-partition** — Unité de stockage automatique (~50–500 Mo non compressés), en **colonnes**, immuable. Snowflake en crée et en gère des millions sans intervention.

**Métadonnées de partition** — Min/max, nombre de valeurs distinctes, etc. par micro-partition ; permettent le *pruning*.

**Pruning (élagage de partitions)** — Le moteur ignore les micro-partitions qui ne peuvent pas contenir de lignes correspondant aux filtres → moins de données lues.

**Clustering** — Ordonnancement des données pour regrouper les valeurs proches dans les mêmes micro-partitions et améliorer le pruning.

**Clustering key** — Colonne(s) choisie(s) pour piloter le clustering d'une grande table (`CLUSTER BY`). Le *reclustering* automatique consomme des crédits.

**Clustering depth** — Mesure du chevauchement des micro-partitions sur la clé : plus c'est **bas** (proche de 1), meilleur est le clustering.

**VARIANT** — Type semi-structuré stockant JSON/Avro/ORC/Parquet/XML de façon optimisée en colonnes.

**FLATTEN** — Fonction de table qui déplie un tableau ou objet semi-structuré en lignes (`LATERAL FLATTEN`).

---

## Types de tables ⭐

**Permanent** — Table standard : Time Travel 0–90 j + Fail-safe 7 j.

**Transient** — Persiste après la session mais Time Travel 0–1 j et **aucun Fail-safe** → moins de coûts de stockage.

**Temporary** — Existe le temps de la session, Time Travel ≤ 1 j, pas de Fail-safe.

**External table** — Table pointant vers des fichiers dans un stage externe (données non chargées dans Snowflake).

**Hybrid table (Unistore)** — Table transactionnelle (OLTP) avec index et contraintes appliquées, faible latence sur lignes.

---

## Protection & récupération

**Time Travel** — Accès à l'état passé des données (`AT`/`BEFORE`) et restauration (`UNDROP`) pendant la fenêtre de rétention (0–90 j en Enterprise, 0–1 j en Standard).

**Fail-safe** — Période **fixe de 7 jours** après le Time Travel, où seul le **support Snowflake** peut récupérer les données. Non configurable, non interrogeable.

**CDP (Continuous Data Protection)** — Ensemble des mécanismes protégeant les données : chiffrement, Time Travel, Fail-safe, réplication.

**Zero-Copy Cloning** — Copie instantanée d'un objet (base, schéma, table) sans duplication initiale : les micro-partitions sont partagées, le stockage ne croît qu'aux écritures divergentes (*copy-on-write*).

**Réplication / Failover** — Copie de bases ou comptes entre régions/clouds pour la continuité d'activité (BCDR).

---

## Chargement & intégration

**COPY INTO** — Commande de chargement (table ← stage) ou déchargement (stage ← table) par **lots**, utilisant ton warehouse.

**Snowpipe** — Service d'ingestion **continue et serverless** : charge les fichiers dès leur arrivée via notifications d'événements (pas de warehouse à gérer).

**Snowpipe Streaming** — Ingestion en quasi temps réel **ligne par ligne** via une API (latence plus faible que Snowpipe fichier).

**File format** — Objet décrivant la structure des fichiers (CSV/JSON/Parquet…) : délimiteurs, compression, gestion d'erreurs.

**INFER_SCHEMA** — Déduit automatiquement le schéma de fichiers Parquet/ORC/Avro/CSV pour créer/charger une table.

**Connector** — Pilote/intégration officiel (Python, JDBC, ODBC, Spark, Kafka, etc.) pour relier des applications à Snowflake.

---

## Performance

**Query Profile** — Vue graphique de l'exécution d'une requête (opérateurs, partitions scannées, spilling…) ; outil n°1 de diagnostic.

**Spilling** — Débordement de mémoire vers le disque : *local* (warehouse) puis *remote* (object store, très coûteux). Signale un warehouse sous-dimensionné.

**Result cache** — Cache (couche Cloud Services, 24 h) renvoyant le résultat exact d'une requête identique déjà exécutée, sans recalcul.

**Warehouse / local disk cache** — Cache SSD des données récemment lues par un warehouse (perdu à la suspension).

**Search Optimization Service (SOS)** — Accélère les recherches très **sélectives** (point lookups) sur de grandes tables.

**Materialized View** — Vue **pré-calculée et maintenue** automatiquement, utile pour des agrégations répétées.

**Query Acceleration Service (QAS)** — Ressources serverless additionnelles pour accélérer les scans massifs ponctuels.

---

## Sécurité & gouvernance

**RBAC (Role-Based Access Control)** — Modèle d'autorisations : les privilèges sont accordés à des **rôles**, eux-mêmes assignés aux utilisateurs ; les rôles peuvent hériter les uns des autres.

**Rôles système** — ORGADMIN, ACCOUNTADMIN, SECURITYADMIN, USERADMIN, SYSADMIN, PUBLIC.

**Privilège** — Droit précis sur un objet (SELECT, INSERT, USAGE, OWNERSHIP…).

**Dynamic Data Masking** — *Masking policy* qui transforme la **valeur** affichée d'une colonne selon le rôle de l'utilisateur.

**Row Access Policy** — Politique filtrant les **lignes** visibles selon le contexte (rôle/utilisateur).

**Object Tagging** — Étiquettes (clé/valeur) posées sur des objets pour classer, suivre les coûts ou piloter le masquage (*tag-based masking*).

**Data Classification** — Détection automatique des colonnes sensibles (PII) avec catégories sémantiques/confidentialité.

**Access History** — Journal d'audit des lectures/écritures, fournissant le **data lineage** (colonnes lues vs modifiées).

**Data Clean Room** — Environnement où deux parties croisent leurs données **sans jamais exposer** les lignes brutes de l'autre.

---

## Collaboration

**Secure Data Sharing** — Partage de données **sans copie** : le consommateur lit en direct les objets du fournisseur via des métadonnées partagées.

**Share** — Objet encapsulant les bases/objets et les comptes autorisés à les consommer.

**Listing** — Offre publiée (Marketplace privé ou public) packageant des données partagées + description.

**Marketplace** — Place de marché Snowflake pour découvrir/consommer des jeux de données et applications tiers.

**Reader Account** — Compte géré par un fournisseur permettant à un non-client Snowflake de consommer des données partagées.

---

## Snowpark & développement

**Snowpark** — Bibliothèque (Python/Java/Scala) pour écrire des transformations dans un langage de programmation, **compilées en SQL et exécutées dans Snowflake** (*pushdown*).

**Pushdown** — Traduction du code DataFrame en SQL exécuté côté serveur : les données ne transitent pas vers le client.

**Évaluation paresseuse (lazy)** — Les transformations (`select`, `filter`, `join`) ne s'exécutent qu'à une **action** (`show`, `collect`, `count`, `save_as_table`).

**Session** — Objet encapsulant la connexion + le contexte (warehouse/base/schéma) pour Snowpark.

**DataFrame** — Représentation tabulaire paresseuse manipulée par l'API Snowpark.

**Client-side vs server-side** — *Client* : construction du plan, récupération de petits résultats. *Server* : exécution réelle dans le warehouse (à maximiser).

**UDF** — Fonction définie par l'utilisateur retournant **une valeur par ligne** (SQL/Python/Java/JS).

**UDF vectorisée** — UDF Python travaillant sur des *batches* (pandas Series) → bien plus performante sur gros volumes.

**UDTF** — Fonction de table retournant **plusieurs lignes** (méthode `process` + `yield`).

**UDAF** — Fonction d'agrégation personnalisée (`accumulate` / `merge` / `finish`) retournant **une valeur par groupe**.

**Stored procedure** — Procédure exécutant de la logique (SQL scripting, JavaScript, ou Snowpark) ; `EXECUTE AS OWNER` (droits du propriétaire) ou `CALLER` (droits de l'appelant).

**External function** — Fonction appelant une **API externe** (Lambda/Azure Functions) ; nécessite une *API integration*.

**Snowpark-Optimized Warehouse** — Type de warehouse avec ~16× plus de mémoire par nœud, pour les charges Python/ML mémoire-intensives.

**Snowflake Cortex** — Fonctions IA/LLM **serverless** (`COMPLETE`, `SUMMARIZE`, `SENTIMENT`, `TRANSLATE`, embeddings…).

---

## Éditions Snowflake

**Standard** — Édition de base (Time Travel max 1 j).

**Enterprise** — Ajoute Time Travel jusqu'à 90 j, Materialized Views, multi-cluster, etc.

**Business Critical** — Renforce la sécurité/conformité (HIPAA, PCI, PrivateLink, Tri-Secret Secure).

**Virtual Private Snowflake (VPS)** — Isolation maximale (environnement dédié).

---

!!! tip "Comment utiliser ce lexique"
    En préparation : relis-le par thème pour ancrer le vocabulaire. En entreprise : sers-t'en comme référence rapide pour cadrer une discussion technique ou onboarder un collègue.
