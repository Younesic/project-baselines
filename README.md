# Project Baselines Repository

Ce repository contient les **baselines Kubernetes** pour les projets créés via Project-as-Code.

## 🏗️ Architecture Enterprise (repos séparés)

**`project-baselines`** (ici) = Standards plateforme  
**`project-manifests`** (séparé) = Déclarations projets XProjectCluster

Recommandation expert : Séparation des responsabilités avec gouvernance CODEOWNERS.

## 📁 Structure

```
project-baselines/
├── environments/           # Baselines Kubernetes par projet
│   └── {environment}/
│       └── {project}/
│           ├── resourcequota.yaml
│           ├── rbac.yaml
│           └── kustomization.yaml
├── .github/
│   ├── ISSUE_TEMPLATE/     # Templates pour demandes
│   └── workflows/          # Automation cross-repository
└── README.md
```

## 🔄 Workflow Enterprise (Cross-Repository)

### 1. Demande développeur
**Option A - Workflow manuel** (pour tester):
1. Aller sur l'onglet "Actions" du repository GitHub  
2. Sélectionner "🚀 Scaffold New Project"
3. Renseigner : nom projet, équipe, environnement

**Option B - Issue template** (workflow complet):
1. Créer une issue avec template "🚀 Demande de nouveau projet"
2. Admin valide et lance le workflow

### 2. Génération automatique (2 PRs)

Le workflow GitHub Actions crée **2 Pull Requests** séparées :

**PR 1 - project-baselines** (validée par @platform-team):
- Baseline `environments/{env}/{project}/`
- ResourceQuota avec limites standard  
- RBAC (ServiceAccount + Role + RoleBinding)

**PR 2 - project-manifests** (validée par équipe + @platform-team):
- Manifeste XProjectCluster `{env}/{project}.yaml`
- Configuration Crossplane pointant vers project-baselines

### 3. Validation et déploiement

1. **Review séparé** selon CODEOWNERS
2. **Merge des 2 PRs** par les validateurs appropriés
3. **ArgoCD sync automatique** :
   - XProjectCluster → Crossplane crée namespace + ArgoCD app
   - Baselines synchronisées depuis project-baselines

## ⚖️ Gouvernance

**Séparation des responsabilités** :
- **Équipes produit** : Déclarent leurs projets (XProjectCluster)
- **Équipe plateforme** : Contrôle les standards (baselines)

**RBAC GitOps** :
- project-baselines : @platform-team uniquement
- project-manifests : équipes + @platform-team (CODEOWNERS)

## 🎯 ArgoCD Structure

**ApplicationSet par repository** :
- App "project-manifests-sync" → Lit les XProjectCluster
- App "baseline-sync" → Applique les baselines
- Apps générées par Crossplane → Sync baseline spécifique

## 🚀 Évolution Backstage  

Cette architecture est optimale pour Backstage :
- Plugin scaffold dans project-manifests
- Templates contrôlés plateforme 
- Ownership clair par repository
- Workflow de validation intégré

## 📋 Exemple

Projet `demo-dev` disponible pour test du système complet.