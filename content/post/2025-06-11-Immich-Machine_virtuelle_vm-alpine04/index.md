+++
title = 'cwwk KVM Alpine (vm-alpine04) - Immich (photos et vidéos)'
date = 2025-06-11 00:10:00
categories = ['virtuel']
+++
*immich - Gestionnaire web photos et vidéos installé sur une machine virtuelle Alpine Linux*

![](alpine-linux-logo.png){:width="300" .left}  
*Alpine Linux est une distribution Linux ultra-légère*

## Alpine Linux

*Création machine virtuelle Alpine  de type KVM avec 4 Go de RAM, 2 cœur de processeur et 5 Go de disque dur.*

* [OpenRC](https://wiki.alpinelinux.org/wiki/OpenRC)
* [Install docker & docker-compose on Alpine Linux](https://geekscircuit.com/install-docker-docker-compose-on-alpine-linux/)
* [10 Alpine Linux apk Command Examples](https://www.cyberciti.biz/faq/10-alpine-linux-apk-command-examples/)

### Créer vm-alpine sur un serveur

[Les dernières images Alpine Linux](https://alpinelinux.org/downloads/)  

Création d'une image virtuelle **vm-alpine04** sous le serveur Lenovo rnmkcy.eu  
On se connecte sur le serveur Lenovo en SSH, puis on exécute la commande suivante pour créer  une machine virtuelle Alpine avec 2 Go de RAM, 1 cœur de processeur et 5 Go de disque dur

```shell
sudo virt-install \
--osinfo alpinelinux3.17 \
--name vm-alpine04 \
--memory 4096 \
--vcpus 2 \
--cpu host \
--hvm \
--disk path=/srv/kvm/libvirt/images/vm-alpine04.qcow2,format=qcow2,size=5 \
--cdrom /home/yick/FreeUSB2To/iso/alpine-standard-3.22.0-x86_64.iso \
--network bridge=br0 \
--graphics vnc  
```

Note:KVM ne connait que alpinelinux3.17 (`sudo virt-install --osinfo list |grep alpine`)

Après exécution dans un terminal de la commande ci dessus, on arrive sur *En attente de fin d'installation*

### Configurer vm-alpine

`ATTENTION: Désactiver "Affichage VNC" des autres machines`{: .prompt-warning }

VNC, configuration xml

```xml
<graphics type="vnc" port="5900" autoport="yes" listen="127.0.0.1">
  <listen type="address" address="127.0.0.1"/>
</graphics>
```

Le serveur Lenovo n'a pas d'affichage, il faut créer un tunnel ssh depuis un terminal d'un poste client

    ssh -L 5900:127.0.0.1:5900 yick@192.168.0.205 -p 55205 -i /home/yann/.ssh/yick-ed25519

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
mot de passe root (rtyuiop)  
![](alpine-linux05.png)  
APK mirror...  
![](alpine-linux05a.png)  

Utilisateur alpi/alpi49 et suite...   
![](alpine-linux06.png)  

alp04/alprtyu  
Relever l'adresse ip allouée : `ip a` --> 192.168.10.129   
Puis redémarrer : `reboot`  
La fenêtre vnc se ferme  

On n'a plus besoin de VNC, en mode graphique   
![](alpine-linux06a.png){:width="400"}  
![](alpine-linux06b.png){:width="400"}  
![](alpine-linux06c.png){:width="400"}  
Eteindre puis redémarrer la machine virtuelle 

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

    adduser alpi

Vous pouvez utiliser l'utilisateur pour vous connecter via SSH (impossible avec le compte root)  

### Connexion vm-alpine via SSH

Sur un poste linux du réseau

    ssh alp04@192.168.10.129

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
  address 192.168.10.214
  gateway 192.168.10.1
```

Fichier de résolution dns

    /etc/resolv.conf

```
nameserver 192.168.0.205
nameserver 192.168.10.1
```

Les modifications apportées à /etc/network/interfaces peuvent être activées en exécutant

```shell
service networking restart 
```

`ATTENTION: Déconnexion SSH car changement adresse IP`{: .prompt-warning }

Connexion SSH avec IP 192.168.10.214

    ssh alp04@192.168.10.214

Message à la connexion SSH, `/etc/motd`

```
    _     _         _                _      _                            
   / \   | | _ __  (_) _ __    ___  | |    (_) _ __   _   _ __  __       
  / _ \  | || '_ \ | || '_ \  / _ \ | |    | || '_ \ | | | |\ \/ /       
 / ___ \ | || |_) || || | | ||  __/ | |___ | || | | || |_| | >  <        
/_/   \_\|_|| .__/ |_||_| |_| \___| |_____||_||_| |_| \__,_|/_/\_\       
            |_|                  _         _                ___   _  _   
__   __ _ __ ___           __ _ | | _ __  (_) _ __    ___  / _ \ | || |  
\ \ / /| '_ ` _ \  _____  / _` || || '_ \ | || '_ \  / _ \| | | || || |_ 
 \ V / | | | | | ||_____|| (_| || || |_) || || | | ||  __/| |_| ||__   _|
  \_/  |_| |_| |_|        \__,_||_|| .__/ |_||_| |_| \___| \___/    |_|  
 _   ___  ____      _   __     ___ |_| _   ___     ____   _  _  _        
/ | / _ \|___ \    / | / /_   ( _ )   / | / _ \   |___ \ / || || |       
| || (_) | __) |   | || '_ \  / _ \   | || | | |    __) || || || |_      
| | \__, |/ __/  _ | || (_) || (_) |_ | || |_| |_  / __/ | ||__   _|     
|_|   /_/|_____|(_)|_| \___/  \___/(_)|_| \___/(_)|_____||_|   |_|       

```

### OpenSSH avec clés

*Connexion ssh sur un autre port avec un jeu de clés*

Générer une paire de clé sur l'ordinateur de bureau PC1  
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec la machine virtuelle vm-alpine04

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/vm-alpine04
    chmod 600 ~/.ssh/vm-alpine04

Copier la clé publique `cat ~/.ssh/vm-alpine04.pub` dans le presse-papier

	ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGVKrZfQL2t0304ELYtXGlFv5jHv7tSxcBOcyxGBfgTE yann@PC1

On se connecte sur la machine virtuelle alpine linux "vm-alpine04"

```shell
ssh alp04@192.168.10.214
```

Créer le répertoire et ouvrir nouveau fichier

    mkdir -p $HOME/.ssh/
    nano $HOME/.ssh/authorized_keys

Coller le contenu du presse-papier , sauver le fichier et sortir

Modifier les droits

```shell
chmod 600 $HOME/.ssh/authorized_keys
```

Passer en mode su

    su -

Modifier la configuration serveur SSH

    nano /etc/ssh/sshd_config

Modifier

```
Port = 55214
PasswordAuthentication no
```

Relancer le serveur

    service sshd restart

Test connexion

```bash
ssh alp04@192.168.10.214 -p 55214 -i /home/yann/.ssh/vm-alpine04
```

### sudo

Passer en root

    su -

**Ajout dépôt communauté**  
Editer la configuration des dépôts

    nano /etc/apk/repositories

Trouvez maintenant la ligne qui se termine dans **/community**  
Ensuite, retirez le `#` au début de la ligne.  
Le fichier résultant devrait ressembler à ceci

```
#/media/cdrom/apks
http://dl-cdn.alpinelinux.org/alpine/v3.22/main
http://dl-cdn.alpinelinux.org/alpine/v3.22/community
```

Installer sudo 

```bash
apk update
apk add sudo
```

Ajouter un utilisateur avec les privlèges root

```bash
echo "alp04     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/20-alpi
```

### Démarrage automatique VM

Démarrage auto  
![](virtiofs02a.png){:width="500"}  

### NFS

Cette page documente la configuration du système de fichiers réseau (NFS) du côté du serveur et du client, y compris les autofs et l'authentification Kerberos.
Installation

Installez le paquet suivant pour le service client et serveur NFS.

```shell
apk add nfs-utils
```

Client NFS

Pour monter automatiquement les actions NFS, une entrée doit être faite à `/etc/fstab` comme suit:

Contenu de /etc/fstab

```shell
192.168.0.205:/sharenfs /home/alp04/sharenfs nfs4 rw,_netdev 0 0
```

Pour monter nfs share depuis le fichier /etc/fstab au démarrage du système :

```shell
rc-update add nfsmount
```

Pour monter les actions de nfs depuis le fichier /etc/fstab :

```shell
rc-service nfsmount start
```

Vous pouvez vérifier vos services de démarrage :

```shell
rc-status
```

Conseil : netmount est un service général pour tous les systèmes de fichiers basés sur le réseau, tandis que nfsmount est spécialement conçu pour NFS.

Pour utiliser netmount, voici les commandes équivalentes :

```
# rc-service netmount démarrage
# rc-update add netmount
```

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

Le script exécute la commande `apk upgrade --update` une fois par jour, apk par défaut ne demande jamais l’intervention de l’utilisateur. 

Additif pour notification ntfy

```shell
echo "#
curl \
-H "X-Email: ntfy@cinay.eu" \
-H "Title: 💻 Alpine Linux vm-alpine04 : Fin exécution script 'apk-autoupgrade'" \
-H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
-H prio:low \
-d "vm-alpine04 192.168.10.214 
✔️ Fin exécution script /etc/periodic/daily/apk-autoupgrade" \
https://noti.rnmkcy.eu/yan_infos
" >> /etc/periodic/daily/apk-autoupgrade
```

### Augmenter taille disque

```
yick@cwwk:~$ sudo virsh shutdown vm-alpine04
yick@cwwk:~$ sudo virsh list
 ID   Nom            État
-------------------------------------------
 1    vm-ntfy        en cours d’exécution
 2    vm-alpine01    en cours d’exécution
 3    vm-debian12    en cours d’exécution
 4    alpine-vm      en cours d’exécution
 5    vm-alpine03    en cours d’exécution
 6    alpine-searx   en cours d’exécution

yick@cwwk:~$ sudo virsh domblklist vm-alpine04
 Target   Source
-----------------------------------------------------
 vda      /srv/kvm/libvirt/images/vm-alpine04.qcow2
 sda      -

yick@cwwk:~$ sudo qemu-img resize /home/yann/virtuel/KVM/vm-alpine04.qcow2 +15G
qemu-img: Could not open '/home/yann/virtuel/KVM/vm-alpine04.qcow2': Could not open '/home/yann/virtuel/KVM/vm-alpine04.qcow2': No such file or directory
yick@cwwk:~$ sudo qemu-img resize /srv/kvm/libvirt/images/vm-alpine04.qcow2 +15G
Image resized.
yick@cwwk:~$ sudo qemu-img info /srv/kvm/libvirt/images/vm-alpine04.qcow2
image: /srv/kvm/libvirt/images/vm-alpine04.qcow2
file format: qcow2
virtual size: 20 GiB (21474836480 bytes)
disk size: 3.41 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: true
    refcount bits: 16
    corrupt: false
    extended l2: false
yick@cwwk:~$ sudo virsh start vm-alpine04
Domain 'vm-alpine04' started

# Connexion à la machine vm-alpine04

# Outils
sudo apk add --no-cache cfdisk e2fsprogs-extra

# cfdisk pour redimensionner la partition vda3
localhost:~$ sudo cfdisk

sélectionner la partition à étendre puis resize write quit

Syncing disks.

localhost:~$ sudo resize2fs /dev/vda3
resize2fs 1.47.2 (1-Jan-2025)
Filesystem at /dev/vda3 is mounted on /; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 3
The filesystem on /dev/vda3 is now 4838144 (4k) blocks long.

localhost:~$ sudo df -H |grep vda3
/dev/vda3                18.1G      1.3G     16.0G   8% /
/dev/vda3                18.1G      1.3G     16.0G   8% /var/lib/docker

```

## Docker

![](docker-logo-a.png)  

### Installation

Passer en root

    su -

**Dépôt communauté actif**  


**Installation docker**  
Installer docker et docker-compose

```bash
apk update
apk add docker docker-compose
```

Activer autostart sur boot en utilisant

```bash
rc-update add docker default
# suppression
# rc-service docker stop
# rc-update del docker default
```

puis vous pouvez lancer le service docker en utilisant la commande

```bash
/etc/init.d/docker start
# ou
service docker start
```

## Immich

![](immich-logo.png){:width="100" .left}
*Immich est une application qui permet de télécharger et d'afficher des actifs (photos et vidéos) sur un serveur Immich auto-hébergé. L'application offre des fonctionnalités de sauvegarde automatique, de marquage, de recherche, de géocodage et de détection d'objet.*

Exigences

*Un système avec au moins 4 Go de RAM et 2 cœurs CPU.*

Voici quelques-unes des fonctionnalités clés qui en font une solution intéressante :

*    **Lecteur photo et vidéo** : Prise en charge les formats de fichiers courants, y compris les fichiers RAW pour les photos
*    **Données Exif** : Les informations Exif (Exchangeable Image File Format) sont utilisées pour fournir des détails supplémentaires aux photos, comme la date de prise de vue, le modèle de l’appareil photo…
*    **Vue carte** : Permet de visualiser où les photos ont été prises sur une carte, cela ajoute une dimension géographique intéressante
*    **Détection des visages** : Immich utilise des algorithmes de reconnaissance faciale pour organiser et retrouver plus facilement des photos de personnes
*    **Gestion par Dossier et Favoris** : Organisez vos photos comme vous le souhaitez et marquez les photos préférées pour un accès plus rapide
*    **Portail d’administration** : Gestion des paramètres de l’application et des utilisateurs
*    **Partage via des liens** : Partagez vos photos et vidéos facilement avec des liens
*    **Moteur de recherche** : Facilite la recherche des photos grâce à des options avancées
*    **Vue 360°** : Profitez d’une vue immersive de vos photos panoramiques
*    **Accélération matérielle (GPU)** : Optimisez la performance de traitement des images et vidéos grâce à l’accélération matérielle
*    **Édition rapide, mode sombre, authentification viaOAuth2 et OIDC**, etc.

Particularités

*    son ergonomie, sobre et efficace, agréable à utiliser
*    les options de tri « intelligentes » pour classer automatiquement les photos/vidéos qui permettent de les retrouver très facilement ( lieu,détection des visages, tags personnalisés etc)
*    les vidéos super fluides, avec un aperçu au survol de la souris digne des perfs d’un Netflix
*    la possibilité de synchroniser les photos de son smartphone
*    autohébergeable, tu peux l’installer sur ton serveur ou sur ton pc en local avec docker
*    une appli conçue et orientée « respect de la vie privée »

### Télécharger les fichiers requis

Créez un répertoire de votre choix (par exemple ./immich-app) pour tenir les fichiers `docker-compose.yml` et `.env`.  
Déplacer vers le répertoire que vous avez créé

```shell
mkdir ./immich-app
cd ./immich-app
```

Télécharger `docker-compose.yml` et `example.env` en exécutant les commandes suivantes:

```shell
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

### Personnaliser le fichier .env

```
# You can find documentation for all the supported env variables at https://immich.app/docs/install/environment-variables

# The location where your uploaded files are stored
UPLOAD_LOCATION=./library

# The location where your database files are stored. Network shares are not supported for the database
DB_DATA_LOCATION=./postgres

# To set a timezone, uncomment the next line and change Etc/UTC to a TZ identifier from this list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
TZ=Europe/Paris

# The Immich version to use. You can pin this to a specific version like "v1.71.0"
IMMICH_VERSION=release

# Connection secret for postgres. You should change it to a random password
# Please use only the characters `A-Za-z0-9`, without special characters or spaces
DB_PASSWORD=postgres

# The values below this line do not need to be changed
###################################################################################
DB_USERNAME=postgres
DB_DATABASE_NAME=immich
```

* **UPLOAD_LOCATION** avec votre emplacement préféré pour stocker des actifs de sauvegarde. Ce devrait être un nouveau répertoire sur le serveur avec suffisamment d'espace libre.
* Envisagez de changer **DB_PASSWORD** en une valeur personnalisée. Postgres n'est pas publiquement exposé, donc ce mot de passe n'est utilisé que pour l'authentification locale. Pour éviter les problèmes avec l'analyse de cette valeur par Docker, il est préférable d'utiliser uniquement les caractères A-Za-z0-9. `pwgen` est un utilitaire pratique pour cela.
* Définir le fuseau horaire en décommentant la ligne **TZ=**.

### Démarrer les conteneurs

À partir du répertoire que vous avez créé à l'étape 1 (qui devrait maintenant contenir vos fichiers `docker-compose.yml` et `.env` personnalisés), exécutez la commande suivante pour démarrer **Immich** comme un service de fond :

```shell
cd ~/immich-app
sudo docker compose up -d
```

Résultat commande 

```
[...] 
[+] Running 6/6
 ✔ Network immich_default             Created                                              0.0s 
 ✔ Volume "immich_model-cache"        Created                                              0.0s 
 ✔ Container immich_machine_learning  Started                                              0.8s 
 ✔ Container immich_redis             Started                                              0.8s 
 ✔ Container immich_postgres          Started                                              0.8s 
 ✔ Container immich_server            Started                                              0.7s 
```

### Essayez l'application web

Le <u>premier utilisateur</u> à s'enregistrer sera l'<u>utilisateur administrateur</u>.  
L'utilisateur administrateur pourra ajouter d'autres utilisateurs à l'application.

Pour s'inscrire à l'utilisateur administrateur, accéder à l'application Web à <http://192.168.10.214:2283> et cliquez sur le bouton Démarrer.

![](immich01.png)  

![](immich02.png)  

![](immich03.png)  


### Proxy nginx immich.rnmkcy.eu

Configurer le fichier `/etc/nginx/conf.d/immich.rnmkcy.eu.conf` sur serveur debian cwwk rnmkcy.eu

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name  immich.rnmkcy.eu;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://immich.rnmkcy.eu$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  immich.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;

    # enable websockets: http://nginx.org/en/docs/http/websocket.html
    proxy_http_version 1.1;
    proxy_set_header   Upgrade    $http_upgrade;
    proxy_set_header   Connection "upgrade";
    proxy_redirect     off;

    # set timeout
    proxy_read_timeout 600s;
    proxy_send_timeout 600s;
    send_timeout       600s;

    location / {
        proxy_pass http://192.168.10.214:2283;
    }

}
```

### Bibliothèque externe

Ce guide vous accompagne dans l'ajout d'une bibliothèque externe. Ce guide suppose que vous exécutez Immich dans Docker et que les fichiers auxquels vous souhaitez accéder sont stockés dans un répertoire sur la même machine.[(External Library)](https://immich.app/docs/guides/external-library/)

Les dossier externes `/home/alp04/sharenfs/rnmkcy/immich-yannick` et `/home/alp04/sharenfs/rnmkcy/immich-claudine`

Modifier le fichier docker-compose.yml  
Ajout de la ligne `- /home/alp04/sharenfs/rnmkcy/immich-yannick:/mnt/photos1:ro`  
Dossier local: `/home/alp04/sharenfs/rnmkcy/immich`  
Dossier container: `/mnt/photos1`
Ajout de la ligne `- /home/alp04/sharenfs/rnmkcy/immich-claudine:/mnt/photos2:ro`  
Dossier local: `/home/alp04/sharenfs/rnmkcy/immich`  
Dossier container: `/mnt/photos2`

```yaml
    volumes:
      # Do not edit the next line. If you want to change the media storage location on your system, edit the value of UPLOAD_LOCATION in the .env file
      - ${UPLOAD_LOCATION}:/usr/src/app/upload
      - /home/alp04/sharenfs/rnmkcy/immich-yannick:/mnt/photos1:ro
      - /home/alp04/sharenfs/rnmkcy/immich-claudine:/mnt/photos2:ro
      - /etc/localtime:/etc/localtime:ro
```

Mise à jour

```shell
docker compose up -d
```

Se connecter en administrateur

![](immich05.png){: .normal}   

![](immich06.png){: .normal}   

![](immich07.png){: .normal}   

![](immich08.png){: .normal}   

![](immich09.png){: .normal}   

![](immich10.png){: .normal}   
Cliquez sur le menu à trois points et sélectionnez "Analyser"

Cliquer sur "Tâches"  
![](immich10a.png){: .normal}   

Après quelques minutes, vous devriez voir `En cours 0 et En attente 0` pour **GÉNÉRATION DES MINIATURES**, **EXTRACTION DES MÉTADONNÉES** et **BIBLIOTHÈQUES EXTERNES**.  
![](immich11.png){: .normal}   

## Configuration SSO pour Immich avec Authelia OIDC

*Dans cet article de blog, nous allons configurer Authelia comme fournisseur OIDC et l'utiliser pour activer l'authentification unique (SSO) dans Immich*
⭐

* [Immich - OAuth Authentication](https://immich.app/docs/administration/oauth/)
* [Immich - Authelia](https://www.authelia.com/integration/openid-connect/immich/)
    * [Configuring SSO for Immich with Authelia OIDC](https://blog.lrvt.de/configuring-authelia-oidc-for-immich/)

### cwwk - Configuration Authelia

La configuration Authelia est définie dans un fichier de configuration.yml. Ce fichier permet de définir les paramètres de configuration OIDC nécessaires, tels qu'un fournisseur et un client.

En consultant la documentation publique OIDC d'Authelia , nous pouvons obtenir un exemple de configuration. Cet exemple de configuration contient plusieurs paramètres nécessitant soit une valeur secrète, soit un certificat.

Ces éléments doivent être générés de manière sécurisée avant d'ajuster la configuration. Par conséquent, générons-les !

**Création d'un certificat RSA**

Authelia vous oblige à définir un certificat (clé privée) via la directive  key et une chaîne de certificats (clé publique) via la directive  certificate_chain.

Vous pouvez générer les deux via la commande suivante :

```shell
authelia crypto certificate rsa generate --common-name auth.rnmkcy.eu && cat public.crt && cat private.pem
```

⚠️ Veuillez ajuster le nom commun du certificat à votre FQDN d'Authelia

**Création d'un secret client**

Authelia vous demande de définir un secret client sécurisé.

Vous pouvez en générer un via la commande suivante :

```shell
authelia crypto hash generate pbkdf2 --variant sha512 --random --random.length 72 --random.charset rfc3986
```

⚠️ Veuillez noter le mot de passe en clair ainsi que le hachage  $pbkdf2  du mot de passe pour une utilisation ultérieure.

**Création d'un fournisseur et d'un client OIDC**

Une fois le certificat RSA et le secret client créés de manière sécurisée, nous pouvons commencer à ajuster le fichier de configuration d'Authelia. Nous ajouterons un fournisseur d'identité et un client en insérant nos secrets et informations de certificat auto-générés.

Mettez ce qui suit à la fin de votre configuration Authelia :

```yaml
identity_providers:
  oidc:
    hmac_secret: 'my-secure-hmac-secret-key-to-change'
    jwks:
      - key_id: 'authelia'
        algorithm: 'RS256'
        use: 'sig'
        certificate_chain: |
          -----BEGIN CERTIFICATE-----
          <PASTE-HERE-YOUR-PUBLIC-KEY-DATA> sur une ligne
          -----END CERTIFICATE-----
        key: |
          -----BEGIN PRIVATE KEY-----
          <PASTE-HERE-YOUR-PRIVATE-KEY-DATA> sur une ligne
          -----END PRIVATE KEY-----
    enable_client_debug_messages: false
    minimum_parameter_entropy: 8
    enforce_pkce: 'public_clients_only'
    enable_pkce_plain_challenge: false
    enable_jwt_access_token_stateless_introspection: false
    discovery_signed_response_alg: 'none'
    discovery_signed_response_key_id: ''
    require_pushed_authorization_requests: false
    lifespans:
      access_token: '1h'
      authorize_code: '1m'
      id_token: '1h'
      refresh_token: '90m'
    cors:
      endpoints:
        - 'authorization'
        - 'token'
        - 'revocation'
        - 'introspection'
      allowed_origins:
        - 'https://immich.rnmkcy.eu'
      allowed_origins_from_client_redirect_uris: false
    clients:
      - client_id: immich
        client_name: Immich OIDC
        client_secret: '$pbkdf2-<PASSWORD-HASH-DATA>'
        public: false
        authorization_policy: 'two_factor'
        redirect_uris:
          - 'https://immich.rnmkcy.eu/auth/login'
          - 'https://immich.rnmkcy.eu/user-settings'
          - 'app.immich:///oauth-callback'
        scopes:
          - 'openid'
          - 'profile'
          - 'email'
        userinfo_signed_response_alg: 'none'
        token_endpoint_auth_method: 'client_secret_post'
```

🛑 Assurez-vous de copier-coller correctement les informations du certificat. La clé publique doit être définie dans certificate_chain. La clé privée doit être définie dans key.

Remplacez toutes les entrées **rnmkcy.eu** par celles de vos domaines FQDN.

Assurez-vous d’ajuster correctement les paramètres clés suivants à votre environnement :

*    hmac_secret
*    certificate_chain
*    key
*    allowed_origins
*    client_secret
     *   Veuillez définir le hachage `$pbkdf2` du mot de passe et non le mot de passe en clair
*    redirect_uris

Après avoir ajusté la configuration d'Authelia, vérifiez et redémarrez-le pour appliquer la nouvelle configuration. Inspectez activement les journaux des conteneurs pour détecter d'éventuels problèmes.

```shell
# vérification
authelia validate-config --config /etc/authelia/configuration.yml
# relancer
systemctl restart authelia
```

### Immich - Configuration OAuth

*La configuration OAuth Immich peut être ajustée en vous connectant à l'application web en tant qu'administrateur.*

Accédez à votre instance web Immich et connectez-vous en tant qu'administrateur. Ensuite, parcourez l'URL /administration/Paramètres. Vous trouverez un paramètre nommé **OAuth** Authentication.

![](immich12.png){: .normal}   

![](immich13.png){: .normal}   

![](immich14.png){: .normal}   

Voici une brève explication des paramètres clés :

*    **ISSUER URL**: il s'agit du nom de domaine complet (FQDN) de votre instance Authelia
*    **CLIENT ID**: Il s'agit du nom du client défini dans le fichier configuration.yml d'Authelia
*    **CLIENT SECRET**:  Il s'agit du secret client sous forme de mot de passe en clair. Défini dans le fichier configuration.yml d'Authelia comme $pbkdf2hachage du mot de passe.
*    **SCOPE**: Peut être laissé inchangé
*    **SIGNING ALGORITHM**: L'algorithme de signature à utiliser est RSA256 ou HS256. Pour Authelia, veuillez choisir RS256.
*    **STORAGE LABEL CLAIN**: Peut être laissé inchangé
*    **STORAGE QUOTA CLAIM**: La limite de quota
*    **BUTTON TEXT**: Votre texte de bouton préféré affiché sur la page de connexion
*    **AUTO REGISTER**: Votre choix : les utilisateurs Authelia sont-ils automatiquement enregistrés/créés dans Immich s'ils n'existent pas encore ? Je recommande de définir ce paramètre sur « False ». Créez les utilisateurs manuellement !
*    **AUTO LAUNCH**: Démarrage automatique du flux d'authentification OAuth/OIDC. Si cette option est activée, l'option d'authentification locale avec nom d'utilisateur et mot de passe ne sera plus disponible.
*    **MOBILE REDIRECT URI OVERRIDE**:  doit être activé pour prendre en charge les redirections appropriées depuis la connexion du navigateur Authelia vers l'application mobile Immich.
*    **MOBILE REDIRECT URI**: Il s'agit d'une URL spécifique fournie par Immich, qui ouvrira l'application mobile Immich si elle est ouverte. Il s'agit d'un schéma personnalisé ou d'un lien profond ; voir ici . Il se compose de votre nom de domaine complet Immich et du chemin d'accès `/api/oauth/mobile-redirect`.

Ensuite, vous pouvez cliquer sur le bouton **Enregistrer** et tester le SSO OIDC nouvellement configuré.

> Les applications web et mobile d'Immich afficheront un nouveau bouton pour se connecter via OAuth/OIDC. Cliquez simplement sur ce bouton, authentifiez-vous sur Authelia et vous serez redirigé vers Immich en tant qu'utilisateur authentifié.
{: .prompt-tip }

![](immich15.png){: .normal}   

![](immich16.png){: .normal}   

![](immich17.png){: .normal}   

![](immich18.png){: .normal}   

## Docker Immich cli  

### Docker cli

Arrêter et supprimer tous les containers

```shell
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

Suppression de toutes les images Docker  
Si vous devez supprimer toutes les images de votre système, utilisez la commande suivante :

```shell
docker rmi $(docker images -q)
```

**Volumes**: `docker volume ls`

```
DRIVER    VOLUME NAME
local     6759a084ed614d62f55591939193e70040dea6c63a9ff363e5ff71218f9d2e71
local     772714802d90e4f60d007649df17e40552a000308d2b26ba00156460648f95f6
local     immich_model-cache
```

Supprimer les volumes

```shell
docker volume rm immich_model-cache
docker volume rm 6759a084ed614d62f55591939193e70040dea6c63a9ff363e5ff71218f9d2e71
docker volume rm 772714802d90e4f60d007649df17e40552a000308d2b26ba00156460648f95f6
```

**Réseaux**: `docker network ls`

```
NETWORK ID     NAME             DRIVER    SCOPE
edfc6860b686   bridge           bridge    local
4704cd997213   host             host      local
4f59432079d3   immich_default   bridge    local
d57684d7dba7   none             null      local
```

Pour supprimer un réseau Docker spécifique, utilisez la commande docker network rm suivie de l’ID ou du nom du réseau.

```shell
docker network rm immich_default
```

Désinstaller Docker

```shell
# Remove unused data
docker system prune --all --volumes

# Remove all services
docker service rm $(docker service ls -q)

# Remove all containers
docker rm -f $(docker ps -aq)

# Remove all images
docker rmi -f $(docker images -aq)

# Remove all volumes
docker volume rm $(docker volume ls -q)
```


### Immich Cli

Liste des containers: `docker ps`

```
CONTAINER ID   IMAGE                                                            COMMAND                  CREATED        STATUS                  PORTS                                         NAMES
b859c6a470da   ghcr.io/immich-app/immich-server:release                         "tini -- /bin/bash s…"   27 hours ago   Up 27 hours (healthy)   0.0.0.0:2283->2283/tcp, [::]:2283->2283/tcp   immich_server
851b32a59d0b   ghcr.io/immich-app/immich-machine-learning:release               "tini -- python -m i…"   3 days ago     Up 2 days (healthy)                                                   immich_machine_learning
317aeaf6ce72   ghcr.io/immich-app/postgres:14-vectorchord0.3.0-pgvectors0.2.0   "/usr/local/bin/immi…"   3 days ago     Up 2 days (healthy)     5432/tcp                                      immich_postgres
9f4520f5586e   valkey/valkey:8-bookworm                                         "docker-entrypoint.s…"   3 days ago     Up 2 days (healthy)     6379/tcp                                      immich_redis

```

Lier un container

```shell
docker exec -it <id or name> <command>          # attach to a container with a command
docker exec -it immich_server bash
docker exec -it immich_machine_learning bash
```

Se lier au serveur

```shell
docker exec -it immich_server bash
```

Liste des utilisateurs: `immich-admin list-users`

```
Initializing Immich v1.134.0
Detected CPU Cores: 2
[
  {
    id: '9e4c8992-8987-4840-9197-219ec8be1f67',
    email: 'yann@yanfi.net',
    name: 'yann',
    profileImagePath: '',
    avatarColor: 'primary',
    profileChangedAt: 2025-06-10T18:07:16.330Z,
    storageLabel: 'admin',
    shouldChangePassword: true,
    isAdmin: false,
    createdAt: 2025-06-10T18:07:16.330Z,
    deletedAt: null,
    updatedAt: 2025-06-10T18:47:44.121Z,
    oauthId: '9aec801a-75f3-4452-a55d-1d3e94e045fe',
    quotaSizeInBytes: null,
    quotaUsageInBytes: 0,
    status: 'active',
    license: null
  },
  {
    id: '45422d67-8352-4ab7-a190-5a65093bfab0',
    email: 'claudine@home.arpa',
    name: 'Claudine',
    profileImagePath: '',
    avatarColor: 'red',
    profileChangedAt: 2025-06-09T15:34:59.700Z,
    storageLabel: null,
    shouldChangePassword: false,
    isAdmin: false,
    createdAt: 2025-06-09T15:34:59.700Z,
    deletedAt: null,
    updatedAt: 2025-06-09T22:00:00.260Z,
    oauthId: '',
    quotaSizeInBytes: null,
    quotaUsageInBytes: 0,
    status: 'active',
    license: null
  },
  {
    id: '341b4d34-516c-4c56-86bd-f9384062b841',
    email: 'yan@home.arpa',
    name: 'yan',
    profileImagePath: '',
    avatarColor: 'yellow',
    profileChangedAt: 2025-06-07T16:59:38.817Z,
    storageLabel: 'yan',
    shouldChangePassword: true,
    isAdmin: true,
    createdAt: 2025-06-07T16:59:38.817Z,
    deletedAt: null,
    updatedAt: 2025-06-10T18:16:40.544Z,
    oauthId: '',
    quotaSizeInBytes: null,
    quotaUsageInBytes: 14688,
    status: 'active',
    license: null
  }
]
```

Désactiver oauth: `immich-admin disable-oauth-login`

```
Initializing Immich v1.134.0
Detected CPU Cores: 2
OAuth login has been disabled.
```

## Watchtower

[Docker - Mise à jour automatique avec Watchtower](/posts/Docker_Mise_a_jour_automatique_avec_Watchtower/)

