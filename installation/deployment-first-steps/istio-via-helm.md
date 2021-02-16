# Istio via Helm

The instructions below show how to install Curiefense on a Kubernetes cluster, embedded in an Istio service mesh. It assumes that the instructions described in [First Tasks](first-tasks.md) have been completed successfully.

The following tasks, each described below in sequence, should be performed:

* [Setup Synchronization](istio-via-helm.md#markdown-header-prerequisites)
* [Create a Kubernetes Cluster Running Helm](istio-via-helm.md#create-a-kubernetes-cluster-running-helm)
* [Reset State](istio-via-helm.md#reset-state)
* [Create Namespaces](istio-via-helm.md#create-namespaces)
* [Setup Secrets](istio-via-helm.md#setup-secrets)
* [Setup TLS](istio-via-helm.md#setup-tls)
* [Deploy Istio and Curiefense Images](istio-via-helm.md#deploy-istio-and-curiefense-images)
* [Deploy the \(Sample\) App](istio-via-helm.md#deploy-the-sample-app)
* [Expose Curiefense services Using NodePorts](istio-via-helm.md#markdown-header-expose-curiefense-services-using-nodeports)
* [Access Curiefense Services](istio-via-helm.md#markdown-header-access-curiefense-services)

At the bottom of this page is a [Reference section](istio-via-helm.md#markdown-header-description-of-the-pods) describing the charts and configuration variables.

## Setup Synchronization <a id="markdown-header-prerequisites"></a>

An AWS S3 bucket must be available to synchronize configurations between the `confserver` and the Curiefense Istio sidecars. The following Curiefense variables must be set:

* In `deploy/istio-helm/chart/values.yaml`:
  * Set`curieconf_manifest_url` to the bucket URL.
* In `deploy/curiefense-helm/curiefense/values.yaml`:
  * Set `curieconf_manifest_url` to the bucket URL.

## Create a Kubernetes Cluster Running Helm

Access to a Kubernetes cluster running Helm v2 is required. Dynamic provisioning of persistent volumes must be supported. To set a StorageClass other than the default, change or override variable `storage_class_name` in `deploy/curiefense-helm/curiefense/values.yaml`.

Below are instructions for several ways to achieve this:

* Using minikube, Kubernetes 1.14.9 and Helm v2.13.1 \(dynamic provisioning is enabled by default\)
* Using Google GKE, Kubernetes 1.16.13 \(with RBAC\) and Helm v2.16.7 \(dynamic provisioning is enabled by default\)
* Using Amazon EKS, Kubernetes 1.18 \(with RBAC\) and Helm v2.16.7 \(dynamic provisioning is enabled by default\)

### Option 1: Using minikube

This section describes the install for a single-node test setup \(which is generally not useful for production\).

#### Install minikube

Starting from a fresh ubuntu 20.04 VM:

* Install docker \([https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)\)
* Install kubectl \([https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/)\) -- use **version 1.14.9**.
* Install minikube \([https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)\)
* Allow your user to interact with docker: `sudo usermod -aG docker $USER && newgrp docker`

```text
minikube start --kubernetes-version=v1.14.9 --driver=docker --memory='8g' --cpus 6
eval $(minikube docker-env)
minikube addons enable ingress
```

Start a `screen` or `tmux`, and keep the following command running:

```text
minikube tunnel
```

#### **Install Helm v2.13.1**

Run the following commands:

```text
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh -v v2.13.1
# socat is required but not installed by get_helm.sh
apt install socat
```

\(Alternately, Helm can be manually downloaded as a binary release, as explained at [https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/). If you choose to do this, ensure that you obtain **v2.13.1**.\)

Now install Helm to the Kubernetes cluster:

```text
helm init
```

### Option 2: Using Google GKE

This option uses a more recent Kubernetes, with RBAC enabled.

#### Install kubectl

Follow instructions at [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/). Use **version 1.16.13**.

#### **Create a cluster**

```text
gcloud container clusters create curiefense-gks --num-nodes=1 --machine-type=n1-standard-4
gcloud container clusters get-credentials curiefense-gks
```

#### **Install Helm v2.16.7**

```text
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh -v v2.16.7
```

\(Alternately, Helm can be manually downloaded as a binary release, as explained at [https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/). If you choose to do this, ensure that you obtain **v2.16.7**.\)

Now we must define RBAC authorizations. Helm needs to be able to deploy applications to both the `curiefense` and `istio-system` namespaces.

To do that, we provide an example configuration, which installs Tiller in the `kube-system` namespaces, and grants it cluster-admin permissions.

```text
kubectl -n kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
```

Finally, install Helm to the Kubernetes cluster:

```text
helm init --service-account tiller
```

### Option 3: Using Amazon EKS

This option uses a more recent Kubernetes, with RBAC enabled.

#### Install kubectl

Follow instructions at [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/). Use **version 1.18**.

**Create a cluster**

```text
eksctl create cluster --name curiefense-eks-2 --version 1.18 --nodes 1 --nodes-max 1 --managed --region us-east-2 --node-type m5.xlarge
```

**Install Helm v2.16.7**

Follow all the "**Install Helm v2.16.7**" instructions shown above in the Google GKE section.

## Reset State

If you have a clean machine where Curiefense has never been installed, skip this step and go to the next.

Otherwise, run these commands:

```text
kubectl delete namespaces bookinfo
helm delete --purge curiefense
helm delete --purge istio-cf
```

Ensure that `helm ls -a` outputs nothing.

## Create Namespaces

Run the following commands:

```text
kubectl create namespace curiefense
kubectl create namespace istio-system
```

## Setup Secrets

### AWS credentials

Encode the AWS S3 credentials that have r/w access to the S3 bucket. This yields a base64 string:

```text
cat << EOF | base64 -w0
[default]
access_key = xxxxxxxxxxxxxxxxxxxx
secret_key = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
EOF
```

Create a local file called `s3cfg.yaml`, with the contents below, replacing both occurrences of `BASE64_S3CFG` with the previously obtained base64 string:

```text
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

```text
kubectl apply -f s3cfg.yaml
```

### Database configuration

Skip this section if you are using elasticsearch to store logs \(that is the default\).

Curiefense requires two database accounts. They will be automatically provisioned by the `logdb` container:

* One with read/write authorization \(described below as `BASE64_READWRITE_USERNAME` and`BASE64_READWRITE_PASSWORD`\). If the `logdb` container is used \(which is the default\), `BASE64_READWRITE_USERNAME` must be set to `postgres`.
* One with read-only permissions \(described below as

  `BASE64_READONLY_USERNAME` and `BASE64_READONLY_PASSWORD`\). If the `logdb` container is used \(which is the default\), `BASE64_READONLY_USERNAME` must be set to `logserver_ro`.

Create a local file called `dbsecret.yaml`, with the contents below, containing database credentials. Replace `BASE64_READWRITE_USERNAME`, `BASE64_READONLY_USERNAME`, `BASE64_READWRITE_PASSWORD` and `BASE64_READONLY_PASSWORD` with the correct base64-encoded values.

```text
---
apiVersion: v1
kind: Secret
data:
  username: "BASE64_READWRITE_USERNAME"
  password: "BASE64_READWRITE_PASSWORD"
metadata:
  namespace: curiefense
  labels:
    app.kubernetes.io/name: curiefense-db-credentials
  name: curiefense-db-credentials
type: Opaque
---
apiVersion: v1
kind: Secret
data:
  username: "BASE64_READONLY_USERNAME"
  password: "BASE64_READONLY_PASSWORD"
metadata:
  namespace: curiefense
  labels:
    app.kubernetes.io/name: curiefense-db-readonly-credentials
  name: curiefense-db-readonly-credentials
type: Opaque
```

Deploy this secret to the cluster:

```text
kubectl apply -f dbsecret.yaml
```

An example file with weak default credentials is provided at `deploy/curiefense-helm/example-dbsecret.yaml`.

## Setup TLS

Using TLS is optional.

The UIServer can be made to be reachable over HTTPS. To do that, two secrets have to be created to hold the TLS certificate and TLS key.

Create a local file called `uiserver-tls.yaml`, replacing `TLS_CERT_BASE64` with the base64-encoded PEM X509 TLS certificate, and `TLS_KEY_BASE64` with the base64-encoded TLS key.

```text
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

```text
kubectl apply -f uiserver-tls.yaml
```

An example file with self-signed certificates is provided at `deploy/curiefense-helm/example-uiserver-tls.yaml`.

When running `./deploy.sh` in the next step, add this argument to enable TLS on the UIServer:

```text
-f curiefense/uiserver-enable-tls.yaml
```

## Deploy Istio and Curiefense Images

Deploy the Istio service mesh:

```text
cd ~/curiefense/deploy/istio-helm 
DOCKER_TAG=main ./deploy.sh
```

And then the Curiefense components:

```text
cd ~/curiefense/deploy/curiefense-helm
DOCKER_TAG=main ./deploy.sh
```

## Deploy the \(Sample\) App

The application to be protected by Curiefense should now be deployed. These instructions are for the sample application `bookinfo`.

### Create the namespace

Create the Kubernetes namespace, and add the `istio-injection=enabled` label that will make Istio automatically inject necessary sidecars to applications that are deployed in this namespace.

```text
kubectl create namespace bookinfo
kubectl label namespace bookinfo istio-injection=enabled
```

### Install the application

```text
git clone https://github.com/istio/istio/ -b 1.5.10
kubectl apply -n bookinfo -f istio/samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -n bookinfo -f istio/samples/bookinfo/networking/bookinfo-gateway.yaml
```

### Test bookinfo <a id="markdown-header-test-bookinfo"></a>

Check that `bookinfo` Pods are running \(wait a bit if they are not\):

```text
kubectl get -n bookinfo pod -l app=ratings
```

Sample output example:

```text
NAME                         READY   STATUS    RESTARTS   AGE
ratings-v1-f745cf57b-cjg69   2/2     Running   0          79s
```

Check that the application is working by querying its API directly without going through the Istio service mesh:

```text
kubectl exec -n bookinfo -it "$(kubectl get -n bookinfo pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep "<title>"
```

Expected output:

```text
<title>Simple Bookstore App</title>
```

### Test access to bookinfo through Istio <a id="markdown-header-test-access-to-bookinfo-through-istio"></a>

```text
curl -sS http://$(kubectl -n istio-system get svc istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')/productpage|grep "<title>"
```

\(Replace "ip" with "hostname" if running in an environment where the LoadBalancer yields a FQDN, as is the case with Amazon's ELB.\)

Expected output:

```text
<title>Simple Bookstore App</title>
```

If this error occurs: `Could not resolve host: a6fdxxxxxxxxxxxxxxxxxxxxxxxxxxxx-xxxxxxxx.us-west-2.elb.amazonaws.com` ...the ELB is not ready yet. Wait and retry until it becomes available \(typically a few minutes\).

### Check that logs reach the accesslog UI <a id="markdown-header-check-that-logs-reach-the-accesslog-ui"></a>

Run this query

```text
curl http://$(kubectl -n istio-system get svc istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')/TEST_STRING
```

\(Replace "ip" with "hostname" if running in an environment where the LoadBalancer yields a FQDN, as is the case with Amazon's ELB.\)

Run this to ensure that the logs have been recorded and are reachable from the UI server:

```text
kubectl exec -ti -n curiefense elasticsearch-0 -- curl http://127.0.0.1:9200/_search -H "Content-Type: application/json" -d '{"query": {"match": {"path": "/TEST_STRING"}}}'
```

Check that a result is return, and that it does contains `TEST_STRING`.

## Expose Curiefense Services using NodePorts <a id="markdown-header-expose-curiefense-services-using-nodeports"></a>

Run the following commands to expose Curiefense services through NodePorts. Warning: if the machine has a public IP, the services will be exposed on the Internet.

Start with this command:

```text
kubectl apply -f ~/curiefense/deploy/curiefense-helm/expose-services.yaml
```

The following command can be used to determine the IP address of your cluster nodes on which services will be exposed:

```text
kubectl get nodes -o wide
```

### For minikube only: <a id="markdown-header-minikube"></a>

If you are using minikube, also run the following commands on the host in order to expose services on the Internet:

```text
sudo iptables -t nat -A PREROUTING -p tcp --match multiport --dports 30000,30080,30081,30200,30300,30443,30601 -j DNAT --to 172.17.0.2
sudo iptables -I FORWARD -p tcp --match multiport --dports 30000,30080,30081,30200,30300,30443,30444,30601 -j  ACCEPT
```

### For Amazon EKS only: <a id="markdown-header-amazon-eks"></a>

If you are using Amazon EKS, you will also need to allow inbound connections for port range 30000-30700 from your IP. Go to the EC2 page in the AWS console, select the EC2 instance for the cluster \(named `curiefense-eks-...-Node`\), select the "Security" pane, select the security group \(named `eks-cluster-sg-curiefense-eks-[0-9]+`\), then add the incoming rule.

## Access Curiefense Services <a id="markdown-header-access-curiefense-services"></a>

The UIServer is now available on port 30080 over HTTP, and on port 30443 over HTTPS.

Grafana is now available on port 30300 over HTTP.

For the `bookinfo` sample app, the Book Review product page is now available on port 30081 over HTTP, and on port 30444 over HTTPS \(if you chose to enable TLS\).

The confserver is now available on port 30000 over HTTP.

Kibana is now available on port 30601 over HTTP.

Elasticsearch is now available on port 30200 over HTTP.

For a full list of ports used by Curiefense containers, see the [Reference page on services and containers](../../reference/services-container-images.md).

## Reference: Description of Helm Charts <a id="markdown-header-description-of-the-pods"></a>

### Curiefense charts <a id="markdown-header-curiefense-components"></a>

Helm charts are divided as follows:

* `curiefense-admin` - confserver, curielogserver and UIServer.
* `curiefense-dashboards` - Grafana and Prometheus.
* `curiefense-log` - log storage: elasticsearch \(default\), postgres; log forwarders for elasticsearch: logstash \(default\), fluentd; log display interface: kibana \(default\)
* `curiefense-proxy` - curielogger, curiesync and redis \(used for synchronizationj.

#### Chart configuration variables <a id="markdown-header-configuration-variables"></a>

Configuration variables in `deploy/curiefense-helm/curiefense/values.yaml` can be modified or overridden to fit your deployment needs:

* Variables in the `images` section define the Docker image names for each component. Override this if you want to host images on your own private registry.
* `storage.storage_class_name` is the StorageClass that is used for dynamic provisioning of Persistent Volumes. It defaults to `null` \(default storage class, which works by default on EKS, GKE and minikube\).
* `storage.*_storage_size` variables define the size of persistent volumes. The defaults are fine for a test or small-scale deployment.
* `settings.curieconf_manifest_url` is the URL of the AWS S3 bucket that is used to synchronize configurations between the `confserver` and the Curiefense Istio sidecars.
* `settings.curiefense_logdb_type` defines whether logs are stored in postgresql or elasticsearch.
* `settings.curiefense_es_forwarder` defines whether logs are forwarded to elasticsearch using fluentd or logstash \(default\). Has no effect if `settings.curiefense_logdb_type` is set to `elasticsearch`.
* `settings.curiefense_es_hosts` is the hostname for the elasticsearch cluster. Changing it is required only if the elasticsearch cluster supplied by this chart is not used, and replaced with an externally-managed cluster.
* `settings.curiefense_logstash_url` is the url of the logstash server. Changing it is required only if the logstash instance supplied by this chart is not used, and replaced with an externally-managed instance.
* `settings.curiefense_fluentd_url` is the url of the fluentd server. Changing it is required only if the fluentd instance supplied by this chart is not used, and replaced with an externally-managed instance.
* `settings.curiefense_kibana_url` is the url of the kibana server. Changing it is required only if the kibana instance supplied by this chart is not used, and replaced with an externally-managed instance.
* `settings.curiefense_db_hostname` is the hostname of the postgres server that will be used to store logs. Defaults to the provided `logdb` StatefulSet. Override this to replace the postgres instance with one you supply, or an AWS Aurora instance.
* `settings.curiefense_bucket_type` is the type of cloud bucket that is used to transfer configurations from `confserver` to envoy proxies \(supported values: `s3` or `gs`\).
* `settings.curiefense_es_index_name` is the name of the elasticsearch index where logs are stored.
* `settings.docker_tag` defines the image tag versions that should be used. `deploy.sh` will override this to deploy a version that matches the current working directory, unless the `DOCKER_TAG` environment variable is set.
* `settings.redis_port` is the port on which redis listens. This value must be set identically in the Istio chart's `values.yaml`.
* `settings.uiserver_enable_tls` is a boolean that defines whether TLS is enabled on the UI server. If it is enabled, then a certificate and key must have been provisioned \(see above\).
* Variables in the `requests` define default CPU requirements for pods.
* Variables in the `enable` allow disabling parts of a deployment, which can be supplied outside of this chart \(ex. postgresql, kibana, logstash, fluentd, elasticsearch, prometheus...\).

### Istio chart <a id="markdown-header-istio"></a>

Components added or modified by Curiefense are defined in `deploy/istio-helm/chart/charts/gateways`. Compared to the upstream Istio Kubernetes distribution, we add or change the following Pods:

* An `initContainer` called `curiesync-initialpull` has been added. It synchronizes configuration before running Envoy.
* A container called `curiesync` has been added. It periodically fetches the configuration that should be applied from an S3 bucket \(configurable with the `curieconf_manifest_url` variable\), and makes it available to Envoy. This configuration is used by the LUA code that inspects traffic.
* The container called `istio-proxy` now uses our custom Docker image, embedding our HTTP Filter, written in Lua.
* An `EnvoyFilter` has been added. It forwards access logs to `curielogger` \(see `curiefense_access_logs_filter.yaml`\).
* An `EnvoyFilter` has been added. It runs Curiefense's Lua code to inspect incoming traffic on the Ingress Gateways \(see `curiefense_lua_filter.yaml`\).

#### Chart configuration variables <a id="markdown-header-configuration-variables_1"></a>

Configuration variables in `deploy/istio-helm/chart/values.yaml` can be modified or overridden to fit your deployment needs:

* `gw_image` defines the name of the image that contains our filtering code and modified Envoy binary.
* `curiesync_image` defines the name of the image that contains scripts that synchronize local Envoy configuration with the AWS S3 bucket defined in `curieconf_manifest_url`.
* `curieconf_manifest_url` is the URL of the AWS S3 or Google Storage bucket that is used to synchronize configurations between the `confserver` and the Curiefense Istio sidecars.
* `curiefense_namespace` is the name of the namespace where Curiefense components defined in `deploy/curiefense-helm/` are running.
* `curiefense_bucket_type` is the type of cloud bucket that is used to transfer configurations from `confserver` to envoy proxies \(supported values: `s3` or `gs`\).
* `redis_host` defines the hostname of the redis server that will be used by `curieproxy`. Defaults to the provided redis StatefulSet. Override this to replace the redis instance with one you supply.
* `redis_port` defines the port of the redis server that will be used by `curieproxy`. Defaults to the provided redis StatefulSet. Override this to replace the redis instance with one you supply.
* `initial_curieconf_pull` defines whether a configuration should be pulled from the AWS S3 bucket before running Envoy \(`true`\), or if traffic should be allowed to flow with a default configuration until the next synchronization \(typically every 10s\).

