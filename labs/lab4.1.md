# Services and Service Discovery
## Goal: 
- Create a service and check its behaviour
## Duration: 
0:15:00
## Preparation

- Create a Namespace called shopping
- Create the ReplicaSet whoami from the following descriptor (source:~/workspaces/Lab4/rs--whoami--4.1.yml):
```yaml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: whoami
  namespace: shopping
spec:
  selector:
    matchLabels:
      app: whoami
      level: expert
  replicas: 2
  template:
    metadata:
      labels:
        app: whoami
        level: expert
    spec:
      containers:
        - name: whoami
          image: traefik/whoami:v1.10
          ports:
            - name: main
              containerPort: 80
          args:
            - "--verbose"

```
## First Service creation
- Write a descriptor to create a Service called whoami which listens on port 8080 and willredirect traffi c to all Pods that correspond to app=whoami,level=expert on port 80 (theService must also carry the labels app=whoami,level=expert)
- Complete the following template available in ~/workspaces/Lab4/svc--whoami.yml by replacing XXXX,YYYY and ZZ with the right values.
--- 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: whoami
  namespace: shopping
  labels:
    stack: whoami
spec:
  selector:
    app: XXXX
    level: YYYY
  ports:
    - name: main
      protocol: TCP
      port: 8080
      targetPort: ZZ
```
- List the Services of the shopping namespace
- Show detailed information of whoami Service with kubectl describe
- Take a look at the Enpoints properties
- What is the IP of the Service in the cluster?

## Create a gateway Pod
- Create in the Namespace shopping a gateway Pod from the following descriptor(source:~/workspaces/Lab4/pod--gateway.yml):
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: gateway
  namespace: shopping
  labels:
    app: gw
spec:
  containers:
    - name: shell-in-pod
      image: bitboxtraining/k8s-training-tools:v5
      command:
        - bash
        - -c
        - sleep infinity
```
- Use kubectl exec ... or k9s to open a shell in the Pod and execute commands(replacing clusterIp-du-svc-whoami with the ClusterIP of your Service) 
- Service ping (which will not work): ping -c 1 -W 1 <clusterIp-du-svc-whoami>
- Request the Service: curl <clusterIp-du-svc-whoami>:8080/api

## Does it still work when you kill the Pods?
- From the gateway Pod, run the command curl <clusterIp-du-svc-whoami>:8080/api several times and check that the
hostname key returns diff erent values
- Delete the 2 Pods associated with the ReplicaSet whoami From the gateway Pod, run the command 
curl <clusterIp-du-svc-whoami>:8080/api several times and check that the hostname key returns diff erent values from the previous ones
## What if we increase the number of replicas?
- Scale the ReplicaSet whoami to 5 replicas 
- From the gateway Pod, run the command curl <clusterIp-du-svc-whoami>:8080/api
several times and check that the hostname key returns additional values compared to the previous case
## Using DNS
- Perform the same curl queries but using full (fqdn) and short DNS names
## Service with multiple ports 
- Create the ReplicaSet multi-ports from the following descriptor (source:~/workspaces/Lab4/rs--multi-ports.yml):

```yaml
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: multi-ports
  namespace: shopping
spec:
  replicas: 1
  selector:
    matchLabels:
      app: multi-ports
  template:
    metadata:
      labels:
        app: multi-ports
    spec:
      containers:
        - name: multi-ports-app
          image: alpine:3.16
          command:
            - ash
            - -c
            - |
              (while true; do
                 echo -ne "HTTP/1.1 200 OK\n\nweb $(date)\n" | nc -l -p 80;
               done &
               while true; do
                 echo -ne "HTTP/1.1 200 OK\n\nadmin $(date -Iseconds)\n" | nc -l -p 1234;
               done)
          ports:
            - name: web
              containerPort: 80
            - name: admin
              containerPort: 1234

```
- Create the multi-port Service:
    - which redirects to Pods with app=multi-ports
    - which exposes port web-port on port 80 and port admin-port on port 8000
- Test from the gateway Pod:
    - curl <ip-du-service>:80 must return a string like "web Sun Feb 18 11:14:54 UTC 2024"
    - curl <ip-du-service>:8000 must return a string like "admin 2024-02-18T11:14:54+00:00"