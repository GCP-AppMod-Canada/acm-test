# Anthos Configuration Management Directory

## Pre-Requisites

- Kubernetes Cluster
- kubectl CLI
- gcloud CLI

### Creating a cluster

Set ENV variables

```
export PROJECT_ID=<my-project>
export NETWORK=default
export SUBNET_NAME=<desired-subnet-name>
export REGION=northamerica-northeast1-a
```

Create the Cluster

```
gcloud container clusters create acm-test-dev \
--enable-shielded-nodes \
--no-enable-basic-auth \
--enable-network-policy \
--release-channel regular \
--enable-stackdriver-kubernetes \
--enable-ip-alias \
--shielded-secure-boot \
--machine-type e2-standard-4 \
--network $NETWORK \
--create-subnetwork name=${SUBNET_NAME} \
--no-issue-client-certificate \
--num-nodes 3 \
--no-enable-legacy-authorization \
--image-type cos_containerd \
--zone $REGION \
--workload-pool=${PROJECT_ID}.svc.id.goog
```

This is the root directory for Anthos Configuration Management.

## Install Config Sync

1. Create the `config-management-system` namespace and a secret containing the public ssh key with access to the target git repository.

```
kubectl create ns config-management-system && \
kubectl create secret generic git-creds \
 --namespace=config-management-system \
 --from-file=ssh=/path/to/KEYPAIR_PRIVATE_KEY_FILENAME
 ```

You will need to add the public (ie `~/.ssh/id_rsa.pub`) key to your git repository.

- [github](https://docs.github.com/en/developers/overview/managing-deploy-keys) deploy keys
- [gitlab](https://docs.gitlab.com/ee/ssh/) deploy keys

2. Copy the Config Sync Operator YAML
```
gsutil cp gs://config-management-release/released/latest/config-management-operator.yaml config-management-operator.yaml
```

Apply the operator YAML.
```
kubectl apply -f config-management-operator.yaml
```

3. Generate the config-management YAML to configure the agent.

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
