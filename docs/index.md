# ❄️ SnowPro Certifications — Documentation Personnelle

Documentation basée sur les **study guides officiels Snowflake** (janvier–mai 2026).

---

## 🎯 Les 3 certifications ciblées

| Certification | Code | Poids | Prérequis |
|---|---|---|---|
| **SnowPro Core** | COF-C03 | — | 6 mois d'expérience Snowflake |
| **SnowPro Advanced: Data Engineer** | DEA-C02 | — | **COF-C03 obligatoire** + 2 ans d'expérience |
| **SnowPro Specialty: Snowpark** | SPS-C01 | — | COF-C03 recommandé + 1 an Snowpark |

!!! danger "Ordre obligatoire"
    Tu **dois** obtenir le COF-C03 avant de pouvoir t'inscrire au DEA-C02. C'est un prérequis officiel.

---

## 📊 Domaines & Poids officiels

=== "COF-C03 Core"
    | Domaine | Poids |
    |---|---|
    | D1 — Snowflake AI Data Cloud Features & Architecture | **31%** |
    | D2 — Account Management & Data Governance | **20%** |
    | D3 — Data Loading, Unloading & Connectivity | 18% |
    | D4 — Performance Optimization, Querying & Transformation | **21%** |
    | D5 — Data Collaboration | 10% |

=== "DEA-C02 Data Engineer"
    | Domaine | Poids |
    |---|---|
    | D1 — Data Movement | **28%** |
    | D2 — Performance Optimization | 19% |
    | D3 — Storage & Data Protection | 14% |
    | D4 — Data Governance | 14% |
    | D5 — Data Transformation | **25%** |

=== "SPS-C01 Snowpark"
    | Domaine | Poids |
    |---|---|
    | D1 — Snowpark Concepts | 15% |
    | D2 — Snowpark API for Python | **30%** |
    | D3 — Snowpark for Data Transformations | **35%** |
    | D4 — Snowpark Performance Optimization | 20% |

---

## ⚠️ Points critiques à maîtriser

!!! danger "Sujets très fréquents tous examens confondus"
    - Architecture 3 couches (Storage / Compute / Cloud Services)
    - Micro-partitions, clustering depth, pruning
    - Time Travel vs Fail-safe (durées, accès, coûts)
    - RBAC : hiérarchie des rôles, DAC, Database Roles
    - COPY INTO vs Snowpipe vs Snowpipe Streaming
    - Streams + Tasks : pattern CDC, SYSTEM$STREAM_HAS_DATA
    - Dynamic Data Masking, Row Access Policies, Projection Policies
    - Snowpark : lazy evaluation, DataFrames, UDFs vectorisées
    - Cortex AI : fonctions SQL, Cortex Search, Cortex Analyst
    - Resource Monitors (aucun crédit supplémentaire !)
    - Trust Center, DAC vs RBAC
