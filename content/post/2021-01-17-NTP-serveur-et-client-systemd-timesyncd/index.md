+++
title = 'NTP serveur ,timedatectl et systemd-timesyncd'
date = 2021-01-17 00:00:00 +0100
categories = ntp
+++
## Mise à jour automatique heure serveur (NTP)

![ntp](ntp.png)  

Avoir un serveur à la bonne heure et synchronisé avec les autres serveurs Internet permet d'avoir une référence de temps commune à tout le monde.  
On utilise pour cela le protocole NTP (Network Time Protocol) qui permet à un ordinateur de synchroniser son horloge sur d'autres ordinateurs de référence via internet.  
Sur l'internet, il existe des serveurs de temps qui fournissent l'heure correcte. Votre fournisseur d'accès Internet peut fournir un service de temps et celui-ci serait votre source la plus proche et probablement la plus précise. Il existe encore de nombreux serveurs NTP indépendants auxquels vous pouvez vous connecter, mais la meilleure source est http://pool.ntp.org.


### Installation du paquet NTP

On commence par installer le paquet ntp qui contient tout le nécessaire.

    sudo apt install ntp

A la fin de l'installation, le service ntp démarre automatiquement

### Configuration /etc/ntp.conf

Le fichier de configuration de ntp se trouve dans **/etc/ntp.conf**

    sudo nano /etc/ntp.conf

