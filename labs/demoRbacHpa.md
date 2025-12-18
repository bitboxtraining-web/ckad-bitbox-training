# TP Kubernetes : RBAC et HPA

## Objectifs
- Comprendre et mettre en place **RBAC** pour contrôler les accès aux ressources Kubernetes.
- Comprendre et déployer un **HPA** pour scaler automatiquement une application selon la charge CPU.

---

## Prérequis
- Kubernetes cluster fonctionnel (Minikube, Kind, ou vrai cluster).
- `kubectl` installé et configuré.
- Docker ou autre outil pour construire des images si besoin.
- Namespace `demo` créé pour isoler l'environnement :
```bash
kubectl create ns demo-rbac
```
## Partie 1 : RBAC (Role-Based Access Control)
### 1.1 Création d’un utilisateur fictif
```bash
# Créer un utilisateur service account
kubectl create sa sa-reader -n demo-rbac
```
### 1.2 Créer un Role
```yaml
# role-pod-reader.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: demo-rbac
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

```
Appliquer le role :

```bash
kubectl apply -f role-pod-reader.yaml
```
### 1.3 Lier le Role à l’utilisateur
```yaml
# rolebinding-pod-reader.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: demo-rbac
subjects:
- kind: ServiceAccount
  name: sa-reader
  namespace: demo-rbac
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io


```
Appliquer le binding :

```bash
kubectl apply -f rolebinding-pod-reader.yaml
```
### 1.4 deployer un pod d etest
```bash
kubectl run test --image=nginx -n demo-rbac
```
### Vérifier les permissions avec auth can-i
```bash
# Actions autorisées
kubectl auth can-i get pods --as=system:serviceaccount:demo-rbac:sa-reader -n demo-rbac
kubectl auth can-i list pods --as=system:serviceaccount:demo-rbac:sa-reader -n demo-rbac
kubectl auth can-i watch pods --as=system:serviceaccount:demo-rbac:sa-reader -n demo-rbac

# Actions interdites
kubectl auth can-i create pods --as=system:serviceaccount:demo-rbac:sa-reader -n demo-rbac
kubectl auth can-i delete pods --as=system:serviceaccount:demo-rbac:sa-reader -n demo-rbac
kubectl auth can-i update pods --as=system:serviceaccount:demo-rbac:sa-reader -n demo-rbac
```
## Partie 2 : HPA (Horizontal Pod Autoscaler)
### 2.1 Déployer une application de test
```yaml
# web-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
  namespace: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: k8s.gcr.io/hpa-example
        resources:
          requests:
            cpu: 100m
          limits:
            cpu: 500m
        ports:
        - containerPort: 80

```
Appliquer le déploiement :

```bash
kubectl apply -f web-deploy.yaml
```
### 2.2 Exposer le service
```yaml
# web-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
  namespace: demo
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```
Appliquer le service :

```bash
kubectl apply -f web-service.yaml
```
### 2.3 Créer un HPA
```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
  namespace: demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deploy
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
Appliquer le HPA :

```bash
kubectl apply -f hpa.yaml
```
### 2.4 Générer une charge pour tester le HPA
```bash
kubectl run -i --tty load-generator --image=busybox --restart=Never -- /bin/sh
# Dans le shell :
while true; do wget -q -O- http://web-svc.demo.svc.cluster.local; done
```
### 2.5 Observer le scaling automatique
```bash
kubectl get hpa -n demo -w
kubectl get pods -n demo
```
Quand la CPU dépasse 50%, le nombre de pods augmentera automatiquement jusqu’à 5.

### Partie 3 : Nettoyage
```bash
kubectl delete ns demo
```
Résumé
RBAC permet de contrôler les actions des utilisateurs et services dans Kubernetes.

HPA permet d’adapter le nombre de pods en fonction de la charge CPU ou mémoire.

Cette démo combine sécurité (RBAC) et scalabilité automatique (HPA), deux concepts essentiels pour la production Kubernetes.