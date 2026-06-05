# 2.2 Fonctionnalités de Data Governance

> **Domain 2.0 — Account Management & Data Governance (20%)**

## Dynamic Data Masking ⭐ (Enterprise+)

Masque les valeurs **à la lecture** selon le rôle.

```sql
CREATE MASKING POLICY mask_email AS (val STRING) RETURNS STRING ->
  CASE WHEN CURRENT_ROLE() IN ('PII_READER') THEN val
       ELSE REGEXP_REPLACE(val, '.+@', '***@') END;

ALTER TABLE clients MODIFY COLUMN email SET MASKING POLICY mask_email;
```

## Row Access Policies (sécurité ligne)

```sql
CREATE ROW ACCESS POLICY rap_region AS (region STRING) RETURNS BOOLEAN ->
  EXISTS (SELECT 1 FROM mapping_roles m
          WHERE m.role = CURRENT_ROLE() AND m.region = region);

ALTER TABLE ventes ADD ROW ACCESS POLICY rap_region ON (region);
```

## Autres briques de gouvernance

| Brique | Rôle |
|---|---|
| **Object tagging** | Tags clé/valeur sur objets/colonnes (classification, coûts) |
| **Privacy policies** | Politiques de confidentialité / agrégation |
| **Trust Center** | Tableau de bord posture sécurité (scanners, recommandations) |
| **Data lineage** | Traçabilité des dépendances entre objets |
| **Encryption key management** | Chiffrement de bout en bout, hiérarchie de clés, tri-secret (Business Critical) |
| **Replication & failover** | Continuité d'activité inter-régions/clouds |
| **Alerts & notifications** | Surveillance et déclenchement d'actions |

!!! danger "Édition requise"
    Dynamic Data Masking et Row Access Policies → **Enterprise minimum**. Tri-secret key → **Business Critical**.

📎 *Réf. : `docs.snowflake.com/en/user-guide/governance-overview`*
