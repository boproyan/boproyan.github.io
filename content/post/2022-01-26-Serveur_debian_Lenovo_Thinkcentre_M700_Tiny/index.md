+++
title = 'Serveur debian 11 - Lenovo Thinkcentre M700 Tiny'
date = 2022-01-26 00:00:00 +0100
categories = ['debian']
+++
![](debian11-logoA.png){:height="100"} ![](lenovo003.png){:height="100"}

  - [Debian 11 bullseye](#debian-11-bullseye)
    - [Ip V4 V6 statiques](#ip-v4-v6-statiques)
    - [OpenSSH, clé et script](#openssh-clé-et-script)
    - [Utilitaires](#utilitaires)
    - [Motd](#motd)
    - [Parefeu UFW](#parefeu-ufw)
    - [Volumes partagés](#volumes-partagés)
      - [Partage](#partage)
      - [media](#media)
    - [Etat des montages LVM](#etat-des-montages-lvm)
  - [NFS](#nfs)
    - [Points de montage /etc/exports](#points-de-montage-etcexports)
    - [Client - Test des groupes de partage NFS](#client---test-des-groupes-de-partage-nfs)
  - [Domaine rnmkcy.eu](#domaine-rnmkcyeu)
    - [DNS local](#dns-local)
    - [Zone DNS OVH](#zone-dns-ovh)
    - [Certificats Let's Encrypt](#certificats-lets-encrypt)
  - [Nginx compilation, PHP8, MariaDB](#nginx-compilation-php8-mariadb)
    - [Nginx compilé](#nginx-compilé)
    - [Nouvelles configurations nginx](#nouvelles-configurations-nginx)
    - [PHP8](#php8)
    - [MariaDB](#mariadb)
  - [rnmkcy.eu](#rnmkcyeu)
    - [Page d'accueil rnmkcy.eu](#page-daccueil-rnmkcyeu)
    - [Droits dossier /var/www/default-www](#droits-dossier-varwwwdefault-www)
    - [Site statique static.rnmkcy.eu](#site-statique-staticrnmkcyeu)
    - [Diceware, osm-new](#diceware-osm-new)
  - [Nextcloud](#nextcloud)
    - [Installation dernière version](#installation-dernière-version)
    - [Utilisateur nextcloud](#utilisateur-nextcloud)
    - [MariaDB base de données mysql](#mariadb-base-de-données-mysql)
    - [PHP](#php)
      - [PHP pool nextcloud](#php-pool-nextcloud)
      - [mémoire pour PHP](#mémoire-pour-php)
      - [PHP OPcache](#php-opcache)
    - [Nginx](#nginx)
      - [/etc/nginx/conf.d/rnmkcy.eu.d/nextcloud.conf](#etcnginxconfdrnmkcyeudnextcloudconf)
    - [Paramétrer Nextcloud](#paramétrer-nextcloud)
      - [Compte administrateur](#compte-administrateur)
      - [Configuration de la région](#configuration-de-la-région)
      - [Thème, Apparence, Messagerie et Stockage](#thème-apparence-messagerie-et-stockage)
      - [Cache de données : APCu & Redis](#cache-de-données-apcu-redis)
      - [Travaux cron](#travaux-cron)
  - [CodeIgniter4](#codeigniter4)
    - [Création projet CodeIgniter avec Composer](#création-projet-codeigniter-avec-composer)
    - [Variables environnement](#variables-environnement)
  - [Connexion et authentification JWT avec PHP MYSQL](#connexion-et-authentification-jwt-avec-php-mysql)
  - [2FA Authentification](#2fa-authentification)
    - [Installation](#installation)
    - [Configurations nginx](#configurations-nginx)
      - [rnmkcy.eu.conf](#rnmkcyeuconf)
    - [locations.conf](#locationsconf)
  - [Audio](#audio)
    - [Navidrome](#navidrome)
      - [Prérequis](#prérequis)
      - [Utilisateur "navidrome"](#utilisateur-navidrome)
      - [Créer une structure de répertoire](#créer-une-structure-de-répertoire)
      - [Obtenir Navidrome](#obtenir-navidrome)
      - [Créer le fichier de configuration](#créer-le-fichier-de-configuration)
      - [Créer une unité Systemd navidrome.service](#créer-une-unité-systemd-navidromeservice)
      - [Proxy nginx zic.rnmkcy.eu (navidrome)](#proxy-nginx-zicrnmkcyeu-navidrome)
    - [SSMTP](#ssmtp)


![](lenovo001.png){:width=300}  
**ThinkCentre M700 Tiny**  

Intel Core i3-6100T 2.20Ghz  
8 Go RAM DDR4  
HDD Mobile 1.8 To  
Gigabit LAN  
M.2 slots	M.2 2280/2242 M-key (SATA); M.2 2230 (WiFi/BT);  
Dimensions (L x P x H) (mm) : 34,5 x 182,9 x 179 mm  
Ports d'E/S (entrées/sorties)	

* Avant : 
    * 2 USB 3.0 (dont un avec technologie Fast Charge)
    * 2 audio
* Arrière : 
    * Jusqu'à 4 USB 3.0
    * DisplayPort™ + DisplayPort™ 1 série par retrait de cache (VGA / HDMI™ DisplayPort™ / Série)
    * 1 LAN
    * 1 audio
* WiFi et Bluetooth®	
    * 1 x 1 802.11 a/c WLAN + Bluetooth® 4.0
    * 2 x 2 802.11 a/c WLAN + Bluetooth® 4.0

## Debian 11 bullseye

![](debian11-logo.png)

Installer avec clé USB contenant image ISO **firmware-11.2.0-amd64-netinst.iso** (pour les pilotes carte wifi/bluetooth)  

Machine : think  
root : root49600  
lenovo : lenovo49600  
Partitions LVM avec home séparé et 50Go utilisé  

La machine reboot à la fin de l'installation  
Se connecter avec l'utilisateur "lenovo"  
Relever l'adresse ip : `ip a` , exemple 192.168.0.11

On peut également se connecter via ssh : `ssh lenovo@192.168.0.11`  


Passer en `su` , installer sudo

    su
    apt install sudo
    echo "lenovo     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers


### Ip V4 V6 statiques

Pour le nexthop IPV6 FreeBox

    ip a |grep "inet6 fe80"

*inet6 fe80::223:24ff:fec9:686/64 scope link*

Passage en ip statique 192.168.0.145 et 2a01:e0a:2de:2c75::1 (nexthop `fe80::223:24ff:fec9:686`)   	
lien local box : fe80::8e97:eaff:fe39:66d6

    nano /etc/network/interfaces

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback
# The primary network interface
allow-hotplug eno1
iface eno1 inet static
 address 192.168.0.145
 netmask 255.255.255.0
 gateway 192.168.0.254
# IPv6 interface
iface eno1 inet6 static
 address 2a01:e0a:2de:2c75::1
 netmask 64
 post-up ip -6 route add default via fe80::8e97:eaff:fe39:66d6 dev eno1

```

Pour éviter l'erreur `ICMPv6: RA: ndisc_router_discovery failed to add default route`  
Correction, désactiver l’annonce des routes ([accept_ra](https://sysctl-explorer.net/net/ipv6/accept_ra/)) :

    nano /etc/sysctl.conf

```
# Uncomment the next line to enable packet forwarding for IPv6
#  Enabling this option disables Stateless Address Autoconfiguration
#  based on Router Advertisements for this host
net.ipv6.conf.all.forwarding=1

# Accept Router Advertisements
net.ipv6.conf.all.accept_ra=0

```

Activer `net.ipv6.conf.all.forwarding=1` et ajouter `net.ipv6.conf.all.accept_ra=0`   
Puis `sysctl -p` pour une validation immédiate  

reboot : `systemctl reboot`

Vérification des adresses IP

    ip a

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:23:24:c9:06:86 brd ff:ff:ff:ff:ff:ff
    altname enp0s31f6
    inet 192.168.0.145/24 brd 192.168.0.255 scope global eno1
       valid_lft forever preferred_lft forever
    inet6 2a01:e0a:2de:2c75::1/64 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::223:24ff:fec9:686/64 scope link 
       valid_lft forever preferred_lft forever
3: wlp1s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 58:fb:84:87:b4:37 brd ff:ff:ff:ff:ff:ff

```

Se connecter via ssh

    ssh lenovo@192.168.0.145

Hostname

    hostnamectl

```
   Static hostname: think
         Icon name: computer-desktop
           Chassis: desktop
        Machine ID: 2354bc7c0cb944afa12f95be83a41118
           Boot ID: 1ed834c964f6446c839098747d1b3571
  Operating System: Debian GNU/Linux 11 (bullseye)
            Kernel: Linux 5.10.0-10-amd64
      Architecture: x86-64
```

Date et heure

    timedatectl

```
               Local time: sam. 2022-01-01 10:39:36 CET
           Universal time: sam. 2022-01-01 09:39:36 UTC
                 RTC time: sam. 2022-01-01 09:39:36
                Time zone: Europe/Paris (CET, +0100)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```


### OpenSSH, clé et script

![OpenSSH](ssh_logo1.png){:width="70"}  
**connexion avec clé**  
<u>sur l'ordinateur de bureau</u>
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec le serveur.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/lenovo-ed25519

Envoyer les clés publiques sur le serveur KVM   

    ssh-copy-id -i ~/.ssh/lenovo-ed25519.pub lenovo@192.168.0.145

<u>sur le serveur KVM</u>
On se connecte  

    ssh lenovo@192.168.0.145

Modifier la configuration serveur SSH  

    sudo nano /etc/ssh/sshd_config

Modifier

```conf
Port = 55145
PasswordAuthentication no
```

Relancer le serveur

    sudo systemctl restart sshd

Test connexion

    ssh -p 55145 -i ~/.ssh/lenovo-ed25519 lenovo@192.168.0.145

### Utilitaires

Installer utilitaires  

    sudo apt install rsync curl tmux jq figlet git

### Motd

Effacer et créer motd

    sudo rm /etc/motd && sudo nano /etc/motd

```
  _                                __  __  ____  __    __  
 | |    ___  _ _   ___ __ __ ___  |  \/  ||__  |/  \  /  \ 
 | |__ / -_)| ' \ / _ \\ V // _ \ | |\/| |  / /| () || () |
 |____|\___||_||_|\___/ \_/ \___/ |_|  |_| /_/  \__/  \__/ 
  _  ___  ___     _   __  ___     __     _  _ _   ___      
 / |/ _ \|_  )   / | / / ( _ )   /  \   / || | | | __|     
 | |\_, / / /  _ | |/ _ \/ _ \ _| () |_ | ||_  _||__ \     
 |_| /_/ /___|(_)|_|\___/\___/(_)\__/(_)|_|  |_| |___/     
```

Script ssh_rc_bash

>ATTENTION!!! Les scripts sur connexion peuvent poser des problèmes pour des appels externes autres que ssh

```bash
wget https://static.xoyaz.xyz/files/ssh_rc_bash
chmod +x ssh_rc_bash # rendre le bash exécutable
./ssh_rc_bash        # exécution
```

![](lenovo002.png)

### Parefeu UFW

![ufw](ufw-logo.png){:width="50"} 

*UFW, ou pare - feu simple , est une interface pour gérer les règles de pare-feu dans Arch Linux, Debian ou Ubuntu. UFW est utilisé via la ligne de commande (bien qu'il dispose d'interfaces graphiques disponibles), et vise à rendre la configuration du pare-feu facile.*

Installation **Debian / Ubuntu**

    sudo apt install ufw

*Par défaut, les jeux de règles d'UFW sont vides, de sorte qu'il n'applique aucune règle de pare-feu, même lorsque le démon est en cours d'exécution.*   

Les règles 

```bash
sudo ufw allow 55145/tcp  # port SSH
sudo ufw allow http       # port 80
sudo ufw allow https      # port 443
sudo ufw allow DNS        # port 53
```

Activer le parefeu

    sudo ufw enable

```
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

Status

     sudo ufw status verbose

```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
55145/tcp                  ALLOW IN    Anywhere                  
80/tcp                     ALLOW IN    Anywhere                  
443                        ALLOW IN    Anywhere                  
53 (DNS)                   ALLOW IN    Anywhere                  
55145/tcp (v6)             ALLOW IN    Anywhere (v6)             
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
443 (v6)                   ALLOW IN    Anywhere (v6)             
53 (DNS (v6))              ALLOW IN    Anywhere (v6)             
```

### Volumes partagés

#### Partage

On va ajouter un volume logique LVM de 500Go avec système de fichier ext4 

    sudo lvcreate -L 500G -n lenovo_partage think-vg
    sudo mkfs.ext4 /dev/think-vg/lenovo_partage

Créer un point de montage 

    sudo mkdir -p /srv/thinkcentre
    sudo  chown nobody:nogroup /srv/thinkcentre
    sudo mount /dev/mapper/think--vg-lenovo_partage /srv/thinkcentre

utilisation de set group ID (Set GID), car tous les fichiers et répertoires créés dans le partage de groupe seront automatiquement définis avec le même propriétaire que le partage lui-même.


    sudo chmod 2770 /srv/thinkcentre

Cela a également défini les autorisations 770 sur le répertoire, de sorte que l'utilisateur racine et le groupe défini disposent d'autorisations complètes. Les 2 premiers permettent setgid.

Ensuite, nous créons un groupe appelé 'partage' et modifions le répertoire /srv/nfs afin que le propriétaire du groupe soit ce groupe 'partage'. Nous spécifions également manuellement le GID qui sera utilisé pour le groupe en tant que 9999; il doit s'agir d'un numéro libre sur votre client et votre serveur. J'ai exécuté groupadd sur le client et sur le serveur, et ajouté un utilisateur au groupe.

    sudo groupadd -g 9999 partage
    sudo chgrp partage /srv/thinkcentre
    sudo usermod -a -G partage $USER

Nous pouvons confirmer que setgid est en place, comme indiqué ci-dessous, où le bit d'exécution pour les autorisations de groupe est une minuscule. Cela passera à une majuscule  si le groupe ne dispose pas de l'autorisation d'exécution et que seul setgid est en place.

Montage du volume partagé par fstab

    sudo nano /etc/fstab

```
/dev/mapper/think--vg-lenovo_partage /srv/thinkcentre      ext4    defaults        0       2
```

Créer un lien "Partage" sur ce montage

    sudo ln -s /srv/thinkcentre /home/lenovo/Partage

#### media

On va ajouter un volume logique LVM de 150Go avec système de fichier ext4 

    sudo lvcreate -L 150G -n lenovo_media think-vg
    sudo mkfs.ext4 /dev/think-vg/lenovo_media
    mkdir ~/media
    sudo mount /dev/think-vg/lenovo_media /home/lenovo/media
    sudo chown $USER.$USER /home/lenovo/media

Montage fstab

    sudo nano /etc/fstab

```
/dev/mapper/think--vg-lenovo_media /home/lenovo/media      ext4    defaults        0       2
```

Les dossiers extérieurs

    mkdir -p /home/lenovo/media/{static,www/diceware,www/osm-new,CalibreTechnique,BiblioCalibre}

Le site statique en provenance de lxcdeb

    mkdir  /home/lenovo/media/static

### Etat des montages LVM

```
root@think:/home/lenovo# pvs
  PV         VG       Fmt  Attr PSize  PFree
  /dev/sda3  think-vg lvm2 a--  <1,82t 1,28t
root@think:/home/lenovo# vgs
  VG       #PV #LV #SN Attr   VSize  VFree
  think-vg   1   4   0 wz--n- <1,82t 1,28t
root@think:/home/lenovo# lvs
  LV             VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home           think-vg -wi-ao---- <29,79g                                                    
  lenovo_partage think-vg -wi-ao---- 500,00g                                                    
  root           think-vg -wi-ao----  15,82g                                                    
  swap_1         think-vg -wi-ao---- 976,00m                                                    
```




## NFS

![nfs](nfs-logo.png){:width="80"} 

Installer le paquetage du serveur NFS 

    sudo apt -y install nfs-kernel-server rpcbind


### Points de montage /etc/exports

Les points de montage du serveur NFS sont configurés avec le fichier /etc/exports. Ce fichier répertorie les répertoires accessibles via NFS. Vous pouvez également créer des fichiers de configuration dans le répertoire /etc/exports.d/, à condition qu’ils portent l’extension .exports.

Vous trouverez ci-dessous un exemple de configuration NFS dans le fichier `/etc/exports` 

    sudo nano /etc/exports

```
/srv/thinkcentre 192.168.0.0/24(rw,sync,no_subtree_check)
```

Après toute modification apportée au fichier /etc/exports, nous devons utiliser la commande exportfs pour mettre à jour le tableau des systèmes de fichiers NFS exportés.

    sudo exportfs -arv

```
exporting 192.168.0.0/24:/srv/thinkcentre
```

L'indicateur **-a** exporte tous les répertoires, l'indicateur **-r** réexporte tous les répertoires et supprime les anciennes entrées, tandis que l'indicateur **-v** fournit des informations détaillées et génère toutes les exportations NFS.

Vérification

    sudo showmount -e

```
Export list for think:
/srv/thinkcentre 192.168.0.0/24
```

### Client - Test des groupes de partage NFS

Créer le groupe (partage ou un autre nom) et associer l'utilisateur

    sudo groupadd -g 9999 partage
    sudo usermod -a -G  partage $USER


Maintenant que le point de montage NFS est prêt, à partir du client, nous montons le partage NFS en tant qu'utilisateur root.

    sudo mkdit -p /mnt/lenovo
    sudo mount 192.168.0.145:/srv/thinkcentre /mnt/lenovo

Désormais, si un utilisateur accède au répertoire `/mnt/temp` et crée un fichier ou un répertoire, il appartiendra au groupe 'partage'.

    touch /mnt/lenovo/setgid-test
    ls -la /mnt/lenovo/

Montage partage sur fstab

    sudo nano /etc/fstab

```
# partage NFS sur rnmkcy.eu "lenovo thinkcentre M700 Tiny"
192.168.0.145:/srv/thinkcentre /mnt/lenovo	nfs	x-systemd.automount,x-systemd.idle-timeout=300,async    0 0
```

Monter et vérifier

    sudo mount -a
    ls -la /mnt/lenovo

## Domaine rnmkcy.eu

### DNS local

Sur le poste du même réseau, modifier `/etc/hosts`

    192.168.0.145   rnmkcy rnmkcy.eu cloud.rnmkcy.eu static.rnmkcy.eu dev.rnmkcy.eu

### Zone DNS OVH 

![dns](dns-logo.png){:width="30"}  
*Zone DNS accessible UNIQUEMENT en IPV6*  


```dns
$TTL 3600
@	IN SOA dns110.ovh.net. tech.ovh.net. (2022010102 86400 3600 3600000 300)
         IN NS     dns110.ovh.net.
         IN NS     ns110.ovh.net.
         IN AAAA   2a01:e0a:2de:2c75::1
         IN CAA    128 issue "letsencrypt.org"
*        IN AAAA   2a01:e0a:2de:2c75::1
```

### Certificats Let's Encrypt

![letsencrypt](letsencrypt-logo1.png){:width="80"}

Installation gestionnaire des certificats Let's Encrypt

```bash
cd ~
sudo apt install socat # prérequis
git clone https://github.com/acmesh-official/acme.sh.git
cd acme.sh
./acme.sh --install 
```

Se déconnecter puis se reconnecter utilisateur  

Les clés OVH API   

    export OVH_AK="xxxxxxxxxxxxxxxxxx"
    export OVH_AS="yyyyyyyyyyyyyyyyyyyyyyyyyyyy"

Génération des certificats  

    acme.sh --dns dns_ovh --server letsencrypt --issue --keylength ec-384 -d 'rnmkcy.eu' -d '*.rnmkcy.eu'

Résultat de l'installation

```
[sam. 01 janv. 2022 15:54:46 CET] Your cert is in: /home/lenovo/.acme.sh/rnmkcy.eu_ecc/rnmkcy.eu.cer
[sam. 01 janv. 2022 15:54:46 CET] Your cert key is in: /home/lenovo/.acme.sh/rnmkcy.eu_ecc/rnmkcy.eu.key
[sam. 01 janv. 2022 15:54:46 CET] The intermediate CA cert is in: /home/lenovo/.acme.sh/rnmkcy.eu_ecc/ca.cer
[sam. 01 janv. 2022 15:54:46 CET] And the full chain certs is there: /home/lenovo/.acme.sh/rnmkcy.eu_ecc/fullchain.cer
```

Installation des certificats

```
sudo mkdir -p /etc/ssl/private/
sudo chown $USER -R /etc/ssl/private/
acme.sh --ecc --install-cert -d 'rnmkcy.eu' -d '*.rnmkcy.eu' --key-file /etc/ssl/private/rnmkcy.eu-key.pem --fullchain-file /etc/ssl/private/rnmkcy.eu-fullchain.pem  --reloadcmd 'sudo systemctl reload nginx.service'
```

Résultat

```
[sam. 01 janv. 2022 15:55:29 CET] Installing key to: /etc/ssl/private/rnmkcy.eu-key.pem
[sam. 01 janv. 2022 15:55:29 CET] Installing full chain to: /etc/ssl/private/rnmkcy.eu-fullchain.pem
```

Supprimer ` --reloadcmd 'sudo systemctl reload nginx.service'` à la ligne précédente si Nginx n'est pas installé
{: .prompt-warning }

Editer le crontab, supprimer la ligne existante et ajouter ce qui suit

    crontab -e

```bash
56 0 * * * "/home/lenovo/.acme.sh"/acme.sh --cron --home "/home/lenovo/.acme.sh" --renew-hook "/home/lenovo/.acme.sh/acme.sh --ecc --install-cert -d 'rnmkcy.eu' -d '*.rnmkcy.eu' --key-file /etc/ssl/private/rnmkcy.eu-key.pem --fullchain-file /etc/ssl/private/rnmkcy.eu-fullchain.pem  --reloadcmd 'sudo systemctl reload nginx.service'" > /dev/null
```

## Nginx compilation, PHP8, MariaDB

### Nginx compilé

![](nginx-logo.png){:width="50"}  

Utilisateur avec droits sudo  

Télécharger le bash

```
wget https://static.xoyaz.xyz/files/compilation-nginx-tls1.3.sh
chmod +x compilation-nginx-tls1.3.sh # rendre le bash exécutable
./compilation-nginx-tls1.3.sh        # exécution
```

A la fin de la compilation

```
Versions Nginx OpenSSL
nginx version: nginx/1.20.2
OpenSSL 1.1.1k  25 Mar 2021
```

### Nouvelles configurations nginx

Tout ce qui suit sera pris en compte sur le bash de compilation
{: .prompt-info }

Créer le dossier log nginx

    sudo mkdir -p /var/log/nginx

Créer un fichier de configuration php fastcgi

    sudo nano /etc/nginx/php_fastcgi.conf

```nginx
# regex to split $uri to $fastcgi_script_name and $fastcgi_path
fastcgi_split_path_info ^(.+?\.php)(/.*)$;

# Check that the PHP script exists before passing it
try_files $fastcgi_script_name =404;

# Bypass the fact that try_files resets $fastcgi_path_info
# see: http://trac.nginx.org/nginx/ticket/321
set $path_info $fastcgi_path_info;
fastcgi_param PATH_INFO $path_info;

fastcgi_param REMOTE_USER $remote_user;
fastcgi_param SCRIPT_FILENAME $request_filename;
fastcgi_index index.php;
include fastcgi.conf;
```

Dans les fichiers de configuration 

REMPLACER

```nginx
    location ~ \.php$ {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param REMOTE_USER $remote_user;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $request_filename;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
    }
```

PAR

```nginx
    location ~ \.php$ {
        include php_fastcgi.conf;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
    }
```


### PHP8

![](php8-logo.png){:width="50"}  

Ajout du dépôt sury.org

    sudo -s

Pour installer la version de 8 de php, ajouter le dépôt sury.

```bash
apt install -y lsb-release apt-transport-https ca-certificates wget
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" |tee /etc/apt/sources.list.d/php.list
```

Mise à jour des dépôts :

    apt update && apt upgrade -y

Installation de php8.0, php8.0-fpm, php8.0-sqlite3 et les paquets PHP nécessaires à nextcloud

    apt install php8.0 php8.0-fpm php8.0-sqlite3 php8.0-cli php8.0-gd php8.0-imap php8.0-mysql php8.0-soap php8.0-apcu php8.0-common php8.0-gmp  php8.0-intl php8.0-opcache php8.0-xml php8.0-curl php8.0-igbinary php8.0-readline  php8.0-zip php8.0-bcmath php8.0-imagick php8.0-mbstring php8.0-redis imagemagick

Nextcloud n'accepte pas les versions PHP > 8.0
{: .prompt-warning }

Composer

    wget https://getcomposer.org/download/latest-stable/composer.phar
    chmod +x composer.phar
    sudo mv composer.phar /usr/local/bin/composer

![composer](composer-version.png)

### MariaDB

![](mariadb-logo.png){:width="50"}  
installer les paquets de MariaDB

    sudo apt install mariadb-server

Une fois que l'installation des composants est terminée, tapez la commande suivante pour finaliser la configuration.

    sudo mysql_secure_installation

Tapez Enter directement à la première question car le mot de passe de l'utilisateur root de MariaDB est vide par défaut après l'installation.  
Puis répondez Y à la question suivante pour spécifier le mot de passe de l'utilisateur root de MariaDB qui, une fois de plus, est différent de l'utilisateur root de votre Debian.  
Cet utilisateur root de la base de données aura tous les droits d'accès. Pour des raisons évidentes de sécurité, je vous recommande d'utiliser un mot de passe complexe !  
Et vous pouvez répondre Y à toutes les questions suivantes: les connexions anonymes seront désactivées, ainsi que les connexions root qui se font depuis un serveur autre que le votre...  

## rnmkcy.eu

On va regrouper TLS/SSL, HSTS et OCSP dans le fichier de configuration global `/etc/nginx/tls-hsts-ocsp.conf`

```nginx
    # Certificats Let's Encrypt 
    ssl_certificate /etc/ssl/private/rnmkcy.eu-fullchain.pem;
    ssl_certificate_key /etc/ssl/private/rnmkcy.eu-key.pem;

    # TLS 1.3 only
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;
 
    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;
 
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
 
    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /etc/ssl/private/rnmkcy.eu-fullchain.pem;

    # replace with the IP address of your resolver
    resolver 1.1.1.1;
```

**rnmkcy.eu.conf**

Créer le fichier `/etc/nginx/conf.d/rnmkcy.eu.conf` 

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name rnmkcy.eu;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name rnmkcy.eu;

    root /var/www/default-www;
    index index/ index.php;

    # Certificats Let's Encrypt 
    # TLS 1.3 only
    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    # OCSP stapling
    # replace with the IP address of your resolver
    include /etc/nginx/tls-hsts-ocsp.conf;

    # fichiers de configuration
    include /etc/nginx/conf.d/rnmkcy.eu.d/*.conf;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param REMOTE_USER $remote_user;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $request_filename;
    }
 

}
```

Créer le sous-dossier

    sudo mkdir -p /etc/nginx/conf.d/rnmkcy.eu.d/

Vérifier

    sudo nginx -t

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Recharger nginx

    sudo systemctl reload nginx



### Page d'accueil rnmkcy.eu

Déposer une image `wallpaper.jpg` dans le dossier `/var/www/default-www`  

Créer un fichier `/var/www/default-www/index/`  

``/
<!DOCTYPE/>
/>
<head>
 <meta charset="UTF-8"> 
 <title>rnmkcy.eu</title>
<style type="text/css" media="screen" >
html { 
  margin:0;
  padding:0;
  background: url(wallpaper.jpg) no-repeat center fixed; 
  -webkit-background-size: cover; /* pour anciens Chrome et Safari */
  background-size: cover; /* version standardisée */
}
body { color: white; }
a:link {
  color: grey;
  background-color: transparent;
  text-decoration: none;
}

a {
  text-decoration: underline;
  background-color: transparent;
  color: #a00;
}
a:visited {
  color: #844;
}
a:hover, a:focus, a:active {
  text-decoration: none;
  color: white;
  background: #800;
}


</style>

</head>
<body>

<h1>Serveur rnmkcy.eu</h1>


</body>
</>
```

Lien <https://rnmkcy.eu>   
![](rnmkcy_eu.png){:width="500"}

### Droits dossier /var/www/default-www

On donne les droits groupe $USER au dossier

    sudo chown $USER.www-data -R /var/www/default-www/

### Site statique static.rnmkcy.eu

Le dossier `/home/lenovo/media/static/_site` est mis à jour par le générateur jekyll sur le serveur debian lxcdeb hébergé par PC1/Archlinux

Configuration `/etc/nginx/conf.d/static.rnmkcy.eu.conf`

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name static.rnmkcy.eu;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name static.rnmkcy.eu;

    root /home/lenovo/media/static/_site;
    index index/;

    # Certificats Let's Encrypt 
    # TLS 1.3 only
    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    # OCSP stapling
    # replace with the IP address of your resolver
    include /etc/nginx/tls-hsts-ocsp.conf;

    # fichiers de configuration
    include /etc/nginx/conf.d/rnmkcy.eu.d/*.conf;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param REMOTE_USER $remote_user;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $request_filename;
    }
 
}
```

Vérifier et recharger nginx

    sudo nginx -t
    sudo systemctl reload nginx

Lien <https://static.rnmkcy.eu>   
![](static_rnmkcy_eu.png){:width="500"}

### Diceware, osm-new 

Le fichier de configuration

    /etc/nginx/conf.d/rnmkcy.eu.d/locations.conf

```
location /diceware/ {

    # Path to source
    alias /home/lenovo/media/www/diceware/;
    index index/;
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
}

location /osm-new/ {

    # Path to source
    alias /home/lenovo/media/www/osm-new/;
    index index/;
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
}

```

Vérification et recharge nginx : `sudo nginx -t` et `sudo systemctl reload nginx`  
<http://rnmkcy.eu/diceware>  
<http://rnmkcy.eu/osm-new>  


## Nextcloud

![](nextcloud_logo.png){:width="50"}  

### Installation dernière version

On télécharge la [dernière version nextcloud](https://download.nextcloud.com/server/releases/) et le SHA256

    wget https://download.nextcloud.com/server/releases/latest.tar.bz2
    wget https://download.nextcloud.com/server/releases/latest.tar.bz2.sha256

Vérifier l'intégrité de l'archive téléchargée 

    sha256sum -c latest.tar.bz2.sha256 < latest.tar.bz2

Doit donner le résultat suivant : **latest.tar.bz2: OK**

Décompresser l'archive Nextcloud :

    tar -xvf latest.tar.bz2

Déplacement

    sudo mv nextcloud /var/www/

Supprimez les fichiers et signatures téléchargés :

    rm latest.tar.bz2* 

### Utilisateur nextcloud

Lors du déploiement basique d’un serveur HTTP, l’utilisateur sous lequel fonctionne ce serveur (Apache, Nginx...) est la plupart du temps www-data, nobody ou apache. Cela signifie que si plusieurs sites existent sous la même instance de Nginx, tous utilisent le même utilisateur. Or si l’un des sites s’avère corrompu par un utilisateur malveillant alors l’assaillant peut profiter pleinement de tous les droits de l’utilisateur sous lequel tourne le serveur web. Tous les sites s'avèrent donc vulnérables.

Pour des raisons évidentes de sécurité, il est donc recommandé de cloisonner ces utilisateurs et d'avoir un utilisateur dédié à la gestion du dossier nextcloud. Cet utilisateur aura des droits aussi restreints que possible à ce répertoire.
Par défaut, les fichiers de Nextcloud possèdent les permissions suivantes :

*    répertoires : 755 (permission de lecture, d'écriture et d'exécution pour le propriétaire et permission de lecture et d'exécution pour le groupe et les autres)
*    fichiers : 644 (permission de lecture et d'écriture pour le propriétaire et permission de lecture uniquement pour le groupe et les autres).

Nous allons donc modifier le propriétaire du répertoire `/var/www/nextcloud` et l'attribuer à un nouvel utilisateur dédié : **nextcloud** 

Par ailleurs, Nginx est lancé sous l'utilisateur **www-data** et doit avoir accès en lecture au répertoire `/var/www/nextcloud` pour lire les ressources statiques (HTML, CSS, JS, etc.). Nous allons donc attribuer le répertoire `/var/www/nextcloud` au groupe **www-data**. Enfin nous retirerons toutes les permissions de ce répertoire aux autres utilisateurs.

Créez un utilisateur nextcloud 

    sudo useradd -r nextcloud

Modifiez le propriétaire et le groupe du répertoire /var/www/nextcloud :

    sudo chown -R nextcloud:www-data /var/www/nextcloud

Retirez toutes les permissions aux autres utilisateurs :

    sudo chmod -R o-rwx /var/www/nextcloud

### MariaDB base de données mysql

Mot de passe base nextcloud

    tr -cd '[:alnum:]' < /dev/urandom | fold -w16 | head -n1 

Tout comme pour la gestion du répertoire nextcloud et pour plus de sécurité, vous allez tout d'abord créer un utilisateur MySQL **nextcloud** dédié à la base de données nextcloud, renseigner un mot de passe et ensuite lui donner les droits sur cette base de données.

```
mysql -uroot -pMp_Root_MySql <<-EOF
CREATE DATABASE nextcloud; 
CREATE USER "nextcloud"@"localhost"; 
SET password FOR "nextcloud"@"localhost" = password('Mp_MySql_Nextcloud'); 
GRANT ALL PRIVILEGES ON nextcloud.* TO "nextcloud"@"localhost" IDENTIFIED BY "Mp_MySql_Nextcloud"; 
FLUSH PRIVILEGES;
EOF
```

### PHP

#### PHP pool nextcloud

Création du pool nextcloud `/etc/php/8.0/fpm/pool.d/nextcloud.conf`

    sudo nano /etc/php/8.0/fpm/pool.d/nextcloud.conf 

```conf
[nextcloud]
listen = /run/php/nextcloud.sock

listen.owner = nextcloud
listen.group = www-data

user = nextcloud
group = www-data

pm = ondemand
pm.max_children = 56
pm.process_idle_timeout = 60s
pm.max_requests = 500

env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

#### mémoire pour PHP

ajouter `memory_limit = 512M`  dans le fichier `/etc/php/8.0/fpm/php.ini`  

    sudo sed -i -e "s/^memory_limit \+= \+.*$/memory_limit = 512M/g" /etc/php/8.0/fpm/php.ini   
    sudo sed -i -e "s/^output_buffering \+= \+.*$/output_buffering = 0/g" /etc/php/8.0/fpm/php.ini

Vérification

    cat /etc/php/8.0/fpm/php.ini |egrep "memory_limit|^output_buffering"

```
output_buffering = 0
memory_limit = 512M
```

#### PHP OPcache

*OPcache (qui signifie Optimizer Plus Cache) est introduit depuis la version 5.5.0 de PHP. Il sert à cacher l’opcode de PHP, c’est-à-dire les instructions de bas niveau générées par la machine virtuelle PHP lors de l’exécution d’un script. Autrement dit, le code pré-compilé est stocké en mémoire. Cela évite ainsi l’étape de compilation à chaque requête PHP. De plus, OPcache va optimiser l’exécution du code afin d’en améliorer les performances.*

Alternative A: Éditez le fichier /etc/php/8.0/fpm/php.ini,ajouter les lignes suivantes dans la section [opcache] :

    sudo nano /etc/php/8.0/fpm/php.ini

```ini
[opcache]
opcache.enable = 1
opcache.interned_strings_buffer = 8
opcache.max_accelerated_files = 10000
opcache.memory_consumption = 128
opcache.save_comments = 1
opcache.revalidate_freq = 1
```

La nouvelle configuration sera prise en compte après redémarrage du service PHP-FPM :

    sudo systemctl restart php8.0-fpm.service

### Nginx 


NGINX Configuration <https://docs.nextcloud.com/server/23/admin_manual/installation/nginx/>

* Vous devez insérer le code suivant dans votre fichier de configuration Nginx.  
* Ajustez `server_name`, `root`, `ssl_certificate` et `ssl_certificate_key` en fonction de vos besoins.  
* Assurez-vous que vos certificats SSL sont lisibles par le serveur (voir la documentation du module HTTP SSL de nginx).  
* Faites attention aux sauts de ligne si vous copiez les exemples, car les longues lignes peuvent être coupées pour le formatage de la page.  
* Certains environnements peuvent avoir besoin d'une valeur de 1 pour `cgi.fix_pathinfo` dans leur `php.ini`.

#### /etc/nginx/conf.d/rnmkcy.eu.d/nextcloud.conf

Fichier de configuration `/etc/nginx/conf.d/rnmkcy.eu.d/nextcloud.conf`

```nginx
location ^~ /.well-known {
  # The following 6 rules are borrowed from `.htaccess`

  # The following 2 rules are only needed for the user_webfinger app.
  # Uncomment it if you're planning to use this app.
  #rewrite ^/\.well-known/host-meta\.json  /nextcloud/public.php?service=host-meta-json  last;
  #rewrite ^/\.well-known/host-meta        /nextcloud/public.php?service=host-meta       last;

  location = /.well-known/carddav     { return 301 /nextcloud/remote.php/dav/; }
  location = /.well-known/caldav      { return 301 /nextcloud/remote.php/dav/; }
 
  location = /.well-known/webfinger     { return 301 /nextcloud/index.php$uri; }
  location = /.well-known/nodeinfo      { return 301 /nextcloud/index.php$uri; }

  try_files $uri $uri/ =404;
}

 rewrite ^/nextcloud$ /nextcloud/ permanent;
location ^~ /nextcloud/ {

  # Path to source
  alias /var/www/nextcloud/;


  # Set max upload size
  client_max_body_size 10G;
  fastcgi_buffers 64 4K;

  # Enable gzip but do not remove ETag headers
  gzip on;
  gzip_vary on;
  gzip_comp_level 4;
  gzip_min_length 256;
  gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
  gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application//+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

  # Pagespeed is not supported by Nextcloud, so if your server is built
  # with the `ngx_pagespeed` module, uncomment this line to disable it.
  #pagespeed off;
        # HTTP response headers borrowed from Nextcloud `.htaccess`
        add_header Referrer-Policy                      "no-referrer"   always;
        add_header X-Content-Type-Options               "nosniff"       always;
        add_header X-Download-Options                   "noopen"        always;
        add_header X-Frame-Options                      "SAMEORIGIN"    always;
        add_header X-Permitted-Cross-Domain-Policies    "none"          always;
        add_header X-Robots-Tag                         "none"          always;
        add_header X-XSS-Protection                     "1; mode=block" always;

  # Remove X-Powered-By, which is an information leak
  fastcgi_hide_header X-Powered-By;

  # Specify how to handle directories -- specifying `/nextcloud/index.php$request_uri`
  # here as the fallback means that Nginx always exhibits the desired behaviour
  # when a client requests a path that corresponds to a directory that exists
  # on the server. In particular, if that directory contains an index.php file,
  # that file is correctly served; if it doesn't, then the request is passed to
  # the front-end controller. This consistent behaviour means that we don't need
  # to specify custom rules for certain paths (e.g. images and other assets,
  # `/updater`, `/ocm-provider`, `/ocs-provider`), and thus
  # `try_files $uri $uri/ /nextcloud/index.php$request_uri`
  # always provides the desired behaviour.
  index index.php index/ /nextcloud/index.php$request_uri;

  # Default Cache-Control policy
  expires 1m;

  # Rule borrowed from `.htaccess` to handle Microsoft DAV clients
  location = /nextcloud/ {
      if ( $http_user_agent ~ ^DavClnt ) {
          return 302 /nextcloud/remote.php/webdav/$is_args$args;
      }
  }

  location = /nextcloud/robots.txt {
    allow all;
    log_not_found off;
    access_log off;
  }

  # Rules borrowed from `.htaccess` to hide certain paths from clients
  location ~ ^/nextcloud/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)    { return 404; }
  location ~ ^/nextcloud/(?:\.|autotest|occ|issue|indie|db_|console)                { return 404; }

  # Ensure this block, which passes PHP files to the PHP process, is above the blocks
  # which handle static assets (as seen below). If this block is not declared first,
  # then Nginx will encounter an infinite rewriting loop when it prepends
  # `/nextcloud/index.php` to the URI, resulting in a HTTP 500 error response.
  location ~ \.php(?:$|/) {
    # Required for legacy support
    # https://github.com/nextcloud/documentation/pull/2197#issuecomment-721432337
    # This line fix the ldap admin page
    rewrite ^/nextcloud/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+|.+\/richdocumentscode\/proxy) /nextcloud/index.php$request_uri;
    fastcgi_split_path_info ^(.+?\.php)(/.*)$;
    set $path_info $fastcgi_path_info;
    try_files $fastcgi_script_name =404;
    include fastcgi_params;

    fastcgi_param SCRIPT_FILENAME $request_filename;
    fastcgi_param PATH_INFO $path_info;
    fastcgi_param HTTPS on;
    fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
    fastcgi_param front_controller_active true;     # Enable pretty urls
    fastcgi_param HTTP_ACCEPT_ENCODING "";          # Disable encoding of nextcloud response to inject ynh scripts
    fastcgi_pass unix:/var/run/php/nextcloud.sock;
    fastcgi_intercept_errors on;
    fastcgi_request_buffering off;
  }

  location ~ \.(?:css|js|svg|gif)$ {
    try_files $uri / /nextcloud/index.php$request_uri;
    expires 6M;         # Cache-Control policy borrowed from `.htaccess`
    access_log off;     # Optional: Don't log access to assets
  }

  location ~ \.woff2?$ {
    try_files $uri / /nextcloud/index.php$request_uri;
    expires 7d;         # Cache-Control policy borrowed from `.htaccess`
    access_log off;     # Optional: Don't log access to assets
  }

  location ~ / {
    if ($request_method ~ ^PUT$) {
        rewrite ^ /nextcloud/index.php$request_uri last;
    }
    if ($request_method ~ ^DELETE$) {
        rewrite ^ /nextcloud/index.php$request_uri last;
    }
    try_files $uri / /nextcloud/index.php$request_uri;
  }

}
```

Vérifier et recharger nginx

    sudo nginx -t
    sudo systemctl reload nginx

Recharger php-fpm

    sudo systemctl reload php8.0-fpm

### Paramétrer Nextcloud

Lancer  <https://rnmkcy.eu/nextcloud>  

#### Compte administrateur

Créer un compte administrateur et son mot de passe
admin 
Saisir les informations sur la base , utilisateur et mot de passe   
![](nextcloud-ouestyan001.png){:width="200"} ![](nextcloud.rnmkcy.eu003.png){:width="200"}  

Ne pas installer les applications recommandées  

En cas de renvoi sur l'adresse https://nextcloud/apps/files/  avec le message suivant

```
Hum, nous ne parvenons pas à trouver ce site.

Impossible de se connecter au serveur à l’adresse nextcloud.
```

Relancer avec le lien https://rnmkcy.eu/nextcloud

INFO : Pour réinitialiser le mot de passe admin nextcloud

    sudo -u nextcloud php /var/www/nextcloud/occ user:resetpassword admin

#### Configuration de la région 

**Votre installation n’a pas de préfixe de région par défaut.** , ajouter `'default_phone_region' => 'FR',` dans le  le fichier `/var/www/nextcloud/config/config.php` ou exécuter la commande suivante   
Configuration de la région par défaut pour les numéros de téléphone

    sudo -u nextcloud php /var/www/nextcloud/occ config:system:set default_phone_region --value="FR"

*System config value default_phone_region set to string FR*

#### Thème, Apparence, Messagerie et Stockage

Un thème sombre basé sur **Breeze Dark**    
Aller dans "Applications &rarr; Personnalisation"  
![](cloud_ouestyan_xyz04.png)  
Aller ensuite dans "Paramètres Administration &rarr; Personnaliser l'apparence"  

Lien Nom page d'acceuil : https://rnmkcy.eu/nextcloud  
Logo : ym-64x64.png  
Image de connexion : sunset_1920x1080.jpg  
Logo d'entête : ym01.png  
Favicon : yannick-white.svg  
Breeze dark : Activer le thème + Thème de la connexion  
![](cloud_ouestyan_xyz06.png)  

Actualiser la page du navigateur  
![](rnmkcy_eu02.png)  

**Messagerie**  
Paramètres &rarr; Personnel &rarr; Informations personnelles  
Photo de profil : yannick-green128x128.png  
![](rnmkcy_eu03.png)  

Administration &rarr; Paramètres de base
![](rnmkcy_eu04.png)  
Lancer le test  
![](cloud_ouestyan_xyz09.png)  

**Stockage externe** (paramétrage en admin)  
Applications &rarr; Applications désactivées , activer external storage support  
![](cloud_ouestyan_xyz10.png)  

Paramètres &rarr; Administration Stockage externe  
Nom du dossier : Lenovo   
Configuration : /srv/thinkcentre  
![](rnmkcy_eu05.png)  

#### Cache de données : APCu & Redis 

*APCu permet notamment de mettre en cache les variables PHP et de les stocker en mémoire vive. Redis est un système de gestion de base de données NoSQL avec un système de clef-valeur scalable (s’adapte à la charge). Une des principales caractéristiques de Redis est de conserver l’intégralité des données en RAM. Cela permet d’obtenir d’excellentes performances en évitant les accès disques, particulièrement coûteux.*


Installez les paquets APCu et Redis :

	sudo apt install php8.0-apcu redis-server php8.0-redis 

Il faut ajouter `apc.enable_cli=1` au fichier `/etc/php/8.0/mods-available/apcu.ini` 

```
extension=apcu.so
apc.enable_cli=1
```

Ajoutez les lignes suivantes dans le fichier /var/www/nextcloud/config/config.php :

	sudo nano /var/www/nextcloud/config/config.php

```
  'memcache.local' => '\OC\Memcache\APCu',
  'memcache.locking' => '\OC\Memcache\Redis',
  'memcache.local' => '\OC\Memcache\Redis',
  'redis' => array(
     'host' => 'localhost',
     'port' => 6379,
     'timeout' => 0.0,
     'password' => '',
  ),
```

La nouvelle configuration sera prise en compte après redémarrage du service PHP-FPM :

	sudo systemctl restart php8.0-fpm.service

#### Travaux cron

Vous pouvez programmer des tâches cron de trois façons : en utilisant **AJAX**, **Webcron** ou **cron**. La méthode par défaut consiste à utiliser AJAX. <u>Cependant, la méthode recommandée est d'utiliser **cron**</u>. 

Si systemd est installé sur le système, un timer systemd peut être une alternative à un cronjob.

Cette approche nécessite deux fichiers : `nextcloudcron.service` et `nextcloudcron.timer`  
Créez ces deux fichiers dans `/etc/systemd/system/` 

`/etc/systemd/system/nextcloudcron.service` doit ressembler à ceci :

```
[Unit]
Description=Nextcloud cron.php job

[Service]
User=nextcloud
ExecStart=/usr/bin/php -f /var/www/nextcloud/cron.php
KillMode=process
```

Remplacez l'utilisateur `User` par l'utilisateur de votre serveur http (**www-data** si ce n'est pas **nextcloud**) et `/var/www/nextcloud/cron.php` par l'emplacement de cron.php dans votre répertoire nextcloud.

Le paramètre `KillMode=process` est nécessaire pour que les programmes externes qui sont lancés par la tâche cron continuent à fonctionner après la fin de la tâche cron.

Notez que le fichier **.service** unit n'a pas besoin d'une section `[Install]`. Veuillez vérifier votre installation car nous l'avons recommandé dans les versions précédentes de ce manuel d'administration.

Le fichier `/etc/systemd/system/nextcloudcron.timer` devrait ressembler à ceci :

```
[Unit]
Description=Run Nextcloud cron.php every 5 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min
Unit=nextcloudcron.service

[Install]
WantedBy=timers.target
```

Les parties importantes de l'unité de minuterie sont **OnBootSec** et **OnUnitActiveSec**. **OnBootSec** démarre la minuterie 5 minutes après le démarrage, sinon vous devriez la démarrer manuellement après chaque démarrage. **OnUnitActiveSec** déclenchera une minuterie de 5 minutes après la dernière activation de l'unité de service.

Maintenant, tout ce qui reste à faire est de démarrer et d'activer le minuteur en exécutant cette commande :

    systemctl enable --now nextcloudcron.timer

Lorsque l'option `--now` est utilisée avec enable, l'unité respective sera également démarrée.

Note : Il n'est pas obligatoire de sélectionner l'option Cron dans le menu d'administration pour les travaux en arrière-plan, car une fois que cron.php est exécuté à partir de la ligne de commande ou du service cron, il sera automatiquement réglé sur Cron.  
![](cloud_xoyaz_xyz06.png){:width="600"}
{: .prompt-info }

Vérifier

    systemctl list-timers

```
NEXT                        LEFT           LAST                        PASSED       UNIT                         ACTIVATES
Fri 2021-12-24 15:53:31 UTC 4min 5s left   Fri 2021-12-24 15:48:31 UTC 54s ago      nextcloudcron.timer          nextcloudcron.service
```

Lien <https://rnmkcy.eu/nextcloud>   
![](rnmkcy_eu_nextcloud.png){:width="500"}

## CodeIgniter4

### Création projet CodeIgniter avec Composer

    composer create-project codeigniter4/appstarter /var/www/default-www/ci

Ceci créera un nouveau projet CodeIgniter dans un dossier nommé `ci-secure-api`. Une fois l'installation terminée, accédez au dossier de projet nouvellement créé à partir du terminal et exécutez l'application sur le serveur de développement local fourni avec CodeIgniter. Pour ce faire, utilisez la commande suivante :

    cd /var/www/default-www/ci && php spark serve

```
CodeIgniter v4.1.6 Command Line Tool - Server Time: 2022-01-04 23:27:57 UTC-06:00

CodeIgniter development server started on http://localhost:8080
Press Control-C to stop.
[Wed Jan  5 06:27:57 2022] PHP 8.0.14 Development Server (http://localhost:8080) started
```

L'appel http se fait depuis le poste archlinux   

Accéder http://localhost:8080 via le navigateur du poste archlinux  
On utilise la redirection port SSH

Vérification,ouvrir un terminal sur le client linux qui dispose des clés ssh et lancer la commande

    ssh -L 9000:localhost:8080 lenovo@192.168.0.145 -p 55145 -i /home/yann/.ssh/lenovo-ed25519

http://localhost:9000  
![](api_restfull_001.png){:width=600}

### Variables environnement

Maintenant que CodeIgniter est installé et en cours d'exécution, l'étape suivante consiste à fournir des variables d'environnement qui seront utilisées par notre application.  
Arrêtez l'exécution de l'application en appuyant sur les touches CTRL + C du clavier et effectuez une copie du fichier .env nommé .env à l'aide de la commande ci-dessous :

    cd /var/www/default-www/ci
    cp env .env
    nano .env

CodeIgniter démarre en mode production par défaut. Dans le cadre de ce tutoriel, nous allons le passer en mode développement. Pour ce faire, annuler le commentaire de la ligne ci-dessous et définissez-la sur development :

```
CI_ENVIRONMENT = development
# Configurer nginx pour un accès en sous répertoire
app.baseURL = 'https://rnmkcy.eu/ci/'
```

Créer le fichier de configuration nginx `/etc/nginx/conf.d/rnmkcy.eu.d/ci.conf`

```nginx
location /ci/ {
alias /var/www/default-www/ci/public/;

	try_files $uri $uri/ /index.php;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param REMOTE_USER $remote_user;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param SCRIPT_FILENAME $request_filename;
    }

}
```

Modifier les droits du dossier

    sudo chmod 775 -R /var/www/default-www/ci/writable/

Recharger nginx

    sudo systemctl reload nginx

http://rnmkcy.eu/ci  
![](api_restfull_001.png){:width=600}

## Connexion et authentification JWT avec PHP MYSQL

[Connexion et authentification JWT avec PHP MYSQL](/posts/Authentification_PHP_JWT_REST_API_connexion_et_inscription/)  
Dossier de travail : `~/media/php-jwt-lenovo`  

```
1-users.sql   3-lib-user.php  5-verify.php   6b-login.js   7-pages.php  8b-api.php     composer.lock
2-config.php  4-php-jwt.txt   6a-login/  6c-login.css  8a-api/  
```

Créer une base de données "dbapi" 

    mysql -uroot -pRenvoieFavoriFoulonIngambeParmi -e "CREATE DATABASE dbapi; GRANT ALL ON dbapi.* TO 'dbapi'@'localhost' IDENTIFIED BY 'GliomeResteCadranMarmite'; FLUSH PRIVILEGES;"

Modifier le fichier  environnement `.env`

```
#--------------------------------------------------------------------
# DATABASE
#--------------------------------------------------------------------

database.default.hostname = localhost
database.default.database = dbapi
database.default.username = dbapi
database.default.password = GliomeResteCadranMarmite
database.default.DBDriver = MySQLi # this is the driver for a MySQL connection. There are also drivers available for postgres & SQLite3.
# database.default.DBPrefix =
```

Créer deux fichiers de migration nommés `add_client` et `add_user` avec la commane `php spark migrate:create`  
Modifier les deux fichiers : 

APPPATH/Database/Migrations/2022-01-05-074219_AddClient.php

```php
<?php
use CodeIgniter\Database\Migration;

class AddClient extends Migration
{
    public function up()
    {
        $this->forge->addField([
            'id' => [
                'type' => 'INT',
                'constraint' => 5,
                'unsigned' => true,
                'auto_increment' => true,
            ],
            'name' => [
                'type' => 'VARCHAR',
                'constraint' => '100',
                'null' => false
            ],
            'email' => [
                'type' => 'VARCHAR',
                'constraint' => '100',
                'null' => false,
                'unique' => true
            ],
            'retainer_fee' => [
                'type' => 'INT',
                'constraint' => 100,
                'null' => false,
                'unique' => true
            ],
            'updated_at' => [
                'type' => 'datetime',
                'null' => true,
            ],
        'created_at datetime default current_timestamp',
        ]);
        $this->forge->addPrimaryKey('id');
        $this->forge->createTable('client');
    }

    public function down()
    {
        $this->forge->dropTable('client');
    }
}

```

APPPATH/Database/Migrations/2022-01-05-074328_AddUser.php

```php
<?php

use CodeIgniter\Database\Migration;

class AddUser extends Migration
{
    public function up()
    {
        $this->forge->addField([
            'id' => [
                'type' => 'INT',
                'constraint' => 5,
                'unsigned' => true,
                'auto_increment' => true,
            ],
            'name' => [
                'type' => 'VARCHAR',
                'constraint' => '100',
                'null' => false
            ],
            'email' => [
                'type' => 'VARCHAR',
                'constraint' => '100',
                'null' => false,
                'unique' => true
            ],
            'password' => [
                'type' => 'VARCHAR',
                'constraint' => '255',
                'null' => false,
                'unique' => true
            ],
            'updated_at' => [
                'type' => 'datetime',
                'null' => true,
            ],
            'created_at datetime default current_timestamp',
        ]);
        $this->forge->addPrimaryKey('id');
        $this->forge->createTable('user');
    }

    public function down()
    {
        $this->forge->dropTable('user');
    }
}
```

Exécuter les migrations à l’aide de la commande ci-dessous :

    php spark migrate

Pour faciliter le développement, seedez des données client factices dans votre base de données.

    php spark make:seeder

L'outil CLI requêtera le nom **ClientSeeder**. Un fichier *ClientSeeder.php* sera créé dans le répertoire App/Database/Seeds (`app/Database/Seeds/ClientSeeder.php`). Ouvrez le fichier et remplacez son contenu par ce qui suit :

```php
<?php

namespace App\Database\Seeds;

use CodeIgniter\Database\Seeder;
use Faker\Factory;

class ClientSeeder extends Seeder
{
    public function run()
    {
        for ($i = 0; $i < 10; $i++) { //to add 10 clients. Change limit as desired
            $this->db->table('client')->insert($this->generateClient());
        }
    }

    private function generateClient(): array
    {
        $faker = Factory::create();
        return [
            'name' => $faker->name(),
            'email' => $faker->email,
            'retainer_fee' => random_int(100000, 100000000)
        ];
    }
}
```

Peuplez la base de données avec des clients factices à l'aide de la commande suivante :

    php spark db:seed ClientSeeder

Accès contenu base

```
mysql -udbapi -pGliomeResteCadranMarmite dbapi -e "SELECT * FROM client"
+----+---------------------+------------------------------+--------------+------------+---------------------+
| id | name                | email                        | retainer_fee | updated_at | created_at          |
+----+---------------------+------------------------------+--------------+------------+---------------------+
|  1 | Lela Anderson       | eliseo32@lockman.info        |     19211006 | NULL       | 2022-01-05 12:04:27 |
|  2 | Yolanda Braun       | ruecker.gisselle@fritsch.com |     85672968 | NULL       | 2022-01-05 12:04:27 |
|  3 | Cielo Pollich       | hope.schultz@murray.net      |     96499819 | NULL       | 2022-01-05 12:04:27 |
|  4 | Hilda Ferry Sr.     | martin20@haag.com            |     32245877 | NULL       | 2022-01-05 12:04:27 |
|  5 | Norris Krajcik      | kiehn.jamarcus@gmail.com     |     22773108 | NULL       | 2022-01-05 12:04:27 |
|  6 | Samson Ratke DDS    | omitchell@lindgren.net       |      3240460 | NULL       | 2022-01-05 12:04:27 |
|  7 | Mr. Randi Boyer PhD | gspencer@wilderman.com       |     59043947 | NULL       | 2022-01-05 12:04:27 |
|  8 | Dora Eichmann       | craig66@ebert.net            |     14032907 | NULL       | 2022-01-05 12:04:27 |
|  9 | Arjun Pagac         | haylee53@hotmail.com         |     31645406 | NULL       | 2022-01-05 12:04:27 |
| 10 | Miss Dolly Kuhn     | eudora.veum@hotmail.com      |     10474105 | NULL       | 2022-01-05 12:04:27 |
+----+---------------------+------------------------------+--------------+------------+---------------------+

mysql -udbapi -pGliomeResteCadranMarmite dbapi -e "SELECT * FROM user"
"Aucun utilsateur"

mysql -udbapi -pGliomeResteCadranMarmite dbapi -e "SELECT * FROM migrations"
+----+-------------------+------------+---------+-----------+------------+-------+
| id | version           | class      | group   | namespace | time       | batch |
+----+-------------------+------------+---------+-----------+------------+-------+
|  1 | 2022-01-05-074219 | \AddClient | default | App       | 1641380415 |     1 |
|  2 | 2022-01-05-074328 | \AddUser   | default | App       | 1641380415 |     1 |
+----+-------------------+------------+---------+-----------+------------+-------+
```


Création de deux modèles : un pour l'utilisateur `UserModel.php` et un autre pour le client `ClientModel.php`

Dans `app/Models/UserModel.php`, ajoutez ce qui suit :

```php
<?php

namespace App\Models;

use CodeIgniter\Model;
use Exception;

class UserModel extends Model
{
    protected $table = 'user';
    protected $allowedFields = [
        'name',
        'email',
        'password',
    ];
    protected $updatedField = 'updated_at';

    protected $beforeInsert = ['beforeInsert'];
    protected $beforeUpdate = ['beforeUpdate'];

    protected function beforeInsert(array $data): array
    {
        return $this->getUpdatedDataWithHashedPassword($data);
    }

    protected function beforeUpdate(array $data): array
    {
        return $this->getUpdatedDataWithHashedPassword($data);
    }

    private function getUpdatedDataWithHashedPassword(array $data): array
    {
        if (isset($data['data']['password'])) {
            $plaintextPassword = $data['data']['password'];
            $data['data']['password'] = $this->hashPassword($plaintextPassword);
        }
        return $data;
    }

    private function hashPassword(string $plaintextPassword): string
    {
        return password_hash($plaintextPassword, PASSWORD_BCRYPT);
    }
                                      
    public function findUserByEmailAddress(string $emailAddress)
    {
        $user = $this
            ->asArray()
            ->where(['email' => $emailAddress])
            ->first();

        if (!$user) 
            throw new Exception('User does not exist for specified email address');

        return $user;
    }
}
```

Les fonctions *beforeInsert* et *beforeUpdate* vous permettent d'effectuer une opération sur l'entité *User* avant de l'enregistrer dans la base de données.  
Dans ce cas, le mot de passe de l'utilisateur est hashé avant d'être enregistré dans la base de données.

Dans `app/Models/ClientModel.php`, ajoutez ce qui suit :

```php
<?php

namespace App\Models;

use CodeIgniter\Model;
use Exception;

class ClientModel extends Model
{
    protected $table = 'client';
    protected $allowedFields = [
        'name',
        'email',
        'retainer_fee'
    ];
    protected $updatedField = 'updated_at';

    public function findClientById($id)
    {
        $client = $this
            ->asArray()
            ->where(['id' => $id])
            ->first();

        if (!$client) throw new Exception('Could not find client for specified ID');

        return $client;
    }
}
```

Le champ `$table` permet au modèle de savoir avec quelle table de base de données il fonctionne principalement. `$allowedFields` permet au modèle de savoir quelles colonnes de la table peuvent être mises à jour. La fonction `findClientById` fournit une abstraction propre pour extraire un client de la base de données en fonction de l'`id` fourni.

<u>Une fois les modèles et la base de données implémentés, les utilisateurs peuvent être ajoutés et authentifiés</u>. Les utilisateurs autorisés peuvent également interagir avec la clientèle actuelle.

Les [Web JSON Web Tokens](https://jwt.io/) seront utilisés pour authentifier les utilisateurs et empêcher les utilisateurs non autorisés d'afficher la liste des clients.

    composer require firebase/php-jwt

Une fois l'installation terminée, ajoutez les éléments suivants à votre fichier `.env` :


```
#--------------------------------------------------------------------
# JWT_SECRET_KEY key est la clé secrète utilisée par l'application pour signer JWTS.
# JWT_TIME_TO_LIVE indique la période de validité d'un JWT signé (en millisecondes)
#--------------------------------------------------------------------

JWT_SECRET_KEY=mIKm1jDusb9Z7BfImUuBIcEfpso4yVV5TWtb3Jxf
JWT_TIME_TO_LIVE=3600
```

Ensuite, créez une <u>fonction d'aide pour obtenir la clé secrète</u> dans la classe Services `app/Config/Services.php` 

```php
<?php

namespace Config;

use CodeIgniter\Config\BaseService;

/**
 * Fichier de configuration des services.
 *
 * Les services sont simplement d'autres classes/libraries que le système utilise
 * pour faire son travail. Ceci est utilisé par CodeIgniter pour permettre au noyau de la structure
 * du framework d'être remplacé facilement sans affecter l'utilisation dans le
 * le reste de votre application.
 *
 * Ce fichier contient tous les services spécifiques à l'application, ou les services prioritaires
 * dont vous pourriez avoir besoin. Un exemple a été inclus avec le format général de la méthode
 * général que vous devez utiliser pour vos méthodes de service. Pour plus d'exemples,
 * consultez le fichier principal Services à l'adresse system/Config/Services.php.
*/
class Services extends BaseService
{
    /*
     * public static function example($getShared = true)
     * {
     *     if ($getShared) {
     *         return static::getSharedInstance('example');
     *     }
     *
     *     return new \CodeIgniter\Example();
     * }
     */
	public static function getSecretKey(){
	    return getenv('JWT_SECRET_KEY');
	} 

}
```

Pour <u>faciliter la génération et la vérification des tokens</u>, un [fichier Helper](https://codeigniter.com/user_guide/general/helpers/) sera créé. Cela nous permet de séparer les préoccupations dans notre application. Dans le répertoire *App/Helpers* (`app/Helpers/`), créez un fichier nommé `jwt_helper.php`. Votre fichier devrait ressembler à ceci :

app/Helpers/jwt_helper.php

```php
<?php

use App\Models\UserModel;
use Config\Services;
use Firebase\JWT\JWT;

function getJWTFromRequest($authenticationHeader): string
{
    if (is_null($authenticationHeader)) { //JWT is absent
        throw new Exception('Missing or invalid JWT in request');
    }
    //JWT is sent from client in the format Bearer XXXXXXXXX
    return explode(' ', $authenticationHeader)[1];
}

function validateJWTFromRequest(string $encodedToken)
{
    $key = Services::getSecretKey();
    $decodedToken = JWT::decode($encodedToken, $key, ['HS256']);
    $userModel = new UserModel();
    $userModel->findUserByEmailAddress($decodedToken->email);
}

function getSignedJWTForUser(string $email)
{
    $issuedAtTime = time();
    $tokenTimeToLive = getenv('JWT_TIME_TO_LIVE');
    $tokenExpiration = $issuedAtTime + $tokenTimeToLive;
    $payload = [
        'email' => $email,
        'iat' => $issuedAtTime,
        'exp' => $tokenExpiration,
    ];

    $jwt = JWT::encode($payload, Services::getSecretKey());
    return $jwt;
}
```

La fonction `getJWTFromRequest` vérifie l'en-tête [Authorization](https://stackoverflow.com/questions/5110361/what-is-the-http-authorization-environment-variable) de la requête entrante et renvoie la valeur du token. Si l'en-tête est manquant, une exception est déclenchée. Celle-ci entraîne à son tour le renvoi d'une réponse `HTTP_UNAUTHORIZED` (401).

La fonction validateJWTFromRequest utilise le token obtenu au moyen de la fonction `getJWTFromRequest`. Elle décode ce token afin d'obtenir l'e-mail pour lequel la clé a été générée. Elle tente ensuite de trouver un utilisateur avec cette adresse e-mail dans la base de données. Si l'utilisateur n'a pas été trouvé, le modèle d'utilisateur déclenche une exception qui est détectée et renvoyée à l'utilisateur sous la forme d'une réponse `HTTP_UNAUTHORIZED` (401).

La fonction `getSignedJWTForUser` est utilisée pour générer un token pour un utilisateur authentifié. Le JWT codé contient les détails suivants :

*    Adresse e-mail de l'utilisateur authentifié. Elle est utilisée dans les requêtes ultérieures pour valider la source de la requête.
*    Heure à laquelle le token a été généré (iat).
*    Heure d'expiration du token (exp). Elle est obtenue en ajoutant la valeur JWT_TIME_TO_LIVE de notre fichier .env à l'heure actuelle.


**Filtres**  
Dans le répertoire *App/Filters* (`app/Filters/`), créez un fichier nommé `JWTAuthenticationFilter.php`. <u>Ce filtre permet de vérifier l'API du JWT avant de transmettre la requête au contrôleur</u>. Si aucun JWT n'est fourni ou si le JWT fourni a expiré, une réponse HTTP_UNAUTHORIZED (401) est renvoyée avec un message d'erreur approprié. Ajoutez ce qui suit à votre fichier :

app/Filters/JWTAuthenticationFilter.php

```php
<?php

namespace App\Filters;

use CodeIgniter\API\ResponseTrait;
use CodeIgniter\Filters\FilterInterface;
use CodeIgniter\HTTP\RequestInterface;
use CodeIgniter\HTTP\ResponseInterface;
use Config\Services;
use Exception;

class JWTAuthenticationFilter implements FilterInterface
{
    use ResponseTrait;

    public function before(RequestInterface $request, $arguments = null)
    {
        $authenticationHeader = $request->getServer('HTTP_AUTHORIZATION');

        try {

            helper('jwt');
            $encodedToken = getJWTFromRequest($authenticationHeader);
            validateJWTFromRequest($encodedToken);
            return $request;

        } catch (Exception $e) {

            return Services::response()
                ->setJSON(
                    [
                        'error' => $e->getMessage()
                    ]
                )
                ->setStatusCode(ResponseInterface::HTTP_UNAUTHORIZED);

        }
    }

    public function after(RequestInterface $request,
                          ResponseInterface $response,
                          $arguments = null)
    {
    }
}
```

Comme vous pouvez le voir, JWT Helper est chargé en premier, puis les fonctions `getJWTFromRequest` et `validateJWTFromRequest` sont utilisées pour s'assurer que la requête provient d'un utilisateur authentifié avec un token valide.

Enregistrez votre filtre JWTAuthentication et spécifiez le chemin à protéger. Cette opération s'effectue dans le fichier *App/Config/Filters.php*. <u>Mettez à jour les tableaux `$aliases` et `$filters`</u> comme suit :

```php
<?php 
namespace Config;

use App\Filters\JWTAuthenticationFilter;
use CodeIgniter\Config\BaseConfig;

class Filters extends BaseConfig
{
    public $aliases = [
        'csrf' => CSRF::class,
        'toolbar' => DebugToolbar::class,
        'honeypot' => \CodeIgniter\Filters\Honeypot::class,
        'auth' => JWTAuthenticationFilter::class // add this line
    ];

    // global filters
    // method filters
    public $filters = [
      'auth' => [
        'before' => [
            'client/*',
            'client'
      ],
    ]
  ];
}
```

En ajoutant ces éléments, la fonction `before` dans *JWTAuthenticationFilter.php* est appelée chaque fois qu'une requête est envoyée à un endpoint commençant par le **client**. Cela signifie que le contrôleur reçoit/traite la requête uniquement si son en-tête contient un token valide.

Même si nous n'avons pas de contrôleur, nous pouvons vérifier que notre application fonctionne jusqu'à présent.  


Pour enregistrer un nouvel utilisateur avec succès, les champs suivants sont obligatoires :

*    Un nom.
*    Une adresse e-mail dans un format valide ne comportant pas moins de 8 caractères et pas plus de 255 caractères.
*    Un mot de passe de 8 caractères minimum et de 255 caractères maximum.

La requête entrante est vérifiée par rapport aux règles spécifiées. Les requêtes non valides sont ignorées avec un code (400) `HTTP_BAD_REQUEST` et un message d'erreur. Si la requête est valide, les données utilisateur sont enregistrées et un token est renvoyé avec les détails enregistrés de l'utilisateur (à l'exception du mot de passe). La réponse `HTTP_CREATED` (201) informe le client qu'une nouvelle ressource a été créée.

En effectuant une requête POST sur le endpoint du registre (http://localhost:8080/auth/register) avec un nom (name), une adresse e-mail (e-mail) et un mot de passe (password) valides, vous obtenez une réponse similaire à celle présentée ci-dessous :

    curl -d "name=Slainie Deblois&email=SlainieDeblois@armyspy.com&password=eiT8bekohb" http://localhost:8080/auth/register

```
{
    "message": "User authenticated successfully",
    "user": {
        "id": "1",
        "name": "Slainie Deblois",
        "email": "SlainieDeblois@armyspy.com",
        "updated_at": null,
        "created_at": "2022-01-05 13:48:47"
    },
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6IlNsYWluaWVEZWJsb2lzQGFybXlzcHkuY29tIiwiaWF0IjoxNjQxMzg2OTI3LCJleHAiOjE2NDEzOTA1Mjd9.u-DHhP9IQfvyq-uNQREb-PBafd696HO4I84jq2swlKg"
}
```

## 2FA Authentification

### Installation

Suivre la procédure d'installation sur ce lien[PHP nginx - Portail d'authentification web authentification à deux facteurs (2FA).](http://lxcdeb:8080/2021/12/29/PHP_Authentification_2FA+nginx_auth/)

### Configurations nginx

```
/etc/nginx/conf.d/
├── rnmkcy.eu.conf
├── rnmkcy.eu.d
│   ├── locations.conf
│   ├── nextcloud.conf
│   └── static.conf
└── static.rnmkcy.eu.conf
```

#### rnmkcy.eu.conf

```nginx
map $request_uri $loggable {
    /submit/api/submit 1;
    default 0;
}

 log_format phpcookie '$remote_addr - $remote_user [$time_local]  ' '"$request" $status $body_bytes_sent ' '"$http_referer" "$http_user_agent" - "$http_cookie"';


server {
    listen 80;
    listen [::]:80;
    server_name rnmkcy.eu;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name rnmkcy.eu;

    root /var/www/default-www;
    index index/ index.php;

    # Certificats Let's Encrypt 
    # TLS 1.3 only
    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    # OCSP stapling
    # replace with the IP address of your resolver
    include /etc/nginx/tls-hsts-ocsp.conf;

    #
    # Le module Nginx auth_request permet l'authentification de chaque requête par rapport à une sous-requête interne spécifiée comme une URL. 
    # La sous-requête doit répondre avec le code d'état HTTP approprié :
    #     HTTP 401 si l'authentification a échoué
    #     HTTP 200 si l'authentification a réussi
    #
#    auth_request /twofactorauth/nginx/auth.php;
#    error_page 401 = @error401;
#    
#    location @error401 {
#	return 302 $scheme://$host/twofactorauth/login/login.php?from=$uri;
#    }
    error_page 401 = @error401;

    location @error401 {
      return 302 $scheme://$host/twofactorauth/login/login.php?from=$uri;
    }


    location = /twofactorauth/nginx/auth.php {
        include php_fastcgi.conf;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
        fastcgi_param  CONTENT_LENGTH "";
    }

    location = /twofactorauth/login/ {
	allow all;
	auth_request off;
        include php_fastcgi.conf;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
    }

    location /twofactorauth/ {
      index index.php;
    }
    
    location /twofactorauth/db/ {
	deny all;
    }

    # fichiers de configuration
    include /etc/nginx/conf.d/rnmkcy.eu.d/*.conf;

    location ~ \.php$ {
        include php_fastcgi.conf;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
    }

    location / {
      access_log /var/log/nginx/rnmkcy.log phpcookie if=$loggable;    
      auth_request /twofactorauth/nginx/auth.php;
    }


}
```

### locations.conf 

```nginx
location /diceware {

    # Path to source
    alias /home/lenovo/media/www/diceware/;
    index index/;
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
     auth_request /twofactorauth/nginx/auth.php;
     error_page 401 =401 $scheme://$host/twofactorauth/login/login.php?from=$uri;

}

location /osm-new {
    # Path to source
    alias /home/lenovo/media/www/osm-new/;
    index index/;
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
     auth_request /twofactorauth/nginx/auth.php;
     error_page 401 =401 $scheme://$host/twofactorauth/login/login.php?from=$uri;

}
```

## Audio 

### Navidrome

Les étapes suivantes ont été testées sur Ubuntu 18.04 et devraient fonctionner sur toutes les versions 16.04 et supérieures ainsi que sur les autres distros basées sur Debian. Tout au long de ces instructions, les commandes auront des espaces réservés pour l'utilisateur (`<user>`) et le groupe (`<group>`) sous lesquels vous souhaitez exécuter Navidrome et le chemin du dossier de musique (`<library_path>`). Si vous utilisez une médiathèque existante, assurez-vous que l'utilisateur a les droits sur la médiathèque.

#### Prérequis

**Conditions préalables à la mise à jour et à l'installation**  
Assurez-vous que votre système est à jour et installez ffmpeg.

    sudo apt update && sudo apt upgrade
    sudo apt install libtag1-dev ffmpeg

#### Utilisateur "navidrome"

Par défaut, la commande useradd ne crée pas de répertoires de base, mais pour un démon, je vous recommande d'utiliser l'option système et de remplacer le shell par un shell inexistant afin que personne ne puisse se connecter à ce compte (sous ssh par exemple):

    sudo useradd -r -s /bin/false navidrome

#### Créer une structure de répertoire

Créez un répertoire pour stocker l'exécutable Navidrome et un répertoire de travail avec les permissions appropriées.

    sudo install -d -o navidrome -g navidrome /opt/navidrome
    sudo install -d -o navidrome -g navidrome /var/lib/navidrome

#### Obtenir Navidrome

**Sur le git officiel**

Téléchargez la dernière version depuis la [page des versions](https://github.com/navidrome/navidrome/releases), extrayez le contenu dans le répertoire exécutable et définissez les autorisations pour les fichiers. (Remplacez l'URL ci-dessous par celle de la page des versions) :

    wget https://github.com/navidrome/navidrome/releases/download/v0.45.1/navidrome_0.45.1_Linux_arm64.tar.gz -O Navidrome.tar.gz
    sudo tar -xvzf Navidrome.tar.gz -C /opt/navidrome/
    sudo chown -R navidrome:navidrome /opt/navidrome

**Version compilée**

[Archlinux Debian - Compilation go Audio Navidrome](/posts/Archlinux_Debian-Compilation_go_Audio_Navidrome/)

Copier le fichier **navidrome** dans le répertoire `/opt/navidrome/` et modifier les droits

    sudo cp navidrome /opt/navidrome/
    sudo chown -R navidrome:navidrome /opt/navidrome
    
#### Créer le fichier de configuration

Dans le répertoire de travail, créer un nouveau fichier nommé `navidrome.toml` ( `sudo nano /var/lib/navidrome/navidrome.toml` ) avec les paramètres suivants.

    MusicFolder = "/home/lenovo/media/music"
    ND_PLAYLISTSPATH = "Playlists"


Pour d'autres options de configuration, voir la page des [options de configuration](https://www.navidrome.org/docs/usage/configuration-options/).

#### Créer une unité Systemd navidrome.service  

Créez un nouveau fichier sous `/etc/systemd/system/` nommé `navidrome.service` avec les données suivantes.

    /etc/systemd/system/navidrome.service

```systemd
[Unit]
Description=Navidrome Music Server and Streamer compatible with Subsonic/Airsonic
After=remote-fs.target network.target
AssertPathExists=/var/lib/navidrome

[Install]
WantedBy=multi-user.target

[Service]
User=navidrome
Group=navidrome
Type=simple
ExecStart=/opt/navidrome/navidrome --configfile "/var/lib/navidrome/navidrome.toml"
WorkingDirectory=/var/lib/navidrome
TimeoutStopSec=20
KillMode=process
Restart=on-failure

# See https://www.freedesktop.org/software/systemd/man/systemd.exec/
DevicePolicy=closed
NoNewPrivileges=yes
PrivateTmp=yes
PrivateUsers=yes
ProtectControlGroups=yes
ProtectKernelModules=yes
ProtectKernelTunables=yes
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
RestrictNamespaces=yes
RestrictRealtime=yes
SystemCallFilter=~@clock @debug @module @mount @obsolete @reboot @setuid @swap
ReadWritePaths=/var/lib/navidrome

# You can uncomment the following line if you're not using the jukebox This
# will prevent navidrome from accessing any real (physical) devices
#PrivateDevices=yes

# You can change the following line to `strict` instead of `full` if you don't
# want navidrome to be able to write anything on your filesystem outside of
# /var/lib/navidrome.
ProtectSystem=full

# You can uncomment the following line if you don't have any media in /home/*.
# This will prevent navidrome from ever reading/writing anything there.
#ProtectHome=true

# You can customize some Navidrome config options by setting environment variables here. Ex:
#Environment=ND_BASEURL="/navidrome"
```

**Démarrez le service Navidrome**  
Rechargez le démon de service, démarrez le service nouvellement créé, et vérifiez qu'il a démarré correctement.

    sudo systemctl daemon-reload
    sudo systemctl start navidrome.service
    sudo systemctl status navidrome.service

Si le service a démarré correctement, vérifiez que vous pouvez accéder à http://localhost:4533.  
Ouvrir un terminal sur le client linux qui dispose des clés ssh et lancer la commande

    ssh -L 9500:localhost:4533  lenovo@192.168.0.145 -p 55145 -i /home/yann/.ssh/lenovo-ed25519

Ouvrir un navigateur sur le client et saisir [localhost:9500](URL) pour afficher le serveur audio  
![](zic_rnmkcy_eu-1.png){:width="400"}  
![](zic_rnmkcy_eu-2.png){:width="400"}

Démarrez Navidrome au démarrage

    sudo systemctl enable navidrome.service

#### Proxy nginx zic.rnmkcy.eu (navidrome)

Le domaine “zic.rnmkcy.eu” est activé avec les certificats let’s encrypt

Le fichier de configuration nginx `/etc/nginx/conf.d/zic.rnmkcy.eu.conf`

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name zic.rnmkcy.eu;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name zic.rnmkcy.eu;

    # Certificats Let's Encrypt 
    # TLS 1.3 only
    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    # OCSP stapling
    # replace with the IP address of your resolver
    include /etc/nginx/tls-hsts-ocsp.conf;

    location / { 
        proxy_pass              http://127.0.0.1:4533;
    } 

}
```

Lien <https://zic.rnmkcy.eu>

## SSMTP

[SSMTP client SMTP pour envoi de messages](/posts/SSMTP_client_SMTP_pour_envoi_de_messages/)
