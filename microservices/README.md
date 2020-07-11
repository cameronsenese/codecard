![](images/helidon.png)

## Introduction
In this guide, we'll show you how to deploy a Helidon microsrvice to Oracle Cloud Infrastructure's Kubernetes Engine. You can then configure your Code Card to invoke the microservice over the internet!

The path that we will take is as follows:

- Deploy a Kubernetes cluster to Oracle Cloud Infrastructure
- Prepare our development environment
- Build and push Helidon microservice container image to the OCI container registry
- Deploy the Helidon microservice to Oracle Container Engine (OKE)
- Configure your Code Card to invoke the Helidon microservice

### About Helidon
Helidon is a collection of Java libraries for writing microservices that run on a fast web core powered by Netty. Helidon is designed to be simple to use, with tooling and examples to get you going quickly.

Helidon supports MicroProfile, and provides familiar APIs like JAX-RS, CDI and JSON-P/B.
The MicroProfile implementation runs on the fast Helidon Reactive WebServer.

With support for health checks, metrics, tracing and fault tolerance, Helidon has what you need to write cloud ready applications that integrate with Prometheus, Jaeger/Zipkin and Kubernetes.

Check out the Helidon [Docs & Guides](https://helidon.io/docs/latest/#/about/01_overview) to learn more.

## Tutorial Prerequisites
Prerequisites to be completed before continuing on with this tutorial:
 - You will need to have deployed your OKE Kubernetes cluster prior to commencing implementation of the deployment scenario. Follow the link to [this tutorial](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/oke-full/index.html) for guidance on the process. *Note: Be sure to provision your cluster to a public Virtual Cloud Network.*
 - Create a 'kube config' authentication artefact. This will be used later in the tutorial to connect to the OKE cluster. Follow the link to [this tutorial](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/oke-full/index.html#DownloadthekubeconfigFilefortheCluster) for guidance on the process.
 - Install Docker on the host from which you will be following this work instruction. If you are running Oracle Linux, follow the link to [this tutorial](https://blogs.oracle.com/blogbypuneeth/a-simple-guide-to-docker-installation-on-oracle-linux-75) for guidance on the process.

## Clone the 'codecard' git repository
With the prerequisites all in place - now clone the `codecard` git repository to obtain the required collateral to build the Helidon microservice:

``` bash
git clone https://github.com/cameronsenese/codecard.git
```

Commands from this point forward will assume that you are in the `../codecard/microservices/` directory.

## Build the Helidon container image
Build the Helidon container image using the `docker build` command:

```bash
docker build -t helidon-mp .
```

## Push the Helidon container image to the OCI registry
In this next step, we'll push the local copy of the microservice image up to the cloud.
*For a great walkthrough on how to use the OCI Registry service, check out [this article](https://www.oracle.com/webfolder/technetwork/tutorials/obe/oci/registry/index.html).*

You will need to log into your Oracle Cloud Infrastructure console. Your user will either need to be a part of the tenancy's Administrators group, or another group with the `REPOSITORY_CREATE` permission.

After confirming you have the proper permissions, generate an auth token for your user. *Be sure to take a copy of the token as you will not be able to access it again.*

In the OCI console, navigate to the `Developer Services` | `Registry (OCIR)` tab, and select  the OCI region to which you would like to push the image. This should be the same region into which you provisioned your OKE cluster.

### Log into the OCI registry
Log into the OCI registry in your development environment using the docker login command:

```bash
docker login <region-key>.ocir.io
```

`<region-key>` corresponds to the code for the Oracle Cloud Infrastructure region you're using. See the [this](https://docs.cloud.oracle.com/iaas/Content/General/Concepts/regions.htm) reference for the available regions and associated keys.

When prompted, enter your username in the format `<tenancy_name>/<username>`. When prompted, enter the auth token you copied earlier as the password.

### Tag the Helidon image
Next we'll tag the Helidon image that we're going to push to the OCI registry:

```bash
docker tag helidon-mp:latest <region-code>.ocir.io/<tenancy-name>/<repo-name>/helidon-mp:latest
```

### Push the image to the OCI registry
And now we'll use the `docker push` command to push the conainer image to the OCI registry:

```bash
docker push <region-code>.ocir.io/<tenancy-name>/<repo-name>/helidon-mp:latest
```

Within the OCI console Registry UI you will now be able to see the newly created repository & image.

## Deploy the Helidon microservice to OKE
In this section, we will deply the microsoervice to your Kubernetes cluster.

### Create namespace
Let's first create a Kubernetes namespace for this project called `helidon`.

```bash
kubectl create namespace helidon
```

### Update the application manifest
Before deploying the application, we need to update the yaml descriptor for the application to include the correct path to your container image. Open the file `app.yaml`, and replace the bracketed values in the image path on line 46:

```bash
image: <region-code>.ocir.io/<tenancy-name>/<repo-name>/helidon-mp:latest
```

*Remember to make sure that your registry is shared as public.*

### Deploy the application
And next deploy the service.

```bash
kubectl create -f app.yaml -n helidon
```

## Configure the CodeCard to invoke the Helidon microservice
Before we go ahead and configure your CodeCard to invoke the microservice, we need to collect a few details about the service running in the Kubernetes cluster.

### Obtain the NodePort for the service
First, obtain the network port that the service is listening on. From your development environment, run the following command.

```bash
kubectl get svc -n helidon
```

The output will be similar to the following:

```bash
NAME        TYPE     CLUSTER-IP     EXTERNAL-IP PORT(S)          AGE
helidon-mp  NodePort 10.96.232.218  <none>      9081:32690/TCP   3d7h
```

Record the port number exposed as a Kubernetes NodePort, which is `32690` per the example above.

### Obtain the external IP address for the service
Next we obtain the external IP address at which the service can be contacted. From your development environment, run the following command.

```bash
kubectl get nodes
```
The output will be similar to the following:

```bash
NAME        STATUS   ROLES   AGE     VERSION
10.0.10.2   Ready    node    3d10h   v1.13.5
```

Record the name of the worker node, which is `10.0.10.2` per the example above.

With the name of the worker node, run the following command.

```bash
kubectl describe node <NodeName>
```

The output will be similar to the following (truncated) summary.

```bash
Addresses:
  InternalIP:  10.0.10.2
  ExternalIP:  129.213.19.161
```
Locate and and record the value for `ExternalIP`, which is `129.213.19.161` per the example above.

With the information that we have collected, we can now construct a http request to invoke the microservice. The format is as follows:

```bash
http://<ExternalIP>:<NodePort>/HelidonMP/<DevName>
```

*Note: `<DevName>` should be populated with the name of the developer implementing the solution, this will form a part of the response data returned by the microservice.*

### Configure Code Card
The final step is to configure the Code Card to invoke the microservice!
In this example, we will program the `shortpress` action for button `B` on the card.

#### Establish serial connection with Code Card
In order to configure our Code Card, we need to establish a serial connection over USB to the CodeCard CLI. Follow [this guide](https://github.com/cameronsenese/codecard/blob/master/terminal/README.md#connect-via-terminal-emulator) to establish the serial over USB connection. Remember to ensure that the Code Card WiFi settings are configure correctly also! (Direction available from the referenced guide).

#### Configure `buttonb1` button action
*In the Code Card CLI, `buttonb1` correlates to button B shortpress action.*

In your terminal session you should now see the CodeCard CLI Menu, as follows.

```bash
***************************************************************************************
  Code Card v1.0
  Oracle Groundbreakers
  developer.oracle.com/codecard
***************************************************************************************
Commands:
  ls                Show all stored key/values
  help              Show this help
  shortpress[a|b]   Simulate the press of a button
  longpress[a|b]    Simulate the long press of a button
  connect           Connect to wifi
  disconnect        Disconnect wifi
  restart           Restart wifi
  status            Show wifi status
  home              Show home screen
  reset             Reset to factory settings

Usage:
  Read saved key value:
    key
  Save new key value:
    key=[value]

Available keys:
  ssid, password, buttona1, buttona2, buttonb1, buttonb2, fingerprinta1, fingerprinta2,
  fingerprintb1, fingerprintb2, methoda1, methoda2, methodb1, methodb2,
>>>
```

First, we will set the HTTP method for the B shortpress by entering the following command.
*Keep in mind that pausing for 2 seconds while typing will automatically enter the command. It may be easier to pre-type the commands elsewhere and copy-paste them into the window.*

```bash
methodb1=GET
```

Code Card will confirm setting update as follows.

```bash
>>>
Value saved for methodb1: GET
>>>
```

Next configure the HTTP endpoint for the B shortpress by entering the following command. Be sure to substitute values in `<brackets>` as appropriate.

```bash
buttonb1=http://<ExternalIP>:<NodePort>/HelidonMP/<DevName>
```

Code Card will confirm setting update as follows.

```bash
>>>
Value saved for buttonb1: http://<ExternalIP>:<NodePort>/HelidonMP/<DevName>
>>>
```

### Invoke the microservice from the Code Card
Ok, so now our Helidon microservice and Code Card are ready to Go! Powercycle your Code Card and perform a button B shortpress. If your card is still connected via the serial connection, you will see output similar to the following.

```bash
Button b - short pressed
>>>
Connecting to 'pmac851' ...................connected!
IP address: 192.168.43.13
MAC address: 84:0D:8E:A7:89:5B
>>>
Request:
  host: 129.213.19.161
  port: 32690
  url: http://129.213.19.161:32690/HelidonMP/Cameron
  method: GET
text/plain;charset=UTF-8
Response:
  {"template":"template1","title":"Hello Cameron!!","subtitle":"Pleased to meet you!","bodytext":"This is your first Helidon Microservice from the Oracle Cloud!","icon":"microservice","backgroundColor":"white"}
>>>
```
