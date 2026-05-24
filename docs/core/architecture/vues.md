# Types de Vues

## Comparatif des 3 types ⭐

| | Vue Standard | Vue Matérialisée | Vue Sécurisée |
|---|---|---|---|
| **Stockage** | Aucun (logique) | Résultats physiques | Aucun (logique) |
| **Performance** | Recalculée à chaque requête | Résultats pré-calculés | Recalculée à chaque requête |
| **DDL visible** | ✅ Oui | ✅ Oui | ❌ Masqué |
| **Utilisation partage** | ❌ | ❌ | ✅ Recommandée |
| **Coût maintenance** | Aucun | Crédits + stockage | Aucun |
| **Refresh automatique** | N/A | ✅ Automatique | N/A |

---

## Vue Standard

```sql
CREATE VIEW vue_ventes_2024 AS
SELECT id, date_vente, montant, region
FROM ventes
WHERE YEAR(date_vente) = 2024;
```

- Aucun stockage physique
- Exécute la requête sous-jacente à chaque appel
- DDL visible par tous les utilisateurs avec accès

---

## Vue Matérialisée ⭐

Stocke physiquement les résultats pré-calculés. Snowflake maintient la vue à jour automatiquement.

```sql
CREATE MATERIALIZED VIEW mv_ventes_region AS
SELECT region,
       DATE_TRUNC('month', date_vente) AS mois,
       SUM(montant)                    AS total,
       COUNT(*)                        AS nb
FROM ventes
GROUP BY 1, 2;
```

!!! warning "Limitations"
    - Ne supporte pas : `HAVING`, sous-requêtes, `JOIN`, `LIMIT`, `ORDER BY`
    - Coût : crédits de maintenance + stockage
    - Nécessite édition **Enterprise minimum**

!!! tip "Quand utiliser ?"
    Requêtes analytiques lourdes exécutées très fréquemment sur des données qui changent peu souvent.

---

## Vue Sécurisée ⭐

Masque la définition SQL de la vue — idéale pour le **partage de données**.

```sql
-- Le consommateur ne peut PAS voir la requête sous-jacente
CREATE SECURE VIEW vue_clients_publique AS
SELECT id, prenom, ville
FROM clients
WHERE statut = 'actif';

-- Combinaison : Matérialisée + Sécurisée
CREATE SECURE MATERIALIZED VIEW smv_stats AS
SELECT region, COUNT(*) AS nb_clients
FROM clients
GROUP BY region;
```

!!! danger "Piège exam"
    Pour partager via un **Share Snowflake**, toujours utiliser des **vues sécurisées** — jamais des vues standard (le DDL serait visible par le consommateur).

---

## Vue Dynamique (Dynamic Table)

Différente des vues — c'est une **table physique** qui se rafraîchit automatiquement.

```sql
CREATE DYNAMIC TABLE dt_resume
  TARGET_LAG = '10 minutes'
  WAREHOUSE  = wh_etl
AS
SELECT region, SUM(montant) AS total
FROM ventes
GROUP BY region;
```

| | Vue Matérialisée | Table Dynamique |
|---|---|---|
| Refresh | Automatique (Snowflake) | Configurable (TARGET_LAG) |
| Transformations complexes | Limitées | Toutes supportées |
| Chaînage | ❌ | ✅ (DOWNSTREAM) |
