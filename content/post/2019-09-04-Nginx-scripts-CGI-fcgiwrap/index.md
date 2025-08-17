+++
title = 'Nginx, exécuter des scripts CGI avec fcgiwrap'
date = 2019-09-04 00:00:00 +0100
categories = ['serveur']
+++
## Comment exécuter des scripts CGI avec fcgiwrap

Origine : Milosz Galazka sur 18 septembre 2017 

### FastCGI

Installez le paquet fcgiwrap .

    sudo apt-get install fcgiwrap

Le service sera lancé immédiatement.

    systemctl status fcgiwrap.service 

```
● fcgiwrap.service - Simple CGI Server
   Loaded: loaded (/lib/systemd/system/fcgiwrap.service; indirect; vendor preset: enabled)
   Active: active (running) since Wed 2019-09-04 09:55:16 CEST; 13s ago
 Main PID: 3880 (fcgiwrap)
    Tasks: 1 (limit: 4915)
   Memory: 364.0K
   CGroup: /system.slice/fcgiwrap.service
           └─3880 /usr/sbin/fcgiwrap -f

sept. 04 09:55:16 srvxo systemd[1]: Started Simple CGI Server.
```

    systemctl status fcgiwrap.socket 

```
● fcgiwrap.socket - fcgiwrap Socket
   Loaded: loaded (/lib/systemd/system/fcgiwrap.socket; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2019-09-04 09:55:16 CEST; 39s ago
   Listen: /run/fcgiwrap.socket (Stream)
    Tasks: 0 (limit: 4915)
   Memory: 0B
   CGroup: /system.slice/fcgiwrap.socket

sept. 04 09:55:16 srvxo systemd[1]: Listening on fcgiwrap Socket.
```

Le service sera automatiquement configuré pour démarrer automatiquement.

```
systemctl is-enabled fcgiwrap.service 
indirect

systemctl is-enabled fcgiwrap.socket
enabled
```

Les fichiers de configuration de l'unité systemd sont très simples et directs.

    cat /lib/systemd/system/fcgiwrap.socket 

```
[Unit]
Description=fcgiwrap Socket

[Socket]
ListenStream=/run/fcgiwrap.socket

[Install]
WantedBy=sockets.target
```

	cat /lib/systemd/system/fcgiwrap.service 

```
[Unit]
Description=Simple CGI Server
After=nss-user-lookup.target
Requires=fcgiwrap.socket

[Service]
Environment=DAEMON_OPTS=-f
EnvironmentFile=-/etc/default/fcgiwrap
ExecStart=/usr/sbin/fcgiwrap ${DAEMON_OPTS}
User=www-data
Group=www-data

[Install]
Also=fcgiwrap.socket
```

Lisez la page de manuel **fcgiwrap** pour définir ou modifier les processus de numérotation à préforger et une URL à laquelle le socket d'écoute doit se lier.

### Installer et configurer nginx

Installez le paquet nginx .

    sudo apt-get install --no-install-recommends nginx

Désactiver le site nginx par défaut.

    sudo unlink /etc/nginx/sites-enabled/default

#### Utiliser un seul script CGI

Créer le répertoire racine du document.

    sudo mkdir -p /var/www/localhost/

Définissez le propriétaire et le groupe pour le répertoire racine du document.

    sudo chown www-data:www-data /var/www/localhost/

Créer un script CGI.

```
$ cat << EOF | sudo tee /var/www/localhost/index.cgi
#!/bin/bash

echo "Content-type: text/html"
echo ""

echo "<html>"
echo "<head>"
echo "<meta http-equiv=\"Content-Type\" content=\"text/html; charset=UTF-8\">"
echo "<title>Bash CGI script</title>"
echo "</head>"
echo "<body>"
echo "<p>Hello, Your IP address is \$REMOTE_ADDR</p>"
echo "<pre>"
env
echo "</pre>"
echo "</body>"
echo "</html>"

exit 0
EOF
```

Définissez le propriétaire et le groupe pour le script CGI créé.

    sudo chown www-data:www-data /var/www/localhost/index.cgi

Définir le bit exécutable.

    sudo chmod +x /var/www/localhost/index.cgi

Préparez la configuration de l'hôte virtuel nginx .

