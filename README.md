# Cluster
```bash
kind create cluster --config kind.yaml --name default
```

# Vault Cluster
Ref: https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-raft-deployment-guide

```bash
kubectl create namespace vault

helm repo add hashicorp https://helm.releases.hashicorp.com

helm install vault hashicorp/vault --namespace vault
```

# Expose vault
```bash
kubectl port-forwarding -n vault service/vault 8200:http
```

# Run app
```bash
kubectl apply -f busybox.yaml
```