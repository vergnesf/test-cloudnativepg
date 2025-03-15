# CloudNativePG

## Kubernetes Section ⚓

Integrating Kubernetes into this project enables efficient deployment and orchestration of microservices. Here are the steps to set up a local test environment using **kind** (Kubernetes IN Docker) 🐳 and to deploy a highly available PostgreSQL database with **CloudNativePG** 🐘.

### Creating a Kubernetes Cluster with kind 🔨

**kind** is a tool that lets you create local Kubernetes clusters using Docker as the container runtime. It’s perfect for quickly testing deployments in a simple environment. ⚡

```sh
cat <<EOF | kind create cluster -n my-cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

## Create nginx controller

[Documentation](https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx)

```sh
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

```sh
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

## Deploying CloudNativePG 🐘

[Documentation](https://cloudnative-pg.io/documentation/current/installation_upgrade/)

**CloudNativePG** is a solution that simplifies managing PostgreSQL databases in a Kubernetes environment, providing high availability features. 🚀 This sets up the necessary resources to run PostgreSQL reliably and efficiently. 🛡️

```sh
kubectl apply --server-side -f \
  https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/release-1.25/releases/cnpg-1.25.1.yaml
```

### Deploy it 🚀

```sh
k apply -k k8s/base .
```

