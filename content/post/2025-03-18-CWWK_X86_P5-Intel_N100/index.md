+++
title = 'CWWK X86-P5 Intel N100 Serveur Debian 12'
date = 2025-08-12
categories = ['debian']
+++
*CWWK carte mère x86-P5 sous processeur Alder Lake-N + module de 4 ports NVMe PCIe x1 + 2 ports Ethernet 2.5 Gigabit avec des fonctions Wake On Lan et de démarrage automatique après reprise du courant, ou programmable. Un port M.2 pour ajouter un module Wi-Fi*

## Description matériel CWWK X86-P5

<mark>CWWK X86-P5 Intel N100 Alder Lake-N + 5 ports NVMe + 2 ports Ethernet 2.5Ghz</mark>  

* [CWWK X86-P5 P6 Specification User Manual](/files/CWWK X86-P5 P6 Specification User Manual.pdf)
* [CW-X86-P5 P6 BIOS Manual](/files/CW-X86-P5 P6 BIOS Manual.pdf)

## Serveur Debian 12

![](debian12-logob.png){: width="200" .normal} 

### Installation avec clé USB ISO

Connecter la machine cwwk sur le réseau  
Brancher un clavier USB  
Connecter un écran HDMI  
Insérer clé USB contenant image ISO **debian-12.1.0-amd64-netinst.iso**  
Brancher le jack d'alimentation

Accès au BIOS par touche clavier DEL  
Dans l'onglet "Boot" --> `Boot Option #1 [USB Device]`  
Touche F10  pour sauver

Installation debian

Réseau: enp3s0

Machine : cwwk  
root : root49600 (temporaire)  
yick : yick49600 (temporaire)  

Disque NVMe 128Go /dev/nvme0n1  
Partitions LVM avec home séparé et 100Go utilisé  
Serveur SSH  
Utilitaires usuels du système  

La machine reboot à la fin de l'installation  
Se connecter avec l'utilisateur "yick"  
Relever l'adresse ip : `ip a` , exemple 192.168.0.26


On peut également se connecter via ssh : `ssh yick@192.168.0.26`  


Passer en `su` , installer sudo

    su
    apt install sudo
    echo "yick     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/yick

### Interface Réseau enp3s0

#### Ip V4 statiques

Désactiver NetworkManager

    systemctl stop NetworkManager
    systemctl disable NetworkManager

configurer une adresse IPv4 en statique  

    nano /etc/network/interfaces

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp3s0
# iface enp3s0 inet dhcp
iface enp3s0 inet static
    address 192.168.0.205
    netmask 255.255.255.0
    gateway 192.168.0.254
```


Fichier résolution DNS `/etc/resolv.conf`

```
nameserver 1.1.1.1
nameserver 9.9.9.9
```

Redémarrer l'ordinateur

    sudo reboot

#### IP V6 DHCP (DEFAUT)

Configuration `/etc/network/interfaces`

```
iface enp3s0 inet6 dhcp
```

Adresse IPV6 : 2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45

#### IP V6 statique (PB DNS)

*Route de lien local. Avec le protocole IPv6, la notion de trafic de diffusion (__broadcast__) disparaît. C'est le trafic __anycast__ associé au __Neighbor Discovery Protocol__ qui permet de contacter les hôtes du voisinage réseau. Pour que les messages ICMPv6 du protocole NDP puissent être échangés, il est nécessaire de disposer d'une route vers le réseau auquel une interface active est raccordé. Cette route utilise toujours le préfixe fe80::/64.*

Relever la route de lien local pour le Next Hop:  `ip a |grep "inet6 fe80"`  
`inet6 fe80::aab8:e0ff:fe04:ec45/64 scope link`

**FreeBox**  
Se connecter sur la freebox, paramètres avancés -> Configuration IPV6  
Relever Adresse IPv6 lien local: `fe80::8e97:eaff:fe39:66d6`  

Configurer le préfixe secondaire : `2a01:e0a:9c8:2085::/64`  
Next Hop : `fe80::aab8:e0ff:fe04:ec45`

> `PROBLEME AVEC NEXTHOP !!!`
{: .prompt-danger }

**Paramétrage IPV6 machine cwwk**  
Ajouter ce qui suit au fichier `/etc/network/interfaces`

```
iface enp3s0 inet6 static
    address 2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45
    netmask 64
    gateway fe80::8e97:eaff:fe39:66d6
```

Redémarrer la machine (`sudo systemctl reboot`) et vérifier (`ip a`) 

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether a8:b8:e0:04:ec:45 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.205/24 brd 192.168.0.255 scope global enp3s0
       valid_lft forever preferred_lft forever
    inet6 2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45/64 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::aab8:e0ff:fe04:ec45/64 scope link 
       valid_lft forever preferred_lft forever
3: enp4s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether a8:b8:e0:04:ec:46 brd ff:ff:ff:ff:ff:ff
```

### Connexion SSH 192.168.0.205

Se connecter sur la nouvelle adresse ip

    ssh yick@192.168.0.205

Modifier les mots de passe "root" et utilisateur "yick" 

### Hostname + Date et heure + Groupe

**Hostname**  Commande : `hostnamectl`

```
 Static hostname: cwwk
       Icon name: computer-desktop
         Chassis: desktop 🖥️
      Machine ID: 2ecf8ec9715a47ec90b45f1e796a7cf2
         Boot ID: 7ec81314da2c43e4a4992d69af26b588
Operating System: Debian GNU/Linux 12 (bookworm)  
          Kernel: Linux 6.1.0-28-amd64
    Architecture: x86-64
 Hardware Vendor: Default string
  Hardware Model: Default string
Firmware Version: 5.27
```

**Date et heure** Commande `timedatectl`

```
               Local time: jeu. 2024-12-12 20:15:50 CET
           Universal time: jeu. 2024-12-12 19:15:50 UTC
                 RTC time: jeu. 2024-12-12 19:15:50
                Time zone: Europe/Paris (CET, +0100)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

**Groupe "users"**  
Vérifier si utilisateur appartient au groupe "users" : `id yick`  

```
uid=1000(yick) gid=1000(yick) groupes=1000(yick),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev)
```

Si non appartenance, exécuter : `sudo usermod -a -G users $USER`

Pour visualiser tous les messages de journal, ajouter l'utilisateur au groupe existant **adm** 

    sudo usermod -a -G adm $USER

### OpenSSH, clé et script

![OpenSSH](ssh_logo1.png){:width="70" .left}  
_OpenSSH_

<u>Générer une paire de clé sur l'ordinateur de bureau PC1</u>  
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec le serveur.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/yick-ed25519
    chmod 600 /home/yann/.ssh/yick-ed25519

Envoyer les clés publiques sur le serveur yick  

    ssh-copy-id -i ~/.ssh/yick-ed25519.pub yick@192.168.0.205

<u>On se connecte sur le serveur debian 12</u>  

    ssh yick@192.168.0.205

Modifier la configuration serveur SSH  

    sudo nano /etc/ssh/sshd_config

Modifier

```conf
Port = 55205
PasswordAuthentication no
```

Relancer le serveur

    sudo systemctl restart sshd

Test connexion

    ssh -p 55205 -i ~/.ssh/yick-ed25519 yick@192.168.0.205

### Utilitaires + Historique ligne de commande + Motd

Installer utilitaires  

    sudo apt install rsync curl tmux jq figlet git tree borgbackup nmap ldap-utils

Ajoutez la recherche d’historique de la ligne de commande au terminal  
Se connecter en utilisateur debian  
Tapez un début de commande précédent, puis utilisez shift + up (flèche haut) pour rechercher l’historique filtré avec le début de la commande.

```bash
# Global, tout utilisateur
echo '"\e[1;2A": history-search-backward' | sudo tee -a /etc/inputrc
echo '"\e[1;2B": history-search-forward' | sudo tee -a /etc/inputrc
```

**Motd**  
Effacer et créer motd

    sudo rm /etc/motd && sudo nano /etc/motd

```
                      _    __  __ ___   __       ___  ___ 
  __ __ __ ____ __ __| |__ \ \/ /( _ ) / /  ___ | _ \| __|
 / _|\ V  V /\ V  V /| / /  >  < / _ \/ _ \|___||  _/|__ \
 \__| \_/\_/  \_/\_/ |_\_\ /_/\_\\___/\___/     |_|  |___/
  ___        _     _               _  ___                 
 |   \  ___ | |__ (_) __ _  _ _   / ||_  )                
 | |) |/ -_)| '_ \| |/ _` || ' \  | | / /                 
 |___/ \___||_.__/|_|\__,_||_||_| |_|/___|                
  _  ___  ___     _   __  ___     __     ___  __   ___    
 / |/ _ \|_  )   / | / / ( _ )   /  \   |_  )/  \ | __|   
 | |\_, / / /  _ | |/ _ \/ _ \ _| () |_  / /| () ||__ \   
 |_| /_/ /___|(_)|_|\___/\___/(_)\__/(_)/___|\__/ |___/   
```

### LVM - SSD NVMe 2To

*Ajout d'un SSD NVMe 2To sur la machine cwwk*

les commandes gdisk et sgdisk sont installées via le paquet gdisk

    sudo apt install gdisk

#### Partitionner SSD NVMe 2To
  
Identifier le périphérique: `lsblk`  
![](cwwk02.png)

Périphérique `/dev/nvme1n1`

On efface tout

    sudo sgdisk --zap-all /dev/nvme1n1

*GPT data structures destroyed! You may now partition the disk using fdisk or
other utilities.*

Création partition

    sudo sgdisk --clear --new=1:0:0 --typecode=1:8e00 /dev/nvme1n1

*Creating new GPT entries in memory.  
The operation has completed successfully.*


#### PV - Volume physique

*Un volume physique ou « PV » pour « physical volume » est tout simplement un disque ou une partition.*

```bash
sudo pvcreate /dev/nvme1n1p1  # Physical volume "/dev/nvme1n1p1" successfully created.
```

#### VG - Groupe de volumes

*Un groupe de volumes ou « VG » pour « volume group » est un ensemble de volumes physiques. On a donc un ou plusieurs volumes physiques dans un groupe de volumes, et pour utiliser LVM, il faut obligatoirement au moins un groupe de volumes.*

```bash
# volume sur le SSD NVMe a
sudo vgcreate nvmea-vg /dev/nvme1n1p1  # Volume group "nvmea-vg" successfully created
```

#### LV - Volume logique

*Un volume logique ou « LV » pour « logical volume » est un espace dans un groupe de volume où l’on peut mettre un système de fichiers.*

Création des volumes logiques 

```bash
# ncdata : pour les données de nextcloud
sudo lvcreate -L 150G -n ncdata nvmea-vg
# sharenfs : pour le partage NFS
sudo lvcreate -L 250G -n sharenfs nvmea-vg
# kvm : pour le virtuel 
sudo lvcreate -L 200G -n kvm nvmea-vg
```

Format des partitions

```bash
sudo mkfs.ext4 /dev/mapper/nvmea--vg-ncdata
sudo mkfs.ext4 /dev/mapper/nvmea--vg-sharenfs
sudo mkfs.ext4 /dev/mapper/nvmea--vg-kvm
```

#### Montages fstab

UUID des partitions logiques LVM : 

```bash
sudo blkid |grep "/dev/mapper/nvmea--vg-ncdata"    # UUID="c8cbc005-259d-4642-863c-b753088cefc3"
sudo blkid |grep "/dev/mapper/nvmea--vg-sharenfs"  # UUID="eb2cfabe-8e1d-463d-a62d-9da921a747f3"
sudo blkid |grep "/dev/mapper/nvmea--vg-kvm"       # UUID="e2c983fd-588b-423c-b687-6346f1f81b0f"
```

Création des points de montage et modifier les propriétés pur le dossier partagé NFS

```bash
sudo mkdir -p /srv/{nextcloud-data,kvm}
sudo mkdir -p /sharenfs
# # Modifications possibles par les utilisateurs avec ID=1000
sudo chown -R $USER:$USER /sharenfs       
sudo chown -R $USER:$USER /srv/kvm       
```

Ajout au fichier `/etc/fstab`

```
# /dev/mapper/nvmea--vg-ncdata
UUID=c8cbc005-259d-4642-863c-b753088cefc3 /srv/nextcloud-data   ext4    defaults        0       2
# /dev/mapper/nvmea--vg-kvm 
UUID=e2c983fd-588b-423c-b687-6346f1f81b0f /srv/kvm 	ext4	defaults	0	2
#  /dev/mapper/nvmea--vg-sharenfs
UUID=eb2cfabe-8e1d-463d-a62d-9da921a747f3 /mnt/sharenfs ext4    defaults     0	2
```

Modifier le fichier /etc/fstab pour les UUID root,home et swap de l'installation debian

```
/dev/mapper/cwwk--vg-swap_1: UUID="9f71df16-af9d-49af-a204-0eb8fdcd0371" TYPE="swap"
/dev/mapper/cwwk--vg-home: UUID="996ee697-528b-4af6-9c4e-6d2a6c759303" BLOCK_SIZE="4096" TYPE="ext4"
/dev/mapper/cwwk--vg-root: UUID="3e842a93-ffd5-4257-b928-e9b6288064a7" BLOCK_SIZE="4096" TYPE="ext4"
```

### Créer volume logique "var"

*Le dossier /var contient , à terme, tous les chemins "root" du web. Il peut dépasser les 5Go*  

Le dossier /var est actuellement dans le volume logique root `/dev//mapper/cwwk--vg-root`

Création volume logique "var" de 10Go

```bash
sudo lvcreate -L 10G -n var cwwk-vg
```

Format partition 

```bash
sudo mkfs.ext4 /dev/mapper/cwwk--vg-var
```

Relever son UUID

```bash
sudo blkid |grep "/dev/mapper/cwwk--vg-var"    # UUID="9d942292-a4d4-4fb3-b8e9-b945b8233d72"
```

Ajout au fichier `/etc/fstab`

```
#/dev/mapper/cwwk--vg-var
UUID=9d942292-a4d4-4fb3-b8e9-b945b8233d72 /var           ext4    defaults        0       2
```

Recharger et monter

    sudo systemctl daemon-reload
    sudo mount -a

Vérification `lsblk`

```
NAME                   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme1n1                259:0    0   1,9T  0 disk 
└─nvme1n1p1            259:1    0   1,9T  0 part 
  ├─nvmea--vg-ncdata   254:0    0   150G  0 lvm  /srv/nextcloud-data
  ├─nvmea--vg-sharenfs 254:1    0   250G  0 lvm  /sharenfs
  └─nvmea--vg-kvm      254:2    0   200G  0 lvm  /srv/kvm
nvme0n1                259:2    0 119,2G  0 disk 
├─nvme0n1p1            259:3    0   512M  0 part /boot/efi
├─nvme0n1p2            259:4    0   488M  0 part /boot
└─nvme0n1p3            259:5    0 118,3G  0 part 
  ├─cwwk--vg-root      254:3    0  27,9G  0 lvm  /
  ├─cwwk--vg-swap_1    254:4    0   976M  0 lvm  [SWAP]
  ├─cwwk--vg-home      254:5    0  64,2G  0 lvm  /home
  └─cwwk--vg-var       254:6    0    10G  0 lvm  /var
```

### Script ssh_rc_bash

>ATTENTION!!! Les scripts sur connexion peuvent poser des problèmes pour des appels externes autres que ssh

```bash
wget https://static.rnmkcy.eu/files/ssh_rc_bash
chmod +x ssh_rc_bash # rendre le bash exécutable
./ssh_rc_bash        # exécution
```

![](cwwk01.png)

### Journaux

Users in groups 'adm', 'systemd-journal' can see all messages.
Pour visualiser tous les messages de journal, ajouter l’utilisateur au groupe existant adm

    sudo usermod -a -G adm $USER
    sudo usermod -a -G systemd-journal $USER

Pour voir les messages sans coupures

    SYSTEMD_LESS=FRXMK journalctl -u caddy

Ajouter au .bashrc

    echo "export SYSTEMD_LESS=FRXMK"


### Parefeu UFW

![ufw](ufw-logo-a.png){:width="50" .left} 

*UFW, ou pare - feu simple , est une interface pour gérer les règles de pare-feu dans Arch Linux, Debian ou Ubuntu. UFW est utilisé via la ligne de commande (bien qu'il dispose d'interfaces graphiques disponibles), et vise à rendre la configuration du pare-feu facile.*

