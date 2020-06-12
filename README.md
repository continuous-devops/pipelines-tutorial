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
  namespace: cptekton
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
  namespace: cptekton
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: pipelines-shared-sc
  resources:
    requests:
      storage: 1
```
