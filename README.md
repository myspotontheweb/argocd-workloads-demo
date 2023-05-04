# argocd-workloads-demo

A demo repository showing how application workloads can be managed by ArgoCD on a Kubernetes cluster

# Software

Install [arkade](https://arkade.dev) and then use it to setup dependent tools

```
ark get kubectl kubectx kubens helm k3d argocd devspace yq jq
```

# Quick start

Create a k8s cluster

```
k3d cluster create argocd-01 --agents=2
```

Core install of ArgoCD

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml
```

Login to ArgoCD

```
kubectl config set-context --current --namespace=argocd
argocd login --core
```

Start a local UI proxy

```
argocd admin dashboard
```

Deploy "dev" workloads

```
argocd app create bootstrap-dev \
   --repo https://github.com/myspotontheweb/argocd-workloads-demo.git \
   --path argocd/dev \
   --dest-namespace argocd \
   --dest-server https://kubernetes.default.svc
```