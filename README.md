# FAP-log-viewer-devops

## Setup

### Local k3d cluster

```bash
k3d cluster create --config k3d-local.yaml
```

Add to hosts file (e.g. `C:\Windows\System32\drivers\etc\hosts` in Windows):
`127.0.0.1 fap.local`

### Cloud k3s cluster

On master node:

```bash
vim /etc/rancher/k3s/config.yaml # Put content from k3s-prod-config.yaml
curl -sfL https://get.k3s.io | sh -
```

On worker nodes:

```bash
server_0_ip=10.0.0.23 # IP of the master node
token= # Get from master node from /var/lib/rancher/k3s/server/token
curl -sfL https://get.k3s.io | K3S_URL=https://${server_0_ip}:6443 K3S_TOKEN=${token} sh -
```

## Deployment using ArgoCD

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
kubectl apply -f FAP-log-viewer-devops/argocd/prometheus-stack-test.yaml
kubectl apply -f FAP-log-viewer-devops/argocd/fap-log-viewer-test.yaml
```

For prod:

```bash
kubectl apply -f FAP-log-viewer-devops/argocd/prometheus-stack-prod.yaml
kubectl apply -f FAP-log-viewer-devops/argocd/fap-log-viewer-prod.yaml
```

## Useful procedures

### Generate sealed secret

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

### Craete debug curl pod

```bash
kubectl -n fap-log-viewer run curl-test --image=alpine/curl -- sleep infinity
```
