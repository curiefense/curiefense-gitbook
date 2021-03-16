# Docker Compose

This page describes the tasks necessary to deploy Curiefense using Docker Compose. It assumes that the instructions described in [First Tasks](first-tasks.md) have been completed successfully.

This process consists of the following tasks, described sequentially below:

* [TLS Setup](docker-compose.md#tls-setup)
* [Set Database Credentials](docker-compose.md#set-database-credentials)
* [Set Deployment Variables](docker-compose.md#set-deployment-variables)
* [Deploy Curiefense](docker-compose.md#deploy-curiefense)
* [Test the Deployment](docker-compose.md#test-the-deployment)
* [Clean Up](docker-compose.md#clean-up)

If during this process you need to rebuild an image, see the instructions here: [Building/Rebuilding an Image](../../reference/services-container-images.md#building-rebuilding-images).

## TLS Setup

If you want Curiefense to use TLS, then you should have already generated the certificates and keys.

To enable TLS for the protected site/application, go to `curiefense/deploy/compose/curiesecrets/curieproxy_ssl/` and do the following:

* Edit `site.crt` and add the certificate.
* Edit `site.key` and add the key.

To enable TLS for the nginx server that is used by `uiserver`, go to `curiefense/deploy/compose/curiesecrets/uiserver_ssl/`and do the following:

* Edit `ui.crt` and add the certificate.
* Edit `ui.key` and add the key.

## Set Deployment Variables

Docker Compose deployments can be configured in two ways:

* By setting values for variables in `deploy/compose/.env` 
* Or by setting OS environment variables \(which will override any variables set in`.env`\)

These variables are described below.

### CURIE\_BUCKET\_LINK

Curiefense uses the storage defined here for synchronizing configuration changes between `confserver` and the Curiefense sidecars. 

By default, this points to the `local_bucket` Docker volume:

```text
$ grep CURIE_BUCKET_LINK .env
CURIE_BUCKET_LINK=file:///bucket/prod/manifest.json
```

For multi-node deployments, or to use S3 for a single node, replace this value with the URL of an S3 bucket:

```text
CURIE_BUCKET_LINK=s3:///BUCKETNAME/prod/manifest.json
```

In that case, you will need to supply AWS credentials in  `deploy/compose/curiesecrets/s3cfg`, following this template:

```text
[default]
access_key = AAAAAAAAAAAAAAAAAAAA
secret_key = AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

### TARGET\_ADDRESS and TARGET\_PORT

The address of the destination service for which Curiefense acts as a reverse proxy. By default, this points to the `echo` container, which simply echoes the HTTP requests it receives.

### DOCKER\_TAG

Defaults to `latest` \(the latest stable image\). To run a version that matches the contents of your working directory, use the following command:

```text
DOCKER_TAG="$(git describe --tag --long --dirty)-$(git rev-parse --short=12 HEAD:curiefense)"
```

## Deploy Curiefense

Once the tasks above are completed, run these commands:

```text
cd curiefense/deploy/compose/
docker-compose up
```

## Test the Deployment

After deployment, the Echo service should be running and protected behind Curiefense. You can test the success of the deployment by querying it:

```text
$ curl http://localhost:30081/
Echo
```

Also verify the following:

* The UIServer is now available at `http://localhost:30080`
* Grafana is now available at `http://localhost:30300/dashboards`
* The `confserver` is now available at `http://localhost:30000/api/v1/`

## Clean Up

To stop all containers and remove any persistent data stored in volumes, run the following commands:

```text
docker-compose rm -f && docker volume prune -f
```

