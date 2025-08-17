+++
title = 'Onduleur "Eaton Protection Station 800 USB" sur serveur Debian + envoi SMS'
date = 2019-09-04 00:00:00 +0100
categories = ['debian', 'serveur']
+++
# Onduleur Eaton Protection Station 800 USB

![eaton](eaton-logo.png)

* [Eaton Protection Station - 650/800 - Manuel d’installation et d’utilisation](http://lit.powerware.com/ll_download.asp?file=Eaton_Protection_Station_650_800_Installation_and_user_manual_FR.pdf&ctry=80)  
* [Installation et gestion d'un UPS USB en réseau sous linux](http://ovanhoof.developpez.com/upsusb/)  
* [Network UPS Tools (nut), doc ubuntu](https://doc.ubuntu-fr.org/nut#network_ups_tools_nut)
* UPS = Uninterruptable Power System
* ASI = Alimentation Sans Interruption
* Le périphérique UPS **Eaton Protection Station 800 USB** à gérer est de type "HID"  

## Matériel

![alt text](/images/onduleur-eaton.png "Eaton Protection Station 800 USB")

No | Eaton Protection Station - 650/800
--- | ---
7  | 4 prises filtrées.
8  | 4 prises secourues par batterie.
9  | Voyant allumé, protection anti-surtensions active sur les 8 prises.
10 | Voyant allumé, défaut de l'Alimentation Sans Interruption.
11 | Bouton de mise en service ou d'arrêt des prises secourues.
12 | Disjoncteur de protection.

Passer en mode su

	sudo -s

Connecter l'onduleur liaison USB sur un port disponible du serveur , vérifier par `dmesg`   

    dmesg

```
[10888050.365095] hid-generic 0003:0463:FFFF.0001: hiddev0,hidraw0: USB HID v10.10 Device [EATON Protection Station] on usb-0000:00:1d.2-1/input0
```

## Network UPS Tools (NUT)

NUT est un ensemble d'outils permettant de monitorer un système relié à un ou des onduleurs.Il se compose de plusieurs éléments :  

* le démon **nut** lancé au démarrage du système
* le démon **upsd** qui permet d'interroger l'onduleur, il est lancé sur le PC relié à l'onduleur
* le démon **upsmon** qui permet de monitorer et lancer les commandes nécessaires sur le réseau ondulé (arrêt de machines ...)
* différents programmes pour envoyer des commandes manuellement à l'onduleur : upsc, upsdrvctl ...

**upsd** peut communiquer avec plusieurs onduleurs si nécessaire.  
**upsmon** interroge à intervalle régulier la machine du réseau sur laquelle est lancée upsd.


### Installer les paquets NUT sous Debian

Installez nut, les paquets supplémentaires seront automatiquement installés : 

	apt install nut -y


### Configuration mode **standalone**  (/etc/nut/nut.conf)

Dans le mode **standalone**, l'onduleur est relié à la machine actuelle (serveur).  

```
MODE=standalone
```

Le monitorage de l'onduleur est effectué depuis cette même machine.  
Le démon **nut** doit lancer **upsd** et **upsmon** (en mode "master").

### Communication avec l'onduleur, driver et port (/etc/nut/ups.conf)

Communication avec l'onduleur, choisir le driver et le port , ce qui fait dans le fichier **/etc/nut/ups.conf**  
Le fichier **/usr/share/nut/driver.list** contient une liste d'onduleurs et le driver qu'il faut utiliser. 

    cat /usr/share/nut/driver.list | grep eaton

```
"Eaton"	"ups"	"5"	"5E1100iUSB"	"USB port"	"usbhid-ups"	# http://powerquality.eaton.com/5E1100iUSB.aspx?CX&GUID=8D85FE66-3102-427C-8F33-B8D56BBDD4D3
"Eaton"	"ups"	"5"	"E Series DX UPS 1-20 kVA"	""	"blazer_ser"	# http://www.eaton.com/Eaton/ESeriesUPS/DXUPS/
"Eaton"	"ups"	"5"	"Powerware 3105"	"USB"	"bcmxcp_usb"	# http://powerquality.eaton.com/Products-services/Backup-Power-UPS/3105-eol.aspx
```

Le choix se porte sur le driver "usbhid-ups"

    nano /etc/nut/ups.conf

```ini
maxretry = 3
[eaton]
	driver = usbhid-ups
	port = auto
	desc = "Eaton Protection Station 800"
```

### Configuration daemon (/etc/nut/upsd.conf)

Configurer le démon réseau au niveau des accès via le fichier **/etc/nut/upsd.conf**  
On écoute en local sur le réseau port 3493

    nano /etc/nut/upsd.conf

```
LISTEN 127.0.0.1 3493
LISTEN ::1 3493
MAXCONN 32 #Nombre maximal de connections simultanées
```

### Utilisateurs (/etc/nut/upsd.users)

Créer les utilisateurs (administrateur et superviseur) fichier **/etc/nut/upsd.users**

```
# 1 seul utilisateur "onduleur" avec tous les droits !
[onduleur]
password = dfYRtY38
upsmon master
```

### Moniteur de supervision (/etc/nut/upsmon.conf)

Le moniteur de supervision va surveiller l'onduleur et lancer différentes actions en fonction des évènements constatés ,fichier **/etc/nut/upsmon.conf**  

```
# Notre serveur n'a pas d'alimentation redondante = 1
MINSUPPLIES 1
# Commande d'arrêt du serveur en cas de fin d'autonomie
SHUTDOWNCMD /usr/sbin/poweroff
# Commande lancée quand quelque chose se passe
NOTIFYCMD /sbin/upssched
# Interval entre deux interrogations de upsd
POLLFREQ 5
# Intervalle entre deux interrogations de upsd en mode "batterie"
POLLFREQALERT 5
# Temps d'attente pour la déconnexion des upsmon esclaves
HOSTSYNC 15
# Temps pendant lequel on tolère la non-réponse d'un onduleur, multiple de POLLFREQ
DEADTIME 15
# Fichier d'état
POWERDOWNFLAG /etc/killpower
# Intervalle de 1/2 journée, pour répéter le message de "remplacement de batteries - NOTIFY_REPLBATT"
RBWARNTIME 43200
# Interval de 5 minutes, pour répéter le message de "onduleur injoignable - NOTIFY_NOCOMM"
NOCOMMWARNTIME 300
# Intervalle entre la "notification d'arrêt - NOTIFY_SHUTDOWN" et le lancement de SHUTDOWNCMD
FINALDELAY 5
# On surveille l'onduleur qui est directement relié (master)
MONITOR eaton@localhost 1 onduleur dfYRtY38 master
# Actions spécifiques autres que par défaut (SYSLOG et WALL) réalisées en fonction de l'état retourné par l'onduleur
NOTIFYFLAG COMMBAD EXEC
NOTIFYFLAG ONLINE SYSLOG+EXEC
NOTIFYFLAG ONBATT SYSLOG+EXEC
NOTIFYFLAG LOWBATT EXEC
NOTIFYFLAG REPLBATT SYSLOG+EXEC
NOTIFYFLAG SHUTDOWN EXEC
NOTIFYFLAG COMMOK IGNORE
```

>On remarque que, à part l'arrêt du système sur fin des batteries qui est géré par **upsmon**, toutes les notifications le sont par **upssched**.  

### Distributeur d'évènements (/etc/nut/upssched.conf)

L'utilitaire **upssched** permet de « temporiser » les actions liées aux évènements générés par **upsmon**.  
C'est surtout intéressant lorsque des évènements furtifs se produisent (microcoupures) fichier **/etc/nut/upssched.conf**.  

```
# Script lancé par upssched pour gérer les évènements et les timers associés
CMDSCRIPT /etc/nut/upssched-cmd
# Fichier pour noter les états internes de upssched
PIPEFN /var/run/nut/upssched.pipe
# Fichier de lock pour éviter un conflit en cas de notification de deux évènements simultanés
LOCKFN /var/run/nut/upssched.lock

# En cas de perte de communication avec l'onduleur
AT COMMBAD * EXECUTE perte-liaison

# En cas de retour secteur, on stope la minuterie et on notifie
AT ONLINE * CANCEL-TIMER attente-retour-secteur
AT ONLINE * EXECUTE charge-sur-secteur

# En cas de perte de secteur, on se laisse au maximum 20 minutes avant d'agir et on notifie
AT ONBATT * START-TIMER attente-retour-secteur 1200
AT ONBATT * EXECUTE charge-sur-batterie

# En cas de niveau de batteries trop bas
AT LOWBATT * EXECUTE batteries-vides

# En cas de fin de vie des batteries on sera prévenu
AT REPLBATT * EXECUTE batteries-hs

# En cas d'arrêt (fin des 20 minutes  ou fin d'autonomie), on notifie.
AT SHUTDOWN * EXECUTE arret-en-cours
```

### Prise en charge des évènements (script /etc/nut/upssched-cmd)

Fichier **/etc/nut/upssched-cmd**  

```bash
#!/bin/bash

# Fichier /etc/nut/upssched-cmd 
#
# Début du programme
#
if [ $# = 0 ]
then
    echo "Il faut au minimum un argument !"
    exit 0
fi
#
# Analyse argument
#
case $1 in

 	"charge-sur-batterie")
			sujet="Onduleur - Charge sur batteries"
			message="L'onduleur est passé sur batteries ... L'arrêt système sera demandé si le secteur ne revient pas." 
	;;

	"attente-retour-secteur")
		sujet="Onduleur - Fin d'attente de retour secteur"
		message="Cela fait trop longtemps que le secteur est absent...Un arrêt forcé est en cours !"

		# Demande d'arrêt forcé (force shut down)
		/sbin/upsmon -c fsd
	;;

	"charge-sur-secteur")
		sujet="Onduleur - Charge sur secteur"
		message="L'onduleur est revenu sur secteur."
	;;

	"batteries-vide")
		sujet="Onduleur - Batteries vides"
		message="Les batteries sont vides, l'arrêt est imminent."
	;;

	"arret-en-cours")
		sujet="Onduleur - Arrêt en cours"
		message="Le système est en cours d'arrêt."
	;;

	"perte-liaison")
                sujet="Onduleur - Perte de liaison avec l'onduleur"
                message="La communication avec l'onduleur est interrompue."
         ;;


	"batteries-hs")
                sujet="Onduleur - URGENT - batteries HS"
                message="Les batteries sont à remplacer d'urgence."
	;;

	*)
		sujet="Onduleur - Commande inconnue ..." 
		message="Une commande inconnue a été envoyée par l'onduleur...La commande est : $comment"
	;;

esac


# Envoi du sms
NOM="12345678" 
PASS="lpoytFjEZA4ph6"

envoi=$(curl -i --insecure "https://smsapi.free-mobile.fr/sendmsg?user=$NOM&pass=$PASS&msg=$sujet%20$message" 2>&1)
retour_HTTP=$(echo "$envoi" | awk '/HTTP/ {print $2}')
case $retour_HTTP  in
	200)
		echo "Le message a été envoyé correctement";;
	400)
		echo "le couple expéditeur/mot de passe est erroné, veuillez les vérifier dans le script";;
	402)
		echo "Trop de SMS ont été envoyés en trop peu de temps. Veuillez renouveler ultérieurement";;
	403)
		echo "Le service n’est pas activé sur l’espace abonné. Veuillez l'activer S.V.P";;
	500)
		echo " Erreur côté serveur. Veuillez réessayez ultérieurement.";;
esac
exit 0
```

Rendre exécutable le script  

	chmod +x /etc/nut/upssched-cmd

Redémarrer les services

	systemctl restart nut-server.service
	systemctl restart nut-client.service

Status

    systemctl status nut-server.service

```
● nut-server.service - Network UPS Tools - power devices information server
   Loaded: loaded (/lib/systemd/system/nut-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2019-07-22 23:04:26 CEST; 1min 3s ago
  Process: 32434 ExecStart=/sbin/upsd (code=exited, status=0/SUCCESS)
 Main PID: 32435 (upsd)
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/nut-server.service
           └─32435 /lib/nut/upsd

juil. 22 23:04:26 srvxo upsd[32434]: fopen /var/run/nut/upsd.pid: No such file or directory
juil. 22 23:04:26 srvxo upsd[32434]: listening on ::1 port 3493
juil. 22 23:04:26 srvxo upsd[32434]: listening on 127.0.0.1 port 3493
juil. 22 23:04:26 srvxo upsd[32434]: listening on ::1 port 3493
juil. 22 23:04:26 srvxo upsd[32434]: Connected to UPS [eaton]: usbhid-ups-eaton
juil. 22 23:04:26 srvxo systemd[1]: Started Network UPS Tools - power devices information server.
juil. 22 23:04:26 srvxo upsd[32435]: Startup successful
juil. 22 23:04:34 srvxo upsd[32435]: User onduleur@::1 logged into UPS [eaton]
juil. 22 23:04:37 srvxo upsd[32435]: User onduleur@::1 logged out from UPS [eaton]
juil. 22 23:04:37 srvxo upsd[32435]: User onduleur@::1 logged into UPS [eaton]

```

    systemctl status nut-client.service

```
● nut-monitor.service - Network UPS Tools - power device monitor and shutdown controller
   Loaded: loaded (/lib/systemd/system/nut-monitor.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2019-09-03 21:11:10 CEST; 9s ago
  Process: 1486 ExecStart=/sbin/upsmon (code=exited, status=0/SUCCESS)
 Main PID: 1488 (upsmon)
    Tasks: 2 (limit: 4915)
   Memory: 1.1M
   CGroup: /system.slice/nut-monitor.service
           ├─1487 /lib/nut/upsmon
           └─1488 /lib/nut/upsmon

sept. 03 21:11:10 srvxo systemd[1]: Starting Network UPS Tools - power device monitor and shutdown controller...
sept. 03 21:11:10 srvxo upsmon[1486]: fopen /var/run/nut/upsmon.pid: No such file or directory
sept. 03 21:11:10 srvxo upsmon[1486]: Using power down flag file /etc/killpower
sept. 03 21:11:10 srvxo upsmon[1486]: UPS: eaton@localhost (master) (power value 1)
sept. 03 21:11:10 srvxo upsmon[1487]: Startup successful
sept. 03 21:11:10 srvxo systemd[1]: nut-monitor.service: Can't open PID file /run/nut/upsmon.pid (yet?) after start: No such file or di
sept. 03 21:11:10 srvxo systemd[1]: nut-monitor.service: Supervising process 1488 which is not our child. We'll most likely not notice 
sept. 03 21:11:10 srvxo systemd[1]: Started Network UPS Tools - power device monitor and shutdown controller.
sept. 03 21:11:10 srvxo upsmon[1488]: Init SSL without certificate database
```

vérifier si les daemons sont lancés  

	ps auxf |grep ups

```bash
root      1493  0.0  0.0   6092   760 pts/0    S+   21:13   0:00                      \_ grep ups
nut       1370  1.5  0.0   5280  2152 ?        Ss   20:49   0:22 /lib/nut/usbhid-ups -a eaton
nut       1419  0.0  0.0   5268   356 ?        Ss   21:03   0:00 /lib/nut/upsd
root      1487  0.0  0.0   6924  2440 ?        Ss   21:11   0:00 /lib/nut/upsmon
nut       1488  0.0  0.0  10768  3988 ?        S    21:11   0:00  \_ /lib/nut/upsmon
```

### Onduleur connecté ?

    upsc eaton@localhost

```
Init SSL without certificate database
battery.charge: 100
battery.charge.low: 20
battery.runtime: 585
battery.type: PbAc
device.mfr: EATON
device.model: Protection Station 800
device.serial: AN2E49008
device.type: ups

[...]
ups.mfr: EATON
ups.model: Protection Station 800
ups.power.nominal: 800
ups.productid: ffff
ups.serial: AN2E49008
ups.status: OL
ups.timer.shutdown: -1
ups.timer.start: -1
ups.vendorid: 0463
```

Pour éviter l'affichage du message "Init SSL without certificate database"

    upsc eaton@localhost 2>&1 | grep -v '^Init SSL'

Pour afficher un paramètre 

    upsc eaton@localhost ups.mfr 2>&1 | grep -v '^Init SSL'


Définitions des codes renvoyés par **ups.status**  

Abrégé | Définition
-------|-----------
OL | En ligne (pas de coupure de courant) (à l'opposé de OB - sur batterie)
LB | Batterie faible
SD | Shutdown load, arrêt en cours
CP | Câble d'alimentation 
CTS | Clear to Send. Reçu de l'UPS.
RTS | Ready to Send. Envoyé par le PC.
DCD | Détection de porteuse de données. Reçu de l'UPS.
RNG | Ring indicateur. Reçu de l'UPS.
DTR | Terminal de données prêt. Envoyé par le PC.
ST | Envoyer un BREAK sur la ligne de données d'émission


### Tester l'envoi de sms

Le plus simple est de débrancher la liaison USB entre l'onduleur et le serveur, puis de le rebrancher  
Un message devrait parvenir au destinataire du sms...


### Envoyer une commande

Liste des commandes disponibles

	upscmd -l eaton@localhost

```
Instant commands supported on UPS [eaton]:

beeper.disable - Disable the UPS beeper
beeper.enable - Enable the UPS beeper
beeper.mute - Temporarily mute the UPS beeper
beeper.off - Obsolete (use beeper.disable or beeper.mute)
beeper.on - Obsolete (use beeper.enable)
load.off - Turn off the load immediately
load.off.delay - Turn off the load with a delay (seconds)
load.on - Turn on the load immediately
load.on.delay - Turn on the load with a delay (seconds)
shutdown.return - Turn off the load and return when power is back
shutdown.stayoff - Turn off the load and remain off
shutdown.stop - Stop a shutdown in progress
```

Envoyer une commande

	upscmd -u <username> -p <password> <system> <command>

### Liens

* [Configurer et surveiller un onduleur avec NUT](https://wiki.debian-fr.xyz/Configurer_et_surveiller_un_onduleur_avec_NUT)
* [Piloter un onduleur sous linux le retour](https://olivier.hoarau.org/?p=3013)
* [NUT – parler à son UPS – notifications mails/push et arrêt propre du système](https://www.monlinux.net/2018/03/nut-ups-notifications-mails-et-arret/)
* [Notification push sous Linux Debian comme un simple mail!](https://www.monlinux.net/2018/03/notification-push-linux-debian-simple-mail/)