Pour un serveur qui est hébergé en France , spécifier des serveurs ntp situés en France  (cf. <http://www.pool.ntp.org/zone/fr>)  
Pour cela, remplacer les 4 entrées server du fichier /etc/ntp.conf 

```
pool 0.debian.pool.ntp.org iburst
pool 1.debian.pool.ntp.org iburst
pool 2.debian.pool.ntp.org iburst
pool 3.debian.pool.ntp.org iburst
```

par

```
pool 0.fr.pool.ntp.org iburst
pool 1.fr.pool.ntp.org iburst
pool 2.fr.pool.ntp.org iburst
pool 3.fr.pool.ntp.org iburst
```

**ntp** utilise plusieurs serveurs pour se mettre à l'heure, cela augmente sa précision et en cas d'indisponibilité sur un des serveurs de référence, les autres sont toujours là ...  
Vous remarquerez l'option *iburst* spécifiés après le nom du serveur, en cas d'indisponibilité du serveur, **ntp** essaie plusieurs fois avant d’abandonner  
Redémarrer le service ntp pour activer nos modifications

    sudo systemctl restart ntp

Utiliser la commande ntpq -p pour afficher les informations de synchronisation (notamment les serveurs qui vous servent de référence de temps).

    ntpq -p

```
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 0.fr.pool.ntp.o .POOL.          16 p    -   64    0    0.000    0.000   0.001
 1.fr.pool.ntp.o .POOL.          16 p    -   64    0    0.000    0.000   0.001
 2.fr.pool.ntp.o .POOL.          16 p    -   64    0    0.000    0.000   0.001
 3.fr.pool.ntp.o .POOL.          16 p    -   64    0    0.000    0.000   0.001
```

## Utiliser timesyncd au lieu de ntp

1-Retirer les services ntp (si installé)

    systemctl stop ntp.service
    systemctl disable ntp.service
    systemctl status ntp.service

```
● ntp.service - Network Time Service
Loaded: loaded (/lib/systemd/system/ntp.service; disabled; vendor preset: enabled)
``` 

2-Configurer timesyncd pour utiliser un serveur de temps

    nano /etc/systemd/timesyncd.conf

```
[Time]  
NTP=  
FallbackNTP=0.fr.pool.ntp.org 1.fr.pool.ntp.org 2.fr.pool.ntp.org 3.fr.pool.ntp.org
```

3-Activer la synchro ntp

    timedatectl set-ntp true

4-Activer le service timesyncd :

    systemctl enable systemd-timesyncd.service
    systemctl start systemd-timesyncd.service
    systemctl status systemd-timesyncd.service

```
Active: active (running) since Tue 2020-08-25 17:42:51 CEST; 36s left
Status: "Synchronized to time server for the first time 51.77.12.38:123 (0.fr.pool.ntp.org)."
```

Si les erreurs suivantes apparaissent :

```
ConditionFileIsExecutable=!/usr/sbin/ntpd was not met  
Condition check resulted in Network Time Synchronization being skipped.  
```

Supprimer ntp : `apt remove ntp ntpdate`

5-Vérifier que tout est OK :

    timedatectl status

```
               Local time: mar. 2020-08-25 17:52:33 CEST
           Universal time: mar. 2020-08-25 15:52:33 UTC
                 RTC time: mar. 2020-08-25 15:52:34
                Time zone: Europe/Paris (CEST, +0200)
    System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

## Debian – NTP, systemd-timesyncd.service et timedatectl

Les systèmes d'exploitation modernes font la distinction entre les deux types d'horloges suivants : 

* Une horloge temps réel (« Real-Time Clock », ou RTC), communément appelée horloge matérielle, (habituellement un circuit intégré sur la carte système) est complètement indépendante de l'état actuel du système d'exploitation et fonctionne même lorsque l'ordinateur est éteint. 
* Une horloge système, également appelée horloge logicielle, est habituellement maintenue par le noyau et sa valeur initiale est basée sur l'horloge temps réel. Une fois le système démarré et l'horloge système initialisée, celle-ci est entièrement indépendante de l'horloge temps réel. 

Le temps système est toujours conservé sous le format du temps universel coordonné (« Coordinated Universal Time », ou UTC) puis converti dans les applications au temps local selon les besoins. Le temps local correspond à l'heure réelle dans votre fuseau horaire et prend en compte l'heure d'été (« daylight saving time », ou DST). L'horloge temps réel peut utiliser l'heure UTC ou l'heure locale. L'heure UTC est recommandée. 

L'utilitaire **timedatectl** est distribué dans le cadre du gestionnaire de services et systèmes systemd et permet de réviser et modifier la configuration de l'horloge système. Vous pouvez utiliser cet outil pour modifier l'heure et la date actuelle, pour définir le fuseau horaire, ou pour activer la synchronisation automatique de l'horloge système avec un serveur distant. 


### Configuration

Fichier de configuration des serveurs NTP : `/etc/systemd/timesyncd.conf`  
Les modifications se font en mode su  

Exemple avec des adresses IP au lieu de nom de domaine.  
Permet la synchronisation NTP même si aucun serveur DNS n’est joignable :

    /etc/systemd/timesyncd.conf

```
[Time]
NTP=145.238.203.14 145.238.203.10
#FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org 3.debian.pool.ntp.org
```

Les serveurs utilisés sont décrits ici : <https://syrte.obspm.fr/spip/services/ref-temps/article/diffusion-de-l-heure-par-internet-ntp-network-time-protocol>

Relancer le service

    systemctl daemon-reload
    systemctl restart systemd-timesyncd.service

### Activation

Vérifier si l’utilisation des serveurs NTP est active :

    timedatectl status

```
               Local time: Sun 2021-01-17 10:51:22 CET
           Universal time: Sun 2021-01-17 09:51:22 UTC
                 RTC time: Sun 2021-01-17 09:51:23
                Time zone: Europe/Paris (CET, +0100)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

Dans ce cas, « Network time on » et « NTP synchronized » sont à « yes », donc l’utilisation de serveurs NTP est activée.

Dans le cas contraire, il faut l’activer par cette commande  :

    timedatectl set-ntp true

Pour désactiver :

    timedatectl set-ntp false

Vérifier la prise en compte du serveur avec :

    systemctl status systemd-timesyncd.service

```
● systemd-timesyncd.service - Network Time Synchronization
   Loaded: loaded (/lib/systemd/system/systemd-timesyncd.service; enabled; vendor pres
  Drop-In: /usr/lib/systemd/system/systemd-timesyncd.service.d
           └─disable-with-time-daemon.conf
   Active: active (running) since Sun 2021-01-17 10:51:17 CET; 9min ago
     Docs: man:systemd-timesyncd.service(8)
 Main PID: 14311 (systemd-timesyn)
   Status: "Synchronized to time server for the first time 145.238.203.14:123 (145.238
    Tasks: 2 (limit: 2146)
   Memory: 932.0K
   CGroup: /system.slice/systemd-timesyncd.service
           └─14311 /lib/systemd/systemd-timesyncd
```

La ligne « Status » affiche le serveur utilisé.

### Mise à l’heure manuelle

Veiller à ce que « Network time on » soit à no :

    timedatectl set-ntp false

Afficher l’heure actuelle avec :

    timedatectl status

Utiliser le même format pour changer l’heure avec cette commande :

    timedatectl set-time "2018-07-24 16:40:00"

Si le message d’erreur « Failed to parse time specification » est renvoyé, vérifier si la timezone est configurée.

### Modifier la timezone

Si la timezone est à modifier, utiliser cette commande pour afficher les timezones disponibles :

    timedatectl list-timezones

Pour cette commande pour définir la timezone sur « Europe/Paris » par exemple :

    timedatectl set-timezone "Europe/Paris"

Vérifier la prise en compte avec :

    timedatectl status
