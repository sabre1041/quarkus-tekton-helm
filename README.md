# quarkus-tekton-helm

Demonstrates the usage of the [Quarkus Helm Chart](https://github.com/redhat-developer/redhat-helm-charts) and orchestrated using [OpenShift Pipelines](https://docs.openshift.com/container-platform/4.6/pipelines/understanding-openshift-pipelines.html) / [Tekton](https://tekton.dev/)

## Prerequisites

The following are required in order to execute the contents of this repository

1. Command Line tools
    1. [OpenShift](https://docs.openshift.com/container-platform/4.6/cli_reference/openshift_cli/getting-started-cli.html)
    2. [Tekton (tkn)](https://docs.openshift.com/container-platform/4.6/cli_reference/tkn_cli/installing-tkn.html)
    3. [Helm](https://helm.sh/) (optional)
1. OpenShift Pipelines or Tekton with the _git-clone_ and _buildah_ ClusterTask installed
2. An image repository and rights to push to the associated image registry

## Configuration

### Install resources

First, clone the repository and change into the cloned directory:

```
$ git clone https://github.com/sabre1041/quarkus-tekton-helm.git
$ cd quarkus-tekton-helm

```

Execute the following command in order to create a new namespace and populate it with resources

```
$ oc apply -k resources
```

Once the resources have been applied, change into the newly created project

```
$ oc project quarkus-tekton-helm
```

### Image Registry Authentication

The pipeline creates and pushes a container image to a remote registry. In most cases, the pipeline will require credentials in order to successfully push the image to the remote registry. Follow the [Tekton Authentication Documentation](https://github.com/tektoncd/pipeline/blob/master/docs/auth.md#configuring-authentication-for-docker) to configure the credentials into a Secret and associate it with the _quarkus-tekton-helm_ Service Account used to execute the pipeline.

## Run Pipeline

Run the pipeline by executing the following command (Be sure to replace the _image_repository_ parameter with the location of your image repository that will be used to push the image)

```
$ tkn pipeline start quarkus-helm-pipeline -s quarkus-tekton-helm -p image_repository=quay.io/user/quarkus-getting-started --workspace name=shared-workspace,volumeClaimTemplateFile=resources/volumeclaimtemplates/pvc.yaml
```

The above command specifies the following:

1. The name of the Pipeline
2. The Service Account Used to execute the pipeline using the `-s` parameter
3. The image repository that will be used to store the resulting image
4. A Volume Claim Template that will dynamically allocate a PersistentVolume for use by the pipeline

## Verify Pipeline

Once the pipeline has been started, it will print out a command that you can use to view the logs from the pipeline run. 

After the pipeline completes successfully, using the Helm CLI, confirm the release of the Quarkus Helm Chart.

```
$ helm ls

NAME                    NAMESPACE               REVISION        UPDATED                                 STATUS          CHART           APP VERSION
quarkus-getting-started quarkus-tekton-helm     1               2021-01-17 17:23:26.570627435 +0000 UTC deployed        quarkus-0.0.3         
```

View the running pod associated with the application

```
$ oc get pods -l=app.kubernetes.io/name=quarkus-getting-started

NAME                                       READY   STATUS    RESTARTS   AGE
quarkus-getting-started-785584688d-9kpgz   1/1     Running   0          89s
```

Finally, attempt to view the deployed application in a web browser:

Locate the url of the Route

```
$ oc get routes quarkus-getting-started -o jsonpath='https://{ .spec.host}'
```


