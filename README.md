# Todo App GitOps - ArgoCD Manifests Repository

![GitOps](https://img.shields.io/badge/GitOps-ArgoCD-FF6B35)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?logo=helm&logoColor=white)

This repository contains ArgoCD Application manifests for the Todo App project, implementing GitOps deployment patterns using the **App of Apps** methodology.

## 📋 Repository Purpose

This GitOps repository serves as the single source of truth for deployment configurations across different environments. It decouples deployment manifests from application code, enabling:

- **Declarative Infrastructure**: All deployments defined as code
- **Version Control**: Full history of infrastructure changes
- **Automated Deployment**: Continuous deployment via ArgoCD
- **Environment Separation**: Isolated staging and production configurations
- **Audit Trail**: Complete deployment history and rollback capabilities

## 🏗️ App of Apps Pattern

This repository implements ArgoCD's **App of Apps** pattern:

```
Root Application (this repo)
├── Staging Application
│   ├── Source: local-devops-infrastructure repo
│   ├── Path: helm-charts/todo-app
│   ├── Branch: master (continuous deployment)
│   └── Values: values.yaml + values-staging.yaml
└── Production Application
    ├── Source: local-devops-infrastructure repo
    ├── Path: helm-charts/todo-app
    ├── Branch: git tags (v1.0.0, v1.1.0, etc.)
    └── Values: values.yaml + values-prod.yaml
```

## 📁 Repository Structure

```
todo-app-gitops/
├── argocd-manifests/
│   ├── root-application.yaml           # App of Apps root
│   └── environments/
│       ├── staging.yaml               # Staging environment app
│       └── production.yaml            # Production environment app
└── README.md                          # This file
```

## 🔄 Deployment Workflow

### Staging Deployment (Automatic)
1. Code changes pushed to `master` branch in [local-devops-infrastructure](https://github.com/KeremAR/local-devops-infrastructure)
2. Jenkins CI/CD pipeline builds and pushes images
3. ArgoCD automatically syncs staging environment
4. Application deployed to `staging` namespace

### Production Deployment (Tag-based)
1. Git tag created (e.g., `v1.0.0`) in local-devops-infrastructure repo
2. Jenkins pipeline updates `production.yaml` with new tag
3. Changes committed to this GitOps repository
4. ArgoCD syncs production environment with specific version
5. Application deployed to `production` namespace

## 🚀 Quick Start

### Prerequisites
- ArgoCD installed and configured
- Access to Kubernetes cluster
- Proper RBAC permissions

### 1. Deploy Root Application

```bash
# Apply the root application to ArgoCD
kubectl apply -f argocd-manifests/root-application.yaml -n argocd
```

This command will:
- Create the root App of Apps application
- Automatically deploy staging and production child applications
- Set up continuous monitoring of this repository

### 2. Verify Deployment

```bash
# Check ArgoCD applications
kubectl get applications -n argocd

# Expected output:
# NAME                  SYNC STATUS   HEALTH STATUS
# root-app             Synced        Healthy
# staging-todo-app     Synced        Healthy
# production-todo-app  Synced        Healthy
```

### 3. Access ArgoCD UI

Navigate to your ArgoCD installation and verify applications are deployed correctly.

## 🔧 Configuration Details

### Root Application (`root-application.yaml`)
- **Purpose**: Entry point for App of Apps pattern
- **Source**: This GitOps repository
- **Path**: `argocd-manifests/environments`
- **Sync Policy**: Automatic with prune and self-heal

### Staging Application (`environments/staging.yaml`)
- **Target Namespace**: `staging`
- **Source Branch**: `master` (continuous deployment)
- **Helm Values**: `values.yaml` + `values-staging.yaml`
- **Sync Policy**: Automatic with namespace creation

### Production Application (`environments/production.yaml`)
- **Target Namespace**: `production`
- **Source Branch**: Git tags (e.g., `v1.0.0`)
- **Helm Values**: `values.yaml` + `values-prod.yaml`
- **Sync Policy**: Manual (no automatic sync)

## 🔒 Security Considerations

### Image Pull Secrets
Applications require Docker registry secrets in each namespace:

```bash
# Create secrets for staging
kubectl create secret docker-registry github-registry-secret \
  --namespace=staging \
  --docker-server=ghcr.io \
  --docker-username=<GITHUB_USERNAME> \
  --docker-password=<GITHUB_TOKEN>

# Create secrets for production
kubectl create secret docker-registry github-registry-secret \
  --namespace=production \
  --docker-server=ghcr.io \
  --docker-username=<GITHUB_USERNAME> \
  --docker-password=<GITHUB_TOKEN>
```

### RBAC Permissions
Ensure ArgoCD has proper permissions to deploy to target namespaces.

## 🎯 Environment-Specific Configuration

### Staging Environment
- **Resource Limits**: Lower CPU/memory for cost optimization
- **Replica Count**: Single replica for most services
- **Monitoring**: Basic health checks
- **Data**: Test data and mock services

### Production Environment  
- **Resource Limits**: Production-grade CPU/memory allocation
- **Replica Count**: Multiple replicas for high availability
- **Monitoring**: Comprehensive health checks and metrics
- **Data**: Production data with proper backups

## 🔄 CI/CD Integration

This repository integrates with Jenkins CI/CD pipeline:

1. **Feature Branches**: No GitOps changes (staging uses master branch)
2. **Master Branch**: Automatic staging deployment via ArgoCD sync
3. **Git Tags**: Jenkins updates `production.yaml` with new tag and commits to this repo


## 🤝 Contributing

### Making Changes

1. **Fork this repository**
2. **Create feature branch** (`git checkout -b feature/update-config`)
3. **Make configuration changes**
4. **Test changes** in staging environment
5. **Create pull request**

### Best Practices

- **Small Changes**: Keep commits focused and atomic
- **Descriptive Messages**: Use clear commit messages
- **Test First**: Validate changes in staging before production
- **Review Process**: Require PR reviews for production changes

## 🔗 Related Repositories

- **[local-devops-infrastructure](https://github.com/KeremAR/local-devops-infrastructure)**: Main application code and Helm charts
- **[jenkins-shared-library](https://github.com/KeremAR/jenkins-shared-library2)**: Reusable Jenkins pipeline functions

## 📝 License

This project is licensed under the MIT License.

## 📞 Contact

Project Owner: Kerem AR
- GitHub: [@KeremAR](https://github.com/KeremAR)

---

**Note**: This is a GitOps repository. Direct changes to Kubernetes clusters should be avoided. All infrastructure changes should go through this repository for proper version control and audit trails.
