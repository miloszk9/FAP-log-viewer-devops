# FAP-log-viewer-devops

## Setup

### Local k3d cluster

```bash
k3d cluster create fap --registry-create mycluster-registry:0.0.0.0:5000
```

### Cloud k3s cluster

TBD

## Deployment

### Manual

```bash
kubectl apply -f FAP-log-viewer-devops/argocd/fap-log-viewer/namespace.yaml
kubectl apply -f FAP-log-viewer-devops/argocd/fap-log-viewer/ -R
```

### ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl apply -f FAP-log-viewer-devops/argocd/fap-log-viewer.yaml
kubectl apply -f FAP-log-viewer-devops/argocd/sealed-secrets.yaml
```

## Kubeseal

- Install kubeseal on local Linux host
- Create sealed secrets controller

```bash
kubectl apply -f FAP-log-viewer-devops/argocd/sealed-secrets/controller.yaml
```

- Create public key

```bash
kubeseal --format yaml --cert=public-key-cert.pem
```

- Create sealed secret

```bash
kubectl create secret generic fap-secrets \
  --from-literal=DB_PASSWORD=password \
  -n fap-log-viewer \
  --dry-run=client \
  -o yaml | \
kubeseal --format yaml --cert=public-key-cert.pem > \
FAP-log-viewer-devops/k8s/fap-log-viewer/sealed_secret.yaml
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
