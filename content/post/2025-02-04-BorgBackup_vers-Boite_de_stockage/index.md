+++
title = 'BorgBackup vers boîte de stockage BX11 Hetzner - sftp'
date = 2025-02-04 00:00:00 +0100
categories = borgbackup yunohost
+++
*Sauvegarder via borg des serveurs VPS sur une "boîte de stockage" BX11 Hetzner*

Stockage disponible : 	1 TB  
Trafic : illimité `*`  
Emplacement :FSN1 (Falkenstein) Allemagne  
Instantanés 10  
à partir de 3.81 € par mois + 0,00 € de frais d'installation unique  
Prix incluant la TVA de 19 %.  

Les boîtes de stockage vous offrent un stockage en ligne sûr et pratique pour vos données.
{: .prompt-info }


## BX11 - Caractéristiques de base

Après la souscription, accéder à la boîte de stockage <https://robot.hetzner.com/storage>  
![bx11](bx11-001.png){:width="600"}  
Donner un nom : **bx11-yann**  

Détails, double-clic sur **u326239**, activer **SSH** et Reset password  
![bx11](bx11-001a.png){:width="600"}  

## Authentification par clé SSH

**Authentification par clé SSH pour Backup Spaces et Storage Boxes**

*Si vous utilisez SCP, SFTP, rsync ou BorgBackup, vous pouvez vous connecter en utilisant l'authentification par clé SSH sans saisir de mot de passe.*

`Remarque importante :`{: .prompt-warning } selon le port SSH du compte de sauvegarde/boîte de stockage que vous utilisez, vous devrez peut-être utiliser un format spécifique pour la clé SSH publique.

* Pour le port SSH 22 (SFTP et SCP uniquement), vous devez utiliser une clé publique SSH au format RFC4716.  
* Pour le port SSH 23 (SFTP, SCP, rsync et BorgBackup), vous devez utiliser une clé publique SSH commune au format OpenSSH.  

Si vous souhaitez utiliser les services sur les deux ports, vous devez stocker la clé publique SSH dans les deux formats.
{: .prompt-info }

L'utilisation d'une clé ed25519 n'est pas prise en charge sur le port SSH 22.
{: .prompt-warning }

Chaque **sous-compte** nécessite son propre fichier `authorized_keys`.

### Générer clé curve25519

Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec le serveur.  

```shell
ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/bx11-yann-ed25519
```

### Créer authorized_keys

Insérer les clés publiques SSH requises dans un nouveau fichier local `authorized_keys` nommé `storagebox_authorized_keys`

Pour SSH sur le port 23 (SCP, SFTP, Rsync et Borg Backup), ajoutez la clé publique SSH au format OpenSSH :

```shell
cat .ssh/bx11-yann-ed25519.pub >> storagebox_authorized_keys
```

Maintenant vous devez envoyer le fichier `authorized_keys` généré vers le compte Storage Box **bx11-yann**   
Créer le répertoire `.ssh` avec les droits de fichier `0700 (rwx------)` et créer le fichier `authorized_keys` avec les clés SSH publiques et les droits de fichier `0600 (rw-------)` 

Vous pouvez exécuter toutes ses opérations avec la commande suivante, il faut **saisir le mot de passe**

```shell
echo -e "mkdir .ssh \n chmod 700 .ssh \n put storagebox_authorized_keys .ssh/authorized_keys \n chmod 600 .ssh/authorized_keys" | sftp u326239@u326239.your-storagebox.de
```

Résultat de la commande

```
The authenticity of host 'u326239.your-storagebox.de (116.202.54.25)' can't be established.
RSA key fingerprint is SHA256:EMlfI8GsRIfpVkoW1H2u0zYVpFGKkIMKHFZIRkf2ioI.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:109: u277865.your-storagebox.de
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'u326239.your-storagebox.de' (RSA) to the list of known hosts.
u326239@u326239.your-storagebox.de's password: 
Connected to u326239.your-storagebox.de.
sftp> mkdir .ssh 
sftp>  chmod 700 .ssh 
Changing mode on /.ssh
sftp>  put storagebox_authorized_keys .ssh/authorized_keys 
Uploading storagebox_authorized_keys to /.ssh/authorized_keys
storagebox_authorized_keys                                       100%   94     1.3KB/s   00:00    
sftp>  chmod 600 .ssh/authorized_keys
Changing mode on /.ssh/authorized_keys
```

