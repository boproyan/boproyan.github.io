+++
title = 'Déployer des applications Flask (python) avec uWSGI et Nginx'
date = 2020-04-23 00:00:00 +0100
categories = python
+++
![Texte alternatif](uwsgi_nginx_python.png){:width="200"}

## Mise en place de l'uWSGI avec Nginx

Avant de commencer à utiliser NGinx, vous devez effectuer une mise à jour de votre version d'Ubuntu.

```
sudo apt-get update
sudo apt-get install python-dev python-pip nginx
```

Ensuite, configurez votre environnement virtuel :

```
sudo pip install virtualenv
source appenv/bin/activate
```

Après avoir complété ce qui précède et configuré votre environnement virtuel, nous pouvons commencer à configurer NGinx. Tous les paquets que nous installons ici n'existeront qu'à l'intérieur de l'environnement virtuel. Non, nous pouvons installer l'uWSGI dans l'environnement :

```
pip install uwsgi
```

Vérifiez l'installation en tapant :

```
uwsgi --version
```

Nous pouvons maintenant créer l'application wsgi.py ou le point d'entrée :

```
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return ["<h1 style='color:blue'>Hello There!</h1>"]
```

Le serveur uWSGI recherche une fonction appelable appelée "application" par défaut. Nous pouvons maintenant tester, exécuter la commande suivante pour démarrer le serveur WSGI :

```
uwsgi --socket 0.0.0.0:8080 --protocol=http -w wsgi
```

Maintenant, visitez l'adresse IP du serveur (je suis en train d'exécuter cette instance sur EC2 - assurez-vous que vous autorisez le port 8080 dans le groupe de sécurité, sinon vous ne pourrez pas accéder à la page) et vérifiez l'accès. Dans ce qui précède, nous avons fourni manuellement la commande permettant de faire fonctionner le serveur et de passer nos paramètres depuis la ligne de commande. Comme terraform, nous pouvons (quelque peu) automatiser ce processus en utilisant un fichier de configuration :

```
nano ~/python-microservices/building_microservices/app.ini
```

Dans notre fichier de configuration, nous créons une section `[uwsgi]` et la faisons pointer vers notre application :

```
[uwsgi]
module = wsgi:application
```

Le premier processus exécuté sur le serveur wsgi sera le "master process", tous les autres processus exécutés par la suite seront des "worker processes"

```
master = true
processes = 5
```

Dans la commande que j'ai utilisée pour faire fonctionner le serveur uWSGI, j'ai précisé qu'il utilisait le protocole http. Nous allons installer NGinx devant uWSGI (Nginx -> uWSGI -> Microservice) et par conséquent, changer le protocole utilisé. NGinx est livré avec un mécanisme uWSGI prêt à l'emploi. En omettant toute spécification de protocole dans la commande uWSGI, nous forçons en fait l'uWSGI à se replier sur le protocole uWSGI par défaut. De plus, comme nous utilisons cette configuration avec NGinx sur la même machine, nous pouvons configurer un socket au lieu d'un port. Cela nous permettra d'obtenir des résultats plus rapides. J'attribue les permissions du socket 664 (accès utilisateur et lecture/écriture, accès groupe en lecture). J'utiliserai également l'option de vide - permettant de libérer à nouveau la socket une fois que le processus s'est arrêté.

```
socket = myapp.sock
chmod-socket = 664
vacuum = true
```

Enfin, nous allons démarrer l'application au démarrage avec un fichier upstart. Upstart et uWSGI utilisent tous deux SIGTERM mais pour des choses différentes. Pour contourner ce problème, nous ajoutons une option appelée "die-on-term" pour empêcher l'uWSGI de redémarrer le programme de manière inattendue.

Ainsi, mon fichier de configuration ressemble à ceci :

```
master = true
processes = 5

socket = myapp.sock
chmod-socket = 664
vacuum = true

die-on-term = true
```

Cela étant fait, nous pouvons créer un script de démarrage pour lancer une application uWSGI au moment du démarrage, en nous assurant que notre application est toujours disponible.

```
sudo nano /etc/init/myapp.conf
```

Nous pouvons commencer par une description et des niveaux de fonctionnement du système.

```
description "uWSGI instance to serve myapp"

start on runlevel [2345]
stop on runlevel [!2345]
```

