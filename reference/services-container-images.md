# Services and Container Images

Containers are built from recipes contained in`curiefense/images/`. Descriptions of the various images are below.

## Image Tagging

Several tag formats are used.

| Single Tag | Meaning |
| :--- | :--- |
| `test` | For testing single images |
| `stable` | For the latest stable version of containers |
| `latest-dev` | For the latest built image |

Images can also have two-part tags to identify what is in the image. The parts are:

* the output of `git describe --tag --long --dirty` which contains the latest git tag, number of commits since last tag, and abbreviated current commit hash
* the shortened hash of the git tree `HEAD:curiefense/` where Docker images are stored. It helps to see quickly whether image sources from two different commits are identical.

## Common Images

### confserver

* A REST API server to read, write, and maintain versioning of Curiefense's configuration.
* Flask is the web interface. Git is the storage engine for historical and versioned configuration. Nginx serves as the frontend for the Flask web application.
* Secrets:
  * If an S3 bucket is used as a synchronization mechanism between `confserver` and `curieproxy` instances \(this is the default with Helm deployments\), then this container requires credentials to access it.
  * In Docker Compose environments, S3 credentials are defined in `deploy/compose/curiesecrets/s3cfg` before deployment, then mounted in `/var/run/secrets/s3cfg`. 
  * In Helm environments, S3 credentials are [defined in a local file and deployed to the cluster](../installation/deployment-first-steps/istio-via-helm.md#setup-secrets). They are expected to be in a Secret object called `s3cfg`, in the same namespace as `confserver`, which is mounted to make credentials available in `/var/run/secrets/s3cfg/s3cfg`.
* Network details:
  * Port 80 is the configuration API. A Swagger interface is available at endpoint `/api/v1/` \(reachable at [http://localhost:30000/api/v1](http://localhost:30000/api/v1) in the sample Docker Compose and Helm deployments\).

### curielogger

* Receives access logs from Envoy \(`curieproxy`images\) over gRPC, and does the following:
  * Pushes logs into the PostgreSQL server \(`logdb`\)
  * Aggregates metrics
  * Serves metrics over HTTP port 2112 for the Prometheus scraper.
* Secrets: 
  * This container uses a postgres account \(`postgres`\) with read-write permissions. Its password is passed either in the `CURIELOGGER_DBPASSWORD` environment variable, or in a file whose path is contained in the `CURIELOGGER_DBPASSWORD_FILE` environment variable.
* Network details:
  * Port 9001 receives logs from Envoy over GRPC
  * Port 2112 exposes Prometheus metrics over HTTP

### curielogserver

* A REST API server used by `uiserver` to read log records from `logdb`.
* Flask is the web interface for the REST API. Nginx serves as the frontend for the Flask web application.
* Secrets:
  * This container uses a postgres account \(`logserver_ro`\) with read-only permissions. Its password is passed in a file whose path is contained in the `CURIELOGSERVER_DBPASSWORD_FILE` environment variable
* Network details:
  * Port 80 exposes an API to retrieve logs

### curiesync

* Periodically \(every 10 seconds\) polls the bucket located at `CURIE_BUCKET_LINK`, and extracts the active configuration to `/config`, which is shared with `curiefense`.
* Secrets: the same as described in `confserver`, above.
* Network details:
  * No exposed network service

### grafana

* Grafana provides visualization for the metrics stored in Prometheus, and sends alerts based on anomaly thresholds.
* Its datasource is prometheus. Its basic dashboards are in the `dashboards` directory. 
* Customization options are described at [https://hub.docker.com/r/grafana/grafana/](https://hub.docker.com/r/grafana/grafana/).
* Secrets
  * Default user: admin
  * Default password: admin
  * These can be changed upon the first connection. 
* Network details:
  * Port 3000 allows access to the Grafana UI over http \(reachable at [http://localhost:30300](http://localhost:30300) in the sample Docker Compose and Helm deployments\).

### logdb

* PostgreSQL server that stores access logs.
* Two accounts are used:
  * `postgres` has write access, is used by `curielogger`.
  * `logserver_ro` has read-only access, is used by `curielogserver`.
* Secrets:
  * The password for the `logserver_ro` account is passed in a file whose path is contained in the `POSTGRES_READONLY_PASSWORD_FILE` environment variable.
  * The password for the `postgres` account is passed in a file whose path is contained in the `POSTGRES_PASSWORD_FILE` environment variable.
* Network details:
  * Port 5432 for PostgreSQL access

### prometheus

* Prometheus scrapes metrics from Envoy, `curielogger`, and Prometheus itself. These are defined in `curiefense/images/prometheus/prometheus.yml`.
* `curielogger` metrics are exposed as follows:
  * `curiemetric_http_request_total`: total number of requests.
  * `curiemetric_request_bytes`: total inbound \(request\) bytes.
  * `curiemetric_response_bytes`: total outbound \(response\) bytes.
  * `curiemetric_session_details_total`: Static labels are a fixed set of labels created for each request, such as "method", "path", "geo", etc.
  * `curiemetric_session_tags_total`: this metric stores the "dynamic" labels: tags and labels created dynamically on a request basis, and per context. For example, while the "blocked" label is set to 0 or 1 for each request, an actual blocked request may carry additional tags such as the block reasons and origin.
* Network details:
  * Port 9090 allows access to the Prometheus user interface over HTTP

### redis

* Redis is accessed by `curieproxy`, and is used to synchronize Curiefense's advanced rate limiting and session control mechanisms. 
* Network details:
  * Port 6379 receives Redis client queries

### UIServer

* Serves the user interface. A Vue js app developed as single page app with NodeJS and serves the management console UI. 
* The UI displays access logs to the user, and displays Curiefense's configuration for editing. API calls for configuration and access logs are routed to `confserver` and `curielogserver` by the Nginx inside the container. Nginx also used to serve the static parts of the UI such as HTML, CSS and JS.
* Secrets: This image will enable TLS on the nginx server if a TLS certificate and key are provided:
  * For Kubernetes \(e.g. Helm\) deployments, the certificate is expected at `/run/secrets/uisslcrt/uisslcrt` and the key at `/run/secrets/uisslkey/uisslkey`
  * For Docker Compose deployments, the certificate is expected at `/run/secrets/uisslcrt` and the key at `/run/secrets/uisslkey`
* Network details:
  * Port 80 allows unencrypted access to the user interface \(reachable at [http://localhost:30080](http://localhost:30080) in the sample Docker Compose and Helm deployments\)
  * Port 443 allows TLS-encrypted access to the user interface, if TLS certificates have been supplied during the deployment \(reachable at [http://localhost:30443](http://localhost:30443) in the sample Docker Compose and Helm deployments\)

## Additional Images for Docker-Compose Deployments

### curieproxy-envoy

* Acts as a reverse proxy to `TARGET_ADDRESS:TARGET_PORT`.
* Filters traffic according to the active configuration.
* Sends access logs over GRPC to`curielogserver`.
* Uses a custom-built Envoy binary, compiled with symbols needed by Lua. The custom Envoy compilation is built automatically [using GitHub actions](https://github.com/curiefense/envoy/blob/main/.github/workflows/build-envoy-for-curiefense.yml), from the curiefense [fork](https://github.com/curiefense/envoy) of envoy, which pushes it to an [image on docker hub](https://hub.docker.com/r/curiefense/envoy-cf/tags?page=1&ordering=last_updated). This image is referenced by the first line of the [Dockerfile](https://github.com/curiefense/curiefense/blob/main/curiefense/images/curieproxy-envoy/Dockerfile#L1) that is used to build the curieproxy-envoy docker image.
* Network details:
  * Port 80 receives unencrypted traffic from users, which will be proxied to `TARGET_ADDRESS:TARGET_PORT` \(reachable at [http://localhost:30081](http://localhost:30081) in the sample  deployments\)
  * Port 443 receives TLS-encrypted traffic from users,  which will be proxied to `TARGET_ADDRESS:TARGET_PORT` \(reachable at [http://localhost:30444](http://localhost:30444) in the sample docker-compose deployment\)
  * Port 8001 is the Envoy administration interface

### echo

* Simple http server that displays `Echo`.
* Network details:
  * Listens on port 5678

## Additional Images for Istio-Helm Deployments

### curieproxy-istio

* Acts as a reverse proxy to `TARGET_ADDRESS:TARGET_PORT`.
* Filters traffic according to the active configuration.
* Sends access logs over GRPC to`curielogserver`.
* * Uses a custom-built Envoy binary, compiled with symbols needed by Lua. The custom Envoy compilation is built automatically [using GitHub actions](https://github.com/curiefense/istio-proxy/blob/cf-1.9.3/.github/workflows/build-envoy-for-curiefense.yml) from the curiefense [fork](https://github.com/curiefense/istio-proxy) of istio
    * , which pushes it to an [image on docker hub](https://hub.docker.com/r/curiefense/envoy-istio-cf/tags?page=1&ordering=last_updated). This image is referenced by the first line of the [Dockerfile](https://github.com/curiefense/curiefense/blob/main/curiefense/images/curieproxy-istio/Dockerfile#L1) that is used to build the curieproxy-envoy docker image.
* In Helm deployments, two EnvoyFilters are defined in `curiefense/deploy/istio-helm/chart/charts/gateways/templates/`:
  * `curiefense_lua_filter.yaml` orders Envoy to apply the Lua HTTP filter to incoming requests.
  * `curiefense_access_logs_filter.yaml` orders Envoy to send access logs to `curielogger`.
* Network details:
  * Port 80 receives unencrypted traffic from users, which will be proxied to TARGET\_ADDRESS:TARGET\_PORT \(reachable at [http://localhost:30081](http://localhost:30081) in the sample  deployments\)
  * Port 8001 is the Envoy administration interface

## **Building/Rebuilding Images**

If for some reason you need to rebuild the images, run the following command:

```bash
$ curiefense/images/build-docker-images.sh
```

