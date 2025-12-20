# Bootstrap Commands

[Documentation Index](README.md)

---

The following commands were executed as part of the one-time bootstrap process. These are intentionally separated from steady-state GitOps operations.

## Local Tool Verification

```bash
docker version
kubectl version --client
helm version
k3d version
git --version
```

## Kubernetes Cluster Creation (k3d)

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

## Argo CD Installation (Bootstrap Only)

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

## Local DNS Configuration in /etc/hosts

```text
127.0.0.1 argocd.local grafana.local backstage.local
```

## TLS Trust Setup (mkcert)

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

[Next: What Has Been Implemented](implementation.md)
