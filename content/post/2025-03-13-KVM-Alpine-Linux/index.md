+++
title = 'cwwk KVM - Alpine Linux Tiny Tiny RSS (ttrss alpine-vm)'
date = 2025-08-15
categories = ['virtuel']
+++
*Alpine Linux est une distribution Linux ultra-légère...*

![](alpine-linux-logo.png){:width="300" .normal}  ![](ttrss-logo-a.png){:width="80" .normal}  

## Alpine Linux

*Création machine virtuelle Alpine  de type KVM avec 2 Go de RAM, 1 cœur de processeur et 5 Go de disque dur.*

* [Création machines virtuelles KVM avec la commande virsh](/posts/Installer_KVM_Kernel_Virtual_Machine_sur_un_serveur/#création-machines-virtuelles-kvm-avec-la-commande-virsh)
* [Accéder aux machines virtuelles KVM via le client VNC](/posts/Installer_KVM_Kernel_Virtual_Machine_sur_un_serveur/#accéder-aux-machines-virtuelles-kvm-via-le-client-vnc)
* [Wiki Alpine Linux](https://wiki.alpinelinux.org/wiki/)

### Créer vm-alpine sur un serveur

[Les dernières images Alpine Linux](https://alpinelinux.org/downloads/)  

Création d'une image virtuelle **alpine-vm** sous le serveur Lenovo rnmkcy.eu  
On se connecte sur le serveur Lenovo en SSH, puis on exécute la commande suivante pour créer  une machine virtuelle Alpine avec 2 Go de RAM, 1 cœur de processeur et 5 Go de disque dur

```shell
sudo virt-install \
--osinfo alpinelinux3.17 \
--name alpine-vm \
--memory 2048 \
--vcpus 1 \
--cpu host \
--hvm \
--disk path=/srv/kvm/libvirt/images/alpine-vm.qcow2,format=qcow2,size=5 \
--cdrom /srv/kvm/libvirt/boot/alpine-standard-3.20.1-x86_64.iso \
--network bridge=br0 \
--graphics vnc  
```

Note: La dernière version Alpine Linux est alpinelinux3.20  au 10/07/2024 mais KVM ne connait que alpinelinux3.17 (`sudo virt-install --osinfo list |grep alpine`)

Après exécution dans un terminal de la commande ci dessus, on arrive sur l'écran suivant  
![](alpine-linux01.png)  

### Configurer vm-alpine

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

Relever l'adresse ip allouée : `ip a` --> 192.168.0.18  
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

    adduser aluser

Vous pouvez utiliser l'utilisateur pour vous connecter via SSH (impossible avec le compte root)  

### Connexion vm-alpine via SSH

Sur un poste linux du réseau

```bash
ssh -o ProxyCommand="ssh -W %h:%p -p 55205 -i /home/yann/.ssh/yick-ed25519 yick@192.168.0.205" aluser@192.168.100.60
```

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
        address 192.168.100.60/24
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

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/ttrss_alpine-vm

Copier la clé publique `cat ~/.ssh/ttrss_alpine-vm.pub` dans le presse-papier

	ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBQu0LRiDjPM7BJAfjECl22fnEchVuxnw38RV2IJs0+l yann@yann-pc1

On se connecte sur la machine virtuelle alpine linux "ttrss alpine-vm"

    ssh aluser@192.168.100.60

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
Port = 55217
PasswordAuthentication no
```

Relancer le serveur

    service sshd restart

Test connexion

```bash
ssh -o ProxyCommand="ssh -W %h:%p -p 55205 -i /home/yann/.ssh/yick-ed25519 yick@192.168.0.205" aluser@192.168.100.60 -p 55217 -i /home/yann/.ssh/ttrss_alpine-vm
```

### sudo

Passer en root

    su -

Editer la configuration des dépôts

    nano /etc/apk/repositories

Trouvez maintenant la ligne qui se termine dans /community Ensuite, retirez le # au début de la ligne. Le fichier résultant devrait ressembler à ceci

```
#/media/cdrom/apks
http://mirrors.ircam.fr/pub/alpine/v3.20/main
http://mirrors.ircam.fr/pub/alpine/v3.20/community
```

Mise à jour des dépôts

    apk update

Installer sudo

    apk add sudo

Pas de mot de passe sudo pour l'utilisateur alouest

    echo "aluser     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

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
http://mirrors.ircam.fr/pub/alpine/v3.20/main
http://mirrors.ircam.fr/pub/alpine/v3.20/community
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

## Tiny Tiny RSS

![](ttrss-logo-a.png)  

Vous aurez besoin des éléments suivants :

*    Un navigateur web moderne. Il s'agit généralement d'un navigateur récent de type Chrome ou compatible 
*   Un serveur (VDS ou VPS) exécutant Docker 

### Installation via Docker

Tiny Tiny RSS utilise un modèle de développement continu basé sur la [branche master de git](https://git.tt-rss.org/fox/tt-rss.git/) qui est considérée comme stable. Si possible, vous devriez toujours utiliser le dernier code de la branche master.

[Avec docker compose](https://tt-rss.org/wiki/InstallationNotes#docker-compose.yml)  
La seule façon supportée d'exécuter tt-rss est sous Docker. Vous pouvez utiliser les images officielles de Docker Hub (AMD64 uniquement).

Cette installation utilise PostgreSQL et exécute tt-rss en utilisant plusieurs conteneurs.

Les fichiers [environnement ".env" et "docker-compose.yml"](https://tt-rss.org/wiki/InstallationNotes#docker-compose.yml) à modifier  
Lier le port exposé à 127.0.0.1 pour fonctionner derrière un proxy inverse sur le même hôte. Si vous prévoyez d'exposer le conteneur, supprimez "127.0.0.1 :" dans le fichier .env

<details>
<summary><b>Etendre Réduire fichier .env</b></summary>
{% highlight text %}  
# .env
OWNER_UID=1000
OWNER_GID=1000
PHP_WORKER_MAX_CHILDREN=5
PHP_WORKER_MEMORY_LIMIT=256M
ADMIN_USER_PASS=Mot_de_passe_admin
ADMIN_USER_ACCESS_LEVEL=10
AUTO_CREATE_USER=ttrss
AUTO_CREATE_USER_PASS=Mot_de_passe_user
AUTO_CREATE_USER_ACCESS_LEVEL=10
TTRSS_DB_USER=postgres
TTRSS_DB_NAME=postgres
TTRSS_DB_PASS=Mot_de_passe_postgres
TTRSS_SELF_URL_PATH=https://rss.rnmkcy.eu
HTTP_PORT=192.168.100.60:8280
{% endhighlight %}
</details>

<details>
<summary><b>Etendre Réduire fichier docker-compose.yml</b></summary>
{% highlight yaml %}  
version: '3'

services:
  db:
    image: postgres:15-alpine
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - POSTGRES_USER=${TTRSS_DB_USER}
      - POSTGRES_PASSWORD=${TTRSS_DB_PASS}
      - POSTGRES_DB=${TTRSS_DB_NAME}
    volumes:
      - db:/var/lib/postgresql/data

  app:
    image: cthulhoo/ttrss-fpm-pgsql-static:latest
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - app:/var/www/
      - ./config.d:/opt/tt-rss/config.d:ro
    depends_on:
      - db

  updater:
    image: cthulhoo/ttrss-fpm-pgsql-static:latest
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - app:/var/www/
      - ./config.d:/opt/tt-rss/config.d:ro
    depends_on:
      - app
    command: /opt/tt-rss/updater.sh

  web-nginx:
    image: cthulhoo/ttrss-web-nginx:latest
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - ${HTTP_PORT}:80
    volumes:
      - app:/var/www/:ro
    depends_on:
      - app

volumes:
  db:
  app:
  backups:
{% endhighlight %}
</details>

Démarrez les conteneurs de services.

```bash
su -
cd /home/aluser
docker compose up -d
```

### Mise à jour conteneur

L'utilisation de Docker Compose simplifie le processus de gestion des conteneurs Docker, y compris leurs mises à jour. Si vous êtes déjà familiarisé avec le téléchargement et l'utilisation des conteneurs Docker

Conditions préalables

*    Persistance des données : un aspect crucial à retenir est la perte potentielle de données lors du processus de mise à jour du conteneur. Sauf si vous avez configuré des volumes persistants dans votre docker-compose.ymlfichier, la mise à jour de votre conteneur peut effacer les données qui lui sont associées. Essentiellement, cela reviendrait à reconstituer le conteneur.
*    Identification des volumes persistants : Dans votre fichier Docker Compose, vous pouvez identifier les volumes persistants avec la balise **volumes:**

Récupérer la dernière image  
accédez au répertoire de votre conteneur qui héberge le fichier Docker Compose

```bash
su -
cd /home/aluser
docker compose pull
```

![](docker-compose1.png)

Docker relayera une sortie indiquant s'il extrait l'image de conteneur la plus récente.

Déployer le conteneur mis à jour  
avec la dernière image en remorque, déployez le conteneur en utilisant :

    docker compose up -d

![](docker-compose2.png)

### Mise à jour automatique

*Les stratégies pour automatiser le processus de mise à jour des conteneurs Docker lorsque leurs images de base sont mises à jour. Nous examinerons des scripts Bash personnalisés et des outils tiers tels que Watchtower, qui offrent un éventail de solutions([Auto-Update Docker Containers for Latest Base Images](https://www.baeldung.com/ops/docker-container-auto-update-newest-base-images))* 

Watchtower est un outil open-source qui automatise le processus de mise à jour des conteneurs Docker. Il interroge le registre Docker pour vérifier les mises à jour des images à partir desquelles les conteneurs ont été initialement instanciés.  
Supposons que Watchtower détecte qu'une image a été mise à jour. Dans ce cas, elle arrête le conteneur exécutant l'image périmée, extrait la nouvelle image du registre Docker et démarre un nouveau conteneur avec les mêmes configurations que le précédent. Cela garantit que nos conteneurs exécutent toujours la dernière version de l'image de base sans intervention manuelle.

**Exécution du conteneur Watchtower**  
Paramètres

* fixer l'intervalle à 86400 secondes, ce qui demande à Watchtower de vérifier les mises à jour une fois par jour  
Le paramètre `--schedule` prend comme valeur une expression crontab. Sinon, le paramètre `--interval` peut-être utilisé pour définir le nombre de secondes entre chaque vérification.  
`--schedule "0 0 4 * * *"`  
`--interval 86400`  
Cela permet de trouver un équilibre entre l'immédiateté et la consommation des ressources du système.

Dans un autre dossier en mode su

```shell
mkdir maj
cd maj
```

Une autre personnalisation de l'image au format docker-compose : docker-compose.yml

```yaml
version: '2.1'
services:
  watchtower:
    image: containrrr/watchtower:latest
    container_name: watchtower
    restart: unless-stopped
    environment:
      - WATCHTOWER_SCHEDULE=0 55 15 * * *
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
time="2025-06-05T15:53:31+02:00" level=debug msg="Sleeping for a second to ensure the docker api client has been properly initialized."
time="2025-06-05T15:53:32+02:00" level=debug msg="Making sure everything is sane before starting"
time="2025-06-05T15:53:32+02:00" level=debug msg="Retrieving running and restarting containers"
time="2025-06-05T15:53:32+02:00" level=debug msg="There are no additional watchtower containers"
time="2025-06-05T15:53:32+02:00" level=debug msg="Watchtower HTTP API skipped."
time="2025-06-05T15:53:32+02:00" level=info msg="Watchtower 1.7.1"
time="2025-06-05T15:53:32+02:00" level=info msg="Using no notifications"
time="2025-06-05T15:53:32+02:00" level=info msg="Checking all containers (except explicitly disabled with label)"
time="2025-06-05T15:53:32+02:00" level=info msg="Scheduling first run: 2025-06-05 15:55:00 +0200 CEST"
time="2025-06-05T15:53:32+02:00" level=info msg="Note that the first check will be performed in 1 minute, 27 seconds"
```

### Problème mise à jour auto

Liste

```
localhost:~$ sudo docker ps -a
CONTAINER ID   IMAGE                                    COMMAND                  CREATED        STATUS                    PORTS                         NAMES
9eeb44099835   cthulhoo/ttrss-fpm-pgsql-static:latest   "/opt/tt-rss/updater…"   11 days ago    Up 9 minutes              9000/tcp                      aluser-updater-1
c82e5ecc13b7   cthulhoo/ttrss-web-nginx:latest          "/docker-entrypoint.…"   11 days ago    Up 9 minutes (healthy)    192.168.100.60:8280->80/tcp   aluser-web-nginx-1
c076070d0371   cthulhoo/ttrss-fpm-pgsql-static:latest   "/bin/sh -c ${SCRIPT…"   11 days ago    Up 9 minutes              9000/tcp                      aluser-app-1
bcb0f4224ab7   postgres:15-alpine                       "docker-entrypoint.s…"   11 days ago    Up 9 minutes              5432/tcp                      aluser-db-1
e2c0f065cecf   containrrr/watchtower                    "/watchtower --sched…"   3 months ago   Exited (1) 3 months ago                                 watchtower
```

On arrête watchtower

    sudo docker stop e2c0f065cecf

Ménage

    sudo docker system prune  

```
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - unused build cache

Are you sure you want to continue? [y/N] y
```

Les images

    sudo docker images

```
REPOSITORY                        TAG         IMAGE ID       CREATED         SIZE
postgres                          15-alpine   f2404745c576   2 weeks ago     273MB
cthulhoo/ttrss-fpm-pgsql-static   latest      68552a20255f   3 weeks ago     138MB
cthulhoo/ttrss-web-nginx          latest      70e727fdd726   3 weeks ago     47MB
cthulhoo/ttrss-fpm-pgsql-static   <none>      5b517b2f6e7e   5 weeks ago     138MB
cthulhoo/ttrss-web-nginx          <none>      63da1ae18ad7   5 weeks ago     52.5MB
cthulhoo/ttrss-fpm-pgsql-static   <none>      b8d04eb80723   2 months ago    138MB
cthulhoo/ttrss-web-nginx          <none>      29802f8f0c5c   2 months ago    52.5MB
postgres                          <none>      28223f2e117a   2 months ago    273MB
postgres                          <none>      701535775237   2 months ago    273MB
cthulhoo/ttrss-fpm-pgsql-static   <none>      734e23fda5f2   3 months ago    133MB
cthulhoo/ttrss-web-nginx          <none>      ad6dd79a036d   3 months ago    47MB
cthulhoo/ttrss-fpm-pgsql-static   <none>      3810e5d94f06   3 months ago    133MB
cthulhoo/ttrss-web-nginx          <none>      26047372a8aa   3 months ago    47MB
cthulhoo/ttrss-fpm-pgsql-static   <none>      70a68572f136   3 months ago    133MB
cthulhoo/ttrss-web-nginx          <none>      b0bbfed37acf   3 months ago    47MB
cthulhoo/ttrss-fpm-pgsql-static   <none>      3d2f88c7fcdc   3 months ago    133MB
cthulhoo/ttrss-web-nginx          <none>      f8e6d0beed42   3 months ago    47MB
postgres                          <none>      0b42deb40e16   6 months ago    248MB
postgres                          <none>      fc9156a9e8b8   6 months ago    243MB
cthulhoo/ttrss-fpm-pgsql-static   <none>      ad195f5ddd71   7 months ago    136MB
cthulhoo/ttrss-web-nginx          <none>      875dde04b88a   7 months ago    43.2MB
cthulhoo/ttrss-fpm-pgsql-static   <none>      07a86abf61c3   7 months ago    136MB
cthulhoo/ttrss-web-nginx          <none>      0b99e1b47f98   7 months ago    43.2MB
postgres                          <none>      f506c9c47418   8 months ago    243MB
postgres                          <none>      00b2bb697bff   8 months ago    243MB
containrrr/watchtower             latest      e7dd50d07b86   15 months ago   14.7MB
```

Suppression image containrrr/watchtower

    sudo docker rmi -f e7dd50d07b86

Réinstaller watchtower

```bash
sudo docker run \
  --name watchtower \
  --volume /var/run/docker.sock:/var/run/docker.sock \
  --detach \
  containrrr/watchtower \
  --schedule "0 0 3 * * *" \
  --cleanup
```

## Serveur hôte rnmkcy.eu

### Proxy nginx rss.rnmkcy.eu

Sur le serveur Lenovo rnmkcy.eu, créer un proxy pour ttrss `/etc/nginx/conf.d/rss.rnmkcy.eu.conf`

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name  rss.rnmkcy.eu;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://rss.rnmkcy.eu$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  rss.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;

   location / {
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header X-Forwarded-Proto $scheme;
      
      proxy_pass http://192.168.100.60:8280/;
      break;
   }

}
```

Recharger nginx

    sudo systemctl reload nginx

Ouvrir le lien <htpps://rss.rnmkcy.eu> et saisir le mot de passe administrateur  

Dans la configuration utilisateur **ttrss**, cocher **Activer l'API**  
![](alpine-linux07.png){:width="300"}  

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
-H "Title: 💻 Alpine Linux ttrss alpine-vm : Fin exécution script 'apk-autoupgrade'" \
-H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
-H prio:low \
-d "Flux RSS ttrss 192.168.100.60 alpine-vm 
✔️ Fin exécution script /etc/periodic/daily/apk-autoupgrade" \
https://noti.rnmkcy.eu/yan_infos
" >> /etc/periodic/daily/apk-autoupgrade
```
