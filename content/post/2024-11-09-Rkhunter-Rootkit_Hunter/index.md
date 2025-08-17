+++
title = 'Rkhunter (Rootkit Hunter)'
date = 2024-11-09 00:00:00 +0100
categories = ['outils']
+++
*Rootkit Hunter analyse les systèmes pour détecter les rootkits, portes dérobées, renifleurs et exploits connus et inconnus.*

Il vérifie :

* SHA256 hash 
* les fichiers couramment créés par rootkits
* les exécutables avec des permissions de fichiers anormales
* les chaînes suspectes dans les modules du noyau
* les fichiers cachés dans les répertoires système
* peut éventuellement scanner dans les fichiers.

Utiliser seul **rkhunter** ne garantit pas qu'un système n'est pas compromis. Il est recommandé d ' effectuer des essais supplémentaires, tels que le **[chkrootkit](https://linuxcapable.com/how-to-install-chkrootkit-on-ubuntu-linux/ 'How to Install Chkrootkit on Ubuntu 24.04, 22.04, or 20.04')**.

## Rkhunter

![](rkhunter01.png)

* [Installer et configurer Rootkit-Hunter](https://wiki-sinp.cbn-alpin.fr/serveurs/installation/rkhunter)
* [Comment installer et configurer Rootkit Hunter sur Ubuntu/Debian](https://www.webhi.com/how-to/fr/comment-installer-et-configurer-rootkit-hunter-sur-ubuntu-debian/)

### Mise à jour système

Avant installation, mise à jour système debian

    sudo apt update && sudo apt upgrade -y

### Installation et configuration

Installation

    sudo apt install rkhunter

Configuration  


* Fichiers de config : `/etc/rkhunter.conf` et `/etc/default/rkhunter`
* Fichier de log : `/var/log/rkhunter.log`


Après avoir installé Rkhunter, vous devez le configurer 

    sudo nano /etc/default/rkhunter

```bash
# Défauts pour rkhunter tâches automatiques
# sourced by /etc/cron.*/rkhunter and /etc/apt/apt.conf.d/90rkhunter
#
# C'est un fragment de shell POSIX
#

# Définir ceci à oui pour permettre les courses quotidiennes rkhunter
# (par défaut: faux)
CRON_DAILY_RUN="true"

# Définir ceci à oui pour permettre la mise à jour de la base de données hebdomadaire rkhunter
# (par défaut: faux)
CRON_DB_UPDATE="true"

# Définir ceci à oui pour permettre les rapports des mises à jour hebdomadaires de la base de données
# (par défaut: faux)
DB_UPDATE_EMAIL="false"

# Définir ceci à l'adresse e-mail où les rapports et la sortie d'exécution doivent être envoyés
# (par défaut: root)
REPORT_EMAIL="root"

# Définir ceci à oui pour activer les mises à jour automatiques de la base de données
# (par défaut: faux)
APT_AUTOGEN="true"

# Les Nicenesses vont de -20 (horaire le plus favorable) à 19 (le moins favorable)
# (par défaut: 0)
NICE="0"

# Si le contrôle quotidien est effectué lors de la batterie
# powermgmt-base est nécessaire pour détecter si fonctionnant sur la batterie ou sur la puissance AC
# (par défaut: faux)
RUN_CHECK_ON_BATTERY="false"
```

Indiquer les faux positifs, en ajoutant au fichier de config `/etc/rkhunter.conf`

    sudo nano /etc/rkhunter.conf

```
ALLOW_SSH_ROOT_USER=prohibit-password
 
# Config permettant la mise à jour pour éviter l'erreur :
# Invalid WEB_CMD configuration option: Relative pathname: "/bin/false"
UPDATE_MIRRORS=1
MIRRORS_MODE=0
WEB_CMD=""
 
# Gestion des emails
MAIL-ON-WARNING=postmaster@rnmkcy.eu
MAIL_CMD=mail -s "[rkhunter] Avertissements sur ${HOST_NAME}" -r postmaster@rnmkcy.eu
 
# Option évitant les faux positifs en se basant sur Dpkg
# ATTENTION : lancer ''rkhunter --propupd'' après avoir modifier cet option !
PKGMGR=DPKG
 
# Pour Debian 10 uniquement, corriger l'emplacement des scripts suivant (/usr/bin/ au lieu de /bin) :
SCRIPTWHITELIST=/usr/bin/egrep
SCRIPTWHITELIST=/usr/bin/fgrep
SCRIPTWHITELIST=/usr/bin/which
 
# Désactiver les faux positifs sur db-srv
ALLOWDEVFILE="/dev/shm/PostgreSQL.*"
 
# Exemples de faux positifs à désactiver :
ALLOWHIDDENDIR="/dev/.udev"
ALLOWHIDDENDIR="/dev/.static"
ALLOWDEVFILE="/dev/.udev/rules.d/root.rules"
```

ATTENTION : suite à l'installation et configuration de Rkhunter, il est nécessaire de lancer la commande suivante : 

    sudo rkhunter --propupd

```
[ Rootkit Hunter version 1.4.6 ]
File updated: searched for 181 files, found 143
```

### Mise à jour de la base de données Rkhunter

Avant de lancer l’analyse de Rkhunter, vous devez mettre à jour sa base de données de rootkits et de logiciels malveillants connus

    sudo rkhunter update

### Exécuter analyse rkhunter

lancer l’analyse Rkhunter

    sudo rkhunter --check

Patienter...

### Examiner rapport Rkhunter

Une fois l’analyse de Rkhunter terminée, examiner le rapport de Rkhunter afin d’identifier toute menace potentielle.   
Le rapport Rkhunter se trouve dans le fichier  `/var/log/rkhunter.log`  

```
System checks summary
=====================

File properties checks...
    Files checked: 143
    Suspect files: 0

Rootkit checks...
    Rootkits checked : 498
    Possible rootkits: 0

Applications checks...
    All checks skipped

The system checks took: 42 minutes and 12 seconds

All results have been written to the log file: /var/log/rkhunter.log

No warnings were found while checking the system.

```

## Utilisation et commandes

* Vérifier dernière version : `rkhunter --versioncheck`
* Mettre à jour le programme : `rkhunter --update`
* Lister les différents tests effectués : `rkhunter --list`
* **ATTENTION** : suite à l'installation et configuration de Rkhunter, il est nécessaire de lancer la commande suivante (=> indique à Rkhunter que tout est OK) : `rkhunter --propupd`
* Effectuer une vérification : `rkhunter --checkall`
* Vérification avec juste les alertes importantes : `rkhunter -c --rwo`
* Tester uniquement les //malwares// : `rkhunter -c -sk --enable malware`
* Accéder aux logs de RkHunter : `nano /var/log/rkhunter.log`

