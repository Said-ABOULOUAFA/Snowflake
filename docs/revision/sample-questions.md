# Questions types officielles (avec réponses)

## COF-C03 — SnowPro Core

### Q1 — Data Classification
**Un analyste de conformité veut détecter et étiqueter automatiquement les données sensibles.**
**Quelle fonctionnalité permet cela directement ?**

- A. Data classification ✅
- B. Object tagging
- C. Dynamic Data Masking
- D. Row access policy

**Réponse : A** — Data Classification détecte et classe automatiquement les données PII, financières, etc.

---

### Q2 — Resource Monitors
**Comment les Resource Monitors affectent-ils la consommation de crédits ?**

- A. Consomment des crédits pendant le suivi
- B. Frais mensuel fixe par compte
- C. Suivent l'usage SANS coût supplémentaire ✅
- D. Crédits facturés par alerte/suspension

**Réponse : C** — Resource Monitors ne consomment AUCUN crédit additionnel.

---

### Q3 — Trust Center
**Un administrateur veut évaluer son compte contre les recommandations de sécurité.**

- A. Trust Center ✅
- B. ACCOUNT_USAGE schema
- C. Access History
- D. Network Policies

**Réponse : A** — Trust Center = tableau de bord sécurité avec recommandations.

---

### Q4 — DAC vs RBAC
**Quel modèle permet au propriétaire d'un objet d'accorder l'accès à cet objet ?**

- A. Discretionary Access Control (DAC) ✅
- B. Role-Based Access Control (RBAC)
- C. User-Based Access Control (UBAC)
- D. Role hierarchy

**Réponse : A** — DAC = le propriétaire contrôle qui peut accéder à ses objets.

---

### Q5 — Snowpark
**Exécuter Python/Java natif dans Snowflake pour une UDF Java et une fonction Python.**

- A. External function
- B. Snowpark ✅
- C. Stored procedure
- D. Cortex Complete

**Réponse : B** — Snowpark exécute du code non-SQL directement dans Snowflake.

---

## DEA-C02 — Data Engineer

### Q1 — COPY INTO / FORCE
**Recharger des fichiers > 64 jours sans dupliquer les données déjà présentes.**

- A. FORCE = TRUE → recharge tout → duplications possibles ❌
- B. DELETE + COPY INTO → perte de données existantes ❌
- C. LOAD_UNCERTAIN_FILES = TRUE ✅
- D. TRUNCATE + COPY INTO → perte totale ❌

**Réponse : C** — `LOAD_UNCERTAIN_FILES = TRUE` force le rechargement des fichiers > 64j sans toucher aux autres.

---

### Q2 — Time Travel par schéma
**Besoin : SALES = 10 jours, autres schémas = 1 jour. Solution la plus économique.**

- A. Default settings (Enterprise = 1j par défaut) → ne couvre pas 10j ❌
- B. ALTER ACCOUNT → applique 10j à TOUT → trop cher ❌
- C. ALTER SCHEMA SALES SET DATA_RETENTION_TIME_IN_DAYS = 10 ✅
- D. ALTER DATABASE → applique 10j à tous les schémas → trop cher ❌

**Réponse : C** — Appliquer uniquement au schéma SALES = ciblé et économique.

---

### Q3 — TRY_PARSE_JSON
```sql
SELECT id, try_parse_json(v) FROM vartab ORDER BY id;
```
| id | v |
|---|---|
| 1 | `[-1, 12, 289, 2188, false,]` (tableau valide) |
| 2 | `{ "x" : "abc", ...}` (objet valide) |
| 3 | `[ "x" : "def",...` (JSON invalide) |
| 4 | `[-1, 12, 289, 2188], NULL` (VARCHAR = NULL) |

**Résultat B :**
```
1 | [ -1, 12, 289, 2188, false, undefined ]
2 | { "x": "abc", "y": false, "z": 10 }
3 | NULL    ← TRY_PARSE_JSON retourne NULL sur JSON invalide
4 | NULL    ← VARCHAR contient le texte "NULL" → TRY_PARSE_JSON retourne NULL
```

**Réponse : B**

---

### Q4 — Task Timeout
**Une task échoue après ajout de données historiques (durée > limite).**

- A. Durée par défaut = **60 min**, recréer avec `USER_TASK_TIMEOUT_MS` plus élevé ✅
- B. Durée 120 min, changer TRIGGER_INTERVAL ❌
- C. Durée 60 min, changer TRIGGER_INTERVAL ❌
- D. Durée 120 min, changer TIMEOUT_MS ❌

**Réponse : A** — Timeout par défaut = 60 minutes, paramètre = `USER_TASK_TIMEOUT_MS`

---

### Q5 — Récupérer table avec UNDROP
**Table SALES recréée par CREATE OR REPLACE. Récupérer les données originales.**

- A. Time Travel AT timestamp → SALES existe déjà ❌ (nom en conflit)
- B. UNDROP → DELETE → INSERT AT OFFSET ❌ (OFFSET approximatif)
- C. RENAME TO SALES_new → UNDROP TABLE SALES ✅ (libère le nom, undrop restaure l'ancienne)
- D. RENAME + UNDROP + INSERT BEFORE(STATEMENT) ❌ (inutile si undrop suffit)

**Réponse : C** — Renommer la table actuelle libère le nom → UNDROP restaure l'ancienne.

---

## SPS-C01 — Snowpark

### Q1 — Snowpark-Optimized Warehouse
**Quel workload bénéficie le PLUS d'un Snowpark-Optimized Warehouse ?**

- A. Machine learning **training** ✅
- B. ML inference ❌ (Standard suffit)
- C. Registering a model into Model Registry ❌
- D. Creating a compute pool ❌

**Réponse : A** — Le training ML nécessite beaucoup de mémoire → Snowpark-Optimized.

---

### Q2 — UDF sans IMPORTS
**Que peut faire un Specialist avec une Python UDF sans clause IMPORTS ?**

- A. Partager la UDF directement ❌
- B. Partager une vue qui appelle la UDF ❌ (possible mais pas la réponse)
- C. Accéder à l'objet session depuis la UDF ❌ (session non disponible dans UDF)
- D. Accorder USAGE sur la UDF à un rôle ✅

**Réponse : D** — GRANT USAGE ON FUNCTION est toujours possible.

---

### Q3 — group_by + agg
**Résumer les ventes par produit depuis un DataFrame (product_id, quantity).**

- A. `df.sum("quantity").group_by("product_id")` ❌
- B. `df.summarize("quantity").over("product_id")` ❌
- C. `df.group_by("product_id").agg(sum("quantity"))` ✅
- D. `df.agg("quantity", type="sum").group_by("product_id")` ❌

**Réponse : C** — `group_by().agg()` = syntaxe Snowpark correcte.

---

### Q4 — Stored Procedure — premier paramètre
**Que doit considérer un Specialist pour définir une fonction Python comme stored procedure ?**

- A. Doit toujours retourner un DataFrame ❌
- B. Le **premier paramètre doit être un objet Session** ✅
- C. Le décorateur @sproc doit toujours être utilisé ❌
- D. pandas DataFrame peut être paramètre ❌

**Réponse : B** — La Session est TOUJOURS le premier paramètre d'une stored procedure Snowpark.

---

### Q5 — MFA Caching
**Réduire les alertes MFA constantes pour les utilisateurs Snowpark.**

- A. Network Policy ❌
- B. `ALLOW_CLIENT_MFA_CACHING = TRUE` ✅
- C. Passcode dans la session ❌
- D. Désactiver MFA ❌ (non sécurisé)

**Réponse : B** — MFA Caching = la méthode la plus sécurisée pour réduire les prompts.
