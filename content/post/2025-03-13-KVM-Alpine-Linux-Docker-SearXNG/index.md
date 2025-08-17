+++
title = 'cwwk KVM - SearXNG Alpine Linux (alpine-searx)'
date = 2025-03-13 00:00:00 +0100
categories = ['virtuel']
+++
*Alpine Linux est une distribution Linux ultra-légère...*

![](alpine-linux-logo.png){:width="300"}  

## Alpine Linux

*Création machine virtuelle Alpine  de type KVM avec 2 Go de RAM, 1 cœur de processeur et 5 Go de disque dur.*

* [Création machines virtuelles KVM avec la commande virsh](/posts/Installer_KVM_Kernel_Virtual_Machine_sur_un_serveur/#création-machines-virtuelles-kvm-avec-la-commande-virsh)
* [Accéder aux machines virtuelles KVM via le client VNC](/posts/Installer_KVM_Kernel_Virtual_Machine_sur_un_serveur/#accéder-aux-machines-virtuelles-kvm-via-le-client-vnc)
* [Wiki Alpine Linux](https://wiki.alpinelinux.org/wiki/)

### Créer alpine-searx sur un serveur

[Les dernières images Alpine Linux](https://alpinelinux.org/downloads/)  

Création d'une image virtuelle **alpine-searx** sous le serveur Lenovo rnmkcy.eu  
On se connecte sur le serveur Lenovo en SSH, puis on exécute la commande suivante pour créer  une machine virtuelle Alpine avec 2 Go de RAM, 1 cœur de processeur et 5 Go de disque dur

```shell
sudo virt-install \
--osinfo alpinelinux3.17 \
--name alpine-searx \
--memory 2048 \
--vcpus 1 \
--cpu host \
--hvm \
--disk path=/srv/kvm/libvirt/images/alpine-searx.qcow2,format=qcow2,size=5 \
--cdrom /srv/kvm/libvirt/boot/alpine-standard-3.20.3-x86_64.iso \
--network bridge=br0 \
--graphics vnc  
```

Note: La dernière version Alpine Linux est alpinelinux3.20  au 10/07/2024 mais KVM ne connait que alpinelinux3.17 (`sudo virt-install --osinfo list |grep alpine`)

Après exécution dans un terminal de la commande ci dessus, on arrive sur l'écran suivant  
![](alpine-linux01.png)  

### Configurer alpine-searx

Le serveur Lenovo n'a pas d'affichage, il faut créer un tunnel ssh depuis un poste client

    ssh -L 5900:127.0.0.1:5900 leno@192.168.0.215 -p 55215 -i /home/yann/.ssh/lenovo-ed25519

Puis lancer de ce même poste un client VNC  
![](alpine-linux02.png){:width="300"}  
la console s'affiche   
![](alpine-linux03.png)  

Une fois l'image ISO lancée, on arrive à un invite de connexion.   
Indiquez `root` comme nom d'utilisateur, aucun mot de passe ne vous sera demandé à cette étape.   

Le système est utilisable, mais on veut l'installer, ce qui passe par la commande suivante (clavier qwerty)

```
setup-alpine # saisir setup)qlpine
```

Une suite de questions :  
![](alpine-linux04.png)  
mot de passe root (toorrtyuiop)  
![](alpine-linux05.png)  
APK mirror (f) ,patienter ...  
![](alpine-linux06.png)  

user : alsearx  
password: MoelleEndetteSingle   

Relever l'adresse ip allouée : `ip a` --> 192.168.0.31  
Puis redémarrer : `reboot`  
La fenêtre vnc se ferme  


### Explications sur la procédure

*Normalement, vous n'avez rien à faire, les paramètres par défaut doivent convenir. Mais si vous le désirez, vous pouvez les modifier pour utiliser une interface particulière, une IP fixe, un serveur proxy, etc.  
Une soixantaine de serveurs mirroir vous seront proposés pour télécharger les paquets. Choisissez un numéro dans la liste ou demandez au système de les tester et de sélectionner le plus rapide. Vous pouvez aussi modifier le fichier des sources. Il vous faudra ensuite choisir votre serveur SSH : OpenSSH, Dropbear ou aucun.* 

On termine par la méthode d'installation. Il en existe quatre : 

*    none : le système et ses données sont placés en RAM et seront perdus après le redémarrage
*    sys : le système et ses données sont placés sur un HDD/SSD
*    data : le système est placé en RAM, les données sur un HDD/SSD
*    lvm : utilisation de Logical Volume Manager, les deux choix précédents seront proposés (lvmsys, lvmdata)

Si vous stockez le système en mémoire, il faudra trouver un moyen de sauvegarder la configuration. Vous pourrez le faire uniquement depuis un lecteur de disquettes (!) ou une clé USB. Une fois le système installé, vous pourrez l'utiliser directement s'il est placé en mémoire ou redémarrer si vous avez opté pour un stockage classique.

Il n'est pas conseillé d'utiliser directement le compte root pour les actions du quotidien.  
Si utilisateur non créé dans la procédure d'installation, le créer avec son propre espace dans /home/ 

    adduser alsearx

Vous pouvez utiliser l'utilisateur pour vous connecter via SSH (impossible avec le compte root)  

### Connexion alpine-searx via SSH

Sur un poste linux du réseau

    ssh alsearx@192.168.0.31

Une fois connecté ,vous pouvez accéder au "root" de manière classique avec la commande :

    su -

Mise à jour

```shell
apk update
apk upgrade 
# Vous pouvez fusionner les deux lignes avec 
apk -U upgrade
```

Editeur nano (Vous pouvez aussi opter pour vi qui est nativement présent sur le système)

    apk add nano

### Réseau - IP statique

[How to configure static IP address on Alpine Linux](https://www.cyberciti.biz/faq/how-to-configure-static-ip-address-on-alpine-linux/)

Le fichier de configuration `/etc/network/interfaces`

    /etc/network/interfaces

```shell
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
        address 192.168.100.70/24
        gateway 192.168.100.1
```

Fichier de résolution dns

    /etc/resolv.conf

```
nameserver 1.1.1.1
nameserver 9.9.9.9
```

Les modifications apportées à /etc/network/interfaces peuvent être activées en exécutant

```shell
service networking restart 
```

### OpenSSH avec clés

*Connexion ssh sur un autre port avec un jeu de clés*

Générer une paire de clé sur l'ordinateur de bureau PC1  
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec le serveur.

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/vm-alpine-searx

Copier la clé publique `cat ~/.ssh/vm-alpine-searx.pub` dans le presse-papier

	ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBQu0LRiDjPM7BJAfjECl22fnEchVuxnw38RV2IJs0+l yann@yann-pc1

On se connecte sur la machine virtuelle alpine linux "ttrss alpine-searx"

    ssh alsearx@192.168.100.70

Créer le répertoire et ouvrir nouveau fichier

    mkdir -p $HOME/.ssh/
    nano $HOME/.ssh/authorized_keys

Coller le contenu du presse-papier , sauver le fichier et sortir

Modifier les droits

    chmod 600 $HOME/.ssh/authorized_keys

Passer en mode su

    su -

Modifier la configuration serveur SSH

    nano /etc/ssh/sshd_config

Modifier

```
Port = 55216
PasswordAuthentication no
```

Relancer le serveur

    service sshd restart

Test connexion

    ssh -p 55216 -i ~/.ssh/vm-alpine-searx alsearx@192.168.100.70

### sudo

Passer en root

    su -

Editer la configuration des dépôts

    nano /etc/apk/repositories

Trouvez maintenant la ligne qui se termine dans /community Ensuite, retirez le # au début de la ligne. Le fichier résultant devrait ressembler à ceci

```
#/media/cdrom/apks
http://alpinelinux.mirrors.ovh.net/v3.20/main
http://alpinelinux.mirrors.ovh.net/v3.20/community
```

Mise à jour des dépôts

    apk update

Installer sudo

    apk add sudo

Pas de mot de passe sudo pour l'utilisateur alouest

    echo "alsearx     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

## Docker

![](docker-logo-a.png)  

### Installer docker sur Alpine Linux

[Install docker & docker-compose on Alpine Linux](https://geekscircuit.com/install-docker-docker-compose-on-alpine-linux/)

Passer en root

    su -

Editer la configuration des dépôts

    nano /etc/apk/repositories

Trouvez maintenant la ligne qui se termine dans /community Ensuite, retirez le # au début de la ligne. Le fichier résultant devrait ressembler à ceci

```
#/media/cdrom/apks
http://alpinelinux.mirrors.ovh.net/v3.20/main
http://alpinelinux.mirrors.ovh.net/v3.20/community
```

Installer docker et docker-compose

```bash
apk update
apk add docker docker-compose
```

Activer autostart sur boot en utilisant

```bash
rc-update add docker default
```

puis vous pouvez lancer le service docker en utilisant la commande

```bash
/etc/init.d/docker start
# ou
service docker start
```

## SearxNG

*[SearXNG](https://docs.searxng.org/) est un métamoteur qui recherche ses informations à travers plusieurs moteurs de recherche généralistes*

### Image docker

Si vous prévoyez de construire et de maintenir une image Docker par vous-même, assurez-vous que Docker est installé. Sous Linux, n'oubliez pas d'ajouter votre utilisateur au groupe Docker (déconnectez-vous et reconnectez-vous pour que votre appartenance au groupe soit réévaluée) :

    sudo adduser $USER docker

L'image docker est basée sur git://Dockerfile et disponible sur searxng/searxng @dockerhub. L'utilisation de l'image docker est assez simple, par exemple vous pouvez extraire l'image searxng/searxng @dockerhub et déployer une instance locale en utilisant docker run:

```bash
mkdir -p YannSearXNG
cd YannSearXNG
export PORT=8080
docker pull searxng/searxng
```

Pour avoir un démarrage automatique au boot : `--restart=always --name restart_always` 

```
cd ~/YannSearXNG
export PORT=8080

docker run   -d -p ${PORT}:8080 \
             --restart=always \
             --name searxng \
             -v "${PWD}/searxng:/etc/searxng" \
             -e "BASE_URL=http://localhost:$PORT/" \
             -e "INSTANCE_NAME=YannSearXNG" \
             searxng/searxng
```

Les variables d'environnement `UWSGI_WORKERS` et `UWSGI_THREADS` remplacent le nombre par défaut de processus UWSGI et de threads UWSGI spécifiés dans le  fichier `/etc/searxng/uwsgi.ini`.

Ouvrez votre navigateur WEB et visitez l'URL :

    xdg-open http://192.168.100.70:8080 

### Paramétrage searxng

Dans ${PWD}/searxng, vous trouverez `settings.yml` et `uwsgi.ini`. Vous pouvez modifier ces fichiers selon vos besoins

```
# résultat sur un nouvel onglet
sudo sed -i -e "s/\# results_on_new_tab: false/results_on_new_tab\: true/g" $HOME/YannSearXNG/searxng/settings.yml

# style du theme simple: sombre
sudo sed -i -e "s/simple_style\: auto/simple_style\: dark/g" $HOME/YannSearXNG/searxng/settings.yml

```

Identifier le container : `docker ps`

```
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                                       NAMES
e3234ae64818   searxng/searxng   "/sbin/tini -- /usr/…"   35 seconds ago   Up 34 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   searxng
```

et redémarrer l'image Docker.

    docker container restart e3234ae64818

### Mise à jour automatique

[Watchtower](https://github.com/v2tec/watchtower) est un outil open-source permettant d'automatiser les mises à jour.

Une autre personnalisation de l'image au format docker-compose : docker-compose.yml

```yaml
version: '2.1'
services:
  watchtower:
    image: containrrr/watchtower:latest
    container_name: watchtower
    restart: unless-stopped
    environment:
      - WATCHTOWER_SCHEDULE=0 45 15 * * *
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_INCLUDE_RESTARTING=true
      - WATCHTOWER_DEBUG=true
      - WATCHTOWER_HTTP_API_METRICS=false
      - TZ=Europe/Paris
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

Pour créer le conteneur, on se place (cd) dans le dossier où se trouve le fichier docker-compose.yml et on tape :

```shell
docker-compose up -d
```

Résultat commande

```
[+] Running 1/1
 ✔ Container watchtower  Started                                                                                                                                       0.2s 
```

Ainsi, nous exécutons Watchtower en mode détaché, ce qui lui donne accès au **socket Docker**, qui est nécessaire à Watchtower pour surveiller et mettre à jour les conteneurs. 

Plus besoin donc de se soucier de mettre à jour ses containers, Watchtower le fait tout seul, et ça fonctionne parfaitement !

    docker logs watchtower

```
time="2025-06-05T15:39:38+02:00" level=debug msg="Sleeping for a second to ensure the docker api client has been properly initialized."
time="2025-06-05T15:39:40+02:00" level=debug msg="Making sure everything is sane before starting"
time="2025-06-05T15:39:40+02:00" level=debug msg="Retrieving running and restarting containers"
time="2025-06-05T15:39:40+02:00" level=debug msg="There are no additional watchtower containers"
time="2025-06-05T15:39:40+02:00" level=debug msg="Watchtower HTTP API skipped."
time="2025-06-05T15:39:40+02:00" level=info msg="Watchtower 1.7.1"
time="2025-06-05T15:39:40+02:00" level=info msg="Using no notifications"
time="2025-06-05T15:39:40+02:00" level=info msg="Checking all containers (except explicitly disabled with label)"
time="2025-06-05T15:39:40+02:00" level=info msg="Scheduling first run: 2025-06-05 15:45:00 +0200 CEST"
time="2025-06-05T15:39:40+02:00" level=info msg="Note that the first check will be performed in 5 minutes, 19 seconds"
```

### Commandes docker

Utilisez la commande `container ls` pour lister les conteneurs en cours d'exécution, ajoutez le drapeau `-a` pour lister les conteneurs éteints également. La commande `container stop` permet d'arrêter un conteneur en cours d'exécution. Pour se débarrasser d'un conteneur, utilisez la commande `container rm`:

```
localhost:~/YannSearXNG$ docker container ls

CONTAINER ID   IMAGE             COMMAND                  CREATED              STATUS              PORTS                                       NAMES
e8557899b8e9   searxng/searxng   "/sbin/tini -- /usr/…"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   nice_chaplygin
```

Si vous n'utilisez plus Docker et que vous souhaitez vous débarrasser de tous les conteneurs et images, utilisez la commande prune suivante :

```
docker stop $(docker ps -aq)       # stop all containers
docker system prune                # make some housekeeping
docker rmi -f $(docker images -q)  # drop all images
```

**Mise à jour automatique du container**  
Watchtower est un outil open-source permettant d'automatiser ces mises à jour. Il se présente sous la forme d'une image Docker à lancer très simplement 

```
docker run \
    --name watchtower \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    --detach \
    v2tec/watchtower \
    --interval "30"
```

Le paramètre `--volume` est obligatoire pour que Watchtower puisse communiquer avec le démon Docker sur le système hôte.

### Proxy nginx searx.rnmkcy.eu

Sur le serveur Lenovo rnmkcy.eu, créer un proxy pour SearXNG `/etc/nginx/conf.d/searx.rnmkcy.eu.conf`

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name  searx.rnmkcy.eu;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://searx.rnmkcy.eu$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  searx.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;

    location / {
	proxy_pass http://192.168.100.70:8080;
    }
}
```

Recharger nginx

    sudo systemctl reload nginx

Ouvrir le lien <https://searx.rnmkcy.eu>  

![](searx.rnmkcy.eu.png){:width="500"}  

## Maintenance

### Alpine Linux - Mises à jour automatique

Installer curl

    sudo apk add curl

Pour chaque nouveau serveur Alpine Linux, créer un script shell nommé apk-autoupgrade dans le dossier /etc/periodic/daily/ avecles permissions suivantes : 700

```shell
echo -e "#!/bin/sh\napk upgrade --update | sed \"s/^/[\`date\`] /\" >> /var/log/apk-autoupgrade.log" > /etc/periodic/daily/apk-autoupgrade && \
	chmod 700 /etc/periodic/daily/apk-autoupgrade
```

Si les tâches cron ne sont pas activées

```shell
rc-service crond start
rc-update add crond
```

Le script exécute la commande apk upgrade --update une fois par jour, apk par défaut ne demande jamais l’intervention de l’utilisateur. 

Additif pour notification ntfy

```
echo "#
curl \
-H "X-Email: ntfy@cinay.eu" \
-H "Title: 💻 Alpine Linux SearXNG alpine-searx : Fin exécution script 'apk-autoupgrade'" \
-H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
-H prio:low \
-d "SearXNG 192.168.100.70 alpine-searx 
✔️ Fin exécution script /etc/periodic/daily/apk-autoupgrade" \
https://noti.rnmkcy.eu/yan_infos
" >> /etc/periodic/daily/apk-autoupgrade
```

