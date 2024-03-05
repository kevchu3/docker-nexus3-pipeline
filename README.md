# OpenShift Pipeline for Nexus Repository

## Prerequisites

To run this reference pipeline, you will need the following:

- OpenShift 4 cluster
- [OpenShift Pipelines] already installed

## Build and Deploy via OpenShift Pipeline

This reference pipeline was built using sample code from the [pipelines-tutorial].

Install the `docker-nexus3-pipeline` pipeline, which also contains `apply-manifests` and `update-deployment` task resources from the `pipelines-tutorial` repository, which contains a list of reusable tasks for pipelines.
```
$ oc apply -f docker-nexus3.pipeline.yaml
```

## Continuous Deployment via OpenShift GitOps

Install the ArgoCD application:
```
$ oc apply -f docker-nexus3.application.yaml
```

## Update Submodule

Clone Git repo and create dev branch:
```
git clone https://github.com/kevchu3/docker-nexus3-pipeline.git
cd docker-nexus3-pipeline
git branch dev
git checkout dev
```

Update submodule to tagged version 3.66.0:
```
cd docker-nexus3
git submodule init
git submodule update
git branch
git checkout tags/3.66.0
```

Commit to dev branch:
```
cd ..
git status
git add docker-nexus3
git commit -S -m "Git submodule update to docker-nexus3 v3.66.0-02"
git push origin dev
```

## License
GPLv3

[OpenShift Pipelines]: https://github.com/openshift/pipelines-tutorial/blob/master/install-operator.md
[pipelines-tutorial]: https://github.com/openshift/pipelines-tutorial

