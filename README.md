# FAP-log-viewer-devops

## Setup

### Local k3d cluster

```bash
k3d cluster create fap \
  --registry-create mycluster-registry:0.0.0.0:5000 \
  --port "80:80@loadbalancer" \
  --port "443:443@loadbalancer"
```

Add to hosts file (e.g. `C:\Windows\System32\drivers\etc\hosts` in Windows):
`127.0.0.1 fap.local`

### Cloud k3s cluster

TBD

## Deployment (ArgoCD)

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl -n argocd port-forward svc/argocd-server 8081:443
kubectl apply -f FAP-log-viewer-devops/argocd/sealed-secrets.yaml
kubectl apply -f FAP-log-viewer-devops/argocd/cert-manager.yaml
```

For local development:

```bash
kubectl apply -f FAP-log-viewer-devops/argocd/fap-log-viewer-test.yaml
```

For prod:

```bash
kubectl apply -f FAP-log-viewer-devops/argocd/fap-log-viewer-prod.yaml
```

## Kubeseal

Prerequisite: Install kubeseal on local Linux host

- Obtain kube-seal public key

```bash
kubeseal --fetch-cert > public-key-cert.pem
```

- Create sealed secret

```bash
kubectl create secret generic fap-secrets \
  --from-literal=DB_PASSWORD=$(tr -dc 'A-Za-z0-9!?%=' < /dev/urandom | head -c 20) \
  -n fap-log-viewer \
  --dry-run=client \
  -o yaml | \
kubeseal --format yaml --cert=public-key-cert.pem > \
FAP-log-viewer-devops/k8s/fap-log-viewer/base/common/sealed_secret.yaml
```

## Other

- Craete debug curl pod

```bash
kubectl -n fap-log-viewer run curl-test --image=alpine/curl -- sleep infinity
```

- Push to local registry

```bash
docker build -t mylosz/fap-log-viewer-frontend:0.0.1 ./FAP-log-viewer/frontend
docker tag mylosz/fap-log-viewer-frontend:0.0.1 localhost:5000/mylosz/fap-log-viewer-frontend:0.0.1
```

Use `mycluster-registry:5000/mylosz/fap-log-viewer-frontend:0.0.1` as k8s image
