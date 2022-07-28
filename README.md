# OpenShift Pipeline for Nexus Repository

## Prerequisites

To run this reference pipeline, you will need the following:

- OpenShift 4 cluster
- [OpenShift Pipelines] already installed

## Deployment

This reference pipeline was built using sample code from the [pipelines-tutorial].

Install the `docker-nexus3-pipeline` pipeline, which also contains `apply-manifests` and `update-deployment` task resources from the `pipelines-tutorial` repository, which contains a list of reusable tasks for pipelines.

```
$ oc create -f docker-nexus3.pipeline.yaml
```

## License
GPLv3

[OpenShift Pipelines]: https://github.com/openshift/pipelines-tutorial/blob/master/install-operator.md
[pipelines-tutorial]: https://github.com/openshift/pipelines-tutorial
