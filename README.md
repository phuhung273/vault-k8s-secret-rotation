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
kubectl port-forwarding -n vault service/vault 8200:http
```

## Run app
```bash
kubectl apply -f busybox.yaml
```

## Run DB
```
helm install my-release oci://registry-1.docker.io/bitnamicharts/postgresql
```

### Note
PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    my-release-postgresql.default.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default my-release-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run my-release-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:17.0.0-debian-12-r9 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host my-release-postgresql -U postgres -d postgres -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/my-release-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432

WARNING: The configured password will be ignored on new installation in case when previous PostgreSQL release was deleted through the helm command. In that case, old PVC will have an old password, and setting it through helm won't take effect. Deleting persistent volumes (PVs) will solve the issue.

## Create read-only DB role
```bash
kubectl exec -it my-release-postgresql-0 -- /bin/sh

psql -U postgres -c "CREATE ROLE \"ro\" NOINHERIT;"

psql -U postgres -c "GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"ro\";"
```