+++
title = 'BorgBackup sauvegarde Nextcloud et sa Base MySQL'
date = 2024-12-10 00:00:00 +0100
categories = borgbackup
+++
*BorgBackup (abrégé : Borg) est un programme de sauvegarde par déduplication. En option, il prend en charge la compression et le chiffrement authentifié.*

![](borg-logo.png)

## BorgBackup 

*Sauvegarde de Nextcloud dans une boite de stockage*

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

## BorgBackup sauvegarde Nextcloud

### Création du dépôt nextcloud (repository)

on va maintenant travailler sur un dépôt distant à travers SSH, il va falloir fournir le chemin du dépôt sous la forme d'une URL avec un peu plus d'informations

    ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/nextcloud

### phrase forte (passphrase)

Ajout de la phrase forte dans un fichier au dossier `/root/.borg`

```
sudo -s
mkdir -p /root/.borg
# ajout phrase
echo "<La phrase de passe forte>" > /root/.borg/nextcloud.passphrase
```

Conserver cette phrase, elle sera demandé pour la création du dépôt borg

### Initialisation dépôt distant

Initialisation dépôt distant en mode su

```shell
export BORG_PASSPHRASE="$(cat /root/.borg/nextcloud.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/nextcloud
borg init --encryption=repokey $BORG_REPOSITORY
```

A la fin de la commande

```
By default repositories initialized with this version will produce security
errors if written to with an older version (up to and including Borg 1.0.8).

If you want to use these older versions, you can disable the check by running:
borg upgrade --disable-tam ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/nextcloud

See https://borgbackup.readthedocs.io/en/stable/changes.html#pre-1-0-9-manifest-spoofing-vulnerability for details about the security implications.

IMPORTANT: you will need both KEY AND PASSPHRASE to access this repo!
If you used a repokey mode, the key is stored in the repo, but you should back it up separately.
Use "borg key export" to export the key, optionally in printable format.
Write down the passphrase. Store both at safe place(s).
```

### Procédure sauvegarde nextcloud vers boîte de stockage