### Modifier authorized_keys 

*Ajouter une autre clé publique au fichier authorized_keys de la boîte de stockage*

On télécharge le fichier existant `authorized_keys` de la boîte de stockage **bx11-yann** dans un fichier local nommé `storagebox_authorized_keys`

```shell
echo -e "get .ssh/authorized_keys storagebox_authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de
```

Ajout clé publique

```shell
cat >> storagebox_authorized_keys
```

Copier/coller la clé publique dans ce terminal  (commande : `cat /root/.ssh/id_borg_ed25519.pub`) de la machine à sauvegarder **Yunohost xoyaz.xyz** ), et presser **[Ctrl]+[D]** pour valider.

On renvoie le fichier modifié `storagebox_authorized_keys` dans le fichier `authorized_keys` de la boîte de stockage **bx11-yann**

```shell
echo -e "put storagebox_authorized_keys .ssh/authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de
```

### Connexion SFTP sans mot de passe

En fonction du format de clé SSH que vous avez téléchargé, vous devriez pouvoir vous connecter sans mot de passe sur le port 23 

```shell
sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de
```

Résultat de la commande

```
The authenticity of host '[u326239.your-storagebox.de]:23 ([116.202.54.25]:23)' can't be established.
ED25519 key fingerprint is SHA256:XqONwb1S0zuj5A1CDxpOSuD2hnAArV1A3wKY7Z3sdgM.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:110: [u277865.your-storagebox.de]:23
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[u326239.your-storagebox.de]:23' (ED25519) to the list of known hosts.
Connected to u326239.your-storagebox.de.
sftp> 
```

Saisir *quit* pour sortir...

### Filezilla

Paramétrer filezilla pour une connexion vers la boîte de stockage 

![](bx20-yann01.png)  
Renseigner avant connexion

Connexion

![](bx20-yann02.png)  
Nos dossiers sont dans le /home de la boîte de stockage  

## Borg - Yunohost 

