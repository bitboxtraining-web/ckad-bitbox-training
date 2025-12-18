#  Deployments
## Goal: 
- simulate the rolling update process of an application
### Duration: 
0:10:00
## Deployment v1


- Check the Deployment descriptor ~/workspaces/Lab7/deploy--bitboxtraining-v1--7.1.yml :
```bash
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bitboxtraining
  labels:
    app: bitboxtraining
spec:
  replicas: 3
  minReadySeconds: 5
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      app: bitboxtraining
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: bitboxtraining
    spec:
      containers:
        - name: app
          image: bitboxtraining/k8s-training-deploy:v1
          ports:
            - name: main
              containerPort: 8080
          livenessProbe:
            httpGet:
              path: /healthcheck
              port: 8080
            periodSeconds: 5
            initialDelaySeconds: 2
          readinessProbe:
            httpGet:
              path: /readicheck
              port: 8080
            periodSeconds: 5
            initialDelaySeconds: 5


```
- Create the Deployment with:
```bash
kubectl apply -f ~/workspaces/Lab7/deploy--bitboxtraining-v1--7.1.yml --record
```
- Check the state of the Deployment and the associated ReplicaSet and Pods


## Update the Deployment to v2
- Update the Deployment descriptor, and change the image tag value to v2
- Apply the change
- Launch to check the deployment status:
```bash
kubectl rollout status deployment bitboxtraining
```
- Check the state of the Deployment and the associated ReplicaSet and Pods
- Check the output of the loop running in the gateway Pod that the application is being updated
- View all recorded revisions for the Deployment:
```bash
kubectl rollout history deploy bitboxtraining