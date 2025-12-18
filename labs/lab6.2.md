#  Secrets

## Goal: 
create secrets and read them from a Pod
### Duration:
0:10:00

## Namespace secret-ops

- Create a Namespace secret-ops
## Create a Secret


- Create a Secret named secret-identities in the Namespace secret-ops with thefollowing key/values:
    - spiderman=Peter Parker
    - superman=Clark Kent

## Use the Secret / environment variable



- Create a Pod from the following descriptor (source:~/workspaces/Lab6/pod--civil-war-expose-secret-identity--6.2.yml):

```bash
---
apiVersion: v1
kind: Pod
metadata:
  name: civil-war
  namespace: secret-ops
spec:
  containers:
    - name: revelation
      image: debian:11-slim
      command:
        - bash
        - -c
        - env; sleep infinity
      env:
        - name: SPIDERMAN_IS
          valueFrom:
            secretKeyRef:
              name: secret-identities
              key: spiderman

```


- Check the value of the environment variable in the Pod's logs
## Use the Secret / files
- Create a Pod from the following descriptor (source:~/workspaces/Lab6/pod--dc-vs-marvel--6.2.yml):
```bash
---
apiVersion: v1
kind: Pod
metadata:
  name: dc-vs-marvel
  namespace: secret-ops
spec:
  volumes:
    - name: for-your-eyes-only
      secret:
        secretName: secret-identities
  containers:
    - name: revelations
      image: debian:11-slim
      command:
        - bash
        - -c
        - grep '^' /etc/revelations/* ; sleep infinity
      volumeMounts:
        - name: for-your-eyes-only
          mountPath: /etc/revelations
          readOnly: true

```
- Connect in the Pod and check the content of the fi les in the /etc/revelations/ folder