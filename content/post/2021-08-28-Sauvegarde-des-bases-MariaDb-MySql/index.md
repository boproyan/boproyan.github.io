+++
title = 'Sauvegarde des bases MariaDb/MySql'
date = 2021-08-28 00:00:00 +0100
categories = ['outils']
+++
*sauvegarde au format SQL des bases MariaDb/MySql avec une rotation de 7 jours* 

Prérequis

* Un répertoire de sauvegarde
* Le mot de passe administrateur MariaDb/MySql sous forme de fichier
* Crontab pour la planification

Création du dossier de sauvegarde

	sudo mkdir -p /srv/ssd-two/XS35V2/mysql
	sudo chown $USER.$USER -R /srv/ssd-two/XS35V2

Le fichier **/etc/xs35v2/mysql** contient le mot de passe administrateur MariaDb/MySql

Création script de sauvegarde des bases de données

	nano /srv/ssd-two/XS35V2/dumpmysql.sh

```
#!/bin/bash

# Vérifier si utilisateur root
if [ $(id -u) != "0" ]; then
    echo "Erreur: Vous devez être 'root' pour exécuter ce script."
    exit 1
fi

# Configuration de base: datestamp e.g. YYYYMMDD
DATE=$(date +"%Y%m%d")

# Dossier où sauvegarder les backups (créez le d'abord!)
BACKUP_DIR="/srv/ssd-two/XS35V2/mysql"

# Identifiants MySQL
MYSQL_USER="root"
MYSQL_PASSWORD=$(cat /etc/xs35v2/mysql)
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

Le rendre exécutable

    chmod +x /srv/ssd-two/XS35V2/dumpmysql.sh

Test , passer en mode su

    sudo -s
    /srv/ssd-two/XS35V2/dumpmysql.sh

Vérifier dans le dossier du jour (AAAAMMJJ)

    ls /srv/ssd-two/XS35V2/mysql/20190127/
        nextcloud.sql.gz

Planification

    sudo -s
    crontab -e

```
# Sauvegarde des bases MariaDb/MySql
30 02 * * * /srv/ssd-two/XS35V2/dumpmysql.sh > /dev/null
```
