+++
title = 'Exécuter votre propre serveur Sync-1.5 (sync firefox)'
date = 2018-11-23 00:00:00 +0100
categories = []
+++
Exécuter votre propre serveur Sync-1.5 (sync firefox)
---
layout: article
title:  Exécuter votre propre serveur de synchronisation firefox
toc: true
date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23
tags: [navigateur]
lang: fr
description:  serveur Sync-1.5
---


## Exécuter votre propre serveur Sync-1.5 (sync firefox)

[Run your own Sync-1.5 Server](https://mozilla-services.readthedocs.io/en/latest/howtos/run-sync-1.5.html)  
[Run-Your-Own Firefox Sync Server](https://github.com/mozilla-services/syncserver)  

### Prérequis

 sudo apt-get install python-dev git-core python-virtualenv g++



### Construire le serveur

Télcharger la dernière version <https://github.com/mozilla-services/syncserver> et exécuter les commandes suivantes:

```
cd ~
git clone https://github.com/mozilla-services/syncserver
sudo mv syncserver/ /srv/
sudo chown $USER. -R /srv/syncserver
cd /srv/syncserver
make build
```

Cette commande créera un environnement Python isolé avec toutes les dépendances requises. Un répertoire **local/bin** est créé et contient une commande **gunicorn** qui peut être utilisée pour exécuter le serveur.

Si vous le souhaitez, vous pouvez exécuter la suite de tests pour vous assurer que tout fonctionne correctement:

	make test

### Création Database mysql

```
mysql -u root -p     # yunohost : mysql -u root -p$(cat /etc/yunohost/mysql)
CREATE DATABASE syncserver;
GRANT ALL PRIVILEGES ON syncserver.* TO sync IDENTIFIED BY "xpwA776FM3";
exit
```



### Configuration de base

Nginx + Gunicorn  
Installer gunicorn dans l'environnement python syncserver  

```
$ cd /usr/src/syncserver
$ local/bin/easy_install gunicorn
```

Activer le serveur gunicorn dans le fichier **/srv/syncserver/syncserver.ini**  

```
[server:main]
use = egg:gunicorn
host = 127.0.0.1
port = 5000
workers = 2
timeout = 60
```

Dans **public_url**, on va mettre l’URI final. Par exemple : <https://media.ouestline.net>.  
`public_url = https://media.ouestline.net/`  
Paramètre  base de données serveur mysql:  
Dans **sqluri**, on va dé-commenter et mettre l’emplacement ou l’URI qui va permettre au serveur de se connecter la base de données  

```
[syncserver]
sqluri = pymysql://sync:xpwA776FM3@localhost:3306/syncserver
```

Pour le paramètre **secret**, on va devoir générer une clé secrète pour les tokens d’authentification. Pour cela, on va entrer la commande suivante, on dé-commente la ligne puis on copie le contenu pour le mettre dans le paramètre secret :

	head -c 20 /dev/urandom | sha1sum
	89a8537db89b230d0ef641b72ca8a2357bfcad1f

`secret = 280aede94abad286bebeb6580e99f96026b76d37`  
Pour le paramètre **allow_new_users**, on va simplement le dé-commenter et mettre comme paramètre **true** pour autoriser notre compte à ce connecter pour la première fois à notre serveur (pensez à mettre **false** une fois que tous les comptes que vous souhaitez héberger ce sont connecter au moins une fois sur votre serveur).  
`allow_new_users = true`  
On va ensuite modifier le paramètre **audiences** et mettre la même chose que le paramètre **public_url** sans oublier de dé-commenter la ligne. (Normalement ce paramètre n’est pas obligatoire mais il évite les erreurs).  
`audiences = https://media.ouestline.net`  
Il suffit d’ajouter la ligne suivante à la fin de votre fichier (pour mes mêmes raisons) :  
`forwarded_allow_ips = *`  

Le proxy nginx  

```
# proxy serveur sync firefox

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name media.ouestline.net;

    include ssl_params;

        location / {
                proxy_set_header Host $http_host;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_redirect off;
                proxy_read_timeout 120;
                proxy_connect_timeout 10;
                proxy_pass http://127.0.0.1:5000/;
        }
}
```

### Démarrage du serveur de synchronisation

Une multitude de moyens s’offrent à vous pour démarrer le serveur de synchronisation comme le lancement au démarrage du système par Cron, le lancement  via systemctl.  

**A-Lancement via cron**  
Pour démarrer le serveur de synchronisation ,lancer la commande suivante :

	./srv/syncserver/local/bin/gunicorn --paste /srv/syncserver/syncserver.ini &

Elle permet de choisir l’emplacement du fichier de configuration mais aussi de mettre l’argument --threads 4 qui permet d’attribuer plus de puissance au serveur de synchronisation.  
Pour démarrer le serveur à chaque démarrage du serveur, vous pouvez ajouter la ligne suivante dans votre crontab en tapant la commande crontab -e :

	@reboot ./srv/syncserver/local/bin/gunicorn --paste /srv/syncserver/syncserver.ini &


**B-Lancement via systemctl**  

Pour lancer le serveur **syncserver** au démarrage  
`sudo nano /etc/systemd/system/syncserver.service`  
Contenu du fichier  

```
[Unit]
Description=syncserver Service
After=network.target

[Service]
Type=simple
User=utilisateur
ExecStart=/srv/syncserver/local/bin/gunicorn --paste /srv/syncserver/syncserver.ini
Restart=on-abort


[Install]
WantedBy=multi-user.target
```

> **ATTENTION!** , remplacer **utilisateur** par votre nom d’utilisateur (echo $USER)

Lancer le service **syncserver** :  
`sudo systemctl start syncserver`  
Vérifier  
`sudo systemctl status syncserver`  


```
● syncserver.service - syncserver Service
   Loaded: loaded (/etc/systemd/system/syncserver.service; disabled; vendor preset: enabled)
   Active: active (running) since lun. 2017-09-25 15:14:15 CEST; 7s ago
 Main PID: 7816 (gunicorn)
   CGroup: /system.slice/syncserver.service
           ├─7816 /srv/syncserver/local/bin/python2 /srv/syncserver/local/bin/gunicorn --paste /srv/sync
           ├─7821 /srv/syncserver/local/bin/python2 /srv/syncserver/local/bin/gunicorn --paste /srv/sync
           └─7822 /srv/syncserver/local/bin/python2 /srv/syncserver/local/bin/gunicorn --paste /srv/sync

sept. 25 15:14:15 shuttle systemd[1]: Started syncserver Service.
sept. 25 15:14:15 shuttle gunicorn[7816]: [2017-09-25 15:14:15 +0000] [7816] [INFO] Starting gunicorn 19
sept. 25 15:14:15 shuttle gunicorn[7816]: [2017-09-25 15:14:15 +0000] [7816] [INFO] Listening at: http:/
sept. 25 15:14:15 shuttle gunicorn[7816]: [2017-09-25 15:14:15 +0000] [7816] [INFO] Using worker: sync
sept. 25 15:14:15 shuttle gunicorn[7816]: [2017-09-25 15:14:15 +0000] [7821] [INFO] Booting worker with 
sept. 25 15:14:15 shuttle gunicorn[7816]: [2017-09-25 15:14:15 +0000] [7822] [INFO] Booting worker with 
```

Valider le lancement du service syncserver au démarrage  
`sudo systemctl enable syncserver`  

### Mise à jour du serveur

Vous devrez périodiquement mettre à jour votre code pour vous assurer que vous avez les dernières corrections. Les commandes suivantes mettront à jour syncserver en place:

```
$ cd /srv/syncserver
$ git stash       # to save any local changes to the config file
$ git pull        # to fetch latest updates from github
$ git stash pop   # to re-apply any local changes to the config file
$ make build      # to pull in any updated dependencies
```

### Configurer le poste client

Ouvrez un nouvel onglet firefox et entrer dans la barre d’adresse :  
`about:config`  # Confirmer "je prends le risque"  
Dans **Rechercher**, saisir *dentity.sync.tokenserver.uri*  
Double-clic sur le paramètre **identity.sync.tokenserver.uri** et saisir
`https://media.ouestline.net/token/1.0/sync/1.5`  
Créer (ou se connecter) un compte sur firefox <https://accounts.firefox.com/signup>  
Lancer la synchronisation   


