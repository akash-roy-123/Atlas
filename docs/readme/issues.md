# Issues and Resolutions

[Documentation Index](README.md)

---

## GitOps Reconciliation Scope

- Issue: Applications placed outside Argo CD's watched path were never applied
- Resolution: Enforced platform/apps as the single GitOps entrypoint

## cert-manager Not Creating Certificates

- Issue: ClusterIssuer created before cert-manager was actually installed
- Resolution: Installed cert-manager via Argo CD with CRDs enabled

## TLS Showing as "Not Secure"

- Issue: Self-signed certificates are not trusted by browsers
- Resolution: Used mkcert to generate certificates trusted by the local OS

## HTTP to HTTPS Redirect Confusion

- Issue: Argo CD defaulted to secure mode while ingress and certs were not aligned
- Resolution: Restarted Argo CD after TLS changes and removed insecure flags

These issues were not syntax problems, but control plane and lifecycle boundary misunderstandings, which are common in real platform environments.

---

[Next: Status, Next Phase, and Rationale](status.md)
