# 2.2 — Snowpark avec données non structurées

> **Domaine D2 API Python — 30% du SPS-C01**

## SnowflakeFile object ⭐

Permet de lire des fichiers depuis un stage dans des UDFs.

```python
from snowflake.snowpark.files import SnowflakeFile

# Dans une UDF — lecture d'un fichier
@udf(packages=["snowflake-snowpark-python"])
def lire_fichier_texte(chemin: str) -> str:
    with SnowflakeFile.open(chemin, 'r') as f:
        return f.read()

# Lire un fichier binaire (PDF, image...)
@udf(packages=["snowflake-snowpark-python", "pypdf2"])
def extraire_texte_pdf(chemin: str) -> str:
    import PyPDF2, io
    with SnowflakeFile.open(chemin, 'rb') as f:
        reader = PyPDF2.PdfReader(io.BytesIO(f.read()))
        return ' '.join(p.extract_text() for p in reader.pages)

# Utiliser dans SQL
# SELECT extraire_texte_pdf(BUILD_SCOPED_FILE_URL(@stage, 'doc.pdf'))
```

## UDFs et UDTFs pour traiter des fichiers ⭐

```python
# UDTF qui lit un CSV ligne par ligne depuis un stage
from snowflake.snowpark.files import SnowflakeFile
import csv, io

class ParseCsvUDTF:
    def process(self, file_url: str):
        with SnowflakeFile.open(file_url, 'r') as f:
            reader = csv.DictReader(io.StringIO(f.read()))
            for row in reader:
                yield (row.get('id'), row.get('nom'), float(row.get('montant', 0)))

session.udtf.register(
    ParseCsvUDTF,
    output_schema=StructType([
        StructField("id", StringType()),
        StructField("nom", StringType()),
        StructField("montant", FloatType())
    ]),
    input_types=[StringType()],
    name="parse_csv_fichier"
)
```

## Procédures stockées pour traiter des fichiers ⭐

```python
def traiter_fichiers_stage(session: Session, stage_path: str) -> str:
    from snowflake.snowpark.files import SnowflakeFile

    # Lister les fichiers du stage
    fichiers = session.sql(f"LIST {stage_path}").collect()

    total_lignes = 0
    for fichier in fichiers:
        path = fichier['name']
        url  = session.sql(f"SELECT BUILD_SCOPED_FILE_URL('{stage_path}', '{path}')").collect()[0][0]

        with SnowflakeFile.open(url, 'r') as f:
            contenu = f.read()
            lignes  = contenu.count('\n')
            total_lignes += lignes

    return f"{len(fichiers)} fichiers traités, {total_lignes} lignes au total"

session.sproc.register(
    traiter_fichiers_stage,
    packages=["snowflake-snowpark-python"],
    is_permanent=True, stage_location="@mon_stage", replace=True
)
```