Créer un script de sauvegarde en mode su (notez l'usage de borg prune pour supprimer les archives trop anciennes) : `/root/.borg/ncbackup.sh` 

<details>
<summary><b>Etendre Réduire</b></summary>
{% highlight bash %}  
#!/bin/bash

# En définissant ceci, le repo n'a pas besoin d'être donné sur la ligne de commande :
export BORG_REPO="ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/nextcloud"

# Définir ceci, afin que l'on ne vous demande pas votre phrase d'authentification du référentiel :
export BORG_PASSPHRASE="$(cat /root/.borg/nextcloud.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
# ou ceci pour demander à un programme externe de fournir la phrase de passe :
#export BORG_PASSCOMMAND='pass show backup'

# quelques aides et gestion des erreurs :
info() { printf "\n%s %s\n\n" "$( date )" "$*" >&2; }
trap 'echo $( date ) Backup interrupted >&2; exit 2' INT TERM

# Fonction pour les messages d'erreur
errorecho() { cat <<< "$@" 1>&2; }

# Bail si le borg est déjà en cours d'exécution, peut-être que l'exécution précédente n'a pas été achevée.
if pidof -x borg >/dev/null; then
    echo "Sauvegarde déjà en cours"
    # mail -s "Nextcloud Backup. Borg fonctionne déjà." youremail@yourdomain < /home/pi/scripts/backup.txt
    exit
fi

#
# Vérifier si root
#
if [ "$(id -u)" != "0" ]
then
	errorecho "ERREUR : Ce script doit être exécuté en tant que root !"
	exit 1
fi

#
# nextcloud vars
#
# nextcloudFileDir = le dossier de votre installation nextcloud
nextcloudFileDir="/var/www/nextcloud"
nextcloudDataDir="/srv/nextcloud-data"
# dbdumpdir = le dossier temporaire pour les dumps de la base de données
dbdumpdir="/root/.borg/dbdump"
mkdir -p $dbdumpdir
# dbdumpfilename = le nom du fichier dump de la base de données
dbdumpfilename=$(hostname)-nextcloud-db.sql-$(date +"%Y-%m-%d_%H:%M:%S")

#
# les variables de la base de données, remplacez vos propres valeurs ici
#
dbUser="nextcloud"
dbPassword="FolioleNatrumRecouseToquadeMoraineDemande"
nextcloudDatabase="nextcloud"

# exclure des fichiers et des dossiers. Vous pouvez les modifier et/ou en ajouter d'autres. 
# Il ne s'agit que des vars. Elles sont ensuite ajoutées à borg create
exclude_updater="'$nextcloudDataDir/updater-*'"
exclude_updater_hidden="'$nextcloudDataDir/updater-*/.*'"

#
# variables du serveur web
#
webserverUser="nextcloud"
webserverServiceName="nginx"

info "Démarrage de la sauvegarde..."

echo "Affichage des fichiers et dossiers exclus..."
echo $exclude_updater
echo $exclude_updater_hidden

#
# Définir le mode de maintenance pour Nextcloud
#
echo "Définir le mode de maintenance pour Nextcloud..."
cd "${nextcloudFileDir}"
sudo -u "${webserverUser}" php occ maintenance:mode --on
cd ~
echo "Done"
echo

#
# Arrêter le serveur web
#
#echo "Arrêter le serveur web..."
#service "${webserverServiceName}" stop
#echo "Done"
echo


#
# Sauvegarde de la base de données. La base de données est sauvegardée dans un dossier temporaire.
# Il sera récupéré par l'archive. Elle sera ensuite supprimée.
#
echo "Sauvegarde de la base de données Nextcloud..."
mysqldump --single-transaction -h localhost -u $dbUser -p$dbPassword $nextcloudDatabase > $dbdumpdir/$dbdumpfilename
echo "mysql dump réussi. Dossier dump ${dbdumpdir}"
echo "Lister dossier dump..."
ls -l ${dbdumpdir}
echo
#echo "Taille sauvegarde base de données: $(stat --printf='%s' ${dbdumpdir}/${dbdumpfilename} | numfmt --to=iec)"
echo
echo "Terminé"
echo

# Sauvegarder les répertoires nextcloud et dbdump 
# dans une archive nommée d'après la machine sur laquelle 
# ce script s'exécute actuellement :
echo "Sauvegarde des fichiers nextcloud..."

borg create                         \
    --verbose                       \
    --filter AME                    \
    --list                          \
    --stats                         \
    --show-rc                       \
    --compression lz4               \
    ::'nextcloud-{now}'            \
    $nextcloudFileDir/config        \
    $nextcloudFileDir/themes        \
    $nextcloudDataDir               \
    $dbdumpdir                      \
    --exclude-caches                \
    --exclude '*.log'               \
    --exclude '*.log.*'             \
    --exclude $exclude_updater      \
    --exclude $exclude_updater_hidden \

backup_exit=$?

#
# Le fichier db dump est supprimé à cette étape car il n'est plus nécessaire. 
# Il a été inclus dans l'archive. 
# Il est supprimé afin de nettoyer le dossier pour les futures sauvegardes.
#
info "Supprimer le fichier dump base de données"
rm  $dbdumpdir/*
echo "Terminé"

info "Elagage du dépôt"
echo "Elagage du dépôt." 
echo "On garde 5 jours, 2 semaines et 1 mois (Daily 5, Weekly 2, Monthly 1)"
# Remarque : vous pouvez modifier ces valeurs à votre guise.
borg prune                          \
    --list                          \
    -v                              \
    --glob-archives 'nextcloud-*'   \
    --show-rc                       \
    --keep-daily=5                  \
    --keep-weekly=2                 \
    --keep-monthly=1                \

prune_exit=$?

#
# Démarrer le serveur web
#
#echo
#echo "Démarrer le serveur web..."
#service "${webserverServiceName}" start
#echo "Terminé"
#echo


#
# Désactiver le mode maintenance
#
echo "Désactivation du mode maintenance..."
cd "${nextcloudFileDir}"
sudo -u "${webserverUser}" php occ maintenance:mode --off
cd ~
echo "Terminé"
echo

# utiliser le code de sortie le plus élevé comme code de sortie global
global_exit=$(( backup_exit > prune_exit ? backup_exit : prune_exit ))

if [ ${global_exit} -eq 1 ];
then
    info "Sauvegarde et/ou élagage terminés avec un avertissement"
fi

if [ ${global_exit} -gt 1 ];
then
    info "La sauvegarde et/ou l'élagage se sont terminés par une erreur"
fi

#
# envoyer un courriel. Décommentez la ligne ci-dessous pour envoyer un courriel. Pour cela, vous devez d'abord configurer un MTA
# Pour envoyer du courrier, configurez votre script cron comme ceci:
# 55 23 * * * /root/backup.sh > /home/<user>/backup.txt 2>&1
#
# mail -s "Nextcloud Backup" youremail@yourdomain.com < /home/<user>/backup.txt

exit ${global_exit}

echo "Terminé!"
{% endhighlight %}
</details>

Le rendre exécutable

    chmode +x /root/.borg/ncbackup.sh 

Exécution

    /root/.borg/ncbackup.sh

### Automatiser sauvegarde 

Automatiser en utilisant systemd timer

Le service `/etc/systemd/system/borg-nextcloud.service`

```
[Unit]
Description=BorgBackup nextcloud

[Service]
User=root
ExecStart=/usr/bin/bash /root/.borg/ncbackup.sh

[Install]
WantedBy=multi-user.target
```

Le timer `/etc/systemd/system/borg-nextcloud.timer`

```
[Unit]
Description=Exécution BorgBackup nextcloud

[Timer]
Unit=borg-nextcloud.service
OnCalendar=*-*-* 02:30

[Install]
WantedBy=timers.target
```

Exécution tous les jours 

Activez/démarrez le timer, puis vérifiez qu'il est chargé et actif 

```bash
# Recharger
sudo systemctl daemon-reload
# Activation et lancement
sudo systemctl enable borg-nextcloud.timer --now
systemctl status borg-nextcloud.timer
```

Vérifiez qu'il a été démarré en vérifiant s'il apparaît dans la liste des minuteries :

    systemctl list-timers

```
NEXT                        LEFT        LAST                        PASSED       UNIT                         ACTIVATES                     
Wed 2024-12-11 02:30:00 CET 8h left       -                           -             borg-nextcloud.timer         borg-nextcloud.service
```

Lorsque vous voulez voir si les sauvegardes se sont déroulées correctement, vous pouvez consulter le journal le plus récent 

    systemctl status borg-nextcloud

Ou afficher tous les journaux avec :

    sudo journalctl -u borg-nextcloud

#### Liste des sauvegardes sous Lenovo

Pour lister les sauvegardes (en mode su)

```bash
export BORG_PASSPHRASE="$(cat /home/leno/sharenfs/pc1/.borg/nextcloud.passphrase)"
export BORG_RSH='ssh -i /home/leno/sharenfs/pc1/.borg/nextcloud.borgssh'
borg list --short $(cat /home/leno/sharenfs/pc1/.borg/nextcloud.repository)
```

Résultat commande

```
nextcloud-2024-12-10T18:39:36
nextcloud-2024-12-10T18:43:27
```
