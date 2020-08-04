# pipelines-tutorial
A repository that walks through tektoncd pipelines examples

## Cluster setup

### Storage

#### cluster_setup/gcp
```
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: pipelines-shared-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: 'true'
provisioner: kubernetes.io/gce-pd
parameters:
  replication-type: none
  type: pd-standard
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pipelines-task-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: pipelines-shared-sc
  resources:
    requests:
      storage: 1
```

## Pipelinerun Examples

### clone-build-push.yaml

* ClusterTask: git-clone - Clones a repository
* Task: extract-build-verify-push-tag
    * Step 1: Inspects the source directory for the image and looks at the Dockerfile
    * Step 2: Builds the container
    * Step 3: Pushes to the internal registry with the git sha of the commit of the repository
    * Step 4: Tag container latest with oc
    * Step 5: Pull local image
              Push to remote registry/organization:GITSHA
              Push to remote registry/organization:latest
* Workspaces - shared-task-storage --> shared-data --> pipelines-task-pvc

_Requirement: A secret with the name regcred is required to push to remote registry_  