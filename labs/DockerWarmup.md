Docker Warmup
Objectifs :
se remémorer les commandes et les principes des conteneurs

simuler comment Kubernetes lance les conteneurs

Accéder à l'environnement Docker
Vérifier que vous pouvez vous connecter au daemon docker
Démarrage d'un conteneur whoami avec Docker
Lancer un conteneur :
en spécifiant son nom: whoami
en mode détaché
à partir de l'image: containous/whoami:latest
Récupérer l'ip du conteneur whoami
Lancer la commande: curl <ip-du-pod-whoami>:80/api
Que se passe-t-il ?
Pourquoi ?
Démarrage d'un conteneur shell avec Docker
Lancer un conteneur :

en spécifiant son nom shell

en mode détaché

à partir de l'image bitboxtraining/k8s-training-tools:v1

dont le process principal est la commande sleep infinity

Se connecter dans le conteneur shell et lancer la commande curl <ip-du-pod-whoami>:80/api

Pourquoi cela fonctionne-t-il ?
(toujours depuis le conteneur shell ) Lancer la commande curl localhost:80/api

Que se passe-t-il ?

Pourquoi ?

Sortir du conteneur shell

Démarrage d'un conteneur shell sidekick du conteneur whoami
Lancer un conteneur :

en spécifiant son nom whoami-shell

en mode détaché

qui utilise le même namespace réseau que le container whoami (--net=container:whoami)

à partir de l'image bitboxtraining/k8s-training-tools:v1

dont le process principal est la commande sleep infinity

Se connecter dans le conteneur whoami-shell et lancer la commande curl <ip-du-pod-whoami>:80/api

Pourquoi cela fonctionne-t-il ?
(toujours depuis le conteneur whoami-shell ) Lancer la commande curl localhost:80/api

Que se passe-t-il ?

Pourquoi ?

Lancer la commande ip addr , quelle est l'ip du conteneur whoami ?

Lancer la commande ss -lntp , quels sont les ports en écoute ?

Sortir du conteneur whoami-shell

Exposer le service whoami en dehors de minikube
Stopper et détruire le conteneur whoami

Le relancer en exposant le port 80 du conteneur sur le port 8080 de la VM hébergeant minikube

Vérifier avec un navigateur que le service est accessible (les urls / et /api doivent répondre)

Nettoyage
Supprimer les conteneurs
whoami
whoami-shell
shell
