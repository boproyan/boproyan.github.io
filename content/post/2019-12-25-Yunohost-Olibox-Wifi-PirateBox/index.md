+++
title = 'Yunohost-Olibox-Wifi-PirateBox'
date = 2018-11-23 00:00:00 +0100
categories = []
+++
2017-05-02-Yunohost-Olibox-Wifi-PirateBox
===============================

---
layout: article
title:  Olibox Yunohost Wifi PirateBox 
toc: true
ref: (falcutatif)
date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23
tags: [yunohost]
lang: fr
description:  Olibox Yunohost Wifi PirateBox 
---

*Réalisation d'une "boîte" yunohost hotspot wifi + vpn + tor + pirate avec une carte olimex A20-OlinuxIno-Micro*

## Carte olimex A20-OlinuxIno-Micro

[Documentation olimex](https://www.olimex.com/Products/OLinuXino/A20/A20-OLinuXino-MICRO-4GB/resources/A20-OLinuXino-Micro.pdf)

![alt text](/images/A20-olinuxino-micro-top.png "Top view")

![alt text](/images/A20-olinuxino-micro-bottom.png "Bottom view")


## Installation Debian Jessie sur carte olimex A20-OLinuXino-MICRO

**Matériel**

* Carte olimex [A20-OLinuXino-MICRO ](https://www.olimex.com/Products/OLinuXino/A20/A20-OLinuXino-MICRO-4GB/)
* Bloc Alimentation 10V 1A
* Dongle Wifi/USB RT5370
* Carte micro SD 4GO
* SSD 512GO
* Batterie Li-ion 3.7v 5000mAh

**SDcard**

Insérer la SDcard dans un interface USB/SDcard puis connecter l'ensemble sur un ordinateur  
Identifier le périphérique  

    dmesg

```
[14778.547242]  sdd: sdd1
[14778.548898] sd 4:0:0:0: [sdd] Attached SCSI removable disk
```

Dans notre cas , périphérique **/dev/sdd**  
Création SDcard pour carte Olimex A20-OlinuxIno-Micro ([Lien sur les images SDcard](http://ftp.uk.debian.org/debian/dists/jessie/main/installer-armhf/current/images/netboot/SD-card-images/))

    sudo -s
	zcat firmware.A20-OLinuXino-MICRO.img.gz partition.img.gz > /dev/sdd
	sync

Hostname : olibox  
Debian Jessie : oli/xxxx root/xxxx

**Connexion liaison série**

Connecter à l'ordinateur via un câble USB-MicroUsb , un module USB/Serial (Tx,Rx,Gnd) et lancer la commande

    sudo minicom

Insérer la micro carte SD puis mise sous tension A20-OlinuxIno-Micro  

```
   x    LVM VG vg-ssd, LV home as ext4   
   x    LVM VG vg-ssd, LV root as ext4    
   x    LVM VG vg-ssd, LV swap as swap  
   x    partition #1 of MMC/SD card #1 (mmcblk0) as ext2 (boot)   
```

20 à 30 minutes pour l'installation de base...  

Passage en super utilisateur

	su

Les points de montage 

	mount

```
...
/dev/mapper/vg--ssd-root on / type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/mmcblk0p1 on /boot type ext2 (rw,relatime)
/dev/mapper/vg--ssd-home on /home type ext4 (rw,relatime,data=ordered)
...
```

**Mises à jour Debian**

	apt update && apt upgrade   # Facultatif si premier démarrage après installation

**Accès sudo sans mot de passe**   
Configuration simplifiée ,ajouter utilisateur "debian" et ses droits en fin de fichier en mode su

    apt install sudo
	echo "oli     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

Connecter le dongle Wifi RT5370

    lsusb
    Bus 001 Device 002: ID 148f:5370 Ralink Technology, Corp. RT5370 Wireless Adapter

Ajouter le dépot source "non-free" dans **/etc/apt/sources.list**

	echo "deb http://http.debian.net/debian/ jessie main contrib non-free" >> /etc/apt/sources.list
	apt update


**TimeZone Europe/Paris ** (facultatif car défini lors de l'installation)  
En mode console ou semi-graphique

	  dpkg-reconfigure tzdata

  Geographic area: 8  # Europe  
  Time zone: 37       # Paris  

```
Current default time zone: 'Europe/Paris'
Local time is now:      Sat Dec 17 13:21:55 CET 2016.
Universal Time is now:  Sat Dec 17 12:21:55 UTC 2016.
```

**Locales**

	  dpkg-reconfigure locales

```
Generating locales (this might take a while)...
  fr_FR.UTF-8... done
Generation complete.
```

**Connexion via SSH**


	ssh oli@192.168.0.43 # depuis un poste du réseau


## YunoHost

### Liens

* [Yunohost](https://yunohost.org/#/docs_fr)
* [La brique internet](https://labriqueinter.net/)
* [Yunohost : Installation d’une Brique Internet](https://yunohost.org/#/installation_brique_fr)
* [La brique internet faite maison](https://blog.matlink.fr/brique-internet-faite-maison/)
* [OpenVPN, IPv6 with ULA and NAT](https://freeaqingme.tweakblogs.net/blog/9237/openvpn-ipv6-with-ula-and-nat.html)


### Installation

Une fois que vous avez accès à votre serveur, directement ou par SSH, vous pouvez installer YunoHost avec le script d’installation.

Installez git

	sudo apt install git

Clonez le dépôt du script d’installation de YunoHost

	git clone https://github.com/YunoHost/install_script /tmp/install_script

Lancez le script d’installation

	cd /tmp/install_script && sudo ./install_yunohost

### Postinstall

Domaine principal : mulleryan.net   
Mot de passe administration : xxxxxxx  

Passage en su

    sudo -s

Utilisateur

    yunohost user create yanm

```
Mot de passe d'administration : 
Prénom : yan
Adresse courriel : yanm@mulleryan.net
Nom : muller
Mot de passe : 
Confirmez : mot de passe : 
Création du répertoire « /home/yanm ».
Succès ! La configuration de SSOwat a été générée
Succès ! L'utilisateur a été créé
fullname: yan muller
mail: yanm@mulleryan.net
username: yanm
```
### Applications la brique

#### Client OpenVpn

    yunohost app install https://github.com/labriqueinternet/vpnclient_ynh

```
Choisissez un domaine pour l'administration web : mulleryan.net
Choisissez un chemin pour l'administration web (default: /vpnadmin) : 
Attention : Synchronizing state for openvpn.service with sysvinit using update-rc.d...
Attention : Executing /usr/sbin/update-rc.d openvpn defaults
Attention : Executing /usr/sbin/update-rc.d openvpn disable
Attention : insserv: warning: current start runlevel(s) (empty) of script `openvpn' overrides LSB defaults (2 3 4 5).
Attention : insserv: warning: current stop runlevel(s) (0 1 2 3 4 5 6) of script `openvpn' overrides LSB defaults (0 1 6).
Attention : Synchronizing state for php5-fpm.service with sysvinit using update-rc.d...
Attention : Executing /usr/sbin/update-rc.d php5-fpm defaults
Attention : Executing /usr/sbin/update-rc.d php5-fpm enable
Attention : Created symlink from /etc/systemd/system/multi-user.target.wants/ynh-vpnclient.service to /etc/systemd/system/ynh-vpnclient.service.
Attention : Created symlink from /etc/systemd/system/default.target.wants/ynh-vpnclient-checker.service to /etc/systemd/system/ynh-vpnclient-checker.service.
Attention : Created symlink from /etc/systemd/system/timers.target.wants/ynh-vpnclient-checker.timer to /etc/systemd/system/ynh-vpnclient-checker.timer.
Attention : Job for ynh-vpnclient.service failed. See 'systemctl status ynh-vpnclient.service' and 'journalctl -xn' for details.
Attention : WARNING: VPN Client is not started because you need to define a server CA through the web admin
Attention : WARNING: VPN Client is not started because you need either a client certificate, either a username (or both)
Succès ! La configuration de SSOwat a été générée
Succès ! Installation terminée
```

#### HotSpot Wifi


    yunohost app install https://github.com/labriqueinternet/hotspot_ynh

```
Choisissez un domaine pour l'administration web : mulleryan.net
Choisissez un chemin pour l'administration web (default: /wifiadmin) : 
Choisissez un nom pour le wifi (SSID) (default: myNeutralNetwork) : YanHotSpot
Choisissez un mot de passe wifi (au minimum 8 caractères pour le WPA2) : 4dGB4q2dYm
Installer des firmwares non-libres (en plus des libres) pour la clé USB wifi (yes/no) (default: yes) : 
Attention : E: Impossible de trouver le paquet firmware-linux-nonfree
Attention : E: Impossible de trouver le paquet firmware-atheros
Attention : E: Impossible de trouver le paquet firmware-realtek
Attention : E: Impossible de trouver le paquet firmware-ralink
Attention : E: Impossible de trouver le paquet firmware-libertas
Attention : E: Impossible de trouver le paquet atmel-firmware
Attention : E: Impossible de trouver le paquet zd1211-firmware
Attention : Synchronizing state for hostapd.service with sysvinit using update-rc.d...
Attention : Executing /usr/sbin/update-rc.d hostapd defaults
Attention : Executing /usr/sbin/update-rc.d hostapd disable
Attention : insserv: warning: current start runlevel(s) (empty) of script `hostapd' overrides LSB defaults (2 3 4 5).
Attention : insserv: warning: current stop runlevel(s) (0 1 2 3 4 5 6) of script `hostapd' overrides LSB defaults (0 1 6).
Attention : Synchronizing state for php5-fpm.service with sysvinit using update-rc.d...
Attention : Executing /usr/sbin/update-rc.d php5-fpm defaults
Attention : Executing /usr/sbin/update-rc.d php5-fpm enable
Attention : Created symlink from /etc/systemd/system/multi-user.target.wants/ynh-hotspot.service to /etc/systemd/system/ynh-hotspot.service.
Succès ! La configuration de SSOwat a été générée
Succès ! Installation terminée

```

#### PirateBox Wifi

*Une PirateBox est un dispositif électronique souvent composé d'un routeur et d'un dispositif de stockage d'information, créant un réseau sans fil qui permet aux utilisateurs qui y sont connectés d'échanger des fichiers anonymement et de manière locale1. Par définition, ce dispositif qui est souvent portable, est déconnecté d'Internet.*

*Les PirateBox sont à l'origine destinées à échanger librement des données libres du domaine public ou sous licence libre. Les logiciels utilisés pour la mise en place d'une PirateBox sont majoritairement open source (source ouverte), voire libres.*([Source WikiPédia](https://fr.wikipedia.org/wiki/PirateBox))

     yunohost app install https://github.com/labriqueinternet/piratebox_ynh

```
Choisissez un domaine pour l'administration web : mulleryan.net
Choisissez un chemin pour l'administration web (default: /piratebox) : 
Choisissez un faux domaine pour la PirateBox (default: share.box) : pirate.box
Choisissez un nom pour la PirateBox (default: ShareBox) : PirateOlibox
Les utilisateurs peuvent-ils supprimer des fichiers ? (yes/no) (default: yes) : 
Les utilisateurs peuvent-ils renommer des fichiers ? (yes/no) (default: yes) : 
Activer le chat ? (yes/no) (default: yes) : 
Attention : Clonage dans '/var/www/piratebox'...
Attention : Synchronizing state for php5-fpm.service with sysvinit using update-rc.d...
Attention : Executing /usr/sbin/update-rc.d php5-fpm defaults
Attention : Executing /usr/sbin/update-rc.d php5-fpm enable
Attention : Created symlink from /etc/systemd/system/multi-user.target.wants/ynh-piratebox.service to /etc/systemd/system/ynh-piratebox.service.
Attention : Job for ynh-piratebox.service failed. See 'systemctl status ynh-piratebox.service' and 'journalctl -xn' for details.
Attention : WARNING: PirateBox is not started because you need to define an associated wifi hotspot through the web admin
Succès ! La configuration de SSOwat a été générée
Succès ! Installation terminée

```

Se connecter en utilisateur sur <https://mulleryan.net/yunohost/sso/> --> Wifi Hotspot --> Ajouter un point d'accès  
SSID : YanPirateBox  
Sécurisé : Off  
Sauvegarder et recharger  
*Configuration mise à jour et service rechargé avec succès*  

Se connecter en utilisateur sur <https://mulleryan.net/yunohost/sso/> --> PirateBox  
PirateBox activée : On  
Point d'accès associé : YanPirateBox  
On laisse les autres options définies auparavant  
Sauvegarder et recharger  
*Configuration mise à jour et service rechargé avec succès*  


### Passage en IPV6

On relève sur **OliBox** les adresses IPV6 locales eth0 et wlan0

    ip addr show

```
[...]
     #eth0
     inet6 fe80::c2:9ff:fe40:f22b/64 scope link

     #wlan0
     inet6 fe80::f:54ff:fe13:2400/64 scope link
[...]
```

#### Paramétrage freebox

Le lien <https://utux.fr/index.php?tag/ipv6>  
La première chose à faire consiste à se rendre dans l'interface de configuration de la freebox grâce à l'adresse <mafreebox.free.fr> qui fonctionne même si vous êtes en bridge.

Puis rendez-vous dans : **Paramètres** de la Freebox, onglet **Mode avancé**, **Configuration IPv6**.

IPV6 activé  
Adresse IPv6 lien local (freebox) :   fe80::224:d4ff:fea6:aa20  

Délégation de prefixe   
Attention si vous configurez un Next Hop pour le premier subnet, il ne sera plus annoncé par la Freebox sur votre réseau  
Laisser vide le premier Next Hop  
Préfixe  : 2a01:e34:ee6a:b270::/64  
Next Hop :   

**Next Hop eth0**  
Préfixe  : 2a01:e34:ee6a:b272::/64  
Next Hop : fe80::c2:9ff:fe40:f22b  



#### Paramétrage olibox (eth0)

Adresse ipv6 freebox ,lien local :  fe80::224:d4ff:fea6:aa20   

    sudo nano /etc/network/interfaces

Commenter *inet6 auto* et ajouter adresse ipv6 statique

```
[...]
#iface eth0 inet6 auto
iface eth0 inet6 static
  address 2a01:e34:ee6a:b272::1
  netmask 64
#  post-up ip -6 route add default via fe80::224:d4ff:fea6:aa20 dev eth0

```

### Redémarrer olibox

Avant reboot installer **libpam-systemd** (session SSH ne se termine pas correctement lors d’un “reboot” à distance) 

    sudo apt install libpam-systemd
	sudo reboot

### Vérification en mode terminal

Connexion ssh

    ssh oli@192.168.0.43

Vérifier les adresses

    ip addr show 

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 02:c2:09:40:f2:2b brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.43/24 brd 192.168.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 2a01:e34:ee6a:b272::1/64 scope global 
       valid_lft forever preferred_lft forever
    inet6 2a01:e34:ee6a:b270:c2:9ff:fe40:f22b/64 scope global mngtmpaddr dynamic 
       valid_lft 86364sec preferred_lft 86364sec
    inet6 fe80::c2:9ff:fe40:f22b/64 scope link 
       valid_lft forever preferred_lft forever
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 02:0f:54:13:24:00 brd ff:ff:ff:ff:ff:ff
    inet 10.0.242.1/24 scope global wlan0
       valid_lft forever preferred_lft forever
    inet6 2a01:e34:ee6a:b271::42/64 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::f:54ff:fe13:2400/64 scope link 
       valid_lft forever preferred_lft forever
4: hotspot1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 02:0f:54:13:24:01 brd ff:ff:ff:ff:ff:ff
    inet 10.214.73.1/24 scope global hotspot1
       valid_lft forever preferred_lft forever
    inet6 fe80::f:54ff:fe13:2401/64 scope link 
       valid_lft forever preferred_lft forever
```

#### Paramétrage DNS OVH mulleryan.net 

Fournisseur OVH ,modification DNS

On accède au domaine <u>uniquement par son adresse IPV6</u>  
**2a01:e34:ee6a:b272::1** correspond au préfixe délégé sur **eth0**  
Les informations **spf,dmarc et dkim** sont affichées dans la consultation du domaine (yunohost en mode administration).

```
$TTL 3600
@	IN SOA dns200.anycast.me. tech.ovh.net. (2017042900 86400 3600 3600000 300)
                       IN NS     dns200.anycast.me.
                       IN NS     ns200.anycast.me.
                       IN MX 10  mulleryan.net.
                       IN AAAA   2a01:e34:ee6a:b272::1
                   600 IN TXT    "v=spf1 a mx ip6:4d02:f25:cc7b:e272::1 -all"
*                      IN CNAME  mulleryan.net.
_dmarc                 IN TXT    "v=DMARC1; p=none"
mail._domainkey        IN TXT    ( "v=DKIM1; k=rsa; t=s; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDIOT+u7P+69Jkg3YenaP15TvCMzjP6tdrg/xMBShr/60gkzK090v3W4ssMpY2mUIXY35gfFWNa3muxpFQvzrHJqJc80vCJDTLE9YZZB2beiSZlpzDiHUFPKANuEj2svmAV763HKAAxWByWrkIlbU6AuQpYvJ4N9Rn5bZehaRgsSwIDAQAB" )
```

> Patienter 24H pour la propagation DNS

Vérification par ping

    ping -c5 mulleryan.net

```
PING mulleryan.net(2a01:e34:ee6a:b272::1 (2a01:e34:ee6a:b272::1)) 56 data bytes
64 bytes from 2a01:e34:ee6a:b272::1 (2a01:e34:ee6a:b272::1): icmp_seq=1 ttl=64 time=0.437 ms
64 bytes from 2a01:e34:ee6a:b272::1 (2a01:e34:ee6a:b272::1): icmp_seq=2 ttl=64 time=0.382 ms
64 bytes from 2a01:e34:ee6a:b272::1 (2a01:e34:ee6a:b272::1): icmp_seq=3 ttl=64 time=0.429 ms
64 bytes from 2a01:e34:ee6a:b272::1 (2a01:e34:ee6a:b272::1): icmp_seq=4 ttl=64 time=0.466 ms
64 bytes from 2a01:e34:ee6a:b272::1 (2a01:e34:ee6a:b272::1): icmp_seq=5 ttl=64 time=0.559 ms

--- mulleryan.net ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4044ms
rtt min/avg/max/mdev = 0.382/0.454/0.559/0.063 ms
```




### motd

[Debian motd](post_url 2017-02-01-motd-message-bienvenue-connexion-ligne-commande %})  
Ne mettre que les fichiers **00-header**,**11-sysinfo** et **99-footer**  
Supprimer ,si existant , **/etc/update-motd.d/10-sysinfo** et **/etc/update-motd.d/20-updates**  
Rendre exécutable les bash , `chmod +x *`  

### Redémarrer olibox

	sudo reboot

Vérification accès depuis un poste distant 

    ping6 -c5 mulleryan.net

### Certificats SSL

*La fonctionnalité principale du gestionnaire de certificat est de permettre l'installation de certificat Let's Encrypt facilement sur vos domaines. Vous pouvez l'utiliser depuis l'interface d'admin web (Certificat SSL sur la page d'info d'un domaine), ou avec la ligne de commande avec **yunohost domain cert-status**, **cert-install** et **cert-renew**.*  

>Letsencrypt ,accès **mulleryan.net** uniquement **IPV6**

```
# yunohost domain cert-install
Error: Certificate installation for mulleryan.net failed !
Exception: [Errno 22] No DNS 'A' record found for mulleryan.net. You need to make your domain name point to your machine to be able to install a Let's Encrypt certificate! (If you know what you are doing, use --no-checks to disable those checks.)

# yunohost domain cert-install --no-checks
Success! The SSOwat configuration has been generated
Success! Successfully installed Let's Encrypt certificate for domain mulleryan.net!
```

Accès par URL <https://mulleryan.net> à l'administration  
Créer un utilisateur mot de passe : yanm  xxxxxx



---


Installer les pilotes dongle wifi Ralink RT5370 ,Realtek r8188eu  et **Outils**  sudo ,curl ,tmux, lsof , iw et libpam-systemd (session SSH ne se termine pas correctement lors d’un “reboot” à distance)  

	apt update && apt -y install firmware-ralink firmware-realtek sudo curl tmux lsof iw libpam-systemd tree



#### Paramétrage freebox

Le lien <https://utux.fr/index.php?tag/ipv6>  
La première chose à faire consiste à se rendre dans l'interface de configuration de la freebox grâce à l'adresse <mafreebox.free.fr> qui fonctionne même si vous êtes en bridge.

Puis rendez-vous dans : **Paramètres** de la Freebox, onglet **Mode avancé**, **Configuration IPv6**.

IPV6 activé  
Adresse IPv6 lien local (freebox) :   fe80::224:d4ff:fea6:aa20  

Délégation de prefixe   
Attention si vous configurez un Next Hop pour le premier subnet, il ne sera plus annoncé par la Freebox sur votre réseau  
Laisser vide le premier Next Hop  
Préfixe  : 2a01:e34:ee6a:b270::/64  
Next Hop :   

**Next Hop wlan0**  
Préfixe  : 2a01:e34:ee6a:b271::/64  
Next Hop : fe80::f:54ff:fe13:2400  

**Next Hop eth0**  
Préfixe  : 2a01:e34:ee6a:b272::/64  
Next Hop : fe80::c2:9ff:fe40:f22b  

#### Paramétrage IPV6 YanHotSpot (wlan0)

Se connecter en utilisateur sur <https://mulleryan.net/yunohost/sso/> --> Wifi Hotspot  
Point d'accès 1 ( wifi YanHotSpot), onglet IPV6  
Préfixe délégué :2a01:e34:ee6a:b271::   
Sauvegarder et recharger  
*Configuration mise à jour et service rechargé avec succès*  

