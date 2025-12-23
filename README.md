# Atlas - Platform Engineering Capstone

A local Internal Developer Platform (IDP) built with Kubernetes, GitOps, and Helm. This project demonstrates real-world platform engineering principles, focusing on building a stable, Git-driven platform control plane that is reproducible and production-shaped while remaining fully runnable on a developer laptop.

## 🎯 Project Overview

Atlas is a Platform Engineering capstone project that showcases how to build an internal platform that helps software teams deploy, run, and operate applications without needing to understand or manage infrastructure details. The platform provides:

- **Secure Environment**: A Kubernetes-based runtime where applications can run safely
- **GitOps Control Plane**: Argo CD as the single source of truth for all platform services
- **Built-in Networking**: Traefik ingress controller for routing and TLS termination
- **Certificate Management**: cert-manager for automated TLS certificate lifecycle
- **Standardized Deployment**: Helm-based packaging and lifecycle management

The platform is intentionally designed to surface and document common GitOps, ingress, and TLS pitfalls, along with how they were resolved, so others can debug similar issues effectively.

## 📚 Documentation

Comprehensive documentation is available in the [`docs/readme/`](docs/readme/) directory:

- **[Overview and Goals](docs/readme/overview.md)** - Project vision, technical details, and core principles
- **[Architecture and Repo Layout](docs/readme/architecture.md)** - System architecture and repository structure
- **[Bootstrap Guide](Bootstrap.md)** - One-time Day-0 setup instructions (manual steps only)
- **[Bootstrap Commands](docs/readme/bootstrap.md)** - Detailed bootstrap command reference
- **[Implementation Status](docs/readme/implementation.md)** - What has been built and deployed
- **[Issues and Resolutions](docs/readme/issues.md)** - Common problems encountered and solutions
- **[Status and Next Steps](docs/readme/status.md)** - Current state and future roadmap

## 🏗️ Repository Structure

```
Atlas/
├── Bootstrap.md                    # Day-0 bootstrap guide (one-time manual steps)
├── docs/
│   └── readme/                     # Comprehensive documentation
│       ├── README.md               # Documentation index
│       ├── overview.md             # Project overview and goals
│       ├── architecture.md         # Architecture and repo layout
│       ├── bootstrap.md           # Bootstrap commands reference
│       ├── implementation.md      # Implementation status
│       ├── issues.md              # Issues and resolutions
│       └── status.md              # Current status and next steps
├── gitops/
│   └── argocd/
│       └── root-app.yaml          # Root Argo CD application (app-of-apps)
└── platform/
    ├── apps/                      # Argo CD Application definitions
    │   ├── argocd-edge-app.yaml   # Argo CD edge configuration
    │   ├── cert-manager-app.yaml  # cert-manager installation
    │   └── cert-manager-bootstrap-app.yaml  # cert-manager ClusterIssuer
    ├── charts/                    # Custom Helm charts
    │   ├── argocd-edge/           # Argo CD ingress and configuration
    │   └── cert-manager-bootstrap/ # Self-signed ClusterIssuer
    ├── configmaps/                # Platform configuration
    │   └── argocd/
    ├── helm/                      # Shared Helm values
    │   └── apps/
    └── patches/                   # Kubernetes patches
```

## 🚀 Quick Start

### Prerequisites

- Docker Engine (running)
- `kubectl`
- `helm`
- `k3d`
- `git`

### Bootstrap (One-Time Setup)

1. **Create Local Kubernetes Cluster**
   ```bash
   k3d cluster create pe-capstone \
     --servers 1 --agents 2 \
     --api-port 6550 \
     -p "80:80@loadbalancer" \
     -p "443:443@loadbalancer"
   ```

2. **Install Argo CD**
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Configure Local DNS**
   Add to `/etc/hosts`:
   ```
   127.0.0.1 argocd.local backstage.local grafana.local prometheus.local
   ```

4. **Deploy Root Application**
   ```bash
   kubectl apply -f gitops/argocd/root-app.yaml
   ```

For detailed bootstrap instructions, see [Bootstrap.md](Bootstrap.md) or [docs/readme/bootstrap.md](docs/readme/bootstrap.md).

## 📈 Project Evolution

Based on the commit history, the project has evolved through several key phases:

### Phase 1: Repository Bootstrap (Initial Setup)
- **Commits**: `55d3c4f`, `e34c853`, `4826734`
- Set up initial repository structure
- Configured root Argo CD application

### Phase 2: Argo CD Integration
- **Commits**: `c7af49c`, `c046114`, `16edd00`, `fc9aeef`, `3e75ed9`
- Exposed Argo CD via Traefik ingress
- Configured ingress routing with proper ingressClassName
- Managed Argo CD configuration via GitOps

### Phase 3: Kustomize to Helm Migration
- **Commits**: `7febaa2`, `ba78d45`, `2cc8e08`
- **Major Refactor**: Removed Kustomize in favor of Helm-only approach
- Standardized on Helm for all platform services
- Implemented app-of-apps pattern

### Phase 4: Certificate Management
- **Commits**: `168cfcb`, `e33c59d`, `15b1ee8`, `236e05f`
- Added cert-manager via Helm (GitOps)
- Created self-signed ClusterIssuer
- Enabled TLS for Argo CD ingress
- Fixed cert-manager installation order issues

### Phase 5: Documentation and Refinement
- **Commits**: `88f688d`, `3202d3b`, `48545d9`
- Added comprehensive documentation
- Organized documentation into structured pages
- Documented issues, resolutions, and best practices

## 🎓 Core Principles

1. **Git as Source of Truth**: All platform components are managed via Git and reconciled by Argo CD
2. **Kubernetes as Runtime**: Manual `kubectl apply` is avoided after bootstrap
3. **Helm as Standard**: Helm is used for both third-party and custom platform components
4. **Platform-Owned Ingress and TLS**: Applications inherit routing and security from the platform

## 🔧 Current Capabilities

✅ **Completed**
- Kubernetes runtime (k3d)
- GitOps control plane (Argo CD)
- Helm-based platform management
- Ingress and routing (Traefik)
- Trusted HTTPS locally (cert-manager + mkcert)

📋 **Planned**
- Observability: Prometheus, Grafana
- Logging: Loki
- Developer Portal: Backstage
- Secrets Management: Vault, External Secrets
- Policy and Guardrails: Kyverno
- Infrastructure as Code: Terraform

## 🔍 Key Learnings and Issues Resolved

This project documents real-world platform engineering challenges:

- **GitOps Reconciliation Scope**: Applications must be in the watched path (`platform/apps`)
- **cert-manager Lifecycle**: ClusterIssuer must be created after cert-manager installation
- **TLS Trust**: Self-signed certificates require local CA (mkcert) for browser trust
- **HTTP/HTTPS Alignment**: Argo CD configuration must align with ingress and certificate setup

See [docs/readme/issues.md](docs/readme/issues.md) for detailed problem-solution pairs.

## 🔗 Repository Information

- **Repository URL**: `https://github.com/akash-roy-123/Atlas.git`
- **Target Branch**: `main`
- **GitOps Path**: `platform/apps`

## 📝 License

This is a capstone project for educational and demonstration purposes.

## 🤝 Contributing

This project is part of a Platform Engineering capstone. For questions or contributions, please refer to the documentation in `docs/readme/`.

---

**Note**: This platform is designed for local development and learning. The architecture mirrors production platform engineering patterns while remaining fully runnable on a developer laptop.

