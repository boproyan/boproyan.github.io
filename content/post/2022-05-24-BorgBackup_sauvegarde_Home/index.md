+++
title = 'BorgBackup sauvegarde Home'
date = 2022-05-24 00:00:00 +0100
categories = borgbackup
+++
## Borg sauvegarde home PC1

![](borg-logo.png)

* [Archlinux : Sauvegarde des données avec BORG](https://wiki.archlinux.fr/Sauvegarde_des_donn%C3%A9es_avec_BORG)

Installation normale  
Le paquet borg étant dans les dépôts Community il suffit de:

    sudo pacman -S borg

Optionnels recommandés :

    sudo pacman -S fuse python-llfuse


**Sauvegarde PC1 "local" sur unité "sav"**

* *La sauvegarde sera exécutée avec l'utilisateur borg sous root pour les dossiers et fichiers systèmes*
* **/root/.borg** : dossier des fichiers (script, passphrase et exclusions)
* **/srv/sav/borg-backups** (lien ~/media/sav/borg-backups) : dossier des dépôts

Créer les dossiers

    sudo mkdir -p /srv/sav/borg-backups /root/.borg
    sudo chown $USER.$USER -R  /srv/sav/borg-backups

Création d'un fichier **passphrase** ( ex: qui contient 6 mots aléatoires séparés par un espace) qui servira à automatiser la procédure de sauvegarde sauvegarde (**/root/.borg/passphrase**)

    sudo nano /root/.borg/passphrase # contient la "phrase de passe"

Création d'un fichier **exclusions** qui contient toutes les exclusions de dossiers et fichiers , une exclusion par ligne (**/root/.borg/exclusions**)

    sudo nano /root/.borg/exclusions

```
/home/yannick/Bureau
/home/yannick/Modèles
/home/yannick/Partage
/home/yannick/Public
/home/yannick/Téléchargements
/home/yannick/Vidéos
/home/yannick/temp
/home/yannick/vps
/home/yannick/media
/home/yannick/VirtualBox\ VMs
```

**Initialisation dépôt** "yannick-pc" de sauvegarde chiffré, la "phrase de passe" sera demandée 

```bash
sudo -s
export BORG_PASSPHRASE="`cat /root/.borg/passphrase`"
BORG_REPOSITORY=/srv/sav/borg-backups/yannick-pc
borg init --encryption=repokey-blake2 /srv/sav/borg-backups/yannick-pc
```

**Création du script borg-backup**

*Borg a la déduplication, le contrôle de cohérence, le cryptage, est configurable par des variables d'environnement, la récupération des fichiers avec FUSE pour le montage des dépôts. En utilisant borg prune, on peut avoir une haute fréquence de sauvegardes avec une gestion du cycle des repo : 6 heures, 7 jours, 4 semaines, et 6 mois.*

Le fichier script **/root/.borg/borg-backup**

    sudo nano /root/.borg/borg-backup

```
#!/bin/bash

set -e
 
# Vérifier si utilisateur est root
if [ $(id -u) != "0" ]; then
    echo "Erreur : Vous devez être root pour exécuter ce script !"
    exit 1
fi

BORG_REPOSITORY=/mnt/lacie/data/borg-backups/yannick-pc
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
echo "Exécution borg-backup yannick-pc $BACKUP_DATE" 
export BORG_PASSPHRASE="`cat /root/.borg/passphrase`"
BORG_ARCHIVE=${BORG_REPOSITORY}::${BACKUP_DATE}

# Sauvegarde de tout /home à l'exception de quelques répertoires et fichiers exclus
borg create \
-v --stats --compression lzma,9 \
--exclude-from /root/.borg/exclusions --exclude-caches \
$BORG_ARCHIVE \
/home/yannick 

# En cas d'erreur de sauvegarde, réinitialiser le mot de passe et quitter.
if [ "$?" = "1" ] ; then
    export BORG_PASSPHRASE=""
    exit 1
fi

# Prune la repo de sauvegardes supplémentaires
borg prune \
-v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 \
$BORG_REPOSITORY 
 
# Inclure espace occupé dans le journal.
echo 'Espace occupé : ' $(df -h |egrep '/dev/mapper/vg--nas--one-sav' |awk '{print $5}')
 
borg list $BORG_ARCHIVE
 
# réinitialiser le mot de passe
export BORG_PASSPHRASE=""
echo "Fin exécution borg-backup" 
exit 0
```

Droits en exécution

    sudo chmod +x /root/.borg/borg-backup

**Utiliser Borg avec SystemD service + timer**  
Pour une automatisation en utilisant systemd :

* vous devez créer un fichier **borg-backup.service** et le mettre dans le dossier */etc/systemd/system/system/*
* Pour une exécution régulière (parce que ce n'est pas un processus "daemon"), un fichier **borg-backup.timer** , dans le même dossier que **borg-backup.service** 
* Le nom de fichier avant le suffixe  ".service" et ".timer" doit correspondre pour que systemd reconnaisse ce qu'il doit faire. 

**Service : /etc/systemd/system/borg-backup.service**

```
[Unit]
Description=Borg Backup Service
After=network-online.target
 
[Service]
Type=simple
ExecStart=/bin/bash /root/.borg/borg-backup 
StandardError=journal
# TimeoutStartSec= configure le délai d'attente pour attendre le démarrage du service.
TimeoutStartSec=600

[Install]
# niveau d'exécution multi utilisateur
WantedBy=multi-user.target
```

Adapter User= et Group= dans le fichier borg-backup.service pour un utilisateur qui a les droits sur la sauvegarde. Exécuter le service par défaut mettrait la propriété root:root sur la sauvegarde.

Il est possible de tester le service avec un simple

    sudo systemctl daemon-reload         # on vient de créer un nouveau service
    sudo systemctl start borg-backup.service

l'état de votre système de sauvegarde 

    systemctl status borg-backup

Le journal

    journalctl   -u borg-backup 

Valider le service *borg-backup.service* 

    sudo systemctl enable borg-backup

**Timer : /etc/systemd/system/borg-backup.timer**

```
[Unit]
Description=Borg Backup Timer
 
[Timer]
#
OnBootSec=15min
OnUnitActiveSec=1d
Unit=borg-backup.service
 
[Install]
WantedBy=timers.target
```

* **Install** va créer la dépendance pour que votre “timer” soit bien exécuté et pris en compte par systemd.
* **OnActiveSec=** définit une minuterie par rapport au moment où la minuterie elle-même est activée.  
* **OnBootSec=** définit une temporisation relative au démarrage de la machine.  
* **OnStartupSec=** définit une minuterie relative à la date du premier démarrage de systemd.  
* **OnUnitActiveSec=** définit une minuterie relative à l'unité dans laquelle la minuterie est utilisée.a été activée pour la dernière fois.   
* **OnUnitInactiveSec=** définit une temporisation relative à lorsque l'unité que la minuterie est en train d'activer a été désactivée pour la dernière fois.  

>Plusieurs directives peuvent être combinées du même type et de types différents. Pour par exemple, en combinant OnBootSec= et OnUnitActiveSec=, il est possible de définir une minuterie qui s'écoule à intervalles réguliers et active un service spécifique à chaque fois du temps.  

Les arguments des directives sont des intervalles de temps configurés en secondes.    
Exemple : "OnBootSec=50" signifie 50 s après le démarrage. L'argument peut aussi inclure le temps unités.    
Exemple : "OnBootSec=5h 30min" signifie 5 heures et 30 minutes après que boot-up.  

**Activation et démarrage du timer**

Ensuite, pour qu'il soit actif, il faut prévenir systemd

    sudo systemctl enable borg-backup.timer
    sudo systemctl start borg-backup.timer

Vous pouvez vérifier quand votre prochaine sauvegarde se déclenchera à l'aide des minuteries

    systemctl list-timers 

```
NEXT                         LEFT     LAST                         PASSED       UNIT                         AC>
Wed 2019-12-11 13:15:52 CET  23h left Tue 2019-12-10 13:15:52 CET  8s ago       borg-backup.timer            bo>
```

**Débloquer un dépôt BORG verrouillé**

Exemple , déblocage du dépôt **yannick-pc**  

```
export BORG_PASSPHRASE="`cat ~root/.borg/passphrase`"
export BORG_RSH='ssh -i /root/.ssh/yannick-pc_ed25519'
BORG_REPOSITORY=ssh://borg@xoyaz.xyz:55036/srv/data/borg-backups/yannick-pc
borg break-lock $BORG_REPOSITORY
```