Installation **Debian / Ubuntu**

    sudo apt install ufw

*Par défaut, les jeux de règles d'UFW sont vides, de sorte qu'il n'applique aucune règle de pare-feu, même lorsque le démon est en cours d'exécution.*   

Les règles 

```bash
sudo ufw allow 55205/tcp  # port SSH
sudo ufw allow https      # port 443
```

Activer le parefeu

    sudo ufw enable

```
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

Status

     sudo ufw status 

```
Status: active

To                         Action      From
--                         ------      ----
55205/tcp                  ALLOW       Anywhere                  
443                        ALLOW       Anywhere                  
55205/tcp (v6)             ALLOW       Anywhere (v6)             
443 (v6)                   ALLOW       Anywhere (v6)             
```

désactiver la journalisation

    sudo ufw logging off

## Configuration du second port Ethernet 2.5Gb enp4s0

*Serveur Debian CWWK x86-P5 configuration du second port Ethernet 2.5 Gigabit enp4s0*

Très largement inspiré de l'article [Configuration du second réseau](https://blog.stephane-robert.info/docs/homelab/machine-rebond-2-interfaces/)

* Machine CWWK x86-P5 Debian 12 equioée de 2 ports ethernet 
* Accès internet
* Switch pour le second réseau

### Configuration Debian des cartes réseau

La machine CWWK Debian sert de passerelle entre lz réseau principal (192.168.0.0/24) et le réseau secondaire (192.168.20.0/24).  
Cette configuration assure que le serveur CWWK peut communiquer à la fois avec les dispositifs internes sur les deux sous-réseaux et avec l’Internet externe.

**Les interfaces réseau:** `ip link` 

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether a8:b8:e0:04:ec:45 brd ff:ff:ff:ff:ff:ff
3: enp4s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether a8:b8:e0:04:ec:46 brd ff:ff:ff:ff:ff:ff
```

> `Interfaces réseau enp3s0 et enp4s0`
{: .prompt-tip }

Configuration interfaces réseau avant modification

    /etc/network/interfaces

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
# allow-hotplug enp3s0
allow-hotplug enp3s0
# iface enp3s0 inet dhcp
iface enp3s0 inet static
    address 192.168.0.205
    netmask 255.255.255.0
    gateway 192.168.0.254
#iface enp3s0 inet6 dhcp
iface enp3s0 inet6 static
    address 2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45
    netmask 64
    gateway fe80::8e97:eaff:fe39:66d6
```

#### A - Interface bridge 192.168.10.0/24

Modification pour le second réseau , Interface enp4s0 192.168.10.0/24  

![](CWWK_X86-P5_02A.drawio.png)

Il faut relier le routeur Tenda avec CWWK enp4s0 par un câble ethernet

```
# Interface pour le réseau secondaire
# Bridge setup
auto br0
iface br0 inet static
    bridge_ports enp4s0
    address 192.168.10.100
    netmask 255.255.255.0
    bridge_stp off
    bridge_fd 0
    bridge_maxwait 0
    # Pas de gateway spécifié pour cette interface, car elle ne se connecte pas directement à Internet
```

* enp3s0 est configurée avec une adresse IP statique qui lui permet de communiquer avec le réseau principal.
* La passerelle par défaut est également définie pour diriger le trafic Internet
* Pour enp4s0, une adresse IP statique est configurée pour le réseau secondaire, mais sans passerelle spécifiée, car le trafic Internet passera par enp3s0

Sur le routeur tenda, ajouter une réservation IP statique sur br0 (link/ether 96:9c:89:a6:a0:8f)  
![](cwwk-ip-statique.png)

Après redémarrage de la machine  

```
4: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 96:9c:89:a6:a0:8f brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.100/24 brd 192.168.10.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 2001:3::949c:89ff:fea6:a08f/64 scope global dynamic mngtmpaddr 
       valid_lft 115sec preferred_lft 115sec
    inet6 fe80::949c:89ff:fea6:a08f/64 scope link 
       valid_lft forever preferred_lft forever
```

#### B - Interface enp4s0 192.168.70.0/24

![](CWWK_X86-P5_02.png)    

Modification pour le second réseau , Interface enp4s0 192.168.70.0/24

```
# Interface pour le réseau secondaire
auto enp4s0
iface enp4s0 inet static
    address 192.168.70.1
    netmask 255.255.255.0
    # Pas de gateway spécifié pour cette interface, car elle ne se connecte pas directement à Internet
```

* enp3s0 est configurée avec une adresse IP statique qui lui permet de communiquer avec le réseau principal.
* La passerelle par défaut est également définie pour diriger le trafic Internet
* Pour enp4s0, une adresse IP statique est configurée pour le réseau secondaire, mais sans passerelle spécifiée, car le trafic Internet passera par enp3s0

### Activation du routage IP

Pour permettre à notre serveur de transmettre le trafic entre les deux réseaux et vers Internet, on active le routage IP en modifiant le fichier `/etc/sysctl.conf` 

```
net.ipv4.ip_forward=1
```

Après avoir modifié ce fichier, appliquer la modification avec la commande `sudo sysctl -p` 

### Configuration pare-feu UFW

Configurer ufw pour permettre et contrôler le trafic entre les réseaux.

Il faut autoriser le forwarding dans la configuration d’ufw.

    nano /etc/default/ufw 


Recherchez la ligne commençant par `DEFAULT_FORWARD_POLICY`.  
Remplacez la valeur **DROP** par **ACCEPT**. Enregistrez le fichier et quittez l’éditeur.

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

Avant de définir des règles spécifiques pour notre cas de routage, configurer les politiques par défaut de ufw pour refuser tout le trafic entrant et autoriser tout le trafic sortant

```
sudo ufw --force disable
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Autoriser le flux ssh et dns 

```
sudo ufw allow ssh
sudo ufw allow dns
sudo ufw allow out to any port 53
```

Ajouter les règles de forwarding entre l’interface enp4s0 et enp3s0 pour permettre la mise à jour des packages sur les machines du second réseau.

    sudo nano /etc/ufw/before.rules

et ajouter ce bloc juste avant *filter :

```
*nat
:POSTROUTING ACCEPT [0:0]

# Forward traffic through enp3s0 - Change to match you out-interface
-A POSTROUTING -s 192.168.70.0/24 -o enp3s0 -j MASQUERADE

# don't delete the 'COMMIT' line or these nat table rules won't
# be processed
COMMIT
```

Activer le service ufw 

    sudo ufw enable

Vérifions son status 

    sudo ufw status

```
Status: active

To                         Action      From
--                         ------      ----
55205/tcp                  ALLOW       Anywhere                  
443                        ALLOW       Anywhere                  
2049                       ALLOW       192.168.0.0/24            
DNS                        ALLOW       Anywhere                  
55205/tcp (v6)             ALLOW       Anywhere (v6)             
443 (v6)                   ALLOW       Anywhere (v6)             
DNS (v6)                   ALLOW       Anywhere (v6)             

53                         ALLOW OUT   Anywhere                  
53 (v6)                    ALLOW OUT   Anywhere (v6)             
```

55025/tcp : Port de communication SSH  
2049 : Port NFS  

## Serveur DNS Unbound

![DNS Unbound](unbound-250.png){:width="150" .left}  
*Les serveurs DNS sont des machines discutant entre elles afin de se communiquer les correspondances entre nom de domaine et adresses IP.*

[Résolveur DNS Unbound](/posts/unbound-resolveur-DNS/)

## Pont Réseau (bridge network)

*Le réseau par défaut offre un moyen très simple d’obtenir une connectivité lors de la création de machines virtuelles : tout est « prêt » et fonctionne dès la sortie de la boîte. Parfois, cependant, nous souhaitons obtenir une connexion  Bridge où les VM sont connectés au réseau local de l’hôte sans utiliser NAT.[(Installing and Using KVM for Virtualization on Debian 12)](https://reintech.io/blog/installing-using-kvm-virtualization-debian-12)*

**en mode su**

Pour configurer le mode Bridge, vous devez créer un pont virtuel et partager l’une des interfaces Ethernet physiques de l’hôte.

Le programme dont vous aurez besoin s'appelle brctl et est inclus dans bridge-utils.

    apt install bridge-utils

### Configuration passerelle br0 

Pour rendre votre configuration permanente, vous devez éditer le fichier `/etc/network/interfaces` 

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
# allow-hotplug enp3s0
allow-hotplug enp3s0
# iface enp3s0 inet dhcp
iface enp3s0 inet static
    address 192.168.0.205
    netmask 255.255.255.0
    gateway 192.168.0.254
#iface enp3s0 inet6 dhcp
iface enp3s0 inet6 static
    address 2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45
    netmask 64
    gateway fe80::8e97:eaff:fe39:66d6

# Interface pour le réseau secondaire
#auto enp4s0
#iface enp4s0 inet static
#    address 192.168.70.1
#    netmask 255.255.255.0
    # Pas de gateway spécifié pour cette interface, car elle ne se connecte pas directement à Internet

# Bridge setup
auto br0
iface br0 inet static
    bridge_ports enp4s0
    address 192.168.10.1
    netmask 255.255.255.0
    bridge_ports eth0
    bridge_stp off
    bridge_fd 0
    bridge_maxwait 0
```

## Serveur DHCP

*Après avoir configuré ufw pour permettre le routage sécurisé entre les réseaux, l’étape suivante consiste à mettre en place isc-dhcp-server sur notre serveur Debian. Ce serveur DHCP distribuera automatiquement des adresses IP, des masques de sous-réseau, des gateways et des informations de serveurs DNS aux dispositifs connectés au réseau secondaire.* 

### Installer isc-dhcp-server

installer le paquet isc-dhcp-server :

```
sudo apt update
sudo apt install isc-dhcp-server
```

### Configurer interface réseau secondaire enp4s0

Configurer **isc-dhcp-server** pour spécifier sur quelle interface écouter les requêtes DHCP.  
Dans notre cas, il s’agit de **enp4s0**, l’interface connectée au réseau secondaire.  

**Modifier serveur DHCP**  
`INTERFACESv4="br0"` pour le dhcp `/etc/default/isc-dhcp-server`

```
# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#	Separate multiple interfaces with spaces, e.g. "eth0 eth1".
#INTERFACESv4="enp4s0"
INTERFACESv4="br0"
INTERFACESv6=""
```

### Configuration serveur DHCP

* <https://wiki.debian.org/resolv.conf>
* [Comment installer et configurer un serveur DHCP sur Debian ?](https://www.it-connect.fr/serveur-dhcp-sous-linux/)  
* [Comment configurer un serveur DHCP sous Linux](https://www.malekal.com/configurer-serveur-dhcp-linux/)

La partie la plus importante de la configuration de **isc-dhcp-server** consiste à définir la plage d’adresses IP à distribuer, ainsi que d’autres options pour le réseau.  
Modifier le fichier `/etc/dhcp/dhcpd.conf` pour ajouter la configuration spécifique au réseau secondaire

```
# dhcpd.conf
#
subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.10 192.168.10.100;
  option routers 192.168.10.1;
  option subnet-mask 255.255.255.0;
  option domain-name-servers 192.168.10.1;
  # option domain-name "yann.priv";
  default-lease-time 600;
  max-lease-time 7200;
}
```

Dans cette configuration

* **range** : plage d’adresses IP disponibles pour la distribution aux clients DHCP.
* **option routers** : passerelle par défaut, qui est l’adresse IP de notre serveur sur **enp4s0**. 
* **option domain-name-servers** : serveur DNS que les clients utiliseront, que nous avons également configuré pour pointer vers notre serveur Debian, agissant comme serveur DNS avec bind9. 
* Les options **default-lease-time** et **max-lease-time** définissent la durée de bail par défaut et maximale en secondes.

### Démarrage et vérification du service

Après avoir configuré le serveur DHCP, démarrer le service et vérifier

```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

Le status

```
● isc-dhcp-server.service - LSB: DHCP server
     Loaded: loaded (/etc/init.d/isc-dhcp-server; generated)
     Active: active (running) since Tue 2025-01-14 18:22:12 CET; 12s ago
       Docs: man:systemd-sysv-generator(8)
    Process: 2331 ExecStart=/etc/init.d/isc-dhcp-server start (code=exited, status=0/SUCCESS)
      Tasks: 1 (limit: 38143)
     Memory: 7.1M
        CPU: 30ms
     CGroup: /system.slice/isc-dhcp-server.service
             └─2343 /usr/sbin/dhcpd -4 -q -cf /etc/dhcp/dhcpd.conf enp4s0

janv. 14 18:22:10 cwwk systemd[1]: Starting isc-dhcp-server.service - LSB: DHCP server...
janv. 14 18:22:10 cwwk isc-dhcp-server[2331]: Launching IPv4 server only.
janv. 14 18:22:10 cwwk dhcpd[2343]: Wrote 0 leases to leases file.
janv. 14 18:22:10 cwwk dhcpd[2343]: Server starting service.
janv. 14 18:22:12 cwwk isc-dhcp-server[2331]: Starting ISC DHCPv4 server: dhcpd.
janv. 14 18:22:12 cwwk systemd[1]: Started isc-dhcp-server.service - LSB: DHCP server.
```

### Test de la configuration DHCP

Pour tester que le serveur DHCP fonctionne correctement, connecter un appareil au réseau secondaire et vérifie qu’il reçoit une adresse IP dans la plage définie, ainsi que les informations correctes de passerelle et de serveur DNS.  

## Partages

### NFS

#### NFS serveur

