# Cardano Node and Cardano-DB-Sync Deployment on Kubernetes


## Prerequisites

- A running Kubernetes cluster
- `kubectl` configured to interact with the cluster
- Persistent storage (EFS, PVC, or local storage)
- Metric-Server installed in your cluster

## Prepare 

### 1. node label and tain
```sh
kubectl taint nodes <node-name> node_role=cardano:NoSchedule
kubectl label nodes <node-name> node_role=cardano
```

### 2. PVC



## Deployment Steps

### 1. Create Kubernetes Namespace

```sh
kubectl create namespace cardano
```

### 2. Create `cardano-config` ConfigMap:
```sh
kubectl apply -f cardano-node/cardano-config-cm.yaml
```

### 3. Create `cardano-genesis` ConfigMap:

```sh
kubectl create configmap cardano-genesis \
  --from-file=cardano-config/preprod-byron-genesis.json \
  --from-file=cardano-config/preprod-shelley-genesis.json \
  --from-file=cardano-config/preprod-alonzo-genesis.json \
  --from-file=cardano-config/preprod-conway-genesis.json \
  -n cardano --dry-run=client -o yaml | kubectl apply -f -
```

### 4. Deploy Cardano Relay Node
```sh
kubectl apply -f cardano-node/relay-nodes-deployment.yaml
```

### 5. Deploy Auto scale for cardano relay node
```sh
kubectl apply -f cardano-node/cardano-relay-hpa.yaml
```

### 6. Deploy Block Producing node
```sh
kubectl apply -f cardano-node/block-producing-sts.yaml
```

### 7. Deploy Postgresql (For example)
```sh
kubectl apply -f postgresql/postgresql.yaml
```

### 8. Create `dbsync-config` Configmaps
```sh
kubectl apply -f cardano-db-sync/cardano-db-sync-cm.yaml
```

### 9. Deploy cardano-db-sync
```sh
kubectl apply -f cardano-db-sync/cardano-db-sync-deployment.yaml
```

### 10. Deploy Auto scale for cardano-db-sync
```sh
kubectl apply -f cardano-db-sync/cardano-db-sync-hpa.yaml
```


## NOTE
If you run Cardano in a production environment, you should use PVC (Persistent Volume Claim) for the persistent data of the Cardano relay node and Cardano-db-sync. If you are using AWS, I prefer to use EFS (Elastic File System) storage to create the PVC. This is because the data needs to sync when scaling the pod




