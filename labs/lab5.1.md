# Shared volume


## Goal: 
create a volume shared between 2 containers of the same Pod
### Duration: 
0:05:00
## Initialization
- Create a Pod from the following descriptor (source:
~/workspaces/Lab5/pod--shared-emptyDir-vol--5.1.yml
):

```bash
---
apiVersion: v1
kind: Pod
metadata:
  name: shared-vol-pod
spec:
  volumes:
    - name: var-log-nginx
      emptyDir: {}
  containers:
    - name: log2fs
      image: bitboxtraining/k8s-training-nginx-log2fs:1.19-alpine-v1
      volumeMounts:
        - name: var-log-nginx
          mountPath: /var/log/nginx
    - name: shell
      image: bitboxtraining/k8s-training-tools:v5
      command:
        - bash
        - -c
        - sleep infinity
      volumeMounts:
        - name: var-log-nginx
          mountPath: /data

```

## Check


- Connect into the containerouf shell of the pod
shared-vol-pod with the command:
```bash
kubectl exec -it shared-vol-pod -c shell -- bash 
```
- Check that the directory /data contains the nginx log files
- Run the command curl localhost several times

- Check that the access.log file has been modified
- Disconnect from the container
