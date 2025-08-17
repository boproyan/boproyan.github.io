+++
title = 'Accès wifi routé vers internet (Access Point) pour analyser les flux http, https et tout autre trafic'
date = 2020-09-04 00:00:00 +0100
categories = ['cubieboard']
+++
## Capturer le trafic avec mitmproxy

*Mettre en place un point d’accès wifi routé vers internet pour analyser les flux http et https (ou tout autre trafic) d'un équipement*    
Document original [Capturer le trafic avec mitmproxy](https://toutetrien.lithio.fr/article/capturer-le-trafic-avec-mitmproxy)

### Prérequis

Il nous faut donc au minimum pour analyser le trafic

* Une monocarte de type RPI , Olimex ou CubieBoard avec debian buster accessible via ssh
* Une clé wifi compatible "point d'accès"
* un point d’entrée des données
* analyser/enregistrer les données
* un point de sortie des données vers internet. 

>Tout est transparent, la circulation des données est bidirectionnelle.

## CubieBoard2

Accès ssh

    ssh ssh cub@192.168.0.47 -p 55035 -i /home/yannick/.ssh/cubie-ed25519

**Modification nom des interfaces**  
Sur une installation **debian** , <u>il est impossible de définir plusieurs points d'accès avec hostapd</u> ,car les interfaces réseau ont un nom complexe au lieu des traditionnels **eth0** et **wlan0**   

**boot.scr** est créé par un fichier script de démarrage. Si vous en avez un, vous pouvez utiliser mkimage pour créer votre fichier.  
Si vous voulez baser votre script de démarrage sur un **boot.scr** et que vous n'avez pas la source d'origine, il est possible de créer le fichier source **boot.txt**  

	sudo dd bs=1 skip=72 if=/boot/boot.scr of=boot.txt

Sauvegarde de l'original  

	sudo cp /boot/boot.scr /boot/boot.scr.bak

Modification du fichier source de boot **boot.txt**,ajout commande kernel (net.ifnames=0 biosdevname=0) pour ne pas renommer les interfaces eth et wlan  

	setenv bootargs  ${bootargs} quiet net.ifnames=0 biosdevname=0

Regénérer le fichier **boot.scr**

	sudo mkimage -A arm -T script -C none -n "Interfaces wlan" -d boot.txt  /boot/boot.scr

```
Image Name:   Interfaces wlan                                                                                                 
Created:      Fri Sep  4 12:09:46 2020                                                                                        
Image Type:   ARM Linux Script (uncompressed)                                                                                 
Data Size:    2593 Bytes = 2.53 KiB = 0.00 MiB                                                                                
Load Address: 00000000                                                                                                        
Entry Point:  00000000                                                                                                        
Contents:                                                                                                                     
   Image 0: 2585 Bytes = 2.52 KiB = 0.00 MiB                                                                                  
```

Redémarrer la machine et vérifier

    ip link

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 02:c4:04:40:f0:ff brd ff:ff:ff:ff:ff:ff
3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 7c:dd:90:5f:68:7b brd ff:ff:ff:ff:ff:ff
```

Connecter la clé wifi

    lsusb

`Bus 004 Device 002: ID 148f:5370 Ralink Technology, Corp. RT5370 Wireless Adapter`

La clé est un modèle "realtek" , installation des pilotes et des outils réseau et wifi  
Installer les pilotes Realtek, les outils  wifi dns et réseau  

    sudo apt install firmware-realtek wireless-tools iw wpasupplicant dnsutils net-tools 

zUn redémarrage est nécessaire : `sudo systemctl reboot`

En mode su

    sudo -s

La compatibilité "ap" de la clé wifi

    iw list |grep -i "ap"

```
		 * AP
		 * AP/VLAN
		Capabilities: 0x17e
		 * start_ap
		 * set_noack_map
		 * set_qos_map
		 * AP: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
		 * AP/VLAN: 0x00 0x10 0x20 0x30 0x40 0x50 0x60 0x70 0x80 0x90 0xa0 0xb0 0xc0 0xd0 0xe0 0xf0
		 * AP: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
		 * AP/VLAN: 0x00 0x20 0x40 0xa0 0xb0 0xc0 0xd0
		 * AP/VLAN
		 * #{ AP, mesh point } <= 8,
	HT Capability overrides:
	Device supports AP scan.
	Driver supports full state transitions for AP/GO clients
```

Les interfaces

* Interface filaire : eth0 
* Interface wifi : wlan0

Ajout ip statique interface wifi manuellement `ip address add 192.168.0.100/255.255.255.0 dev wlx7cdd905f687b`  
Ajouter au fichier interfaces

    nano /etc/network/interfaces

```
auto wlan0
iface wlan0 inet static
  address 192.168.0.100
  netmask 255.255.255.0

```

## Point d'accès Wifi

hostapd, un programme conçu pour gérer des points d’accès (avec ou sans authentification).

    sudo apt install hostapd

**Configurer HostApd** , éditer ou créer le fichier **hostapd.conf**  

    sudo nano /etc/hostapd/hostapd.conf

```
interface=wlan0
driver=nl80211
ssid=wificub
channel=1

# WPA and WPA2 configuration
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=3
wpa_passphrase=MON_MOT_DE_PASSE
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

On précise plusieurs paramètres comme l’interface qui sert de point d’accès, le driver nl80211 (certaines cartes wifi peuvent nécessiter des drivers spécifiques), le SSID de notre point d’accès, les méthodes de sécurité et chiffrement de notre réseau.

On peut à présent lancer hostapd via systemctl ou directement via le binaire en lui passant le fichier de configuration.  
À partir de maintenant nous pouvons voir notre réseau et nous y connecter mais nous allons tout de même ajouter  un serveur DHCP et un DNS afin d’attribuer une adresse aux machines qui s’y connectent et leur donner un moyen de résoudre des noms de domaine.

## DHCP DNS

2 cas : dnsmasq ou isc-dhcp + Unbound

### Dnsmasq

nous allons utiliser **dnsmasq** qui va nous permettre d’attribuer nos adresses IP tout en redirigeant les requêtes DNS vers un résolveur de notre choix.   
Installer dnsmasq 

    apt install dnsmasq

le configurer via `/etc/dnsmasq.conf` afin de prendre en compte les fichiers **.conf** placés dans `/etc/dnsmasq.d/`

```
# Include all files in a directory which end in .conf
conf-dir=/etc/dnsmasq.d/,*.conf
```

On crée ensuite un fichier `/etc/dnsmasq.d/ap.conf` pour la configuration

```
interface=wlx7cdd905f687b
listen-address=192.168.0.100
bind-interfaces
no-resolv
domain-needed
bogus-priv
dhcp-range=192.168.0.110,192.168.0.150,255.255.255.0,1h
dhcp-option=3,192.168.0.100
dhcp-option=6,192.168.0.100
server=80.67.169.12
log-queries
log-dhcp
```

nous indiquons à dnsmasq d’écouter sur notre interface afin de fournir des adresses IP sur l’interval 192.168.0.110-192.168.0.150 pour une durée d’une heure.  
On lui fournit également les IP du routeur et du serveur DNS et on redirige les requêtes DNS vers un DNS extérieur.

Ajout de la ligne suivante dans `/etc/default/hostapd` :

    DAEMON_CONF="/etc/hostapd/hostapd.conf"


### isc-dhcp + unbound

[Configuration d'un point d'accès wifi sur Raspbian](https://xael.org/2015/raspberry_pi-AP_wifi.html)

* isc-dhcp-server : serveur DHCPD simple et robuste
* unbound : un validateur DNS qui évite d'utiliser les DNS menteurs fournis par mon fournisseur d'accès

Installation

    apt install isc-dhcp-server unbound

Configurer le DHCP  
Éditer le fichier `/etc/dhcp/dhcpd.conf` pour ajouter notre zone

```
# option definitions common to all supported networks...
option domain-name "rnmkcy.eu";
option domain-name-servers 192.168.0.100;

default-lease-time 600;
max-lease-time 7200;

# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
ddns-update-style none;

subnet 192.168.0.0 netmask 255.255.255.0 {
  range 192.168.0.110 192.168.0.150;
  option routers 192.168.0.100;
}

```

Vérification

    dhcpd -t -cf /etc/dhcp/dhcpd.conf

```
Internet Systems Consortium DHCP Server 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /etc/dhcp/dhcpd.conf
Database file: /var/lib/dhcp/dhcpd.leases
PID file: /var/run/dhcpd.pid
```

Maintenant il faut spécifier sur quelle interface réseau le dhcp doit écouter

    sudo nano /etc/default/isc-dhcp-server

```

# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
DHCPDv4_CONF=/etc/dhcp/dhcpd.conf
#DHCPDv6_CONF=/etc/dhcp/dhcpd6.conf

# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPDv4_PID=/var/run/dhcpd.pid
#DHCPDv6_PID=/var/run/dhcpd6.pid

# Additional options to start dhcpd with.
#       Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS="-4"

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACESv4="wlan0"
#INTERFACESv6=""
```

### Redirection du trafic

Créer des règles Iptables afin de rediriger le trafic entrant vers notre interface de sortie vers internet, tout en permettant l’interception du trafic entre les deux.  

Nous activons le postrouting sur notre interface vers internet :

    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

Nous faisons en sorte que les connexions soit possibles de l’interface AP jusqu’à l’interface Internet mais également dans l’autre sens pour les connexions déjà établies :

    iptables -A FORWARD -i eth0 -o wlx7cdd905f687b -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -A FORWARD -i wlx7cdd905f687b -o eth0 -j ACCEPT

Nous faisons un pré-routage des ports 80 et 443 de l’interface AP vers le port d’écoute de notre outil d’analyse :

    iptables -t nat -A PREROUTING -i wlx7cdd905f687b -p tcp --match multiport --dports 80,443 -j REDIRECT --to-port 8080

À partir de là tout devrait fonctionner, il nous reste à bien activer hostapd et dnsmasq :

    systemctl start hostapd
    systemctl start dnsmasq

###  Analyser le trafic

Nous allons à présent utiliser un outil pour analyser le trafic des appareils connectés sur notre point d’accès, utilisation de [mitmproxy](https://mitmproxy.org/) qui est un outil rapide à prendre en main et très complet.

Nous lançons donc mitmproxy en mode transparent sur le port 8080 :

    mitmproxy –mode transparent -p 8080

En connectant notre appareil on verra les flux http passer dans mitmproxy. Pour le https il faut installer le certificat de mitmproxy sur notre système en allant sur http://mitm.it.  

### Aller plus loin

Ce qui est vraiment intéressant dans l’analyse du trafic d’un téléphone sous android par exemple c’est le trafic des applications installées sur l’appareil.   
En modifiant nos règles de routage pour rediriger tout le trafic vers notre outil d’analyse (et plus seulement http et https) il est possible de regarder ce que font nos applications quand on les lance.  
Il faut ajouter le certificat (celui généré par mitmproxy) dans le magasin de certificats du téléphone, ce qui nécessite quelques manipulations (uniquement avec un téléphone rooté).

Android enregistre chaque certificat sous un nom contenant le hash de ce dernier suivi de .0, nous allons donc récupérer ce hash grâce à openssl :

    openssl x509 -subject_hash_old -in mitmproxy-ca-cert.pem | head -1

Et nous copions donc notre certificat sur notre téléphone (sur la carte SD par exemple) sous le nom c8750f0d.0 dans mon cas. Maintenant il nous faut ajouter ce fichier dans le bon dossier pour que notre système le prenne en compte.   
Ouvrir un shell root sur notre téléphone (via ADB ou directement sur l’appareil) et remonter notre système en read-write (il est en read-only par défaut) :

    su    
    mount -o remount,rw /system

Puis nous allons copier notre certificat dans le magasin du téléphone :

    cp /emplacement/du/certificat/c8750f0d.0 /system/etc/security/cacerts/

On lui donne les permissions :

    chmod 644 /system/etc/security/cacerts/c8750f0d.0

On remonte notre système en read-only :

    mount -o remount,ro /system

Il peut être nécessaire de redemarrer le téléphone afin que le certificat soit pris en compte.

### Conclusion

C’est fini, on peut maintenant tranquillement lancer nos applications et regarder chez qui elles vont converser. Cependant l’analyse du trafic des applications sur mobile reste un sujet très complexe car nombre d’applications sur notre téléphone risquent de générer du « bruit » durant notre analyse. Vous pouvez regarder le travail d'[Exodus Privacy](https://exodus-privacy.eu.org/fr/) sur le sujet.

