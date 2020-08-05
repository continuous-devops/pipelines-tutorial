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
```
This pipeline uses tekton to clone a repository check git sha vs image stream tag 
to see if it has already been built and pushed to local image stream and tagged.  
It then pushes the image built to a remote repository.  There is a check to see
files in the workspace.
```

* Pipeline: ```clone-build-push```
  * ClusterTask: git-clone - Clones a repository
  * Task: ```extract-build-verify-push```
    * Step 1: extract-container-info
      * Inspects the source directory for the image and looks at the Dockerfile
    * Step 2: ```check-image-tag```
      * Check image stream tag in local registry
    * Following steps are executed if the git sha and image stream tag don't exist
      * Step 3: ```build-image``` 
        * Builds the container
      * Step 4: ```push-image```
        * Pushes to the internal registry with the git sha of the commit of the repository
      * Step 5: ```tag-image```
        * Tag container in openshift image stream with latest tag
      * Step 6: ```push-image-to-remote-reg```
        * Pull local image of image stream in OpenShift
        * Push to remote registry/organization:$GITSHA
        * Push to remote registry/organization:latest
  * Task: ```get-shared-workspace-info```
    * Step 1: ```cat-txt-files-workspace```
      * View files in the shared workspace used for the pipeline and underlying tasks
* Workspaces - shared-task-storage --> shared-data --> pipelines-task-pvc

__Requirement:__ _A secret with the name regcred is required to push to remote registry_
__Note:__ _Can't use tekton condition since it doesn't support workspaces_  