+++
title = 'BorgBackup de serveur à serveur'
date = 2024-12-21 00:00:00 +0100
categories = borgbackup
+++
*BorgBackup (abrégé : Borg) est un programme de sauvegarde par déduplication. En option, il prend en charge la compression et le chiffrement authentifié.*

![](borg-logo.png)


## BorgBackup

### Conventions  

* srvhote &rarr; Serveur qui contiendra les sauvegardes BorgBackup
* srvclient &rarr; Serveur à sauvegarder

### A - Actions sur srvhote

#### srvhote - Installer BorgBackup

on passe en mode su  

    sudo -s

Installer borgbackup

    apt install borgbackup

#### srvhote - Créer utilisateur borg

Créer un utilisateur **borg** dédié aux sauvegardes par BorgBackup :

    mkdir -p /srv/data/
    useradd borg --create-home --home-dir /srv/data/borg-backups

Créer le dossier .ssh utilisateur "borg"

    sudo -u borg mkdir /srv/data/borg-backups/.ssh

Autoriser utilisateur **borg** à exécuter */usr/bin/borg* uniquement

    echo "borg ALL=NOPASSWD: /usr/bin/borg" >> /etc/sudoers

Créer le fichier authorized_keys

    sudo -u borg touch /srv/data/borg-backups/.ssh/authorized_keys

#### srvhote - Les clés publiques

*Chaque client voulant effectuer des sauvegardes borg devra fournir en premier lieu une clé publique* 


### B - Actions sur srvclient

#### srvclient - Installer BorgBackup

Installer borgbackup

    sudo apt install borgbackup

#### srvclient - Créer utilisateur borg

Créer un utilisateur borg (sans home) dédié aux sauvegardes par BorgBackup 

    useradd -M borg

Autoriser utilisateur **borg** à exécuter */usr/bin/borg* uniquement

    sudo -s
    echo "borg ALL=NOPASSWD: /usr/bin/borg" >> /etc/sudoers

#### srvclient - Clés ssh borg

En root,créer une **clé SSH pour l’authentification borg**  

    sudo -s
    ssh-keygen -t ed25519 -f /root/.ssh/id_borg_ed25519

Validez en appuyant sur la touche « Entrée » à toutes les questions

Vous devriez maintenant avoir une clé privée contenue dans le fichier `/root/.ssh/id_borg_ed25519`, et une clé publique contenue dans le fichier `/root/.ssh/id_borg_ed25519.pub`  
La clé privée ne doit jamais être partagée.

**1 -  Ajouter clé publique à srvhote**
 
Copier la clé publique de chacun des clients avec l’option command en ajout sur le fichier `/srv/data/borg-backups/.ssh/authorized_keys` du serveur   

**EXEMPLES**  

Ajout de la clé publique `id_borg_ed25519.pub` sans option 

```bash
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIws5uLxgF1UgGT8d/gfYueeEm5pFmRoXesbfD81QEpV root@yanfi.space' >> /srv/data/borg-backups/.ssh/authorized_keys
```

Ajout de la clé publique `id_borg_ed25519.pub` avec option

```bash
sudo -u borgbackup -s
# Avec quota : borg serve --storage-quota 5G
# Client yanfi.space
echo 'command="cd /srv/data/borg-backups/yanfi_space; borg serve --restrict-to-path /srv/data/borg-backups/yanfi_space",no-port-forwarding,no-x11-forwarding,no-agent-forwarding,no-pty,no-user-rc ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIws5uLxgF1UgGT8d/gfYueeEm5pFmRoXesbfD81QEpV root@yanfi.space' >> /srv/data/borg-backups/.ssh/authorized_keys
exit
```

L’option `command` permet de restreindre l’utilisation de la connexion SSH à une commande donnée, pour le client qui utilisera la clé publique associée.
{: .prompt-info }

**2 - Connexion ssh vers srvhote**

**AU PREMIER passage une question est posée , saisir oui ou yes (suivant language)**

    sudo -s
    ssh -p 55178 -i /root/.ssh/id_borg_ed25519 borg@45.145.166.178

