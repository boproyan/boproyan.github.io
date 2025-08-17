+++
title = 'cwwk KVM vm-alpine05'
date = 2025-08-16
categories = ['virtuel']
+++
*Alpine Linux est utilisé pour héberger les applications IT-Tools - Stirling PDF - Monitoring Docker et Proxmox - Wallabag - Linkwarden*

![](alpine-linux-logo.png){:width="300"}  

## Alpine Linux

*Création machine virtuelle Alpine  de type KVM avec 2 Go de RAM, 1 cœur de processeur et 5 Go de disque dur.*

### Créer vm-alpine05 sur serveur cwwk

[Les dernières images Alpine Linux](https://alpinelinux.org/downloads/)  

Création d'une image virtuelle **vm-alpine05** sous le serveur Lenovo rnmkcy.eu  
On se connecte sur le serveur Lenovo en SSH, puis on exécute la commande suivante pour créer  une machine virtuelle Alpine avec 2 Go de RAM, 1 cœur de processeur et 45 Go de disque dur

```shell
sudo virt-install \
--osinfo alpinelinux3.17 \
--name vm-alpine05 \
--memory 4096 \
--vcpus 1 \
--cpu host \
--hvm \
--disk path=/srv/kvm/libvirt/images/vm-alpine05.qcow2,format=qcow2,size=45 \
--cdrom /home/yick/FreeUSB2To/iso/alpine-standard-3.22.1-x86_64.iso \
--network bridge=br0 \
--graphics vnc,port=5904  
```

Note: KVM ne connait que `alpinelinux3.17` (`sudo virt-install --osinfo list |grep alpine`)

Après exécution dans un terminal de la commande ci dessus, on arrive sur *En attente de fin d'installation*

ARRETER la VM pour changer le port vnc (5903)  
![](vnc-port.png){:width="400" .normal}  
Puis redémarrer la VM  

### Configurer vm-alpine

`ATTENTION: Désactiver "Affichage VNC" des autres machines ou changer le port`{: .prompt-warning }

Le serveur Lenovo n'a pas d'affichage, il faut créer un tunnel ssh depuis un terminal d'un poste client

    ssh -L 5900:127.0.0.1:5904 yick@192.168.0.205 -p 55205 -i /home/yann/.ssh/yick-ed25519

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

Relever l'adresse ip allouée : `ip a` --> 192.168.10.195  
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

    ssh alpi@192.168.10.195

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

### Modifier hosts, hostname et resolv.conf

Fichier /etc/hosts

```
127.0.0.1	vm-alpine05 localhost localhost.localdomain localhost
::1		localhost localhost.localdomain
```

Fichier /etc/hostname

```
vm-alpine05
```

### Réseau IPV4 statique

[How to configure static IP address on Alpine Linux](https://www.cyberciti.biz/faq/how-to-configure-static-ip-address-on-alpine-linux/)

Le fichier de configuration `/etc/network/interfaces`

    /etc/network/interfaces

```shell
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
  address 192.168.10.215/24
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

Un redémarrage de la macine est préférable

```shell
reboot
```

`ATTENTION: Déconnexion SSH car changement adresse IP`{: .prompt-warning }

Connexion SSH avec IP 192.168.10.215

    ssh alpi@192.168.10.215

Message à la connexion SSH, `/etc/motd`

```
    _     _         _                _      _                           
   / \   | | _ __  (_) _ __    ___  | |    (_) _ __   _   _ __  __      
  / _ \  | || '_ \ | || '_ \  / _ \ | |    | || '_ \ | | | |\ \/ /      
 / ___ \ | || |_) || || | | ||  __/ | |___ | || | | || |_| | >  <       
/_/   \_\|_|| .__/ |_||_| |_| \___| |_____||_||_| |_| \__,_|/_/\_\      
            |_|                  _         _                ___   ____  
__   __ _ __ ___           __ _ | | _ __  (_) _ __    ___  / _ \ | ___| 
\ \ / /| '_ ` _ \  _____  / _` || || '_ \ | || '_ \  / _ \| | | ||___ \ 
 \ V / | | | | | ||_____|| (_| || || |_) || || | | ||  __/| |_| | ___) |
  \_/  |_| |_| |_|        \__,_||_|| .__/ |_||_| |_| \___| \___/ |____/ 
 _   ___  ____      _   __     ___ |_| _   ___     ____   _  ____       
/ | / _ \|___ \    / | / /_   ( _ )   / | / _ \   |___ \ / || ___|      
| || (_) | __) |   | || '_ \  / _ \   | || | | |    __) || ||___ \      
| | \__, |/ __/  _ | || (_) || (_) |_ | || |_| |_  / __/ | | ___) |     
|_|   /_/|_____|(_)|_| \___/  \___/(_)|_| \___/(_)|_____||_||____/      

