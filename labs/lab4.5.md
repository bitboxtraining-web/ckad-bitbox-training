# Headless services
 
## Goal: 
test the implementation of a Headless service
### Duration: 
0:05:00
## Restore previous confi guration for whoami ReplicaSet

- This exercice is in the Namespace shopping
- Restore the previous whoami ReplicaSet confi guration without the Readiness probe
- Delete Pods to ensure they are created without the probe

## whoami-headless


- Create a new whoami-headless Service that redirects traffi c to all Pods that match app=whoami,level=expert
on port 80, but type Headless
- Run the following command and check that the result is the expected one 
```bash
kubectl exec gateway -- nslookup whoami headless
```