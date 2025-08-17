+++
title = 'NFS (Network File System), partages réseau linux'
date = 2024-10-11 00:00:00 +0100
categories = ['nfs']
+++
*Introduction. NFS, pour Network File System (système de fichiers en réseau) est à l'origine un protocole qui permet à un ordinateur d'accéder à des fichiers via un réseau. Il permet de partager des données principalement entre systèmes UNIX ([How to Set Up and Configure an NFS Server on Debian 12](https://shape.host/resources/how-to-set-up-and-configure-an-nfs-server-on-debian-12))*


## NFS serveur

![nfs](nfs-logo.png){:width="120"} 

### Debian NFS

Pour commencer, nous devons installer le paquet serveur NFS sur la machine qui agira comme serveur. Le paquet NFS est disponible par défaut dans le dépôt Debian.

    sudo apt update
    sudo apt install nfs-kernel-server nfs-common

Une fois l'installation terminée, le service serveur NFS sera créé, exécuté et activé par défaut sur votre machine Debian.  
Vous pouvez vérifier l'état du service en exécutant la commande suivante :

    sudo systemctl status nfs-server

```
● nfs-server.service - NFS server and services
     Loaded: loaded (/lib/systemd/system/nfs-server.service; enabled; preset: enabled)
     Active: active (exited) since Sun 2024-01-07 14:29:34 CET; 23s ago
   Main PID: 521654 (code=exited, status=0/SUCCESS)
        CPU: 7ms

janv. 07 14:29:34 rnmkcy.eu systemd[1]: Starting nfs-server.service - NFS server and services...
janv. 07 14:29:34 rnmkcy.eu exportfs[521653]: exportfs: can't open /etc/exports for reading
janv. 07 14:29:34 rnmkcy.eu systemd[1]: Finished nfs-server.service - NFS server and services.
```

### NFSv4

La dernière version du protocole NFS est NFSv4, qui offre des fonctionnalités de sécurité et de performance améliorées. Dans cette section, nous allons configurer NFSv4 sur le serveur.

Ouvrez le fichier de configuration NFS `/etc/default/nfs-common`  
Dans ce fichier, trouvez le paramètre NEED_ STATD et le mettre à «no». Ensuite, trouvez le paramètre NEED_IDMAPD et définissez-le à « yes ». Ces changements sont nécessaires pour que NFSv4 fonctionne correctement.

```
NEED_STATD="no"
NEED_IDMAPD="yes"
```

Ouvrir la configuration du serveur NFS `/etc/default/nfs-kernel-server`

    sudo nano /etc/default/nfs-kernel-server

<u>Pour désactiver la version 3</u>  
Sur Debian 12, la version 2 de NFS est désactivée. Les versions 3 et 4 sont activées. Vérifiez la version installée à l'aide de cette commande : `sudo cat /proc/fs/nfsd/versions`

```
+3 +4 +4.1 +4.2
```

Ajouter la configuration suivante pour désactiver NFSv3 lors du fonctionnement du service nfs-server, et désactiver les demandes de montage de clients pour NFSv2 et NFSv3.

```
RPCNFSDOPTS="-N 3"
RPCMOUNTDOPTS="--manage-gids -N 3"
```

Redémarrer le service nfs-server pour appliquer les modifications, le serveur NFS n'acceptera que le protocole NFSv4.

    sudo systemctl restart nfs-server

###  Pare-feu 

*Pour sécuriser votre serveur NFS, il est essentiel de configurer le pare-feu.*

#### UFW

*UFW (Uncomplicated Firewall) pour permettre l'accès au serveur NFS uniquement à partir de réseaux de confiance.*

Permettre l'accès au serveur NFS à partir de votre réseau local

    sudo ufw allow from 192.168.0.0/24 to any port nfs

vérifier l'état pour s'assurer que les changements ont été appliqués

    sudo ufw status

```
2049                       ALLOW       192.168.0.0/24            
```

on voit que le port NFS 2049 est ajouté à UFW, permettant l'accès à partir de votre réseau local.

### Point de montage partage

#### Dossier

Créer le point de montage

    sudo mkdir /sharenfs

#### Volume LVM

*Dans le cas d'un système de fichiers LVM*

Créer un volume logique de 150Go sur le volume think-vg nommé nfs-share

```shell
sudo lvcreate -L 150G -n nfs-share think-vg        #  Attribue 150G
sudo mkfs.ext4 /dev/mapper/think--vg-nfs--share    
```

Relever UUID : `sudo blkid |grep "nfs--share"`

    /dev/mapper/think--vg-nfs--share: UUID="a06443f5-5684-4fcd-9cc0-998680ae41c2" BLOCK_SIZE="4096" TYPE="ext4"

Montage fstab  /sharenfs

    sudo mkdir /sharenfs

ajouter ce qui suit au fichier `/etc/fstab`

```
# /dev/mapper/think--vg-nfs--share
UUID=a06443f5-5684-4fcd-9cc0-998680ae41c2 /sharenfs   ext4    defaults        0       2
```

Montage et droits 

    sudo systemctl daemon-reload
    sudo mount -a

### Partage NFS

*choix Avec ou Sans pseudo système de fichiers*

#### Sans pseudo système de fichiers (défaut)

Déclarer un partage NFS

```shell
sudo mkdir -p /sharenfs
sudo chown -R $USER:$USER /sharenfs       # Modifications possibles par les utilisateurs avec ID=1000
```

Pour déclarer les partages NFS, il faut s'appuyer sur le fichier "/etc/exports"

    sudo nano /etc/exports

Ajouter la ligne suivante au fichier :

```
/sharenfs   192.168.0.0/255.255.255.0(rw,no_root_squash,no_subtree_check)
```

Voici les options d'exportation qui peuvent être utilisées :

*    **rw** permet l'accès en lecture et en écriture au système de fichiers.
*    **sync** indique au serveur NFS d'effectuer des opérations d'écriture (écriture d'informations sur le disque) lorsqu'elles sont demandées (s'applique par défaut).
*    **all_squash** fait correspondre tous les UID et GID des demandes des clients à l'utilisateur anonyme.
*    **no_all_squash** permet d'affecter tous les UID et GID des requêtes des clients à des UID et GID identiques sur le serveur NFS.
*    **root_squash** fait correspondre les demandes de l'utilisateur root ou de l'UID/GID 0 du client à l'UID/GID anonyme.
*    **no_root_squash** : Ne force pas le mapping de l'utilisateur root (UID=0) vers l'utilisateur anonyme
*    **subtree_check** : Active la vérification des droits d'accès des sous-dossiers. C'est le comportement par défaut.
*    **no_subtree_check** : Ignore la vérification des droits d'accès des sous-dossiers (perte d'un petit peu de sécurité mais peut améliorer la fiabilité)

Une fois les systèmes de fichiers définis dans le fichier exports, nous devons exécuter la commande `exportfs` pour qu'ils soient exportés. La commande exportfs peut être exécutée avec le drapeau

* `-a` exporter ou non-exporter tous les répertoires
* `-r` réexporter tous les répertoires, synchroniser `/var/lib/nfs/etab` avec `/etc/exports` et les fichiers sous `/etc/exports.d`
* `-v` obtenir une sortie en mode verbeux.

Une fois les systèmes de fichiers définis dans le fichier exports, nous devons exécuter la commande exportfs pour qu'ils soient exportés. La commande exportfs peut être exécutée avec le drapeau -a qui signifie exporter ou non-exporter tous les répertoires, -r qui signifie réexporter tous les répertoires, synchroniser /var/lib/nfs/etab avec /etc/exports et les fichiers sous /etc/exports.d, et -v qui permet d'obtenir une sortie en mode verbeux.

    sudo exportfs -arv

sudo exportfs -arv

Voici le résultat sur un serveur

```
exporting 192.168.0.0/24:/mnt/nfs-ssd
exporting 192.168.0.0/24:/sharenfs
```

exportant 10.70.5.0/24:/mnt/nfs_shares/backup

Pour afficher la liste des exportations en cours, exécutez la commande suivante. Veuillez noter que la table exportfs applique également certaines options par défaut qui ne sont pas explicitement définies 

    sudo exportfs -s

Résultat commande

```
/sharenfs  192.168.0.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
/mnt/nfs-ssd  192.168.0.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

#### Avec pseudo système de fichiers

*configurer un pseudo système de fichiers sur le serveur NFS et les exportations*

Un pseudo système de fichiers vous permet de configurer un seul système de fichiers partagés pour tous vos répertoires NFS au lieu d'utiliser plusieurs répertoires partagés.

D'abord, créez les répertoires nécessaires en utilisant les commandes suivantes :

```shell
sudo mkdir -p /sharenfs/{rnmkcy,alpine,e6230,pc1,multimedia}
#sudo chown -R nobody:nogroup /sharenfs    # pas de modification directe possible par le client
sudo chown -R $USER:$USER /sharenfs       # Modifications possibles par les utilisateurs avec ID=1000
```

Ensuite, créez les répertoires d'exportation :

```shell
sudo mkdir -p /exports/{home,rnmkcy,alpine,e6230,pc1,multimedia}
#sudo chown -R nobody:nogroup /exports     # pas de modification directe possible par le client
sudo chown -R $USER:$USER /exports        # Modifications possibles par les utilisateurs avec ID=1000
```

IMPORTANT : Un des avantages de NFS est la gestion des utilisateurs basée sur l'ID  
Les dossiers et fichiers de "home" sont modifiables par tous les utilisateurs ayant un ID=1000
{: .prompt-info }

**répertoires comme pseudo systèmes de fichiers**

montez les répertoires comme pseudo systèmes de fichiers

```shell
sudo mount --bind /home /exports/home
sudo mount --bind /sharenfs/rnmkcy /exports/rnmkcy
sudo mount --bind /sharenfs/alpine /exports/alpine
sudo mount --bind /sharenfs/e6230 /exports/e6230
sudo mount --bind /sharenfs/pc1 /exports/pc1
sudo mount --bind /sharenfs/multimedia /exports/multimedia
```

Pour que les pseudo-systèmes de fichiers soient montés en permanence, vous pouvez les ajouter au fichier /etc/fstab.

    sudo nano /etc/fstab

Ajouter les lignes suivantes au fichier 

```
/home /exports/home none bind
/sharenfs/rnmkcy /exports/rnmkcy none bind
/sharenfs/alpine /exports/alpine none bind
/sharenfs/e6230 /exports/e6230 none bind
/sharenfs/pc1 /exports/pc1 none bind
/sharenfs/multimedia /exports/multimedia none bind
```

**fichier des exportations**

Ensuite, ouvrez le fichier des exportations :

    sudo nano /etc/exports

Ajouter la ligne suivante au fichier :

    /exports   192.168.0.0/255.255.255.0(rw,no_root_squash,no_subtree_check,crossmnt,fsid=0)

Redémarrer le service serveur NFS pour appliquer les modifications :

    sudo systemctl restart nfs-server

Pour vérifier les répertoires exportés, exécutez la commande suivante :

    sudo showmount -e 192.168.0.215

Vous devriez voir le répertoire /exports listé comme un répertoire exporté.

```
Export list for 192.168.0.215:
/exports 192.168.0.0/255.255.255.0
```

## Client NFS

*Le serveur NFS est configuré, nous pouvons configurer la machine client NFS et monter le système de fichiers partagés.*

Installez le paquetage du client NFS 

```shell
sudo apt install nfs-common # debian
sudo pacman -S nfs-utils    # archlinux
```

### Montage 

#### Options montage fstab

*/etc/fstab est utilisé par le noyau Linux pour déterminer ce qui doit être monté au moment du démarrage et, par conséquent, les partages NFS peuvent également être montés par son intermédiaire. Cependant, si vous ne fournissez qu'une entrée régulière comme pour une partition ou un disque, vous rencontrerez le problème que le partage NFS est monté bien avant que les interfaces réseau n'aient été établies.*


Pour résoudre ce problème, vous devez fournir les paramètres suivants dans l'entrée du fichier /etc/fstab  
`x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=10s`

* **x-systemd.automount** : une unité de montage automatique systemd sera créée pour le système de fichiers.
* **x-systemd.requires=network-online.target** : configure une dépendance Requires= et After= entre l'unité de montage créée et l'autre unité spécifiée. En bref, cela indique à systemd d'exécuter l'automount une fois que le réseau est en ligne, ce qui signifie que c'est la sauce secrète pour s'assurer que le montage NFS dispose d'un réseau disponible.
* **x-systemd.device-timeout** : configure combien de temps systemd doit attendre que le montage apparaisse avant d'abandonner l'entrée de /etc/fstab. Vous pouvez utiliser des unités telles que ms, s, min et h, la valeur par défaut étant s pour les secondes.

Pour résumer, voici l'entrée du fichier /etc/fstab 

```
<server IP address>:<NFS share> <local mount location> nfs nofail,x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=10s
```

**nofail** n'est plus nécessaire dans le cas où le réseau n'est pas encore disponible, car systemd s'en est chargé. Cependant, il peut y avoir d'autres raisons pour lesquelles le partage n'est pas disponible, par exemple, le serveur NFS lui-même est inaccessible, donc **nofail** est un pari sûr pour s'assurer que la machine se réveille indépendamment du montage NFS.

Une fois que vous avez ajouté l'entrée dans le fichier /etc/fstab et redémarré le système, vous trouverez une entrée supplémentaire signalée par la commande mount

    systemd-<N> on <local mount location> type autofs (rw,relatime,fd=46,pgrp=1,timeout=0,minproto=5,maxproto=5,direct)

#### Montage fstab [défaut)

Créez le(s) répertoire(s) de montage cible 

    mkdir -p /mnt/sharenfs

Pour monter le serveur NFS de manière permanente, nous pouvons éditer le fichier /etc/fstab.

    sudo nano /etc/fstab

Ajoutez la ligne suivante au fichier :

```
#192.168.0.215:/ /mnt/sharenfs nfs4 x-systemd.after=network-online.target,soft,timeo=100,rsize=8192,wsize=8192
192.168.0.215:/ /mnt/sharenfs nfs4 nofail,x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=10s,rsize=8192,wsize=8192
```

Rechargez le gestionnaire systemd et montez tous les systèmes de fichiers dans le fichier /etc/fstab 

    sudo systemctl daemon-reload
    sudo mount -a

Vérifiez les systèmes de fichiers montés à l'aide de la commande df 

    df -h

Vous devriez voir que le serveur NFS est monté 

```
192.168.0.215:/                147G     48G   92G  35% /mnt/sharenfs
```

#### Montage fstab (pseudo système de fichiers sur le serveur)

Créez le(s) répertoire(s) de montage cible 

    mkdir -p /mnt/sharenfs/{users,rnmkcy,alpine,e6230,pc1,multimedia}

Montez les pseudo-systèmes de fichiers exportés dans les répertoires cibles 

```shell
sudo mount.nfs4 192.168.0.215:/home /mnt/sharenfs/users
sudo mount.nfs4 192.168.0.215:/rnmkcy /mnt/sharenfs/rnmkcy
sudo mount.nfs4 192.168.0.215:/alpine /mnt/sharenfs/alpine
sudo mount.nfs4 192.168.0.215:/e6230 /mnt/sharenfs/e6230
sudo mount.nfs4 192.168.0.215:/pc1 /mnt/sharenfs/pc1
sudo mount.nfs4 192.168.0.215:/multimedia /mnt/sharenfs/multimedia
```

Pour monter le serveur NFS de manière permanente, nous pouvons éditer le fichier /etc/fstab.

    sudo nano /etc/fstab

Ajoutez les lignes suivantes au fichier :

```
192.168.0.215:/home /mnt/sharenfs/users nfs4 soft,timeo=100,rsize=8192,wsize=8192
192.168.0.215:/rnmkcy /mnt/sharenfs/rnmkcy nfs4 soft,timeo=100,rsize=8192,wsize=8192
192.168.0.215:/alpine /mnt/sharenfs/alpine nfs4 soft,timeo=100,rsize=8192,wsize=8192
192.168.0.215:/e6230 /mnt/sharenfs/e6230 nfs4 soft,timeo=100,rsize=8192,wsize=8192
192.168.0.215:/pc1 /mnt/sharenfs/pc1 nfs4 soft,timeo=100,rsize=8192,wsize=8192
192.168.0.215:/multimedia /mnt/sharenfs/multimedia nfs4 soft,timeo=100,rsize=8192,wsize=8192
```

Rechargez le gestionnaire systemd et montez tous les systèmes de fichiers dans le fichier /etc/fstab 

    sudo systemctl daemon-reload
    sudo mount -a

Vérifiez les systèmes de fichiers montés à l'aide de la commande df 

    df -h

Vous devriez voir que le serveur NFS est monté sur chaque répertoire cible

```
192.168.0.215:/home             63G     18G   43G  29% /mnt/sharenfs/users
192.168.0.215:/rnmkcy          147G     40K  140G   1% /mnt/sharenfs/rnmkcy
192.168.0.215:/alpine          147G     40K  140G   1% /mnt/sharenfs/alpine
192.168.0.215:/e6230           147G     40K  140G   1% /mnt/sharenfs/e6230
192.168.0.215:/pc1             147G     40K  140G   1% /mnt/sharenfs/pc1
```

#### Systemd automount

*A mettre en place sur un ordinateur portable qui n'est pas toujours connecté au réseau local*

Les unités de montage doivent être nommées d'après les répertoires de points de montage qu'elles contrôlent.  
Exemple : le point de montage `/mnt/sharenfs/e6230` doit être configuré dans un fichier d'unité `mnt-sharenfs-e6230.mount`
{: .prompt-warning }

On peut définir le fichier unité en ligne de commande

    systemd-escape -p --suffix=mount /mnt/sharenfs/e6230

Renvoie : `mnt-sharenfs-e6230.mount`

Exemple ordinateur portable DELL e6230

Le partage local NFS : `192.168.0.215:/e6230`  
Le point de montage : `/mnt/sharenfs/e6230`  

Créer le point de montage

    sudo mkdir -p /mnt/sharenfs/e6230

Créer les fichiers systemd mount et automount

    sudo nano /etc/systemd/system/sharenfs-e6230.mount

Le contenu du fichier `/etc/systemd/system/mnt-sharenfs-e6230.mount`

```
[Unit]
Description=montage nfs

[Mount]
What=192.168.0.215:/e6230
Where=/mnt/sharenfs/e6230
Type=nfs
Options=defaults

[Install]
WantedBy=multi-user.target
```

`ajouter le paramètre  TimeoutSec  au fichier ci-dessus pour éviter les blocages lorsque le partage n'est pas présent sur votre réseau local`{: .prompt-info }

Le contenu du fichier `/etc/systemd/system/mnt-sharenfs-e6230.automount`

```
[Unit]
Description= automontage nfs

[Automount]
Where=/mnt/sharenfs/e6230
TimeoutIdleSec=0

[Install]
WantedBy=multi-user.target
```

A ce stade, vos fichiers unitaires sont prêts à être utilisés.  
Recharger systemd et lancer le mount pour tester

```shell
sudo systemctl daemon-reload
sudo systemctl start mnt-sharenfs-e6230..mount
```

Activez  automount pour qu'il se lance au démarrage et permettre au partage d'être monté à la demande.

```shell
sudo systemctl enable mnt-sharenfs-e6230.automount
```

## Liens

* [How to Set Up Nfs Server and Client on Debian 12](https://citizix.com/how-to-set-up-nfs-server-and-client-on-debian-12/)
* [How to Install NFS Server on Debian 12 Step-by-Step](https://www.linuxtechi.com/how-to-install-nfs-server-on-debian/)
* [Checking the Version of NFS](https://www.baeldung.com/linux/check-nfs-version)