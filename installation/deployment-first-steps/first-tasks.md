# First Tasks

This page describes the initial tasks that must be performed when deploying Curiefense:

* [Select and prepare for TLS](./#select-and-prepare-for-tls).

After completing the tasks below, proceed to the tasks for the specific method being used:

* [Deployment into Istio via Helm Cha](istio-via-helm.md)[rts](istio-via-helm.md)
* [Deployment via Docker Compose](docker-compose.md)

During this process, you might find it helpful to read the descriptions \(which include the purpose, secrets, and network/port details\) of the services and their containers:

* [Services and Container Images](../../reference/services-container-images.md)

## Clone the Repository

Clone the repository, if you have not already done so:

```text
git clone https://github.com/curiefense/curiefense-helm.git
```

This documentation assumes it has been cloned to `~/curiefense-helm`.

## Select and Prepare for TLS

Curiefense can use TLS, but this is optional. \(If you do not choose to set it up, HTTPS will be disabled.\)

At this point in the setup process, you should decide whether or not to do so:

* An Istio Helm deployment can use TLS for communication with Curiefense's UI server.
* A Docker Compose deployment can use TLS for communication with Curiefense's UI server and also for the protected service.

If you do not want Curiefense to use TLS, then skip this step and proceed to the next section.

If you want Curiefense to use TLS, generate the certificate\(s\) and key\(s\) now. You will add them to Curiefense later.

## Method-Specific Tasks

After performing the previous tasks, perform the tasks for the specific method being used:

* [Deployment into Istio via Helm Charts](istio-via-helm.md)
* [Deployment via Docker Compose](docker-compose.md)

