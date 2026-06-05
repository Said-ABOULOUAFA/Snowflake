# ❄️ SnowPro Certifications — Documentation personnelle

Documentation structurée d'après les **study guides officiels Snowflake** (versions 2025–2026), organisée par **certification → domaine → sujet**.

!!! abstract "Comment naviguer dans ce site"
    Utilise le **menu du haut** (onglets COF-C03 · DEA-C02 · SPS-C01) et le **sommaire latéral** pour passer d'un sujet à l'autre. Chaque page suit la numérotation officielle de l'examen.

    - 🎓 **Pendant la préparation** : suis l'ordre des domaines, puis révise avec les **Cheat Sheets** et la **FAQ**.
    - 🏢 **En entreprise** : garde le **[📖 Lexique](revision/lexique.md)** sous la main comme référence rapide, et les pages par domaine comme aide-mémoire technique.
    - 🔎 La **barre de recherche** (en haut) indexe l'intégralité du site.

---

## 🎯 Les 3 certifications ciblées

| Certification | Code | Questions | Durée | Score | Prix |
|---|---|---|---|---|---|
| **SnowPro Core** | COF-C03 | 100 | 115 min | 750/1000 | 175 $ |
| **SnowPro Advanced: Data Engineer** | DEA-C02 | 65 | 115 min | 750/1000 | 375 $ |
| **SnowPro Specialty: Snowpark** | SPS-C01 | 55 | 85 min | 750/1000 | 225 $ |

---

## 🔐 Prérequis officiels (très important)

!!! danger "Ordre d'enchaînement imposé par Snowflake"
    Le **SnowPro Core (COF-C03) est le prérequis obligatoire** de **toutes** les certifications Advanced et Specialty. Sans Core en cours de validité, l'inscription au DEA-C02 et au SPS-C01 est **bloquée**.

| Certification | Prérequis formel | Expérience recommandée |
|---|---|---|
| **COF-C03** | *Aucun prérequis formel* | **6 mois minimum** d'utilisation de Snowflake + ANSI SQL de base + notions de cloud |
| **DEA-C02** | **SnowPro Core certifié (valide)** | **2 ans** d'expérience data engineering en production, dont du Snowflake |
| **SPS-C01** | **SnowPro Core certifié (valide)** | **1 an** de pratique Snowpark + Python solide (client-side / server-side) |

```
COF-C03 (Core)  ──►  DEA-C02 (Data Engineer)
       │
       └──────────►  SPS-C01 (Snowpark)
```

!!! info "Validité & renouvellement"
    Chaque certification est valable **2 ans**. Renouvellement via le programme Continuing Education (CE) ou en obtenant une certification de niveau égal/supérieur.

---

## 📚 Poids des domaines par examen

=== "COF-C03 (Core)"

    | # | Domaine | Poids |
    |---|---|---|
    | 1.0 | Snowflake AI Data Cloud Features & Architecture | **31%** |
    | 4.0 | Performance Optimization, Querying & Transformation | **21%** |
    | 2.0 | Account Management & Data Governance | 20% |
    | 3.0 | Data Loading, Unloading & Connectivity | 18% |
    | 5.0 | Data Collaboration | 10% |

=== "DEA-C02 (Data Engineer)"

    | # | Domaine | Poids |
    |---|---|---|
    | 1.0 | Data Movement | **28%** |
    | 5.0 | Data Transformation | **25%** |
    | 2.0 | Performance Optimization | 19% |
    | 3.0 | Storage & Data Protection | 14% |
    | 4.0 | Data Governance | 14% |

=== "SPS-C01 (Snowpark)"

    | # | Domaine | Poids |
    |---|---|---|
    | 3.0 | Snowpark for Data Transformations | **35%** |
    | 2.0 | Snowpark API for Python | **30%** |
    | 4.0 | Snowpark Performance Optimization | 20% |
    | 1.0 | Snowpark Concepts | 15% |

---

## 🔑 Sujets transverses critiques (tombent dans plusieurs examens)

!!! tip "À maîtriser absolument"
    - Architecture 3 couches (stockage / calcul / services cloud)
    - Micro-partitions, pruning, clustering & clustering depth
    - Time Travel, Fail-safe & CDP
    - RBAC, privilèges sur clones et shares
    - COPY INTO vs Snowpipe vs Snowpipe Streaming
    - Streams, Tasks, Dynamic Tables (pipelines continus)
    - Dynamic Data Masking & Row Access Policies
    - Snowpark : exécution lazy / pushdown client vs serveur

---

## 📎 Sources officielles

- Study guide **COF-C03** : `learn.snowflake.com/en/certifications/snowpro-core-c03/`
- Study guide **DEA-C02** : `learn.snowflake.com` → SnowPro Advanced: Data Engineer
- Page **SPS-C01** : `learn.snowflake.com/en/certifications/snowpro-snowpark/`
- Documentation produit : `docs.snowflake.com`

!!! note "Conseil d'utilisation"
    Commence par la **Vue d'ensemble** de chaque certification (prérequis + format), puis descends domaine par domaine. Les **Cheat Sheets** en fin de doc servent à la révision finale (J-3).