**Créer le répertoire pour le référentiel de sauvegarde dans la boîte de stockage**.  
Créer un dossier `backup/borg``    
Sous `backup/borg`, vous pouvez ensuite créer d'autres répertoires pour les autres serveurs à sauvegarder.

```shell
echo -e "mkdir backup \n mkdir backup/borg \n chmod 775 backup/borg" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de
```

Résultat de la commande

```
Connected to u326239.your-storagebox.de.
sftp> mkdir backup 
sftp>  mkdir backup/borg 
sftp>  chmod 775 backup/borg
Changing mode on /home/backup/borg
```

Exemple de dépôt Borg pour la sauvegarde du VPS xoyaz.xyz:

```shell
ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/xoyaz.xyz
```

**borg yunohost xoyize.xyz est installé et paramétré pour la boîte de stockag**e  
![](bx11-002.png){:width="600"}  
Dépôt : `ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/xoyaz_xyz`

*Ajouter la clé publique borg `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPuVXP+pUjvedC/htJmKXamAotLESDCRqU0MOoD7vqCA root@422x.l.time4vps.cloud` au fichier authorized_keys de la boîte de stockage*


### Modifier authorized_keys bx11

*Cette opération va se dérouler sur poste ayant accès à la boîte de stockage avec clés SSH*

On télécharge le fichier existant `authorized_keys` de la boîte de stockage **bx11-yann** dans un fichier local nommé `storagebox_authorized_keys`

```shell
echo -e "get .ssh/authorized_keys storagebox_authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de
```

Ajout clé publique

```shell
cat >> storagebox_authorized_keys
```

Copier/coller la clé publique dans ce terminal  (commande : `cat /root/.ssh/id_borg_ed25519.pub`) de la machine à sauvegarder **Yunohost xoyize.xyz** ), et presser **[Ctrl]+[D]** pour valider.

On renvoie le fichier modifié `storagebox_authorized_keys` dans le fichier `authorized_keys` de la boîte de stockage **bx11-yann**

```shell
echo -e "put storagebox_authorized_keys .ssh/authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de
```

### Tester la configuration borg

Tester la connexion SSH avec la clé privée borg située sous /root/.ssh

    sftp -P 23 -i /root/.ssh/id_borg__2_ed25519 u326239@u326239.your-storagebox.de

À cette étape, votre sauvegarde devrait se dérouler à l'heure prévue (tous les jours à 3h20). Notez que la première sauvegarde peut être très longue, car de nombreuses données doivent être copiées via ssh. Les sauvegardes suivantes sont incrémentielles : seules les données nouvellement générées depuis la dernière sauvegarde seront copiées.
{: .prompt-info }


Si vous voulez tester la configuration correcte de Borg Apps avant l'heure prévue, vous pouvez lancer une sauvegarde manuellement sur le serveur xoyaz.xyz (cx21) :

    systemctl start borg

Si tout démarre correctement, un message est envoyé  
![](bx11-003.png)

Visualiser les logs en cours sur un autre terminal

    journalctl -f -u borg

```
[...]
Nov 21 18:11:55 xoyaz.xyz sudo[264670]:     root : PWD=/ ; USER=root ; COMMAND=/usr/bin/test ! -f /etc/yunohost/apps/my_webapp__2/scripts/backup
Nov 21 18:11:55 xoyaz.xyz sudo[264670]: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=0)
Nov 21 18:11:55 xoyaz.xyz sudo[264670]: pam_unix(sudo:session): session closed for user root
Nov 21 18:11:55 xoyaz.xyz sudo[264672]:     root : PWD=/ ; USER=root ; COMMAND=/usr/bin/yunohost backup create -n auto_my_webapp__2 --method borg__2_app --apps my_webapp__2
Nov 21 18:11:55 xoyaz.xyz sudo[264672]: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=0)
Nov 21 18:12:10 xoyaz.xyz sudo[264672]: pam_unix(sudo:session): session closed for user root
Nov 21 18:12:10 xoyaz.xyz sudo[265126]:     root : PWD=/ ; USER=root ; COMMAND=/usr/bin/test ! -f /etc/yunohost/apps/nextcloud/scripts/backup
Nov 21 18:12:10 xoyaz.xyz sudo[265126]: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=0)
Nov 21 18:12:10 xoyaz.xyz sudo[265126]: pam_unix(sudo:session): session closed for user root
[...]
```

A la fin de la sauvegarde borg  
![](bx11-004.png)


Avec les apps borg un email est envoyé pour dire si la sauvegarde échoue ou si le repo distant n'a rien reçu. On peut toutefois analyser manuellement pour s'assurer que tout va bien de façon plus complète.

```bash
# Dépôt borg sur storage box : ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/<Nom du dépôt>  
# Dépôt borg yunohost : $(yunohost app setting $app repository)

# Lister les fichiers
app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " \
borg list "$(yunohost app setting $app repository)" | less

# Lister les exports de base de données
app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " \
borg list "$(yunohost app setting $app repository)" | grep "(db|dump)\.sql"

# Lister les fichiers de l'archive
app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " \
borg list "$(yunohost app setting $app repository)::ARCHIVE" | less

# Voir les infos de l'archive
app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " \
borg info "$(yunohost app setting $app repository)::ARCHIVE"

# Vérifier l'intégrité des données
app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " \
borg check "$(yunohost app setting $app repository)::ARCHIVE" --verify-data

# Supprimer une archive
BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " \
borg delete "$(yunohost app setting $app repository)::ARCHIVE"

