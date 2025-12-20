# What Has Been Implemented

[Documentation Index](README.md)

---

## Kubernetes and GitOps Bootstrap

- Local Kubernetes cluster created using k3d
- Argo CD installed once as a bootstrap requirement
- Root Argo CD application introduced to manage all platform apps
- Transitioned fully to Git-driven reconciliation

## Helm-Only Platform Management

- Standardized on Helm for all platform services
- Removed Kustomize to reduce complexity
- Each platform capability is represented as:
  - Helm chart
  - Argo CD Application

## Ingress with Traefik

- Traefik configured as the ingress controller
- Hostname-based routing using *.local domains
- Ingress resources managed entirely via Git

## TLS Enablement

- cert-manager installed via Argo CD using Helm
- ClusterIssuer created for certificate issuance
- TLS enabled on Argo CD ingress

## Trusted HTTPS for Local Development

- Self-signed certificates issued correctly but flagged as untrusted by browsers
- mkcert introduced to create a locally trusted Certificate Authority
- TLS secret replaced with mkcert-generated certificate
- cert-manager issuance disabled for Argo CD to avoid conflict

The PEM files generated during this step are intentionally kept outside the repo (for example in ~/.config/argocd/certs/). They exist to establish local trust during development and demonstrate how platform teams solve HTTPS trust issues without weakening security. In production environments, certificates would be issued by a centralized, trusted certificate authority.

---

[Next: Issues and Resolutions](issues.md)
