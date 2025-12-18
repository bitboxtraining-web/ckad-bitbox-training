# DaemonSets
## Goal: 
- create a DaemonSet
### Duration: 
0:15:00
## Setting up the label on the nodes
- Add the disk=ssd label on all the nodes:
```yaml
kubectl label nodes --all disk=ssd
```
## First DaemonSet
- Create a DaemonSet from the following descriptor (source :~/workspaces/Lab3/ds-ssd-tp-3.2.yml) :
```yaml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disk: ssd
      containers:
        - name: main
          image: debian:11-slim
          command:
            - bash
            - -c
            - |
              while true; do
                echo 'SSD OK'
                sleep 5
              done
      terminationGracePeriodSeconds: 2
```
- List DaemonSets using:
```yaml
kubectl get ds
```
- List Pods associated with the ssd-monitor DaemonSet
- What do you notice?
- Access the logs of the Pods created by the DaemonSet ssd-monitor
- Delete one of the Pod created by the DaemonSet and check that it is recreated
## Changing the label on the nodes
- Replace the disk=ssd label on the minikube node with disk=hdd 
- What do you notice?