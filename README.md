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

   ```sh
   kind create cluster
   ```

1. Install Crossplane

   ```sh
   kubectl create namespace crossplane-system
   helm repo add crossplane-alpha https://charts.crossplane.io/alpha
   helm install crossplane --namespace crossplane-system crossplane-alpha/crossplane --set alpha.oam.enabled=true
   ```

1. Install the Crossplane CLI

   ```sh
   curl -sL https://raw.githubusercontent.com/crossplane/crossplane/release-0.14/install.sh | sh
   sudo mv kubectl-crossplane /usr/local/bin
   ```

1. Install CAPI's
   [clusterctl](https://cluster-api.sigs.k8s.io/user/quick-start.html#install-clusterctl).
1. Initialize Cluster API with the Equinix Metal infrastructure provider (still
   called `Packet` in CAPI docs) as indicated in the [CAPI
   quickstart](https://cluster-api.sigs.k8s.io/user/quick-start.html).

   ```sh
   export PACKET_API_KEY="34ts3g4s5g45gd45dhdh"
   export PROJECT_ID="abcds"
   clusterctl init --infrastructure packet
   ```

1. Install Crossplane (must be v1.0.0 or later).

   ```sh
   kubectl create namespace crossplane-system

   helm repo add crossplane-alpha https://charts.crossplane.io/alpha

   helm install crossplane --namespace crossplane-system crossplane-alpha/crossplane
   ```

1. Install the Equinix Metal Crossplane Provider

   ```sh
   kubectl crossplane install provider registry.upbound.io/equinix/crossplane-provider-equinix-metal:v0.0.5
   ```

1. Install Cross CAPI Metal package.

   ```sh
   kubectl crossplane install configuration registry.upbound.io/xp/cross-capi-metal
   ```

1. Create a `ProviderConfig` for Crossplane's Equinix Metal provider.

   ```sh
   kubectl create -n crossplane-system secret generic equinix-metal-creds --from-file=key=<(echo '{"apiKey":"'$PACKET_API_KEY'", "projectID":"'$PROJECT_ID'"}')
   ```

   ```sh
   cat << EOS | kubectl apply -f -
   apiVersion: metal.equinix.com/v1beta1
   kind: ProviderConfig
   metadata:
      name: default
   spec:
      projectID: $PROJECT_ID
      credentials:
         source: Secret
         secretRef:
            namespace: crossplane-system
            name: equinix-metal-creds
            key: key
   EOS
   ```

1. Create a `Demo` instance.
1. Use `capi-quickstart-kubeconfig` to connect to cluster and determine the IP
   address for the application `Service`. Go to the address and interact with
   the app.
