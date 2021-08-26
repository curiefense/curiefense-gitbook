# Services and Container Images

Containers are built from recipes contained in`curiefense/images/`. Descriptions of the various images are below.

## Image Tagging <a id="image-tagging"></a>

| Single Tag | Meaning |
| :--- | :--- |
| `main` | For the latest built image, from the main branch of the github repository |

Images can also have two-part tags to identify what is in the image. The parts are:

* the output of `git describe --tag --long --dirty` which contains the latest git tag, number of commits since last tag, and abbreviated current commit hash
* the shortened hash of the git tree `HEAD:curiefense/` where Docker images are stored. It helps to see quickly whether image sources from two different commits are identical.

## Common Images <a id="common-images"></a>

### confserver <a id="confserver"></a>

* A REST API server to read, write, and maintain versioning of Curiefense's configuration.
* Flask is the web interface. Git is the storage engine for historical and versioned configuration. Nginx serves as the frontend for the Flask web application.
* Secrets:
  * If an S3 bucket of Google Storage bucket is used as a synchronization mechanism between `confserver` and `curieproxy` instances \(S3 is the default with Helm deployments\), then this container requires credentials to access it.
  * In Docker Compose environments, S3 credentials are defined in `deploy/compose/curiesecrets/s3cfg` before deployment, then mounted in `/var/run/secrets/s3cfg`. By default, Docker Compose environments use a "local bucket" \(shared volume\) so that an S3 bucket is not necessary to test curiefense locally.
  * In Helm environments, S3 credentials are [defined in a local file and deployed to the cluster](https://docs.curiefense.io/v/main/installation/deployment-first-steps/istio-via-helm#setup-secrets). They are expected to be in a Secret object called `s3cfg`, in the same namespace as `confserver`, which is mounted to make credentials available in `/var/run/secrets/s3cfg/s3cfg`. Google Storage bucket credentials are mounted to `/var/run/secrets/gs/`.
* Network details:
  * Port 80 is the configuration API. A Swagger interface is available at endpoint `/api/v1/` \(reachable at [http://localhost:30000/api/v1](http://localhost:30000/api/v1) in the sample Docker Compose and Helm deployments\).

### curielogger <a id="curielogger"></a>

* Receives access logs from Envoy \(`curieproxy`images\) over gRPC or from nginx over syslog, and does the following:
  * Pushes logs to elasticsearch through either fluentd or filebeat,
  * Aggregates metrics,
  * Serves metrics over HTTP port 2112 for the Prometheus scraper.
* Network details:
  * Port 9001 receives logs from Envoy over GRPC
  * Port 9514 receives logs from nginx over syslog
  * Port 2112 exposes Prometheus metrics over HTTP

### curiesync <a id="curiesync"></a>

* Periodically \(every 10 seconds\) polls the bucket located at `CURIE_BUCKET_LINK`, and extracts the active configuration to `/config`, which is shared with `curiefense`.
* In Helm deployments, `curiesync` runs first as an `initContainer` to fetch the correct configuration before istio is started. It then runs in a regular container, to sync periodically.
* Secrets: the same as described in `confserver`, above.
* Network details:
  * No exposed network service

### grafana <a id="grafana"></a>

* Grafana provides visualization for the metrics stored in Prometheus, and sends alerts based on anomaly thresholds.
* Its datasource is prometheus. Its basic dashboards are in the `dashboards` directory.
* Customization options are described at [https://hub.docker.com/r/grafana/grafana/](https://hub.docker.com/r/grafana/grafana/).
* Secrets
  * Default user: admin
  * Default password: admin
  * These can be changed upon the first connection.
* Network details:
  * Port 3000 allows access to the Grafana UI over http \(reachable at [http://localhost:30300](http://localhost:30300/) in the sample Docker Compose and Helm deployments\).

### prometheus <a id="prometheus"></a>

* Prometheus scrapes metrics from Envoy, `curielogger`, and Prometheus itself. These are defined in `curiefense/images/prometheus/prometheus.yml`.
* `curielogger` metrics are exposed as follows:
  * `curiemetric_http_request_total`: total number of requests.
  * `curiemetric_request_bytes`: total inbound \(request\) bytes.
  * `curiemetric_response_bytes`: total outbound \(response\) bytes.
  * `curiemetric_session_details_total`: Static labels are a fixed set of labels created for each request, such as "method", "path", "geo", etc.
  * `curiemetric_session_tags_total`: this metric stores the "dynamic" labels: tags and labels created dynamically on a request basis, and per context. For example, while the "blocked" label is set to 0 or 1 for each request, an actual blocked request may carry additional tags such as the block reasons and origin.
* Network details:
  * Port 9090 allows access to the Prometheus user interface over HTTP

### redis <a id="redis"></a>

* Redis is accessed by `curieproxy`, and is used to synchronize Curiefense's advanced rate limiting and session control mechanisms.
* Network details:
  * Port 6379 receives Redis client queries

### UIServer <a id="uiserver"></a>

* Serves the user interface. A Vue js app developed as single page app with NodeJS and serves the management console UI.
* The UI displays access logs to the user, and displays Curiefense's configuration for editing. API calls for configuration and access logs are routed to `confserver` by the Nginx inside the container. Nginx also serves the static parts of the UI such as HTML, CSS and JS.
* Secrets: This image will enable TLS on the nginx server if a TLS certificate and key are provided:
  * For Kubernetes \(e.g. Helm\) deployments, the certificate is expected at `/run/secrets/uisslcrt/uisslcrt` and the key at `/run/secrets/uisslkey/uisslkey`
  * For Docker Compose deployments, the certificate is expected at `/run/secrets/uisslcrt` and the key at `/run/secrets/uisslkey`
* Network details:
  * Port 80 allows unencrypted access to the user interface \(reachable at [http://localhost:30080](http://localhost:30080/) in the sample Docker Compose and Helm deployments\)
  * Port 443 allows TLS-encrypted access to the user interface, if TLS certificates have been supplied during the deployment \(reachable at [http://localhost:30443](http://localhost:30443/) in the sample Docker Compose and Helm deployments\)

## Additional Images for Docker-Compose Deployments <a id="additional-images-for-docker-compose-deployments"></a>

### curieproxy-envoy <a id="curieproxy-envoy"></a>

* Acts as a reverse proxy to `TARGET_ADDRESS:TARGET_PORT`.
* Filters traffic according to the active configuration.
* Sends access logs over GRPC to`curielogserver`.
* Uses a custom-built Envoy binary, compiled with symbols needed by Lua. The custom Envoy compilation is built automatically [using GitHub actions](https://github.com/curiefense/envoy/blob/main/.github/workflows/build-envoy-for-curiefense.yml), from the curiefense [fork](https://github.com/curiefense/envoy) of envoy, which pushes it to an [image on docker hub](https://hub.docker.com/r/curiefense/envoy-cf/tags?page=1&ordering=last_updated). This image is referenced in the beginning of the [Dockerfile](https://github.com/curiefense/curiefense/blob/main/curiefense/images/curieproxy-envoy/Dockerfile) that is used to build the curieproxy-envoy docker image.
* Network details:
  * Port 80 receives unencrypted traffic from users, which will be proxied to `TARGET_ADDRESS:TARGET_PORT` \(reachable at [http://localhost:30081](http://localhost:30081/) in the sample deployments\)
  * Port 443 receives TLS-encrypted traffic from users, which will be proxied to `TARGET_ADDRESS:TARGET_PORT` \(reachable at [http://localhost:30444](http://localhost:30444/) in the sample docker-compose deployment\)
  * Port 8001 is the Envoy administration interface

### echo <a id="echo"></a>

* Simple http server that echoes received HTTP requests.
* Uses the public `jmalloc/echo-server` image.
* Network details:
  * Listens on port 5678

## Additional Images for Istio-Helm Deployments <a id="additional-images-for-istio-helm-deployments"></a>

### curieproxy-istio <a id="curieproxy-istio"></a>

* Acts as a reverse proxy to `TARGET_ADDRESS:TARGET_PORT`.
* Filters traffic according to the active configuration.
* Sends access logs over GRPC to`curielogserver`.
* * Uses a custom-built Envoy binary, compiled with symbols needed by Lua. The custom Envoy compilation is built automatically [using GitHub actions](https://github.com/curiefense/istio-proxy/blob/cf-1.9.3/.github/workflows/build-envoy-for-curiefense.yml) from the curiefense [fork](https://github.com/curiefense/istio-proxy) of istio, which pushes it to an [image on docker hub](https://hub.docker.com/r/curiefense/envoy-istio-cf/tags?page=1&ordering=last_updated). This image is referenced in the beginning of the [Dockerfile](https://github.com/curiefense/curiefense/blob/main/curiefense/images/curieproxy-istio/Dockerfile) that is used to build the curieproxy-envoy docker image.
* In Helm deployments, two EnvoyFilters are defined:
  * `​`[`curiefense_lua_filter.yaml`](https://github.com/curiefense/curiefense-helm/blob/main/istio-helm/charts/gateways/istio-ingress/templates/curiefense_lua_filter.yaml) orders Envoy to apply the Lua HTTP filter to incoming requests.
  * `​`[`curiefense_access_logs_filter.yaml`](https://github.com/curiefense/curiefense-helm/blob/main/istio-helm/charts/gateways/istio-ingress/templates/curiefense_access_logs_filter.yaml) orders Envoy to send access logs to `curielogger`.
* Network details:
  * Port 80 receives unencrypted traffic from users, which will be proxied to TARGET\_ADDRESS:TARGET\_PORT \(reachable at [http://localhost:30081](http://localhost:30081/) in the sample deployments\)
  * Port 8001 is the Envoy administration interface

## **Building/Rebuilding Images** <a id="building-rebuilding-images"></a>

If for some reason you need to rebuild the images, run the following command:

```text
$ curiefense/images/build-docker-images.sh
```

To build images with a custom tag, the `DOCKER_TAG` environment variable may be set:

```text
$ DOCKER_TAG=mytag curiefense/images/build-docker-images.sh
```

