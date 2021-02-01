# Anthos Configuration Management Directory

This is the root directory for Anthos Configuration Management.

## Install Config Sync

### Pre-Requisite

Create the `config-management-system` namespace and a secret containing the public ssh key with access to the target git repository.

```
kubectl create ns config-management-system && \
kubectl create secret generic git-creds \
 --namespace=config-management-system \
 --from-file=ssh=/path/to/KEYPAIR_PRIVATE_KEY_FILENAME
 ```

Copy the Config Sync Operator YAML
```
gsutil cp gs://config-management-release/released/latest/config-management-operator.yaml config-management-operator.yaml
```

Apply the operator YAML.
```
kubectl apply -f config-management-operator.yaml
```

Generate the config-management YAML to configure the agent.

```
# config-management.yaml

apiVersion: configmanagement.gke.io/v1
kind: ConfigManagement
metadata:
  name: config-management
spec:
  # clusterName is required and must be unique among all managed clusters
  clusterName: CLUSTER_NAME
  git:
    syncRepo: REPO
    syncBranch: BRANCH
    secretType: TYPE
    policyDir: "DIRECTORY"
```

Once that is complete apply the YAML.

```
kubectl apply -f config-management.yaml
```

### Verify Installation

```
kubectl -n kube-system get pods | grep config-management
```
The output should be similar to the below:
```
config-management-operator-6f988f5fdd-4r7tr 1/1 Running 0 26s
```

See [our documentation](https://cloud.google.com/anthos-config-management/docs/repo) for how to use each subdirectory.
