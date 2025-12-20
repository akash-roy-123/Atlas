# Platform Engineering Capstone

Repository scaffold for a platform engineering capstone project. The layout is pre-wired for common platform concerns (clusters, infra as code, platform services, GitOps) but intentionally empty so you can drop in your manifests, charts, and docs.

## Repository layout

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

## Getting started

- Fill in each component directory with the manifests, Helm charts, or Terraform modules required for your platform.
- Document environment-specific values (dev/stage/prod) under `docs/` and keep runbooks alongside the services they support.
- Keep GitOps-ready Kubernetes manifests under `gitops/argocd/` so the Argo CD control plane can reconcile them.
- Track local cluster setup steps in `clusters/k3d/` to make onboarding reproducible.

## Contributing

- Add a short `README.md` to any non-trivial subfolder that explains how to apply, test, or operate that component.
- Prefer declarative configs and keep secrets out of the repo; reference external secret managers instead.
- Before pushing, run format/validation steps for your IaC or Helm tooling (e.g., `terraform fmt`/`validate`, `helm lint`) once they are added.
