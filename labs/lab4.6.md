
# Port forward
## Goal: 
- access the whoami Pod using the port-forward
### Duration: 
0:05:00

- This exercice is in the Namespace shopping
- Show online help for kubectl port-forward
- Launch a whoami Pod with the command:
```bash
kubectl run whoami --image=traefik/whoami:v1.10 --port=80
```
- Use the kubectl port-forward command to forward port 80 of the whoami Pod to the local port 8888
- Check that the url http://localhost:8888/api
returns the expected result with thecommand curl or with a browser if you useminikube locally.
```bash
curl http://localhost:8888/api
```


- Delete the whoami Pod