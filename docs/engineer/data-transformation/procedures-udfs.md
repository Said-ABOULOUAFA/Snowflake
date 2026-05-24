# Procédures stockées & UDFs avancées

## Procédures stockées — Langages ⭐

| Langage | Cas d'usage |
|---|---|
| **JavaScript** | Logique conditionnelle, boucles, gestion d'erreurs |
| **Python (Snowpark)** | ML, pandas, traitement avancé |
| **Java/Scala (Snowpark)** | Entreprises Java existantes |
| **SQL** | Séquences de requêtes simples |

---

## Procédure JavaScript ⭐

```sql
CREATE OR REPLACE PROCEDURE archiver_ventes(annee NUMBER)
RETURNS STRING
LANGUAGE JAVASCRIPT
EXECUTE AS CALLER  -- ou OWNER
AS $$
    // Créer la table d'archive si elle n'existe pas
    snowflake.execute({
        sqlText: `CREATE TABLE IF NOT EXISTS ventes_archive_${ANNEE}
                  LIKE ventes`
    });

    // Copier les données
    var result = snowflake.execute({
        sqlText: `INSERT INTO ventes_archive_${ANNEE}
                  SELECT * FROM ventes
                  WHERE YEAR(date_vente) = ${ANNEE}`
    });

    // Compter les lignes insérées
    var count = snowflake.execute({
        sqlText: `SELECT COUNT(*) FROM ventes_archive_${ANNEE}`
    });
    count.next();
    var nb = count.getColumnValue(1);

    // Supprimer les données archivées
    snowflake.execute({
        sqlText: `DELETE FROM ventes WHERE YEAR(date_vente) = ${ANNEE}`
    });

    return `Archive ${ANNEE} créée : ${nb} lignes`;
$$;

-- Appeler la procédure
CALL archiver_ventes(2022);
```

---

## EXECUTE AS CALLER vs OWNER ⭐

| | `EXECUTE AS CALLER` | `EXECUTE AS OWNER` |
|---|---|---|
| **Droits utilisés** | Droits de l'appelant | Droits du propriétaire de la procédure |
| **Sécurité** | Appelant doit avoir accès aux objets | Peut accéder à des objets privés |
| **Cas d'usage** | Opérations sur les données de l'utilisateur | Procédures de gestion système |

---

## Procédure SQL

```sql
CREATE OR REPLACE PROCEDURE nettoyer_staging()
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN
    -- Supprimer les anciens enregistrements
    DELETE FROM staging WHERE date_chargement < DATEADD('day', -7, CURRENT_DATE());

    -- Compacter les petites tables
    ALTER TABLE staging_events CLUSTER BY (date_event);

    RETURN 'Nettoyage terminé';
END;
$$;
```

---

## UDFs — Comparatif ⭐

| Type | Retourne | Langage | Cas d'usage |
|---|---|---|---|
| **UDF scalaire** | 1 valeur par ligne | SQL, Python, Java, JS | Transformation simple |
| **UDTF** (tabulaire) | Plusieurs lignes | Python, Java, JS | Dépliage, parsing |
| **UDAF** (agrégat) | 1 valeur par groupe | Python, Java, JS | Agrégation custom |
| **UDF vectorisée** | 1 valeur par ligne | Python (pandas) | Performance sur gros volumes |

---

## UDTF — User-Defined Table Function ⭐

```sql
-- UDTF SQL : diviser une liste CSV en lignes
CREATE OR REPLACE FUNCTION split_csv(val STRING, sep STRING)
RETURNS TABLE (item STRING)
AS $$
    SELECT value::STRING
    FROM TABLE(SPLIT_TO_TABLE(val, sep))
$$;

-- Utiliser
SELECT t.item
FROM ma_table,
TABLE(split_csv(ma_table.tags, ',')) t;
```

```python
# UDTF Python
CREATE OR REPLACE FUNCTION parse_log_lines(log_text STRING)
RETURNS TABLE (
    timestamp STRING,
    niveau    STRING,
    message   STRING
)
LANGUAGE PYTHON
RUNTIME_VERSION = '3.11'
HANDLER = 'ParseLog'
AS $$
import re

class ParseLog:
    def process(self, log_text):
        pattern = r'(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) \[(\w+)\] (.+)'
        for line in log_text.split('\n'):
            m = re.match(pattern, line)
            if m:
                yield (m.group(1), m.group(2), m.group(3))
$$;
```

---

## Gestion des erreurs dans les procédures ⭐

```sql
-- SQL avec gestion d'exception
CREATE OR REPLACE PROCEDURE charger_securise(table_src STRING, table_dst STRING)
RETURNS STRING
LANGUAGE SQL
AS
$$
DECLARE
    nb_lignes INTEGER;
    msg_erreur STRING;
BEGIN
    -- Essai de chargement
    EXECUTE IMMEDIATE 'INSERT INTO ' || table_dst || ' SELECT * FROM ' || table_src;
    nb_lignes := SQLROWCOUNT;
    RETURN 'OK : ' || nb_lignes || ' lignes chargées';

EXCEPTION
    WHEN OTHER THEN
        msg_erreur := SQLERRM;
        INSERT INTO log_erreurs VALUES (CURRENT_TIMESTAMP(), table_src, msg_erreur);
        RETURN 'ERREUR : ' || msg_erreur;
END;
$$;
```

---

## Bonnes pratiques ⭐

```sql
-- ✅ Toujours spécifier EXECUTE AS
-- ✅ Logger les erreurs dans une table dédiée
-- ✅ Utiliser des paramètres plutôt que la concaténation SQL
-- ✅ Préférer Snowpark Python pour la logique complexe
-- ❌ Ne jamais mettre de mots de passe en dur dans une procédure
-- ❌ Éviter les boucles sur de grands volumes (utiliser SQL set-based)
```
