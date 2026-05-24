# Data Governance (10%) — DEA-C02

## Object Tagging & Classification ⭐

### Création et application de tags

```sql
-- Créer un tag avec valeurs autorisées
CREATE TAG sensibilite
  ALLOWED_VALUES 'PII', 'Confidentiel', 'Interne', 'Public';

-- Appliquer sur différents objets
ALTER TABLE clients MODIFY COLUMN email     SET TAG sensibilite = 'PII';
ALTER TABLE clients MODIFY COLUMN telephone SET TAG sensibilite = 'PII';
ALTER TABLE produits                         SET TAG sensibilite = 'Interne';
ALTER SCHEMA ma_db.finance                   SET TAG sensibilite = 'Confidentiel';

-- Voir les tags appliqués
SELECT * FROM TABLE(SNOWFLAKE.INFORMATION_SCHEMA.TAG_REFERENCES(
    'MA_DB.PUBLIC.CLIENTS', 'TABLE'
));

-- Rechercher toutes les colonnes avec un tag
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.TAG_REFERENCES
WHERE TAG_NAME = 'SENSIBILITE'
  AND TAG_VALUE = 'PII';
```

### Classification automatique

Snowflake peut détecter automatiquement les données sensibles (emails, téléphones, numéros de carte…).

```sql
-- Lancer la classification sur une table
SELECT SYSTEM$CLASSIFY('MA_DB.PUBLIC.CLIENTS', NULL);

-- Voir les recommandations
SELECT * FROM TABLE(RESULT_SCAN(LAST_QUERY_ID()));

-- Appliquer les recommandations automatiquement
SELECT SYSTEM$CLASSIFY_SCHEMA('MA_DB.PUBLIC', NULL);
```

---

## Data Lineage ⭐

Traçabilité automatique des flux de données.

```sql
-- Voir les dépendances d'objets
SELECT referenced_object_name,
       referenced_object_type,
       referencing_object_name,
       referencing_object_type
FROM SNOWFLAKE.ACCOUNT_USAGE.OBJECT_DEPENDENCIES
WHERE referenced_object_name = 'VENTES'
ORDER BY referencing_object_name;
```

Visualisation graphique disponible dans **Snowsight → Governance → Data Lineage**.

---

## Projection Policies

Empêche certains rôles de sélectionner une colonne dans les résultats.

```sql
-- Créer une projection policy
CREATE PROJECTION POLICY pp_ssn
  AS () RETURNS PROJECTION_CONSTRAINT ->
  CASE
    WHEN CURRENT_ROLE() IN ('DBA', 'COMPLIANCE') THEN PROJECTION_CONSTRAINT(ALLOW => TRUE)
    ELSE PROJECTION_CONSTRAINT(ALLOW => FALSE)
  END;

-- Appliquer sur une colonne
ALTER TABLE employes MODIFY COLUMN numero_secu
  SET PROJECTION POLICY pp_ssn;

-- Effet : SELECT numero_secu FROM employes → ERROR pour les rôles non autorisés
-- Même avec SELECT * → la colonne est exclue automatiquement
```

!!! info "Différence avec Dynamic Data Masking"
    - **Dynamic Data Masking** → colonne visible mais obfusquée (ex: `***@***.***`)
    - **Projection Policy** → colonne complètement **inaccessible** (erreur si tentative)

---

## Aggregation Policies

Empêche les requêtes qui retournent moins de N lignes (protection contre la ré-identification).

```sql
-- Créer une aggregation policy
CREATE AGGREGATION POLICY ap_min_5
  AS () RETURNS AGGREGATION_CONSTRAINT ->
  AGGREGATION_CONSTRAINT(MIN_GROUP_SIZE => 5);

-- Appliquer sur une table
ALTER TABLE enquetes SET AGGREGATION POLICY ap_min_5;

-- Effet : toute requête GROUP BY retournant < 5 lignes → résultat masqué
```

---

## Data Clean Rooms (avancé)

```sql
-- Utiliser la Clean Room via l'API développeur
CALL samooha_by_snowflake.provider.create_cleanroom('ma_cleanroom');

-- Ajouter une table au clean room
CALL samooha_by_snowflake.provider.link_datasets(
    'ma_cleanroom',
    ['MA_DB.PUBLIC.CLIENTS']
);

-- Activer la differential privacy
CALL samooha_by_snowflake.provider.set_dp_sensitivity(
    'ma_cleanroom',
    'MA_DB.PUBLIC.CLIENTS',
    0.5
);
```

---

## External Tokenization

Remplace les données sensibles par des **tokens** via un service externe (ex: Protegrity, Voltage).

```sql
-- Créer une external function pour la tokenisation
CREATE EXTERNAL FUNCTION tokenize_credit_card(carte STRING)
  RETURNS STRING
  API_INTEGRATION = ma_api_integration
  AS 'https://mon-service-tokenisation.com/tokenize';

-- Utiliser dans une masking policy
CREATE MASKING POLICY masque_carte
  AS (val STRING) RETURNS STRING ->
  CASE
    WHEN CURRENT_ROLE() IN ('PAYMENT_ADMIN') THEN val
    ELSE tokenize_credit_card(val)
  END;
```

---

## Data Quality — Data Metric Functions ⭐

Mesurer automatiquement la qualité des données.

```sql
-- Appliquer des métriques de qualité natives
ALTER TABLE clients ADD DATA METRIC FUNCTION
  SNOWFLAKE.CORE.NULL_COUNT ON (email);

ALTER TABLE clients ADD DATA METRIC FUNCTION
  SNOWFLAKE.CORE.DUPLICATE_COUNT ON (id);

-- Créer une métrique personnalisée
CREATE DATA METRIC FUNCTION mon_taux_vide(t TABLE(col STRING))
  RETURNS NUMBER AS
  'SELECT RATIO_TO_REPORT(SUM(CASE WHEN col IS NULL OR col = '''' THEN 1 ELSE 0 END))
   OVER() FROM t';

-- Voir les résultats
SELECT * FROM SNOWFLAKE.LOCAL.DATA_QUALITY_MONITORING_RESULTS
WHERE table_name = 'CLIENTS';
```
