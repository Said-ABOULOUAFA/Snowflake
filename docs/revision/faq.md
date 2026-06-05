# ❓ FAQ & pièges d'examen

## Prérequis & logistique
??? question "Quels examens exigent SnowPro Core au préalable ?"
    **DEA-C02** et **SPS-C01** requièrent une certification **SnowPro Core (COF-C03) valide**. COF-C03 n'a aucun prérequis formel.

??? question "Quelle est la durée de validité d'une certification SnowPro ?"
    **2 ans**. Une recertification (ou un examen supérieur) renouvelle la validité.

??? question "Le score minimal de passage ?"
    **750 / 1000** pour les trois examens (scaled score).

## Architecture
??? question "Disque partagé vs sans partage ?"
    Snowflake **combine** les deux : stockage central partagé **+** calcul MPP indépendant par nœud.

??? question "Snowflake on-premise ?"
    **Non** — uniquement AWS, Azure, GCP.

## Performance
??? question "Spilling 'to remote storage' dans le Query Profile ?"
    Le warehouse manque de mémoire → **agrandir la taille** du warehouse (le multi-cluster gère la concurrence, pas la mémoire d'une requête).

??? question "Multi-cluster ou warehouse plus grand ?"
    **Multi-cluster** = beaucoup d'utilisateurs **simultanés** (concurrence). **Plus grand** = une requête lourde / spilling.

??? question "Quand utiliser Search Optimization vs Materialized View vs QAS ?"
    Point lookup sélectif → **Search Optimization** ; agrégation répétée → **Materialized View** ; scan ponctuel énorme → **Query Acceleration Service**.

## Storage
??? question "Durée du Fail-safe ?"
    **7 jours**, fixe, non configurable, accessible uniquement par le support Snowflake.

??? question "Un clone hérite-t-il des privilèges de la source ?"
    **Non.** Le propriétaire du clone est le rôle qui exécute le `CREATE ... CLONE`.

??? question "Coût initial d'un zero-copy clone ?"
    **Nul** : partage des micro-partitions (copy-on-write). Le stockage croît avec les écritures divergentes.

## Transformation / Snowpark
??? question "Pourquoi une UDF vectorisée plutôt que scalaire ?"
    Elle traite des **batches** (pandas Series) → bien plus performante sur gros volumes.

??? question "External function : prérequis ?"
    Une **API integration** (objet sécurisé) référençant le proxy/role cloud. Sinon, préférer une UDF Python native (pas d'appel réseau).

??? question "Snowpark est-il eager ou lazy ?"
    **Lazy** : rien ne s'exécute avant une **action** (`show`, `collect`, `count`, `save_as_table`, `to_pandas`).

??? question "EXECUTE AS OWNER vs CALLER ?"
    OWNER (défaut) = droits du propriétaire (exposer une opération sans donner accès aux objets) ; CALLER = droits de l'appelant.

??? question "Quel warehouse pour une UDF Python en out-of-memory ?"
    **Snowpark-Optimized Warehouse** (~16× de mémoire par nœud).

## Gouvernance
??? question "Masking policy vs row access policy ?"
    Masking = transforme la **valeur** d'une colonne selon le rôle ; Row access = filtre les **lignes** visibles.

??? question "Quelle vue pour le data lineage ?"
    `SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY` (objets lus vs modifiés), édition Enterprise+.

---

!!! note "Sujet sensible"
    Ces fiches synthétisent les guides officiels Snowflake. Vérifie toujours les guides à jour (les pondérations et objectifs peuvent évoluer) sur le portail de certification Snowflake.
