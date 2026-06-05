# FAQ & Pièges d'examen

## Questions qui tombent souvent

### Quelle édition pour Dynamic Data Masking ?
**Enterprise minimum** — pas Standard.

### Quelle édition pour Row Access Policies ?
**Enterprise minimum** — pas Standard.

### Quelle édition pour Time Travel 90 jours ?
**Enterprise minimum** — Standard = 1 jour max.

### Quelle édition pour les clés gérées par le client ?
**Business Critical** — Tri-secret key (KMS client).

### Fail-safe : qui peut accéder ?
**Snowflake Support UNIQUEMENT** — l'utilisateur ne peut PAS accéder via SQL. Time Travel = user, Fail-safe = Support.

### Resource Monitors consomment-ils des crédits ?
**NON** — aucun crédit supplémentaire. Surveille uniquement.

### Une task est-elle active par défaut ?
**NON** — toujours `ALTER TASK ... RESUME` après création.

### Snowpipe utilise-t-il mon warehouse ?
**NON** — warehouse serverless géré par Snowflake.

### Quel est l'ordre des crédits des warehouses ?
XS=1, S=2, M=4, L=8, XL=16, 2XL=32 — chaque taille **double** la précédente.

### Quand utiliser QAS vs Multi-cluster ?
- **QAS** = accélérer une **requête individuelle lente** (scale up automatique)
- **Multi-cluster** = gérer la **haute concurrence** (beaucoup de requêtes simultanées)

### Quel cache ne nécessite pas de warehouse ?
**Metadata Cache** et **Result Cache** → gérés par Cloud Services → AUCUN crédit.

### Trust Center vs ACCOUNT_USAGE ?
- **Trust Center** = évaluer les recommandations **sécurité** du compte
- **ACCOUNT_USAGE** = analyser l'**utilisation** et la **consommation** de crédits

### DAC vs RBAC ?
- **DAC** = le propriétaire de l'objet décide qui peut y accéder
- **RBAC** = les droits sont définis par les rôles dans la hiérarchie

### COF-C03 requis pour DEA-C02 ?
**OUI** — prérequis **obligatoire** (pas recommandé, obligatoire).

### Snowpark premier paramètre ?
Stored procedure Python = **toujours Session en premier paramètre**.

### Lazy evaluation Snowpark ?
Les transformations ne s'exécutent qu'à une **action** : `.show()`, `.collect()`, `.count()`, `.write`.

### Snowpark-Optimized pour quoi ?
**ML Training** — 16x plus de mémoire par nœud. Pas pour l'inference.

### MFA Caching Snowpark ?
`ALTER ACCOUNT SET ALLOW_CLIENT_MFA_CACHING = TRUE` — le plus sécurisé.
