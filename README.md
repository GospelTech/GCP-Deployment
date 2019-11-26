# Overview

The Gospel Data Platform is a shared store of enterprise information, spread amongst a number of entities, used in the form of a graph database. Access control is enforced at the data layer, based on the context of who the user is, what they are doing, and how they are doing it. Consensus amongst the peers of the underlying blockchain network is used to calculate and enforce read and write access control and a specially designed external component (the “keystore”) is used to ensure that the consensus mechanism cannot be bypassed.

Gospel is deployed by configuring a least 3 LedgerNodes with your partners in the GCP service, for redundancy, each LedgerNode can be located in a separate region.

A LedgerNode is a Kubernetes deployment consisting of multiple containers that communicate with each other. A load balancer can be configured in each region to forward traffic and is configured with a static public IP address. Your GCP service also ensures that all outgoing traffic shares the same source address as the load balancer. Nginx acts as a reverse proxy to direct traffic to the Gospel components within the Kubernetes deployment.

Gospel Components
-----------------

#### LedgerNodes

LedgerNode is the name given to a Gospel deployment with one or more of the components below. A LedgerNode typically contains a Peer, a CA and an Orderer.

#### Peers

Peers are the core part of Gospel that, together with the Orderer, form the private, permissioned blockchain. The peers participate actively in the blockchain to create consensus on transactions. They store data (blockchain) and the data access rules (chaincode) in Gospel.

#### Orderer

The orderer ensures that transactions are processed in the same sequence to maintain the integrity of the network. In other words, the Orderer outputs the same messages to all connected peers in the same logical order.

#### Certificate Authority (CA)

The Gospel CA is used to authenticate end users to Gospel. Once a user has completed the authentication process, they are provided with a certificate that is used to validate their transactions for the session. The CA supports many plugins for authentication, authorisation and assurance.

#### Nginx

Nginx has two purposes. When a user first connects, Nginx is a reverse proxy that will send them to their intended destination. This could be the Gospel backend or CA. Where a user is connecting to the Gospel administration UI, Nginx will also serve this to the browser.

#### Backend

The backend is a middleware component within Gospel. It is the endpoint for API connections and its purpose is to send transactions to read/write data through the Gospel process on behalf of an authenticated user.

#### Keystore

The keystore is a required component of Gospel and is part of the secure process to enforce consensus on reads. This component runs outside of LedgerNode and is deployed for each multi-tenant namespace. It is a lightweight Go application that runs on a VM or Kubernetes Pod. The specific details of this module are patent pending.

Installing Gospel from the Google Marketplace is achieved clicking on the installation button and following the steps in the installation wizard. By following the steps in this wizard you will be able to deploy your own fully operational Gospel infrastructure in a few clicks.

# Installation

## Quick install with Google Cloud Marketplace

Click **Configure**.

Select the Kubernetes cluster that you want to deploy the app to. If you want to create a new cluster, click **Create cluster**

Select a Namespace to use for the application

In the **App instance name** box, enter a name for the app, such as `sandbox-dev-app` (the name must be unique within the namespace) and then click **Deploy**.

Gospel is deployed by taking you through a step-by-step installer. When the installer starts, you will be given 3 options:

-   Install a demonstration network - This installs a Gospel instance pre-configured with data for you to quickly test the features of the platform

-   Join an existing network - Use this option if you are intending to join an existing Gospel network

-   Create a new network - Use this option when you and your partners are setting up a Gospel network for the first time

Choose an option related to the type of installation you require and follow the on-screen instructions to deploy your Gospel instance.

## Command line instructions