```

### OpenSSH avec clés

*Connexion ssh sur un autre port avec un jeu de clés*

Générer une paire de clé sur l'ordinateur de bureau PC1  
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec la machine virtuelle vm-alpine05

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/vm-alpine05
    chmod 600 ~/.ssh/vm-alpine05

Copier la clé publique `cat ~/.ssh/vm-alpine05.pub` dans le presse-papier

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIXzOxqt9LMDqTrbHZ05+ZBRLdQkRUpoBG0fTPsjjq10 yann@pc1
```

On se connecte sur la machine virtuelle alpine linux "vm-alpine05"

Créer le répertoire et ouvrir nouveau fichier

```shell
mkdir -p $HOME/.ssh/
nano $HOME/.ssh/authorized_keys
```

Coller le contenu du presse-papier , sauver le fichier et sortir

Modifier les droits

```shell
chmod 600 $HOME/.ssh/authorized_keys
```

Passer en mode su

```shell
su -
```

Modifier la configuration serveur SSH

```shell
nano /etc/ssh/sshd_config
```

Modifier

```
Port = 55215
PasswordAuthentication no
```

Relancer le serveur

    service sshd restart

Test connexion

```bash
ssh alpi@192.168.10.215 -p 55215 -i /home/yann/.ssh/vm-alpine05
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
http://mirrors.ircam.fr/pub/alpine/v3.20/main
http://mirrors.ircam.fr/pub/alpine/v3.20/community
```

Installer sudo 

```bash
apk update
apk add sudo
```

Ajouter un utilisateur avec les privlèges root

```bash
echo "alpi     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/20-alpi
```

### Alpine Linux - Mise à jour automatique

Installer curl

    sudo apk add curl

Pour chaque nouveau serveur Alpine Linux, créer en mode su un script shell nommé `apk-autoupgrade` dans le dossier `/etc/periodic/daily/` avec les permissions suivantes : 700

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

    nano /etc/periodic/daily/apk-autoupgrade

Ajouter ce qui suit en fin de fichier

```
curl \
-H "X-Email: ntfy@cinay.eu" \
-H "Title: 💻 Alpine Linux vm-alpine05 : Fin exécution script 'apk-autoupgrade'" \
-H "Authorization: Bearer tk_xxxxxxxxxxxxxxxxxxxxxxxxxxxx" \
-H prio:low \
-d "vm-alpine05 192.168.10.215 
✔️ Fin exécution script /etc/periodic/daily/apk-autoupgrade" \
https://noti.rnmkcy.eu/yan_infos
```

### NFS

Cette page documente la configuration du système de fichiers réseau (NFS) du côté du serveur et du client, y compris les autofs et l'authentification Kerberos.
Installation

Installez le paquet suivant pour le service client et serveur NFS.

```shell
sudo apk add nfs-utils
```

Client NFS

Créer dossier de montage

```shell
mkdir ~/sharenfs
```

Pour monter automatiquement les actions NFS, une entrée doit être faite à `/etc/fstab` comme suit:

```shell
192.168.0.205:/sharenfs /home/alpi/sharenfs nfs4 rw,_netdev 0 0
```

Pour monter nfs share depuis le fichier /etc/fstab au démarrage du système :

```shell
sudo rc-update add nfsmount
```

Pour monter les actions de nfs depuis le fichier /etc/fstab :

```shell
sudo rc-service nfsmount start
```

Vous pouvez vérifier vos services de démarrage :

```shell
sudo rc-status
```

Conseil : netmount est un service général pour tous les systèmes de fichiers basés sur le réseau, tandis que nfsmount est spécialement conçu pour NFS.

Pour utiliser netmount, voici les commandes équivalentes :

```
# rc-service netmount démarrage
# rc-update add netmount
```

> En cas d'erreur au montage NFS , `REDEMARRER LA MACHINE`.
{: .prompt-warning }


### docker et docker-compose

![](docker-logo-a.png)  

