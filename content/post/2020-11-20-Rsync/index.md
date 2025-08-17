+++
title = 'Rsync  via SSH et systemd Timer'
date = 2020-11-20 00:00:00 +0100
categories = ['outils', 'ssh']
+++
# Rsync

![rsync](rsynca.png)  
*rsync (pour **r**emote **sync**hronization ou synchronisation à distance), est un logiciel de synchronisation de fichiers.  
Il est fréquemment utilisé pour mettre en place des systèmes de sauvegarde distante.
rsync travaille de manière unidirectionnelle c'est-à-dire qu'il synchronise, copie ou actualise les données d'une source (locale ou distante) vers une destination (locale ou distante) en ne transférant que les octets des fichiers qui ont été modifiés.*

La notion d’unidirectionnalité semble souvent mal comprise : elle signifie qu'en une commande, la synchronisation ne peut se faire que dans un sens. Rien n'empêche ensuite de relancer la commande une seconde fois dans l'autre sens !{: .prompt-info }

## Utilisation basique

L'appel de base :

    rsync source/ destination/

L'intérêt est une utilisation à travers le réseau. **rsync** utilise SSH par défaut

    rsync -az source/ login@serveur.org:/destination/

où: 

* `-a` ou `--archive` : est un moyen rapide de dire que vous voulez la récursivité et préserver pratiquement tout. La seule exception est que si `--files-from` a été spécifiée alors `-r` n'est pas utilisée. Ceci est équivalent à `-rlptgoD`.
* `-z` ou `--compress` : compresse les données lors du transfert. (Limite la bande passante mais augmente l'utilisation processeur et le temps de transfert : inutile en réseau local ou avec très bon débit)


Attention, il convient d'être vigilant dans l'utilisation ou non du slash (« / ») dans le chemin de la source. Ainsi, les deux commandes suivantes **ne sont pas** équivalentes :  
  `rsync source destination/`  
  `rsync source/ destination/`  
En effet, la première commande va **créer** le dossier source dans le dossier destination en ajoutant donc un niveau dans l'arborescence. La deuxième commande copie le **contenu** du dossier source dans le dossier destination.  
Autrement dit, les deux commandes suivantes sont, elles, équivalentes (\*) :  
  `rsync source destination/`  
  `rsync source/ destination/source/`  
Enfin, il faut noter que l'utilisation ou non d'un slash final dans le chemin de la destination n'a aucune incidence. Les deux commandes suivantes sont donc équivalentes :  
   `rsync source destination/`  
   `rsync source destination`  
(\*)Sauf dans le cas ou source est un lien symbolique vers un répertoire, la première commande ne copie que le lien, tandis que la seconde travaille bien à l'intérieur du répertoire  
{: .prompt-warning }

Pour une gestion du port ssh, utiliser la syntaxe suivante:

    rsync -avz source -e "ssh -p port" user@ip:"/chemin/de destination avec espaces/"

### Créer un dossier miroir

Voici un exemple d'une commande, utilisant le protocole SSH, qui copie à l'identique le dossier `<source>` vers le dossier `<destination>`.

Copie du dossier source vers le serveur :

    rsync -e ssh -avz --delete-after /home/source user@ip_du_serveur:/dossier/destination/

où :

  * `<nowiki>--</nowiki>delete-after` : à la fin du transfert, supprime les fichiers dans le dossier de destination ne se trouvant pas dans le dossier source.
  * `-z` : compresse les fichiers  (Limite la bande passante mais augmente l'utilisation processeur et le temps de transfert : inutile en réseau local ou avec très bon débit)
  * `-v` : verbeux
  * `-e ssh` : utilise le protocole  SSH 

Si les noms des chemins contiennent des espaces, on peut les écrire entre guillemet pour échapper les espaces :

    rsync -e ssh -avz --delete-after "/home/source avec espace/" user@ip_du_serveur:"/dossier/destination avec espace/"

Avec l'option `-n` la commande liste ce qu'elle va faire sans l'exécuter :

    rsync -e ssh -avzn --delete-after /home/mondossier_source user@ip_du_serveur:/dossier/destination/

### Exclure des fichiers

On peut exclure des fichiers/dossiers selon beaucoup de schémas. C'est utile pour ne pas sauvegarder le cache, les fichiers temporaires, le répertoire `.git`, la corbeille, etc…

Liste dans la commande :

    rsync --exclude="nom_de_dossier" --exclude="- autre_nom_de_dossier" source/ destination/

Un fichier de règles d'exclusion

    rsync --exclude-from=ExclusionRSync source/ destination/

Et le fichier ExclusionRSync dans le dossier courant sera de cette forme :

```
tmp
.Trash
.cache
.PlayOnLinux
```

### Inclure des fichiers

Dès lors qu'on exclut, il peut être nécessaire d'inclure.  
Exemple, vous souhaitez ne synchroniser qu'un type de fichier, mettons des .csv, cela donne

    rsync --include="*.csv" --exclude="*" source/ destination/

il faut respecter l'ordre `include` **puis** `exclude`{: .prompt-warning }

### Sauvegarde distante du serveur web


Cas présenté :

  * un serveur distant s'exécutant sous le compte système www-data.
    * ce serveur est accessible via ssh
    * on a un compte utilisateur pour se connecter sur ce serveur
    * ce compte (ou un autre) a les droits d'administration de la machine
  * une machine sur laquelle sauvegarder les données
    * on a un compte utilisateur avec le droit sudo

Pour l'exemple qui suit :

  1. sur la machine locale, on devient `www-data` pour travailler avec les droits de ce dernier
  1. www-data exécute la commande rsync qui va établir une connexion via ssh au serveur distant avec le compte `utilisateur` (on peut avoir besoin de saisir le mot de passe de l'utilisateur distant si on n'a pas déposé de clef publique)
  1. sur le serveur distant, via ssh, `utilisateur` va lancer sudo pour devenir `www-data`
  1. `www-data` exécute la commande rsync qui échange les informations avec la machine locale

Sur le serveur distant :

  * Autoriser l'utilisateur à lancer la commande `rsync` sous le compte système** www-data** grace à sudo ( sans mot de passe):   
    `sudo visudo`

```
     utilisateurssh ALL=(www-data) NOPASSWD: /usr/bin/rsync
```

  * Optionnel : déposer une clef publique ssh au besoin pour l'utilisateur

Sur la machine locale :

  * Lancer une synchronisation en tant qu'utilisateur www-data grace à sudo  
    `sudo -u www-data rsync -a --progress -e ssh --rsync-path "sudo -u www-data rsync" utilisateur@serveur_distant:/var/www/ /var/www/`  

### man rsync

[man rsync](/files/rsync.1.html)  

## Rsync via SSH+Clés

*Comment copier des fichiers avec Rsync via SSH+Clés*

### Configuration des clés publiques SSH

Sur notre serveur d'origine, nous allons générer des clés SSH publiques sans mot de passe 

    ssh-keygen -f ~/.ssh/id_rsa -q -P ""
    cat ~/.ssh/id_rsa.pub

C'est notre clé publique SSH qui peut être placée sur d'autres hôtes pour nous donner accès :

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAADAQABAAABAAABAQDLVDBIpdpfePg/a6h8au1HTKPPrg8wuTrjdh0QFVPpTI4KHctf6/FGg1NOgM+++hrDlbrDVStKn/b3Mu65///tuvY5SG9sR4vrINCSQF+++ a+YRTGU6Sn4ltKpyj 3usHERvBndtFXoD root@cloudads
```

Copiez cette clé dans votre presse-papiers et connectez-vous à votre serveur de destination.

Placez cette clé SSH dans votre fichier **~/.ssh/authorized_keys**   
Si votre dossier SSH n'existe pas, créez-le manuellement :

```
mkdir ~/.ssh
chmod 0700 ~/.ssh
touch ~/.ssh/authorized_key
chmod 0644 ~/.ssh/authorized_key
```

### Synchroniser les fichiers

Rsync est un grand utilitaire, car il vous permet, entre autres, de copier des fichiers récursivement avec compression, et sur un canal crypté.

Nous copierons un fichier de notre serveur d'origine (198.211.117.101) dans /root/bigfile.txt vers notre serveur de destination (IP : 198.211.117.129) et l'enregistrerons dans /root/bigfile.txt également.

Connectez-vous au 198.211.117.101 et synchronisez le fichier avec 198.211.117.129 :

    rsync -avz -e "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null" --progress /root/bigfile.txt 198.211.117.129:/root/


Si vous utilisez un autre utilisateur, par exemple "username", vous devrez l'ajouter devant le serveur de destination. Assurez-vous d'avoir votre clé publique dans le fichier ~/.ssh/authorized_keys de cet utilisateur :

    rsync -avz -e "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null" --progress /root/bigfile.txt username@198.211.117.129:/

Les options SSH sont utiles pour garder Rsync silencieux et ne pas vous demander une validation à chaque fois que vous vous connectez à un nouveau serveur.

>L'option **UserKnownHostsFile** définit un fichier à utiliser pour la base de données des clés hôte utilisateur au lieu de **~/.ssh/known_hosts** par défaut. Vous pouvez le définir sur **/dev/null**. Le **StrictHostKeyChecking** doit être réglé sur *"no"*, pour que ssh ajoute automatiquement de nouvelles clés hôte aux fichiers hôtes connus de l'utilisateur.  
Si cet indicateur est défini sur *"ask"*, de nouvelles clés hôte seront ajoutées aux fichiers hôte connus de l'utilisateur seulement après que l'utilisateur ait confirmé que c'est ce qu'il veut vraiment faire, et ssh refusera de se connecter aux hôtes dont la clé hôte a changé.  
Les clés hôte des hôtes connus seront vérifiées automatiquement dans tous les cas. L'argument doit être *"yes", "no" ou "ask"*. La valeur par défaut est *"ask"*.


Avec utilisateur, port, fichier clé, destination dans un domaine. La clé doit avoir les droits suivants : `chmod 0700 ~/.ssh/kvm-vps591606`

    rsync -avz -e "ssh -p 55031 -i ~/.ssh/kvm-vps591606 " --progress /srv/data/musique/* debadm@cinay.xyz:/srv/musique/

## Rsync via SSH+Clés et sudo 

*Utiliser rsync (+ ssh et sudo)*

### exemple rsync :

    rsync -av --progress --delete --stats --human-readable -e 'ssh -p xxxx' user@serveurdistant.fr:/home/user/* /home/user/

* `-a` : c'est l'option de la "mort-qui-tue". En fait ça fait tout (ou presque). C'est un moyen rapide de dire que vous voulez la récursivité et préserver pratiquement tout. C'est équivalent aux optissn combinées -rlptgoD.
* `-v` : verbeux
* `--progress` : vous indique la progression de la copie/transfert
* `--stats` : affichage de stats sur le transfert des fichiers
* --human-readable : lecture "humaine" des chiffres. Idem à l'option ls `-h` (transforme en KO, MO, GB, ...)
* `- e` : spécifie un shell distant
* `--delete` : cette option demande à rsync d'effacer tous les fichiers superflus côté réception (ceux qui ne sont pas du côté envoi); uniquement pour les répertoires synchronisés.

une fois rentré le mot de passe de l'utilisateur distant (en ayant précisé un éventuel port ssh au cas où le serveur ssh ne tournerait pas sur le traditionnel port 22), rsync va "copier" tous les fichiers du répertoire /home/user (/home/user/*) depuis le serveur distant VERS votre nouveau serveur dans le répertoire /home/user.

### rsync et sudo

Il peut arriver que certains répertoires ou fichiers ne puissent être récupérés pour des questions de droits. Il va alors falloir, sur le serveur distant, configurer sudo

Sur le serveur distant, si sudo n'est pas installé 

    sudo apt install sudo 

Il faut configurer sudo 

    sudo visudo

Nous allons rajouter dans le fichier la ligne suivante  

    user ALL= NOPASSWD:/usr/bin/rsync 

Puis on va utiliser l'option "--rsync-path" pour préciser à rsync de démarrer avec l'option sudo 

    rsync -av --progress --stats --human-readable --rsync-path="sudo rsync" -e "ssh -p xxxx"  useronremoteserver@remoteserver:/data/to/sync /archive/data/

### Transfert volumineux

Lors du transfert de grandes quantités de données, il est recommandé d'exécuter la commande rsync dans une session d'écran ou d'utiliser l'option -P :

    rsync -a -P remote_user@remote_host_or_ip:/opt/media/ /opt/media/

## Rsync serveur + locale 

*Sauvegarde des serveurs distants via ssh + rsync et locale via systemd timer*

### Description

Il faut disposer d'un serveur debian que nous appellerons **serveur source** qui se connectera (via *ssh* + clés) et exécutera, via le planificateur (cron), des sauvegardes de **serveurs distants** (via *rsync*) dans un dossier de sauvegarde du **serveur source** 

### Serveur distant

**Les opérations suivantes sont à faire sur tous les serveurs distants**

#### Vérifier ou installer rsync

	sudo apt install rsync # debian

#### Utilisateur et dossier backupuser

Ajout utilisateur de **backupuser** dans le groupe **users** ,qui ne peut exécuter que **rsync** ,et copier la clé publique du "serveur source"

	sudo useradd -g users --system --shell /bin/bash --home-dir /home/backupuser --create-home backupuser  

Se connecter en *backupuser* et accéder au dossier 

	sudo su backupuser
	cd /home/backupuser

Création dossier .ssh

	mkdir .ssh

#### Ajout clé publique ssh du "serveur source" 

dans le fichier **authorized_keys** du nouvel utilisateur   

    nano .ssh/authorized_keys

coller le contenu du "fichier local"  

#### Script rsync-wrapper.sh

Création script bash **rsync-wrapper.sh**

	nano rsync-wrapper.sh

Contenu du script

```
#!/bin/sh
 
date > /home/backupuser/backuplog
#echo $@ >> /home/backupuser/backuplog
/usr/bin/sudo /usr/bin/rsync "$@";
```

Droits en exécution

	chmod +x rsync-wrapper.sh

fin session backupuser

	exit                                                # fin session backupuser

#### Autoriser utilisateur à exécuter rsync uniquement

Autoriser *backupuser* à exécuter **rsync** en root sans mot de passe, ajouter au fichier sudoers

	sudo -s
	echo "backupuser ALL=NOPASSWD:/usr/bin/rsync" >> /etc/sudoers
	exit

### Serveur source

#### Utilisateur et dossier backupuser

Création utilisateur **backupuser** dans le groupe **users** et d'un jeu de clé **ssh**  

	sudo useradd -g users --system --shell /bin/bash --home-dir /home/backupuser --create-home backupuser  

Se connecter en *backupuser* et accéder au dossier 

	sudo su backupuser
	cd /home/backupuser

Création dossier .ssh

	mkdir .ssh

Création clé (ed25519 SHA512)

	ssh-keygen -t ed25519 -f .ssh/backup_key_ed25519    # Accepter les valeurs par défaut

#### Sauvegarde complète (sauvegarde.sh)

*Ce script permet la connexion sur un serveur distant via ssh et la sauvegarde complète via rsync*

Fichiers à exclure de la sauvegarde : `"dev/*","proc/*","sys/*","tmp/*","run/*","mnt/*","media/*","lost+found"`

Script *sauvegarde.sh* exécuté sur le **serveur source**

	nano sauvegarde.sh

```
#!/bin/bash

# -a Archive mode (keep file permissions etc...)
# hôtes distants
echo $(date) "Sauvegarde hôte distant yanfi.net"  >> /srv/sauvegarde/savdistant.log
/usr/bin/rsync -aev \
    --delete \
    --rsync-path=/home/backupuser/rsync-wrapper.sh \
    --exclude={"dev/*","proc/*","sys/*","tmp/*","run/*","mnt/*","media/*","lost+found"} \
    --rsh="/usr/bin/ssh -p 49022 -i /home/backupuser/.ssh/backup_key_ed25519" backupuser@yanfi.net:/ /srv/sauvegarde/yanfi &>> /srv/sauvegarde/savdistant.log
echo $(date) "Fin sauvegarde hôte distant yanfi.net"  >> /srv/sauvegarde/savdistant.log

#envoi des logs du jour par mail
# grep "$(date +"%d %B %Y")" /srv/sauvegarde/savdistant.log |mail -s "Sauvegarde du $(date +"%d %B %Y")" $desti
```

Les droits en exécution

	chmod +x sauvegarde.sh

fin session backupuser

	exit                                                # fin session backupuser

#### Création dossier de sauvegarde

En mode su

    sudo mkdir -p /srv/sauvegarde

Donner les droits utilisateur backupuser au dossier de sauvegarde

    sudo chown backupuser:users /srv/sauvegarde

#### Autoriser utilisateur à exécuter rsync uniquement

Autoriser *backupuser* à exécuter **rsync** en root sans mot de passe, ajouter au fichier sudoers

	sudo -s
	echo "backupuser ALL=NOPASSWD:/usr/bin/rsync" >> /etc/sudoers
	exit


##### Copie clé publique sur serveur distant 

Copie de la clé publique (.ssh/backup_key_ed25519.pub) dans le fichier **.ssh/authorized_keys** du serveur distant à sauvegarder :

	sudo cat /home/backupuser/.ssh/backup_key_ed25519.pub | ssh backupuser@serveur-distant 'cat >> .ssh/authorized_keys'

>***ATTENTION !!! ceci n'est possible que si l'utilisateur backupuser existe sur le serveur distant***

Dans le cas contraire, il faut copier la clé publique dans un fichier local (on suppose que l'on est connecté au "serveur source" pat ssh) 

    sudo cat /home/backupuser/.ssh/backup_key_ed25519.pub  # Sélectionner tout le fichier et faire un Shift+Ctrl+v pour le copier dans le presse papier

Ouvrir un "fichier local" , puis y copier le contenu du presse-papier


#### Rotation du fichier log (facultatif)

Rotation avec compression

  sudo nano /etc/logrotate.d/sauvegarde

```
/srv/sauvegarde/savdistant.log {
        rotate 6
        monthly
        compress
        missingok
}
```

* surveille le fichier savdistant.log et génère une rotation une fois par mois - c'est l' "intervalle de rotation".
* 'rotate 6' signifie qu'à chaque intervalle, on conserve 6 semaines de journalisation.
* Les fichiers de logs peuvent sont compressés au format gzip en spécifiant 'compress' 
* 'missingok' permet au processus de ne pas s'arrêter à chaque erreur et de poursuivre avec le fichier de log suivant.

#### Ordonnancement (cron)

Sauvegarde une fois par jour à 3h15 du matin

sudo crontab -e

```
# tous les jours à 3h15
15 3 * * * /home/backupuser/sauvegarde.sh
```

#### Connexion ssh manuelle pour valider le distant

Le serveur source doit effectuer une connexion ssh manuelle pour valider le distant 

	sudo su backupuser
	cd /home/backupuser
	#ssh -i .ssh/backup_key_ed25519 backupuser@serveur-distant # si le port du distant est différent de celui par défaut ,il faut le déclarer par -p N°Port.

retour sur utilisateur précédent

	exit                   # retour sur utilisateur précédent

### Sauvegarde Locale Yunohost

#### Outil yunohost backup

La sauvegarde **Yunohost** sera générée dans le répertoire **/home/yunohost.backup/archives** (certaines applications ne sont pas sauvegardées). 
L'archive est au format **AAAAMMJJ-HHMMSS.tar.gz** ainsi que le fichier info associé **AAAAMMJJ-HHMMSS.info.json**  
La sauvegarde est LOCALE , il faut copier le contenu sur une source externe (USB formatée en ext4 , via le réseau par rsync , etc...)  
Donner les droits d'accès aux archives au groupe de l'utilisateur $USER :

    sudo usermod -a -G users $USER
    sudo chown -Rv root:users /home/yunohost.backup

#### Yunohost bash savyuno.sh

Se connecter en *backupuser* et accéder au dossier 

	sudo su backupuser
	cd /home/backupuser

Créer un bash pour la sauvegarde yunohost

	nano  savyuno.sh

```
#!/bin/bash

DOMAINE="domaine_tld"
SAVDIR="/srv/sauvegarde"
if [ -f "$SAVDIR/$DOMAINE.info.json" ];then
 rm $SAVDIR/{$DOMAINE.info.json,$DOMAINE.tar.gz}
fi
/usr/bin/yunohost backup create -n $DOMAINE 
mv /home/yunohost.backup/archives/{$DOMAINE.info.json,$DOMAINE.tar.gz} $SAVDIR/
```

> Remplacer domaine_tld par votre nom de backup (!!! remplacer les "." par des "_" dans les noms)

droits en exécution

	chmod +x  savyuno.sh

retour sur utilisateur précédent

	exit                   # retour sur utilisateur précédent

Vérifier si le répertoire de sauvegarde **/srv/sauvegarde** existe

#### sauvegarde des bases MariaDB/Mysql (backupmysql.sh)

Se connecter en *backupuser* et accéder au dossier 

	sudo su backupuser
	cd /home/backupuser

Créer un bash pour la sauvegarde des bases

	nano  backupmysql.sh

```
#!/bin/bash

# Configuration de base: datestamp e.g. YYYYMMDD
DATE=$(date +"%Y%m%d")

# Dossier où sauvegarder les backups (créez le d'abord!)
BACKUP_DIR="/srv/sauvegarde/mysql"

# Identifiants MySQL
MYSQL_USER="root"
MYSQL_PASSWORD=$(cat /etc/yunohost/mysql)
GZIP="$(which gzip)"

# Commandes MySQL (aucune raison de modifier ceci)
MYSQL="$(which mysql)"
MYSQLDUMP="$(which mysqldump)"

# Bases de données MySQL à ignorer
SKIPDATABASES="Database|information_schema|performance_schema|mysql"

# Nombre de jours à garder les dossiers (seront effacés après X jours)
RETENTION=7

# ---- NE RIEN MODIFIER SOUS CETTE LIGNE ------------------------------------------
#
# Create a new directory into backup directory location for this date
mkdir -p $BACKUP_DIR/$DATE

# Retrieve a list of all databases
databases=`$MYSQL -u$MYSQL_USER -p$MYSQL_PASSWORD -e "SHOW DATABASES;" | grep -Ev "($SKIPDATABASES)"`

# Dump the databases in seperate names and gzip the .sql file
for db in $databases; do
echo $db
$MYSQLDUMP --force --opt --user=$MYSQL_USER -p$MYSQL_PASSWORD --skip-lock-tables --events --databases $db | $GZIP > "$BACKUP_DIR/$DATE/$db.sql.gz"
done

# Remove files older than X days

find $BACKUP_DIR/* -mtime +$RETENTION -delete

```

droits en exécution

	chmod +x  backupmysql.sh


retour sur utilisateur précédent

	exit                   # retour sur utilisateur précédent

Créer le répertoire de sauvegarde des bases

	sudo mkdir -p /srv/sauvegarde/mysql



### Ordonnancement des tâches (cron)

Sauvegarde yunohost tous les jours à 2h15  
Sauvegarde des bases mysql tous les jours à 2h30  
Vérification validité des certificats SSL 1 fois par semaine à 2h30 et regénération auto si nécessaire (NE PAS AJOUTER si serveur Yunohost)

le fichier  crontab

	sudo -s
	crontab -e


```
# m h  dom mon dow   command
30 1 * * * /home/backupuser/savyuno.sh
30 2 * * * /home/backupuser/backupmysql.sh
30 2 * * 1 /usr/local/sbin/le-renew-webroot >> /var/log/le-renewal.log
```

**ATTENTION!!! La sauvegarde de yunohost (savyuno.sh) est très longue en temps**

### Sauvegarde locale via systemd timer

Le fonctionnement de **systemd** impose cependant d'avoir deux fichiers :  
**service**, qui contient la définition du programme  
**timer**, qui dit "quand" le lancer.  

>Ils doivent porter le même nom et se situer dans **/etc/systemd/system/**.

#### Création service et timer

Si vous gérez déjà vos services via systemd, vous avez déjà utilisé des “unit” systemd de type “service”.  
Ces “unit” permettent de définir un process et son mode d’éxécution.  
Pour implémenter un “timer” sous systemd, il va nous falloir un fichier “service”.

Pour notre tâche à planifier, nous allons avoir au final 3 fichiers :

    Le script à exécuter
    Le fichier “service” qui va dire quel script exécuter
    Le fichier “timer” qui va indiquer quand il doit être exécuté.

>A noter que par convention, les fichiers service et timer doivent avoir le même nom

Nous devons exécuter ,une fois par jour , un script de sauvegarde **/home/yannick/scripts/savarch** sur un ordinateur qui n'est pas sous tension 24/24h.

Pour le fichier service **/etc/systemd/system/savarch.service**, une base simple

```
[Unit]
Description=Sauvegarde jour

[Service]
Type=simple
ExecStart=/bin/bash /home/yannick/scripts/savarch
StandardError=journal
Type=oneshot
```

Je fournis une description à mon service, indique que c’est un process de type simple, le chemin vers mon script et je rajoute que le flux d’erreur est envoyé dans le journal.Il ne faut pas de section [Install] car le script va être piloté par le fichier timer.  
La ligne `Type=oneshot` est importante, c'est elle qui dit à systemd de ne pas relancer le service en boucle.

Le fichier “timer” **/etc/systemd/system/savarch.timer**  

```
[Unit]
Description=Sauvegarde jour

[Timer]
# lisez le man systemd.timer(5) pour tout ce qui est disponible
OnCalendar=daily
# Autoriser la persistence entre les reboot
Persistent=true
Unit=savarch.service

[Install]
WantedBy=timers.target
```

* **OnCalendar** permet d’indiquer l’occurrence et la fréquence d’exécution du script. Il y a les abréviations classiques ("minutely", "hourly", "daily", "monthly", "weekly", "yearly", "quarterly", "semiannually", etc) mais vous pouvez avoir des choses plus complexes comme "Mon,Tue *-*-01..04 12:00:00" - voir **systemd.time**
* **Persistent** va forcer l’exécution du script si la dernière exécution a été manquée suite à un reboot de serveur ou autre événement.
* **Install** va créer la dépendance pour que votre “timer” soit bien exécuté et pris en compte par systemd.

#### Activation et démarrage du timer

Il est possible de tester le service avec un simple `systemctl start savarch.service`, de regarder les logs avec `journalctl -u savarch.service`.

Ensuite, pour qu'il soit actif, il faut prévenir systemd

    sudo systemctl enable savarch.timer
    sudo systemctl start savarch.timer

#### Gestion et suivi d’un timer

Pour voir la liste des “timers” actifs et la date de leur dernière et prochaine exécution

    systemctl list-timers

```
NEXT                         LEFT     LAST                         PASSED    UNIT                         ACTIVATES
Fri 2018-03-02 00:00:00 CET  15h left Thu 2018-03-01 07:49:43 CET  56min ago logrotate.timer              logrotate.service
Fri 2018-03-02 00:00:00 CET  15h left Thu 2018-03-01 07:49:43 CET  56min ago man-db.timer                 man-db.service
Fri 2018-03-02 00:00:00 CET  15h left Thu 2018-03-01 07:49:43 CET  56min ago savarch.timer                savarch.service
Fri 2018-03-02 00:00:00 CET  15h left Thu 2018-03-01 07:49:43 CET  56min ago shadow.timer                 shadow.service
Fri 2018-03-02 00:00:00 CET  15h left Thu 2018-03-01 07:49:43 CET  56min ago updatedb.timer               updatedb.service
Fri 2018-03-02 08:04:45 CET  23h left Thu 2018-03-01 08:04:45 CET  41min ago systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service
```

et accéder aux logs de vos “timers” :

    journalctl -u savarch.service

```
...
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:15 Départ sauvegarde
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:15 synchronisation source partagée /home/yannick/media/Musique cible locale /home/yannick/media/savlocal/Musique
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:21 synchronisation partagée /home/yannick/media/devel cible locale /home/yannick/media/savlocal/devel
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:33 synchronisation partagée /home/yannick/media/BiblioCalibre cible locale /home/yannick/media/savlocal/BiblioCalibre
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:41 SAUVEGARDE /home/yannick -> /home/yannick/media/savlocal/yannick-pc/yannick
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:42 SAUVEGARDE /home/yannick/media/dplus -> /home/yannick/media/savlocal/dplus
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:47 SAUVEGARDE /home/yannick/media/yanspm-md -> /home/yannick/media/savlocal/yanspm-md
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:47 FIN sauvegarde
mars 01 08:43:47 yannick-pc systemd[1]: Started Sauvegarde jour.
```

En cas de modification du **.timer** ou du **.service**, ne pas oublier de faire un **systemctl daemon-reload** pour que la version actualisée de vos fichiers soit prise en compte par systemd.


## Exemples de script crontab

### Rsync , transfert des sauvegardes borg xoyaz.xyz &rarr; xoyize.xyz

*Pour les serveurs yanspm.com yanfi.net et cinay.xyz ,les sauvegardes sont effectuées en utilisant le logiciel borg ,vers le serveur de backup xoyaz.xyz et on utilise rsync pour les transférer localement*

**connexion avec clé**  
<u>sur le serveur appelant srvxo (xoyize.xyz - 192.168.0.45)</u>
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) nommé **xoyaz_ed25519** pour une liaison SSH avec le serveur KVM.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/xoyaz_ed25519

Ajouter la clé publique au fichier **~/.ssh/authorized_keys** du serveur de backup xoyaz.xyz    

Test connexion

    ssh -p 55036 -i /home/admbust/.ssh/xoyaz_ed25519 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null usernl@xoyaz.xyz

Créer la commande rsync avec son fichier d'exclusion

	nano .ssh/exclude-list.txt

```
.bash_logout
.bashrc
.profile
.ssh
```

Lancer la commande rsync dans une fenêtre terminal tmux en mode su

	tmux
	sudo -s
	rsync -avz --progress --stats --human-readable --exclude-from '/home/admbust/.ssh/exclude-list.txt' --rsync-path="sudo rsync" -e "ssh -p 55036 -i /home/admbust/.ssh/xoyaz_ed25519 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"  usernl@xoyaz.xyz:/srv/data/borg-backups/* /srv/data/borg-backups/

Paramétrer pour une exécution chaque jour à 2h30

    sudo crontab -e

```
# Synchro Backup (contient les dossiers de sauvegardes borg) avec xoyize 
30 02 * * * rsync -avz --progress --stats --human-readable --exclude-from '/home/admbust/.ssh/exclude-list.txt' --rsync-path="sudo rsync" -e "ssh -p 55036 -i /home/admbust/.ssh/xoyaz_ed25519 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"  usernl@xoyaz.xyz:/srv/data/borg-backups/* /srv/data/borg-backups/ ; if [ $? -eq 0 ]; then echo "Rsync usernl@xoyaz.xyz:/srv/data/borg-backups/* /srv/data/borg-backups/ : Synchronisation OK" | systemd-cat -t rsyncborg -p info ; else echo "Rsync usernl@xoyaz.xyz:/srv/data/borg-backups/* /srv/data/borg-backups/ :Erreur Synchronisation" | systemd-cat -t rsyncborg -p emerg ; fi

```

On redirige le résultat de la commande ($?) vers le journal avec **systemd-cat**   
`if [ $? -eq 0 ]; then echo "Rsync usernl@xoyaz.xyz:/srv/data/borg-backups/* /srv/data/borg-backups/ : Synchronisation OK" | systemd-cat -t rsyncborg -p info ; else echo "Rsync usernl@xoyaz.xyz:/srv/data/borg-backups/* /srv/data/borg-backups/ :Erreur Synchronisation" | systemd-cat -t rsyncborg -p emerg ; fi`  

la ligne de commande décortiquée:

```
if [ $? -eq 0 ]
 then
  echo "Rsync usernl@xoyaz.xyz:/srv/data/borg-backups/* /srv/data/borg-backups/ : Synchronisation OK" | systemd-cat -t rsyncborg -p info
 else
  echo "Rsync usernl@xoyaz.xyz:/srv/data/borg-backups/* /srv/data/borg-backups/ :Erreur Synchronisation" | systemd-cat -t rsyncborg -p emerg
fi
```
 
On peut lire le journal avec l'identifiant *rsyncborg* 

    journalctl -t rsyncborg

```
-- Logs begin at Thu 2019-08-29 11:07:55 CEST, end at Tue 2019-09-17 17:50:32 CEST. --
sept. 17 17:50:32 srvxo rsyncborg[11728]: Rsync usernl@xoyaz.xyz:/srv/data/borg-backups/* /srv/data/borg-backups/ : Synchronisation OK
```

### Sauvegarde dossiers Musique Calibre &rarr; xoyaz.xyz (backup)

Ajouter  au lanceur de tâches programmées

    sudo crontab -e

```
# Sauvegarde de dossiers /srv/data/{Musique,CalibreTechnique} sur serveur distant backup (nl) xoyaz.xyz
# Synchro Musique avec Backup
00 02 * * * rsync -avz --progress --stats --human-readable --exclude-from '/home/admbust/.ssh/exclude-list.txt' --rsync-path="sudo rsync" -e "ssh -p 55036 -i /home/admbust/.ssh/xoyaz_ed25519 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"  /srv/data/Musique/* usernl@xoyaz.xyz:/home/usernl/backup/musique/ 
# Synchro CalibreTechnique avec Backup
10 02 * * * rsync -avz --progress --stats --human-readable --exclude-from '/home/admbust/.ssh/exclude-list.txt' --rsync-path="sudo rsync" -e "ssh -p 55036 -i /home/admbust/.ssh/xoyaz_ed25519 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"  /srv/data/CalibreTechnique/* usernl@xoyaz.xyz:/home/usernl/backup/CalibreTechnique/
```




