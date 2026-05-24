# 📋 Cheat Sheet — SnowPro Data Engineer

## Domaines & poids

| Domaine | Poids | Focus |
|---|---|---|
| Data Transformation | **30%** | Dynamic tables, Snowpark, MERGE, UDFs |
| Data Movement | **28%** | COPY INTO, Snowpipe, Streaming, Kafka |
| Performance | **22%** | Query Profile, Clustering, Warehouse sizing |
| Storage & Protection | 10% | Time Travel, Fail-safe, Réplication |
| Security | 10% | Masking, Row Access, Network policies |

---

## Data Movement — Réponses rapides

| Question | Réponse |
|---|---|
| Chargement avec transformation inline ? | **COPY INTO + SELECT** |
| Tester sans charger ? | `VALIDATION_MODE = 'RETURN_ERRORS'` |
| Snowpipe serverless ? | **Oui** |
| Kafka vers Snowflake temps réel ? | **Snowflake Connector for Kafka** (streaming mode) |
| Diagnostiquer erreurs Snowpipe ? | `VALIDATE_PIPE_LOAD()` |
| Historique chargements ? | `INFORMATION_SCHEMA.COPY_HISTORY` |

---

## Data Transformation — Réponses rapides

| Question | Réponse |
|---|---|
| Transformation déclarative auto ? | **Dynamic Tables** |
| `TARGET_LAG = DOWNSTREAM` ? | Se synchronise avec tables dynamiques en aval |
| Pattern CDC Snowflake natif ? | **Stream + Task** |
| Upsert SQL ? | **MERGE INTO** |
| UDF qui retourne plusieurs lignes ? | **UDTF** (User-Defined Table Function) |
| Snowpark langages supportés ? | **Python, Java, Scala** |

---

## Performance — Réponses rapides

| Question | Réponse |
|---|---|
| Requête lente + spillage ? | **Augmenter taille warehouse** |
| Trop de requêtes en queue ? | **Multi-cluster** (scale out) |
| Requêtes ponctuelles (equality) ? | **Search Optimization** |
| Requêtes analytiques (range filter) ? | **Clustering Key** |
| Vérifier pruning ? | Query Profile → % partitions scanned |
| Clustering depth idéale ? | **Proche de 1** |
| Vue pré-calculée et stockée ? | **Materialized View** |

---

## Security — Réponses rapides

| Question | Réponse |
|---|---|
| Masquer colonnes selon rôle ? | **Dynamic Data Masking** |
| Filtrer lignes selon rôle ? | **Row Access Policy** |
| Masking = édition minimum ? | **Enterprise** |
| Restreindre par IP ? | **Network Policy** |
| Connexion sans Internet public ? | **PrivateLink** (Business Critical) |

---

## Storage & Protection — Réponses rapides

| Question | Réponse |
|---|---|
| Réplication entre régions ? | `ALTER DATABASE ... ENABLE REPLICATION` |
| Failover base de données ? | `ALTER DATABASE replica PRIMARY` |
| CDP = ? | Time Travel + Fail-safe |
| Coût stockage CDP ? | Actif + Time Travel + Fail-safe |

---

## Anti-patterns à éviter (pièges exam)

❌ Utiliser `ACCOUNTADMIN` pour les tâches quotidiennes
❌ Oublier `ALTER TASK ... RESUME` après création
❌ `SELECT *` sur de grandes tables sans filtre
❌ Appliquer une fonction sur une colonne filtrée (ex: `WHERE YEAR(date) = 2024`)
❌ Créer un clone et croire que les GRANTs sont copiés
❌ Suspendre un warehouse fréquemment pour des requêtes répétitives (perd le cache)
❌ Confondre Fail-safe (support uniquement) avec Time Travel (utilisateur)
❌ Croire que Snowpipe utilise ton propre warehouse (serverless)
