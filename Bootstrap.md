PLATFORM ENGINEERING CAPSTONE
BOOTSTRAP GUIDE (DAY-0 ONLY)

==================================================

PURPOSE

This document captures the minimal, one-time bootstrap steps
required to bring the platform control plane online.

These steps are intentionally NOT GitOps-managed.

Once completed, all platform services MUST be deployed
via Git and Argo CD only.

If something appears here and is run repeatedly,
the platform model is broken.

==================================================

WHAT IS BOOTSTRAP VS GITOPS

Bootstrap (Manual, One-Time)
- Create Kubernetes cluster
- Install Argo CD
- Install Ingress Controller (Traefik)
- Configure local networking

GitOps (Steady State)
- Platform services
- Applications
- Configuration changes
- Upgrades

Rule:
"If it is not installed by Argo CD, it does not exist."

==================================================

PREREQUISITES

Local machine must have:
- Docker Engine (running)
- kubectl
- Helm
- k3d
- git

Verify installations:

docker version
kubectl version --client
helm version
k3d version
git --version

Do not proceed if any command fails.

==================================================

STEP 1: CREATE LOCAL KUBERNETES CLUSTER (k3d)

Purpose:
Provision a local Kubernetes runtime for the platform.

Command:

k3d cluster create pe-capstone --servers 1 --agents 2 --api-port 6550 -p "80:80@loadbalancer" -p "443:443@loadbalancer"

Validate:

kubectl get nodes
kubectl get pods -A

Expected:
- All nodes in Ready state
- Core system pods running

==================================================

STEP 2: INSTALL ARGO CD (GITOPS ENGINE)

Purpose:
Argo CD is the control plane that enables Git-driven installs.

Create namespace:

kubectl create namespace argocd

Install Argo CD:

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

Wait for readiness:

kubectl -n argocd get pods

Temporary access (bootstrap only):

kubectl -n argocd port-forward svc/argocd-server 8080:443

Retrieve admin password:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

Login:
- URL: https://localhost:8080
- Username: admin
- Password: output from above command

==================================================

STEP 3: INSTALL TRAEFIK (INGRESS CONTROLLER)

Purpose:
Provide ingress routing for platform services.

Check if Traefik already exists:

kubectl -n kube-system get deploy | grep -i traefik

If Traefik exists:
- Reuse it

If Traefik does not exist:

helm repo add traefik https://traefik.github.io/charts
helm repo update

kubectl create namespace traefik

helm install traefik traefik/traefik -n traefik --set ports.web.exposedPort=80 --set ports.websecure.exposedPort=443

Validate:

kubectl -n traefik get pods
kubectl -n traefik get svc

==================================================

STEP 4: LOCAL HOSTNAME CONFIGURATION

Purpose:
Enable hostname-based routing for local development.

Edit /etc/hosts and add:

127.0.0.1 argocd.local backstage.local grafana.local prometheus.local

==================================================

END OF BOOTSTRAP

At this point:
- Kubernetes is running
- Argo CD is available
- Ingress is operational

From here onward:
- NO manual helm install
- NO kubectl apply for platform services
- All installs MUST be GitOps-driven via Argo CD

Next step:
Deploy cert-manager via GitOps.