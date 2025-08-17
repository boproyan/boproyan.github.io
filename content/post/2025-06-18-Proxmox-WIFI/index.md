+++
title = 'Proxmox - Configuration WIFI'
date = 2025-06-18
categories = ['debian']
+++
Note: Cette configuration n'utilise pas DHCP, UNIQUEMENT des adresses statiques.

## Proxmox VE - WIFI

### Préalables

1) Connexion Ethernet filaire - ceci est nécessaire pour installer **wpasupplicant**

2) Configurez votre routeur wifi vers des réseaux qui seront associés à l'adaptateur wifi. Exemple pour le réseau /24:

Destination = 192.168.3.0
Netmask = 255.255.255.0
Gateway = 192.168.10.240 (spécifier addresse IP adaptateur wifi)



### Installation

En mode su

1) Connectez le câble Ethernet.

2) Installer Proxmox 8-1.2.

3) Après l'installation complète et après redémarrage système, installer wpasupplicant 

```shell
apt update && apt install wpasupplicant wireless-tools
systemctl disable wpa_supplicant
```

Afficher les périphériques: `iwconfig`

```shell
lo        no wireless extensions.

eno1      no wireless extensions.

wlp1s0    IEEE 802.11  ESSID:off/any  
          Mode:Managed  Access Point: Not-Associated   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:on
          
vmbr0     no wireless extensions.
```

4) Configurer wpasupplicant 

Exécuter l'instruction suivante

```shell
wpa_passphrase SSIDNAME PASSWORD >> /etc/wpa_supplicant/wpa_supplicant.conf
```

5) Déterminer le nom de l'adaptateur sans fil:

```shell
dmesg | grep -i wlp
```

Renvoie

```
[    5.262610] iwlwifi 0000:01:00.0 wlp1s0: renamed from wlan0
```

6) Créer `/etc/systemd/system/wpa_supplicant.service` 

```shell
touch /etc/systemd/system/wpa_supplicant.service
```

ajouter la configuration:

```
[Unit]
Description=WPA supplicant
Before=network.target
After=dbus.service
Wants=network.target
IgnoreOnIsolate=true
 
[Service]
Type=dbus
BusName=fi.w1.wpa_supplicant1
ExecStart=/sbin/wpa_supplicant -u -s -c /etc/wpa_supplicant/wpa_supplicant.conf -i wlp1s0
Restart=always
 
[Install]
WantedBy=multi-user.target
Alias=dbus-fi.w1.wpa_supplicant1.service
```

7) Activer le service wpasupplicant:

```shell
systemctl daemon-reload
systemctl enable wpa_supplicant
```

8) Ajout de la configuration wifi au fichier `/etc/network/interfaces` 


```
auto lo
iface lo inet loopback

iface eno1 inet manual

auto wlp1s0
iface wlp1s0 inet manual
    address 192.168.10.240/24
    gateway 192.168.10.1

auto vmbr0
iface vmbr0 inet static
	address 192.168.0.215/24
	gateway 192.168.0.205
	bridge-ports eno1
	bridge-stp off
	bridge-fd 0

## uncomment these lines after completing step 13
#iface vnet1 inet static
#       address 192.168.3.1/24
#        bridge-ports none
#        bridge-stp off
#        bridge-fd 0
#        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
#        post-up iptables -t nat -A POSTROUTING -s '192.168.3.0/24' -o wlp1s0 -j MASQUERADE
#        post-up iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone zone1
#        post-down iptables -t nat -D POSTROUTING -s '192.168.3.0/24' -o wlp1s0 -j MASQUERADE
#        post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone zone1


source /etc/network/interfaces.d/*
```

9) Redémarrer les services wpa_supplicant et de réseau pour connecter l'adaptateur sans fil au réseau wifi 

```shell
systemctl restart wpa_supplicant && systemctl restart networking
```

10) Supprimer l'abonnement nag message:

```shell
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```

11) Se connecter à l'interface web proxmox: `https://<ip_of_your_wifi_adapter>:8006`

12)  Créer une configuration SDN (Centre de données --> SDN):

Zones: Simple, ID = Zone1 (utiliser tout nom que vous souhaitez pour ID)  
Vnet: Name = vnet1 (utiliser tout nom que vous souhaitez comme Nom), Zone = Zone1 (doit correspondre à l'ID de zone)  
Subnet: Subnet = 192.168.3.0/24, Gateway = 192.168.3.1, SNAT (check)

13) Appliquer la configuration: SDN --> Appliquer

14) Editer `/etc/network/intefaces` et oter les commentaires:

```
auto lo
iface lo inet loopback

iface eno1 inet manual

auto wlp1s0
iface wlp1s0 inet manual
    address 192.168.10.240/24
    gateway 192.168.10.1

auto vmbr0
iface vmbr0 inet static
	address 192.168.0.215/24
	gateway 192.168.0.205
	bridge-ports eno1
	bridge-stp off
	bridge-fd 0

               post-up echo 1 > /proc/sys/net/ipv4/ip_forward
               post-up iptables -t nat -A POSTROUTING -s '192.168.3.0/24' -o wlp1s0 -j MASQUERADE
               post-up iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone zone1  ## Zone ID
               post-down iptables -t nat -D POSTROUTING -s '192.168.3.0/24' -o wlp1s0 -j MASQUERADE
               post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone zone1  ## Zone ID


source /etc/network/interfaces.d/*
```

15) Redémarrer le service réseau :

```shell
systemctl restart networking
```
