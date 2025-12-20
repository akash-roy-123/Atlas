# Architecture and Repo Layout

[Documentation Index](README.md)

---

## Architecture (Current State)

| Layer | Technology |
|------|------------|
| Kubernetes Runtime | k3d (k3s in Docker) |
| GitOps Engine | Argo CD (app-of-apps model) |
| Package Management | Helm |
| Ingress Controller | Traefik |
| TLS Management | cert-manager + mkcert (local trusted CA) |
| DNS | /etc/hosts (*.local) |
| Repository Model | Monorepo |

---

## Repository Structure

```
.
├─ platform-engineering-capstone/
│  ├─ clusters/
│  │  └─ k3d/                        # Local/dev cluster definitions
│  ├─ docs/
│  │  ├─ local-setup/                # Developer onboarding guides
│  │  └─ networking/                 # Network design/runbooks
│  ├─ gitops/
│  │  └─ argocd/                     # Argo CD app definitions and config
│  ├─ infra/
│  │  └─ terraform/                  # Cloud infrastructure as code
│  └─ platform/
│     ├─ ingress/traefik/            # Ingress controller config
│     ├─ security/
│     │  ├─ external-secrets/        # Secret sync setup (e.g., ESO)
│     │  └─ vault/                   # Vault deployment/config
│     ├─ cert-manager/               # Cluster certificate management
│     ├─ observability/
│     │  ├─ logging/                 # Centralized logging stack
│     │  ├─ prometheus/              # Metrics collection/alerts
│     │  ├─ grafana/                 # Dashboards
│     │  └─ tracing/                 # Distributed tracing
│     ├─ registry/                   # Container registry config
│     ├─ helm/                       # Shared Helm charts or values
│     └─ policy/kyverno/             # Policy as code
```

Important boundary: Argo CD only reconciles resources under platform/apps. Any Application defined outside this path is effectively ignored.

---

[Next: Bootstrap Commands](bootstrap.md)
