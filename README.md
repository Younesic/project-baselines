# Project Baselines Repository

Ce repository contient les **baselines Kubernetes standardisées** pour Project-as-Code.

## 🏗️ Architecture Enterprise (Baselines réutilisables)

**Principe :** Les projets sont des instances, les baselines sont des produits.

- ❌ **Pas de baseline par projet**
- ✅ **Baselines par type réutilisables** 
- ✅ **Crossplane pointe vers le bon type**
- ✅ **Une seule PR par projet (XProjectCluster)**

## 📁 Structure

```
project-baselines/
├── baselines/
│   ├── small/              # Projets développement/test
│   │   ├── resourcequota.yaml    # 1 CPU, 2Gi RAM, 5Gi storage
│   │   ├── limitrange.yaml       # Limites containers
│   │   ├── rbac.yaml             # Permissions développeur
│   │   ├── networkpolicy.yaml    # Réseau basic
│   │   └── kustomization.yaml
│   ├── standard/           # Projets production light
│   │   ├── resourcequota.yaml    # 4 CPU, 8Gi RAM, 20Gi storage
│   │   ├── limitrange.yaml
│   │   ├── rbac.yaml
│   │   ├── networkpolicy.yaml
│   │   └── kustomization.yaml
│   ├── large/              # Projets critiques
│   │   └── ...
│   └── restricted/         # Projets sécurisés/compliance
│       └── ...
└── README.md
```

## 🔄 Workflow Enterprise

### 1. Tech Lead crée projet
```yaml
apiVersion: platform.example.com/v1
kind: XProjectCluster
metadata:
  name: ecommerce-prod
spec:
  parameters:
    projectName: ecommerce
    teamName: backend-team
    environment: prod
    projectType: standard    # ← Sélectionne le baseline
```

### 2. Validation et merge
- **Tech Lead/EM** valide la PR XProjectCluster
- **Une seule validation** par projet

### 3. Déploiement automatique
1. **ArgoCD** sync → Applique XProjectCluster
2. **Crossplane** → Crée namespace + ArgoCD Application  
3. **Application ArgoCD** pointe vers `baselines/standard/`
4. **ArgoCD sync baseline** → Applique ResourceQuota, RBAC, NetPol

## ⚖️ Gouvernance

**Baselines (validées par Platform Team) :**
- Auditées et sécurisées
- Rarement modifiées
- Réutilisées des centaines de fois

**Projets (validés par Tech Leads) :**
- Déclaration d'intention
- Sélection du type approprié
- Validation business/technique

## 🎯 Types de baseline

| Type | Usage | CPU | Memory | Storage | Pods |
|------|-------|-----|--------|---------|------|
| **small** | Dev/Test | 1 CPU | 2Gi | 5Gi | 10 |
| **standard** | Prod Light | 4 CPU | 8Gi | 20Gi | 30 |
| **large** | Critique | 8+ CPU | 16+ Gi | 50+ Gi | 50+ |
| **restricted** | Compliance | Custom + Security policies | - | - |

## ✅ Avantages Architecture

- ✅ **UNE SEULE PR** par projet
- ✅ **Pas de scaffolding complexe**
- ✅ **Baselines auditées et standardisées**
- ✅ **Scaling optimal** (réutilisation)
- ✅ **GitOps pur** (reconstruction depuis Git)
- ✅ **Gouvernance claire** (Platform vs Projects)

## 🚀 Migration

Cette architecture **remplace définitivement** :
- ❌ Scaffolding par projet GitHub Actions
- ❌ Génération dynamique de baselines
- ❌ Double validation (PR projet + PR baseline)
- ❌ Complexité workflow