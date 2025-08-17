+++
title = 'YunohostDebian Jessie ,nginx ,php ,VPN ,HotSpot ,Wifi ,PirateBox'
date = 2018-11-23 00:00:00 +0100
categories = []
+++
Yunohost/Debian Jessie ,nginx ,php ,VPN ,HotSpot ,Wifi ,PirateBox
---
layout: article
title:  Yunohost - Hotspot wifi + vpn + tor + pirate
toc: true
ref: (falcutatif)
date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23
tags: [olimex]
lang: fr
description:  "Yunohost/Debian Jessie ,nginx ,php ,VPN ,HotSpot ,Wifi ,PirateBox"
---

*Réalisation d'une "boîte" Yunohost - hotspot wifi + vpn + tor + pirate avec une carte olimex A20-OlinuxIno-Micro*

## Carte olimex A20-OlinuxIno-Micro

[Documentation olimex](https://www.olimex.com/Products/OLinuXino/A20/A20-OLinuXino-MICRO-4GB/resources/A20-OLinuXino-Micro.pdf)

![alt text](/images/A20-olinuxino-micro-top.png "Top view")

![alt text](/images/A20-olinuxino-micro-bottom.png "Bottom view")



## Installation Debian Jessie sur carte olimex A20-OLinuXino-MICRO

**Matériel**

* Carte olimex [A20-OLinuXino-MICRO ](https://www.olimex.com/Products/OLinuXino/A20/A20-OLinuXino-MICRO-4GB/)
* Bloc Alimentation 10V 1A
* Dongle Wifi/USB RT5370
* Carte micro SD min 4GO
* SSD 512GB
* Batterie Li-ion 3.7v 5000mAh

**SDcard**

SDcard créer avec les paquets debian armhf  
[Index of /debian/dists/jessie/main/installer-armhf/current/images/netboot/SD-card-images/](http://ftp.uk.debian.org/debian/dists/jessie/main/installer-armhf/current/images/netboot/SD-card-images/)

```
sudo -s
cd ~/media/dplus/iso/jessie
#Insérer le lecteur USB/SDcard
dmesg #relever le périphérique ,ici /dev/sdb
#Ecriture image
zcat firmware.A20-OLinuXino-MICRO.img.gz partition.img.gz > /dev/sdb
```

**Connexion liaison série**

Utilisation module USB/Série **/dev/ttyUSB0** et **minicom**   
Insertion carte SD et mise sous tension A20-OlinuxIno-Micro  
Installation :

* Europe/France
* Hostname : **olibox**
* Domaine : 
* Miroir : France , ftp.fr.debian.org
* Http Proxy : 
* Utilisateur : **oli**
* Partionnement
    * Disque ssd en LVM : stretch-vg ,root-lv ext4 15G /, home-lv ext4 20G /home, swap-lv 4G
    * SDcard /boot ext2 512M
* software to install : **SSH server** et **standard system utilities**

A la fin de l'installation,redémarrage(par le jack d'alimentation)  

Connexion utilisateur **oli** via liaison USB/Série et **minicom**  
Passage en super utilisateur  
`su`  
Les points de montage  
`mount`  

```
/dev/mapper/vg--ssd-lv--root on / type ext4 (rw,relatime,errors=remount-ro,data)
/dev/mmcblk0p1 on /boot type ext2 (rw,relatime)                                 
/dev/mapper/vg--ssd-lv--home on /home type ext4 (rw,relatime,data=ordered)      
```

Version linux et debian:  
Linux olibox 3.16.0-4-armmp-lpae #1 SMP Debian 3.16.43-2+deb8u5 (2017-09-19) armv7l GNU/Linux
8.9  

Relever adresse IP et Mac :  
`ip addr`  
192.168.0.43  

**Paraméter SSH**  
Pas de connexion root **PermitRootLogin no** dans fichier **/etc/ssh/sshd_config**  
Installer **libpam-systemd** (session SSH ne se termine pas correctement lors d’un “reboot” à distance) :  
`apt install libpam-systemd`  
Relancer le service ssh :  
`systemctl restart ssh`  

**Connexion via SSH**  
Se connecter depuis un poste du réseau :  
`ssh oli@192.168.0.43`  
Installer **sudo** et modifier **/etc/sudoers** pour accès sans mot de passe à l’utilisateur oli

    su
    apt install sudo
	echo "oli     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
	exit


Modification fichier **/etc/apt/sources.list** pour installer les "firmwares" Ralink et Realtek  
Ajouter **contrib non-free** après **jessie main** de chaque ligne  
Mise à jour des dépôts et installation des pilotes

    sudo apt update && sudo apt -y install firmware-ralink firmware-realtek

Les pilotes dongle wifi RT5370 sont installés (firmware-ralink)  
**Outils**  curl ,tmux, tree, iw, git   

	sudo apt -y install  curl tmux tree iw git

**locales**

	sudo dpkg-reconfigure locales

```
Generating locales (this might take a while)...
  fr_FR.UTF-8... done
Generation complete.
```


## Freebox IPV6

Configuration freebox pour délégation sur **olibox**  
On relève adresse ip et mac

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 1000
    link/ether 02:c2:09:40:f2:2b brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.43/24 brd 192.168.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 2a01:e34:ee6a:b270:c2:9ff:fe40:f22b/64 scope global mngtmpaddr dynamic 
       valid_lft 86124sec preferred_lft 86124sec
    inet6 fe80::c2:9ff:fe40:f22b/64 scope link 
       valid_lft forever preferred_lft forever
```

Ajout délégation de préfixe pour **olibox** (configuration IPV6 sur la freebox)   
Prefixe :  2a01:e34:ee6a:b272::/64  
Next Hop : fe80::c2:9ff:fe40:f22b  

## Installation yunohost

Connexion sur "shuttle" en su (par console USB/Série ou SSH)  
Cloner le dépôt du script d’installation de YunoHost  
`sudo -s`  
`git clone https://github.com/YunoHost/install_script /tmp/install_script`  
Lancer le script d’installation  
`cd /tmp/install_script`  
`./install_yunohost`  
Poursuivre la post-installation :  
Domaine : **oli.ovh**  
Mot de passe administration : xxxxxx  
Patienter !!!

```
Success !

Installation logs are located in /var/log/yunohost-installation.log
```

Créer un utilisateur  

	yunohost user create olione

```
Administration password: 
First name: oli
Last name: one
Email address: olione@oli.ovh
Password: 
Confirm password: 
Creating directory '/home/olione'.
Success! The SSOwat configuration has been generated
Success! The user has been created
fullname: oli one
mail: olione@oli.ovh
username: olione
```

Installer application "HotSpot" [La Brique Internet](https://github.com/labriqueinternet/hotspot_ynh)  

	yunohost app install https://github.com/labriqueinternet/hotspot_ynh

```
Choisissez un domaine pour l'administration web : oli.ovh
Choisissez un chemin pour l'administration web (default: /wifiadmin) : 
Choisissez un nom pour le wifi (SSID) (default: myNeutralNetwork) : YanHotSpot
Choisissez un mot de passe wifi (au minimum 8 caractères pour le WPA2) : 4dGB4q2dYm
Installer des firmwares non-libres (en plus des libres) pour la clé USB wifi (yes/no) (default: yes) : 
Attention : Synchronizing state for hostapd.service with sysvinit using update-rc.d...
Attention : Executing /usr/sbin/update-rc.d hostapd defaults
Attention : Executing /usr/sbin/update-rc.d hostapd disable
Attention : insserv: warning: current start runlevel(s) (empty) of script `hostapd' overrides LSB defaults (2 3 4 5).
Attention : insserv: warning: current stop runlevel(s) (0 1 2 3 4 5 6) of script `hostapd' overrides LSB defaults (0 1 6).
Attention : Synchronizing state for php5-fpm.service with sysvinit using update-rc.d...
Attention : Executing /usr/sbin/update-rc.d php5-fpm defaults
Attention : Executing /usr/sbin/update-rc.d php5-fpm enable
Attention : Created symlink from /etc/systemd/system/multi-user.target.wants/ynh-hotspot.service to /etc/systemd/system/ynh-hotspot.service.
Attention : WARNING: Wifi Hotspot is not started because no wifi device was found (please, check the web admin)
Succès ! La configuration de SSOwat a été générée
Succès ! Installation terminée
```

---




## Hotspot Wifi + DHCP

Installation d'un point d'accès Wifi avec dhcp pour fournir des adresses ip aux clients wifi  
les interfaces réseau

	ip link

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default
 qlen 1000
    link/ether 02:c2:09:40:f2:2b brd ff:ff:ff:ff:ff:ff
3: wlx7cdd905f687b: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen
 1000
    link/ether 7c:dd:90:5f:68:7b brd ff:ff:ff:ff:ff:ff
```

**HotSpot Wifi**  ([lien original "Hot-Spot Wifi"](http://elinux.org/RPI-Wireless-Hotspot))

Installer  **hostapd** et les outils wifi

	sudo apt install hostapd wireless-tools wpasupplicant



**Configurer HostApd** , éditer ou créer le fichier **hostapd.conf**

	sudo nano /etc/hostapd/hostapd.conf 

```
interface=wlx7cdd905f687b
driver=nl80211
ssid=HotSpotYan
hw_mode=g
channel=5
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=4dGB4q2dYm
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

Modifier le fichier **/etc/default/hostapd** 

	sudo nano /etc/default/hostapd

	DAEMON_CONF="/etc/hostapd/hostapd.conf"


**Dnsmasq Dhcp**  
Installation

    sudo apt install dnsmasq

Configuration **/etc/dnsmasq.conf**

```
no-resolv
interface=wlx7cdd905f687b
dhcp-range=10.10.10.3,10.10.10.20,12h
server=208.67.222.222
server=208.67.222.220
```

Relancer **dnsmasq**

    sudo service dnsmasq restart

**wlx7cdd905f687b**  
 Nous allons configurer la connexion **wlx7cdd905f687b** pour être statique

	sudo nano /etc/network/interfaces

Supprimer tout ce qui fait référence aux anciens paramètres de configuration de wlan0, nous allons les changer  
Ajouter en fin de fichier

```
allow-hotplug wlx7cdd905f687b  
iface wlx7cdd905f687b inet static  
    address 10.10.10.1
    netmask 255.255.255.0
```

Activer et définir une adresse statique wifi dans une autre plage privée d'adresse 10.10.10

	sudo ip link set dev wlx7cdd905f687b up
	        

Pour corriger l'erreur **RTNETLINK answers: Operation not possible due to RF-kill**

	sudo -s	
	apt install rfkill
	rfkill unblock all
	ip link set dev wlx7cdd905f687b up

On définit l'IP de **wlx7cdd905f687b** manuellement pour éviter de relancer le service

	sudo ip addr add 10.10.10.1/24 dev wlx7cdd905f687b


**Configuration NAT (Network Address Translation)** (les opérations suivantes s'exécutent en mode su)  
NAT est une technique qui permet à plusieurs périphériques d'utiliser une seule connexion à Internet.  
Linux prend en charge NAT en utilisant Netfilter (également connu sous le nom iptables) et est assez facile à configurer. 

	su # sudo -s

Tout d'abord, activez le redirection IP dans le noyau: 

```
sysctl net.ipv4.ip_forward=1 
#sysctl net.ipv6.conf.all.forwarding=1
```

Pour le configurer automatiquement au démarrage, modifiez le fichier **/etc/sysctl.conf** et décommenter les lignes suivantes:

```
# enable packet forwarding for IPv4
net.ipv4.ip_forward=1
# enable packet forwarding for IPv6
#net.ipv6.conf.all.forwarding=1
```

Les règles  
Activer la mascarade sur l'interface partageant Internet (eth0 ou wlan1)  

```
iptables -A POSTROUTING -t nat -o eth0 -j MASQUERADE -m comment --comment "Activer la mascarade sur l'interface eth0 partageant Internet"
iptables -A FORWARD --match state --state RELATED,ESTABLISHED --jump ACCEPT -m comment --comment "Accepter toutes les connexions établies et reliées entre elles"
iptables -A FORWARD -i wlx7cdd905f687b --destination 10.10.10.0/24 --match state --state NEW --jump ACCEPT -m comment --comment "Accepter les nouvelles connexions venant de l'interface wlx7 et ayant pour destination notre sous-réseau"
iptables -A INPUT -s 10.10.10.0/24 --jump ACCEPT -m comment --comment "Accepter les connexions entrantes venant de notre sous-réseau"
```

Les sauvegarder dans **/etc/iptables/rules.v4** et **/etc/iptables/rules.v6** pour une restauration automatique avec le paquet **iptables-persistent**

    mkdir -p /etc/iptables/
    iptables-save > /etc/iptables/rules.v4
    apt install iptables-persistent

**Le hotspot wifi a accès à internet**

## Multi SSID

Pour paramétrer le point d'accès Wifi avec plusieurs SSID ,il faut modifier l'adresse MAC du périphérique réseau wlan ([Multiple SSIDs with hostapd](http://wiki.stocksy.co.uk/wiki/Multiple_SSIDs_with_hostapd))  
L'adresse mac doit se terminer par un 0  
Eventuellement ,pour éviter tout conflit , l'adresse mac débutera par 02 ('locally administered')  
Relever l'adresse mac de l'interface wlan

	ip link show dev wlx7cdd905f687b
	HWaddr 7c:dd:90:5f:68:7b

On va la modifier en  **7c:dd:90:5f:68:70**

```
sudo ip link set down dev wlx7cdd905f687b
sudo ip link set dev wlx7cdd905f687b address 7c:dd:90:5f:68:70
sudo ip link set up dev wlx7cdd905f687b
```

Modifier la configuration **/etc/hostapd/hostapd.conf** en spécifiant l'adresse mac et en ajoutant les nouveaux SSID

```
interface=wlx7cdd905f687b
bssid=7c:dd:90:5f:68:70
driver=nl80211
ssid=YanHotSpot
hw_mode=g
channel=5
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=passphrase_YanHotSpot
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

bss=wlx0_0
ssid=YanPirate
wpa=2
wpa_passphrase=passphrase_YanPirate
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

```

Nous allons modifier **/etc/network/interfaces**  
Ajouter en fin de fichier

```
allow-hotplug wlx0_0  
iface wlx0_0 inet static  
    address 10.10.20.1
    netmask 255.255.255.0

pre-up ip link set dev wlx7cdd905f687b address 7c:dd:90:5f:68:70
```

Modifier configuration **/etc/dnsmasq.conf**

```
no-resolv
interface=wlx7cdd905f687b
dhcp-range=10.10.10.3,10.10.10.20,12h
interface=wlx0_0
dhcp-range=10.10.20.3,10.10.20.20,12h
server=208.67.222.222
server=208.67.222.220
```

Pour que les adresses statiques soient attribuées après chargement hostapd, modification fichier **/etc/init.d/hostapd**

```
[...]
case "$1" in
  start)
        log_daemon_msg "Starting $DESC" "$NAME"
        start-stop-daemon --start --oknodo --quiet --exec "$DAEMON_SBIN" \
                --pidfile "$PIDFILE" -- $DAEMON_OPTS >/dev/null && ip addr add 10.10.20.1/24 dev wlx0_0
        log_end_msg "$?"
        ;;
[...]
```

Redémarrer la "machine"

## Nginx PHP7 mariadb

[nginx compilé + php (procédures)](post_url 2017-06-29-Serveur-web-nginx-PHP7 %})  
Créer un dossier pour les configurations **nginx** :   
`sudo mkdir -p /etc/nginx/conf.d/olibox.d`  
Installer **MariaDb** :   
`sudo apt install mariadb-server`  
[réinitialiser le mot de passe root oublié pour MariaDB ou MySql serveur en 5 étapes](https://memo-linux.com/reinitialiser-le-mot-de-passe-root-oublie-pour-mariadb-ou-mysql-serveur-en-5-etapes/)  
`sudo mysqladmin -u root password 'votre-nouveau-mot-de-passe'`


## PirateBox

En mode su , se rendre sous /var/www et cloner [php-piratebox](https://github.com/jvaubourg/php-piratebox)

	cd /var/www
	git clone https://github.com/jvaubourg/php-piratebox.git
	mv php-piratebox piratebox # renommer le dossier


Le fichier de configuration **nginx**  

	nano /etc/nginx/conf.d/olibox.d/piratebox.conf

```
# Max file size
client_max_body_size 10G;

# OPTIONAL
location /public/uploads/ {

  # OPTIONAL: use a public/uploads/ folder located elsewhere
  # WITH: $options['base_uploads'] = '/var/spool/piratebox/public/uploads/'
  root /var/spool/piratebox/;

  # OPTIONAL: force download for all files
  add_header Content-Type "application/octet-stream";
  add_header Content-Disposition "attachment; filename=$1";
}

# OPTIONAL
location /public/chat/ {

  # OPTIONAL: use a public/chat/ folder located elsewhere
  # WITH: $options['base_chat'] = '/var/spool/piratebox/public/chat/'
  root /var/spool/piratebox/;

  # OPTIONAL: deny direct access to the chat log
  deny all;
  return 403;
}

# PHP
location ~ \.php {
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           fastcgi_pass unix:/run/php/php7.0-fpm.sock;    # PHP7.0 
           fastcgi_index index.php;
           include fastcgi_params;
	   fastcgi_param SCRIPT_FILENAME $request_filename;
  # 10 minutes max for uploading a file
  fastcgi_send_timeout 600;
}

# OPTIONAL: use fancy urls
# WITH: $options['fancyurls'] = true
location @piratebox {

  # WITH: $options['base_uri'] = '/foobar/'
  rewrite ^/foobar/(.*)$ /foobar/?/get&dir=$1;
}

location / {
  index index.html index.php;

  # OPTIONAL: use fancy urls
  try_files $uri $uri/ @piratebox;
}
```

Pour inclure la configuration

	nano /etc/nginx/conf.d/default.conf	

```
server {
    listen 80;
    listen [::]:80;
    server_name _;
    root /var/www/piratebox/;

    include conf.d/olibox.d/*.conf;	

    access_log /var/log/nginx/olibox-access.log;
    error_log /var/log/nginx/olibox-error.log;

}
```

Permissions

	chown www-data: /var/www/piratebox/public/uploads/
	chown www-data: /var/www/piratebox/public/chat/


Le fichier de configuration **/etc/php5/fpm/php.ini** ou **/etc/php/7.0/cli/php.ini** suivant version php  
*post_max_size* integer Définit la taille maximale des données reçues par la méthode POST. Cette option affecte également les fichiers chargés.  
Pour charger de gros fichiers, cette valeur doit être plus grande que la valeur de *upload_max_filesize*.  
De façon générale, *memory_limit* doit être plus grand que *post_max_size*. 

```
; Max file size , upload_max_filesize = 2M par défaut
upload_max_filesize = 50M
;post_max_size = 8M par défaut
post_max_size = 80M

; 5 minutes max for uploading a file , max_execution_time = 30 par défaut
max_execution_time = 300
```

## OpenVpn

On installe openvpn

	sudo apt install openvpn

On se base sur un fichier de configuration client généré lors de l'installation dur serveur openvpn  
IMPORTANT : fonctionnement ipv4 (proto udp) , ipv6 ? (proto udp6) non fonctionnel sur les clients android  
Le fichier **/etc/openvpn/client/protonvpn.conf**

```
# ==============================================================================
# Copyright (c) 2016-2017 ProtonVPN A.G. (Switzerland)
# Email: contact@protonvpn.com
#
# The MIT License (MIT)
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR # OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS
# IN THE SOFTWARE.
# ==============================================================================

client
dev tun
proto udp

remote 185.159.156.15 1194

remote-random
resolv-retry infinite
nobind
cipher AES-256-CBC
auth SHA512
comp-lzo
verb 3

tun-mtu 1500
tun-mtu-extra 32
mssfix 1450
persist-key
persist-tun

ping 15
ping-restart 0
ping-timer-rem
reneg-sec 0

remote-cert-tls server
auth-user-pass
pull
fast-io


<ca>
-----BEGIN CERTIFICATE-----
MIIFozCCA4ugAwIBAgIBATANBgkqhkiG9w0BAQ0FADBAMQswCQYDVQQGEwJDSDEV
MBMGA1UEChMMUHJvdG9uVlBOIEFHMRowGAYDVQQDExFQcm90b25WUE4gUm9vdCBD
QTAeFw0xNzAyMTUxNDM4MDBaFw0yNzAyMTUxNDM4MDBaMEAxCzAJBgNVBAYTAkNI
MRUwEwYDVQQKEwxQcm90b25WUE4gQUcxGjAYBgNVBAMTEVByb3RvblZQTiBSb290
IENBMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAt+BsSsZg7+AuqTq7
vDbPzfygtl9f8fLJqO4amsyOXlI7pquL5IsEZhpWyJIIvYybqS4s1/T7BbvHPLVE
wlrq8A5DBIXcfuXrBbKoYkmpICGc2u1KYVGOZ9A+PH9z4Tr6OXFfXRnsbZToie8t
2Xjv/dZDdUDAqeW89I/mXg3k5x08m2nfGCQDm4gCanN1r5MT7ge56z0MkY3FFGCO
qRwspIEUzu1ZqGSTkG1eQiOYIrdOF5cc7n2APyvBIcfvp/W3cpTOEmEBJ7/14RnX
nHo0fcx61Inx/6ZxzKkW8BMdGGQF3tF6u2M0FjVN0lLH9S0ul1TgoOS56yEJ34hr
JSRTqHuar3t/xdCbKFZjyXFZFNsXVvgJu34CNLrHHTGJj9jiUfFnxWQYMo9UNUd4
a3PPG1HnbG7LAjlvj5JlJ5aqO5gshdnqb9uIQeR2CdzcCJgklwRGCyDT1pm7eoiv
WV19YBd81vKulLzgPavu3kRRe83yl29It2hwQ9FMs5w6ZV/X6ciTKo3etkX9nBD9
ZzJPsGQsBUy7CzO1jK4W01+u3ItmQS+1s4xtcFxdFY8o/q1zoqBlxpe5MQIWN6Qa
lryiET74gMHE/S5WrPlsq/gehxsdgc6GDUXG4dk8vn6OUMa6wb5wRO3VXGEc67IY
m4mDFTYiPvLaFOxtndlUWuCruKcCAwEAAaOBpzCBpDAMBgNVHRMEBTADAQH/MB0G
A1UdDgQWBBSDkIaYhLVZTwyLNTetNB2qV0gkVDBoBgNVHSMEYTBfgBSDkIaYhLVZ
TwyLNTetNB2qV0gkVKFEpEIwQDELMAkGA1UEBhMCQ0gxFTATBgNVBAoTDFByb3Rv
blZQTiBBRzEaMBgGA1UEAxMRUHJvdG9uVlBOIFJvb3QgQ0GCAQEwCwYDVR0PBAQD
AgEGMA0GCSqGSIb3DQEBDQUAA4ICAQCYr7LpvnfZXBCxVIVc2ea1fjxQ6vkTj0zM
htFs3qfeXpMRf+g1NAh4vv1UIwLsczilMt87SjpJ25pZPyS3O+/VlI9ceZMvtGXd
MGfXhTDp//zRoL1cbzSHee9tQlmEm1tKFxB0wfWd/inGRjZxpJCTQh8oc7CTziHZ
ufS+Jkfpc4Rasr31fl7mHhJahF1j/ka/OOWmFbiHBNjzmNWPQInJm+0ygFqij5qs
51OEvubR8yh5Mdq4TNuWhFuTxpqoJ87VKaSOx/Aefca44Etwcj4gHb7LThidw/ky
zysZiWjyrbfX/31RX7QanKiMk2RDtgZaWi/lMfsl5O+6E2lJ1vo4xv9pW8225B5X
eAeXHCfjV/vrrCFqeCprNF6a3Tn/LX6VNy3jbeC+167QagBOaoDA01XPOx7Odhsb
Gd7cJ5VkgyycZgLnT9zrChgwjx59JQosFEG1DsaAgHfpEl/N3YPJh68N7fwN41Cj
zsk39v6iZdfuet/sP7oiP5/gLmA/CIPNhdIYxaojbLjFPkftVjVPn49RqwqzJJPR
N8BOyb94yhQ7KO4F3IcLT/y/dsWitY0ZH4lCnAVV/v2YjWAWS3OWyC8BFx/Jmc3W
DK/yPwECUcPgHIeXiRjHnJt0Zcm23O2Q3RphpU+1SO3XixsXpOVOYP6rJIXW9bMZ
A1gTTlpi7A==
-----END CERTIFICATE-----
</ca>

key-direction 1
<tls-auth>
# 2048 bit OpenVPN static key
-----BEGIN OpenVPN Static key V1-----
6acef03f62675b4b1bbd03e53b187727
423cea742242106cb2916a8a4c829756
3d22c7e5cef430b1103c6f66eb1fc5b3
75a672f158e2e2e936c3faa48b035a6d
e17beaac23b5f03b10b868d53d03521d
8ba115059da777a60cbfd7b2c9c57472
78a15b8f6e68a3ef7fd583ec9f398c8b
d4735dab40cbd1e3c62a822e97489186
c30a0b48c7c38ea32ceb056d3fa5a710
e10ccc7a0ddb363b08c3d2777a3395e1
0c0b6080f56309192ab5aacd4b45f55d
a61fc77af39bd81a19218a79762c3386
2df55785075f37d8c71dc8a42097ee43
344739a0dd48d03025b0450cf1fb5e8c
aeb893d9a96d1f15519bb3c4dcb40ee3
16672ea16c012664f8a9f11255518deb
-----END OpenVPN Static key V1-----
</tls-auth>
```


Pour utiliser le fichier de conf **/etc/openvpn/client/protonvpn.conf** lors du démarrage du service d'OpenVPN.  

	systemctl start openvpn-client@protonvpn.service

On regarde quel interface de tunneling

	ip link |grep tun    # tun0  
	6: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN mode DEFAULT group default qlen 100

Activer pour le prochain redémarrage

	systemctl enable openvpn-client@protonvpn.service

Effacer les règles précédentes (il faut être connecté via USB/Serial et minicom)

```
  sudo iptables -t nat -F
  sudo iptables -F
  sudo iptables -X
```

Ecrire les règles iptables ipv4 :

```
iptables -A POSTROUTING -t nat -o eth0 -j MASQUERADE -m comment --comment "Activer la mascarade sur interface eth0 partageant Internet"
iptables -A FORWARD --match state --state RELATED,ESTABLISHED --jump ACCEPT -m comment --comment "Accepter toutes les connexions etablies et reliees entre elles"
iptables -A FORWARD -i wlx7cdd905f687b --destination 10.10.10.0/24 --match state --state NEW --jump ACCEPT -m comment --comment "Accepter les nouvelles connexions venant interface wlx7 et ayant pour destination notre sous-reseau"
iptables -A INPUT -s 10.10.10.0/24 --jump ACCEPT -m comment --comment "Accepter les connexions entrantes venant de notre sous-reseau"

iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE -m comment --comment "Activer la mascarade sur l'interface tun0"  
iptables -A FORWARD -s 10.10.10.0/24 -i wlx7cdd905f687b -o eth0 -m conntrack --ctstate NEW -j REJECT -m comment --comment "VPN : Bloquer le trafic des clients wlx7cdd905f687b vers eth0"  
iptables -A FORWARD -s 10.10.10.0/24 -i wlx7cdd905f687b -o tun0 -m conntrack --ctstate NEW -j ACCEPT -m comment --comment "VPN : Autoriser uniquement le trafic des clients wlx7cdd905f687b vers tun0" 

iptables -A FORWARD -s 10.10.20.0/24 -i wlx0_0 -o tun0 -m conntrack --ctstate NEW -j REJECT -m comment --comment "VPN : Bloquer le trafic des clients wlx0_0 vers tun0"  

```

Sauvegardez les pour le prochain redémarrage :

	iptables-save > /etc/iptables/rules.v4

Le contenu

```
# Generated by iptables-save v1.6.0 on Wed Jul 12 17:07:20 2017
*filter
:INPUT ACCEPT [1:66]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [1:69]
-A INPUT -s 10.10.10.0/24 -m comment --comment "Accepter les connexions entrantes venant de notre sous-rs
eau" -j ACCEPT
-A INPUT -s 10.10.20.0/24 -m comment --comment "Accepter les connexions entrantes venant de notre sous-r�
éseau" -j ACCEPT
-A FORWARD -m state --state RELATED,ESTABLISHED -m comment --comment "Accepter toutes les connexions �éta
blies et reli�ées entre elles" -j ACCEPT
-A FORWARD -d 10.10.10.0/24 -i wlx7cdd905f687b -m state --state NEW -m comment --comment "Accepter les no
uvelles connexions venant de l\'interface wlx7 et ayant pour destination notre sous-r�éseau" -j ACCEPT
-A FORWARD -d 10.10.20.0/24 -i wlx0_0 -m state --state NEW -m comment --comment "Accepter les nouvelles c
onnexions venant de l\'interface wlx0_0 et ayant pour destination notre sous-r�éseau" -j ACCEPT
-A FORWARD -s 10.10.20.0/24 -i wlx0_0 -o eth0 -m conntrack --ctstate NEW -m comment --comment "Bloquer le
 trafic des clients wlx0_0 vers eth0" -j REJECT --reject-with icmp-port-unreachable
-A FORWARD -s 10.10.20.0/24 -i wlx0_0 -o tun0 -m conntrack --ctstate NEW -m comment --comment "Autoriser 
uniquement le trafic des clients wlx0_0 vers tun0" -j ACCEPT
COMMIT
# Completed on Wed Jul 12 17:07:20 2017
# Generated by iptables-save v1.6.0 on Wed Jul 12 17:07:20 2017
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o eth0 -m comment --comment "Activer la mascarade sur l\'interface eth0 partageant Intern
et" -j MASQUERADE
-A POSTROUTING -o tun0 -m comment --comment "Activer la mascarade sur l\'interface tun0" -j MASQUERADE
COMMIT
# Completed on Wed Jul 12 17:07:20 2017
```

## Liens

* [La brique internet faite maison](https://blog.matlink.fr/brique-internet-faite-maison/)
* [OpenVPN, IPv6 with ULA and NAT](https://freeaqingme.tweakblogs.net/blog/9237/openvpn-ipv6-with-ula-and-nat.html)
* [Hostapd : The Linux Way to create Virtual Wifi Access Point](https://nims11.wordpress.com/2012/04/27/hostapd-the-linux-way-to-create-virtual-wifi-access-point/)


