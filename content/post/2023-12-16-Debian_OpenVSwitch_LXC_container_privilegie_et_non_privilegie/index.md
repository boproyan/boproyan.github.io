+++
title = 'Debian OpenVSwitch LXC container privilégié et non privilégié'
date = 2023-12-16 00:00:00 +0100
categories = ['virtuel']
+++
## OpenvSwitch

*Le projet Open vSwitch a publié sur son site web le code de son commutateur virtuel open source  pour Xen, XenServer, KVM et VirtualBox. Open vSwitch se présente comme un équivalent du commutateur distribué Cisco Nexus 1000V. Il permet notamment aux administrateurs réseaux de gérer de façon précise le trafic réseau au sein d'un cluster de machines virtuelles, mais aussi de définir des politiques avancées de gestion des ressources du réseau virtuel.*

* [Introduction à OpenVSwitch](https://bellefab.medium.com/introduction-%C3%A0-openvswitch-36091f13e4d6)
* [Tuto Virtualisation V 2022 : Debian 11, KVM et Openvswitch](https://blog.debugo.fr/tuto-virtualisation-2022-debian-11-kvm-openvswitch/)
* [Open vSwitch / Debian 12](https://infoloup.no-ip.org/openvswitch-debian12-creation/)

En mode su

### installer Openvswitch sur hyperviseur

    apt install openvswitch-switch

Ajout switch br0

    ovs-vsctl add-br br0
    ovs-vsctl show

```
e97a17cd-50a9-4d4a-83ca-65794ae8146a
    Bridge br0
        Port br0
            Interface br0
                type: internal
    ovs_version: "3.1.0"
```

### Connexion Hyperviseur sur switch br0

Pour « connecter » ce switch a notre hyperviseur (en gros, l’intercaler entre le noyau et le port réseau physique), nous allons modifier le fichier interfaces **/etc/network/interfaces** de l’hyperviseur

Avant modification

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback
# The primary network interface
# Remplacer allow-hotplug par auto pour les montages CIFS
auto eno1
iface eno1 inet static
 address 192.168.0.215
 netmask 255.255.255.0
 gateway 192.168.0.254
# IPv6 interface
iface eno1 inet6 static
 address 2a01:e0a:9c8:2081::1
 netmask 64
 gateway fe80::8e97:eaff:fe39:66d6
```

Nouveau fichier de configuration

```
auto lo
iface lo inet loopback

## Configuration Open vSwitch
# Activation de l'interface br0
auto br0
allow-ovs br0


# Configuration IP de l'interface br0
iface br0 inet static
 address 192.168.0.215
 netmask 255.255.255.0
 gateway 192.168.0.254
 ovs_type OVSBridge
 ovs_ports eno1

# Attachement du port/interface eno1 au bridge br0
allow-br0 eno1
iface eno1 inet manual
 ovs_bridge br0
 ovs_type OVSPort

```

Après redémarrage de la machine, on vérifie le réseau `ip a`

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master ovs-system state UP group default qlen 1000
    link/ether 00:23:24:c9:06:86 brd ff:ff:ff:ff:ff:ff
    altname enp0s31f6
    inet6 fe80::223:24ff:fec9:686/64 scope link 
       valid_lft forever preferred_lft forever
3: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 22:a6:90:7b:5e:46 brd ff:ff:ff:ff:ff:ff
4: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 00:23:24:c9:06:86 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.215/24 brd 192.168.0.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::223:24ff:fec9:686/64 scope link 
       valid_lft forever preferred_lft forever
```

## LXC

* [OVS – LXC / Debian 11 : 1/2](https://infoloup.no-ip.org/virtualbox-debian11-lxc-partie-1/)
* [OVS – LXC / Debian 11 : 2/2](https://infoloup.no-ip.org/virtualbox-debian11-lxc-partie-2/)

![](/images/ctn204.png){:width="500"}  
Commandes lxc courantes

### Installation de LXC

*Les conteneurs non privilégiés contrairement aux conteneurs privilégiés sont créés et accessibles sans avoir besoin d'être root ou un utilisateur du groupe sudo. Ceci est plus sécurisé car l'on ne peut pas devenir root sur l'hôte même si l'on réussit à sortir du conteneur.* 

Installez les paquets lxc et lxc-templates :

    sudo apt install lxc lxc-templates

Vérifiez ensuite l'activation du routage au sein de la VM 

    sudo sysctl net.ipv4.ip_forward

*net.ipv4.ip_forward = 1*  
La valeur retournée doit être égale à 1.

Vérifiez aussi la bonne activation des services LXC :

    sudo systemctl status lxc 
    sudo systemctl status lxc-net

Les statuts retournés doivent être : `active(exited)`.

LXC a besoin que **cgroup** soit monté pour fonctionner.  
Control groups (cgroup) est une fonctionnalité du noyau Linux pour limiter, compter et isoler l'utilisation des ressources (processeur, mémoire, disque, etc...).  
Vérifiez le montage automatique de cgroup version 2 

    mount | grep cgroup

Retour *cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot)*

vérifiez la nouvelle configuration réseau :

[switch@ovs:~$] ip address

Un nouveau bridge de nom lxcbr0 a fait son apparition :
![](/images/ctn202.png)  

Ce bridge fourni par défaut avec LXC sera non exploité au profit des bridges br0 et br2 d'OVS.

## LXC (Création d'un conteneur privilégié)

### Création d'un conteneur de nom cnt1

*Les configurations LXC seront stockées dans /etc/lxc qui contient pour l'instant un unique fichier default.conf*

Des modèles de configuration sont disponibles dans `/usr/share/doc/lxc/examples/`

La création de conteneurs peut être réalisée à l'aide de templates situés dans `/usr/share/lxc/templates/` ou en téléchargeant une distribution spécifique sur Internet.

C'est la méthode de téléchargement qui sera utilisée ici.

Créez le conteneur de nom ctn1  

```bash
sudo lxc-create -n ctn1 -t download -- -d debian -r bookworm -a amd64
# OU
sudo lxc-create -n ctn1 -t download -- --dist debian --release bookworm --arch amd64
```

Le résultat de la commande lxc-create  
![](/images/ctn205.png)

La distribution téléchargée a été mise en cache dans `/var/cache/lxc/download/` 

Vérifiez la création et le statut stoppé du conteneur :

    sudo lxc-ls -f

```
NAME STATE   AUTOSTART GROUPS IPV4 IPV6 UNPRIVILEGED 
ctn1 STOPPED 0         -      -    -    false        
```

et afficher sa configuration par défaut :

    sudo cat /var/lib/lxc/ctn1/config

```
# Distribution configuration
lxc.include = /usr/share/lxc/config/common.conf
lxc.arch = linux64

# Container specific configuration
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
lxc.rootfs.path = dir:/var/lib/lxc/ctn1/rootfs
lxc.uts.name = ctn1

# Network configuration
lxc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
```


### Configuration réseau conteneur

La partie réseau doit être traitée avant de démarrer ctn1.  

LXC configure de base une connexion virtuelle de type **veth** pour raccorder **ctn1** au bridge **lxcbr0**.

Le type **veth** permet de créer une interface réseau sur le bridge lxcbr0 et une autre sur ctn1.

La configuration de **ctn1** doit être modifiée pour raccorder celui-ci sur le bridge **br0** d'OVS.

    sudo nano /var/lib/lxc/ctn1/config

et modifiez la partie réseau comme suit :

```
# Network configuration
lxc.net.0.type = veth
#lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
lxc.net.0.script.up = /etc/lxc/ovsup-ctn1
lxc.net.0.script.down = /etc/lxc/ovsdown-ctn1
lxc.net.0.ipv4.address = 192.168.0.216/24
lxc.net.0.ipv4.gateway = 192.168.0.215
```

Créez ensuite le script ovsup-ctn1 

    sudo nano /etc/lxc/ovsup-ctn1

et entrez le contenu suivant 

```bash
#!/bin/bash

BRIDGE="br0"
ovs-vsctl --may-exist add-br $BRIDGE
ovs-vsctl --if-exists del-port $BRIDGE $5
ovs-vsctl --may-exist add-port $BRIDGE $5
```

Au démarrage de ctn1, le script attachera celui-ci à OVS.

Créez le script ovsdown-ctn1 

    sudo nano /etc/lxc/ovsdown-ctn1

et entrez le contenu suivant 


```bash
#!/bin/bash

BRIDGE="br0"
ovs-vsctl --if-exists del-port $BRIDGE $5
```

A l'arrêt de ctn1, le script détachera celui-ci d'OVS.

Rendez les 2 scripts exécutables :

    sudo chmod +x /etc/lxc/ovs*

La configuration réseau de ctn1 est maintenant prête.

### Démarrage du conteneur

Pour démarrer ctn1, lancez la Cde suivante 

    sudo lxc-start -n ctn1

et vérifiez ensuite le statut du conteneur :

    sudo lxc-ls -f

```
NAME STATE   AUTOSTART GROUPS IPV4                        IPV6                                 UNPRIVILEGED 
ctn1 RUNNING 0         -      192.168.0.216, 192.168.0.39 2a01:e0a:9c8:2080:c69:a7ff:fe41:ffa2 false    
```

Conteneur ctn1 démarré

Vérifiez aussi la création d'une interface réseau veth :

    ip address

![](/images/ctn206.png)  
Interface réseau veth...@if2 créée sur la VM ovs

Remarque : Depuis Debian 11.3, l'adresse IPv4 peut ne pas apparaître du fait de l'intervention du service systemd-networkd activé par défaut dans le conteneur.

Pour régler ce problème, procédez comme suit :

```bash
sudo lxc-attach -n ctn1

systemctl stop systemd-networkd
systemctl disable systemd-networkd
exit

sudo reboot
sudo lxc-start -n ctn1
sudo lxc-ls -f
```

L'adresse IPv4 192.168.0.216 doit cette fois apparaître.

### Exécution de commandes dans le conteneur

Le conteneur a été créé par défaut sans MDP root.

La commande lxc-attach permet d’exécuter des commandes dans un conteneur, ceci en tant que root.

    sudo lxc-attach -n ctn1

Un prompt `root@ctn1:/#` doit s'afficher.

Affichez ensuite la configuration réseau de ctn1 

    ip address

![](/images/ctn207.png)  
Le conteneur montre une interface réseau eth0@if6

Il faut, afin que ctn1 accède à Internet, lui préciser l'IP du serveur DNS à utiliser.

Pour cela, ajoutez la ligne  `nameserver 192.168.0.254` 

    echo "nameserver 192.168.0.254" > /etc/resolv.conf

Instructions pour utiliser l'éditeur de textes vim :  
Touche i (mode insertion) > Pour entrez nameserv... .  
Touche Escape > Pour quitter le mode insertion.  
Touches :wq suivies de Entrée > Pour sauvegarder.  

Pour tester l'accès à Internet, installez l'éditeur nano 

    apt install nano

Enfin, utilisez la commande exit pour quitter le conteneur.

Remarque : Depuis Debian 11.3, le nameserver peut être par la suite écrasé du fait de l'intervention du service systemd-resolved activé par défaut dans le conteneur.

Pour régler ce problème, procédez comme suit :

    sudo lxc-attach -n ctn1
    systemctl stop systemd-resolved
    systemctl disable systemd-resolved
    rm /etc/resolv.conf
    nano /etc/resolv.conf

Entrez ce contenu dans le nouveau fichier resolv.conf :

```
nameserver 192.168.0.254    # IP de votre Box Internet
```

Puis, quittez ctn1, rebootez ovs et redémarrez ctn1 

```
exit

sudo reboot
sudo lxc-start -n ctn1
sudo lxc-ls -f
```

Entrez ensuite dans ctn1 et vérifiez si le nameserver du fichier resolv.conf est correct.

### Démarrage automatique du conteneur

Il est possible de démarrer automatiquement ctn1 au boot de l'hôte LXC soit la VM ovs.

Pour cela, éditez la configuration du conteneur 

    sudo nano /var/lib/lxc/ctn1/config

et ajoutez ce contenu à la fin du fichier 

```
# Démarrage auto du conteneur au bout de 10s
lxc.start.auto = 1
lxc.start.delay = 10
```

Puis rebootez la VM ovs et contrôlez le statut de ctn1 

    sudo lxc-ls -f

Retour :
![](/images/ctn207.png)  
Démarrage automatique du conteneur ctn1

### Quelques commandes LXC utiles

lxc-stop -n ctn1 > Stoppe ctn1  
lxc-snapshot -n ctn1 > Crée un snapshot de ctn1  
lxc-destroy -s -n ctn1 > Détruit ctn1 et ses snapshots  
lxc-info -n ctn1 > Fournit des informations sur ctn1  
lxc-copy -n ctn1 > Crée un clone de ctn1  

Utilisez la commande man lxc pour en découvrir d'autres.

### Tests divers sur le réseau virtuel

Testez depuis l'intérieur de ctn1 les pings suivants 

```
[root@ctn1:/#] ping mappy.fr               # Internet
[root@ctn1:/#] ping 192.168.2.1         # VM srvsec
[root@ctn1:/#] ping 192.168.4.2         # VM srvdmz
[root@ctn1:/#] ping 192.168.3.4         # VM debian11-vm2
```

Tous doivent recevoir une réponse positive.

Pour finir, testez un ping depuis la VM srvsec sur ctn1  :

    ping -c3 192.168.0.126

## LXC (Création d'un conteneur non privilégié)

### utilisateur propriétaire

Utilisateur : leno

Affichez ensuite les UID et GID attribués à 

    sudo cat /etc/s*id | grep leno

Retour :

```
leno:100000:65536
leno:100000:65536
```

UID = Identifiant Utilisateur  
GID = Identifiant Groupe

Vérifiez que la valeur du paramètre suivant est à 1 

    sudo sysctl kernel.unprivileged_userns_clone

*kernel.unprivileged_userns_clone = 1*

Si OK, le système permettra aux utilisateurs normaux d'exécuter des conteneurs non privilégiés.

Autorisez leno à créer et connecter 1'interface réseau en créant le fichier lxc-usernet 

    sudo nano /etc/lxc/lxc-usernet

et en y entrant le contenu suivant 

    leno veth br0 1

l'utilisateur "leno" est autorisé à créer 1 périphérique de type veth et à les ajouter au pont appelé br0   
soit 1 interface veth sur le bridge br0 d'Open vSwitch.

### Création d'un conteneur de nom ctn2

Créez un fichier de configuration par défaut pour LXC :

    mkdir -p .config/lxc
    nano .config/lxc/default.conf

et entrez le contenu suivant :

```
lxc.idmap = u 0 100000 65536
lxc.idmap = g 0 100000 65536

lxc.net.0.type = veth
lxc.net.0.link = br0
lxc.net.0.flags = up

lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
```

Installez acl pour une gestion avancée des droits Linux :

    sudo apt install acl

et affectez ces permissions aux fichiers de leno :

```
sudo setfacl -m u:362144:x /home/leno
sudo setfacl -m u:362144:x /home/leno/.local
sudo setfacl -m u:362144:x /home/leno/.local/share
```

Puis créez le conteneur ctn2 

    lxc-create -n ctn2 -t download -- --dist debian --release bookworm --arch amd64

Le résultat de la commande lxc-create doit donner ceci 

```
Downloading the image index
Downloading the rootfs
Downloading the metadata
The image cache is now ready
Unpacking the rootfs

---
You just created a Debian bookworm amd64 (20231123_05:24) container.

To enable SSH, run: apt install openssh-server
No default root or user password are set by LXC.
```

La distribution téléchargée a été mise en cache dans

    ls /home/leno/.cache/lxc/download/

*debian*  
Vérifiez la création et le statut stoppé du conteneur :

    lxc-ls -f

```
NAME STATE   AUTOSTART GROUPS IPV4 IPV6 UNPRIVILEGED 
ctn2 STOPPED 0         -      -    -    true         
```

et afficher sa configuration par défaut 

    cat .local/share/lxc/ctn2/config

```
# Distribution configuration
lxc.include = /usr/share/lxc/config/common.conf
lxc.include = /usr/share/lxc/config/userns.conf
lxc.arch = linux64

# Container specific configuration
lxc.idmap = u 0 100000 65536
lxc.idmap = g 0 100000 65536
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
lxc.rootfs.path = dir:/home/leno/.local/share/lxc/ctn2/rootfs
lxc.uts.name = ctn2

# Network configuration
lxc.net.0.type = veth
lxc.net.0.link = br0
lxc.net.0.flags = up
```

### Démarrage du conteneur

Au préalable, entrez la commande suivante :

    sudo loginctl enable-linger leno

La commande `enable-linger` activera les processus persistants pour l'utilisateur leno.

Cela maintiendra **ctn2** démarré en cas de fermeture de session utilisateur **leno** et permettra le démarrage automatique du conteneur ctn2 au boot 

Vérifiez la prise en compte de la commande :

    sudo loginctl list-users

```
 UID USER LINGER
1000 leno yes
```

Pour ensuite démarrer ctn2, lancez la commande suivante :

```
lxc-unpriv-start -n ctn2
```

systemd-run --scope --quiet --user --property=Delegate=yes lxc-start -n ctn2

ou celle-ci pour disposer d'un fichier log de démarrage :

    l

Vérifiez enfin le statut du conteneur :

    lxc-ls -f

```
```

et la création d'une seconde interface veth :

    ip address

Capture - LXC : Interface réseau veth1001...@if2 créée sur la VM ovsLXC : Interface réseau veth1001...@if2 créée sur la VM ovs

Remarque : Depuis Debian 11.3, l'adresse IPv4 peut ne pas apparaître du fait de l'intervention du service systemd-networkd activé dans le conteneur.