1. [Installation serveur NFSv4](/posts/NFSv4/#installation-du-serveur-nfs)
2. [Parefeu UFW](/posts/NFSv4/#configuration-du-pare-feu-ufw)
3. [Partage Sans pseudo système de fichiers](/posts/NFS/#sans-pseudo-système-de-fichiers-défaut)  
`sudo mkdir -p /sharenfs`  
`sudo chown $USER:$USER -R /sharenfs`  

Ouvrez le fichier `/etc/exports` à l’aide de la commande d’éditeur nano suivante.

    sudo nano /etc/exports

Ajoutez la ligne suivante au fichier.

```bash
/sharenfs    192.168.0.0/255.255.255.0(rw,no_root_squash,no_subtree_check)
```

Enregistrez et fermez le fichier lorsque vous avez terminé.

Vous trouverez ci-dessous quelques options NFS utilisées pour ce cas :

*    **rw** : autorise l'accès en lecture et en écriture pour le serveur et le client NFS au volume/répertoire.
*    **no_subtree_check** : désactive la vérification des sous-arbres, ce qui a de légères implications en matière de sécurité, mais peut améliorer la fiabilité dans certaines circonstances.
*    **no_root_sqash** : désactive le squashing de la racine. Cette option est principalement utile pour les clients sans disque.
*    **fsid=0** : pour NFSv4, si utilisé, ce paramètre est utilisé pour informer le serveur NFS que cette exportation est la racine de tous les systèmes de fichiers exportés.

Ensuite, exécutez la commande suivante pour redémarrer le service nfs-server et appliquer les modifications. Puis, vérifiez le serveur nfs pour vous assurer que le service est en cours d'exécution.

```bash
sudo systemctl restart nfs-server 
sudo systemctl status nfs-server
```

Enfin, exécutez la commande suivante pour vérifier le répertoire et le système de fichiers exportés sur le serveur NFS à l’aide de la commande suivante.

```bash
sudo showmount -e 192.168.0.205 
#sudo showmount -e
```

Si tout se passe bien, vous devriez voir que le répertoire `//sharenfs` est exporté sur le serveur NFS.  
`/sharenfs 192.168.0.0/255.255.255.0`

Créer des liens

```bash
ln -s /sharenfs $HOME/sharenfs
ln -s /sharenfs/scripts $HOME/scripts
sudo ln -s /sharenfs /mnt/sharenfs
```

#### NFS client

Installez le paquetage du client NFS

```bash
sudo apt install nfs-common # debian
sudo pacman -S nfs-utils    # archlinux
```

Créer dossier cible

    sudo mkdir -p /mnt/cwwk

Montage manuel

    sudo mount.nfs4 192.168.0.205:/sharenfs /mnt/cwwk

Démontage

    sudo umount /mnt/cwwk

Montage `/etc/fstab`

```
# Serveur debian 12 cwwk 192.168.0.205
192.168.0.205:/sharenfs	/mnt/cwwk nfs4 nofail,x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=10s,rsize=8192,wsize=8192 0 0
```

Rechargement et montage

```bash
sudo systemctl daemon-reload
sudo mount -a
```

### Samba FreeUSB2To (Freebox)

*Partage disque USB 2To monté sur FreeboX*

Accés partage samba depuis linux

Installation paquet cifs-utils

```bash
sudo apt install cifs-utils  # Debian
sudo pacman -S cifs-utils    # Archlinux 
```

Création répertoire de montage et lien

```bash
sudo mkdir /mnt/FreeUSB2To   
sudo ln -s /mnt/FreeUSB2To $HOME/FreeUSB2To
```

Création fichier caché pour login et mot de passe

```bash
sudo nano /root/.smbcredentials
```

Dans le fichier, ajoutez les lignes suivantes (remplacez les XXXX par votre login et mot de passe)

```bash
username=XXXXXX
password=XXXXXX
```

Donnez les droits adéquats au fichier

```bash
sudo chown -R root:root /root/.smbcredentials
sudo chmod -R 600 /root/.smbcredentials
```

[Montage linux du disque USB Freebox](/posts/Partage_disque_externe_USB_sur_Freebox/#montage-linux-du-disque-usb-freebox)

**Résumé**  
Partage : //192.168.0.254/FreeUSB2To  
Point de montage local : `sudo mkdir -p /mnt/FreeUSB2To`  
Outil cifs : `sudo apt install cifs-utils`  
Utlisateur mot de passe:  `/root/.smbcredentials`  
Lien:  `ln -s /mnt/FreeUSB2To/ $HOME/FreeUSB2To`  

Trois options de montage, via  systemd.mount, systemd.automount ou fstab

**A-Montage avec systemd mount

Fichier `/etc/systemd/system/mnt-FreeUSB2To.mount`**

```
[Unit]
  Description=cifs mount script
  Requires=network-online.target
  After=network-online.service

[Mount]
  What=//192.168.0.254/FreeUSB2To
  Where=/mnt/FreeUSB2To
  Options=credentials=/root/.smbcredentials,rw,uid=1000,gid=1000,vers=3.0
  Type=cifs

[Install]
  WantedBy=multi-user.target
```

Lancement et activation

    sudo systemctl enable mnt-FreeUSB2To.mount --now

Vérifier : `ls /mnt/FreeUSB2To/`

**B-Montage avec systemd automount**

Fichier `/etc/systemd/system/mnt-FreeUSB2To.mount`

```
[Unit]
  Description=cifs mount script
  Requires=network-online.target
  After=network-online.service

[Mount]
  What=//192.168.0.254/FreeUSB2To
  Where=/mnt/FreeUSB2To
  Options=credentials=/root/.smbcredentials,rw,uid=1000,gid=1000,vers=3.0
  Type=cifs

[Install]
  WantedBy=multi-user.target
```

Fichier `/etc/systemd/system/mnt-FreeUSB2To.automount`

```
[Unit]
  Description=cifs mount script
  Requires=network-online.target
  After=network-online.service

[Automount]
  Where=/mnt/FreeUSB2To
  TimeoutIdleSec=0

[Install]
  WantedBy=multi-user.target
```

Lancement et activation

    sudo systemctl enable mnt-FreeUSB2To.automount --now

Vérifier : `ls /mnt/FreeUSB2To/`

**C-Montage via fstab**

Modèle pour fstab

```
//[Serveur]/[Partage]  [PointDeMontage]  cifs  noauto,x-systemd.automount,_netdev,file_mode=0750,dir_mode=0750,uid=[Mon_UID],gid=[Mon_GID],credentials=~/.smbcredentials  0  0
```

Explication des options:

*    `//[Serveur]/[Partage]`: Adresse du serveur et nom du partage CIFS. Exemple: //192.168.1.10/Documents.
*    `[PointDeMontage]`: Chemin du dossier local où le partage sera monté. Exemple: /mnt/partage_documents.
*    `cifs`: Indique que le système de fichiers à monter est de type CIFS.
*    `noauto`: Empêche le montage automatique du partage au démarrage du système.
*    `x-systemd.automount`: Active le montage automatique à la demande via Systemd.
*    `_netdev`: Indique que le partage se trouve sur un périphérique réseau.
*    `file_mode=0750`: Définit les permissions des fichiers sur le partage monté (lecture, écriture et exécution pour le propriétaire et le groupe, aucune permission pour les autres).
*    `dir_mode=0750`: Définit les permissions des dossiers sur le partage monté.
*    `uid=[Mon_UID]`: ID utilisateur qui sera propriétaire des fichiers et dossiers sur le partage monté.
*    `gid=[Mon_GID]`: ID du groupe qui sera propriétaire des fichiers et dossiers sur le partage monté.
*    `credentials=~/.smbcredentials`: Chemin vers le fichier contenant les informations d’identification.

Ajout de la ligne suivante au fichier /etc/fstab

    //192.168.0.254/FreeUSB2To /mnt/FreeUSB2To cifs _netdev,noauto,x-systemd.automount,uid=1000,gid=1000,credentials=/root/.smbcredentials 0 0

*Le partage CIFS sera automatiquement monté lorsque vous accéderez au dossier de montage `/mnt/FreeUSB2To`. Il sera démonté automatiquement après une période d’inactivité.*

Recharger

    sudo systemctl daemon-reload
    sudo mount -a

### Forcer le démontage FreeUSB2To avant arrêt ou redémarrage machine

PROBLEME DE DEMONTAGE PARTITION CIFS

Comment exécuter un script avec systemd juste avant l'arrêt sous Linux  
[How to run script with systemd right before shutdown in Linux](https://www.golinuxcloud.com/run-script-with-systemd-before-shutdown-linux/)

Le script va forcer le démontage de /mnt/FreeUSB2To

    nano /home/yick/scripts/umount_force.sh

```bash
#!/bin/bash
# Pour éviter que le montage ne se bloque et se débarrasser du montage lui-même
# nous pouvons forcer (-f) un démontage paresseux (-l) :
echo "Forcer démontage FreeUSB2To" | systemd-cat -t cwwk -p info
umount -f -l /mnt/FreeUSB2To
```

Rendre exécutable

    chmod u+x 

Créer un service run-before-shutdown.service

    sudo nano /etc/systemd/system/run-before-shutdown.service

```
[Unit]
Description=Force umount
DefaultDependencies=no
Before=shutdown.target

[Service]
Type=oneshot
ExecStart=/home/yick/scripts/umount_force.sh
TimeoutStartSec=0

[Install]
WantedBy=shutdown.target
```

Rafraîchir les fichiers de configuration de systemd

    sudo systemctl daemon-reload

Activer le script pour qu'il démarre automatiquement au prochain démarrage

    sudo systemctl enable run-before-shutdown.service

### Alias (.bash_aliases)

Le fichier des alias `~/.bash_aliases`

```
alias borglist='$HOME/scripts/borglist.sh'
alias etat='$HOME/scripts/etat_des_lieux.sh'
alias sshm='$HOME/scripts/ssh-manager.sh'
```

## Virtualisation KVM

![](kvm-logo-jaune.png){: .left}  
*KVM, acronyme de Kernel-based Virtual Machine, est un outil de virtualisation complet et opensource pour les systèmes Linux avec des extensions de virtualisation matérielle basées sur les processeurs Intel-VT ou AMD-V.*

Vérifiez la prise en charge de la virtualisation matérielle sur votre système avec la commande `grep --color -iE 'vmx|svm' /proc/cpuinfo` qui recherche les drapeaux du processeur : Intel <mark>vmx</mark> (Virtual Machine Extension) ou AMD <mark>svm</mark> (Secure virtual Machine).

### Installer KVM sur un serveur

Sur un serveur headless sans interface graphique, il suffit d'exécuter la commande ci-dessous  

    sudo apt install qemu-system libvirt-daemon-system virtinst qemu-utils

Une fois l'installation terminée, vérifiez si les modules KVM sont chargés

    lsmod | grep -i kvm

```
kvm_intel             380928  0
kvm                  1146880  1 kvm_intel
irqbypass              16384  1 kvm
```

### Démarrer le démon de gestion libvirt (libvirtd)

*le programme __libvirtd__ est le composant démon côté serveur du système de gestion de la virtualisation libvirt. Ce démon s'exécute sur les serveurs hôtes et exécute les tâches de gestion requises pour les invités virtualisés. Cela inclut des activités telles que le démarrage, l'arrêt et la migration des invités entre les serveurs hôtes, la configuration et la manipulation du réseau, et la gestion du stockage pour l'utilisation par les invités. Les bibliothèques et utilitaires clients libvirt se connectent à ce démon pour exécuter des tâches et collecter des informations sur la configuration et les ressources du système hôte et des invités.*

Il convient donc de lancer et d'activer ce service au démarrage du système

    sudo systemctl enable --now libvirtd

Vérifier l'état

    systemctl  status libvirtd

![](cwwk03.png)  
> `le service libvirtd est activé et démarré !`
{: .prompt-tip }

Par défaut, le démon libvirtd écoute les requêtes sur un socket local de domaine Unix. En utilisant l'option de ligne de commande `-l | --listen`, il est possible de demander au démon libvirtd d'écouter également sur une socket TCP/IP. Le socket TCP/IP à utiliser est défini dans le fichier de configuration de libvirtd.

Ajouter en fin de fichier /etc/libvirt/qemu.conf 

```
user = "yick"
group = "yick"
```

Ajoutez maintenant votre utilisateur actuel aux groupes **libvirt-qemu** et **libvirt** afin qu’il puisse créer et gérer des VM à l’aide de virsh utilitaire de ligne de commande ou Virt-manager interface graphique.

```bash
sudo usermod -aG libvirt-qemu $USER
sudo usermod -aG libvirt $USER
```

Recharger l'appartenance à un groupe Linux à l'aide de la commande newgrp 

```bash
newgrp libvirt
newgrp libvirt-qemu
```

La connexion "virsh" par défaut dans la configuration globale: `uri_default = "qemu:///system"`  
Pour tester, se connecter à l'hyperviseur en exécutant 

    virsh list

Le résultat est une liste vide de machines virtuelles.

```
 ID   Nom   État
------------------

```

### PC1 Virt Manager

La boîte de dialogue GUI de virt-manager ne permet pas de spécifier un port ssh autre que celui par défaut ou la clé privée à utiliser lors de la connexion au serveur distant, mais cela est facilement réalisable en démarrant virt-manager avec le paramètre '-c'

```bash
virt-manager -c 'qemu+ssh://yick@192.168.0.205:55205/system?keyfile=/home/yann/.ssh/yick-ed25519'
```

**Sur PC1**  
![](cwwk04.png)  

Créer un stockage nommé **machines**   
![](cwwk05.png)  

Modifier le réseau par défaut  
![](cwwk07.png)  
![](cwwk07a.png){: .left} pour démarrer le réseau  

Modifier la plage DHCP `<range start="192.168.110.128" end="192.168.110.254"/>`

### KVM - Pont réseau vituel

Créez un fichier XML qui définit le pont virtuel : `bridged-network.xml`

```xml
<network>
    <name>bridged-network</name>
    <forward mode="bridge" />
    <bridge name="br0" />
</network>
```

Pour définir un réseau à partir d'un fichier XML sans le démarrer 

```bash
sudo virsh net-define bridged-network.xml
```

*Réseau bridged-network défini depuis bridged-network.xml*

Pour démarrer un réseau inactif (précédemment défini) 

```bash
sudo virsh net-start bridged-network
```

*Réseau bridged-network démarré*

Pour configurer le réseau de manière à ce qu'il démarre automatiquement au démarrage du service 

```bash
sudo virsh net-autostart bridged-network
```

*Réseau bridged-network marqué en démarrage automatique*

Vérifier pour confirmer que le drapeau de démarrage automatique est à oui - Persistent devrait aussi lire oui.

```bash
sudo virsh net-list --all
```

Résultat

```
 Nom               État    Démarrage automatique   Persistent
---------------------------------------------------------------
 bridged-network   actif   Oui                     Oui
 network-nat       actif   Oui                     Oui
```


Créer une machine virtuelle alpine linux

```bash
sudo virt-install \
--osinfo alpinelinux3.17 \
--name vm-alpine01 \
--memory 2048 \
--vcpus 1 \
--cpu host \
--hvm \
--disk path=/srv/kvm/libvirt/images/vm-alpine01.qcow2,format=qcow2,size=5 \
--cdrom /home/yick/FreeUSB2To/iso/alpine-standard-3.21.0-x86_64.iso \
--network bridge=br0 \
--graphics vnc  
```

Prendre la main en VNC

    ssh -L 9500:127.0.0.1:5005 yick@192.168.0.205 -p 55205 -i /home/yann/.ssh/yick-ed25519

### Structure des dossiers KVM 

Les dossiers libvirt : `sudo find / -name "libvirt" -type d`

```
/sys/fs/cgroup/machine.slice/machine-qemu\x2d4\x2dalpine\x2dsearx.scope/libvirt
/sys/fs/cgroup/machine.slice/machine-qemu\x2d1\x2dalpine\x2dvm.scope/libvirt
/sys/fs/cgroup/machine.slice/machine-qemu\x2d2\x2dvm\x2dntfy.scope/libvirt
/sys/fs/cgroup/machine.slice/machine-qemu\x2d3\x2dvm\x2ddebian12.scope/libvirt
/etc/apparmor.d/libvirt
/etc/libvirt
/var/log/libvirt
/var/log/swtpm/libvirt
/var/cache/libvirt
/var/lib/libvirt
/dev/hugepages/libvirt
/usr/lib/x86_64-linux-gnu/libvirt
/usr/lib/libvirt
/usr/share/libvirt
/home/yick/.cache/libvirt
/home/yick/.config/libvirt
/srv/kvm/libvirt
/run/libvirt
```

Les configurations **xml** réseau et machines virtuelles : `tree -L 3 /etc/libvirt/qemu`

```
/etc/libvirt/qemu
├── alpine-searx.xml
├── alpine-vm.xml
├── autostart
│   ├── alpine-searx.xml -> /etc/libvirt/qemu/alpine-searx.xml
│   ├── alpine-vm.xml -> /etc/libvirt/qemu/alpine-vm.xml
│   ├── vm-debian12.xml -> /etc/libvirt/qemu/vm-debian12.xml
│   └── vm-ntfy.xml -> /etc/libvirt/qemu/vm-ntfy.xml
├── networks
│   ├── autostart
│   │   ├── bridged-network.xml -> /etc/libvirt/qemu/networks/bridged-network.xml
│   │   └── network-nat.xml -> /etc/libvirt/qemu/networks/network-nat.xml
│   ├── bridged-network.xml
│   └── network-nat.xml
├── vm-alpine01.xml
├── vm-alpine02.xml
├── vm-debian12.xml
├── vm-lldap.xml
└── vm-ntfy.xml
```

## PHP8 + MariaDB

![](php8-logo.png){: .left}  
*Installer la dernière version PHP8 sur debian 12*

### Ajout dépôt sury

Pour installer la version de 8 de php, ajouter le dépôt sury.

```bash
sudo apt install -y lsb-release apt-transport-https ca-certificates wget
sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" |sudo tee /etc/apt/sources.list.d/php.list
```

Mise à jour des dépôts :

    sudo apt update && sudo apt upgrade -y

### Installation PHP8.3 + Modules nécessaires à Nextcloud

Installer php8.3 et modules

```bash
sudo apt -y install \
   php8.3 \
   php8.3-fpm \
   php8.3-sqlite3 \
   php8.3-cli \
   php8.3-gd \
   php8.3-imap \
   php8.3-mysql \
   php8.3-soap \
   php8.3-apcu \
   php8.3-common \
   php8.3-gmp  \
   php8.3-intl \
   php8.3-opcache \
   php8.3-xml \
   php8.3-curl \
   php8.3-igbinary \
   php8.3-readline  \
   php8.3-zip \
   php8.3-bcmath \
   php8.3-imagick \
   php8.3-mbstring \
   php8.3-redis \
   php8.3-bz2 \
   php8.3-smbclient \
   imagemagick libmagickcore-6.q16-6-extra 
```

Vérifiez votre installation PHP en vérifiant sa version : `php --version`

```
PHP 8.3.14 (cli) (built: Nov 25 2024 18:23:27) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.3.14, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.14, Copyright (c), by Zend Technologies
```

Remplacer l’utilisateur et le groupe par défaut par www-data dans le fichier de configuration PHP-FPM.

    sudo nano /etc/php/8.3/fpm/pool.d/www.conf

Dans le fichier, recherchez les lignes ci-dessous et changez leur valeur en www-data :

```
user = www-data
group = www-data
listen.owner = www-data
listen.group = www-data

listen = /run/php/php8.3-fpm.sock
```

Redémarrer PHP-FPM pour appliquer les modifications :

    sudo systemctl restart php8.3-fpm

### MariaDB

![](mariadb-logo-a.png){: .left}

Installer MariaDB :

```bash
sudo apt install mariadb-server -y
sudo mysql_secure_installation # Y à tout et nouveau mot de passe n
```

MariaDB: Modifier ou ajouter un mot de passe root  
Basculer en su (root)

```
# ouvrir mariadb
mariadb -uroot
# sélection base mysql
use mysql;
# définir un nouveau mot de passe root
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('NouveauMotDePasse');
flush privileges;
quit;
```

## Développement

### Go + Node 

![](go-logo.png){: width="100" .normal} ![](nodejs-logo.png){: width="60" .normal}

* [Installer la dernière version de Go](/posts/Debian_installer_Go+Node/#installer-la-dernière-version-de-go)

```bash
cd ~
wget https://go.dev/dl/go1.23.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin"  >> ~/.bashrc
source ~/.bashrc
```

* [Installer Node.js](/posts/Debian_installer_Go+Node/#installer-nodejs-sur-debian-12-11-ou-10-via-nodesource)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
source ~/.bashrc
nvm ls-remote  # affiche les versions
```

Now using node v20.10.0 (npm v10.2.3)  
Creating default alias: default -> 20.10.0 (-> v20.10.0)


## Nginx + Certificats + sites


### nginx 

![](nginx-logo-a.png){: width="100" .left}  
Nginx extras

    sudo apt install nginx-extras

Modifier la configuration `/etc/nginx/nginx.com`

```
variables_hash_max_size 2048;

# ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
ssl_protocols TLSv1.2 TLSv1.3;

        include /etc/nginx/conf.d/*.conf;
        #include /etc/nginx/sites-enabled/*;

```

### Certificats Let’s Encrypt

![](certificat-a.png){: width="100" .left}  
Les dossiers

```bash
sudo mkdir -p /etc/ssl/private/
sudo chown $USER -R /etc/ssl/private/
```

#### acme.sh

Installer acme

```bash
cd ~
sudo apt install socat -y # prérequis
#git clone https://github.com/Neilpang/acme.sh.git
git clone https://github.com/acmesh-official/acme.sh.git
cd acme.sh
./acme.sh --install 
# déconnexion et reconnexion utilisateur
```

La création des certificats pour le domaine rnmkcy.eu Exporter les clés OVH

    acme.sh --dns dns_ovh --server letsencrypt --issue --keylength ec-384 -d 'rnmkcy.eu' -d '*.rnmkcy.eu'

L’installation dans les dossiers locaux

    acme.sh --ecc --install-cert -d rnmkcy.eu --key-file /etc/ssl/private/rnmkcy.eu-key.pem --fullchain-file /etc/ssl/private/rnmkcy.eu-fullchain.pem

Fichier pour ssl `/etc/nginx/conf.d/security.conf.inc`

```
    ssl_certificate /etc/ssl/private/rnmkcy.eu-fullchain.pem;
    ssl_certificate_key /etc/ssl/private/rnmkcy.eu-key.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    # modern configuration
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;

    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    #add_header Strict-Transport-Security "max-age=63072000" always;

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /etc/ssl/private/rnmkcy.eu-fullchain.pem;

    # replace with the IP address of your resolver
    resolver 1.1.1.1 9.9.9.9 valid=300s;
    resolver_timeout 5s;
```

#### Renouvellement des certificats

La tâche crontab qui vérifie si la date de renouvellement est atteinte pour regénérer les certificats

```
6 8 * * * "/home/yick/.acme.sh"/acme.sh --cron --home "$HOME/.acme.sh" > /dev/null
```

Les certificats changent de date à chaque renouvellement  
Le dossier certificat rnmkcy.eu après le renouvellement du 6 février 2025  
![](renew-cert01.png)

> Le but est de copier les certificats nouvellement générés dans le dossier de destination puis de recharger le service nginx.Nous allons surveiller dans le dossier */home/yick/.acme.sh/rnmkcy.eu_ecc/* la modification du fichier **fullchain.cer** qui entrainera l'exécution d'un script. Cette modification de fichier ayant lieu au renouvellement des certificats Let's Encrypt du domaine rnmkcy.eu
{: .prompt-info }

Création dossier systemd utilisateur

    mkdir -p /home/yick/.config/systemd/user/

Dans le répertoire systemd utilisateur nous créons une unité de cheminement **renew-cert-rnmkcy.path**

    nano /home/yick/.config/systemd/user/renew-cert-rnmkcy.path

```ini
[Unit]
Description=Renouvellement certificats Lets Encrypt

[Path]
PathChanged=/home/yick/.acme.sh/rnmkcy.eu_ecc/fullchain.cer
Unit=renew-cert-rnmkcy.service

[Install]
WantedBy=default.target
```

Dans la section `[Path]`, `PathChanged=` indique le chemin absolu du fichier à surveiller, tandis que `Unit=` indique l'unité de service à exécuter si le fichier change. Cette unité (**renew-cert-rnmkcy.path**) doit être lancée lorsque le système est en mode multi-utilisateur.

Ensuite, nous créons l'unité de service correspondante, **renew-cert-rnmkcy.service**, dans le répertoire `~/.config/systemd/user/`    
Si le fichier **sitemap.xml** change (c'est-à-dire qu'il est à la fois écrit et fermé), l'unité de service suivante sera appelée pour exécuter le script spécifié :

    nano /home/yick/.config/systemd/user/renew-cert-rnmkcy.service

```ini
[Unit] 
Description=Renouvellement certificats Lets Encrypt

[Service]
ExecStart=/home/yick/scripts/renew-cert-rnmkcy.sh
```

Création du script `/home/yick/scripts/renew-cert-rnmkcy.sh` pour l'installation des certificats et la recharge du service nginx

<details>
<summary><b>Etendre Réduire renew-cert-rnmkcy.sh</b></summary>  
{% highlight shell %}
#!/bin/bash

/home/yick/.acme.sh/acme.sh --install-cert -d rnmkcy.eu \
 --key-file /etc/ssl/private/rnmkcy.eu-key.pem \
 --fullchain-file /etc/ssl/private/rnmkcy.eu-fullchain.pem \
 --reloadcmd "sudo systemctl reload nginx.service"

# Envoi information
curl \
    -H X-Email: ntfy@cinay.eu \
    -H "Title: $(date '+%d/%m/%Y %H:%M')" \
    -H "Authorization: Bearer tk_xxxxxxxxxxxxxxxxxxxxxxxx" \
    -H prio:low \
    -H tags:information_source \
    -d "Renouvellement certificats Let's Encrypt domaine rnmkcy.eu" \
    https://noti.rnmkcy.eu/yan_infos
{% endhighlight %}

</details>

Le rendre exécutable

    chmod +x /home/yick/scripts/renew-cert-rnmkcy.sh

Activation et lancement

```bash
systemctl --user daemon-reload
systemctl --user enable renew-cert-rnmkcy.path --now
```

il est possible de dire à systemd que l’instance d’un utilisateur soit tout le temps présente, du boot au shutdown, ce qui permet d’avoir des processus (services) utilisateurs qui tournent sans avoir de session ouverte.

    sudo loginctl enable-linger $USER

Voir si path actif  
![](renew-cert02.png)

## Nextcloud (cloud.rnmkcy.eu]

![](nextcloud-logo-a.png){: width="100" .left}

### Base mysql nextcloud

en mode su

```
MOTPASSEDB="Mot_passe_base_bextcloud"
mysql -uroot -e "CREATE DATABASE nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci; \
   CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY '$MOTPASSEDB'; \
   GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY '$MOTPASSEDB'; FLUSH PRIVILEGES;"
```

Pour info  
Effacer une base : `mysql -uroot -e 'DROP DATABASE nextcloud'`  
Effacer un utilisateur : `mysql -uroot -e 'DROP USER "nextcloud"@"localhost";'`  

### Nextcloud Nginx mimes

Les sections Activité et Logging dans le cadre de l’administration affiche des pages vierges

C’est une indication que votre serveur web n’est pas configuré pour gérer correctement les fichiers mjs.

Il faut modifier le fichier “mime.types” situé dans `/etc/nginx/mime.type`, remplaçer la ligne suivante:

    application/javascript js;

par

    application/javascript js mjs;

Puis redémarrer Nginx et php-fpm.

### Installer dernière version nextcloud

```bash
# mode su
wget https://download.nextcloud.com/server/releases/latest.tar.bz2
# checksum et vérification
#wget https://download.nextcloud.com/server/releases/latest.tar.bz2.sha256
#sha256sum -c latest.tar.bz2.sha256 < latest.tar.bz2 
# Décompression, déplacement et effacement
tar -xvf latest.tar.bz2
mv nextcloud /var/www/
rm latest.tar.bz2
# Utilisateur nextcloud et droits
useradd -r nextcloud
chown -R nextcloud:www-data /var/www/nextcloud
chmod -R o-rwx /var/www/nextcloud
# Nextcloud data
mkdir -p /srv/nextcloud-data/
chown -R nextcloud:nextcloud /srv/nextcloud-data/
chmod -R o-rwx /srv/nextcloud-data/
```

### Pool PHP-FPM Nextcloud

Pool PHP-FPM 8.3 Nextcloud : `/etc/php/8.3/fpm/pool.d/nextcloud.conf`
en mode su

```
[nextcloud]

user = nextcloud
group = nextcloud

chdir = /var/www/nextcloud

listen = /var/run/php/php8.3-fpm-nextcloud.sock
listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 16
pm.max_requests = 500
request_terminate_timeout = 1d


pm.start_servers = 6
pm.min_spare_servers = 5
pm.max_spare_servers = 8


; Additional php.ini defines, specific to this pool of workers.
env[PATH] = /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
php_admin_value[memory_limit] = 512M
php_value[upload_max_filesize] = 10G
php_value[post_max_size] = 10G
php_value[default_charset] = UTF-8
php_value[opcache.enable_cli]=1
php_value[opcache.memory_consumption]=256
php_value[opcache.interned_strings_buffer]=128
php_value[opcache.max_accelerated_files]=32530
php_value[opcache.save_comments]=1
php_value[opcache.revalidate_freq]=60
php_value[opcache.jit]=1255
php_value[opcache.jit_buffer_size]=128M
php_value[apc.enabled]=1
php_value[apc.enable_cli]=1
```

Avec ldap

    sudo apt install php8.3-ldap

Relancer le service

    sudo systemctl restart php8.3-fpm

### Nextcloud Vhost nginx (cloud.rnmkcy.eu)

Nexcloud sur le domaine cloud.rnmkcy.eu avec certificats Let’s Encrypt

Le fichier de configuration web cloud.rnmkcy.eu.conf `/etc/nginx/conf.d/cloud.rnmkcy.eu.conf`

<details>
<summary><b>Etendre Réduire cloud.rnmkcy.eu.conf</b></summary>

{% highlight nginx %}  
upstream php-handler {
    server unix:/var/run/php/php8.3-fpm-nextcloud.sock;
}

server {
    listen 80;
    listen [::]:80;
    server_name cloud.rnmkcy.eu;
    # enforce https
    return 301 https://$server_name:443$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name cloud.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;
    more_set_headers "Strict-Transport-Security : max-age=63072000; includeSubDomains; preload";


#    include snippets/authelia-location.conf; # Authelia auth endpoint

  
location ^~ /.well-known {
  # The following 6 rules are borrowed from `.htaccess`

  # The following 2 rules are only needed for the user_webfinger app.
  # Uncomment it if you're planning to use this app.
  #rewrite ^/\.well-known/host-meta\.json  /public.php?service=host-meta-json  last;
  #rewrite ^/\.well-known/host-meta        /public.php?service=host-meta       last;

  location = /.well-known/carddav     { return 301 /remote.php/dav/; }
  location = /.well-known/caldav      { return 301 /remote.php/dav/; }

  location = /.well-known/webfinger     { return 301 /index.php$uri; }
  location = /.well-known/nodeinfo      { return 301 /index.php$uri; }

  try_files $uri $uri/ =404;
}

#sub_path_only rewrite ^/$ / permanent;
location ^~ / {

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

  # The settings allows you to optimize the HTTP2 bandwitdth.
  # See https://blog.cloudflare.com/delivering-http-2-upload-speed-improvements/
  # for tunning hints
  client_body_buffer_size 512k;

  # HTTP response headers borrowed from Nextcloud `.htaccess`
  #more_set_headers "Strict-Transport-Security: max-age=15768000; includeSubDomains; preload;";
  more_set_headers "Referrer-Policy: no-referrer";
  more_set_headers "X-Content-Type-Options: nosniff";
  more_set_headers "X-Download-Options: noopen";
  more_set_headers "X-Frame-Options: SAMEORIGIN";
  more_set_headers "X-Permitted-Cross-Domain-Policies: none";
  more_set_headers "X-Robots-Tag: noindex, nofollow";
  more_set_headers "X-XSS-Protection: 1; mode=block";

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
  index index.php index/ /index.php$request_uri;

  # Rule borrowed from `.htaccess` to handle Microsoft DAV clients
  location = / {
      if ( $http_user_agent ~ ^DavClnt ) {
          return 302 /remote.php/webdav/$is_args$args;
      }
  }

  location = /robots.txt {
    allow all;
    log_not_found off;
    access_log off;
  }

  # Rules borrowed from `.htaccess` to hide certain paths from clients
  location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)    { return 404; }
  location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console)                  { return 404; }

  # Ensure this block, which passes PHP files to the PHP process, is above the blocks
  # which handle static assets (as seen below). If this block is not declared first,
  # then Nginx will encounter an infinite rewriting loop when it prepends
  # `/nextcloud/index.php` to the URI, resulting in a HTTP 500 error response.
  location ~ \.php(?:$|/) {
    # Required for legacy support
    rewrite ^/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|ocs-provider\/.+|.+\/richdocumentscode\/proxy|.+\/richdocumentscode_arm64\/proxy) /index.php$request_uri;
    
    fastcgi_split_path_info ^(.+?\.php)(/.*)$;
    set $path_info $fastcgi_path_info;
    
    try_files $fastcgi_script_name =404;
    
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $request_filename;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_param HTTPS on;

    fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
    fastcgi_param front_controller_active true;     # Enable pretty urls
    fastcgi_param HTTP_ACCEPT_ENCODING "";          # Disable encoding of nextcloud response to inject ynh scripts
    fastcgi_pass php-handler;

    fastcgi_intercept_errors on;
    fastcgi_request_buffering off;

    fastcgi_read_timeout 600;
    fastcgi_send_timeout 600;
    fastcgi_connect_timeout 600;
    proxy_connect_timeout 600;
    proxy_send_timeout 600;
    proxy_read_timeout 600;
    send_timeout 600;
  }

  location ~ ^/(?:updater|ocs-provider)(?:$|/) {
       try_files $uri/ =404;
       index index.php;
  }

  # Serve static files
  location ~ \.(?:css|js|mjs|svg|gif|png|jpg|ico|wasm|tflite|map)$ {
    try_files $uri / /index.php$request_uri;
    expires 6M;         # Cache-Control policy borrowed from `.htaccess`
    access_log off;     # Optional: Don't log access to assets

    location ~ \.wasm$ {
            default_type application/wasm;
    }
  }

  location ~ \.woff2?$ {
    try_files $uri / /index.php$request_uri;
    expires 7d;         # Cache-Control policy borrowed from `.htaccess`
    access_log off;     # Optional: Don't log access to assets
  }

  # Rule borrowed from `.htaccess`
    location /remote {
      return 301 /remote.php$request_uri;
    }

  location ~ / {
    if ($request_method ~ ^(PUT|DELETE|PATCH|PROPFIND|PROPPATCH)$) {
        rewrite ^ /index.php$request_uri last;
    }
    try_files $uri / /index.php$request_uri;
  }
# include snippets/authelia-authrequest.conf; # Protect this endpoint
}


    access_log /var/log/nginx/cloud.rnmkcy.eu-access.log;
    error_log /var/log/nginx/cloud.rnmkcy.eu-error.log;
}

{% endhighlight %}
</details>

Vérifier et recharger nginx : `sudo nginx -t && sudo systemctl reload nginx`

### Finaliser installation Nextcloud

> `Tout est paramétré avec le domaine ouestyan.fr qu'il faut remplacer par votre domaine`
{: .prompt-warning }  

Ouvrir le lien <https://cloud.rnmkcy.eu>  
Créer un compte administrateur et son mot de passe  
Renseigner les éléments de la base mysql  
![](cloud.rnmkcy.eu02.png){:width="200"}  ![](cloud.ouestyan.fr04.png){:width="400"}  
![](cloud.rnmkcy.eu03.png)  

Se déconnecter...  
![](cloud.rnmkcy.eu04.png)  

**Nextcloud Travaux cron**  
Vous pouvez programmer des tâches cron de trois façons : en utilisant **AJAX**, **Webcron** ou **cron**.  
La méthode recommandée est **cron**</u>.  

> Note : Il n'est pas obligatoire de sélectionner l'option Cron dans le menu d'administration pour les travaux en arrière-plan, car une fois que cron.php est exécuté à partir de la ligne de commande ou du service cron, il sera automatiquement réglé sur Cron. ![](cloud_xoyaz_xyz06.png){:width="600"}
{: .prompt-info }

Il faut créer une tâche utilisateur nextcloud pour cron

    su
    sudo -u nextcloud crontab -e

```
# tous les 5 min
*/5  *  *  *  * php -f /var/www/nextcloud/cron.php
```

**Modifier le fichier /var/www/nextcloud/config/config.php**  
Ajouter `'default_phone_region' => 'FR',` et  les lignes suivantes dans le fichier `/var/www/nextcloud/config/config.php` avant le tag de fin de fichier  `);`  

```
  'default_locale' => 'fr_FR',
  'default_phone_region' => 'FR',
  'filelocking.enabled' => true,
  'memcache.locking' => '\OC\Memcache\Redis',
  'memcache.local' => '\\OC\\Memcache\\APCu',
  'redis' => array(
    'host' => 'localhost',
    'port' => 6379,
    'timeout' => 0.0,
    'password' => '',
  ),
```

**Messagerie pour notifications**

Serveur de messagerie, ajouter les lignes dans le fichier `/var/www/nextcloud/config/config.php` avant le tag de fin de fichier  `);`   

```
  'mail_from_address' => 'yick',
  'mail_smtpmode' => 'smtp',
  'mail_sendmailmode' => 'smtp',
  'mail_domain' => 'yanfi.net',
  'mail_smtphost' => 'mx1.xoyize.xyz',
  'mail_smtpport' => '587',
  'mail_smtpauth' => 1,
  'mail_smtpname' => 'yanfi@yanfi.net',
  'mail_smtppassword' => 'Mot_de_passe_smtp_yanfi',
```

**Nextcloud maintenance_window_start**

> Le paramètre `maintenance_window_start` n'est pris en compte qu'en mode cron.
{: .prompt-warning }

Dans le fichier config/config.php, vous pouvez spécifier cette configuration. Certaines tâches de fond ne s'exécutent qu'une fois par jour. Lorsqu'une heure est définie (le fuseau horaire est UTC) pour cette configuration, les tâches d'arrière-plan qui s'annoncent comme non sensibles au temps seront retardées pendant les heures "ouvrables" et ne s'exécuteront que dans les 4 heures suivant l'heure donnée. Ceci est par exemple utilisé pour l'expiration des activités, la formation aux connexions suspectes et les vérifications de mise à jour.

> Une valeur de 1, par exemple, n'exécutera ces tâches d'arrière-plan qu'entre 01h00 UTC et 05h00 UTC.
{: .prompt-info }

Ajouter le paramètre au fichier `/var/www/nextcloud/config/config.php`

```
  'maintenance_window_start' => 1,
```

**Avertissements**

One or more mimetype migrations are available. Occasionally new mimetypes are added to better handle certain file types. Migrating the mimetypes take a long time on larger instances so this is not done automatically during upgrades. Use the command `occ maintenance:repair --include-expensive` to perform the migrations

```
# en mode su
cd /var/www/nextcloud
sudo -u nextcloud php occ maintenance:repair --include-expensive
```

Des indices facultatifs manquants ont été détectés. De nouveaux indices sont parfois ajoutés (par Nextcloud ou des applications installées) pour améliorer les performances de la base de données. L'ajout d'indices peut parfois prendre un certain temps et nuire temporairement aux performances. Cette opération n'est donc pas effectuée automatiquement lors des mises à niveau. Une fois les indices ajoutés, les requêtes sur ces tables devraient être plus rapides. Utilisez la commande `occ db:add-missing-indices` pour les ajouter.Index manquants : "systag_by_objectid" dans le tableau "systemtag_object_mapping". Pour plus d’information, voir la documentation ↗.

```
sudo -u nextcloud php occ db:add-missing-indices
```

**overwrite.cli.url**  
*Utilisez ce paramètre de configuration pour spécifier l'URL de base pour toutes les URL qui sont générées dans Nextcloud en utilisant n'importe quel type d'outils de ligne de commande (cron ou occ). La valeur doit contenir l'URL de base complète*

    sudo -u nextcloud php occ config:system:set overwrite.cli.url --value=https://cloud.rnmkcy.eu

System config value overwrite.cli.url set to string https://cloud.rnmkcy.eu

### Comptes utilisateurs nextcloud via LLDAP

On utilise LLDAP pour la gestion des utilisateurs  
Se connecter sur nextcloud avec adminyann  
Ci-dessous les paramètres

1. Host: 127.0.0.1
2. Port: 3890
3. Utilisateur DN : uid=admin,ou=people,dc=rnmkcy,dc=eu
4. Mot de passe admin LLDAP
5. DN de base: dc=rnmkcy,dc=eu

Utilisateurs , requête LLDAP : `(|(objectclass=inetOrgPerson) (memberOf=cn=nextcloud_users,ou=groups,dc=rnmkcy,dc=eu))`

Suivre la procédure [LLDAP - Gestion des utilisateurs](/posts/Nginx-PHP-MariaDB-Nextcloud_Hub/#lldap---gestion-des-utilisateurs)

### Synchronisations

Depuis le poste PC1, se connecter **yann**

![](cwwk08.png){:height="350"}

### HTTP Observatory Report 

*Cet outil, développé par Mozilla, est un observatoire qui vous aidera, en tant que propriètaire de site, à vérifier divers éléments de sécurité.
Il effectuera des test pour vérifier de nombreux éléments de sécurité
Donnera une note sous forme de lettre à votre site.
Et vous donnera aussi des directives d'amélioration et de nombreuses explications.*  
<https://observatory.mozilla.org/>

Résultat  
![](observatory_cloud.rnmkcy.eu.png)

## Authentification unique (SSO Authelia)

![](authelia-logo-64x64.png){: width="60" .left}

*L'authentification unique, souvent désignée par le sigle anglais SSO (de single sign-on) est une méthode permettant à un utilisateur d'accéder à plusieurs applications informatiques (ou sites web sécurisés) en ne procédant qu'à une seule authentification.([Authentification unique ![WikipédiA](wikipedia-logo.png){:width="20" .normal} ](https://fr.wikipedia.org/wiki/Authentification_unique))  
Authelia pour gérer les autorisations d'accès à des applications en s'appuyant sur l'annuaire __lldap__*

[SSO Authelia](/posts/Authelia_serveur_authentification_et_autorisation/)

Light Lightweight Directory Access Protocol (LLDAP) est à l'origine un protocole permettant l'interrogation et la modification des services d'annuaire. Ce protocole repose sur TCP/IP. Il a cependant évolué pour représenter une norme pour les systèmes d'annuaires, incluant un modèle de données, un modèle de nommage, un modèle fonctionnel basé sur le protocole LDAP, un modèle de sécurité et un modèle de réplication. C'est une structure arborescente dont chacun des nœuds est constitué d'attributs associés à leurs valeurs

Avec une solution SSO, le login est redirigé vers une seule application et l'utilisateur se verra toujours présenter la même page de connexion quelque soient les applications finales qu'il utilisera.   
![](authelia-page-cnx.png){:width="450" .normal}

### LLdap - Gestionnaire web (lldap.rnmkcy.eu)

![](ldap-logo.png){: width="100" .left} 

*Gestion de l'annuaire lldap des utilisateurs [(cwwk - Service LLdap)](/posts/LLDAP_simple_serveur_authentification/)*

Domaine  **lldap.home.arpa** ([Domaine home.arpa](/posts/home.arpa/))

Ajout au fichier `/etc/unbound/unbound.conf.d/local-unbound.conf`

```
    local-data: "lldap.home.arpa.  86400 IN A 192.168.0.205"
    local-data-ptr: "192.168.10.205 86400 lldap.home.arpa."
```

Redémarrer unbound

```shell
sudo systemctl restart unbound
```

Générer les certificats **lldap.home.arpa**

```shell
# génération
cd ~
./gensert.sh lldap.home.arpa
# déplacer certificat et clé
sudo mv lldap.home.arpa.key /etc/ssl/private/
sudo mv lldap.home.arpa.crt /etc/ssl/private/
```

Création proxy nginx pour le gestionnaire web LLdap 

    /etc/nginx/conf.d/lldap.home.arpa.conf

```
server {
    listen 80;
    listen [::]:80;
    server_name  lldap.home.arpa;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://lldap.home.arpa$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  lldap.home.arpa;

    ssl_certificate      /etc/ssl/private/lldap.home.arpa.crt;
    ssl_certificate_key  /etc/ssl/private/lldap.home.arpa.key;

    # connexion nginx fermée si sous domaine inexistant
    if ($http_host != "lldap.home.arpa") {
     return 444;
    }

  location / { 
     proxy_pass              http://127.0.0.1:17170;
  } 

}
```

Vérifier et recharger nginx: `sudo nginx -t && sudo systemctl reload nginx`  
Accès par le lien <https://lldap.home.arpa/>  
![](auth.ouestline.xyz.png){:width="300"}

### Authelia SSO+Nginx (auth.rnmkcy.eu)

[SSO Authelia](/posts/Authelia_serveur_authentification_et_autorisation/)

Modifier **authentication_backend** dans le fichier de configuration **authelia** pour communiquer avec le serveur lldap

```yaml
authentication_backend:
  # Password reset through authelia works normally.
  password_reset:
    disable: false
  # How often authelia should check if there is an user update in LDAP
  refresh_interval: 1m
  ldap:
    implementation: custom
    # Pattern is ldap://HOSTNAME-OR-IP:PORT
    # Port de saut normal est 389, standard dans LLDAP est 3890
    url: ldap://127.0.0.1:3890
    # The dial timeout for LDAP.
    timeout: 5s
    # Utilisez StartTLS avec la connexion LDAP, TLS est non supporté maintenant
    start_tls: false
    #tls:
    #  skip_verify: false
    #  minimum_version: TLS1.2
    # Set base dn, like dc=google,dc.com
    base_dn: dc=rnmkcy,dc=eu
    username_attribute: uid
    # Vous devez configurer cela à ou=people, parce que tous les utilisateurs sont stockés dans cette ou!
    additional_users_dn: ou=people
    # Pour permettre la connexion avec le nom d'utilisateur et l'email
    # on peut utiliser un filtre comme
    # (&(|({username_attribute}={input})({mail_attribute}={input}))(objectClass=person))
    users_filter: "(&({username_attribute}={input})(objectClass=person))"
    # Définir ceci à ou=groups, parce que tous les groupes sont stockés dans cette ou
    additional_groups_dn: ou=groups
    # Les groupes ne sont pas affichés dans l'interface utilisateur, mais ce filtre fonctionne.
    groups_filter: "(member={dn})"
    # L'attribut tenant le nom du groupe.
    group_name_attribute: cn
    # Attribut email
    mail_attribute: mail
    # L'attribut tenant le nom d'affichage de l'utilisateur. 
    # Cela sera utilisé pour accueillir un utilisateur authentifié.
    display_name_attribute: displayName
    # Le nom d'utilisateur et mot de passe de l'utilisateur admin.
    # "admin" devrait être le nom d'utilisateur administrateur que vous définissez dans la configuration LLDAP
    user: uid=admin,ou=people,dc=rnmkcy,dc=eu
    # Le mot de passe peut également être défini en utilisant un secret:
    # https://www.authelia.com/docs/configuration/secrets/
    password: '<Mot de passe administrateur ldap>'
```


Le fichier complet de configuration authelia `/etc/authelia/configuration.yml`

<details>
<summary><b>Etendre Réduire</b> <i>configuration.yml</i></summary>
{% highlight yaml %}  
###############################################################################
#                           Authelia Configuration                            #
###############################################################################

theme: dark
##
## Identity Validation Configuration
##
## This configuration tunes the identity validation flows.
identity_validation:
  ## Reset Password flow. Adjusts how the reset password flow operates.
  reset_password:
    jwt_secret: "Générer avec la commande : tr -cd '[:alnum:]' < /dev/urandom | fold -w 64 | head -n 1 | tr -d '\n' ; echo"

server:
  #host: 127.0.0.1
  #port: 9091
  #path: ""
  address: 'tcp://127.0.0.1:9091/'
  disable_healthcheck: false
  tls:
    key: ""
    certificate: ""
  ## Server Endpoints configuration.
  ## This section is considered advanced and it SHOULD NOT be configured unless you've read the relevant documentation.
  endpoints:
    ## Enables the pprof endpoint.
    enable_pprof: false
    ## Enables the expvars endpoint.
    enable_expvars: false

log:
  level: info
  file_path: '/etc/authelia/authelia.log'

totp:
  issuer: rnmkcy.eu
  period: 30
  skew: 1

##
## WebAuthn Configuration
##
webauthn:
  ## Désactiver Webauthn.
  disable: false

  ## Ajuster le délai d'interaction pour les dialogues Webauthn.
  timeout: 60s

  ## Le nom d'affichage que le navigateur doit montrer à l'utilisateur lorsqu'il utilise
  ##  Webauthn pour se connecter ou s'enregistrer.
  display_name: Authelia

  ## La préférence de transmission contrôle si nous collectons la déclaration d'attestation,
  ##  y compris l'AAGUID, à partir de l'appareil.
  ## Les options sont none, indirect, direct.
  attestation_conveyance_preference: indirect

  ## Contrôle si l'utilisateur doit faire un geste ou une action pour confirmer sa présence.
  ## Les options sont : required, preferred, discouraged.
  user_verification: preferred


##
## authentification LLDAP
##
authentication_backend:
  password_reset:
    disable: false
  refresh_interval: 1m
  ldap:
    implementation: custom
    address: ldap://127.0.0.1:3890
    timeout: 5s
    start_tls: false
    base_dn: dc=rnmkcy,dc=eu
    additional_users_dn: ou=people
    users_filter: "(&({username_attribute}={input})(objectClass=person))"
    additional_groups_dn: ou=groups
    groups_filter: "(member={dn})"
    user: uid=admin,ou=people,dc=rnmkcy,dc=eu
    password: 'Mot_de_passe_LLDAP'
    attributes:
      mail: 'mail'
      username: 'uid'
      group_name: 'cn'
      display_name: 'displayName'

access_control:
  default_policy: deny
  rules:
    ## bypass rule
    - domain:
        - "auth.rnmkcy.eu"
        - "cloud.rnmkcy.eu"
        - "searx.rnmkcy.eu"
      policy: bypass
    ## catch-all
    - domain:
        - "calibre.rnmkcy.eu"
      policy: one_factor
    - domain:
        - "site.rnmkcy.eu"
        - "cockpit.rnmkcy.eu"
      policy: two_factor

session:
  secret: 'Générer avec la commande : tr -cd '[:alnum:]' < /dev/urandom | fold -w 64 | head -n 1 | tr -d '\n' ; echo'
  name: 'authelia_session'
  same_site: 'lax'
  inactivity: '45m'
  expiration: '12h'
  remember_me: '2M'
  cookies:
    - domain: 'rnmkcy.eu'
      authelia_url: 'https://auth.rnmkcy.eu'
      default_redirection_url: 'https://rnmkcy.eu'
      name: 'authelia_session'
      same_site: 'lax'
      inactivity: '45m'
      expiration: '12h'
      remember_me: '1d'
  redis:
    host: localhost
    port: 6379
    #password: ""
    database_index: 0
    maximum_active_connections: 10
    minimum_idle_connections: 0

regulation:
  max_retries: 3
  find_time: 10m
  ban_time: 12h

storage:
  encryption_key: "Générer avec la commande : tr -cd '[:alnum:]' < /dev/urandom | fold -w 64 | head -n 1 | tr -d '\n' ; echo"
  mysql:
    address: '127.0.0.1:3306'
    database: authelia
    username: authelia
    password: "Mot_de_passe_base_mysql_authelia"

#notifier:
#  disable_startup_check: false
#
#  filesystem:
#    filename: /etc/authelia/notification.txt

notifier:
  disable_startup_check: false

  smtp:
    username: "yanfi@yanfi.net"
    ## Password can also be set using a secret: https://www.authelia.com/docs/configuration/secrets/
    password: "Mot_de_passe_smtp_yanfi"
    address: "submission://mx1.xoyize.xyz:587"
    sender: "yanfi@yanfi.net"
    ## Subject configuration of the emails sent. {title} is replaced by the text from the notifier.
    subject: "[Authelia] {title}"
{% endhighlight %}
</details>

Les fichiers de configuration nginx pour authelia  
![](authelia09.png)

<details>
<summary><b>Etendre Réduire</b> <i>/etc/nginx/snippets/authelia-authrequest.conf</i></summary>
{% highlight nginx %}  
# Basic Authelia Config
# Send a subsequent request to Authelia to verify if the user is authenticated
# and has the right permissions to access the resource.
auth_request /authelia;
# Set the `target_url` variable based on the request. It will be used to build the portal
# URL with the correct redirection parameter.
auth_request_set $target_url $scheme://$http_host$request_uri;
# Set the X-Forwarded-User and X-Forwarded-Groups with the headers
# returned by Authelia for the backends which can consume them.
# This is not safe, as the backend must make sure that they come from the
# proxy. In the future, it's gonna be safe to just use OAuth.
auth_request_set $user $upstream_http_remote_user;
auth_request_set $groups $upstream_http_remote_groups;
auth_request_set $name $upstream_http_remote_name;
auth_request_set $email $upstream_http_remote_email;
proxy_set_header Remote-User $user;
proxy_set_header Remote-Groups $groups;
proxy_set_header Remote-Name $name;
proxy_set_header Remote-Email $email;
# If Authelia returns 401, then nginx redirects the user to the login portal.
# If it returns 200, then the request pass through to the backend.
# For other type of errors, nginx will handle them as usual.
error_page 401 =302 https://auth.rnmkcy.eu/?rd=$target_url;
{% endhighlight %}
</details>

<details>
<summary><b>Etendre Réduire</b> <i>/etc/nginx/snippets/authelia-location.conf</i></summary>
{% highlight nginx %}  
# Virtual endpoint created by nginx to forward auth requests.
location /authelia {
    internal;
    set $upstream_authelia http://127.0.0.1:9091/api/verify;
    proxy_pass_request_body off;
    proxy_pass $upstream_authelia;    
    proxy_set_header Content-Length "";

    # Timeout if the real server is dead
    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;

    # [REQUIRED] Needed by Authelia to check authorizations of the resource.
    # Provide either X-Original-URL and X-Forwarded-Proto or
    # X-Forwarded-Proto, X-Forwarded-Host and X-Forwarded-Uri or both.
    # Those headers will be used by Authelia to deduce the target url of the     user.
    # Basic Proxy Config
    client_body_buffer_size 128k;
    proxy_set_header Host $host;
    proxy_set_header X-Original-URL $scheme://$http_host$request_uri;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr; 
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Forwarded-Uri $request_uri;
    proxy_set_header X-Forwarded-Ssl on;
    proxy_redirect  http://  $scheme://;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_cache_bypass $cookie_session;
    proxy_no_cache $cookie_session;
    proxy_buffers 4 32k;

    # Advanced Proxy Config
    send_timeout 5m;
    proxy_read_timeout 240;
    proxy_send_timeout 240;
    proxy_connect_timeout 240;
}
{% endhighlight %}
</details>

<details>
<summary><b>Etendre Réduire</b> <i>/etc/nginx/snippets/proxy.conf</i></summary>
{% highlight nginx %}  
## Headers
proxy_set_header Host $host;
proxy_set_header X-Original-URL $scheme://$http_host$request_uri;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Host $http_host;
proxy_set_header X-Forwarded-Uri $request_uri;
proxy_set_header X-Forwarded-Ssl on;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header Connection "";

## Basic Proxy Configuration
client_body_buffer_size 128k;
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503; ## Timeout if the real server is dead.
proxy_redirect  http://  $scheme://;
proxy_http_version 1.1;
proxy_cache_bypass $cookie_session;
proxy_no_cache $cookie_session;
proxy_buffers 64 256k;

## Trusted Proxies Configuration
## Please read the following documentation before configuring this:
##     https://www.authelia.com/integration/proxies/nginx/#trusted-proxies
# set_real_ip_from 10.0.0.0/8;
# set_real_ip_from 172.16.0.0/12;
# set_real_ip_from 192.168.0.0/16;
# set_real_ip_from fc00::/7;
real_ip_header X-Forwarded-For;
real_ip_recursive on;

## Advanced Proxy Configuration
send_timeout 5m;
proxy_read_timeout 360;
proxy_send_timeout 360;
proxy_connect_timeout 360;
{% endhighlight %}
</details>

Le sso est en place , il faut modifier les fichiers de configuration nginx des applications concernées

### Protection des applications

* Ne pas oublier de recharger nginx après chaque modification des fichiers de configuration : `sudo systemctl reload nginx`
* Ajouter le site dans le fichier de configuration `/etc/authelia/configuration.yml` entre **domain** et **policy: one_factor** de la balise  **acces_control** entre  
* Relancer le service authelia : `sudo systemctl restart authelia`

## Domaines et sous Domaines rnmkcy.eu

### Dossier racine rnmkcy.eu

![](cwwk-logob.png){: width="50" .left}  


    /etc/nginx/conf.d/rnmkcy.eu.conf

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;
    add_header Content-Security-Policy "script-src 'self' *.rnmkcy.eu; object-src 'self' *.rnmkcy.eu";

    root /sharenfs/rnmkcy/racine/ ;
        location / {
            index index.htm index/ index.php;
        }
  location ~ \.php(?:$|/) {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $request_filename;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_param HTTPS on;

    fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
    fastcgi_param front_controller_active true;     # Enable pretty urls
    fastcgi_param HTTP_ACCEPT_ENCODING "";          # Disable encoding of nextcloud response to inject ynh scripts
    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    fastcgi_intercept_errors on;
    fastcgi_request_buffering off;
  }

}
```

### Site statique jekyll (static.rnmkcy.eu)

Regroupe static site diceware et cartes

    /etc/nginx/conf.d/static.rnmkcy.eu.conf 

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name static.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;
    root /sharenfs/multimedia/Divers/static/;

    location / {
      index index.htm index/ index.php;
		  location ~ \.php(?:$|/) {
		    include fastcgi_params;
		    fastcgi_param SCRIPT_FILENAME $request_filename;
		    fastcgi_split_path_info ^(.+\.php)(/.+)$;
		    fastcgi_param HTTPS on;
		
		    fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
		    fastcgi_param front_controller_active true;     # Enable pretty urls
		    fastcgi_param HTTP_ACCEPT_ENCODING "";          # Disable encoding of nextcloud response to inject ynh scripts
		    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
		    fastcgi_intercept_errors on;
		    fastcgi_request_buffering off;
		  }
    }
}
```

### Site statique jekyll (chirpy.rnmkcy.eu)

    /etc/nginx/conf.d/chirpy.rnmkcy.eu.conf 

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name chirpy.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;
    root /sharenfs/multimedia/Divers/_site/;

    location / {
      index index.html;
    }
}
```

### Mot passe haute entropie (dice.rnmkcy.eu)

    /etc/nginx/conf.d/dice.rnmkcy.eu.conf 

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name dice.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;
    root /sharenfs/multimedia/Divers/diceware/;

    location / {
      index index.htm index/;
    }
}
```

### Leaflet cartographie osm-new (osm.rnmkcy.eu)

    /etc/nginx/conf.d/osm.rnmkcy.eu.conf 

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name osm.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;
    root /sharenfs/multimedia/Divers/osm-new/;

    location / {
      index index.htm index/;
    }
}
```

Recharger : `sudo systemctl reload nginx`

### Shaarli liens (shaarli.rnmkcy.eu)

*[Shaarli](https://shaarli.readthedocs.io/en/master/), le service personnel, minimaliste, super rapide, sans base de données, signet.*

![](shaarli_logo.png){: width="100" .left}

**Installer Shaarli**

Pour installer Shaarli, il suffit de placer les fichiers de la [dernière archive .zip](https://github.com/shaarli/Shaarli/releases) sous la racine du document de votre serveur web (directement à la racine du document, ou dans un sous-répertoire). Téléchargez l'archive shaarli-vX.X.X-full pour y inclure les dépendances.

```bash
wget https://github.com/shaarli/Shaarli/releases/download/v0.13.0/shaarli-v0.13.0-full.zip
unzip shaarli-v0.13.0-full.zip
sudo rsync -avP Shaarli/ /var/www/shaarli.rnmkcy.eu/
```

**Définir les permissions de fichier**

Quelle que soit la méthode d'installation, les autorisations de fichiers appropriées doivent être définies:

```bash
sudo chown -R root:www-data /var/www/shaarli.rnmkcy.eu
sudo chmod -R g+rX /var/www/shaarli.rnmkcy.eu
sudo chmod -R g+rwX /var/www/shaarli.rnmkcy.eu/{cache/,data/,pagecache/,tmp/}
```

**nginx php-fpm**

Installer si nécessaire nginx et php-fpm

    sudo apt install nginx php-fpm

Extensions PHP nécessaires  
![](shaarli-php-extensions.png)

Installer

    sudo apt install php8.3-xml php8.3-common php8.3-gd php8.3-intl php8.3-curl php8.3-mbstring

Le fichier php fpm `/etc/php/8.3/fpm/pool.d/shaarli.conf`

```
[shaarli]

user = www-data
group = www-data

listen = /var/run/php/php8.3-fpm-shaarli.sock

listen.owner = www-data
listen.group = www-data

pm = dynamic
pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3

; Default Value: current directory or / when chroot
chdir = /var/www/shaarli.rnmkcy.eu
```

Modifier le fichier de configuration de virtualhost

    sudo nano /etc/nginx/conf.d/shaarli.rnmkcy.eu.conf

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name  shaarli.rnmkcy.eu;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://shaarli.rnmkcy.eu$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  shaarli.rnmkcy.eu;
    root         /var/www/shaarli.rnmkcy.eu;

    # log file locations
    # combined log format prepends the virtualhost/domain name to log entries
    access_log  /var/log/nginx/access.log combined;
    error_log   /var/log/nginx/error.log;

    include /etc/nginx/conf.d/security.conf.inc;
    #include snippets/authelia-location.conf; # Authelia auth endpoint

    # increase the maximum file upload size if needed: by default nginx limits file upload to 1MB (413 Entity Too Large error)
    client_max_body_size 100m;

    # relative path to shaarli from the root of the webserver
    # if shaarli is installed in a subdirectory of the main domain, edit the location accordingly
    location / {
        # default index file when no file URI is requested
        index index.php;
        try_files _ /index.php$is_args$args;
    }

    location ~ (index)\.php$ {
        try_files $uri =404;
        # slim API - split URL path into (script_filename, path_info)
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        # pass PHP requests to PHP-FPM
        fastcgi_pass   unix:/var/run/php/php8.3-fpm-shaarli.sock;
        fastcgi_index  index.php;
        include        fastcgi.conf;
     #include snippets/authelia-authrequest.conf; # Protect this endpoint
    }

    location ~ /doc// {
        default_type "text/";
        try_files $uri $uri/ $uri/ =404;
    }

    location = /favicon.ico {
        # serve the Shaarli favicon from its custom location
        alias /var/www/shaarli/images/favicon.ico;
    }

    # allow client-side caching of static files
    location ~* \.(?:ico|css|js|gif|jpe?g|png|ttf|oet|woff2?)$ {
        expires    max;
        add_header Cache-Control "public, must-revalidate, proxy-revalidate";
        # HTTP 1.0 compatibility
        add_header Pragma public;
    }
}
```

Vérifier

    sudo nginx -t

Recharger les configurations php-fpm nginx

    sudo systemctl reload php8.3-fpm nginx

**Configurer Shaarli**

Ouvrir le lien <https://shaarli.rnmkcy.eu>

![](shaarli.rnmkcy.eu01.png)  
![](shaarli.rnmkcy.eu02.png)  

Après avoir cliqué sur "Install" on arrive sur la page de connexion  
![](shaarli.rnmkcy.eu03.png)  

**Pour une utilisation avec Lldap**(A VERIFIER)  
IP_SRV_LLDAP=127.0.0.1  
Si utilisation serveur LLDAP, ajouter les lignes suivantes au fichier `/var/www/shaarli.rnmkcy.eu/data/config.json.php`

```
    "ldap": {
        "host": "ldap://127.0.0.1:3890",
        "dn": "uid=%s,ou=people,dc=domain,dc=com"
    }
```

### Linkwarden (link.home.arpa)

**cwwk - Domaine link.home.arpa**  

Générer les certificats link.home.arpa 

```shell
cd /home/yick/
./gencert.sh link.home.arpa
# Une passphrase est demandée (dans cle.txt)
```

A l'issue de cette commande des fichiers sont générés

```
├── link.home.arpa.crt
├── link.home.arpa.csr
├── link.home.arpa.ext
├── link.home.arpa.key
```

Copier les fichiers link.home.arpa.crt et link.home.arpa.key dans /etc/ssl/private/

```shell
sudo cp link.home.arpa.{crt,key} /etc/ssl/private/
```

Ajout à la configuration des accès réseaux locaux dans le fichier `/etc/unbound/unbound.conf.d/local-unbound.conf`

```
    # IPv4
    local-data: "link.home.arpa.  86400 IN A 192.168.0.205"

    # On ajoute les enregistrements PTR
    # PTR IPv4
    local-data-ptr: "192.168.0.205 86400 link.home.arpa."
```

**Vérifications**  
Vérifier la configuration

```shell
sudo unbound-checkconf /etc/unbound/unbound.conf.d/local-unbound.conf
```

`unbound-checkconf: no errors in /etc/unbound/unbound.conf.d/local-unbound.conf`

Redémarrer Unbound

```shell
sudo systemctl restart unbound
```

on peut faire quelques tests

```shell
dig link.home.arpa
```

La configuration du proxy  `/etc/nginx/conf.d/link.home.arpa.conf` 

```
server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  link.home.arpa;

    ssl_certificate      /etc/ssl/private/link.home.arpa.crt;
    ssl_certificate_key  /etc/ssl/private/link.home.arpa.key;

    ##################################################
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    # modern configuration
    ssl_protocols TLSv1.3;
    ssl_ecdh_curve X25519:prime256v1:secp384r1;
    ssl_prefer_server_ciphers off;

    # Ajouts 2025
    more_set_headers "Strict-Transport-Security : max-age=63072000; includeSubDomains; preload";
    more_set_headers "Referrer-Policy: no-referrer";
    more_set_headers "X-Content-Type-Options: nosniff";
    more_set_headers "X-Download-Options: noopen";
    more_set_headers "X-Frame-Options: SAMEORIGIN";
    more_set_headers "X-Permitted-Cross-Domain-Policies: none";
    more_set_headers "X-Robots-Tag: noindex, nofollow";
    more_set_headers "X-XSS-Protection: 1; mode=block";
    # Fin ajouts 2025

    # replace with the IP address of your resolver
    resolver 192.168.0.205 valid=300s;
    resolver_timeout 5s;
    ##################################################


   location / {
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $remote_addr;
      proxy_set_header X-Forwarded-Proto $scheme;

	 proxy_buffering off;
	 proxy_request_buffering off;
	 proxy_http_version 1.1;
	 proxy_intercept_errors on;
      
      proxy_pass http://192.168.0.216:3000;
   }

}
```

Vérifier

```shell
sudo nginx -t
```

Redémarrer le serveur

```shell
sudo systemctl reload nginx
```

Accessible par le lien <https://link.home.arpa>  

### Gitea (gitea.rnmkcy.eu)

*Gitea est une forge logicielle libre en Go sous licence MIT, pour l'hébergement de développement logiciel, basé sur le logiciel de gestion de versions Git pour la gestion du code source, comportant un système de suivi des bugs, un wiki, ainsi que des outils pour la relecture de code.*  
![](gitea-logo.png){: width="100" .left}  

* [Gitea](/posts/Gitea/)


/etc/nginx/conf.d/gitea.rnmkcy.eu.conf

```
server {
    listen 80;
    listen [::]:80;
    server_name  gitea.rnmkcy.eu;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://gitea.rnmkcy.eu$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  gitea.rnmkcy.eu;
    client_max_body_size 100M;
    include /etc/nginx/conf.d/security.conf.inc;

  location / { 
      proxy_pass              http://127.0.0.1:3000;
  } 

}
```

### Gpx studio (gpx.rnmkcy.eu)

![](gpxstudio-logo.png){: width="150" .left}  


* [Moteur de routage (BRouter) + Visualisation et édition traces gpx (gpx.studio)](/posts/Visualisation_et_edition_des_traces_gpx_studio/)

Après la construction on copie le dossier build sous /var/www/gpx

```bash
sudo cp -r /sharenfs/rnmkcy/gpx.studio/website/build /var/www/gpx.studio
# les droits
sudo chown www-data:www-data -R /var/www/gpx.studio
```

Le site est accessible dans le dossier /sharenfs/rnmkcy/gpx.studio/website/build/
Configuration nginx /etc/nginx/conf.d/gpx.rnmkcy.eu.conf

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name gpx.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;
    root /var/www/gpx.studio/;

    location / {
      index index/;
    }
}
```

Vérifier et recharger nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Accès par le lien https://gpx.rnmkcy.eu

### Divers et tests (site.rnmkcy.eu)

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name site.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;
    # Authelia
    include snippets/authelia-location.conf; # Authelia auth endpoint
    root /sharenfs/rnmkcy/site/;

    location / {
      index index/ index.php /_h5ai/public/index.php;
		  location ~ \.php(?:$|/) {
		    include fastcgi_params;
		    fastcgi_param SCRIPT_FILENAME $request_filename;
		    fastcgi_split_path_info ^(.+\.php)(/.+)$;
		    fastcgi_param HTTPS on;
		
		    fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
		    fastcgi_param front_controller_active true;     # Enable pretty urls
		    fastcgi_param HTTP_ACCEPT_ENCODING "";          # Disable encoding of nextcloud response to in>
		    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
		    fastcgi_intercept_errors on;
		    fastcgi_request_buffering off;
		  }
          # Authelia
	  include snippets/authelia-authrequest.conf; # Protect this endpoint
	}
}
```

Accès par le lien https://site.rnmkcy.eu

### Calibre web (calibre.rnmkcy.eu)

![](Calibre_logo.png){:height="60"}  

* [Calibre web (calibre.rnmkcy.eu)](/posts/calibre-web-gestion-livres/#calibre-web-calibrernmkcyeu)

Les instructions pour une installation de calibre web

```bash
sudo apt install python3-venv python3-dev 
sudo apt install build-essential libssl-dev libffi-dev 
sudo apt install libsasl2-dev libldap2-dev
# Création dossier puis un environnement virtuel pour calibre-web
sudo mkdir -p /sharenfs/rnmkcy/calibreweb
sudo chown $USER:$USER /sharenfs/rnmkcy/calibreweb
python3 -m venv /sharenfs/rnmkcy/calibreweb
# Activer l’environnement
source /sharenfs/rnmkcy/calibreweb/bin/activate
# nouveau prompt -> (calibreweb) yick@cwwk:~$ 
pip3 install --upgrade pip
pip3 install wheel
pip3 install cmake
pip3 install calibreweb
# ldap
pip3 install python-ldap
pip3 install Flask-SimpleLDAP

```

Utilisation fichier systemd pour le lancement automatique

    sudo nano /etc/systemd/system/calibreweb.service

Contenu du fichier

```
[Unit]
Description=Service calibreweb
After=network.target

[Service]
Type=simple
User=yick
ExecStart=/sharenfs/rnmkcy/calibreweb/bin/cps

[Install]
WantedBy=multi-user.target
```

> `ATTENTION! , User est l’utilisateur connecté ($USER)`
{: .prompt-warning }

Recharger et activer le service calibreweb 

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now calibreweb
```

Vérifie 

```bash
systemctl status calibreweb
```

**proxy nginx + authelia**

/etc/nginx/conf.d/calibre.rnmkcy.eu.conf

```
server {
    listen 80;
    listen [::]:80;
    server_name  calibre.rnmkcy.eu;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://calibre.rnmkcy.eu$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  calibre.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;
    include snippets/authelia-location.conf; # Authelia auth endpoint

#  location / { 
#     proxy_pass              http://127.0.0.1:8083;
#     include snippets/authelia-authrequest.conf; # Protect this endpoint
#  } 
            location / { # Reduced to "/"
                proxy_pass              http://127.0.0.1:8083;
                proxy_set_header        Host            $http_host;
                proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header        X-Scheme        $scheme;
                include snippets/authelia-authrequest.conf; # Protect this endpoint
                # Removed proxy_set_header
        }

}
```


suite...

* [Calibre web proxy nginx](/posts/calibre-web-gestion-livres/#proxy-nginx-1)

![](observatory_calibre.rnmkcy.eu.png)

### Traduction LibreTranslate (traduction.rnmkcy.eu)

![](LibreTranslate.png){:height="60"}  
<https://github.com/LibreTranslate/LibreTranslate>

* [LibreTranslate API de traduction](/posts/LibreTranslate/)

Les instructions pour une installation rapide

```bash
#sudo apt install python3-venv python3-dev 
#sudo apt install build-essential libssl-dev libffi-dev 
#sudo apt install libsasl2-dev libldap2-dev
sudo apt install python3-msgpack 
# Création dossier puis un environnement virtuel pour calibre-web
sudo mkdir -p /sharenfs/rnmkcy/libretranslate
sudo chown $USER:$USER /sharenfs/rnmkcy/libretranslate
python3 -m venv /sharenfs/rnmkcy/libretranslate
# Activer l’environnement
source /sharenfs/rnmkcy/libretranslate/bin/activate
# nouveau prompt -> (libretranslate) yick@cwwk:~$ 
pip3 install --upgrade pip
pip3 install libretranslate

```

Tester libretranslate

	libretranslate [args]

```
Loaded support for 46 languages (92 models total)!
Running on http://127.0.0.1:5000
```

Puis ouvrez un navigateur Web à http://localhost:5000

    ssh -L 9500:127.0.0.1:5000 yick@192.168.0.205 -p 55205 -i /home/yann/.ssh/yick-ed25519

![](cwwk10.png)

**Créer un service LibreTranslate**  
pour s'assurer que notre application fonctionne juste après le démarrage du système.

    sudo nano /etc/systemd/system/LibreTranslate.service

```
[Unit]
Description=Traduction libre
 
[Service]
Type=exec
User=yick
WorkingDirectory=/sharenfs/rnmkcy/libretranslate
ExecStart=/sharenfs/rnmkcy/libretranslate/bin/libretranslate
Restart=on-failure
RestartSec=30s
 
# Optional hardening to improve security
ReadWritePaths=/sharenfs/rnmkcy/libretranslate
 
[Install]
WantedBy=multi-user.target
```

Le fichier d'unité, le fichier de configuration source ou les drop-ins de `LibreTranslate.service` ont changé sur le disque. Exécutez 'sudo systemctl daemon-reload' pour recharger les unités.

    sudo systemctl daemon-reload

Activer et démarrer le service

    sudo systemctl enable LibreTranslate --now

Status

    sudo systemctl status LibreTranslate

![](traduire.rnmkcy.eu02.png)

**Proxy nginx**  
Le fichier de configuration nginx `/etc/nginx/conf.d/traduction.rnmkcy.eu.conf` 

```
server {
    listen 80;
    listen [::]:80;
    server_name  traduction.rnmkcy.eu;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://traduction.rnmkcy.eu$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  traduction.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;

  location / { 
      proxy_pass              http://127.0.0.1:5000;
  } 

}
```

Vérifier nginx

    sudo nginx -t

Redémarrer nginx

    sudo systemctl reload nginx

Lien <https://traduction.rnmkcy.eu>  
![](traduire.rnmkcy.eu03.png)

## Machines virtuelles

Machines virtuelles utilisées pour gérer les applications:

* Collabora (Collabora 192.168.100.40 vm-debian12)
* ttrss (Flux RSS ttrss 192.168.100.60 alpine-vm)
* ntfy (Notification ntfy 192.168.100.50 vm-ntfy)
* Searx (SearXNG 192.168.100.70 alpine-searx)
* Audio Navidrome (Navidrome vm-alpine01 192.168.10.210)

### Collabora Online (bora.rnmkcy.eu)

![](collabora_logo.png){: width="70" .left}

*Installer collabora serveur sur une machine virtuelle debian 12*

* [cwwk KVM - Serveur Debian 12 (image cloud "generic")](/posts/cwwk-KVM-debian_bookworm/)  
Machine virtuelle **vm-debian12** (192.168.100.40) qui héberge le serveur **collabora online**
* Installation de **collabora online** sur le serveur virtuel debian12 **vm-debian12**, voir le lien [Collabora](/posts/Collabora_Debian/)  

Proxy nginx `/etc/nginx/conf.d/bora.rnmkcy.eu.conf`

```
server {
    listen 80;
    listen [::]:80;
    server_name  bora.rnmkcy.eu;

  # redirect all plain HTTP requests to HTTPS
  return 301 https://bora.rnmkcy.eu$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name bora.rnmkcy.eu;

    # Sécurité + https
    include /etc/nginx/conf.d/security.conf.inc;

    error_log /var/log/nginx/bora.error;

    # static files
    location ^~ /browser {
    proxy_pass http://192.168.100.40:9980;
    proxy_set_header Host $http_host;
    }

    # WOPI discovery URL
    location ^~ /hosting/discovery {
    proxy_pass http://192.168.100.40:9980;
    proxy_set_header Host $http_host;
    }

    # Capabilities
    location ^~ /hosting/capabilities {
    proxy_pass http://192.168.100.40:9980;
    proxy_set_header Host $http_host;
    }

    # main websocket
    location ~ ^/cool/(.*)/ws$ {
    proxy_pass http://192.168.100.40:9980;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Host $http_host;
    proxy_read_timeout 36000s;
    }

    # download, presentation and image upload
    location ~ ^/(c|l)ool {
    proxy_pass http://192.168.100.40:9980;
    proxy_set_header Host $http_host;
    }

    # Admin Console websocket
    location ^~ /cool/adminws {
    proxy_pass http://192.168.100.40:9980;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Host $http_host;
    proxy_read_timeout 36000s;
    }
}
```

### Notifications (noti.rnmkcy.eu)

![](ntfy-logo.png){:width="50" .left}  
*[Ntfy](https://ntfy.sh/), qui se prononce “notify”, est un service de notification ultra léger, permettant d'envoyer des messages vers un smartphone ou un ordinateur via de simples scripts, sans besoin de compte et totalement gratuitement !*  
[cwwk KVM - Service de notification ntfy (Alpine Linux vm-ntfy)](/posts/KVM-Alpine-Linux-1/)

* [Publishing](https://docs.ntfy.sh/publish/)
* [Emoji reference](https://docs.ntfy.sh/emojis/)
* [UTF-8 emojis in bash or zsh](https://gist.github.com/BuonOmo/77b75349c517defb01ef1097e72227af)
* [How to Use the Command 'ntfy' (with examples)](https://commandmasters.com/commands/ntfy-common/)

**Proxy nginx noti.rnmkcy.eu**

Sur le serveur cwwk rnmkcy.eu, créer un proxy `/etc/nginx/conf.d/noti.rnmkcy.eu.conf`

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name  noti.rnmkcy.eu;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://noti.rnmkcy.eu$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  noti.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;

  location / { 
    proxy_pass              http://192.168.100.50:8080;
    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_connect_timeout 3m;
    proxy_send_timeout 3m;
    proxy_read_timeout 3m;

    client_max_body_size 0; # Stream request body to backend
  } 

}

```

Recharger nginx

    sudo systemctl reload nginx

Connexion depuis un navigateur <https://noti.rnmkcy.eu>   
![](ntfy01.png)

Envoi de notifications depuis le serveur cwwk avec curl en ligne de commande

```bash
curl -d "Backup successful 😀" https://noti.rnmkcy.eu/yan_infos
```

### Flux Tiny Tiny RSS (rss.rnmkcy.eu)

![](ttrss-logo-a.png){: .left}  
[cwwk KVM - Alpine Linux Tiny Tiny RSS (ttrss alpine-vm)](/posts/KVM-Alpine-Linux/)

Fichier de configuration nginx

    /etc/nginx/conf.d/rss.rnmkcy.eu.conf 

```
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

### Moteur de recherche (searx.rnmkcy.eu)

>Le projet n'est plus maintenu
{: .prompt-danger }

![](searxng-logo.png){: width="150" .left}  
[cwwk KVM - SearXNG Alpine Linux (alpine-searx)](/posts/KVM-Alpine-Linux-Docker-SearXNG/)

Fichier de configuration nginx

    /etc/nginx/conf.d/searx.rnmkcy.eu.conf

```
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

### Audio Navidrome (audio.rnmkcy.eu)

[cwwk KVM Alpine Linux Navidrome (vm-alpine01)](/posts/Machine_virtuelle_vm-alpine01/)  

Serveur Navidrome accessible ip 192.168.10.201 port 4533

Le fichier proxy nginx `/etc/nginx/conf.d/audio.rnmkcy.eu.conf` 

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name  audio.rnmkcy.eu;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://audio.rnmkcy.eu$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  audio.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;

    # Proxy audio navidrome server
    location / {
      proxy_pass          http://192.168.10.210:4533;
    }

}
```

## Cockpit Admin gestion serveurs (cockpit.rnmkcy.eu)

*__Cockpit__ est une interface d’administration de serveur interactive. Il est facile à utiliser et très léger. Il interagit directement avec le système d’exploitation depuis une vraie session Linux dans un navigateur.*

[Cockpit, installation, ...](/posts/Cockpit_Web/)  
Résumé des instructions

```bash
sudo apt update && sudo apt install cockpit cockpit-machines
sudo usermod -aG sudo $USER
```

Proxy cat `/etc/nginx/conf.d/cockpit.rnmkcy.eu.conf` 

```
server {
    listen 80;
    listen [::]:80;
    server_name  cockpit.rnmkcy.eu;
    return 301 https://cockpit.rnmkcy.eu$request_uri;
}

server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name  cockpit.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;
    include snippets/authelia-location.conf; # Authelia auth endpoint

    location / {
        # Required to proxy the connection to Cockpit
        proxy_pass https://127.0.0.1:9090;
        include snippets/authelia-authrequest.conf; # Protect this endpoint
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Required for web sockets to function
        proxy_http_version 1.1;
        proxy_buffering off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Pass ETag header from Cockpit to clients.
        # See: https://github.com/cockpit-project/cockpit/issues/5239
        gzip off;
    }

}
```

### Gestion cockpit des machines externes via SSH

Pour gérer les machines externes, il faut générer des clés SSH car la communication cockpit se fait via SSH

Liste de machines externes

* PC1 Bureau EndeavourOS
* e6230 DELL EndeavourOS
* xoyaz.xyz Debian
* xoyize.xyz Debian
* iceyan.xyz debian

Se rendre dans le dossier $HOME de la machine cwwk

Générer les clés privées/publiques SSH 

```bash
ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/pc1-ed25519
chmod 600 ~/.ssh/pc1-ed25519
ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/e6230-ed25519
chmod 600 ~/.ssh/e6230-ed25519
ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/xoyaz.xyz-ed25519
chmod 600 ~/.ssh/xoyaz.xyz-ed25519
ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/xoyize.xyz-ed25519
chmod 600 ~/.ssh/xoyize.xyz-ed25519
ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/iceyan.xyz-ed25519
chmod 600 ~/.ssh/iceyan.xyz-ed25519
```

La clé publique (.pub) sera ajoutée au fichier **authorized_keys** de la machine correspondante

La connexion SSH est du type

```
ssh Utilisateur@<Adresse IP ou Domaine> -p N°_Port -i Clef
# Par défaut le chemin pour "Clef" est $HOME/.ssh/
```

Le fichier `~/.ssh.ssh_servers`

```
Alias;Utilisateur;Adresse IP ou Domaine;Port;Clef;Opt;Chemin Local;Chemin Distant
iceyan;iceyan;185.112.146.46;55046;iceyan.xyz-ed25519;;;Y;;4
xoyaz;yani;37.60.230.30;55030;xoyaz.xyz-ed25519;;;Y;;4
xoyize;xouser;158.220.91.148;55148;xoyize.xyz-ed25519;;;Y;;4
pc1;yann;192.168.0.20;22;pc1-ed25519;;;Y;;4
e6230w;yano;192.168.10.90;56230;e6230-ed25519;;;Y;;4
```

### .bashrc et alias

Dans le fichier `.bashrc`, on se réfère au fichier nommé `~/.bash_aliases` pour les alias

`~/.bash_aliases`

```
alias borglist='$HOME/scripts/borglist.sh'
alias etat='$HOME/scripts/etat_des_lieux_cwwk.sh > /sharenfs/rnmkcy/site/etat_cwwk.txt'
alias sshm='$HOME/scripts/ssh-manager.sh'
alias nmapl='echo "sudo nmap -T4 -sP 192.168.70.0/24" && sudo nmap -T4 -sP 192.168.70.0/24'
```

>Après ajout ou suppression alias, ne pas oublier la commande `source ~/.bash_aliases`
{: .prompt-info }

## Sauvegardes

### BorgBackup dépôt cwwk

**BorgBackup**

1. [Installation](/posts/BorgBackup_entre_serveurs/#installation)
* [Créer utilisateur borg](/posts/BorgBackup_entre_serveurs/#créer-utilisateur-borg)  
`useradd -M borg`  
`echo "borg ALL=NOPASSWD: /usr/bin/borg" >> /etc/sudoers.d/borg`
* [Clés ssh borg](/posts/BorgBackup_entre_serveurs/#clés-ssh-borg)
* [Ajout clé publique à la boite de stockage](/posts/BorgBackup_entre_serveurs/#ajout-clé-publique-à-la-boite-de-stockage)

Création dossier borg sous root

    sudo mkdir -p /root/.borg

Création dépôt dans la boîte de stockage (repository)

Dépôt distant à travers SSH

    ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/cwwk

Ajout de la phrase forte dans un fichier au dossier `/root/.borg`

```bash
# ajout phrase
sudo cp /sharenfs/pc1/.borg/cwwk.passphrase /root/.borg/
```

Initialisation dépôt distant en mode su

```shell
export BORG_PASSPHRASE="$(cat /root/.borg/cwwk.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/cwwk
borg init --encryption=repokey $BORG_REPOSITORY
```

Le fichier d'exclusion `/root/.borg/cwwk.exclusions`

```
/proc
/sys
/dev
/media
/mnt
/cdrom
/tmp
/run
/var/tmp
/var/run
/sharenfs
lost+found
```

Créer un script de sauvegarde (notez l'usage de borg prune pour supprimer les archives trop anciennes)  

    sudo nano /root/.borg/borg-backup.sh 

```shell
export BORG_PASSPHRASE=`cat /root/.borg/cwwk.passphrase`
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/cwwk
borg create -v --progress --stats --exclude-from /root/.borg/cwwk.exclusions ${BORG_REPOSITORY}::${BACKUP_DATE} /
borg prune -v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 $BORG_REPOSITORY
```

Le rendre exécutable

    chmode +x /root/.borg/borg-backup.sh 

Exécution

    /root/.borg/borg-backup.sh

Automatiser la sauvegarde en utilisant systemd timer

Le service `/etc/systemd/system/autoborg.service`

```
[Unit]
Description=BorgBackup

[Service]
User=root
ExecStart=/usr/bin/bash /root/.borg/borg-backup.sh

[Install]
WantedBy=multi-user.target
```

Le timer `/etc/systemd/system/autoborg.timer`

```
[Unit]
Description=Exécution BorgBackup

[Timer]
Unit=autoborg.service
OnCalendar=*-*-* 02:50

[Install]
WantedBy=timers.target
```

Exécution tous les jours 

Activez/démarrez le timer, puis vérifiez qu'il est chargé et actif 

```bash
sudo systemctl enable autoborg.timer --now
systemctl status autoborg.timer
```

Vérifiez s'il apparaît dans la liste des minuteries :

    systemctl list-timers

```

NEXT                         LEFT          LAST                         PASSED     UNIT                         ACTIVATES                     
Mon 2024-12-16 02:50:00 CET 18h left            -                           -             autoborg.timer               autoborg.service
```

Lorsque vous voulez voir si les sauvegardes se sont déroulées correctement, vous pouvez consulter le journal le plus récent 

    systemctl status autoborg

Ou afficher tous les journaux avec :

    sudo journalctl -u autoborg

### Synchronisation des backup borg  

*Synchronisation des backup borg entre le serveur de stockage (storage box) et le disque USB partagé monté sur la freebox en utilisant systemd timer.*

Le script à exécuter en mode su `/sharenfs/scripts/synchro-backup.sh`

```shell
#!/bin/bash

# Révision août 2025

# Vérifier si utilisateur "root"
if [ $(id -u) != "0" ]; then
    echo "Erreur : Vous devez être root pour exécuter ce script"
    exit 1
fi

servers="pc1 media iceyan.xyz e6230 cwwk asus-xh51 xoyize.xyz"
echo "Synchronisation entre 'BOX Hetzner' et 'Freebox disque USB externe'"
echo "Serveurs et dossiers: $servers"
 hote_array=(${servers// / })
 for hote in "${hote_array[@]}"
 do
  echo "$hote --> /mnt/FreeUSB2To/sauvegardes/borgbackup/"
  rsync -az --delete --rsync-path='rsync' -e 'ssh -p 23 -i /root/.ssh/id_borg_ed25519' u326239@u326239.your-storagebox.de:backup/borg/$hote /mnt/FreeUSB2To/sauvegardes/borgbackup/
 done
```

Script exécutable

    chmod +x /sharenfs/scripts/synchro-backup.sh

Le service `/etc/systemd/system/storage-freedisk.service`

```
[Unit]
Description=Rsync storagebox vers FreeUSB2To

[Service]
Type=oneshot
ExecStart=/usr/bin/bash /sharenfs/scripts/synchro-backup.sh
```

Le timer /etc/systemd/system/storage-freedisk.timer

```
[Unit]
Description=Rsync storagebox vers FreeUSB2To

[Timer]
OnCalendar=*-*-* 04:25
Persistent=true

[Install]
WantedBy=timers.target
```

Exécution tous les jours à 4h25 du matin

Activez/démarrez le timer, puis vérifiez qu’il est chargé et actif

```bash
sudo systemctl daemon-reload
sudo systemctl enable storage-freedisk.timer --now
systemctl status storage-freedisk.timer
```

Vérifiez qu’il a été démarré en vérifiant s’il apparaît dans la liste des minuteries :

    systemctl list-timers

```
NEXT                        LEFT          LAST                        PASSED       UNIT                         ACTIVATES                     
Mon 2024-12-16 04:25:00 CET 20h left      -                           -            storage-freedisk.timer       storage-freedisk.service
```

## Domaine local home.arpa + Certificats Internes

*Le domaine de deuxième niveau __home.arpa__ a été réservé pour une utilisation dans les réseaux résidentiels [(RFC 8375)](https://tools.ietf.org/html/rfc8375), bon choix pour un nom de domaine lors de la mise en place d'un réseau interne.*  
[Domaine home.arpa](/posts/home.arpa/)

### Gérer un domaine local avec Unbound

*[Gérer un domaine local avec Unbound](https://www.shaftinc.fr/unbound-domaine-local/), créer un domaine local, pour le réseau privé, ce qui est bien plus pratique que de retenir des IPs ou maintenir un fichier hosts sur les machines où l'on y a accès*

Configuration des accès réseaux locaux dans le fichier `/etc/unbound/unbound.conf.d/cwwk-unbound.conf`

```
    interface: 192.168.0.205

    port: 53

    # refuser tout le monde sauf les connexions locales (pas forcément
    # nécessaire vu que le serveur n'écoute que sur la boucle locale en IPv4)
    access-control: 0.0.0.0/0 refuse
    access-control: 127.0.0.1/32 allow
    # Adresse privée 
    access-control: 192.168.0.0/16 allow
    access-control: 192.168.10.0/16 allow
```

**Domaine cwwk.home.arpa**  
Vérifier et/ou Ajouter ce qui suit dans le fichier `/etc/unbound/unbound.conf.d/cwwk-unbound.conf`

```

    #
    local-zone: "home.arpa." transparent
    # IPv4
    local-data: "cwwk.home.arpa.  86400 IN A 192.168.0.205"
    local-data: "alp01.home.arpa. 86400 IN A 192.168.10.210"
    # IPv6
    local-data: "cwwk.home.arpa.  86400 IN AAAA 2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45"
    # On ajoute les enregistrements PTR
    # PTR IPv4
    local-data-ptr: "192.168.0.205  86400 cwwk.home.arpa."
    local-data-ptr: "192.168.10.210 86400 alp01.home.arpa."
    # PTR IPv6
    local-data-ptr: "2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45  86400 cwwk.home.arpa."
```

Vérifier la configuration

    sudo unbound-checkconf /etc/unbound/unbound.conf.d/cwwk-unbound.conf

Une fois tout configuré et Unbound relancé

    systemctl restart unbound

on peut faire quelques tests

```shell
dig cwwk.home.arpa
```

![](cwwk.home.arpa.png)

```shell
dig -x 192.168.0.205
```

![](192.168.0.205.png)

### Autorité de Certification interne et Certificats Clients

[Openssl - Création d'une Autorité de Certification interne et de Certificats Clients](/posts/Openssl_Autorite_de_Certification_interne+Certificats_Clients/)

## Nebula + Serveur NFS

### Nebula

* [Nebula est un outil pour interconnecter de manière transparente des ordinateurs](/posts/Nebula/)
    * [Serveur cwwk](/posts/Nebula/#serveur-cwwk)

IP Privée: 10.19.55.4

### Serveur NFS

**Parefeu ufw**  
Exécutez la commande ufw suivante pour permettre à votre sous-réseau de réseau local d’accéder au serveur NFS. La commande suivante permettra au réseau 10.19.55.0/24 d’accéder au serveur NFS.

```shell
sudo ufw allow from 10.19.55.0/24 to any port nfs
sudo ufw reload
```

La règle ajoutée

```
2049                       ALLOW       10.19.55.0/24             
```

Ouvrir le fichier `/etc/exports` à l’aide de la commande d’éditeur nano suivante.

    sudo nano /etc/exports

Ajoutez la ligne suivante au fichier.

```bash
/nebunfs    10.19.55.0/255.255.255.0(rw,no_root_squash,no_subtree_check)
```

Redémarrer serveur NFS

```bash
sudo systemctl restart nfs-server 
sudo systemctl status nfs-server
```

Enfin, exécutez la commande suivante pour vérifier le répertoire et le système de fichiers exportés sur le serveur NFS à l’aide de la commande suivante.

```bash
sudo showmount -e
```

Réponse

```
Export list for cwwk:
/sharenfs 10.19.55.0/255.255.255.0,192.168.0.0/255.255.255.0
```

>Pour autoriser les accès au serveu NFS sur le réseau 10.19.55.0/24, il faut ouvrir les ports entrants du firewall nebula 
{: .prompt-tip }

Dans le fichier `/etc/nebula/config.yaml`, dans la zone `firewall:` , `inbound:` , remplacer `proto: icmp` par `proto: any` et relancer le service nebula; `sudo systemctl restart nebula`

## Maintenance

### Connexion BOX hetzner

Crér un jeu de clés

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/cwwk_bx11_plakar

Ajout clé publique: `cat ~/.ssh/cwwk_bx11_plakar.pub` au fichier `/home/.ssh/authorized_keys` de la boite BX1

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBqtUtZFA8uOHZve309r1pjGtKnkBB5hQfUNH9rTm88P yick@cwwk
```

Tester la connexion pour valider la clé

```shell
sftp -P 23 -i ~/.ssh/cwwk_bx11_plakar u326239@u326239.your-storagebox.de
```

### Mises à jour automatiques sur debian

[Mises à jour automatiques sur Debian](/posts/Debian_unattended-upgrades_mise_a_jour_auto/)

### Disque externe USB3 NVME 2To

Connecter le disque externe USB3 NVME sur un port de la machine cwwk

Relever le mappage des disques : `lsblk` 

```
NAME                   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                      8:0    0   1,8T  0 disk 
```

Le disque NVME de 1.8T est sur `/dev/sda`

Zapper le disque,

(**Attention** Ceci effacera de manière irréversible toutes les données de votre disque, veuillez sauvegarder toutes les données importantes) :

```shell
sgdisk --zap-all /dev/sda
```

On partitionne un disque en 2 avec `gdisk`

* Partition 1 : 1024M EFI (code ef00) système de fichier FAT32
* Partition restante Linux (code 8300) système de fichier EXT4

Créer une table de partition GPT à l'aide de la commande `sgdisk` :

```
sgdisk --clear --new=1:0:+1024MiB --typecode=1:ef00 --new=2:0:0 --typecode=2:8300 /dev/sda
```

Format des partitions 

```
# EFI FAT32
mkfs.fat -F32 /dev/sda1 
# EXT4
mkfs.ext4 /dev/sda2
```

**Montage auto disque externe USB3 NVME**

[How to Automount USB Devices](https://www.baeldung.com/linux/automount-usb-device)

Disque USB NVME connecté sur un port USB3  

Identifié le disque: `lsblk`

```
NAME                   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                      8:0    0   1,8T  0 disk 
├─sda1                   8:1    0     1G  0 part 
└─sda2                   8:2    0   1,8T  0 part 
```

Dans notre cas `/dev/sda2`  
Son UUID: `sudo blkid |grep '/dev/sda2'`

```
/dev/sda2: UUID="d58a4656-bb6c-4750-8fbb-e5e356c353a0" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="3d0c8a43-270a-466e-8156-f93fb796b7ad"
```

Créer un point de montage

```shell
sudo mkdir -p /mnt/usbnvme  
```

Montage `/etc/fstab`

```
# USB3 NVME 2To
UUID=d58a4656-bb6c-4750-8fbb-e5e356c353a0 /mnt/usbnvme auto defaults,nofail,x-systemd.automount 0 2
```

Recharger la configuration et le montage

```shell
sudo systemctl daemon-reload 
sudo mount -a
```

### Disque externe USB3 SSD 1To

Connecter le disque externe SSD 1To sur un port USB3 de la machine cwwk

Relever le mappage des disques : `lsblk` 

```
NAME                   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                      8:0    0 953,9G  0 disk 
├─sda1                   8:1    0     1G  0 part 
└─sda2                   8:2    0   700G  0 part 
```

Le disque SSD de 1To est sur `/dev/sda`

Zapper le disque,

(**Attention** Ceci effacera de manière irréversible toutes les données de votre disque, veuillez sauvegarder toutes les données importantes) :

```shell
sgdisk --zap-all /dev/sda
```

On partitionne le disque avec `gdisk`

* Partition entière Linux (code 8300) système de fichier EXT4

Créer une table de partition GPT à l'aide de la commande `sgdisk` :

```
sgdisk --clear --new=1:0:0 --typecode=1:8300 /dev/sda
```

Format partition  

```
# EXT4
mkfs.ext4 /dev/sda1
```

**Montage auto disque externe USB3 SSD 1To**

[How to Automount USB Devices](https://www.baeldung.com/linux/automount-usb-device)

Identifié le disque: `lsblk`

```
NAME                   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                      8:0    0 953,9G  0 disk 
└─sda1                   8:1    0 953,9G  0 part 
sdb                      8:16   0   1,8T  0 disk 
├─sdb1                   8:17   0     1G  0 part 
└─sdb2                   8:18   0   1,8T  0 part 
```

Dans notre cas `/dev/sda1`  
Son UUID: `sudo blkid |grep '/dev/sda1'`

```
/dev/sda1: UUID="ad820226-db7c-48f1-ba78-12a5d7c8f553" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="9e9841bb-6372-46d0-a065-1514dfc2a73c"
```

Créer un point de montage

```shell
sudo mkdir -p /mnt/ssd  
```

Montage `/etc/fstab`

```
# USB3 SSD 1To
UUID=ad820226-db7c-48f1-ba78-12a5d7c8f553 /mnt/ssd auto defaults,nofail,x-systemd.automount 0 2
```

Recharger la configuration et le montage

```shell
sudo systemctl daemon-reload 
sudo mount -a
```

Supprimer lost+found: `sudo rm -r /mnt/ssd/lost+found/`

