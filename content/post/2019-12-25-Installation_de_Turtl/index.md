+++
title = 'Installation_de_Turtl'
date = 2018-11-23 00:00:00 +0100
categories = []
+++
2017-06-12-Installation de Turtl
---
layout: article  
title:  Framanotes : installation serveur Turtl   
toc: true
ref:   
date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23
tags: [frama]  
lang: fr  
description: dégooglisation  
---


# Installation de Turtl

## Description

[Turtl](https://turtlapp.com/) est un logiciel libre distribué sous [licence AGPLv3](https://github.com/turtl/api/blob/master/LICENSE) qui a pour objectif de fournir un système de notes synchronisables sur (presque) toutes les plateformes (presque parce l’application iOS n’est pas encore lancée). Il s’agit de la solution logicielle qui propulse le service en ligne [Framanotes](https://framanotes.org/).  
Ce guide est prévu pour Debian Stable. À l’heure où nous écrivons ces mots, il s’agit de Debian 8.6 Jessie.  
Nous partons du principe que vous venez d’installer le système, que celui-ci est à jour et que vous avez déjà installé le serveur web Nginx.  
Nous considérerons que nous installerons Turtl sous l’utilisateur **www-data**.

## Nourrir la terre

![Nourrir la terre](preparer.png)

La partie serveur de Turtl est codée en [Common Lisp](https://common-lisp.net/) et utilise le moteur de base de données [RethinkDB](https://www.rethinkdb.com/).

Nous allons donc devoir avant tout les installer.  

## Common Lisp

Common Lisp possède plusieures implémentations différentes. Nous utiliserons [Clozure Common Lisp](http://ccl.clozure.com/) car [Steel Bank Common Lisp](http://www.sbcl.org/), qui a pourtant l’avantage d’être dans les paquets Debian (sous le petit nom de sbcl), pose quelques problèmes avec **Turtl**.

Installation de Clozure CL :

```
cd /opt
wget ftp://ftp.clozure.com/pub/release/1.11/ccl-1.11-linuxx86.tar.gz
tar xvf ccl-1.11-linuxx86.tar.gz
cd ccl
cp scripts/ccl64 /usr/bin/ccl
sed -e "s@CCL_DEFAULT_DIRECTORY=/usr/local/src/ccl@CCL_DEFAULT_DIRECTORY=/opt/ccl@" -i /usr/bin/ccl
```

Nous aurons besoin de quelques paquets du système pour les dépendances de Turtl :

```
apt-get install cl-cffi cl-quicklisp libuv1-dev build-essential 
```

Nous allons maintenant configurer un peu l’environnement de l’utilisateur **www-data** pour qu’il puisse lancer Turtl sans souci.  
Il faut commencer par créer le fichier `~www-data/.ccl-init.lisp` (~www-data signifie « le répertoire personnel de www-data » pour le système, soit /var/www dans une Debian) et y mettre ceci :

```
#-quicklisp
(let ((quicklisp-init (merge-pathnames "quicklisp/setup.lisp" (user-homedir-pathname))))
  (when (probe-file quicklisp-init)
    (load quicklisp-init)))
```

Il faudra aussi créer quelques répertoires et corriger les permissions :

```
mkdir ~www-data/quicklisp ~www-data/.cache/
chown www-data: ~www-data/quicklisp ~www-data/.cache/ ~www-data/.ccl-init.lisp
```

>Explication : *[Quicklisp](https://www.quicklisp.org/beta/) est un gestionnaire de dépendances pour Common Lisp. Nous l’avons installé via les paquets du système (cl-quicklisp), nous indiquons que www-data doit le charger lorsqu’il utilise ccl et enfin nous créons les dossiers où Quicklisp va compiler et installer les dépendances.*

Initialisons Quicklisp :

```
su www-data -s /bin/bash
ccl --load /usr/share/cl-quicklisp/quicklisp.lisp
```

Dans la console Lisp, tapez :

```
(quicklisp-quickstart:install)
(quit)
```

Puis, de retour au shell :

```
echo "(pushnew \"./\" asdf:*central-registry* :test #'equal)" >> ~www-data/.ccl-init.lisp
```

## RethinkDB

Pour l’installation, nous suivons la procédure de la [documentation officielle](https://www.rethinkdb.com/docs/install/debian/) :

```
echo "deb http://download.rethinkdb.com/apt `lsb_release -cs` main" | tee /etc/apt/sources.list.d/rethinkdb.list
wget -qO- https://download.rethinkdb.com/apt/pubkey.gpg | apt-key add -
apt-get update
apt-get install rethinkdb
```

Créons une instance pour Turtl et lançons-la :

```
echo "http-port=8091" > /etc/rethinkdb/instances.d/turtl.conf
service rethinkdb restart
```

Il est toujours bon d’être capable de sauvegarder et restaurer sa base de données.  
Installation des dépendances :

```
apt-get install python-pip
pip install rethinkdb
```

Création d’une sauvegarde :

rethinkdb dump

Restauration d’une sauvegarde :

```
rethinkdb restore backup.tar.gz
```

## Semer

![Semer](semer.png)

Créons quelques dossiers nécessaires :

```
cd /var/www
mkdir turtl/data -p
```

Passons maintenant à l’installation de Turtl proprement dite :

```
cd turtl
git clone https://github.com/turtl/api.git
cd api
# Copions le modèle de fichier de configuration
cp config/config.default.lisp config/config.lisp
```

Voici les principaux paramètres à modifier dans config/config.lisp :

```
# Ne faire écouter le serveur qu'en local
(defvar *server-bind* "127.0.0.1"
# Autoriser les requêtes depuis un autre site web (ce sera notre interface web)
# https ou http, c'est selon ce que vous mettrez en place,
# mais il est évident qu'il est mieux de sécuriser les communications
(defvar *enabled-cors-resources* "https://notes.example.org"
# L'adresse de l'API
# https ou http, c'est selon ce que vous mettrez en place,
# mais il est évident qu'il est mieux de sécuriser les communications
(defvar *site-url* "https://api.notes.examples.org"
# Une adresse de contact
(defvar *admin-email* "contact@example.org"
# L'adresse d'envoi de mail
(defvar *email-from* "noreply@example.org"
# L'adresse du serveur SMTP à utiliser pour envoyer les mails
(defvar *smtp-host* "localhost")
# Ne pas envoyer les erreurs
# (utile pour le debug, pas pour la production)
(defvar *display-errors* nil
# La limite de stockage par défaut (en Mio)
(defparameter *default-storage-limit* 100
# Est-ce qu'on offre du stockage en plus à ceux qui invitent des amis ? (en Mio)
(defparameter *storage-invite-credit* 0
# Où stocker les fichiers
(defvar *local-upload* "/var/www/turtl/data"
# L'adresse d'envoi des fichiers
# https ou http, c'est selon ce que vous mettrez en place,
# mais il est évident qu'il est mieux de sécuriser les communications
(defvar *local-upload-url* "https://api.notes.example.org"
```

Nous pouvons dès lors lancer le service à la main :

```
ccl --load start.lisp
```

(Faites Ctrl+C puis (quit) pour le couper)  

## Arroser

![Arroser](arroser.png)

Installons un service **systemd** pour lancer **Turtl** : cela évite de devoir laisser un terminal ouvert et permettra de lancer automatiquement Turtl au démarrage du serveur.  
Créez le fichier `/etc/systemd/system/turtl.service` :

```ini
[Unit]
Description=Note taking service
Documentation=http://turtl.it
Requires=network.target
Requires=rethinkdb.service
After=network.target
After=rethinkdb.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/turtl/api/
ExecStart=/usr/bin/ccl -Q -b --load start.lisp

[Install]
WantedBy=multi-user.target
```

Ensuite, pour activer et lancer Turtl :

```
systemctl daemon-reload
systemctl start turtl
systemctl enable turtl
```

Rendons maintenant notre Turtl accessible depuis l’extérieur grâce à **Nginx**.  

Créez le fichier `/etc/nginx/sites-available/api.notes.example.org.conf` :

```
upstream turtl {
    server 127.0.0.1:8181;
}

server {
    listen 80;
    listen [::]:80;
    #listen 443;
    #listen [::]:443;

    server_name api.notes.example.org;

    access_log  /var/log/nginx/notes.example.org-ssl.access.log;
    error_log   /var/log/nginx/notes.example.org-ssl.error.log;

    #if ($scheme = http) {
    #    return 301 https://$server_name$request_uri;
    #}

    #ssl                 on;
    #
    #ssl_certificate     /etc/ssl/private/api.notes.example.org/fullchain.pem;
    #ssl_certificate_key /etc/ssl/private/api.notes.example.org/privkey.pem;
    #
    #ssl_ecdh_curve secp384r1;
    #ssl_prefer_server_ciphers   on;
    #ssl_protocols               TLSv1 TLSv1.1 TLSv1.2; # not possible to do exclusive
    #ssl_ciphers 'EDH+CAMELLIA:EDH+aRSA:EECDH+aRSA+AESGCM:EECDH+aRSA+SHA384:EECDH+aRSA+SHA256:EECDH:+CAMELLIA256:+AES256:+CAMELLIA128:+AES128:+SSLv3:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!DSS:!RC4:!SEED:!ECDSA:CAMELLIA256-SHA:AES256-SHA:CAMELLIA128-SHA:AES128-SHA';
    #
    #ssl_stapling on;
    #resolver 192.168.1.1;

    index index.html;

    location / {
        proxy_set_header    Host $host;
        proxy_pass http://turtl;
    }
}
```

(tout ce qui est commenté est à décommenter si vous souhaitez utiliser le protocole sécurisé https, ce que nous recommandons vivement !)

Activez la configuration :

```
ln -s /etc/nginx/sites-available/api.notes.example.org.conf /etc/nginx/sites-enabled/api.notes.example.org.conf
```

Vérifiez qu’il n’y a pas d’erreur et rechargez Nginx :

```
nginx -t && nginx -s reload
```

## Regarder pousser

![Regarder pousser](pailler.png)

Avec tout ce que nous avons fait jusque-là, vous avez un serveur Turtl prêt à être utilisé avec les [applications](https://turtlapp.com/download/).

Mais nous nous sommes rendu compte que les différentes applications ne sont qu’une seule et même base de code en Javascript empaqueté selon les plateformes. Nous avons donc tenté de mettre tout ce code Javascript sur le web et ça fonctionne : on peut s’en servir comme d’un client web ! Tout du moins, ça fonctionne parfaitement bien avec Chrome et Chromium, il y a quelques bugs graphiques pas trop gênants avec Firefox et cela ne semble pas fonctionner avec IE ou Edge.  

Mettons donc en place cette interface web.  

Nous aurons besoin de quelques paquets :

```
apt-get install nodejs npm
ln -s /usr/bin/nodejs /usr/bin/node
```

Installons cette interface :

```
cd /var/www/turtl/
git clone https://github.com/turtl/js.git www
cd www
npm install
# Copions le modèle de fichier de configuration
cp config/config.js.default config/config.js
```

Il y a un peu de configuration à faire dans `config/config.js` :

```
# L'adresse du serveur installé précédemment
# https ou http, c'est selon ce que vous mettrez en place,
# mais il est évident qu'il est mieux de sécuriser les communications
api_url: 'https://api.notes.examples.org',
# L'adresse à laquelle sera accessible notre client web
# https ou http, c'est selon ce que vous mettrez en place,
# mais il est évident qu'il est mieux de sécuriser les communications
site_url: 'https://notes.example.org',
```

Après cela, il n’y a plus qu’à mettre la dernière touche :

```
make minify
```

On corrige les permissions du dossier :

```
chown -R www-data: /var/www/turtl/
```

Et enfin, nous créons la configuration Nginx `/etc/nginx/sites-available/notes.example.org` :

```
server {
    listen 80; 
    listen [::]:80;
    #listen 443;
    #listen [::]:443;

    server_name notes.example.org;

    access_log  /var/log/nginx/notes.example.org-ssl.access.log;
    error_log   /var/log/nginx/notes.example.org-ssl.error.log;

    #if ($scheme = http) {
    #    return 301 https://$server_name$request_uri;
    #}
    #
    #ssl                 on; 
    #
    #ssl_certificate     /etc/ssl/private/notes.example.org/fullchain.pem;
    #ssl_certificate_key /etc/ssl/private/notes.example.org/privkey.pem;
    #
    #ssl_ecdh_curve secp384r1;
    #ssl_prefer_server_ciphers   on; 
    #ssl_protocols               TLSv1 TLSv1.1 TLSv1.2; # not possible to do exclusive
    #ssl_ciphers 'EDH+CAMELLIA:EDH+aRSA:EECDH+aRSA+AESGCM:EECDH+aRSA+SHA384:EECDH+aRSA+SHA256:EECDH:+CAMELLIA256:+AES256:+CAMELLIA128:+AES128:+SSLv3:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!DSS:!RC4:!SEED:!ECDSA:CAMELLIA256-SHA:AES256-SHA:CAMELLIA128-SHA:AES128-SHA';
    #
    #ssl_stapling on; 
    #resolver 192.168.1.1;

    root /var/www/turtl/www/;
    index index.html;
}
```

(tout ce qui est commenté est à décommenter si vous souhaitez utiliser le protocole sécurisé https, ce que nous recommandons vivement !)

Activez la configuration :

```
ln -s /etc/nginx/sites-available/notes.example.org.conf /etc/nginx/sites-enabled/notes.example.org.conf
```

Vérifiez qu’il n’y a pas d’erreur (nginx -t) et rechargez Nginx :

```
nginx -t && nginx -s reload
```

Et voilà, vous avez maintenant un client web disponible sur <http://notes.example.org>

