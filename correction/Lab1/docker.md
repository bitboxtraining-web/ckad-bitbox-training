## TP 1.4 : Docker Warmup

### Accéder à l'environnement Docker

```shell
docker version
#    Client: Docker Engine - Community
#     Version:           20.10.1
#     API version:       1.41
#     Go version:        go1.13.15
#     Git commit:        831ebea
#     Built:             Tue Dec 15 04:35:01 2020
#     OS/Arch:           linux/amd64
#     Context:           default
#     Experimental:      true
#
#    Server: Docker Engine - Community
#     Engine:
#      Version:          20.10.1
#      API version:      1.41 (minimum version 1.12)
#      Go version:       go1.13.15
#      Git commit:       f001486
#      Built:            Tue Dec 15 04:32:57 2020
#      OS/Arch:          linux/amd64
#      Experimental:     false
#     containerd:
#      Version:          1.4.3
#      GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
#     runc:
#      Version:          1.0.0-rc92
#      GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
#     docker-init:
#      Version:          0.19.0
#      GitCommit:        de40ad0
```

### Démarrage d'un conteneur whoami avec Docker

```shell
docker  run --name whoami --detach containous/whoami:latest
#    Unable to find image 'containous/whoami:latest' locally
#    latest: Pulling from containous/whoami
#    29015087d73b: Pull complete
#    0109a00d13bc: Pull complete
#    d3caffff64d8: Pull complete
#    Digest: sha256:7d6a3c8f91470a23ef380320609ee6e69ac68d20bc804f3a1c6065fb56cfa34e
#    Status: Downloaded newer image for containous/whoami:latest
#    e7334163b24f16031322b54ab4e9660830a575e6e495a2b677812762a4edcb9d
```

```shell
WHOAMI_IP=$(docker container inspect --format "{{ .NetworkSettings.IPAddress }}" whoami)
echo ${WHOAMI_IP}
#    172.17.0.7
```

```shell
curl ${WHOAMI_IP}:80/api
#    {"hostname":"e7334163b24f","ip":["127.0.0.1","172.17.0.7"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.29.0"]},"url":"/api","host":"172.17.0.7","method":"GET"}
```

Cela fonctionne car les conteneurs sont accessibles localement grâce au bridge `docker0` :

```shell
# After installing `brctl` (`sudo apt install bridge-utils` on Ubuntu)
brctl show docker0
#    bridge_name    bridge_id               STP enabled     interfaces
#    docker0        8000.0242f2449856       no              veth055f330

ip a
#    3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
#    link/ether 02:42:e7:d4:81:51 brd ff:ff:ff:ff:ff:ff
#    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
#       valid_lft forever preferred_lft forever
#    inet6 fe80::42:e7ff:fed4:8151/64 scope link
#       valid_lft forever preferred_lft forever
#    20: veth055f330@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
#        link/ether 86:a6:f0:e4:1c:eb brd ff:ff:ff:ff:ff:ff link-netnsid 1
#        inet6 fe80::84a6:f0ff:fee4:1ceb/64 scope link
#           valid_lft forever preferred_lft forever
```

```shell
curl localhost:80/api
#    curl: (7) Failed connect to localhost:80; Connection refused
```

Cela ne fonctionne pas car le service `whoami` du conteneur n'écoute que au sein de celui-ci et que le port n'a pas été exposé sur la machine hôte.

### Démarrage d'un conteneur shell avec Docker

```shell
docker  run --name shell --detach bitboxtraining/k8s-training-tools:v1 sleep infinity
#    Unable to find image 'bitboxtraining/k8s-training-tools:v1' locally
#    v1: Pulling from bitboxtraining/k8s-training-tools
#    6c33745f49b4: Pull complete
#    f6a08eab62bf: Pull complete
#    Digest: sha256:0364b3fc6935c17afe4c2eb3c75ef3718054829076e050bfc857960328fe7f40
#    Status: Downloaded newer image for bitboxtraining/k8s-training-tools:v1
#    3f428f558cec36c7fd1c5664675a0f1e0fa31dd372c4dd465581441431df5220
```

```shell
docker  exec shell curl --silent ${WHOAMI_IP}:80/api
#    {"hostname":"e7334163b24f","ip":["127.0.0.1","172.17.0.7"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.29.0"]},"url":"/api","host":"172.17.0.7","method":"GET"}
```

Cela fonctionne car les conteneurs du même _network_ peuvent s'atteindre par leur IP (et leur nom s'il ne sont pas sur le bridge par défaut).

```shell
docker  exec --interactive --tty shell bash
> curl localhost:80/api
#    curl: (7) Failed to connect to localhost port 80: Connection refused
> exit
```

Cela ne fonctionne pas car chaque conteneur possède sa propre pile réseau et donc son propre `localhost`.

### Démarrage d'un conteneur shell sidekick du conteneur whoami

```shell
docker  run --name whoami-shell --detach --net=container:whoami bitboxtraining/k8s-training-tools:v1 sleep infinity
#    2e298d03f951e65f3aa5cabdcfdb5eaa1d75532ddefb0eab214c39a4a9578b6a
```

```shell
docker  exec whoami-shell curl --silent ${WHOAMI_IP}:80/api
#    {"hostname":"e7334163b24f","ip":["127.0.0.1","172.17.0.7"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.29.0"]},"url":"/api","host":"172.17.0.7","method":"GET"}
```

Aucune différence par rapport au conteneur `shell`.

```shell
docker container exec whoami-shell curl --silent localhost:80/api
#    {"hostname":"e7334163b24f","ip":["127.0.0.1","172.17.0.7"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.29.0"]},"url":"/api","host":"localhost","method":"GET"}
```

Cela fonctionne car on utilise la même pile réseau que le conteneur `whoami` (`--net=container:whoami`).

```shell
docker  exec --interactive --tty whoami-shell bash
> ip addr
#    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
#        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
#        inet 127.0.0.1/8 scope host lo
#           valid_lft forever preferred_lft forever
#    14: eth0@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
#        link/ether 02:42:ac:11:00:07 brd ff:ff:ff:ff:ff:ff link-netnsid 0
#        inet 172.17.0.7/16 brd 172.17.255.255 scope global eth0
#           valid_lft forever preferred_lft forever
> ss -lptn
State       Recv-Q   Send-Q   Local Address:Port   Peer Address:Port
LISTEN      0        128           *:80              *:*
> exit
```

### Exposer le service whoami en dehors de minikube

```shell
docker rm --force whoami
#    whoami
```

```shell
docker  run --name whoami --detach --publish 8080:80 containous/whoami:latest
#    8d412d0766ac2332e8212a47192aae8a3b871ee272432608615f8ea4349ef6d2
```

```shell
curl localhost:8080/api
#    {"hostname":"8d412d0766ac","ip":["127.0.0.1","172.17.0.7"],"headers":{"Accept":["*/*"],"User-Agent":["curl/7.29.0"]},"url":"/api","host":"localhost:8080","method":"GET"}
```

### Nettoyage

```shell
docker  rm --force whoami shell whoami-shell
#    whoami
#    shell
#    whoami-shell
```