```
The authenticity of host '[45.145.166.178]:55178 ([45.145.166.178]:55178)' can't be established.
ECDSA key fingerprint is SHA256:WdJQvWq72wgHST7vBNNifJvkDR6/p2uEdxsvBsVUttE.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[45.145.166.178]:55178' (ECDSA) to the list of known hosts.
Linux server32771 5.10.0-16-cloud-amd64 #1 SMP Debian 5.10.127-2 (2022-07-23) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
$ 
```

saisir `exit` pour sortir

#### srvclient - Créer une phrase de passe forte

**Créer une phrase de passe forte pour crypter les sauvegardes Borg** (sans espace vide)  
Vous pouvez utiliser [Diceware Password Generator](https://diceware.dmuth.org/) pour générer une phrase de passe forte  

**Ajout de la phrase forte dans un fichier** au dossier `/root/.passphrase`

```bash
sudo -s
mkdir -p /root/.passphrase
# ajout phrase
echo "<La phrase de passe forte>" > /root/.passphrase/ouestline_xyz.passphrase
```

Conserver cette phrase, elle sera demandé pour la création du dépôt borg
{: .prompt-info }

#### srvclient - Initialisation dépôt borg

Initialisation dépôt 

    sudo -s

```bash
export BORG_PASSPHRASE="`cat /root/.passphrase/ouestline_xyz.passphrase`"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://borg@45.145.166.178:55178/srv/data/borg-backups/ouestline_xyz
borg init --encryption=repokey-blake2 $BORG_REPOSITORY
```

Le résultat de la commande précédente

```
By default repositories initialized with this version will produce security
errors if written to with an older version (up to and including Borg 1.0.8).

If you want to use these older versions, you can disable the check by running:
borg upgrade --disable-tam ssh://borg@45.145.166.178:55178/srv/data/borg-backups/ouestline_xyz

See https://borgbackup.readthedocs.io/en/stable/changes.html#pre-1-0-9-manifest-spoofing-vulnerability for details about the security implications.

IMPORTANT: you will need both KEY AND PASSPHRASE to access this repo!
Use "borg key export" to export the key, optionally in printable format.
Write down the passphrase. Store both at safe place(s).
```

### Sauvegarde vers srvhote

**Générer une sauvegarde d'un dossier local vers le dépôt distant** pour test (facultatif)

    borg create ssh://borg@xoyize.xyz:55029/srv/ssd-two/borg-backups/cinay.eu::2019-01-11 /home/yanfi

Création d'un fichier **exclusions** qui contient toutes les exclusions de dossiers et fichiers , une exclusion par ligne (**/root/.exclusions**)

    sudo nano /root/.exclusions

```
/dev/*
/proc/*
/sys/*
/tmp/*
/run/*
/mnt/*
/media/*
lost+found
```

**Automatiser la procédure de sauvegarde pour srvclient**  
script de sauvegarde (notez l'usage de borg prune pour supprimer les archives trop anciennes)  

    nano /root/borg-backup.sh 


```bash
#!/bin/sh
#
# Script de sauvegarde.
#
# Envoie les sauvegardes sur un serveur distant, via le programme Borg.
# Les sauvegardes sont chiffrées
#
 
set -e
 
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
LOG_PATH=/var/log/borg-backup.log
 
export BORG_PASSPHRASE="`cat /root/.passphrase/ouestline_xyz.passphrase`"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://borg@45.145.166.178:55178/srv/data/borg-backups/ouestline_xyz
BORG_ARCHIVE=${BORG_REPOSITORY}::${BACKUP_DATE}
 
borg create \
-v --progress --stats --compression lzma,9 \
--exclude-caches --exclude-from /root/.exclusions \
$BORG_ARCHIVE \
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
```

Le rendre exécutable

    chmode +x /root/borg-backup.sh 

Exécution

    /root/borg-backup.sh

## BorgBackup Lenovo

*Sauvegarde de la machine Lenovo dans une boite de stockage*

### Installation

Installer borgbackup

    sudo apt install borgbackup

### Créer utilisateur borg

En mode su : `sudo -s`

Créer un utilisateur borg (sans home) dédié aux sauvegardes par BorgBackup

    useradd -M borg

Autoriser utilisateur borg à exécuter `/usr/bin/borg` uniquement

    echo "borg ALL=NOPASSWD: /usr/bin/borg" >> /etc/sudoers

### Clés ssh borg

créer une clé SSH pour l’authentification borg

    ssh-keygen -t ed25519 -f /root/.ssh/id_borg_ed25519

Validez en appuyant sur la touche « Entrée » à toutes les questions

Vous devriez maintenant avoir une clé privée contenue dans le fichier `/root/.ssh/id_borg_ed25519`, et une clé publique contenue dans le fichier `/root/.ssh/id_borg_ed25519.pub`  
`La clé privée ne doit jamais être partagée.`{: .prompt-warning }

### Ajout clé publique à la boite de stockage

Depuis un poste ayant accès à la boîte de stockage, on récupère le fichier `authorized_keys` de la boîte de stockage **bx11-yann** dans un fichier nommé `storagebox_authorized_keys`

    echo -e "get .ssh/authorized_keys storagebox_authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de

	cat >> storagebox_authorized_keys

Copier/coller le contenu du fichier du fichier de clef publique (fichier `cat /root/.ssh/id_borg_ed25519.pub`* de la machine à sauvegarder dans ce terminal, et presser **[Ctrl]+[D]** pour valider.

On renvoie le fichier modifié `storagebox_authorized_keys` dans le fichier `authorized_keys` de la boîte de stockage **bx11-yann**

    echo -e "put storagebox_authorized_keys .ssh/authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de

Tester la connexion à la boîte de stockage (toujours en mode su) 

    sftp -P 23 -i /root/.ssh/id_borg_ed25519 u326239@u326239.your-storagebox.de

```
The authenticity of host '[u326239.your-storagebox.de]:23 ([2a01:4f8:b23:2000::35]:23)' can't be established.
ECDSA key fingerprint is SHA256:oDHZqKXnoMtgvPBjjC57pcuFez28roaEuFcfwyg8O5c.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[u326239.your-storagebox.de]:23,[2a01:4f8:b23:2000::35]:23' (ECDSA) to the list of known hosts.
Connected to u326239.your-storagebox.de.
sftp> 
```

Saisir *quit* pour sortir

### Création du dépôt Borg et création d'archives

Les dépôts de la boîte de stockage sont sous `./backup/borg/`

#### Dépôt (repository)

on va maintenant travailler sur un dépôt distant à travers SSH, il va falloir fournir le chemin du dépôt sous la forme d'une URL avec un peu plus d'informations

    ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/rnmkcy.eu

#### phrase forte (passphrase)

Ajout de la phrase forte dans un fichier au dossier `/root/.borg`

```
sudo -s
mkdir -p /root/.borg
# ajout phrase
echo "<La phrase de passe forte>" > /root/.borg/rnmkcy_eu.passphrase
```

Conserver cette phrase, elle sera demandé pour la création du dépôt borg

#### Initialisation dépôt distant

Initialisation dépôt distant en mode su

```shell
export BORG_PASSPHRASE="$(cat /root/.borg/rnmkcy_eu.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/rnmkcy.eu
borg init --encryption=repokey $BORG_REPOSITORY
```

#### fichier d'exclusion

Créer un fichier d'exclusion `/root/.borg/exclusions-borg.txt`

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
lost+found
```

#### Sauvegarde rnmkcy.eu vers boîte de stockage

Lancer le premier backup

    borg create ssh://borg@xoyize.xyz:55029/srv/ssd-two/borg-backups/cinay.eu::2019-01-11 /home/yanfi
    sudo nice -n 19 ionice -c 3 borg create -v --stats --progress --exclude-from="/media/veracrypt2/excludes-backup-asus.txt" /media/veracrypt2/backup.borg::asus-{now} /

```shell
export BORG_PASSPHRASE="$(cat /root/.borg/rnmkcy_eu.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/rnmkcy.eu
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
nice -n 19 ionice -c 3 borg create -v --stats --progress --exclude-from="/root/.borg/exclusions-borg.txt" $BORG_REPOSITORY::$BACKUP_DATE /
```

**nice/ionice** pour minimiser l'impact (priorité CPU et disque minimale)

Créer un script de sauvegarde (notez l'usage de borg prune pour supprimer les archives trop anciennes)  

    sudo nano /root/.borg/borg-backup.sh 

```shell
export BORG_PASSPHRASE=`cat /root/.borg/rnmkcy_eu.passphrase`
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/rnmkcy.eu
borg create -v --progress --stats --exclude-from /root/.borg/exclusions-borg.txt ${BORG_REPOSITORY}::${BACKUP_DATE} /
borg prune -v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 $BORG_REPOSITORY

root@rnmkcy:/home/leno# cat /root/.borg/borg-backup.sh
export BORG_PASSPHRASE=`cat /root/.borg/rnmkcy_eu.passphrase`
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/rnmkcy.eu
borg create -v --progress --stats --exclude-from /root/.borg/exclusions-borg.txt ${BORG_REPOSITORY}::${BACKUP_DATE} /
borg prune -v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 $BORG_REPOSITORY
```

Le rendre exécutable

    chmode +x /root/.borg/borg-backup.sh 

Exécution

    /root/.borg/borg-backup.sh

#### Débloquer un dépôt verrouillé

Commande 

```bash
export BORG_PASSPHRASE="$(cat /root/.borg/rnmkcy_eu.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/rnmkcy.eu
borg break-lock $BORG_REPOSITORY
```

##### Automatiser sauvegarde 

Automatiser en utilisant systemd timer

Le service `/etc/systemd/system/autoborg.service`

```
[Unit]
Description=BorgBackup

[Service]
User=root
ExecStart=/usr/bin/bash /root/.borg/borg-backup.sh

[Install]
WantedBy=multi-user.target
```

Le timer `/etc/systemd/system/autoborg.timer`

```
[Unit]
Description=Exécution BorgBackup

[Timer]
Unit=autoborg.service
OnCalendar=*-*-* 02:55

[Install]
WantedBy=timers.target
```

Exécution tous les jours 

Activez/démarrez le timer, puis vérifiez qu'il est chargé et actif 

```bash
sudo systemctl enable autoborg.timer
sudo systemctl start autoborg.timer
systemctl status autoborg.timer
```

Vérifiez qu'il a été démarré en vérifiant s'il apparaît dans la liste des minuteries :

    systemctl list-timers

```
NEXT                        LEFT        LAST                        PASSED       UNIT                         ACTIVATES                     
Wed 2024-12-11 02:55:00 CET 17h left    Tue 2024-12-10 02:55:00 CET 6h ago       autoborg.timer               autoborg.service
```

Lorsque vous voulez voir si les sauvegardes se sont déroulées correctement, vous pouvez consulter le journal le plus récent 

    systemctl status autoborg

Ou afficher tous les journaux avec :

sudo journalctl -u autoborg


#### Liste des sauvegardes

Pour lister les sauvegardes (en mode su)

```bash
export BORG_PASSPHRASE="$(cat /root/.borg/rnmkcy_eu.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
borg list --short  ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/rnmkcy.eu
```

Résultat commande

```
2024-11-24-02h55
2024-11-30-02h55
2024-12-01-02h55
2024-12-04-02h55
2024-12-05-02h55
```

### Modification dépôt

*10/12/2024, on effectue la sauvegarde sur le dossier partagé /mnt/FreeUSB2To*

Modifier le script

    sudo nano /root/.borg/borg-backup.sh 

```shell
export BORG_PASSPHRASE=`cat /root/.borg/rnmkcy_eu.passphrase`
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
export BORG_RELOCATED_REPO_ACCESS_IS_OK=yes
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
#BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/rnmkcy.eu
BORG_REPOSITORY=/mnt/FreeUSB2To/sauvegardes/borgbackup/rnmkcy.eu
borg create -v --progress --stats --exclude-from /root/.borg/exclusions-borg.txt ${BORG_REPOSITORY}::${BACKUP_DATE} /
borg prune -v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 $BORG_REPOSITORY
```

## BorgBackup VPS

*Sauvegarde VPS dans une boite de stockage*

### Installation

Installer borgbackup

    sudo apt install borgbackup

### Créer utilisateur borg

En mode su : `sudo -s`

Créer un utilisateur borg (sans home) dédié aux sauvegardes par BorgBackup

    useradd -M borg

Autoriser utilisateur borg à exécuter `/usr/bin/borg` uniquement

    echo "borg ALL=NOPASSWD: /usr/bin/borg" >> /etc/sudoers.d/borg

### Clés ssh borg

créer une clé SSH pour l’authentification borg

    ssh-keygen -t ed25519 -f /root/.ssh/id_borg_ed25519

Validez en appuyant sur la touche « Entrée » à toutes les questions

Vous devriez maintenant avoir une clé privée contenue dans le fichier `/root/.ssh/id_borg_ed25519`, et une clé publique contenue dans le fichier `/root/.ssh/id_borg_ed25519.pub`  
`La clé privée ne doit jamais être partagée.`{: .prompt-warning }

### Ajout clé publique au serveur borg

#### boite de stockage

Depuis un poste ayant accès à la boîte de stockage, on récupère le fichier `authorized_keys` de la boîte de stockage **bx11-yann** dans un fichier nommé `storagebox_authorized_keys`

    echo -e "get .ssh/authorized_keys storagebox_authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de

	cat >> storagebox_authorized_keys

Copier/coller le contenu du fichier du fichier de clef publique (fichier `cat /root/.ssh/id_borg_ed25519.pub`* de la machine à sauvegarder dans ce terminal, et presser **[Ctrl]+[D]** pour valider.

On renvoie le fichier modifié `storagebox_authorized_keys` dans le fichier `authorized_keys` de la boîte de stockage **bx11-yann**

    echo -e "put storagebox_authorized_keys .ssh/authorized_keys" | sftp -P 23 -i ~/.ssh/bx11-yann-ed25519 u326239@u326239.your-storagebox.de

Tester la connexion depuis le vps à la boîte de stockage (toujours en mode su) 

    sftp -P 23 -i /root/.ssh/id_borg_ed25519 u326239@u326239.your-storagebox.de

```
The authenticity of host '[u326239.your-storagebox.de]:23 ([167.235.97.31]:23)' can't be established.
ED25519 key fingerprint is SHA256:XqONwb1S0zuj5A1CDxpOSuD2hnAArV1A3wKY7Z3sdgM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[u326239.your-storagebox.de]:23' (ED25519) to the list of known hosts.
Connected to u326239.your-storagebox.de.
sftp> 
```

Saisir *quit* pour sortir

#### Autre debian (option)

Copier la clé publique de chacun des clients avec l’option command en ajout sur le fichier /srv/data/borg-backups/.ssh/authorized_keys du serveur

A - Ajout de la clé publique id_borg_ed25519.pub sans option

    echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIws5uLxgF1UgGT8d/gfYueeEm5pFmRoXesbfD81QEpV root@yanfi.space' >> /srv/data/borg-backups/.ssh/authorized_keys

B - Ajout de la clé publique id_borg_ed25519.pub avec options

```
# connexion utilisateur borg
sudo -u borg -s
# Avec quota : borg serve --storage-quota 5G
# Client nom dépôt yanfi.space
echo 'command="cd /srv/data/borg-backups/yanfi_space; borg serve --restrict-to-path /srv/data/borg-backups/yanfi_space",no-port-forwarding,no-x11-forwarding,no-agent-forwarding,no-pty,no-user-rc ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIws5uLxgF1UgGT8d/gfYueeEm5pFmRoXesbfD81QEpV root@yanfi.space' >> /srv/data/borg-backups/.ssh/authorized_keys
exit
```

L’option command permet de restreindre l’utilisation de la connexion SSH à une commande donnée, pour le client qui utilisera la clé publique associée.

### Création du dépôt Borg et création d'archives

Les dépôts de la boîte de stockage sont sous `./backup/borg/`

#### Dépôt (repository)

on va maintenant travailler sur un dépôt distant à travers SSH, il va falloir fournir le chemin du dépôt sous la forme d'une URL avec un peu plus d'informations

    ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/iceyan.xyz

#### phrase forte (passphrase)

Ajout de la phrase forte dans un fichier au dossier `/root/.borg`

```
sudo -s
mkdir -p /root/.borg
# ajout phrase
echo "<La phrase de passe forte>" > /root/.borg/iceyan_xyz.passphrase
```

Conserver cette phrase, elle sera demandé pour la création du dépôt borg

#### Initialisation dépôt distant

Initialisation dépôt distant en mode su

```shell
export BORG_PASSPHRASE="$(cat /root/.borg/iceyan_xyz.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/iceyan.xyz
borg init --encryption=repokey $BORG_REPOSITORY
```

Résultat de la commande

```
By default repositories initialized with this version will produce security
errors if written to with an older version (up to and including Borg 1.0.8).

If you want to use these older versions, you can disable the check by running:
borg upgrade --disable-tam ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/iceyan.xyz

See https://borgbackup.readthedocs.io/en/stable/changes.html#pre-1-0-9-manifest-spoofing-vulnerability for details about the security implications.

IMPORTANT: you will need both KEY AND PASSPHRASE to access this repo!
If you used a repokey mode, the key is stored in the repo, but you should back it up separately.
Use "borg key export" to export the key, optionally in printable format.
Write down the passphrase. Store both at safe place(s).
```

#### fichier d'exclusion

Créer un fichier d'exclusion `/root/.borg/iceyan.xyz.exclusions`

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
```

#### Sauvegarde iceyan.xyz vers boîte de stockage

Lancer le premier backup

```shell
export BORG_PASSPHRASE="$(cat /root/.borg/iceyan_xyz.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/iceyan.xyz
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
nice -n 19 ionice -c 3 borg create -v --stats --progress --exclude-from="/root/.borg/iceyan.xyz.exclusions" $BORG_REPOSITORY::$BACKUP_DATE /
```

**nice/ionice** pour minimiser l'impact (priorité CPU et disque minimale)

Créer un script de sauvegarde (notez l'usage de borg prune pour supprimer les archives trop anciennes)  

    nano /root/.borg/borg-backup.sh 

```bash
export BORG_PASSPHRASE="$(cat /root/.borg/iceyan.xyz.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/iceyan.xyz
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
borg create -v --progress --stats --exclude-from /root/.borg/iceyan.xyz.exclusions ${BORG_REPOSITORY}::${BACKUP_DATE} /
borg prune -v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 $BORG_REPOSITORY
```

Le rendre exécutable

    chmod +x /root/.borg/borg-backup.sh 

#### Débloquer un dépôt verrouillé

Commande 

```bash
export BORG_PASSPHRASE="$(cat /root/.borg/iceyan_xyz.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/iceyan.xyz
borg break-lock $BORG_REPOSITORY
```

##### Automatiser sauvegarde 

Automatiser en utilisant systemd timer

Le service `/etc/systemd/system/autoborg.service`

```
[Unit]
Description=BorgBackup

[Service]
User=root
ExecStart=/usr/bin/bash /root/.borg/borg-backup.sh

[Install]
WantedBy=multi-user.target
```

Le timer `/etc/systemd/system/autoborg.timer`

```
[Unit]
Description=Exécution BorgBackup

[Timer]
Unit=autoborg.service
OnCalendar=*-*-* 04:10

[Install]
WantedBy=timers.target
```

Exécution tous les jours 

Activez/démarrez le timer, puis vérifiez qu'il est chargé et actif 

```bash
sudo systemctl enable autoborg.timer --now
systemctl status autoborg.timer
```

Vérifiez s'il apparaît dans la liste des minuteries :

    systemctl list-timers

```

NEXT                         LEFT          LAST                         PASSED     UNIT                         ACTIVATES                     
Tue 2024-09-17 04:10:00 GMT 10h left            -                           -             autoborg.timer               autoborg.service
```

Lorsque vous voulez voir si les sauvegardes se sont déroulées correctement, vous pouvez consulter le journal le plus récent 

    systemctl status autoborg

Ou afficher tous les journaux avec :

sudo journalctl -u autoborg

Pour lancer une sauvegarde manuellement

    /usr/bin/bash /root/.borg/borg-backup.sh

#### Liste des sauvegardes

Pour lister les sauvegardes

```bash
export BORG_PASSPHRASE="$(cat /root/.borg/iceyan_xyz.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
borg list --short  ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/iceyan.xyz
```

Donne un résultat de ce type

```
2024-09-16-17h43
2024-09-16-18h01
```

