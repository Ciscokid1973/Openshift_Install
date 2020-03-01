## Install the Tekton CLI onto your Laptop

```sh
curl -LO https://github.com/tektoncd/cli/releases/download/v0.4.0/tkn_0.4.0_Darwin_x86_64.tar.gz
 
sudo tar xvzf tkn_0.4.0_Darwin_x86_64.tar.gz -C /usr/local/bin tkn
```

## Install Pipelines Operator
Save the following YAML as pipeline_operator.yaml locally.

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-pipelines-operator
  namespace: openshift-operators
spec:
  channel: dev-preview
  name: openshift-pipelines-operator
  source: community-operators
  sourceNamespace: openshift-marketplace
```
Then login to the cluster and run the following command

```sh
oc apply -f pipeline_operator.yaml
```

This will install the pipelines Operator (Tekton) onto the OpenShift Cluster.

To watch installation run 

```sh
until oc api-resources --api-group=tekton.dev | grep tekton.dev &> /dev/null
do 
 echo "Operator installation in progress..."
 sleep 5
done

echo "Operator ready"
```

A service account called pipeline will be created with the required permissions to run privileged pods for building images.  You can view this using

```sh
oc get serviceaccount pipeline
```

View New resources

```sh
oc api-resources --api-group=tekton.dev
NAME                SHORTNAMES   APIGROUP     NAMESPACED   KIND
clustertasks                     tekton.dev   false        ClusterTask
conditions                       tekton.dev   true         Condition
eventlisteners      el           tekton.dev   true         EventListener
pipelineresources                tekton.dev   true         PipelineResource
pipelineruns        pr,prs       tekton.dev   true         PipelineRun
pipelines                        tekton.dev   true         Pipeline
taskruns            tr,trs       tekton.dev   true         TaskRun
tasks                            tekton.dev   true         Task
triggerbindings     tb           tekton.dev   true         TriggerBinding
triggertemplates    tt           tekton.dev   true         TriggerTemplate
```

## Tutorial

For a tutorial on how to create a pipeline you can follow this OpenShift tutorial https://github.com/openshift/pipelines-tutorial

### Deploy sample application
```sh
oc new-project pipelines-tutorial

oc get serviceaccount pipeline

oc create -f update_deployment_task.yaml

oc create -f apply_manifest_task.yaml

```

Take a look at the tasks created using cli

```sh
tkn task ls
```

We will be using `buildah` and `s2i-python-3` tasks also which gets installed along with Operator. Operator installs few ClusterTask which you can see.

```sh
tkn clustertask ls
```

### Create pipeline

We will  create a pipeline that takes the source code of the application from GitHub and then builds and deploys it on OpenShift.

This pipeline performs the following:

    1. Clones the source code of the frontend application from a git repository (api-repo resource) and the backend application from a git repository (ui-reporesource)
    2. Builds the container image of frontend using the s2i-python-3 task that generates a Dockerfile for the application using Source-to-Image (S2I). and uses Buildah to build the image
    3. Builds the container image of backend using the buildah task that uses Buildah to build the image
    4. The application image is pushed to an image registry (api-image and ui-image resource)
    5. The new application image is deployed on OpenShift using the apply-manifests and update-deployment tasks.

Apply the pipeline using

```sh
oc create -f pipleine.yaml
```

Check the list of pipelines you have created using the CLI:

```sh
tkn pipeline ls
```

Creating Pipeline resources
We will now create a number of PipelineResources that contain the details of the git repository and image registry to be used in the pipeline. These are reusable across multiple pipelines.

You can see the list of resources created using tkn cli

```sh
tkn resource ls
```

### Running the pipeline

We run our pipleine using a piepline run.

```sh
tkn pipeline start build-and-deploy

? Choose the git resource to use for api-repo: api-repo (http://github.com/openshift-pipelines/vote-api.git)
? Choose the image resource to use for api-image: api-image (image-registry.openshift-image-registry.svc:5000/pipelines-tutorial/api:latest)
? Choose the git resource to use for ui-repo: ui-repo (http://github.com/openshift-pipelines/vote-ui.git)
? Choose the image resource to use for ui-image: ui-image (image-registry.openshift-image-registry.svc:5000/pipelines-tutorial/ui:latest)
Pipelinerun started: build-and-deploy-run-z2rz8

In order to track the pipelinerun progress run:
tkn pipelinerun logs build-and-deploy-run-z2rz8 -f -n pipelines-tutorial
```

We can then view the progress using

```sh
tkn pipeline list

NAME               AGE             LAST RUN                     STARTED          DURATION   STATUS
build-and-deploy   6 minutes ago   build-and-deploy-run-z2rz8   36 seconds ago   ---        Running
```

We can view logs using

```sh
tkn pipeline logs -f
? Select pipeline : build-and-deploy
```

View logs again to check progress

You can get the route of the application by executing the following command and access the application

```sh
  oc get route ui --template='http://{{.spec.host}}'
```

If you want to re-run the pipeline again, you can use the following short-hand command to rerun the last pipelinerun again that uses the same pipeline resources and service account used in the previous pipeline run:

```sh
tkn pipeline start build-and-deploy --last
```


