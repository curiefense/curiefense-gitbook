# Istio via Helm

## Introduction

The instructions below show how to install Curiefense on a Kubernetes cluster, embedded in an Istio service mesh. 

The following tasks, each described below in sequence, should be performed:

* [Clone the Helm Repository](istio-via-helm.md#clone-the-helm-repository)
* [Setup Synchronization](istio-via-helm.md#markdown-header-prerequisites)
* [Create a Kubernetes Cluster Running Helm](istio-via-helm.md#create-a-kubernetes-cluster)
* [Reset State](istio-via-helm.md#reset-state)
* [Create Namespaces](istio-via-helm.md#create-namespaces)
* [Setup Secrets](istio-via-helm.md#setup-secrets)
* [Setup TLS](istio-via-helm.md#setup-tls-for-the-ui-server)
* [Deploy Istio and Curiefense Images](istio-via-helm.md#deploy-istio-and-curiefense-images)
* [Deploy the (Sample) App](istio-via-helm.md#deploy-the-sample-app)
* [Expose Curiefense Services Using NodePorts](istio-via-helm.md#markdown-header-expose-curiefense-services-using-nodeports)
* [Access Curiefense Services](istio-via-helm.md#markdown-header-access-curiefense-services)

At the bottom of this page is a [Reference section](istio-via-helm.md#markdown-header-description-of-the-pods) describing the charts and configuration variables.

During this process, you might find it helpful to read the descriptions (which include the purpose, secrets, and network/port details) of the services and their containers: [Services and Container Images](../../reference/services-container-images.md)

## Clone the Helm Repository

Clone the repository, if you have not already done so:

```
git clone https://github.com/curiefense/curiefense-helm.git
```

This documentation assumes it has been cloned to `~/curiefense-helm`.

## Setup Synchronization <a href="markdown-header-prerequisites" id="markdown-header-prerequisites"></a>

An AWS S3 bucket must be available to synchronize configurations between the `confserver` and the Curiefense Istio sidecars. The following Curiefense variables must be set:

* In `~/curiefense-helm/istio-helm/charts/gateways/istio-ingress/values.yaml`:
  * Set`curieconf_manifest_url` to the bucket URL.
* In `~/curiefense-helm/curiefense-helm/curiefense/values.yaml`:
  * Set `curieconf_manifest_url` to the bucket URL.

## Create a Kubernetes Cluster

Access to a Kubernetes cluster is required. Dynamic provisioning of persistent volumes must be supported. To set a StorageClass other than the default, change or override variable `storage_class_name` in `~/curiefense-helm/curiefense-helm/curiefense/values.yaml`.

Below are instructions for several ways to achieve this:

* Using minikube, Kubernetes 1.20.2 (dynamic provisioning is enabled by default)
* Using Google GKE, Kubernetes 1.16.13 (RBAC and dynamic provisioning are enabled by default)
* Using Amazon EKS, Kubernetes 1.18 (RBAC and dynamic provisioning are enabled by default)

You will need to install the following clients:

* Install kubectl ([https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)) -- use the same version as your cluster.
* Install Helm v3 ([https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/))

### Option 1: Using minikube

This section describes the install for a single-node test setup (which is generally not useful for production).

#### Install minikube

Starting from a fresh ubuntu 21.04 VM:

* Install docker ([https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)), and allow your user to interact with docker with `sudo usermod -aG docker $USER && newgrp docker`
* Install minikube ([https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/))

```
minikube start --kubernetes-version=v1.20.2 --driver=docker --memory='8g' --cpus 6
minikube addons enable ingress
```

Start a `screen` or `tmux`, and keep the following command running:

```
minikube tunnel
```

### Option 2: Using Google GKE

#### **Create a cluster**

```
gcloud container clusters create curiefense-gks --num-nodes=1 --machine-type=n1-standard-4
gcloud container clusters get-credentials curiefense-gks
```

### Option 3: Using Amazon EKS

**Create a cluster**

```
eksctl create cluster --name curiefense-eks-2 --version 1.18 --nodes 1 --nodes-max 1 --managed --region us-east-2 --node-type m5.xlarge
```

## Reset State

If you have a clean machine where Curiefense has never been installed, skip this step and go to the next.

Otherwise, run these commands:

```
helm delete curiefense
helm delete -n curiefense curiefense
helm delete -n istio-system istio-ingress
helm delete -n istio-system istiod
helm delete -n istio-system istio-base
```

Ensure that `helm ls -a --all-namespaces` outputs nothing.

## Create Namespaces

Run the following commands:

```
kubectl create namespace curiefense
kubectl create namespace istio-system
```

## Setup storage

Curiefense's confserver exports configurations to object storage services, from which they are retrieved by curieproxy. Four backends are currently supported: AWS S3, Google Cloud Storage, minio (which can be self-hosted), or local storage (for single-node test deployments). To use curiefense, you must pick one, and define Secrets that allow interacting with the chosen storage service (except for local storage).

### Option 1:  AWS credentials

Encode the AWS S3 credentials that have r/w access to the S3 bucket. This yields a base64 string:

```
cat << EOF | base64 -w0
[default]
access_key = xxxxxxxxxxxxxxxxxxxx
secret_key = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
EOF
```

Create a local file called `s3cfg.yaml`, with the contents below, replacing both occurrences of `BASE64_S3CFG` with the previously obtained base64 string:

```
---
apiVersion: v1
kind: Secret
data:
  s3cfg: "BASE64_S3CFG"
metadata:
  namespace: curiefense
  labels:
    app.kubernetes.io/name: s3cfg
  name: s3cfg
type: Opaque
---
apiVersion: v1
kind: Secret
data:
  s3cfg: "BASE64_S3CFG"
metadata:
  namespace: istio-system
  labels:
    app.kubernetes.io/name: s3cfg
  name: s3cfg
type: Opaque
```

Deploy this secret to the cluster:

```
kubectl apply -f s3cfg.yaml
```

### Option 2: Google Cloud Storage credentials

Create a bucket, and a service account that has read/write access to the bucket. Obtain a private key for this account, which should look like this:

```
{
  "type": "service_account",
  "project_id": "PROJECT",
  "private_key_id": "1234abcd1234abcd1234abcd1234abcd1234abcd",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIE.....ABCD=\n-----END PRIVATE KEY-----\n",
  "client_email": "....@PROJECT.iam.gserviceaccount.com",
  "client_id": "123412341234123412341",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/....%40PROJECT.iam.gserviceaccount.com"
}
```

Create a local file called `gs.yaml`, with the contents below, replacing both occurrences of `BASE64_GS_PRIVATE_KEY` with the previously obtained base64 string:

```
---
apiVersion: v1
kind: Secret
data:
  gs.json: "BASE64_GS_PRIVATE_KEY"
metadata:
  labels:
    app.kubernetes.io/name: gs
  name: gs
  namespace: curiefense
type: Opaque
---
apiVersion: v1
kind: Secret
data:
  gs.json: "BASE64_GS_PRIVATE_KEY"
metadata:
  labels:
    app.kubernetes.io/name: gs
  name: gs
  namespace: istio-system
type: Opaque
```

Deploy this secret to the cluster:

```
kubectl apply -f gs.yaml
```

Set the `curieconf_manifest_url` variables in `curiefense-helm/curiefense/values.yaml` and `istio-helm/charts/gateways/istio-ingress/values.yaml` to the following URL: `gs://BUCKET_NAME/prod/manifest.json` (replace BUCKET_NAME with the actual name of the bucket).

Also set the `curiefense_bucket_type` variables in the same values.yaml files to `gs`.

### Option 3: minio credentials

Install a minio server, create a bucket and a Service Account that has read/write permissions to that bucket. The curiefense helm charts may be used to deploy such a minio server (single-node, default credentials, for testing).

Encode the minio credentials that have r/w access to the bucket. This yields a base64 string:

```
cat << EOF | base64 -w0
[default]
access_key = minioadmin
secret_key = minioadmin
EOF
```

Create a local file called `miniocfg.yaml`, with the contents below, replacing both occurrences of `BASE64_MINIOCFG` with the previously obtained base64 string:

```
 ---
apiVersion: v1
kind: Secret
data:
  miniocfg: "BASE64_MINIOCFG"
metadata:
  labels:
    app.kubernetes.io/name: miniocfg
  name: miniocfg
  namespace: curiefense
type: Opaque
---
apiVersion: v1
kind: Secret
data:
  miniocfg: "BASE64_MINIOCFG"
metadata:
  labels:
    app.kubernetes.io/name: miniocfg
  name: miniocfg
  namespace: istio-system
type: Opaque
```

Deploy this secret to the cluster:

```
kubectl apply -f miniocfg.yaml
```

An example miniocfg.yaml file is provided in  `~/curiefense-helm/curiefense-helm/example-miniocfg.yaml`. It contains default credentials for minio, that will work with the minio installation that is provided in the curiefense helm charts.

Set the `curieconf_manifest_url` variables in `curiefense-helm/curiefense/values.yaml` and `istio-helm/charts/gateways/istio-ingress/values.yaml` to the following URL: `minio://BUCKET_NAME/prod/manifest.json` (replace BUCKET_NAME with the actual name of the bucket; use `curiefense-minio-bucket` with the minio installation that is provided in the curiefense helm charts).

Also set the `curiefense_bucket_type` variables in the same values.yaml files to `minio`.

### Option 4: local bucket

For clusters where all istio ingress proxies as well as the confserver run on the same kubernetes  node (typically test environments), a simple `hostPath` volume can be used. It is mounted to `/bucket` on the host machine, as well as in relevant containers.

Set the `curieconf_manifest_url` variables in `curiefense-helm/curiefense/values.yaml` and `istio-helm/charts/gateways/istio-ingress/values.yaml` to the following URL: `file:///bucket/prod/manifest.json`.

Also set the `curiefense_bucket_type` variables in the same values.yaml files to `local-bucket`.

## Setup TLS for the UI server

Using TLS is optional. Follow these steps if only if you want to use TLS for communicating with the UI server, and you do not rely on istio to manage TLS.

The UIServer can be made to be reachable over HTTPS. To do that, two secrets have to be created to hold the TLS certificate and TLS key.

Create a local file called `uiserver-tls.yaml`, replacing `TLS_CERT_BASE64` with the base64-encoded PEM X509 TLS certificate, and `TLS_KEY_BASE64` with the base64-encoded TLS key.

```
---
apiVersion: v1
data:
  uisslcrt: TLS_CERT_BASE64
kind: Secret
metadata:
  labels:
    app.kubernetes.io/name: uisslcrt
  name: uisslcrt
  namespace: curiefense
type: Opaque
---
apiVersion: v1
data:
  uisslkey: TLS_KEY_BASE64
kind: Secret
metadata:
  labels:
    app.kubernetes.io/name: uisslkey
  name: uisslkey
  namespace: curiefense
type: Opaque
```

Deploy this secret to the cluster:

```
kubectl apply -f uiserver-tls.yaml
```

An example file with self-signed certificates is provided at `~/curiefense-helm/curiefense-helm/example-uiserver-tls.yaml`.

## Deploy Istio and Curiefense Images

Deploy the Istio service mesh:

```
cd ~/curiefense-helm/istio-helm 
DOCKER_TAG=main ./deploy.sh
```

And then the Curiefense components:

```
cd ~/curiefense-helm/curiefense-helm
DOCKER_TAG=main ./deploy.sh
```

## Deploy the (Sample) App

The application to be protected by Curiefense should now be deployed. These instructions are for the sample application `bookinfo` which is deployed in the `default` kubernetes namespace. Installation instructions are summarized below. More detailed instruction are available [on the istio website](https://istio.io/v1.9/docs/examples/bookinfo/).

### Enable Istio injection

Add the `istio-injection=enabled` label that will make Istio automatically inject necessary sidecars to applications that are deployed in the `default` namespace.

```
kubectl label namespace default istio-injection=enabled
```

### Install the application

```
cd ~
wget 'https://github.com/istio/istio/releases/download/1.9.3/istio-1.9.3-linux-amd64.tar.gz'
tar -xf istio-1.9.3-linux-amd64.tar.gz
cd ~/istio-1.9.3/
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

### Test bookinfo <a href="markdown-header-test-bookinfo" id="markdown-header-test-bookinfo"></a>

Check that `bookinfo` Pods are running (wait a bit if they are not):

```
kubectl get pod -l app=ratings
```

Sample output example:

```
NAME                         READY   STATUS    RESTARTS   AGE
ratings-v1-f745cf57b-cjg69   2/2     Running   0          79s
```

Check that the application is working by querying its API directly without going through the Istio service mesh:

```
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

Expected output:

```
<title>Simple Bookstore App</title>
```

### Test access to bookinfo through Istio <a href="markdown-header-test-access-to-bookinfo-through-istio" id="markdown-header-test-access-to-bookinfo-through-istio"></a>

Set the GATEWAY_URL variable by following instructions on the [Istio website](https://istio.io/v1.9/docs/examples/bookinfo/#determine-the-ingress-ip-and-port).

Alternatively, with minikube, this command can be used instead:

```
export GATEWAY_URL=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'):80
```

Check that bookinfo is reachable through Istio:

```
curl -sS http://$GATEWAY_URL/productpage | grep -o "<title>.*</title>"
```

Expected output:

```
<title>Simple Bookstore App</title>
```

If this error occurs: `Could not resolve host: a6fdxxxxxxxxxxxxxxxxxxxxxxxxxxxx-xxxxxxxx.us-west-2.elb.amazonaws.com` ...the ELB is not ready yet. Wait and retry until it becomes available (typically a few minutes).

### Check that logs stored in the Elasticsearch cluster <a href="markdown-header-check-that-logs-reach-the-accesslog-ui" id="markdown-header-check-that-logs-reach-the-accesslog-ui"></a>

Run this query to access the protected website, bookinfo, and thus generate an access log entry:

```
curl http://$GATEWAY_URL/TEST_STRING
```

Run this to ensure that the logs have been recorded:

```
kubectl exec -ti -n curiefense elasticsearch-0 -- curl http://127.0.0.1:9200/_search -H "Content-Type: application/json" -d '{"query": {"bool": {"must":{"match":{"request.attributes.uri": "/TEST_STRING"}}}}}'|grep -Eo '"uri":"/TEST_STRING"'
```

Expected output:.

```
"uri":"/TEST_STRING"
```

## Expose Curiefense Services Using NodePorts <a href="markdown-header-expose-curiefense-services-using-nodeports" id="markdown-header-expose-curiefense-services-using-nodeports"></a>

Run the following commands to expose Curiefense services through NodePorts. Warning: if the machine has a public IP, the services will be exposed on the Internet.

Start with this command:

```
kubectl apply -f ~/curiefense-helm/curiefense-helm/expose-services.yaml
```

The following command can be used to determine the IP address of your cluster nodes on which services will be exposed:

```
kubectl get nodes -o wide
```

### For minikube only: <a href="markdown-header-minikube" id="markdown-header-minikube"></a>

If you are using minikube, also run the following commands on the host in order to expose services on the Internet (ex. if you are running this on a cloud VM):

```
sudo iptables -t nat -A PREROUTING -p tcp --match multiport --dports 30000,30080,30300,30443 -j DNAT --to $(minikube ip)
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to $(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
sudo iptables -I FORWARD -p tcp --match multiport --dports 80,30000,30080,30300,30443,30444 -j  ACCEPT
```

### For Amazon EKS only: <a href="markdown-header-amazon-eks" id="markdown-header-amazon-eks"></a>

If you are using Amazon EKS, you will also need to allow inbound connections for port range 30000-30500 from your IP. Go to the EC2 page in the AWS console, select the EC2 instance for the cluster (named `curiefense-eks-...-Node`), select the "Security" pane, select the security group (named `eks-cluster-sg-curiefense-eks-[0-9]+`), then add the incoming rule.

## Access Curiefense Services <a href="markdown-header-access-curiefense-services" id="markdown-header-access-curiefense-services"></a>

The UIServer is now available on port 30080 over HTTP, and on port 30443 over HTTPS.

Grafana is now available on port 30300 over HTTP.

For the `bookinfo` sample app, the Book Review product page is now available on port 80 over HTTP, and on port 30444 over HTTPS. Try reaching `http://IP/productpage`.

The confserver is now available on port 30000 over HTTP: try reaching `http://IP:30000/api/v1/`.

For a full list of ports used by Curiefense containers, see the [Reference page on services and containers](broken-reference).

## Reference: Description of Helm Charts <a href="markdown-header-description-of-the-pods" id="markdown-header-description-of-the-pods"></a>

### Curiefense charts <a href="markdown-header-curiefense-components" id="markdown-header-curiefense-components"></a>

Helm charts are divided as follows:

* `curiefense-admin` - confserver, and UIServer.
* `curiefense-dashboards` - Grafana and Prometheus.
* `curiefense-log` - elasticsearch, filebeat, fluentd, kibana, logstash.
* `curiefense-proxy` - curielogger and redis.

#### Chart configuration variables <a href="markdown-header-configuration-variables" id="markdown-header-configuration-variables"></a>

Configuration variables in `~/curiefense-helm/curiefense-helm/curiefense/values.yaml` can be modified or overridden to fit your deployment needs:

* Variables in the `images` section define the Docker image names for each component. Override this if you want to host images on your own private registry.
* `storage_class_name` is the StorageClass that is used for dynamic provisioning of Persistent Volumes. It defaults to `null` (default storage class, which works by default on EKS, GKE and minikube).
* `..._storage_size` variables define the size of persistent volumes. The defaults are fine for a test or small-scale deployment.
* `curieconf_manifest_url` is the URL of the AWS S3 or Google Cloud Storage bucket that is used to synchronize configurations between the `confserver` and the Curiefense Istio sidecars.
* `docker_tag` defines the image tag versions that should be used. `deploy.sh` will override this to deploy a version that matches the current working directory, unless the `DOCKER_TAG` environment variable is set.

### Istio chart <a href="markdown-header-istio" id="markdown-header-istio"></a>

Components added or modified by Curiefense are defined in `~/curiefense-helm/istio-helm/charts/gateways/istio-ingress/`. Compared to the upstream Istio Kubernetes distribution, we add or change the following Pods:

* An `initContainer` called `curiesync-initialpull` has been added. It synchronizes configuration before running Envoy.
* A container called `curiesync` has been added. It periodically fetches the configuration that should be applied from an S3 or GS bucket (configurable with the `curieconf_manifest_url` variable), and makes it available to Envoy. This configuration is used by the LUA code that inspects traffic.
* The container called `istio-proxy` now uses our custom Docker image, embedding our HTTP Filter, written in Lua.
* An `EnvoyFilter` has been added. It forwards access logs to `curielogger` (see `curiefense_access_logs_filter.yaml`).
* An `EnvoyFilter` has been added. It runs Curiefense's Lua code to inspect incoming traffic on the Ingress Gateways (see `curiefense_lua_filter.yaml`).

#### Chart configuration variables <a href="markdown-header-configuration-variables_1" id="markdown-header-configuration-variables_1"></a>

Configuration variables in `~/curiefense-helm/istio-helm/charts/gateways/istio-ingress/values.yaml` can be modified or overridden to fit your deployment needs:

* `gw_image` defines the name of the image that contains our filtering code and modified Envoy binary.
* `curiesync_image` defines the name of the image that contains scripts that synchronize local Envoy configuration with the AWS S3 bucket defined in `curieconf_manifest_url`.
* `curieconf_manifest_url` is the URL of the AWS S3 bucket that is used to synchronize configurations between the `confserver` and the Curiefense Istio sidecars.
* `curiefense_namespace` should contain the name of the namespace where Curiefense components defined in `~/curiefense-helm/curiefense-helm/` are running.
* `redis_host` defines the hostname of the redis server that will be used by `curieproxy`. Defaults to the provided redis StatefulSet. Override this to replace the redis instance with one you supply.
* `initial_curieconf_pull` defines whether a configuration should be pulled from the AWS S3 bucket before running Envoy (`true`), or if traffic should be allowed to flow with a default configuration until the next synchronization (typically every 10s).
