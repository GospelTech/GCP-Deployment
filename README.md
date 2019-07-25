Overview
========

The Gospel Data Platform is a shared store of enterprise information spread amongst a number of entities used in the form of a graph database. Access control is enforced at the data layer, based on the context of who the user is, what they are doing and how they are doing it. Consensus amongst the peers of the underlying blockchain network is used to calculate and enforce read and write access control and a specially designed external component (the “keystore”) is used to ensure that the consensus mechanism cannot be bypassed.

Gospel is deployed by configuring a least 3 LedgerNodes with your partners in the GCP service, for redundancy, each LedgerNode can be located in a separate region.

A LedgerNode is a Kubernetes deployment consisting of multiple containers that communicate with each other. A load balancer can be configured in each region to forward traffic and is configured with a static public IP address. Your GCP service also ensures that all outgoing traffic shares the same source address as the load balancer. Nginx acts as a reverse proxy to direct traffic to the Gospel components within the Kubernetes deployment.

Gospel Components
-----------------

**LedgerNodes**

LedgerNodes are the name given to a Gospel deployment with one or more of the components below. A LedgerNode typically contains a Peer, a CA and an Orderer.

**Peers**

Peers are the core part of Gospel that, together with the Orderer, form the private, permissioned blockchain. The peers participate actively in the blockchain to create consensus on transactions. They store data (blockchain) and the data access rules (chaincode) in Gospel.

**Orderer**

The Orderer ensures that transactions are processed in the same sequence to maintain the integrity of the network. In other words, the Orderer outputs the same messages to all connected peers in the same logical order.

**Certificate Authority (CA)**

The Gospel CA is used to authenticate end users to Gospel. Once a user has completed the authentication process, they are provided with a certificate that is used to validate their transactions for the session. The CA supports many plugins for authentication, authorisation and assurance.

**Nginx**

Nginx has two purposes. When a user first connects, Nginx is a reverse proxy that will send them to their intended destination. This could be the Gospel backend or CA. Where a user is connecting to the Gospel administration UI, Nginx will also serve this to the browser.

**Backend**

The backend is a middleware component within Gospel. It is the endpoint for API connections and its purpose is to send transactions to read/write data through the Gospel process on behalf of an authenticated user.

**Keystore**

The Keystore is a required component of Gospel and is part of the secure process to enforce consensus on reads. This component runs outside of LedgerNode and is deployed for each multi-tenant namespace. It is a lightweight Go application that runs on a VM or Kubernetes Pod. The specific details of this module are patent pending.

Installing Gospel from the Google Marketplace is achieved clicking on the installation button and following the steps in the installation wizard. By following the steps in this wizard you will be able to deploy your own fully operational Gospel infrastructure in a few clicks.

Installation
------------

Quick install with Google Cloud Marketplace
-------------------------------------------

Click **Configure**.

Select the Kubernetes cluster that you want to deploy the app to. If you want to create a new cluster, click **Create cluster**

Select a Namespace to use for the application

In the **App instance name** box, enter a name for the app, such as `sandbox-dev-app`. The name must be unique within the namespace..

Enter a name for your app instance, and then click **Deploy**.

Gospel is deployed by taking you through a step-by-step installer. When the installer starts you will be given 3 options:

-   Install a demonstration network - This installs a Gospel instance pre-configured with data for you to quickly test the features of the platform

-   Join an existing network - Use this option if you are intending to join an existing Gospel network

-   Create a new network - Use this option when you and your partners are setting up a Gospel network for the first time

Choose an option related to the type of installation you require and [follow the on-screen instructions](https://gospelGCP-place-holder-url.com) to deploy your Gospel instance.

Command line instructions
-------------------------

To deploy the app from the command line, you need to download a license file from Google Cloud Platform. When you deploy the app using the steps below, you must apply the license file to your `kubectl` config.

Follow the steps below to deploy the app:

Connect to your Kubernetes cluster

Add the license file to your `kubectl` configuration

Clone the Git repository for the app

Verify the app's configuration and deploy the app

Gospel is then deployed by taking you through a step-by-step installer. When the installer starts you will be given 3 options:

-   Install a demonstration network - This installs a Gospel instance pre-configured with data for you to quickly test the features of the platform

-   Join an existing network - Use this option if you are intending to join an existing Gospel network

-   Create a new network - Use this option when you and your partners are setting up a Gospel network for the first time

Choose an option related to the type of installation you require and [follow the on-screen instructions](https://gospelGCP-place-holder-url.com) to deploy your Gospel instance.

**Basic Usage**

When the installer has finished you will be presented with a page with instructions on how to access the Gospel admin console interface and instructions on:

-   how to log in

-   Where to find the Gospel user guide

-   how to access the Gospel SDK and API documents

**Backup**

Instructions for how to backup your Gospel deployment can be found in the [Gospel deployment and operation guide.](https://deployment.gospel.tech/gospel-operations-guide/)

**Updates**

Gospel have a release lifecycle policy of:

-   2 Major releases per year

-   2 minor releases per year

Please contact your Gospel account manager for more details on the release cycle.

**Deletion**

Using the Google Cloud Platform Console

In the GCP Console, open [Kubernetes Applications](https://console.cloud.google.com/kubernetes/application).

From the list of applications, click **Gospel**.

On the Application Details page, click **Delete**.