```

### Vérification sauvegarde borg

Lister les fichiers

    app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " borg list --short "$(yunohost app setting $app repository)"

Résultat

```
_auto_conf-2021-10-04_13:56
_auto_data-2021-10-04_13:56
_auto_borg-2021-10-04_13:57
_auto_calibreweb-2021-10-04_13:58
_auto_gitea-2021-10-04_14:03
_auto_librespeed-2021-10-04_14:04
_auto_my_webapp-2021-10-04_14:04
_auto_my_webapp__2-2021-10-04_14:05
_auto_my_webapp__3-2021-10-04_14:05
_auto_nextcloud-2021-10-04_14:06
_auto_rainloop-2021-10-04_14:12
_auto_redirect-2021-10-04_14:13
_auto_redirect__2-2021-10-04_14:13
_auto_shaarli-2021-10-04_14:14
_auto_ttrss-2021-10-04_14:14
```

### Restaurer

Si on est dans le cas d'une migration ou d'une réinstallation, il faut réinstaller borg de la même façon. Si le repo est distant il faut changer la clé publique.

Lister les archives disponibles

```
app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " borg list "$(yunohost app setting $app repository)"
```

Créer les archives tar (une archive par app et partie de système)

```
app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " borg export-tar "$(yunohost app setting $app repository)::ARCHIVE" /home/yunohost/archives/ARCHIVE.tar
```

#### Exemple restauration shaarli

Liste des sauvegardes "shaarli"

    app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " borg list "$(yunohost app setting $app repository)" |grep "shaarli"

On veut restaurer l'application "shaarli" du 23 juillet (_auto_shaarli-2022-07-23_02:25)

```
app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 " borg export-tar "$(yunohost app setting $app repository)::_auto_shaarli-2022-07-23_02:25" /home/yunohost.backup/archives/_auto_shaarli-2022-07-23_02:25.tar.gz
```

Désinstaller l'application existante

    yunohost app remove shaarli

Puis restaurer l'archive de façon classique.

    yunohost backup restore _auto_shaarli-2022-07-23_02:25.tar.gz --apps

```
Info: Preparing archive for restoration...
Info: Restoring shaarli...
Info: [+...................] > Loading installation settings...
Info: [#++.................] > Validating restoration parameters...
Info: [###++...............] > Restoring the NGINX configuration...
Info: [#####++.............] > Recreating the dedicated system user...
Info: [#######++...........] > Restoring the app main directory...
Info: [#########+..........] > Restoring the PHP-FPM configuration...
Info: [##########++........] > Reinstalling dependencies...
Info: [############++......] > Restoring the logrotate configuration...
Info: [##############++....] > Restoring the Fail2Ban configuration...
Info: [################++..] > Reloading NGINX web server and PHP-FPM...
Info: [##################++] > Restoration completed for shaarli
Success! Restoration completed
apps: 
  shaarli: Success
system: 
```

#### Restaurer des grosses archives

Si l'espace disponible est inférieur au poids de votre archive, des données décompressées et des dépendances, vous devrez restaurer partie par partie, app par app.

Si restaurer app par app ne suffit pas OU si une archive est trop grosse, il peut être judicieux de générer une archive tar sans les "grosses" données d'une app comme si elle avait été générée avec l'[option BACKUP_CORE_ONLY](/backup/include_exclude_files#ne-pas-sauvegarder-les-grosses-quantites-de-donnees). Exemple avec Nextcloud:
```
app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " borg export-tar -e apps/nextcloud/backup/home/yunohost.app "$(yunohost app setting $app repository)::ARCHIVE" /home/yunohost/archives/ARCHIVE.tar
```

Il faudra ensuite extraire ces données directement avec borg
```
cd /home/yunohost.app/
app=borg; BORG_PASSPHRASE="$(yunohost app setting $app passphrase)" BORG_RSH="ssh -i /root/.ssh/id_${app}_ed25519 -oStrictHostKeyChecking=yes " borg extract "$(yunohost app setting $app repository)::ARCHIVE" apps/nextcloud/backup/home/yunohost.app/
mv apps/nextcloud/backup/home/yunohost.app/nextcloud ./
rm -r apps
```

Puis restaurer de façon classique

## Borg - Laptop Dell e6230

### Installer borg

Installer les paquets suivants

```
sudo pacman -S borg fuse python-llfuse
```

Passer en mode su 

Créer une clé SSH pour l’authentification borg

    ssh-keygen -t ed25519 -f /root/.ssh/id_borg_ed25519

Validez en appuyant sur la touche « Entrée » à toutes les questions

Vous devriez maintenant avoir une clé privée contenue dans le fichier `/root/.ssh/id_borg_ed25519`, et une clé publique contenue dans le fichier `/root/.ssh/id_borg_ed25519.pub`
La clé privée ne doit jamais être partagée.

### Clé publique borg

**Ajouter une autre clé publique** au fichier `authorized_keys` de la boîte de stockage  

<u>Depuis un poste ayant un accès à la boîte de stockage</u>  
On télécharge le fichier existant authorized_keys de la boîte de stockage bx11-yann dans un fichier local nommé `storagebox_authorized_keys`

    echo -e "get .ssh/authorized_keys storagebox_authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de

Ajout clé publique

    cat >> storagebox_authorized_keys

Copier/coller la clé publique dans ce terminal (commande : cat /root/.ssh/id_borg_ed25519.pub) du portable Lenovo e6230 ), et presser [Ctrl]+[D] pour valider.

On renvoie le fichier modifié storagebox_authorized_keys dans le fichier authorized_keys de la boîte de stockage bx11-yann

    echo -e "put storagebox_authorized_keys .ssh/authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de

### Tester connexion SSH

Tester la connexion SSH à la boîte de stockage

    sftp -P 23 -i /root/.ssh/id_borg_ed25519 u326239@u326239.your-storagebox.de

```
The authenticity of host '[u326239.your-storagebox.de]:23 ([167.235.97.31]:23)' can't be established.
ED25519 key fingerprint is SHA256:XqONwb1S0zuj5A1CDxpOSuD2hnAArV1A3wKY7Z3sdgM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[u326239.your-storagebox.de]:23' (ED25519) to the list of known hosts.
Connected to u326239.your-storagebox.de.
sftp> quit
```

### Initialisation Dépôt (repository)

**Dépôt (repository)**.Les dépôts de la boîte de stockage sont sous ./backup/borg/

on va maintenant travailler sur un dépôt distant à travers SSH, il va falloir fournir le chemin du dépôt sous la forme d’une URL avec un peu plus d’informations

    ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/e6230

phrase forte (passphrase)

Ajout de la phrase forte dans un fichier au dossier `/root/.borg`

```
mkdir -p /root/.borg
# ajout phrase
echo "<La phrase de passe forte>" > /root/.borg/rnmkcy_eu.passphrase
```

**Initialisation dépôt distant** 

```
export BORG_PASSPHRASE="$(cat /root/.borg/e6230.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY="$(cat /root/.borg/e6230.repository)"
borg init --encryption=repokey $BORG_REPOSITORY
```

### Script de sauvegarde

Créer un fichier d'exclusion `/root/.borg/e6230.exclusions`

```
/proc
/sys
/dev
/media
/mnt
/cdrom
/tmp
/run
/var/tmp
/var/run
/var/cache
lost+found
/home/yano/.cache
/home/yano/.keepassx
/home/yano/Téléchargements
/home/yano/Musique
/home/yano/Vidéos
/home/yano/media
/home/yano/scripts
/home/yano/sharenfs
```

Script à exécuter en mode su : `/root/.borg/borg-backup.sh`

```bash
#!/bin/sh
#
# Script de sauvegarde.
#
# Envoie les sauvegardes sur un serveur distant, via le programme Borg.
# Les sauvegardes sont chiffrées
#

flag="/var/tmp/$(basename -- $0).flag"
if [ -e "$flag" ] ;then
echo "flag : $flag"
  if [ "$(date +%F)" == "$(date +%F -r $flag)" ]; then
   # script déjà exécuté 1 fois aujourd'hui, on sort
   echo "script $0 déjà exécuté ce jour($(date +%F))"
   echo "script $0 déjà exécuté ce jour($(date +%F))" | systemd-cat -t sauvegardes -p info
	# Envoi notification
	# DISPLAY=:0 notify-send "$0" "déjà exécuté ce jour\n($flag)" -i /root/.borg/information.png -t 10000
    curl \
    -H "Title: DELL e6230 $0" \
    -H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
    -H prio:low \
    -H tags:information_source \
    -d "déjà exécuté ce jour ($flag)" \
    https://noti.rnmkcy.eu/yan_infos

   exit 0
  fi

fi

touch $flag

set -e
 
LOG_PATH=/var/log/borg-backup.log
 
export BORG_PASSPHRASE=$(cat /root/.borg/e6230.passphrase)
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
export BORG_RELOCATED_REPO_ACCESS_IS_OK=yes
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
BORG_REPOSITORY=$(cat /root/.borg/e6230.repository)

borg create \
-v --progress --stats \
--exclude-from /root/.borg/e6230.exclusions \
${BORG_REPOSITORY}::${BACKUP_DATE} \
/ \
>> ${LOG_PATH} 2>&1
 
# Nettoyage des anciens backups
# On conserve
# - une archive par jour les 7 derniers jours,
# - une archive par semaine pour les 4 dernières semaines,
# - une archive par mois pour les 6 derniers mois.
 
borg prune \
-v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 \
$BORG_REPOSITORY \
>> ${LOG_PATH} 2>&1

    curl \
    -H "Title: DELL e6230 $0" \
    -H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
    -H prio:low \
    -H tags:information_source \
    -d "Fin sauvegarde borgbackup" \
    https://noti.rnmkcy.eu/yan_infos

```

### Exécution script au boot après 3 min

On va utiliser la fonction timer de systemd en créant 2 fichier de même nom et suffixes différents

Le service

    /etc/systemd/system/run-script-with-delay.service

```
[Unit]
Description=Lancer script au démarrage

[Service]
Type=oneshot
ExecStart=/root/.borg/borg-backup.sh
TimeoutStartSec=0
```

Le timer

    /etc/systemd/system/run-script-with-delay.timer

```
[Unit]
Description="Lancer script au démarrage après 3min"

[Timer]
OnBootSec=3min

[Install]
WantedBy=default.target
```

Rafraîchir la configuration systemd

    sudo systemctl daemon-reload

Activer le timer

    sudo systemctl enable run-script-with-delay.timer

## BorgBackup - xoyize.net

*Sauvegarde borg dans une boîte de stockage*

Passer en mode su 

### Installer borg

Installer borg

```
apt install borgbackup
```

Créer une clé SSH pour l’authentification borg

    ssh-keygen -t ed25519 -f /root/.ssh/id_borg_ed25519

Validez en appuyant sur la touche « Entrée » à toutes les questions

Vous devriez maintenant avoir une clé privée contenue dans le fichier `/root/.ssh/id_borg_ed25519`, et une clé publique contenue dans le fichier `/root/.ssh/id_borg_ed25519.pub`
La clé privée ne doit jamais être partagée.

### Clé publique borg

**Ajouter une autre clé publique** au fichier `authorized_keys` de la boîte de stockage  

<u>Depuis un poste ayant un accès à la boîte de stockage</u>  
On télécharge le fichier existant authorized_keys de la boîte de stockage bx11-yann dans un fichier local nommé `storagebox_authorized_keys`

    echo -e "get .ssh/authorized_keys storagebox_authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de

Ajout clé publique

    cat >> storagebox_authorized_keys

Copier/coller la clé publique dans ce terminal (commande : `cat /root/.ssh/id_borg_ed25519.pub`) du VPS Contabo debian 12 xoyize.net ), et presser [Ctrl]+[D] pour valider.

On renvoie le fichier modifié storagebox_authorized_keys dans le fichier authorized_keys de la boîte de stockage bx11-yann

    echo -e "put storagebox_authorized_keys .ssh/authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de

### Tester connexion SSH depuis le VPS

Tester la connexion SSH à la boîte de stockage

    sftp -P 23 -i /root/.ssh/id_borg_ed25519 u326239@u326239.your-storagebox.de

```
The authenticity of host '[u326239.your-storagebox.de]:23 ([2a01:4f8:261:59d3::2]:23)' can't be established.
ED25519 key fingerprint is SHA256:XqONwb1S0zuj5A1CDxpOSuD2hnAArV1A3wKY7Z3sdgM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[u326239.your-storagebox.de]:23' (ED25519) to the list of known hosts.
Connected to u326239.your-storagebox.de.
sftp> quit
```

Création dossier pour les paramètres de borg

    mkdir -p /root/.borg

### Initialisation Dépôt (repository)

**Dépôt (repository)**.Les dépôts de la boîte de stockage sont sous ./backup/borg/

on va maintenant travailler sur un dépôt distant à travers SSH, il va falloir fournir le chemin du dépôt sous la forme d’une URL

```
# dépôt distant
echo "ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/xoyize.net" > /root/.borg/xoyize.net.repository
```

phrase forte (passphrase)

Ajout de la phrase forte dans un fichier au dossier `/root/.borg`

```
# ajout phrase
echo "<La phrase de passe forte>" > /root/.borg/xoyize.net.passphrase
```

**Initialisation dépôt distant** 

```
export BORG_PASSPHRASE="$(cat /root/.borg/xoyize.net.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY="$(cat /root/.borg/xoyize.net.repository)"
borg init --encryption=repokey $BORG_REPOSITORY
```

Si tout se passe bien, vous aurez le message suivant

```
By default repositories initialized with this version will produce security
errors if written to with an older version (up to and including Borg 1.0.8).

If you want to use these older versions, you can disable the check by running:
borg upgrade --disable-tam ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/xoyize.net

See https://borgbackup.readthedocs.io/en/stable/changes.html#pre-1-0-9-manifest-spoofing-vulnerability for details about the security implications.

IMPORTANT: you will need both KEY AND PASSPHRASE to access this repo!
If you used a repokey mode, the key is stored in the repo, but you should back it up separately.
Use "borg key export" to export the key, optionally in printable format.
Write down the passphrase. Store both at safe place(s).
```

### Script de sauvegarde

Créer un fichier d'exclusion `/root/.borg/exclusions`

```
/proc
/sys
/dev
/media
/mnt
/tmp
/run
/var/tmp
/var/run
/var/cache
lost+found
```

Script à exécuter en mode su : `/root/.borg/borg-backup.sh`

<details>
<summary><b>Etendre Réduire</b></summary>
{% highlight text %}  
#!/bin/bash
#
# Script de sauvegarde.
#
# Envoie les sauvegardes sur un serveur distant, via le programme Borg.
# Les sauvegardes sont chiffrées
#

flag="/var/tmp/$(basename -- $0).flag"
if [ -f "$flag" ]; then
 echo "flag : $flag"
 echo "Comparer : $(date +%F) avec $(date +%F -r $flag)"
  if [[ "$(date +%F)" -eq "$(date +%F -r $flag)" ]]; then
   # script déjà exécuté 1 fois aujourd'hui, on sort
   echo "script $0 déjà exécuté ce jour($(date +%F))"
   echo "script $0 déjà exécuté ce jour($(date +%F))" | systemd-cat -t sauvegardes -p info
	# Envoi notification
	# DISPLAY=:0 notify-send "$0" "déjà exécuté ce jour\n($flag)" -i /root/.borg/information.png -t 10000
    curl \
    -H "Title: $0" \
    -H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
    -H prio:low \
    -H tags:information_source \
    -d "déjà exécuté ce jour ($flag)" \
    https://noti.rnmkcy.eu/yan_infos

   exit 0
  fi
else
  echo "Création fichier $flag"
  touch "$flag"
fi

set -e
 
LOG_PATH=/var/log/borg-backup.log
 
export BORG_PASSPHRASE=$(cat /root/.borg/e6230.passphrase)
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
export BORG_RELOCATED_REPO_ACCESS_IS_OK=yes
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
BORG_REPOSITORY=$(cat /root/.borg/e6230.repository)

borg create \
-v --progress --stats \
--exclude-from /root/.borg/e6230.exclusions \
${BORG_REPOSITORY}::${BACKUP_DATE} \
/ \
>> ${LOG_PATH} 2>&1
 
# Nettoyage des anciens backups
# On conserve
# - une archive par jour les 7 derniers jours,
# - une archive par semaine pour les 4 dernières semaines,
# - une archive par mois pour les 6 derniers mois.
 
borg prune \
-v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 \
$BORG_REPOSITORY \
>> ${LOG_PATH} 2>&1

    curl \
    -H "Title: $0" \
    -H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
    -H prio:low \
    -H tags:information_source \
    -d "Fin sauvegarde borgbackup" \
    https://noti.rnmkcy.eu/yan_infos

{% endhighlight %}
</details>



Le rendre exécutable

    chmod +x /root/.borg/borg-backup.sh

### Exécution script tous les jours

On va utiliser cron

    crontab -e

Ajouter la ligne suivante

```
15 2 * * * /root/.borg/borg-backup.sh
```

Script exécuté tous les jours à 2h15

## SFTP

### Taille du dossier

Pour afficher la taille du dossier via SFTP

```bash
echo -e "df -h" |sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de
```

![](sftp-df.png)

### Commandes

*SFTP ou Secure File Transfer Protocol est un utilitaire de transfert de fichiers à distance sécurisé basé sur le protocole de transfert de fichiers FTP. [(15 Exemples de commandes SFTP sous Linux)](https://geekflare.com/fr/sftp-command-examples/)*

Pour télécharger un fichier à partir d’un serveur distant, utilisez la syntaxe de commande suivante :
`$ sftp {utilisateur}@{hôte distant}:{nom-de-fichier-distant} {nom-de-fichier-local}`

Connexion serveur SFTP avec clef  
`sftp -P {Port} -i {nom-de-fichier-clef} {utilisateur}@{hôte distant}`

Pour obtenir de l’aide sur les commandes disponibles et la syntaxe de SFTP, utilisez `‘?‘` ou `‘help‘`

```shell
sftp> ?
Commandes disponibles :
bye Quitter sftp
cd chemin Changer le répertoire distant en "chemin"
chgrp [-h] grp path Change le groupe du fichier 'path' en 'grp'
chmod [-h] mode chemin Changer les permissions du fichier 'chemin' en 'mode'
chown [-h] own chemin Changer le propriétaire du fichier 'chemin' en 'own'
df [-hi] <x><x>[chemin]</x></x> Affiche les statistiques du répertoire ou du système de
répertoire ou du système de fichiers contenant 'path'
exit Quitter sftp
get [-afpR] remote <x>[local]</x> Télécharger un fichier
help Affiche ce texte d'aide
lcd path Modifier le répertoire local en 'path'
lls [ls-options <x><x>[path]]</x></x> Affiche la liste du répertoire local
lmkdir path Créer un répertoire local
ln [-s] oldpath newpath Lier un fichier distant (-s pour un lien symbolique)
lpwd Imprimer le répertoire de travail local
ls [-1afhlnrSt] <x><x>[path]</x></x> Afficher la liste des répertoires distants
lumask umask Définir l'umask local à 'umask'
mkdir path Créer un répertoire distant
progress Basculer l'affichage de l'indicateur de progression
put [-afpR] local <x>[remote]</x> Télécharger un fichier
pwd Afficher le répertoire de travail distant
quit Quitter sftp
reget [-fpR] remote <x>[local]</x> Reprendre le téléchargement du fichier
rename oldpath newpath Renommer le fichier distant
reput [-fpR] local <x>[remote]</x> Reprendre le téléchargement du fichier
rm path Supprimer le fichier distant
rmdir path Supprimer le répertoire distant
symlink oldpath newpath Symlink fichier distant
version Affiche la version SFTP
!command Exécuter 'command' dans l'interpréteur de commandes local
! Echapper à l'interpréteur de commandes local
? Synonyme de help
sftp>
```

