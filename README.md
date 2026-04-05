# helm-to-argo

This repo contains Helm-based applications deployed to Kubernetes via ArgoCD. All apps use the [Nautilus](https://rama3319.github.io/Nautilus) generic Helm chart as a dependency.

## Prerequisites

- Helm 3
- kubectl configured against your cluster
- ArgoCD installed in the cluster

---

## Nautilus Chart

All apps in this repo use the Nautilus chart hosted at:

```
https://rama3319.github.io/Nautilus
```

Add it to Helm:

```bash
helm repo add nautilus https://rama3319.github.io/Nautilus
helm repo update
```

---

## Creating a New App

### 1. Create the app directory

```
helm-to-argo/
└── my-new-app/
    ├── Chart.yaml
    └── values.yaml
```

### 2. `Chart.yaml` — reference Nautilus as a dependency

```yaml
apiVersion: v2
name: my-new-app
version: 0.1.0
dependencies:
  - name: nautilus
    version: "0.4.0"
    repository: "https://rama3319.github.io/Nautilus"
    alias: app
```

> Check [Nautilus releases](https://github.com/rama3319/Nautilus/releases) for the latest version.

### 3. `values.yaml` — configure your app

```yaml
app:
  app:
    name: my-new-app
  image:
    repository: nginx         # your container image
    tag: "1.27"               # your image tag
  replicaCount: 1
  containerPort: 80
  service:
    type: ClusterIP
    port: 80
  ingress:
    enabled: true
    className: traefik
    host: my-new-app.local
  resources:
    requests:
      cpu: 50m
      memory: 64Mi
    limits:
      cpu: 100m
      memory: 128Mi
```

### 4. Pull dependency and test locally

```bash
cd my-new-app
helm dependency update .
helm template .             # preview rendered manifests
helm install my-new-app .  # install to cluster
```

---

## Deploying via ArgoCD

### 1. Register the Nautilus Helm repo in ArgoCD (one-time setup)

```bash
kubectl apply -f nautilus-repo-secret.yaml
```

### 2. Create an ArgoCD Application manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-new-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/rama3319/helm-to-argo.git
    targetRevision: HEAD
    path: my-new-app
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### 3. Apply it

```bash
kubectl apply -f my-new-app-argo-app.yaml
```

### 4. Verify

```bash
kubectl get application -n argocd
kubectl get pods
kubectl get ingress
```

---

## Apps in this Repo

| App | Chart Version | Host |
|-----|--------------|------|
| [nginx-webapp](./nginx-webapp) | custom | `nginx.local` |
| [my-app](./my-app) | nautilus 0.1.0 | `my-app.local` |
| [nautilus-app](./nautilus-app) | nautilus 0.4.0 | `nautilus-app.local` |
