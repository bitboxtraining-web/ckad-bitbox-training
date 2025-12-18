# PVC and PV
## Goal: 
Use PV and PVC
### Duration: 
0:10:00
## Pre-requisites
- Enable the csi-hostpath-driver addon:
```bash
minikube addons enable csi-hostpath-driver
```

- Set the corresponding StorageClass as the default one:
```bash
# Remove default annotation from the existing default StorageClass
kubectl annotate storageclass standard storageclass.kubernetes.io/is-default-class-

# Set the default annotation on the csi-hostpath-sc StorageClass
kubectl annotate storageclass csi-hostpath-sc storageclass.kubernetes.io/is-default-class=true

```

## Pre-provision of a PV
- Create a PersistentVolume from the following descriptor
(source:~/workspaces/Lab5/pv--my-hostpath-pv--5.2.yml):

```bash
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-hostpath-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/pv/pv003

```


- Check that the PersistentVolume has been created and that it is currently not bound to any PersistentVolumeClaim
## Creating a PersistentVolumeClaim

- Create a PersistentVolumeClaim from the following descriptor (source:~/workspaces/Lab5/pvc--firstclaim--5.2.yml):

```bash
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-first-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: ""

```

- Check that the PersistentVolumeClaim has been created and that it is bound to the PersistentVolume created previously
- Check the status of the PersistentVolume created previously
- What is the capacity associated with the PVC created? Why?
## Using the PVC
- Adapt the following descriptor (source:~/workspaces/Lab5/pod--use-pvc-bootstrap--5.2.yml):
```bash
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-pv
spec:
  containers:
    - name: log2hostfs
      image: bitboxtraining/k8s-training-nginx-log2fs:1.19-alpine-v1
    - name: shell
      image: bitboxtraining/k8s-training-tools:v5
      command:
        - bash
        - -c
        - sleep infinity

```
- Make the container log2hostfs mount a volume called var-log-nginx on the directory /var/log/nginx
- Make volume var-log-nginx a volume provisioned from a PersistentVolumeClaim

## check
- Connect into the container shell 
- Run the command curl localhost several times
- Check where was the file access.log created ?

    - Check on which node the Pod was created
    - Check the hostPath of the created persistent volume
    - Connect to the node minikube ssh --node <node> and check in the path foundabove

## Creating a second PVC
- Create a second PVC from the following descriptor (source:~/workspaces/Lab5/pvc--second-claim--5.2.yml):
```bash
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-second-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: ""

```
- What is the state of the PVC?
## Fix the situation
- Do what is necessary so that the second PVC can be bound to a PV:
    - either by manually creating another PV
    - or by recreating the PVC by modifying the storageClassName (either completelyremove the fi eld, the default StorageClass will be used, or use the name of the existingStorageClass kubectl get storageclass)Creating a second PVC Optional: Fix the situation