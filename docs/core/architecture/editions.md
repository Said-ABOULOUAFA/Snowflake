# Éditions Snowflake

## Comparatif des éditions ⭐

| Fonctionnalité | Standard | Enterprise | Business Critical | Virtual Private |
|---|---|---|---|---|
| **Time Travel** | 1 jour | 90 jours | 90 jours | 90 jours |
| **Multi-cluster warehouse** | ❌ | ✅ | ✅ | ✅ |
| **Dynamic Data Masking** | ❌ | ✅ | ✅ | ✅ |
| **Row Access Policies** | ❌ | ✅ | ✅ | ✅ |
| **Chiffrement tri-secret** | ❌ | ❌ | ✅ | ✅ |
| **AWS PrivateLink** | ❌ | ❌ | ✅ | ✅ |
| **HIPAA / PCI-DSS** | ❌ | ❌ | ✅ | ✅ |
| **Environnement isolé** | ❌ | ❌ | ❌ | ✅ |

!!! danger "Questions très fréquentes aux examens"
    - Dynamic Data Masking → nécessite **Enterprise minimum**
    - Time Travel 90 jours → nécessite **Enterprise minimum**
    - Tri-secret key (clé client) → nécessite **Business Critical**
    - HIPAA / données de santé → nécessite **Business Critical**

---

## Multi-cluster Warehouse (Enterprise+)

Permet de **scaler horizontalement** en ajoutant des clusters automatiquement.

```sql
CREATE WAREHOUSE wh_analytics
  WAREHOUSE_SIZE = 'LARGE'
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 5
  SCALING_POLICY = 'STANDARD'  -- ou 'ECONOMY'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE;
```

| Politique | Comportement |
|---|---|
| **STANDARD** | Ajoute des clusters dès qu'une requête attend |
| **ECONOMY** | Attend plus longtemps avant d'ajouter un cluster |
