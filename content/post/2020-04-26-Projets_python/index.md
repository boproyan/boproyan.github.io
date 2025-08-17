+++
title = 'Projets python'
date = 2020-04-26 00:00:00 +0100
categories = python
+++
## Liens

* [Create a CRUD Restful Service API using Flask + Mysql [in 7 minutes!]](https://www.nintyzeros.com/2019/11/flask-mysql-crud-restful-api.html)
* [Storing JSON in a SQLite table using Python](https://devopsheaven.com/sqlite/databases/json/python/api/2017/10/11/sqlite-json-data-python.html)
* [User authentication with Flask-JWT](https://medium.com/trabe/user-authentication-with-flask-jwt-be0e6f457dff)
* [How to Secure a Flask REST API with JSON Web Token? ](https://geekflare.com/securing-flask-api-with-jwt/)


## Projet python supysonic

* [supysonic](https://github.com/spl0k/supysonic)
* [Python - Création d'environnements virtuels VENV et "Wing Personnal" ,installer applis avec "pip"](https://blog.cinay.xyz/2019/07/Python-environnements-virtuels-VENV-et-Wing_Personnal_Python_IDE.html)
* [Python/Environnement virtuel (archwiki)](https://wiki.archlinux.fr/Python/Environnement_virtuel)


>Python 3.3+ fournit le module **venv**. Pour les applications qui requièrent une version antérieure de Python, utiliser virtualenv

On se rend dans le répertoire **/home/yannick/media/dplus/python-dev** 

    cd /home/yannick/media/dplus/python-dev
    git clone https://github.com/spl0k/supysonic


### Environnement virtuel
    
Aller dans le répertoire supysonic et créer environnement 'supysonic' puis l'activer

    cd supysonic
    python -m venv supysonic
    source supysonic/bin/activate

(supysonic) [yannick@yannick-pc supysonic]$   
Une fois dans l'environnement virtuel, les modules peuvent être installés avec pip et les scripts peuvent être exécutés normalement.  
Sortir de l'environnement virtuel par cette fonction fournie par bin/activate:

    (supysonic) [yannick@yannick-pc supysonic]$ deactivate

### Utiliser "Wing Personal IDE" avec un environnement existant

Pour utiliser un virtualenv existant avec Wing, il suffit de définir l’exécutable Python dans les propriétés du projet de Wing (Project Properties) sur le python dans votre répertoire virtualenv. Wing l’utilise pour déterminer l’environnement à utiliser pour l’analyse des sources et pour exécuter, tester et déboguer votre code.

La façon la plus simple de trouver la valeur correcte à définir est de lancer votre Python virtualenv à l’extérieur de Wing et d’exécuter ce qui suit de manière interactive :

```bash
(supysonic) [yannick@yannick-pc supysonic]$ python -V
Python 3.8.2
(supysonic) [yannick@yannick-pc supysonic]$ python
Python 3.8.2 (default, Apr  8 2020, 14:31:25) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.executable
'/home/yannick/media/dplus/python-dev/supysonic/supysonic/bin/python'
>>> quit()
```

Ouvrir "Wing Personal" , Projet &rarr; Nouveau Projet

![](wingperso001.png){:width="600"}  
![](wingperso002.png){:width="600"}  
![](wingperso003.png){:width="600"}  
![](wingperso004.png){:width="600"}  
![](wingperso005.png){:width="600"}  
![](wingperso006.png){:width="600"}  
![](wingperso007.png){:width="600"}  

En mode terminal , dans l'environnement actif `(supysonic) [yannick@yannick-pc supysonic]$`   
Mise à jour pip

    pip install --upgrade pip

```
Successfully installed pip-20.0.2
```

Installation supysonic

    pip install .

```
[...]
Successfully installed Jinja2-2.11.2 MarkupSafe-1.1.1 Pillow-7.1.1 Werkzeug-1.0.1 certifi-2020.4.5.1 chardet-3.0.4 click-7.1.1 flask-1.1.2 idna-2.9 itsdangerous-1.1.0 mutagen-1.44.0 pathtools-0.1.2 pony-0.7.13 requests-2.23.0 supysonic-0.5.0 urllib3-1.25.9 watchdog-0.10.2 zipstream-1.1.4
```

Database mysql au lieu de sqlite (par défaut)  
Vérifier si module installé ou pas  

```
(supysonic) [yannick@yannick-pc supysonic]$ python
Python 3.8.2 (default, Apr  8 2020, 14:31:25) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import MySQLdb
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'MySQLdb'
>>> quit()
```

Pas installé

    pip install pymysql


## Configuration


_Supysonic_ cherche quatre fichiers pour sa configuration : `/etc/supysonic`,
`~/.supysonic`, `~/.config/supysonic/supysonic.conf` et `supysonic.conf` dans
le dossier en cours, en fusionnant les valeurs de tous les fichiers.

Les fichiers de configuration doivent respecter une structure similaire à celle du fichier INI de Windows, avec entête `[section]` et en utilisant une syntaxe "KEY = VALUE" ou "KEY : VALUE".

Vous trouverez un fichier d'exemple de configuration grossièrement documenté à la racine de l'en-tête
le fichier nommé "config.sample". Plus de détails ci-dessous.

Il y a six sections dans la configuration :

-  base  : définit la configuration de la base de données et des analyses supplémentaires
-  webapp  : configuration relative au serveur HTTP
-  daemon  : configuration de l'observateur de fichiers d'analyse
-  lastfm  : clés pour activer le scrobbling Last.FM
-  transcoding ) : définit les programmes de transcodage
-  mimetypes  : extension de fichier pour les correspondances de mimetype

### Section "base".

`database_uri` : la configuration la plus importante, définit le type et paramètres de la base de données à laquelle _Supysonic_ doit se connecter. Elle comprend généralement nom d'utilisateur, mot de passe, nom d'hôte et nom de la base de données. La forme typique d'une base de données. C'est l'URI :

    driver://nom d'utilisateur:mot de passe@hôte:port/base de données

Si la connexion nécessite des paramètres supplémentaires, ceux-ci peuvent être fournis sous forme de
chaîne d'interrogation, telle que :

    driver://nom d'utilisateur:mot de passe@hôte:port/base de données?param1=valeur1&param2=valeur2

Les pilotes pris en charge sont "sqlite", "mysql" et "postgres" (ou "postgresql").

Comme SQLite se connecte aux fichiers locaux, le format est légèrement différent. Le "fichier 
de l'URI est le nom de fichier de la base de données. Pour un chemin relatif, il nécessite trois barres obliques, pour les chemins absolus c'est aussi trois barres obliques suivies du chemin absolu, c'est-à-dire en fait quatre barres obliques sur les systèmes Unix.

```ini
; Cheminement relatif
database_uri = sqlite:///relative-file.db
; Chemin absolu sur les systèmes basés sur Unix
database_uri = sqlite:////home/user/supysonic.db
; Chemin absolu sous Windows
database_uri = sqlite:///C:\Users\user\supysonic.db
```

Une base de données compatible avec MySQL nécessite soit `MySQLdb` soit `pymysql` pour être installé. PostgreSQL a besoin de `psycopg2`.

>Notez que pour MySQL, si aucun jeu de caractères n'est défini sur l'URI, la valeur par défaut est
utf8mb4", quel que soit le paramétrage de votre installation MySQL.

Si `database_uri` n'est pas fourni, il s'agit par défaut d'une base de données SQLite stockée dans
`/tmp/supysonic/supysonic.db`.

`scanner_extensions` : Une liste d'extensions de fichiers séparées par des espaces, que l'analyseur est
limité à. Utile si vous avez plusieurs formats audio dans votre bibliothèque mais
ne veulent en servir qu'une partie. S'il est laissé vide, le scanner essaiera de lire chaque fichier
qu'il trouve.

`follow_symlinks` : si défini à `yes`, permet au scanner de suivre des liens symboliques.
Désactivé par défaut, ne l'activez que si vous faites confiance à votre système de fichiers car rien n'est
fait pour traiter les liens ou les boucles rompus.

```ini
[base]
Une base de données URI. Voir le dossier "schema" pour les scripts de création de schémas
; Par défaut : sqlite:////tmp/supysonic/supysonic.db
database_uri = sqlite:////var/supysonic/supysonic.db
;database_uri = mysql://supysonic:supysonic@localhost/supysonic
;database_uri = postgres://supysonic:supysonic@localhost/supysonic

;Facultatif, limiter le scanner à ces extensions. Valeur par défaut : none
scanner_extensions = mp3 ogg

;Le scanner doit-il suivre des liens symboliques ? Par défaut : no
follow_symlinks = no
```

### Section "webapp"

`cache_dir` : répertoire utilisé pour stocker les fichiers générés, tels que la couverture redimensionnée
l'art ou les fichiers transcodés. La valeur par défaut est `/tmp/supysonic`.

`cache_size` : taille maximale (en mégaoctets) du cache (sauf pour les trancodes).
La valeur par défaut est de 512 Mo.

`transcode_cache_size` : taille maximale (en mégaoctets) du cache de transcode.
La valeur par défaut est de 1024 Mo (1 Go)

`log_file` : fichier tournant où se trouvent certains événements générés par le serveur web
enregistré. Laissez vide pour désactiver l'enregistrement.

`log_level` : définit le seuil de gravité minimum des messages à ajouter à
`log_file`. Les valeurs possibles sont : `DEBUG`, `INFO`, `WARNING`, `ERROR` et `CRITICAL`  
La valeur par défaut est `WARNING`.

`mount_api` : Activer ou désactiver l'API Subsonic REST.Existe surtout pour les tests  
Par défaut : "on".

`mount_webui` : "on/off" active ou désactive l'interface web administrative.
Notez que si cette option est désactivée, les utilisateurs ne pourront pas définir un
format de transcodage.  
Par défaut : "on" 

```ini
[webapp]
; Répertoire de cache optionnel. Par défaut : /tmp/supysonic
cache_dir = /var/supysonic/cache

;Taille maximale du cache principal en Mo. Valeur par défaut : 512
cache_size = 512

;Transcodage de la taille maximale du cache en Mo. Valeur par défaut : 1024 (1GB)
transcode_cache_size = 1024

;Fichier journal tournant facultatif. Valeur par défaut : aucune
log_file = /var/supysonic/supysonic.log

;Niveau log. Valeurs possibles : DEBUG, INFO, WARNING, ERROR, CRITICAL.
Par défaut : AVERTISSEMENT
log_level = AVERTISSEMENT

Activer l'API Subsonic REST. Vous voudrez très probablement la garder activée.
Ici pour les tests. Valeur par défaut : on
;mount_api = 
```

### Section "daemon

`socket` : Fichier socket du domaine Unix (ou "named pipe" sous Windows) utilisé pour communiquer
entre le démon et les clients qui en dépendent (par exemple CLI, folder admin web page,
etc.) Notez que l'utilisation d'une adresse IP ici n'est pas prise en charge.
Par défaut : /tmp/supysonic/supysonic.sock

`run_watcher` : démarrer ou non l'observateur qui va écouter la bibliothèque
changements. Default: yes

wait_delay` : délai (en secondes) avant de déclencher l'opération de balayage après
un changement a été détecté. Cela permet d'éviter d'effectuer trop de balayages lorsque plusieurs
les changements sont détectés pour un seul fichier sur une courte période.
La valeur par défaut est de 5 secondes.Default: 5 seconds

`jukebox_command` : commande utilisée par le mode jukebox pour lire un seul fichier.
Voir la [documentation du jukebox](jukebox.md) pour plus de détails.

`log_file` : fichier en rotation où les événements générés par l'observateur de fichiers sont enregistrés.
S'il est laissé vide, tout enregistrement sera envoyé à stderr.

`log_level` : définit le seuil minimum de gravité des messages à ajouter à `log_file`. Les valeurs possibles sont : `DEBUG`, `INFO`, `WARNING`, `ERROR` et `CRITICAL`. La valeur par défaut est `WARNING`

```ini
[daemon]
;Fichier de socket que le démon écoutera pour les commandes de gestion entrantes
;Par défaut : /tmp/supysonic/supysonic.sock
socket = /var/run/supysonic.sock

;Définit si l'observateur de fichiers doit être lancé. Valeur par défaut : yes
run_watcher = yes

;Délai en secondes avant de déclencher l'opération de balayage après une modification détecté.
;Cela permet d'éviter d'effectuer trop de scans lorsque de multiples changements sont détectés pour un seul fichier sur une courte période. Valeur par défaut : 5
wait_delay = 5

;Commande utilisée par le juke-box
jukebox_command = mplayer -ss %offset %path

;Fichier journal tournant optionnel pour le démon du scanner. Enregistrements dans stderr si vide
log_file = /var/supysonic/supysonic-daemon.log
log_level = INFO
```

### Section "lastfm"

Cette section permet de définir les clés API pour permettre l'intégration de Last.FM dans Supysonic   Actuellement, il n'est utilisé que pour _scrobbler_ les morceaux joués et mettre à jour
l'information _qui joue maintenant_. Voir <https://www.last.fm/api> pour obtenir ces clés.  
Une fois les clés définies, les utilisateurs doivent relier leur compte en visitant leur profil
page sur l'interface utilisateur administrative de _Supysonic_.

`api_key` : Clé API Last.FM

`secret` : clé secrète associée à la clé API

```ini
[lastfm]
;API et clé secrète pour permettre le scrobbling. http://www.last.fm/api/accounts
;Défaut : none
;api_key =
;secret =
```

### Section "transcodage"

Cette section définit les programmes de ligne de commande à utiliser pour convertir un fichier audio à un autre format ou changer son débit binaire.  
Toutes les configurations dans l'exemple ci-dessous n'ont **pas** été testées de manière approfondie.  
Pour plus de détails, veuillez vous référer à la [configuration du transcodage](transcoding.md).

```ini
[transcoding]
;Programmes utilisés pour convertir d'un format/bitrate à un autre. Valeurs par défaut : aucune
transcoder_mp3_mp3 = lame --quiet --mp3input -b %outrate %srcpath -
transcoder = ffmpeg -i %srcpath -ab %outratek -v 0 -f %outfmt -
decoder_mp3 = mpg123 --quiet -w - %srcpath
decoder_ogg = oggdec -o %srcpath
decoder_flac = flac -d -c -s %srcpath
encoder_mp3 = lame --quiet -b %outrate - -
encoder_ogg = oggenc2 -q -M %outrate -
```

### Section "mimetypes

Utilisez cette section si le système sur lequel _Supysonic_ est installé a du mal à deviner le "mime" de certains dossiers. Cela ne peut être utile que dans de rares cas.  Voir les liens suivants pour une liste d'exemples :

* https://en.wikipedia.org/wiki/Media_type#Common_examples
* https://www.iana.org/assignments/media-types/media-types.xhtml

```ini
[mimetypes]
;Extension aux cartographies mimetype au cas où votre système aurait des difficultés à deviner
Défaut : aucun
;mp3 = audio/mpeg
;ogg = audio/vorbis
```

### Créer le fichier de configuration

Fichier **~/.config/supysonic/supysonic.conf**

    mkdir -p ~/.config/supysonic/
    nano ~/.config/supysonic/supysonic.conf

```
[base]
; A database URI. See the 'schema' folder for schema creation scripts
; Default: sqlite:////tmp/supysonic/supysonic.db
;database_uri = sqlite:////var/supysonic/supysonic.db
database_uri = mysql://supysonic:supysonic@localhost/supysonic
;database_uri = postgres://supysonic:supysonic@localhost/supysonic

; Optional, restrict scanner to these extensions. Default: none
;scanner_extensions = mp3 ogg

; Should the scanner follow symbolic links? Default: no
follow_symlinks = no

[webapp]
; Optional cache directory. Default: /tmp/supysonic
cache_dir = /var/supysonic/cache

; Main cache max size in MB. Default: 512
cache_size = 512

; Transcode cache max size in MB. Default: 1024 (1GB)
transcode_cache_size = 1024

; Optional rotating log file. Default: none
log_file = /var/log/supysonic.log

; Log level. Possible values: DEBUG, INFO, WARNING, ERROR, CRITICAL.
; Default: WARNING
log_level = WARNING

; Enable the Subsonic REST API. You'll most likely want to keep this on, here
; for testing purposes. Default: on
;mount_api = on

; Enable the administrative web interface. Default: on
;mount_webui = on

[daemon]
; Socket file the daemon will listen on for incoming management commands
; Default: /tmp/supysonic/supysonic.sock
socket = /var/run/supysonic.sock

; Defines if the file watcher should be started. Default: yes
run_watcher = yes

; Delay in seconds before triggering scanning operation after a change have been
; detected.
; This prevents running too many scans when multiple changes are detected for a
; single file over a short time span. Default: 5
wait_delay = 5

; Command used by the jukebox
jukebox_command = mplayer -ss %offset %path

; Optional rotating log file for the scanner daemon. Logs to stderr if empty
log_file = /var/supysonic/supysonic-daemon.log
log_level = INFO

[lastfm]
; API and secret key to enable scrobbling. http://www.last.fm/api/accounts
; Defaults: none
;api_key =
;secret =

[transcoding]
; Programs used to convert from one format/bitrate to another. Defaults: none
transcoder_mp3_mp3 = lame --quiet --mp3input -b %outrate %srcpath -
transcoder = ffmpeg -i %srcpath -ab %outratek -v 0 -f %outfmt -
decoder_mp3 = mpg123 --quiet -w - %srcpath
decoder_ogg = oggdec -o %srcpath
decoder_flac = flac -d -c -s %srcpath
encoder_mp3 = lame --quiet -b %outrate - -
encoder_ogg = oggenc2 -Q -M %outrate -

; Default format, used when a client requests a bitrate lower than the original
; file and no specific format
default_transcode_target = mp3

[mimetypes]
; Extension to mimetype mappings in case your system has some trouble guessing
; Default: none
;mp3 = audio/mpeg
;ogg = audio/vorbis

```

Créer une base mysql **supysonic** utilisateur *supysonic* ayant tous les droits

    # debian
    mysql -u root -p$(cat /etc/mysql/mdp) -e "CREATE DATABASE supysonic; GRANT ALL ON supysonic.* TO 'supysonic'@'localhost' IDENTIFIED BY '8gKwwF5yVu'; FLUSH PRIVILEGES;"
    # archlinux
    Créer la base mysql pour supysonic
    mysql -u root -p$(cat /home/yannick/Private/mdp/mariadb/root) -e "CREATE DATABASE supysonic; GRANT ALL ON supysonic.* TO 'supysonic'@'localhost' IDENTIFIED BY '8gKwwF5yVu'; FLUSH PRIVILEGES;"

## How to Implement CRUD Using Ajax and Json

* [How to Implement CRUD Using Ajax and Json](https://simpleisbetterthancomplex.com/tutorial/2016/11/15/how-to-implement-a-crud-using-ajax-and-json.html)

On se rend dans le répertoire **/home/yannick/media/dplus/python-dev** 

    cd /home/yannick/media/dplus/python-dev
    git clone https://github.com/sibtc/simple-ajax-crud 

Aller dans le répertoire simple-ajax-crud et créer environnement 'simple-ajax-crud' puis l'activer

    cd simple-ajax-crud
    python -m venv simple-ajax-crud
    source simple-ajax-crud/bin/activate

(simple-ajax-crud) [yannick@yannick-pc simple-ajax-crud]$   

La façon la plus simple de trouver la valeur correcte à définir pour "Wing Personal" est de lancer votre Python virtualenv à l’extérieur de Wing et d’exécuter ce qui suit de manière interactive :

```bash
(simple-ajax-crud) [yannick@yannick-pc simple-ajax-crud]$ python
Python 3.8.2 (default, Apr  8 2020, 14:31:25) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.executable
'/home/yannick/media/dplus/python-dev/simple-ajax-crud/simple-ajax-crud/bin/python'
>>> quit()
```

Installation 

Changer les versions dans **requirements.txt**

```
Django==3.0
django-widget-tweaks==1.4.8
```

Installer

    pip install -r requirements.txt
    python manage.py migrate
    python manage.py runserver

lien 127.0.0.1:8000


## flask-crud-app

[flask-crud-app](https://github.com/macagua/flask-crud-app)

On se rend dans le répertoire **/home/yannick/media/dplus/python-dev** 

    cd /home/yannick/media/dplus/python-dev
    git clone https://github.com/macagua/flask-crud-app

Aller dans le répertoire flask-crud-appet créer environnement 'simple-ajax-crud' puis l'activer

    cd flask-crud-app
    python -m venv flask-crud-app
    source flask-crud-app/bin/activate

(simple-ajax-crud) [yannick@yannick-pc simple-ajax-crud]$   

Valeur correcte à définir pour "Wing Personal" , lancer votre Python virtualenv à l’extérieur de Wing et d’exécuter ce qui suit de manière interactive :

```bash
(flask-crud-app) [yannick@yannick-pc flask-crud-app]$ python
Python 3.8.2 (default, Apr  8 2020, 14:31:25) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.executable
'/home/yannick/media/dplus/python-dev/flask-crud-app/flask-crud-app/bin/python'
>>> quit()
```

On met à jour pip

    pip install --upgrade pip

Installer

    pip install -r requirements.txt

Exécution

    python bookmanager.py

```
 * Serving Flask app "bookmanager" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://0.0.0.0:8087/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 265-845-360
```


Lien http://127.0.0.1:8087

## How to Use JWT Authentication with Django REST Framework

JWT signifie JSON Web Token et c'est une stratégie d'authentification utilisée par les applications client/serveur où le client est une application Web utilisant JavaScript et un cadre frontal comme Angular, React ou VueJS.

Dans ce tutoriel, nous allons explorer les spécificités de l'authentification JWT. Si vous souhaitez en savoir plus sur l'authentification par jeton à l'aide du Django REST Framework (DRF), ou si vous voulez savoir comment démarrer un nouveau projet DRF, vous pouvez lire ce tutoriel : [Comment mettre en œuvre l'authentification par jeton en utilisant Django REST Framework (anglais)](https://simpleisbetterthancomplex.com/tutorial/2018/11/22/how-to-implement-token-authentication-using-django-rest-framework.html). Les concepts sont les mêmes, nous allons juste changer le backend d'authentification.

* [How to Use JWT Authentication with Django REST Framework](https://simpleisbetterthancomplex.com/tutorial/2018/12/19/how-to-use-jwt-authentication-with-django-rest-framework.html)
* <https://github.com/sibtc/drf-jwt-example>


## Basic Django CRUD using AJAX, JSON, and JQuery – Monolithic Architecture

En génie logiciel, une application monolithique décrit une application logicielle à un seul niveau dans laquelle l'interface utilisateur et le code d'accès aux données sont combinés en un seul programme à partir d'une seule plate-forme.

Une application monolithique est autonome et indépendante des autres applications informatiques.

* [Basic Django CRUD using AJAX, JSON, and JQuery – Monolithic Architecture](https://pinoylearnpython.com/basic-django-crud-using-ajax-json-and-jquery-monolithic-architecture/)
* <https://github.com/pinoylearnpython/dev>

## sublime-music

https://sumner.gitlab.io/sublime-music/

Environnement

```
cd /home/yannick/media/dplus/python-dev
python -m venv sublime-music
source sublime-music/bin/activate
pip install --upgrade pip
```

Pour éviter l'erreur  **Failed to build PyGObject**

```
 ERROR: Command errored out with exit status 1:
   command: /home/yannick/media/dplus/python-dev/sublime-music/bin/python /home/yannick/media/dplus/python-dev/sublime-music/lib/python3.8/site-packages/pip/_vendor/pep517/_in_process.py build_wheel /tmp/tmpp619_l9p
       cwd: /tmp/pip-install-z_bmujb7/PyGObject
  Complete output (43 lines):
[...]
  Package gobject-introspection-1.0 was not found in the pkg-config search path.
  Perhaps you should add the directory containing `gobject-introspection-1.0.pc'
  to the PKG_CONFIG_PATH environment variable
  Package 'gobject-introspection-1.0', required by 'virtual:world', not found
  Command '('pkg-config', '--print-errors', '--exists', 'gobject-introspection-1.0 >= 1.46.0')' returned non-zero exit status 1.
  ----------------------------------------
  ERROR: Failed building wheel for PyGObject
Failed to build PyGObject
ERROR: Could not build wheels for PyGObject which use PEP 517 and cannot be installed directly
```

il faut installer **gobject-introspection** (gobject-introspection-runtime ne suffit pas)  

    yay -S gobject-introspection

Installer

    pip install sublime-music