[Install docker & docker-compose on Alpine Linux](https://geekscircuit.com/install-docker-docker-compose-on-alpine-linux/)

Passer en root

    su -

Editer la configuration des dépôts `/etc/apk/repositories`
Trouvez maintenant la ligne qui se termine dans /community Ensuite, retirez le # au début de la ligne.

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

### Watchtower - Mise à jour auto

[Watchtower](https://github.com/v2tec/watchtower) est un outil open-source permettant d'automatiser les mises à jour. Il se présente sous la forme d'une image Docker à lancer très simplement :

```shell
# Exemple Syntaxe mode utilisateur (non root)
docker run --detach \
    --name watchtower \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower
```

Le paramètre `--volume` est obligatoire pour que Watchtower puisse communiquer avec le démon Docker sur le système hôte.

Il est possible de lui préciser quand faire les mises à jour, histoire d'éviter une coupure de service (même mineure) en pleine heure de pointe. Le paramètre `--schedule` prend comme valeur une expression crontab. Sinon, le paramètre `--interval` peut-être utilisé pour définir le nombre de secondes entre chaque vérification.

À noter également l'option `--cleanup` qui permet de supprimer les anciennes images afin de ne pas surcharger l'espace disque de l'hôte.

nous devons d'abord lancer le conteneur Watchtower lui-même en exécutant une commande Docker :

```bash
docker run \
  --name watchtower \
  --volume /var/run/docker.sock:/var/run/docker.sock \
  --detach \
  containrrr/watchtower \
  --schedule "0 0 4 * * *" \
  --cleanup
```

Ainsi, nous exécutons Watchtower en mode détaché, ce qui lui donne accès au **socket Docker**, qui est nécessaire à Watchtower pour surveiller et mettre à jour les conteneurs.  
Notre sortie montre l'identifiant unique du conteneur Watchtower qui s'exécute maintenant en arrière-plan.  

```text
Status: Downloaded newer image for containrrr/watchtower:latest
66ded140d58a02a2a5132b556ee0ca1c42c267a7b75e273e7e6f8fe4992ab420
```

Plus besoin donc de se soucier de mettre à jour ses containers, Watchtower le fait tout seul, et ça fonctionne parfaitement !

    docker logs watchtower

```
time="2025-08-16T13:58:50Z" level=info msg="Watchtower 1.7.1"
time="2025-08-16T13:58:50Z" level=info msg="Using no notifications"
time="2025-08-16T13:58:50Z" level=info msg="Checking all containers (except explicitly disabled with label)"
time="2025-08-16T13:58:50Z" level=info msg="Scheduling first run: 2025-08-17 04:00:00 +0000 UTC"
time="2025-08-16T13:58:50Z" level=info msg="Note that the first check will be performed in 14 hours, 1 minute, 9 seconds"
```

## cAdvisor+ Prometheus + Grafana + InfluxDB

[Monitoring Docker (cAdvisor+ Prometheus + Grafana)](/posts/Monitoring/#monitoring-docker-cadvisor-prometheus--grafana)  

## Linkwarden

*Linkwarden est un gestionnaire de liens open-source robuste et polyvalent, conçu pour aider les utilisateurs à organiser, sauvegarder et retrouver facilement leurs liens favoris.  
Linkwarden a des exigences matérielles minimales - il a été testé sur un VPS avec 4Go de mémoire.*

Lien <https://docs.linkwarden.app>

### Exigences

* Docker
* Curl
* Nano

### Installation

1 - Télécharger les fichiers requis

```shell
mkdir linkwarden && cd linkwarden
curl -O https://raw.githubusercontent.com/linkwarden/linkwarden/refs/heads/main/docker-compose.yml
curl -L https://raw.githubusercontent.com/linkwarden/linkwarden/refs/heads/main/.env.sample -o ".env"
```

2 - Configurer les variables d'environnement

Les variables d'environnement requises sont :

```
NEXTAUTH_URL=http://localhost:3000/api/v1/auth
NEXTAUTH_SECRET=VERY_SENSITIVE_SECRET
MEILI_MASTER_KEY=VERY_SENSITIVE_MEILI_MASTER_KEY
POSTGRES_PASSWORD=CUSTOM_POSTGRES_PASSWORD
```

La seule chose que vous devez changer ici est **NEXTAAUTH_SECRET**, **POSTGRES_PASSWORD**, et **MEILI_MASTER_KEY**, ils devraient tous être des phrases secrètes différentes. La phrase doit être enveloppée en guillemets simples ou doubles si des caractères spéciaux sont utilisés.

La NEXTAUTH_URL ne doit être changée que si vous l'hébergez ailleurs.

3 - Exécution

Dans le dossier principal (où vous avez créé le fichier .env), lancez simplement ce qui suit :

```shell
sudo docker-compose up
```

Après quelques minutes (selon votre connexion Internet), vous pouvez accéder à Linkwarden via <http://192.168.10.215:3000> (ou le nom d'hôte sur lequel vous avez déployé Linkwarden).

![](Linkwarden01.png){: .normal}

Après avoir créer un compte  
![](Linkwarden02.png){: .normal}

[A Step-by-Step Guide How to Start docker-compose Container at Boot](https://jason19970210.medium.com/a-step-by-step-how-to-start-docker-compose-container-at-boot-effe9964e57f)  
![](Linkwarden03.png){: .normal}

### cwwk - linkwarden

Domaines link.home.arpa et link.rnmkcy.eu

* [Extension firefox](https://docs.linkwarden.app/getting-started/browser-extension)