You can use [Google Cloud Shell](https://cloud.google.com/shell/) or a local workstation to complete these steps.

[![Open in Cloud Shell](http://gstatic.com/cloudssh/images/open-btn.svg)](https://console.cloud.google.com/cloudshell/editor?cloudshell_git_repo=https://github.com/GospelTech/GCP-Deployment)

### Prerequisites

#### Set up command line tools

You'll need the following tools in your environment. If you are using Cloud Shell, these tools are installed in your environment by default.

- [gcloud](https://cloud.google.com/sdk/gcloud/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [docker](https://docs.docker.com/install/)
- [openssl](https://www.openssl.org/)
- [helm](https://helm.sh/docs/using_helm/#installing-helm)
- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

Configure `gcloud` as a Docker credential helper:
```shell script
gcloud auth configure-docker
```

#### Create a Google Kubernetes Engine cluster

Create a cluster from the command line. If you already have a cluster that
you want to use, this step is optional.

```shell script
export CLUSTER=gospel-cluster
export ZONE=us-west1-a

gcloud container clusters create "$CLUSTER" --zone "$ZONE"
```

#### Configure kubectl to connect to the cluster

```shell script
gcloud container clusters get-credentials "$CLUSTER" --zone "$ZONE"
```

#### Clone this repo

Clone this repo and the associated tools repo:

```shell script
git clone --recursive https://github.com/GospelTech/GCP-Deployment.git
```

#### Install the Application resource definition

An Application resource is a collection of individual Kubernetes components,
such as Services, Deployments, and so on, that you can manage as a group.

To set up your cluster to understand Application resources, run the following command:

```shell script
kubectl apply -f "https://raw.githubusercontent.com/GoogleCloudPlatform/marketplace-k8s-app-tools/master/crd/app-crd.yaml"
```

You need to run this command once for each cluster.

The Application resource is defined by the
[Kubernetes SIG-apps](https://github.com/kubernetes/community/tree/master/sig-apps) community. The source code can be found on
[github.com/kubernetes-sigs/application](https://github.com/kubernetes-sigs/application).

### Install the application

Navigate to the `GCP-Deployment` directory:

```shell script
cd GCP-Deployment
```

#### Configure the application with environment variables

Choose an instance name and
[namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
for the application. In most cases, you can use the `default` namespace, but in case you want to segregate it off from anything else in your Kubernetes cluster, we suggest `gospel`.

```shell script
export APP_INSTANCE_NAME=gospel-technology-1
export NAMESPACE=gospel
```

Configure container images:

```shell script
TAG=5.0
REGISTRY="marketplace.gcr.io/gospel-technology/gospel-technology"
export FRONTEND_IMAGE="${REGISTRY}:${TAG}"
export BACKEND_IMAGE="${REGISTRY}/gospel-installer-backend:${TAG}"
export CHAINCODE_IMAGE="${REGISTRY}/chaincode:${TAG}"
export DATAIMPORTER_IMAGE="${REGISTRY}/dataimporter:${TAG}"
export ENDORSING_PEER_IMAGE="${REGISTRY}/endorsing-peer:${TAG}"
export GOSPEL_BACKEND_IMAGE="${REGISTRY}/gospel-backend:${TAG}"
export KEYSTORE_IMAGE="${REGISTRY}/keystore:${TAG}"
export NGINX_IMAGE="${REGISTRY}/nginx:${TAG}"
export ORCHESTRATOR_IMAGE="${REGISTRY}/ca-orchestrator:${TAG}"
export ORDERER_IMAGE="${REGISTRY}/orderer:${TAG}"
export SIGNER_IMAGE="${REGISTRY}/ca-signer:${TAG}"
export SOAS_IMAGE="${REGISTRY}/ca-soas:${TAG}"
export SOAU_IMAGE="${REGISTRY}/ca-soau:${TAG}"
export SOI_IMAGE="${REGISTRY}/ca-soi:${TAG}"
```

Specify a service account name:
This account will be used to configure your Gospel cluster

```shell script
export INSTALLER_SERVICE_ACCOUNT="${APP_INSTANCE_NAME}-sa"
```

Create the service account:

```shell script
kubectl create serviceaccount ${INSTALLER_SERVICE_ACCOUNT} --namespace ${NAMESPACE}
```

Grant the service account permission to create API objects in your Kubernetes cluster:
```shell script
kubectl create clusterrolebinding gospel-installer-role-binding --clusterrole=cluster-admin --serviceaccount=${NAMESPACE}:${INSTALLER_SERVICE_ACCOUNT}
```

#### Expand the manifest template

Use `helm template` to expand the template. We recommend that you save the
expanded manifest file for future updates to the application.

```shell script
helm template chart/gospel-technology \
  --name=$APP_INSTANCE_NAME \
  --namespace=$NAMESPACE \
  --set frontend.image=$FRONTEND_IMAGE \
  --set backend.image=$BACKEND_IMAGE \
  --set chaincode.image=$CHAINCODE_IMAGE \
  --set dataImporter.image=$DATAIMPORTER_IMAGE \
  --set endorsingPeer.image=$ENDORSING_PEER_IMAGE \
  --set gospelBackend.image=$GOSPEL_BACKEND_IMAGE \
  --set keystore.image=$KEYSTORE_IMAGE \
  --set nginx.image=$NGINX_IMAGE \
  --set orchestrator.image=$ORCHESTRATOR_IMAGE \
  --set orderer.image=$ORDERER_IMAGE \
  --set signer.image=$SIGNER_IMAGE \
  --set soas.image=$SOAS_IMAGE \
  --set soau.image=$SOAU_IMAGE \
  --set soi.image=$SOI_IMAGE \
  --set installer.serviceAccount=$INSTALLER_SERVICE_ACCOUNT > "${APP_INSTANCE_NAME}_manifest.yaml"
```

#### Apply the manifest to your Kubernetes cluster

Use `kubectl` to apply the manifest to your Kubernetes cluster:

```shell script
kubectl apply -f "${APP_INSTANCE_NAME}_manifest.yaml" --namespace "${NAMESPACE}"
```

#### View the app in the Google Cloud Console
By default, the application is exposed externally. To get the GCP Console URL for your installer, run the following command:

```shell script
echo "https://console.cloud.google.com/kubernetes/application/${ZONE}/${CLUSTER}/${NAMESPACE}/${APP_INSTANCE_NAME}"
```

Optionally, you may use the flag

```shell script
  --set publicIP=false
```

in the `helm template` command to ensure that the installer runs with a ClusterIP, which will not be exposed to the outside world.
In this scenario, to get access to the Gospel UI, run the following command:

```shell script
kubectl port-forward --namespace $NAMESPACE svc/$APP_INSTANCE_NAME 8080:80
```

and then go to the address `127.0.0.1:8080` in your web browser.

# Basic Usage

When the installer has finished you will be presented with a page with instructions on how to access the Gospel admin console interface and instructions on:

-   how to log in

-   Where to find the Gospel user guide

-   how to access the Gospel SDK and API documents

# Scaling

The Gospel installer does not support scaling.

# Backup and restore

Instructions for how to backup your Gospel deployment can be found in the [Gospel deployment and operation guide.](https://deployment.gospel.tech/gospel-operations-guide/)

# Updates

Gospel have a release lifecycle policy of:

-   2 Major releases per year

-   2 minor releases per year

Please contact your Gospel account manager for more details on the release cycle.

# Uninstalling the Application

## Using the Google Cloud Platform Console

1. In the GCP Console, open [Kubernetes Applications](https://console.cloud.google.com/kubernetes/application).

1. From the list of applications, click **gospel-technology-1**.

1. On the Application Details page, click **Delete**.

## Using the command line

### Prepare the environment

Set your installation name and Kubernetes namespace:

```shell script
export APP_INSTANCE_NAME=gospel-technology-1
export NAMESPACE=gospel
```

### Delete the resources

> **NOTE:** We recommend to use a `kubectl` version that is the same as the version of your cluster. Using the same versions of `kubectl` and the cluster helps avoid unforeseen issues.

To delete the resources, use the expanded manifest file used for the
installation.

Run `kubectl` on the expanded manifest file:

```shell script
kubectl delete -f ${APP_INSTANCE_NAME}_manifest.yaml --namespace $NAMESPACE
```

Otherwise, delete the resources using types and a label:

```shell script
kubectl delete application,statefulset,service,pvc,secret \
  --namespace $NAMESPACE \
  --selector app.kubernetes.io/name=$APP_INSTANCE_NAME
```

### Delete the persistent volumes of your installation

By design, the removal of StatefulSets in Kubernetes does not remove
PersistentVolumeClaims that were attached to their Pods. This prevents your
installations from accidentally deleting stateful data.

To remove the PersistentVolumeClaims with their attached persistent disks, run
the following `kubectl` commands:

```shell script
for pv in $(kubectl get pvc --namespace $NAMESPACE \
  --selector app.kubernetes.io/name=$APP_INSTANCE_NAME \
  --output jsonpath='{.items[*].spec.volumeName}');
do
  kubectl delete pv/$pv --namespace $NAMESPACE
done

kubectl delete persistentvolumeclaims \
  --namespace $NAMESPACE \
  --selector app.kubernetes.io/name=$APP_INSTANCE_NAME
```

### Delete the GKE cluster

Optionally, if you don't need the deployed application or the GKE cluster,
delete the cluster using this command:

```shell script
gcloud container clusters delete "$CLUSTER" --zone "$ZONE"
```