Nous pouvons maintenant spécifier les informations de l'utilisateur pour exécuter le processus. Nous allons également spécifier "www-data" (c'est un compte utilisateur dans Nginx) pour que Nginx puisse parler à la socket définie dans notre fichier .ini.

```
setuid demo
setgid www-data
```

Ensuite, nous utiliserons un bloc de script pour activer l'uWSGI dans l'environnement virtuel. Cela facilite l'accès aux logiciels qui sont également configurés dans l'environnement virtuel.

```
script
    cd /home/demo/myapp
    . myappenv/bin/activate
    uwsgi --ini myapp.ini
end script
```

Le produit fini doit ressembler à ceci (remplacez votre propre nom d'utilisateur évidemment) :

```
description "uWSGI instance to serve myapp"

start on runlevel [2345]
stop on runlevel [!2345]

setuid demo
setgid www-data

script
    cd /home/demo/myapp
    . myappenv/bin/activate
    uwsgi --ini myapp.ini
end script
```

Nous devrions maintenant être en mesure de lancer le service en tapant simplement

```
sudo start myapp
```

### Mise en place de Nginx

Nous allons configurer NGinx comme un "proxy inversé", qui prendra les requêtes http et les redirigera vers le socket uWSGI, puis éventuellement vers notre application. Naviguez jusqu'au répertoire suivant et créez un fichier appelé "myapp" ou comme vous préférez l'appeler. Il s'agit de notre fichier de configuration pour Nginx. Dans ce cas, il sera très rudimentaire :

```
server {
    listen 80;
    server_name server_domain_or_IP;

    location / {
      include         uwsgi_params;
      uwsgi_pass      unix:/home/demo/myapp/myapp.sock;
  }
}
```

Le bloc de localisation qui commence par "/" dans la configuration ci-dessus, achemine en gros tout le trafic au même endroit (uwsgi --> myapp). Liez le fichier de configuration aux sites disponibles avec la commande suivante :

```
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled
```

![app_available](https://s3-ap-southeast-2.amazonaws.com/innablr/app_available.PNG)

Il existe également une commande permettant de tester toute erreur de syntaxe dans votre fichier de configuration :

```
sudo service nginx configtest
```

![config_ok](https://s3-ap-southeast-2.amazonaws.com/innablr/config_ok.PNG)

Si le message de configuration est correct, redémarrez le serveur et rafraîchissez l'adresse du serveur dans votre navigateur web, mais sans le numéro de port, et cela devrait fonctionner normalement.

![welcome_to_nginx](https://s3-ap-southeast-2.amazonaws.com/innablr/welcome_to_nginx.PNG)



## Déployer des applications Flask (python) avec uWSGI et Nginx

*Article original : [Deploy Flask Applications with uWSGI and Nginx](https://hackersandslackers.com/deploy-flask-uwsgi-nginx/)*  

Nous devons installer un tas de paquets de développement Python sur Ubuntu (ou autre) pour que l'uWSGI fonctionne. Même si cette ligne vous semble familière, ne sautez pas cette partie (comme je l'ai fait). Il est presque certain qu'il vous manque au moins un paquet en dessous :

```
$ apt update
$ apt upgrade -y
$ sudo apt install python3.8 python3.8-dev python3-distutils uwsgi uwsgi-src uuid-dev libcap-dev libpcre3-dev python3-pip python3.8-venv
```

### Installer toutes les dépendances de dev Python.

L'installation de uwsgi-plugin-python3 est une étape importante qui mérite une attention particulière. uWSGI est traditionnellement un paquet Python, vous pouvez donc vous attendre à ce que nous lancions pip install uWSGI à un moment donné. Au contraire, si nous devions installer uWSGI via pip, uWSGI serait un paquet Python appartenant à la version par défaut de Python3 qui se trouve être installée sur notre machine. Notre projet va probablement utiliser une version de Python autre que Python 3.6.9 (la version par défaut d'Ubuntu 18.04). Nous avons donc besoin d'une version de l'uWSGI qui transcende les versions de Python. C'est là qu'intervient uwsgi-plugin-python3 :

    $ apt-get install uwsgi-plugin-python3

Installez le plugin Python uWSGI.

Notre dernière étape de configuration Ubuntu consiste à ouvrir le port 5000 :

    $ ufw allow 5000

Port ouvert 5000.

### Préparez votre projet

Ensuite, nous devons juste nous assurer que votre application **Flask** est sur votre serveur distant et prête à l'emploi. Clonez votre projet sur votre VPS et assurez-vous que votre projet possède un fichier wsgi.py approprié pour servir de point d'entrée à l'application :

```
from myapp import create_app

app = create_app()

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```

wsgi.py

Vous savez déjà qu'il faut utiliser des environnements virtuels sans que je vous le dise, mais nous devons utiliser spécifiquement virtualenv ici - PAS Pipenv ou toute autre alternative. uWSGI est difficile à ce sujet pour une raison quelconque. Épargnez-vous la peine et utilisez virtualenv, même si c'est un peu nul par principe :

```
$ python3.8 -m venv myenv
$ source myenv/bin/activate
$ python3 -m pip install -r requirements.txt
```

### Créez et construisez votre environnement virtuel.

Nous avons installé toutes les dépendances, installé l'uWSGI, et notre projet se présente bien... nous devrions être prêts à tester ce truc, non ?

    uwsgi --http-socket :5000 --plugin python3 --module wsgi:app

Tentative de ligne de commande de l'uWSGI.

Surprise, surprise !

```
Python version: 3.6.9 (default, Nov  7 2019, 10:44:02)  [GCC 8.3.0]
*** Python threads support is disabled. You can enable it with --enable-threads ***
Python main interpreter initialized at 0x55721b8125c0
dropping root privileges after plugin initialization
uWSGI running as root, you can use --uid/--gid/--chroot options
*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 72768 bytes (71 KB) for 1 cores
*** Operational MODE: single process ***
Traceback (most recent call last):
  File "wsgi.py", line 1, in <module>
    from pythonmyadmin import create_app
  File "./pythonmyadmin/__init__.py", line 1, in <module>
    from flask import Flask
ModuleNotFoundError: No module named 'flask'
unable to load app 0 (mountpoint='') (callable not found or import error)
*** no app loaded. going in full dynamic mode ***
```

Les résultats de notre test WSGI.

Vous pouvez peut-être discerner ce qui se passe ci-dessus ; la sortie du WSGI indique que nous utilisons Python 3.6.9 (et non ce que nous voulons) et que nous ne pouvons pas trouver les paquets associés à notre environnement virtuel activé. Lorsque nous avons spécifié --plugin python3 dans la ligne précédente, nous étions trop généraux : nous avons besoin d'un plugin uWSGI spécifique pour notre version, qui est appelée python38.

### Installer le plugin uWSGI Python 3.8

Grâce à la bibliothèque **uwsgi-plugin-python3** que nous avons installée plus tôt, l'installation de plugins uWSGI spécifiques à une version est facile :

    $ export PYTHON=python3.8
    $ uwsgi --build-plugin "/usr/src/uwsgi/plugins/python python38"

Vous devriez voir le résultat comme ceci :

```
*** uWSGI building and linking plugin from /usr/src/uwsgi/plugins/python ***
[x86_64-linux-gnu-gcc -pthread] python38_plugin.so
/usr/src/uwsgi/plugins/python/python_plugin.c: In function ‘uwsgi_python_post_fork’:
/usr/src/uwsgi/plugins/python/python_plugin.c:394:3: warning: ‘PyOS_AfterFork’ is deprecated [-Wdeprecated-declarations]
   PyOS_AfterFork();
   ^~~~~~~~~~~~~~
In file included from /usr/include/python3.8/Python.h:144:0,
                 from /usr/src/uwsgi/plugins/python/uwsgi_python.h:2,
                 from /usr/src/uwsgi/plugins/python/python_plugin.c:1:
/usr/include/python3.8/intrcheck.h:18:37: note: declared here
 Py_DEPRECATED(3.7) PyAPI_FUNC(void) PyOS_AfterFork(void);
                                     ^~~~~~~~~~~~~~
/usr/src/uwsgi/plugins/python/python_plugin.c: In function ‘uwsgi_python_worker’:
/usr/src/uwsgi/plugins/python/python_plugin.c:1957:3: warning: ‘PyOS_AfterFork’ is deprecated [-Wdeprecated-declarations]
   PyOS_AfterFork();
   ^~~~~~~~~~~~~~
In file included from /usr/include/python3.8/Python.h:144:0,
                 from /usr/src/uwsgi/plugins/python/uwsgi_python.h:2,
                 from /usr/src/uwsgi/plugins/python/python_plugin.c:1:
/usr/include/python3.8/intrcheck.h:18:37: note: declared here
 Py_DEPRECATED(3.7) PyAPI_FUNC(void) PyOS_AfterFork(void);
                                     ^~~~~~~~~~~~~~
build time: 6 seconds
*** python38 plugin built and available in python38_plugin.so ***
```

Ceci télécharge un fichier appelé python38_plugin.so dans votre dossier actuel. Nous devons le déplacer là où il se trouve et définir certaines autorisations :

    $ sudo mv python38_plugin.so /usr/lib/uwsgi/plugins/python38_plugin.so
    $ sudo chmod 666 /usr/lib/uwsgi/plugins/python38_plugin.so

### Essayons encore une fois

Cette fois-ci, nous allons spécifier `--plugin python38` pour faire fonctionner spécifiquement uWSGI avec Python 3.8. Nous allons également ajouter un autre drapeau appelé `--virtualenv`, qui définit le chemin d'accès où nos bibliothèques Python sont installées. Nous allons supprimer le processus uWSGI précédent et lui donner une nouvelle chance :

    $ pkill -9 uwsgi
    $ uwsgi --http-socket :5000 --plugin python38 --module wsgi:app  --virtualenv /var/www/pythonmyadmin/myenv/

Nous y voilà...

```
*** Python threads support is disabled. You can enable it with --enable-threads ***
Python main interpreter initialized at 0x55daa18a3e40
dropping root privileges after plugin initialization
uWSGI running as root, you can use --uid/--gid/--chroot options
*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 72768 bytes (71 KB) for 1 cores
*** Operational MODE: single process ***
WSGI app 0 (mountpoint='') ready in 1 seconds on interpreter 0x55daa18a3e40 pid: 28434 (default app)
dropping root privileges after application loading
uWSGI running as root, you can use --uid/--gid/--chroot options
*** WARNING: you are running uWSGI as root !!! (use the --uid flag) ***
*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI worker 1 (and the only) (pid: 28434, cores: 1)
```

C'est une bonne nouvelle ! Visitez l'adresse IP de votre serveur au port 5000 et faites un rapport. Est-ce que ça fonctionne ? ÇA MARCHE ! BIEN ! !!

Ce qui est génial avec l'uWSGI, c'est la facilité avec laquelle on peut utiliser plusieurs cœurs dans notre machine en spécifiant le nombre de threads et de processus que l'on veut utiliser. Si votre machine est équipée de plusieurs cœurs de processeur, voici à quel point il est facile de les utiliser :

```
*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI worker 1 (pid: 28688, cores: 2)
spawned uWSGI worker 2 (pid: 28690, cores: 2)
spawned uWSGI worker 3 (pid: 28691, cores: 2)
spawned uWSGI worker 4 (pid: 28692, cores: 2)
```

Gardez à l'esprit que les processus uWSGI ne sont pas tués par un simple Contrôle+C dans le terminal. N'oubliez pas de tuer les processus uWSGI non désirés en utilisant `pkill -9 uwsgi`.

### Exécution d'uWSGI via le fichier de configuration

Nous avons prouvé que nous pouvons servir notre application via le CLI uWSGI, mais nous voulons que notre application persiste pour toujours derrière un nom de domaine enregistré. Nous avons besoin d'un moyen pour Nginx de se connecter à notre processus uWSGI avec tous les drapeaux que nous avons passés via le CLI (tels que le plugin uWSGI à utiliser, l'emplacement de notre environnement virtuel, etc.) ). Heureusement, nous pouvons enregistrer les drapeaux/valeurs que nous avons transmis dans le CLI dans un fichier **.ini** avec la même convention de nommage :

```
[uwsgi]
chdir = /var/www/myapp/
module = wsgi:app

processes = 4
threads = 2
plugin = python38
virtualenv = /var/www/myapp/myappenv

master = true
socket = myapp.sock
chmod-socket = 666
vacuum = true

die-on-term = true
```

myapp.ini

Au lieu de spécifier http-socket ici, nous mettons `socket` à **myapp.sock**. Nginx va traiter nos requêtes HTTP, mais il a besoin d'un moyen d'associer les requêtes entrantes à notre application en cours d'exécution. Nous gérons cela en créant une socket : chaque fois que nous exécutons uWSGI avec cette configuration, un fichier est créé dans notre répertoire de projet appelé **myapp.sock**. Tout ce dont Nginx doit se préoccuper, c'est de pointer vers cette socket pour le trafic entrant.

Nous pouvons maintenant exécuter notre application avec la bonne configuration de manière efficace :

    $ uwsgi myapp.ini

C'est beaucoup mieux. En prime, la présence de `die-on-term = true` dans notre configuration signifie que notre processus uWSGI se terminera lorsque nous ferons Control+C, pour des raisons de commodité.

## uWSGI & Nginx 4 Eva

En supposant que vous ayez installé Nginx, créez une configuration pour notre application dans les sites disponibles :

    $ sudo vim /etc/nginx/sites-available/myapp.conf

Il s'agit peut-être de l'une des configurations Nginx les plus simples que vous aurez à créer. Écoutez votre domaine sur le port 80 et transférez ce trafic (avec des paramètres) à l'emplacement du fichier socket que nous avons spécifié dans **myapp.ini** :

```
server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:///var/www/myapp/myapp.sock;
    }
}
```

Notez les triples barres obliques dans l'URI `uwsgi_pass`.

Faisons un lien symbolique entre cette configuration et sites-enabled :

    $ ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/myapp.conf

Redémarrez Nginx pour que cela prenne effet :

    sudo service nginx restart

### Sceller l'accord (Seal the Deal)

Vous êtes à une commande d'exposer le déploiement de votre application au monde entier pour toujours. Avant d'appuyer sur la gâchette, soyez fier de votre réussite. Toute l'industrie du cloud computing a massivement profité du paradigme des micro-services "sans serveur" parce que la plupart des gens ne sont pas comme vous. La plupart des gens préfèrent payer des frais pour des ressources partagées au détriment de la performance pour éviter de lire un tutoriel comme celui-ci. Vous êtes libéré. Vous êtes belle. Vous êtes Batman.

    $ cd /var/www/myapp
    $ nohup uwsgi myapp.ini &

`nohup` est une commande unix qui permet aux processus de persister en arrière-plan jusqu'à ce qu'ils soient explicitement tués (ou que votre machine s'éteigne). L'exécution de nohup uwsgi myapp.ini & (notez l'esperluette qui suit) fera tourner un processus uwsgi qui reste en vie pendant que vous vous occupez de vos affaires.


## Gérer les applications uWSGI en mode empereur

Si vous prévoyez d'héberger une tonne d'applications Python sur votre serveur, faire tourner chaque application en utilisant nohup uwsgi etc. sera ennuyeux. Ne serait-il pas agréable de pouvoir gérer toutes nos applications uWSGI de manière globale, comme NGINX gère les hôtes ? C'est tout à fait possible !  **cd /etc/uwsgi** et regardez ce qu'il y a à l'intérieur :

*    **/apps-available** : Un dossier global pour contenir tous vos fichiers uwsgi .ini (comme celui que nous avons créé précédemment). C'est l'équivalent uWSGI du dossier sites-available de Nginx.
*    **/apps-enabled** : Tout comme Nginx, ce dossier attend des liens symboliques de son homologue apps-available. L'exécution de l'uWSGI en mode empereur recherchera tous les fichiers de configuration dans ce dossier et les exécutera en conséquence.

Lançons notre application en mode empereur ! Copiez votre config dans apps-available et mettez un lien symbolique vers apps-enabled :

    $ sudo cp /var/www/myapp/myapp.ini /etc/uwsgi/apps-available/myapp.ini
    $ sudo ln -s /etc/uwsgi/apps-available/myapp.ini /etc/uwsgi/apps-enabled/myapp.ini

Maintenant, faites le tour :

    $ sudo uwsgi --emperor /etc/uwsgi/apps-enabled/

### Démarrer sur la mise en route de la machine

Le meilleur aspect de l'uWSGI en mode empereur est que nous pouvons lancer nos applications au démarrage de la machine sans avoir à écrire de services. Ajoutez un fichier appelé /etc/rc.local et incluez ce qui suit :

    /usr/local/bin/uwsgi --emperor /etc/uwsgi/sites-enabled --daemonize /var/log/uwsgi-emperor.log