```
$ cat << EOF | sudo tee /etc/nginx/sites-available/cgi.example.org
server {
  listen 80 default_server;
  server_name default;

  location / {
    index index.cgi;
    root /var/www/localhost;
  }

  location /index.cgi {
    root /var/www/localhost;

    fastcgi_intercept_errors on;
    include fastcgi_params;

    #fastcgi_param DOCUMENT_ROOT \$document_root;
    #fastcgi_param SCRIPT_NAME   \$fastcgi_script_name;
    fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;

    fastcgi_pass unix:/var/run/fcgiwrap.socket;
  }
}
EOF
```

Activer l'hôte virtuel.

    sudo ln -s /etc/nginx/sites-available/cgi.example.org /etc/nginx/sites-enabled/

Rechargez la configuration du serveur HTTP.

    sudo systemctl reload nginx

Ouvrez le navigateur Web et pointez sur le serveur HTTP configuré.

### Utiliser plusieurs scripts CGI

Créez le répertoire cgi-bin .

    sudo mkdir -p /usr/lib/cgi-bin

Définissez le propriétaire et le groupe pour le répertoire cgi-bin .

    sudo chown www-data:www-data /usr/lib/cgi-bin

Créer un script CGI.

```
$ cat << EOF | sudo tee /usr/lib/cgi-bin/bashcgiscript.cgi
#!/bin/bash

echo "Content-type: text/html"
echo ""

echo "<html>"
echo "<head>"
echo "<meta http-equiv=\"Content-Type\" content=\"text/html; charset=UTF-8\">"
echo "<title>Bash CGI script</title>"
echo "</head>"
echo "<body>"
echo "<p>Hello, Your IP address is \$REMOTE_ADDR</p>"
echo "<pre>"
env
echo "</pre>"
echo "</body>"
echo "</html>"

exit 0
EOF
```

Définissez le propriétaire et le groupe pour le script CGI créé.

    sudo chown www-data:www-data /usr/lib/cgi-bin/bashcgiscript.cgi

Définir le bit exécutable.

    sudo chmod +x /usr/lib/cgi-bin/bashcgiscript.cgi

Copiez la configuration nginx par défaut pour l'emplacement **/cgi-bin/**

    sudo cp /usr/share/doc/fcgiwrap/examples/nginx.conf /etc/nginx/fcgiwrap.conf

Préparez la configuration de l'hôte virtuel nginx .

```
$ cat << EOF | sudo tee /etc/nginx/sites-available/cgi.example.org
server {
  listen 80 default_server;
  server_name default;

  include fcgiwrap.conf;

  location / {
    index index.html;
    root  /var/www/;
  }
}
EOF
```

Activer l'hôte virtuel.

    sudo ln -s /etc/nginx/sites-available/cgi.example.org /etc/nginx/sites-enabled/

Rechargez la configuration du serveur HTTP.

    sudo systemctl reload nginx

Ouvrez le navigateur Web et pointez sur le **/cgi-bin/bashcgiscript.cgi** CGI script sur le serveur HTTP configuré.

### Notes complémentaires

Il s'agit de la configuration par défaut de l'emplacement cgi-bin utilisé dans l'exemple précédent.

    cat /usr/share/doc/fcgiwrap/examples/nginx.conf

```
# Include this file on your nginx.conf to support debian cgi-bin scripts using
# fcgiwrap
location /cgi-bin/ { 
  # Disable gzip (it makes scripts feel slower since they have to complete
  # before getting gzipped)
  gzip off;

  # Set the root to /usr/lib (inside this location this means that we are
  # giving access to the files under /usr/lib/cgi-bin)
  root	/usr/lib;

  # Fastcgi socket
  fastcgi_pass  unix:/var/run/fcgiwrap.socket;

  # Fastcgi parameters, include the standard ones
  include /etc/nginx/fastcgi_params;

  # Adjust non standard parameters (SCRIPT_FILENAME)
  fastcgi_param SCRIPT_FILENAME  /usr/lib$fastcgi_script_name;
}
```

### Références


*    [Web CGI with Bash scripts](http://www.yolinux.com/TUTORIALS/BashShellCgi.html)
*    [Common Gateway Interface](https://en.wikipedia.org/wiki/Common_Gateway_Interface)
*    [systemd.socket — Socket unit configuration](https://www.freedesktop.org/software/systemd/man/systemd.socket.html)
*    [systemd.service — Service unit configuration](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
