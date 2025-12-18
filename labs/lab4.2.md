# NodePort and LoadBalancer
## Goal: 
Expose a Service to the outside in 2 ways
### Duration: 
0:15:00

## NodePort
- This exercice is in the Namespace shopping
- Change the whoami Service to a NodePort Service type without specifying the nodePortvalue
- Check with kubectl get svc whoami
- Test the Service locally on minikube ipusing the nodePort
```yaml
curl$(minikube ip):<NODEPORT>
```

In this lab we need an additional tool to expose services directly on the host VM, this is due tothe fact that our nodes are spawned as containers To do this we use the tool socat with acontainer exposing a host port and redirecting to a minikube node
- Expose the nodePort outside the minikube with:
```bash
WHOAMI_NODE_PORT=$(
  kubectl get svc \
    -n shopping \
    whoami \
    -ojsonpath="{.spec.ports[0].nodePort}"
)

docker container run \
  --name expose-port-${WHOAMI_NODE_PORT} \
  --detach \
  --network minikube \
  --publish ${WHOAMI_NODE_PORT}:${WHOAMI_NODE_PORT} \
  alpine/socat \
    tcp-listen:${WHOAMI_NODE_PORT},fork,reuseaddr \
    tcp-connect:minikube:${WHOAMI_NODE_PORT}


```
- Test the Service from outside the minikube using the nodePort
- Show detailed information of whoami Service 

## LoadBalancer

- Retrieve the minikube IP with minikube ip (e.g. 192.168.49.2).
- Configure the minikube addon metallb with the command
```yaml
minikube addons configure metallb
```

    - StartIP: minikube IP whith last octet set to 10 (e.g. 192.168.49.10)
    - End IP: minikube IP whith last octet set to 250 (e.g. 192.168.49.250)
- Activate the minikube addon metallb with the command
```yaml
minikube addons enable metallb
```


- Remove whoami Service
- Recreate it with type LoadBalancer
- Check with kubectl get svc whoami
- Test the Service from the kubernetes machine on the announced External IP using the Service port
- Show detailed information of whoami Service