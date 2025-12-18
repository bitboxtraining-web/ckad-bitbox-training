# Principe de base

- requests = la quantité minimale de ressource (CPU ou mémoire) que le container demande au scheduler pour être placé sur un nœud.
→ Le scheduler additionne les requests des containers d’un Pod pour décider sur quel nœud il peut être planifié.

- limits = la limite maximale que le container est autorisé à utiliser au moment de l’exécution (enforceée par le kubelet via cgroups).
→ Si le container dépasse le limit, il sera throttlé (CPU) ou tué (memory).

## 2) Différence CPU vs Mémoire (comportement concret)
### CPU

- Unité : 1 = 1 cœur CPU, 500m = 500 millicores = 0.5 cœur.

- Si un container dépasse son limit CPU, il n’est pas tué : il est throttlé (sa consommation est réduite par cgroups).

- Si il dépasse son request CPU mais reste en dessous du limit, il peut utiliser la CPU disponible sur le nœud tant qu’il n’est pas limité par limit.

### Mémoire (RAM)

- Unité : Mi (Mebibytes), Gi (Gibibytes) — ex : 256Mi, 2Gi.

- Si un container dépasse son limit mémoire, le kernel (OOM killer) va tuer le processus du container → OOMKilled.

- Il n’y a pas de throttling pour la mémoire : dépassement = risque de kill.

## 3) Comment le scheduler et le kubelet utilisent ces valeurs

- Scheduling : Kubernetes regarde les requests (somme de tous les containers) pour savoir si le Pod rentre sur un nœud (par rapport aux ressources allocatable du nœud).

- Exécution : Kubelet / cgroups impose les limits.

- Eviction : Si le nœud manque de mémoire, les Pods avec faible QoS ou qui dépassent leur request sont plus susceptibles d’être évincés.

## 4) QoS Classes (très important)

- La combinaison requests/limits détermine la QoS (Quality of Service) d’un Pod :

  - Guaranteed : pour chaque ressource (CPU et mémoire) requests == limits pour tous les containers → le Pod a la priorité la plus haute pour ne pas être evicté.

  - Burstable : si au moins un container a des requests mais requests != limits → prioritaire mais moins que Guaranteed.

  - BestEffort : si aucune ressource n’a de requests → priorité la plus basse, fortement évictable.

## 5) Exemple YAML & explication
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:
        cpu: "500m"     # 0.5 core demandé au scheduler
        memory: "256Mi" # 256 MiB demandé au scheduler
      limits:
        cpu: "1"        # max 1 core, au-delà -> throttling
        memory: "512Mi" # max 512Mi, au-delà -> OOMKilled
  - name: sidecar
    image: sidecar:1.0
    resources:
      requests:
        cpu: "250m"
        memory: "64Mi"
      limits:
        cpu: "500m"
        memory: "128Mi"

```
- Comment le scheduler voit le Pod :

  - CPU requests total = 500m + 250m = 750m = 0.75 core.

  - Memory requests total = 256Mi + 64Mi = 320Mi.
- Le nœud doit avoir au moins 0.75 core et 320Mi de mémoire libre allocatable pour accepter ce Pod.

- Comportement si containers consomment plus :

  - Si app utilise 1.2 cores → il sera limité à 1 core (throttled).

  - Si app utilise 600Mi mémoire → dépasse 512Mi → OOMKilled possible.

## 6) Exemples numériques — comment raisonner

- cpu: "500m" = 0.5 cœur.
- Addition : 500m + 250m = 750m → c’est 0.75 cœur.

- memory: "256Mi" + 64Mi = 320Mi.

- Toujours additionner les requests de tous les containers d’un Pod pour la planification.

## 7) Bonnes pratiques (pragmatiques)

- Définir toujours des requests réalistes. Sans request, Pod devient BestEffort et risque d’être évincé.

- Mettre un limit raisonnable pour éviter qu’un container "bruyant" n’affecte les autres.

- Pour QoS Guaranteed, si tu veux éviter les évictions, mets requests == limits pour CPU et mémoire.

- Ne mets pas requests trop faibles (le scheduler pourrait placer trop de Pods sur un nœud → contention).

- Surveille : utilise kubectl top pod (metrics-server) ou Prometheus pour connaître l’utilisation réelle avant d’ajuster les valeurs.

- Start small, then tune : commencer par requests = moyenne observée, limits = pic acceptable.

## 8) Commandes utiles pour debug / observabilité

- Voir détails d’un Pod (événements, raisons d’OOM) :
```bash
kubectl describe pod <pod-name>
```
- Voir utilisation en temps réel (si metrics-server installé) :
```bash
kubectl top pod <pod-name>
``` 
- Voir les ressources demandées et limites dans YAML :
```bash
kubectl get pod <pod-name> -o yaml
```
- Evénements OOM / Kill apparaissent dans kubectl describe pod (status, last state, events).



## 9) Politique de cluster (quota/limitrange)

- Dans un projet/namespace, un ResourceQuota peut limiter la somme des requests/limits.

- Un LimitRange peut imposer des valeurs par défaut ou des min/max sur requests/limits pour chaque Pod/container.
