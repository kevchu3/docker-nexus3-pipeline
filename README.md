# OpenShift CI/CD Pipeline for Nexus Repository

## Prerequisites

To run this reference pipeline, you will need the following:

- OpenShift 4 cluster
- [OpenShift Pipelines] already installed
- [OpenShift GitOps] already installed

## Build and Deploy via OpenShift Pipeline

This reference pipeline was built using sample code from the [pipelines-tutorial].

In a new namespace, install the `docker-nexus3-pipeline` pipeline, which also contains `apply-manifests` and `update-deployment` task resources from the `pipelines-tutorial` repository, which contains a list of reusable tasks for pipelines.
```
$ oc new-project nexus
$ oc apply -f docker-nexus3.pipeline.yaml
```

## Continuous Deployment via OpenShift GitOps

Install the ArgoCD application to the `openshift-gitops` namespace:
```
$ oc apply -f docker-nexus3.application.yaml
```

## Configuring Docker Registry in Nexus

Once the Nexus application has been deployed, you can log into the Nexus web console via the route and follow the steps for initial setup.
```
$ oc get route nexus-web
```

To set up a Docker private (hosted) repository, you'll need to configure a [Docker repository connector] which will allow a repository connector to be used on a separate port.

> What is a Repository Connector?
> When you make a request using the Docker client, you provide a hostname and port followed by the Docker image. Docker does not support the use of a context to specify the path to the repository. 
>
> Does not work:
> `docker pull centos7:8081/repository/docker-group/postgres:9.4`
>
> Does work:
> `docker pull centos7:18080/postgres:9.4`
>
> Since we cannot include the repository name in the Docker client request, we use a Repository Connector to assign a port to the Docker repository which can be used in Docker client commands. The Repository Connector is found in the settings for each docker repository.

The following steps were inspired by my previous work in setting up [Nexus alongside Apache Httpd in a VM], and have been customized for deployment within OpenShift.

From the Nexus web console, navigate to **Repositories** -> **Create repository** -> **Docker (hosted)**.

Fill in the name, and select **HTTP** (Create an HTTP connector at specified port), using port 5000.  This will correspond to the `nexus-registry` service that was deployed.  Press Create repository.

Configure Nexus security to enable Docker logins.  From the Nexus UI, navigate to **Security** -> **Realms** -> set **Docker Bearer Token Realm** to **Active**, and **Save**.  Afterwards, you may need to restart the Nexus pod:
```
$ oc delete pod -l app=docker-nexus3
```

## Using Docker Registry

The Docker registry on the repository connector port corresponding to the OpenShift service is exposed via its associated route:
```
$ oc get route nexus-registry
```

You can use podman to log in and push images using this route (replace `<nexus-registry.example.com>` below with your route).  We'll pull an image from Red Hat container catalog and push it to Nexus:
```
$ podman pull registry.access.redhat.com/ubi9/ubi:latest
$ podman login <nexus-registry.example.com> --tls-verify=false
$ podman tag registry.access.redhat.com/ubi9/ubi:latest <nexus-registry.example.com>/ubi9/ubi:latest
$ podman push <nexus-registry.example.com>/ubi9/ubi:latest --tls-verify=false --remove-signatures
```

## Update Submodule

Clone Git repo and create dev branch:
```
git clone https://github.com/kevchu3/docker-nexus3-pipeline.git
cd docker-nexus3-pipeline
git branch dev
git checkout dev
```

Update submodule to tagged version 3.77.2:
```
cd docker-nexus3
git submodule init
git submodule update
git branch
git checkout tags/3.77.2
```

Commit to dev branch:
```
cd ..
git status
git add docker-nexus3
git commit -S -m "Git submodule update to docker-nexus3 v3.77.2-02"
git push origin dev
```

## License
GPLv3

[OpenShift Pipelines]: https://docs.redhat.com/en/documentation/red_hat_openshift_pipelines/1.17/html/installing_and_configuring/index
[OpenShift GitOps]: https://docs.redhat.com/en/documentation/red_hat_openshift_gitops/1.15/html/installing_gitops/index
[pipelines-tutorial]: https://github.com/openshift/pipelines-tutorial
[Docker repository connector]: https://support.sonatype.com/hc/en-us/articles/115013153887-Docker-Repository-Configuration-and-Client-Connection
[Nexus alongside Apache Httpd in a VM]: https://github.com/kevchu3/nexus-docker-repo
