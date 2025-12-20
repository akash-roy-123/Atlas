# Platform Engineering Capstone

Local Internal Developer Platform built with Kubernetes, GitOps, and Helm.

---

## Overview

This project is a demonstration of building an internal platform that helps software teams deploy, run, and operate applications without needing to understand or manage infrastructure details.

The platform acts as a shared foundation that provides:
- A secure environment where applications can run
- A single control panel to deploy and manage services
- Built-in networking, security, and observability
- Standardized ways to release and operate software

Instead of each team solving these problems independently, the platform centralizes them and exposes simple, repeatable workflows. While this implementation runs locally for learning and demonstration purposes, the architecture mirrors how modern engineering organizations build internal platforms at scale.

---

## Technical Details

This repository demonstrates a **local-first Internal Developer Platform (IDP)** designed using **real-world platform engineering principles**. The focus of this project is not tool installation alone, but building a **stable platform control plane** that is reproducible, Git-driven, and production-shaped, while remaining fully runnable on a developer laptop.

The platform is intentionally designed to surface and document common **GitOps, ingress, and TLS pitfalls**, along with how they were resolved, so others can debug similar issues effectively.

---

## Goals of the Project

- Build a Kubernetes-based platform using GitOps as the control mechanism
- Treat Git as the single source of truth
- Use Helm consistently for packaging and lifecycle management
- Own ingress, routing, and TLS at the platform level
- Create a foundation that can be extended with observability, developer portals, security, and policy

---

## Core Platform Principles

- **Git is the source of truth**  
  All platform components are managed via Git and reconciled by Argo CD.

- **Kubernetes is the runtime, not the interface**  
  Manual `kubectl apply` is avoided after bootstrap.

- **Helm as the standard abstraction**  
  Helm is used for both third-party and custom platform components.

- **Platform-owned ingress and TLS**  
  Applications inherit routing and security from the platform.

---

## Architecture (Current State)

| Layer | Technology |
|------|------------|
| Kubernetes Runtime | k3d (k3s in Docker) |
| GitOps Engine | Argo CD (app-of-apps model) |
| Package Management | Helm |
| Ingress Controller | Traefik |
| TLS Management | cert-manager + mkcert (local trusted CA) |
| DNS | /etc/hosts (`*.local`) |
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

**Important boundary:**  
Argo CD only reconciles resources under `platform/apps`. Any Application defined outside this path is effectively ignored.

---

## Bootstrap Commands (Executed)

The following commands were executed as part of the one-time bootstrap process. These are intentionally separated from steady-state GitOps operations.

### Local Tool Verification
```bash
docker version
kubectl version --client
helm version
k3d version
git --version
```

### Kubernetes Cluster Creation (k3d)
```bash
k3d cluster create pe-capstone \
  --servers 1 --agents 2 \
  --api-port 6550 \
  -p "80:80@loadbalancer" \
  -p "443:443@loadbalancer"
```

Verification:
```bash
kubectl get nodes
kubectl get pods -A
```

### Argo CD Installation (Bootstrap Only)
```bash
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Initial access:
```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

Retrieve admin password:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

### Local DNS Configuration in /etc/hosts
```text
127.0.0.1 argocd.local grafana.local backstage.local
```

### TLS Trust Setup (mkcert)
```bash
brew install mkcert nss
mkcert -install
mkcert argocd.local
```

Move the generated certs to a local, non-repo path:
```bash
mkdir -p ~/.config/argocd/certs
mv argocd.local.pem argocd.local-key.pem ~/.config/argocd/certs/
```

Create Kubernetes TLS secret:
```bash
kubectl -n argocd create secret tls argocd-tls \
  --cert=~/.config/argocd/certs/argocd.local.pem \
  --key=~/.config/argocd/certs/argocd.local-key.pem
```

---

## What Has Been Implemented

### Kubernetes & GitOps Bootstrap
- Local Kubernetes cluster created using k3d
- Argo CD installed once as a bootstrap requirement
- Root Argo CD application introduced to manage all platform apps
- Transitioned fully to Git-driven reconciliation

### Helm-Only Platform Management
- Standardized on Helm for all platform services
- Removed Kustomize to reduce complexity
- Each platform capability is represented as:
  - Helm chart
  - Argo CD Application

### Ingress with Traefik
- Traefik configured as the ingress controller
- Hostname-based routing using `*.local` domains
- Ingress resources managed entirely via Git

### TLS Enablement
- cert-manager installed via Argo CD using Helm
- ClusterIssuer created for certificate issuance
- TLS enabled on Argo CD ingress

### Trusted HTTPS for Local Development
- Self-signed certificates issued correctly but flagged as untrusted by browsers
- mkcert introduced to create a locally trusted Certificate Authority
- TLS secret replaced with mkcert-generated certificate
- cert-manager issuance disabled for Argo CD to avoid conflict

The PEM files generated during this step are intentionally kept outside the repo (for example in `~/.config/argocd/certs/`). They exist to establish local trust during development and demonstrate how platform teams solve HTTPS trust issues without weakening security. In production environments, certificates would be issued by a centralized, trusted certificate authority.

---

## Issues Encountered and How They Were Resolved (Summary)

### GitOps Reconciliation Scope
- Issue: Applications placed outside Argo CD’s watched path were never applied
- Resolution: Enforced `platform/apps` as the single GitOps entrypoint

### cert-manager Not Creating Certificates
- Issue: ClusterIssuer created before cert-manager was actually installed
- Resolution: Installed cert-manager via Argo CD with CRDs enabled

### TLS Showing as “Not Secure”
- Issue: Self-signed certificates are not trusted by browsers
- Resolution: Used mkcert to generate certificates trusted by the local OS

### HTTP to HTTPS Redirect Confusion
- Issue: Argo CD defaulted to secure mode while ingress and certs were not aligned
- Resolution: Restarted Argo CD after TLS changes and removed insecure flags

These issues were not syntax problems, but **control plane and lifecycle boundary misunderstandings**, which are common in real platform environments.

---

## Current Status

### Completed
- Kubernetes runtime
- GitOps control plane
- Helm-based platform management
- Ingress and routing
- Trusted HTTPS locally

### Planned
- Observability: Prometheus, Grafana
- Logging: Loki
- Developer Portal: Backstage
- Secrets Management: Vault, External Secrets
- Policy and Guardrails: Kyverno
- Infrastructure as Code: Terraform

---

## Next Phase

With the platform control plane stable, the next focus is **platform capabilities**, starting with **observability**, followed by developer experience, security, and governance.

---

## Why This Repository Exists

This project documents the **real experience of building a platform**, including missteps and corrections. Most platform issues arise from ownership boundaries, reconciliation scope, or lifecycle timing rather than incorrect YAML.

The intent is to help others:
- Understand GitOps boundaries
- Debug ingress and TLS issues faster
- Avoid repeating common bootstrap mistakes

---

## Final Note

Reaching this point means the platform is no longer “set up”, it is **operated**.  
Everything beyond this stage is capability onboarding, not infrastructure firefighting.
