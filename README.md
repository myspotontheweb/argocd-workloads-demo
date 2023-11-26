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

## Deploy workloads

Deploy "dev" workloads

```
argocd app create bootstrap-dev \
   --repo https://github.com/myspotontheweb/argocd-workloads-demo.git \
   --path argocd/dev \
   --dest-namespace argocd \
   --dest-server https://kubernetes.default.svc \
   --sync-policy automated
```

Deploy "test" workloads

```
argocd app create bootstrap-test \
   --repo https://github.com/myspotontheweb/argocd-workloads-demo.git \
   --path argocd/test \
   --dest-namespace argocd \
   --dest-server https://kubernetes.default.svc \
   --sync-policy automated
```

Deploy "prod" workloads

```
argocd app create bootstrap-prod \
   --repo https://github.com/myspotontheweb/argocd-workloads-demo.git \
   --path argocd/prod \
   --dest-namespace argocd \
   --dest-server https://kubernetes.default.svc \
   --sync-policy automated
```

# Configuration

## Application configuration

Applications are deployed using Helm. Each application has a "dev", "test" and "prod" sub-directory corresponding to the environments used in the application release promotion process.

```
apps
├── springboot-demo1
    ├── dev
    │   ├── Chart.yaml
    │   └── values.yaml
    ├── prod
    │   ├── Chart.yaml
    │   └── values.yaml
    └── test
        ├── Chart.yaml
        └── values.yaml
```

Each helm chart uses dependency management to pull in the correct chart version. In this case the [springboot-demo1](https://github.com/myspotontheweb/springboot-demo1) repository is publishing a versioned Helm chart, 1.0.1.

```
apiVersion: v2
name: springboot-demo1
description: A Helm chart for Spring Boot demo application
type: application
version: 1.0.0
appVersion: "v1.0.0"
dependencies:
  - name: demo1
    repository: oci://ghcr.io/myspotontheweb/spring-boot-demo1/charts
    version: 1.0.1
    alias: app
```

The values.yaml file is used to customize any settins for the target environment

```
app:
  replicaCount: 3
```

## ArgoCD configuration

Each environment type is represented within ArgoCD as a [Project](https://argo-cd.readthedocs.io/en/stable/user-guide/projects/). There is an additional [ApplicationSet](https://argo-cd.readthedocs.io/en/stable/user-guide/application-set/) to generate the Application configuration for each deployment using helm.

```
argocd
├── dev
│   ├── ApplicationSet.yaml
│   └── AppProject.yaml
├── prod
│   ├── ApplicationSet.yaml
│   └── AppProject.yaml
└── test
    ├── ApplicationSet.yaml
    └── AppProject.yaml
```

Notes:

* The environment settings are contained in separate directories, enabling them to be deployed to different Kubernetes clusters

# Testing Helm charts

The following command will download the remote helm chart dependencies

```
helm dependencies update apps/springboot-demo1/dev

```

These are stored locally as follows

```
apps/springboot-demo1
└── dev
    ├── Chart.lock
    ├── Chart.yaml
    └── charts
        └── demo1-1.0.1.tgz
```

Now possible to review the generated YAML manifests for Kubernetes

```
helm template rel1 apps/springboot-demo1/dev
```
