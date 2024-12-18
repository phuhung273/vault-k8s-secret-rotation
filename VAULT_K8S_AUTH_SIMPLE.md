## Ref
- https://github.com/hashicorp/vault-plugin-auth-kubernetes/issues/227
- https://github.com/hashicorp/vault/issues/29055

## Cluster
```bash
kind create cluster --config kind.yaml --name default
```

## Vault Cluster
Ref: https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-raft-deployment-guide

```bash
kubectl create namespace vault

helm repo add hashicorp https://helm.releases.hashicorp.com

helm install vault hashicorp/vault --namespace vault
```

## Expose vault
```bash
kubectl port-forward -n vault service/vault 8200:http
```

## Create Vault k8s auth
```bash
export VAULT_ADDR=http://127.0.0.1:8200
export VAULT_TOKEN=<your_token>

vault auth enable kubernetes

vault write auth/kubernetes/config kubernetes_host="https://10.96.0.1:443"
```

## Create Vault role
```bash
vault write auth/kubernetes/role/internal-app \
    bound_service_account_names="*" \
    bound_service_account_namespace_selector="{\"matchLabels\":{\"allow-vault\":\"true\"}}"
```

## Setup service account and namespace
```bash
kubectl create sa app

kubectl label ns default allow-vault=true

kubectl create clusterrolebinding app \
  --clusterrole cluster-admin \
  --serviceaccount default:app
```

## Allow vault to view namespace
```bash
kubectl create clusterrolebinding vault-view \
  --clusterrole view \
  --serviceaccount vault:vault
```

## Generate token and login
```bash
kubectl create token app

curl --request POST \
    --data "{\"role\":\"internal-app\",\"jwt\":\"$TOKEN\"}" \
    http://127.0.0.1:8200/v1/auth/kubernetes/login
```