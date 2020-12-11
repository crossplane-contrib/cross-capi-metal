# cross-capi-metal

This repository contains a guide and related manifests for using
[Crossplane](https://github.com/crossplaneio/crossplane) and [Cluster
API](https://github.com/kubernetes-sigs/cluster-api) with infrastructure on
[Equinix Metal](https://metal.equinix.com/).

## Repository Contents

- Simple Golang app that connects to a MySQL database.
- Helm chart to install app into a Kubernetes cluster.
- Crossplane package to deploy:
   - CAPI cluster on Equinix Metal
   - Application into CAPI cluster
   - MySQL database

## Guide

1. Create a control plane cluster. This can be any Kubernetes cluster. We
   recommend starting out with [KIND](https://kind.sigs.k8s.io/).
2. Install CAPI's
   [clusterctl](https://cluster-api.sigs.k8s.io/user/quick-start.html#install-clusterctl).
3. Initialize Cluster API with the Equinix Metal infrastructure provider (still
   called `Packet` in CAPI docs) as indicated in the [CAPI
   quickstart](https://cluster-api.sigs.k8s.io/user/quick-start.html).

```
export PACKET_API_KEY="34ts3g4s5g45gd45dhdh"

clusterctl init --infrastructure packet
```

4. Install Crossplane (must be v1.0.0 or later).

```
kubectl create namespace crossplane-system

helm repo add crossplane-alpha https://charts.crossplane.io/alpha

helm install crossplane --namespace crossplane-system crossplane-alpha/crossplane
```

5. Install Cross CAPI Metal package.

```
kubectl crossplane install configuration registry.upbound.io/xp/cross-capi-metal
```

6. Create a `ProviderConfig` for Crossplane's Equinix Metal provider.

```
kubectl create -n crossplane-system secret generic equinix-metal-creds --from-file=key=<(echo '{"apiKey":"'$APIKEY'", "projectID":"'$PROJECT_ID'"}')
```

_TODO: we can likely reuse the credentials that are created in the clusterctl
init phase._

```
cat << EOS | kubectl apply -f -
apiVersion: metal.equinix.com/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  projectID: $PROJECT_ID
  credentials:
    secretRef:
      source: Secret
      namespace: crossplane-system
      name: equinix-metal-creds
      key: key
EOS
```

8. Create a `Demo` instance.
9. Use `capi-quickstart-kubeconfig` to connect to cluster and determine the IP
   address for the application `Service`. Go to the address and interact with
   the app.
