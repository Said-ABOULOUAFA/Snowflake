# 🐍 SnowPro Specialty: Snowpark (SPS-C01) — Vue d'ensemble

## Format de l'examen

| Élément | Valeur |
|---|---|
| Code | **SPS-C01** |
| Questions | **55** (choix multiple, sélection multiple, interactif) |
| Durée | **85 minutes** |
| Score requis | **750 / 1000** |
| Prix | 225 $ |
| Langues | Anglais |
| Validité | 2 ans |

## Prérequis officiels

!!! danger "Prérequis bloquant"
    - **SnowPro Core (COF-C03) certifié et valide** : **obligatoire** pour toute certification Specialty.
    - **Recommandé** : **1 an** d'expérience Snowpark + **Python** solide, avec compréhension des opérations **client-side vs server-side**.

!!! info "Profil visé"
    Développeur / data engineer / data scientist qui utilise l'**API Snowpark** (Python, Java, Scala) pour construire des applications data, des transformations DataFrame et optimiser la performance.

## Domaines & poids officiels

| # | Domaine | Poids | Priorité |
|---|---|---|---|
| 3.0 | Snowpark for Data Transformations | **35%** | 🔴 Critique |
| 2.0 | Snowpark API for Python | **30%** | 🔴 Critique |
| 4.0 | Snowpark Performance Optimization | 20% | 🟠 Important |
| 1.0 | Snowpark Concepts | 15% | 🟡 Fondations |

!!! tip "Stratégie"
    Transformations + API Python = **65%** de l'examen. Le cœur du sujet : manipuler des **DataFrames**, comprendre l'**évaluation lazy / pushdown**, et savoir quand le calcul se fait côté serveur vs côté client.

## Idée maîtresse à intégrer

!!! abstract "Pushdown"
    Snowpark traduit ton code Python en **SQL exécuté dans le warehouse Snowflake** : les **données ne sont pas déplacées** vers le client. Les transformations sont **lazy** (plan logique) ; seules les **actions** (`show`, `collect`, `save_as_table`) déclenchent l'exécution.
