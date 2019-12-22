# Catalogue

This repository defines common tasks, pipelines and operations. They are
designed to be reused in many pipelines and applications.

## Tasks

There are two kinds of tasks providing different levels of scope within a
cluster. They are as follows:

1. `ClusterTask` with a cluster scope. This can be installed by a cluster
   operator and made available to users in all namespaces.
1. `Task` with a namespace scope. This can only be installed by an operator
   with access to the namespace, and only effects resources within that
   namespace.

## Using Tasks

First install a `Task` on the cluster:

```
$ kubectl apply -f https://raw.githubusercontent.com/tomasbasham/catalogue/master/tasks/kaniko/kaniko.yaml
task.tekton.dev/kaniko created
```

You can see which Tasks are installed using `kubectl` as well:

```
$ kubectl get tasks
NAME    AGE
kaniko  3s
```

Or if the task has been declared scoped to the entire cluster:

```
$ kubectl get clustertasks
NAME    AGE
kaniko  3s
```

With tasks installed on the cluster (scoped either to the entire cluster or a
namespace) they can be used within a `Pipeline`, being sure to supply the
required input parameters and resources defined within the `Task` definition.

```yaml
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: git-repository
spec:
  type: git
  params:
  - name: revision
    value: master
  - name: url
    value: https://github.com/tomasbasham/project-to-build
---
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: docker-image
spec:
  type: image
  params:
  - name: url
    value: gcr.io/project-name/image-to-build
```

Either install a preconfigured piepline or define one. For example a pipeline
containing a single kaniko task could be defined as follows:

```yaml
apiVersion: tekton.dev/v1alpha1
kind: Pipeline
metadata:
  name: kaniko-pipeline
spec:
  resources:
  - name: source-repo
    type: git
  - name: docker-image
    image: image
  tasks:
  - name: build-docker-image
    taskRef:
      name: kaniko
    resources:
      inputs:
      - name: source
        resource: source-repo
      outputs:
      - name: image
        resource: docker-image
```

Then define a `PipelineRun` which will invoke the `Pipeline` and run the
individual `Task`s.

```yaml
apiVersion: tekton.dev.v1alpha1
kind: PipelineRun
metadata:
  name: kaniko-pipeline-run
spec:
  pipelineRef:
    name: kaniko-pipeline
  resources:
  - name: source-repo
    resourceRef:
      name: git-repository
  - name: docker-image
    resourceRef:
      name: docker-image
```
