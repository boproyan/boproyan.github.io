+++
title = 'Debian , compilation et installation nginx OU openresty (nginx + lua + openssl TLSv1.3 + modules dynamiques) + PHP7.3 + MariaDb'
date = 2019-09-11 00:00:00 +0100
categories = ['serveur']
+++
# Compilation Nginx OU Openresty sur Debian Buster

><font color="red"><b>ATTENTION !!!<br>Les compilations se font sur une base "Debian Buster" 
pour valider le TLS1.3</b></font>

## Compilation nginx Debian Buster


### Les modules ajoutés à la compilation nginx


 * [Headers More : Set and clear input and output headers... more than “add”!(openresty/headers-more-nginx-module)](https://github.com/openresty/headers-more-nginx-module)  
 * [PAM Authentication : HTTP Basic Authentication using PAM (stogh/ngx_http_auth_pam_module)](https://github.com/stogh/ngx_http_auth_pam_module)  
 * [Cache Purge : Adds ability to purge content from FastCGI, proxy, and uWSGI caches (FRiCKLE/ngx_cache_purge)](https://github.com/FRiCKLE/ngx_cache_purge)  
 * [Development Kit : An extension to the core functionality of NGINX (simpl/ngx_devel_kit)](https://github.com/simpl/ngx_devel_kit nginx-development-kit)  
 * [HTTP Echo : Provides familiar shell-style commands to NGINX HTTP servers (openresty/echo-nginx-module)](https://github.com/openresty/echo-nginx-module)  
 * [Fancy Index : Like the built-in autoindex module, but fancier (aperezdc/ngx-fancyindex)](https://github.com/aperezdc/ngx-fancyindex)  
 * [Nchan ex HTTP Push Stream ](https://github.com/slact/nchan)  
 * [HTTP Lua : Embed the power of Lua into NGINX HTTP servers.(openresty/lua-nginx-module)](https://github.com/openresty/lua-nginx-module.git)  
 * [NGINX Upload Progress Module : Tracks and reports upload progress (masterzen/nginx-upload-progress-module)](https://github.com/masterzen/nginx-upload-progress-module)  
 * [Substitutions : Performs regular expression and string substitutions on response bodies (yaoweibin/ngx_http_substitutions_filter_module)](https://github.com/yaoweibin/ngx_http_substitutions_filter_module)  
 * [Encrypted Session : Encrypt NGINX variables for light-weight session-based authentication (openresty/encrypted-session-nginx-module)](https://github.com/openresty/encrypted-session-nginx-module.git)  
 * [HTTP Set Misc : Various set_xxx directives added to NGINX’s rewrite module (openresty/set-misc-nginx-module)](https://github.com/openresty/set-misc-nginx-module)  
 * [Upstream Fair Balancer : Distributes incoming requests to least-busy servers](https://github.com/itoffshore/nginx-upstream-fair.git)  

### Prérequis compilation nginx 

Passage en mode super utilisateur  
`sudo -s`  
REMARQUE : Dans le cas **Raspberry PI** et **raspbian lite Jessie**
`apt install apt-transport-https`  

### Bash compilation nginx Debian Buster

Ce script installe nginx openssl TLSv1.3 PHP7.3 et les services nginx (init.d et systemd) pour le démarrage

Fichier bash pouvant être exécuté ,copier le contenu ci dessous dans une fenêtre terminal   
`nano compil.sh`  

```bash
#!/bin/bash
#
# Rev. 23 août 2019
# debian10-compilation-nginx-lua-tls1.3-php7.3-MariaDB.sh
# Vérifier si utilisateur "root"
if [ $(id -u) != "0" ]; then
    echo "Erreur : Vous devez être root pour exécuter ce script"
    exit 1
fi

# Version "Stable" nginx (http://nginx.org/en/download.html)
stable_nginx="nginx-1.16.1"

# répertoire de compilation
mkdir -p /usr/src/nginx-custom
#logiciels pour compilation 
apt update
apt install dpkg-dev build-essential zlib1g-dev libpcre3 libpcre3-dev unzip curl libcurl4-openssl-dev libossp-uuid-dev libssl-dev libxslt-dev libgd-dev libgeoip-dev libperl-dev libpam0g-dev libbz2-dev tar unzip curl git -y

# Aller dans le dossier nginx-custom
cd /usr/src/nginx-custom

# Effacer les sources
rm -rf nginx*
#

# Téléchargement des sources nginx
cd /usr/src/nginx-custom
if wget -q --method=HEAD http://nginx.org/download/$stable_nginx.tar.gz;
 then
  echo "version existante."
  wget http://nginx.org/download/$stable_nginx.tar.gz 
  tar -xaf $stable_nginx.tar.gz
  rm *tar.gz

 else
  echo "version nginx inexistante."
  exit 0
fi


# Luajit (fork OpenResty)
git clone https://github.com/openresty/luajit2
cd luajit2/
make
make install

# modules
mkdir -p /usr/src/nginx-custom/modules
cd /usr/src/nginx-custom/modules
#Clonage des modules externes avant compilation
# headers-more-nginx-module
git clone https://github.com/openresty/headers-more-nginx-module
# ngx_http_auth_pam_module
git clone https://github.com/stogh/ngx_http_auth_pam_module nginx-auth-pam
#ngx_cache_purge
git clone https://github.com/FRiCKLE/ngx_cache_purge.git nginx-cache-purge
#
git clone https://github.com/aviafelix/nginx-dav-ext-module
#ngx_devel_kit
git clone https://github.com/simplresty/ngx_devel_kit
#echo-nginx-module
git clone https://github.com/openresty/echo-nginx-module nginx-echo
#ngx-fancyindex
git clone https://github.com/aperezdc/ngx-fancyindex
# modification fancyindex pour avoir la ligne complète
#nchan ex nginx-push-stream-module
git clone https://github.com/slact/nchan
#lua-nginx-module
git clone https://github.com/openresty/lua-nginx-module
#nginx-upload-progress-module
git clone https://github.com/masterzen/nginx-upload-progress-module nginx-upload-progress
#ngx_http_substitutions_filter_module
git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module
#Nginx Upstream Fair Proxy Load Balancer
git clone https://github.com/itoffshore/nginx-upstream-fair

#Configuration , compilation et installation nginx
cd /usr/src/nginx-custom/$stable_nginx

# indiquer au système de compilation de nginx où trouver LuaJIT 2.1
export LUAJIT_LIB=/usr/local/lib
export LUAJIT_INC=/usr/local/include/luajit-2.1

# 

./configure \
 --with-cc-opt='-g -O2 -fdebug-prefix-map=/usr/src/nginx-custom/$stable_nginx=. -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2' \
 --with-ld-opt='-Wl,-z,relro -Wl,-z,now,-rpath,/usr/local/lib' \
 --prefix=/usr/share/nginx \
 --conf-path=/etc/nginx/nginx.conf \
 --http-log-path=/var/log/nginx/access.log \
 --error-log-path=/var/log/nginx/error.log \
 --lock-path=/var/lock/nginx.lock \
 --pid-path=/run/nginx.pid \
 --modules-path=/usr/lib/nginx/modules \
 --http-client-body-temp-path=/var/lib/nginx/body \
 --http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
 --http-proxy-temp-path=/var/lib/nginx/proxy \
 --http-scgi-temp-path=/var/lib/nginx/scgi \
 --http-uwsgi-temp-path=/var/lib/nginx/uwsgi \
 --with-debug \
 --with-pcre-jit \
 --with-http_ssl_module \
 --with-http_stub_status_module \
 --with-http_realip_module \
 --with-http_auth_request_module \
 --with-http_v2_module \
 --with-http_dav_module \
 --with-http_slice_module \
 --with-threads \
 --with-http_addition_module \
 --with-http_flv_module \
 --with-http_geoip_module=dynamic \
 --with-http_gunzip_module \
 --with-http_gzip_static_module \
 --with-http_image_filter_module=dynamic \
 --with-http_mp4_module \
 --with-http_perl_module=dynamic \
 --with-http_random_index_module \
 --with-http_secure_link_module \
 --with-http_sub_module \
 --with-http_xslt_module=dynamic \
 --with-mail=dynamic \
 --with-mail_ssl_module \
 --with-stream \
 --with-stream_ssl_module \
 --with-stream_ssl_preread_module \
 --with-http_ssl_module \
 --add-dynamic-module=../modules/headers-more-nginx-module \
 --add-dynamic-module=../modules/nginx-auth-pam \
 --add-dynamic-module=../modules/nginx-dav-ext-module \
 --add-dynamic-module=../modules/ngx_devel_kit \
 --add-dynamic-module=../modules/nginx-echo \
 --add-dynamic-module=../modules/ngx-fancyindex \
 --add-dynamic-module=../modules/nchan \
 --add-dynamic-module=../modules/lua-nginx-module \
 --add-dynamic-module=../modules/nginx-upload-progress \
 --add-dynamic-module=../modules/nginx-upstream-fair \
 --add-dynamic-module=../modules/nginx-cache-purge \
 --add-dynamic-module=../modules/ngx_http_substitutions_filter_module


make
make install

#Copier le binaire pour le PATH
cp /usr/share/nginx/sbin/nginx /usr/sbin/
#Effacement compilation
#make clean
#Dossier temporaire
mkdir -p /var/lib/nginx
#Dossier config
mkdir -p /etc/nginx/conf.d/
#dossier vhost
mkdir -p /var/www


# Modules configuration nginx
mkdir -p /etc/nginx/modules-enabled
for file in $(find "/usr/lib/nginx/modules/" -type f -name "*.so")
  do 
   tempfile="${file##*/}"
   prefix="50-mod"
	if [ $tempfile = "ndk_http_module.so" ]
	 then
	  prefix="10-mod"
	fi
   echo "load_module /usr/lib/nginx/modules/$tempfile;" | tee  /etc/nginx/modules-enabled/$prefix-$tempfile-upload.conf

  done

# service nginx
# /etc/init.d/nginx
cat > /etc/init.d/nginx << EOF
#!/bin/sh

### BEGIN INIT INFO
# Provides:	  nginx
# Required-Start:    $local_fs $remote_fs $network $syslog $named
# Required-Stop:     $local_fs $remote_fs $network $syslog $named
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the nginx web server
# Description:       starts nginx using start-stop-daemon
### END INIT INFO

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/sbin/nginx
NAME=nginx
DESC=nginx

# Include nginx defaults if available
if [ -r /etc/default/nginx ]; then
	. /etc/default/nginx
fi

STOP_SCHEDULE="${STOP_SCHEDULE:-QUIT/5/TERM/5/KILL/5}"

test -x $DAEMON || exit 0

. /lib/init/vars.sh
. /lib/lsb/init-functions

# Try to extract nginx pidfile
PID=$(cat /etc/nginx/nginx.conf | grep -Ev '^\s*#' | awk 'BEGIN { RS="[;{}]" } { if ($1 == "pid") print $2 }' | head -n1)
if [ -z "$PID" ]; then
	PID=/run/nginx.pid
fi

if [ -n "$ULIMIT" ]; then
	# Set ulimit if it is set in /etc/default/nginx
	ulimit $ULIMIT
fi

start_nginx() {
	# Start the daemon/service
	#
	# Returns:
	#   0 if daemon has been started
	#   1 if daemon was already running
	#   2 if daemon could not be started
	start-stop-daemon --start --quiet --pidfile $PID --exec $DAEMON --test > /dev/null \
		|| return 1
	start-stop-daemon --start --quiet --pidfile $PID --exec $DAEMON -- \
		$DAEMON_OPTS 2>/dev/null \
		|| return 2
}

test_config() {
	# Test the nginx configuration
	$DAEMON -t $DAEMON_OPTS >/dev/null 2>&1
}

stop_nginx() {
	# Stops the daemon/service
	#
	# Return
	#   0 if daemon has been stopped
	#   1 if daemon was already stopped
	#   2 if daemon could not be stopped
	#   other if a failure occurred
	start-stop-daemon --stop --quiet --retry=$STOP_SCHEDULE --pidfile $PID --name $NAME
	RETVAL="$?"
	sleep 1
	return "$RETVAL"
}

reload_nginx() {
	# Function that sends a SIGHUP to the daemon/service
	start-stop-daemon --stop --signal HUP --quiet --pidfile $PID --name $NAME
	return 0
}

rotate_logs() {
	# Rotate log files
	start-stop-daemon --stop --signal USR1 --quiet --pidfile $PID --name $NAME
	return 0
}

upgrade_nginx() {
	# Online upgrade nginx executable
	# http://nginx.org/en/docs/control.html
	#
	# Return
	#   0 if nginx has been successfully upgraded
	#   1 if nginx is not running
	#   2 if the pid files were not created on time
	#   3 if the old master could not be killed
	if start-stop-daemon --stop --signal USR2 --quiet --pidfile $PID --name $NAME; then
		# Wait for both old and new master to write their pid file
		while [ ! -s "${PID}.oldbin" ] || [ ! -s "${PID}" ]; do
			cnt=`expr $cnt + 1`
			if [ $cnt -gt 10 ]; then
				return 2
			fi
			sleep 1
		done
		# Everything is ready, gracefully stop the old master
		if start-stop-daemon --stop --signal QUIT --quiet --pidfile "${PID}.oldbin" --name $NAME; then
			return 0
		else
			return 3
		fi
	else
		return 1
	fi
}

case "$1" in
	start)
		log_daemon_msg "Starting $DESC" "$NAME"
		start_nginx
		case "$?" in
			0|1) log_end_msg 0 ;;
			2)   log_end_msg 1 ;;
		esac
		;;
	stop)
		log_daemon_msg "Stopping $DESC" "$NAME"
		stop_nginx
		case "$?" in
			0|1) log_end_msg 0 ;;
			2)   log_end_msg 1 ;;
		esac
		;;
	restart)
		log_daemon_msg "Restarting $DESC" "$NAME"

		# Check configuration before stopping nginx
		if ! test_config; then
			log_end_msg 1 # Configuration error
			exit $?
		fi

		stop_nginx
		case "$?" in
			0|1)
				start_nginx
				case "$?" in
					0) log_end_msg 0 ;;
					1) log_end_msg 1 ;; # Old process is still running
					*) log_end_msg 1 ;; # Failed to start
				esac
				;;
			*)
				# Failed to stop
				log_end_msg 1
				;;
		esac
		;;
	reload|force-reload)
		log_daemon_msg "Reloading $DESC configuration" "$NAME"

		# Check configuration before stopping nginx
		#
		# This is not entirely correct since the on-disk nginx binary
		# may differ from the in-memory one, but that's not common.
		# We prefer to check the configuration and return an error
		# to the administrator.
		if ! test_config; then
			log_end_msg 1 # Configuration error
			exit $?
		fi

		reload_nginx
		log_end_msg $?
		;;
	configtest|testconfig)
		log_daemon_msg "Testing $DESC configuration"
		test_config
		log_end_msg $?
		;;
	status)
		status_of_proc -p $PID "$DAEMON" "$NAME" && exit 0 || exit $?
		;;
	upgrade)
		log_daemon_msg "Upgrading binary" "$NAME"
		upgrade_nginx
		log_end_msg $?
		;;
	rotate)
		log_daemon_msg "Re-opening $DESC log files" "$NAME"
		rotate_logs
		log_end_msg $?
		;;
	*)
		echo "Usage: $NAME {start|stop|restart|reload|force-reload|status|configtest|rotate|upgrade}" >&2
		exit 3
		;;
esac
EOF

# droits en exécution
chmod u+x /etc/init.d/nginx

# Création systemd nginx.service
cat > /etc/systemd/system/nginx.service << EOF
[Unit]
Description=A high performance web server and a reverse proxy server
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
EOF


# Fichier de configuration nginx
rm /etc/nginx/nginx.conf
cat > /etc/nginx/nginx.conf << EOF
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	#ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_protocols TLSv1.2 TLSv1.3;
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;
	gzip_disable "msie6";

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
EOF

# PHP7.3
apt-get -y install apt-transport-https lsb-release ca-certificates
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
sh -c 'echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
apt update 
# Le paquet php7.3-mcrypt est inexistant (si besoin ,installer php7.1-mcrypt)
apt install php7.3 php7.3-fpm php7.3-mysql php7.3-curl php7.3-json php7.3-gd php7.3-tidy php7.3-intl php7.3-imagick php7.3-xml php7.3-mbstring php7.3-zip -y

# Contenu fichier /etc/nginx/conf.d/default.conf
# back slash pour prise en compte request_filename
cat > /etc/nginx/conf.d/default.conf << EOF
server {
    listen 80;
    listen [::]:80;
    root /var/www/ ;
        location ~ \.php$ {
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           fastcgi_pass unix:/run/php/php7.3-fpm.sock;   # PHP7.3
           fastcgi_index index.php;
           include fastcgi_params;
           fastcgi_param SCRIPT_FILENAME \$request_filename;
        }
}
EOF

# Contenu fichier /var/www/index.html
cat > /var/www/index.html << EOF
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx on Debian!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx on Debian!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working on Debian. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a></p>

<p>
      Please use the <tt>reportbug</tt> tool to report bugs in the
      nginx package with Debian. However, check <a
      href="http://bugs.debian.org/cgi-bin/pkgreport.cgi?ordering=normal;archive=0;src=nginx;repeatmerged=0">existing
      bug reports</a> before reporting a new bug.
</p>

<p><em>Thank you for using debian and nginx.</em></p>


</body>
</html>
EOF

# test php
echo "<?php phpinfo(); ?>" > /var/www/info.php

# Réinitialiser
systemctl daemon-reload
# Activer nginx 
systemctl enable nginx
# Relancer les services
systemctl restart nginx php7.3-fpm 

# Installer MariaDb
apt install mariadb-server -y
# Générer un mot de passe pour mysql
echo $(head -c 12 /dev/urandom | openssl enc -base64) > /etc/mysql/mdp
# fichier sql
cat > /tmp/mysql_secure.sql << EOF
GRANT ALL ON *.* TO 'admin'@'localhost' IDENTIFIED BY '$(cat /etc/mysql/mdp)' WITH GRANT OPTION;
FLUSH PRIVILEGES; /* Applique les changements effectués précédemment concernant la gestion des droits */
EOF

# Exécuter la requête sql
mysql -uroot < /tmp/mysql_secure.sql

echo "Versions Nginx OpenSSL MariaDB et PHP"
nginx -v
openssl version
mysql --version
echo "Mot de passe MySql/MariaDB : /etc/mysql/mdp"
php -v

echo "********** FIN EXECUTION SCRIPT ************"

```

Le rendre exécutable  
`chmod +x compil.sh`  


Exécution   
`./compil.sh`  




## Script téléchargeable pour compiler nginx

Ce script bash compile et installe nginx,lua, TLSV1.3 , PHP7.3 et MariaDB. [Téléchargement](/files/debian10-compilation-nginx-lua-tls1.3-php7.3-MariaDB.sh)  
Après téléchargement mettre à jour dans le fichier ,si nécessaire  , la variable **latest_nginx**   

Résumé des commandes

```
sudo -s             # Passer en mode super utilisateur (su ou sudo)  
wget -O compil.sh "https://yann.cinay.eu/files/debian10-compilation-nginx-lua-tls1.3-php7.3-MariaDB.sh" # Télécharger le script
chmod +x compil.sh  # Rendre le script exécutable  
./compil.sh     # Exécuter le script
#******** Patienter **********
```

Tester sur le lien de type http://votre-ip/info.php

## Compilation OpenResty

[OpenResty](https://openresty.org/en/) embarque le module HttpLuaModule permettant l’exécution de script Lua. Plusieurs directives permettent de lancer un script à différents moments de la requête, elles sont toutes de la forme « *_by_lua ». Les fonctions disponibles dans ces directives sont limitées :

* **init_by_lua**: le code sera exécuté au démarrage quand le serveur NGINX lit la configuration. Il est utile pour déclarer des variables globales ou précharger des modules
* **set_by_lua**: permet d’effectuer un traitement et de récupérer le résultat dans un variable. Le code exécuté bloque la boucle d’événement de NGINX et doit donc être rapide 
* **rewrite_by_lua**: le code est exécuté pour chaque requête et après l’exécution du module HttpRewrite
* **access_by_lua**: le code est exécuté pour chaque requête et après l’exécution du module HttpAccess
* **header_filter_by_lua**: utilisé uniquement pour filtrer les headers de la requête
* **content_by_lua**: utilisé lorsqu’un script renvoie du contenu via par exemple ngx.say
* **body_filter_by_lua**: le code est exécuté après réception de données de réponse et permet de modifier le contenu renvoyé au client. Il peut être lancé plusieurs fois par requête selon le volume de données
* **log_by_lua**: le code est exécuté après l’écriture dans le access log.


### Les modules ajoutés à la compilation OpenResty 

Vous trouverez [ici](https://openresty.org/en/components.html) la liste de tous les composants inclus dans OpenResty. Tous les composants peuvent être activés ou désactivés selon les besoins.

### Prérequis à la compilation OpenResty 

Passage en mode super utilisateur  
`sudo -s`  
REMARQUE : Dans le cas **Raspberry PI** et **raspbian lite**
`apt install apt-transport-https`  

### Bash de compilation OpenResty

Ce script compile et installe openresty (nginx + openssl TLSv1.3 + modules dynamiques) ,le  service  nginx (systemd) pour le démarrage, PHP7.3 et mariadb  

Fichier bash pouvant être exécuté ,copier le contenu ci dessous dans une fenêtre terminal   
`nano compil`  

```bash
#!/bin/bash
#
# 10 septembre 2019

#+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# Nginx est un serveur HTTP et reverse proxy utilisé par de nombreux sites. 
# OpenResty est une surcouche construite avec de nombreux modules par défaut, ils permettent par exemple 
# la personnalisation via des scripts Lua ou des accès simplifiés à des bases de données.
#+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

# Vérifier si utilisateur "root"
if [ $(id -u) != "0" ]; then
    echo "Erreur : Vous devez être root pour exécuter ce script"
    exit 1
fi

#logiciels pour compilation 
apt update
apt install dpkg-dev build-essential zlib1g-dev libpcre3 libpcre3-dev libcurl4-openssl-dev libossp-uuid-dev libssl-dev libxslt-dev libgd-dev libgeoip-dev libperl-dev libpam0g-dev libbz2-dev tar unzip curl git -y
# 
# openresty voir https://openresty.org/en/download.html pour la dernière version
wget https://openresty.org/download/openresty-1.15.8.1.tar.gz
tar -xvf openresty-1.15.8.1.tar.gz
cd openresty-1.15.8.1

# les modules 
mkdir -p modules
cd modules
#### Clonage des modules externes avant compilation ####

# headers-more-nginx-module
git clone https://github.com/openresty/headers-more-nginx-module
# ngx_http_auth_pam_module
git clone https://github.com/stogh/ngx_http_auth_pam_module nginx-auth-pam
#ngx_cache_purge
git clone https://github.com/FRiCKLE/ngx_cache_purge.git nginx-cache-purge
#
git clone https://github.com/aviafelix/nginx-dav-ext-module
#echo-nginx-module
git clone https://github.com/openresty/echo-nginx-module nginx-echo
#ngx-fancyindex
git clone https://github.com/aperezdc/ngx-fancyindex
# modification fancyindex pour avoir la ligne complète
#nchan ex nginx-push-stream-module
git clone https://github.com/slact/nchan
#nginx-upload-progress-module
git clone https://github.com/masterzen/nginx-upload-progress-module nginx-upload-progress
#ngx_http_substitutions_filter_module
git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module
#Nginx Upstream Fair Proxy Load Balancer
git clone https://github.com/itoffshore/nginx-upstream-fair

# configurer
cd ..
./configure \
 --with-debug \
 --with-pcre-jit \
 --with-http_ssl_module \
 --with-http_stub_status_module \
 --with-http_realip_module \
 --with-http_auth_request_module \
 --with-http_v2_module \
 --with-http_dav_module \
 --with-http_slice_module \
 --with-threads \
 --with-http_addition_module \
 --with-http_flv_module \
 --with-http_geoip_module=dynamic \
 --with-http_gunzip_module \
 --with-http_gzip_static_module \
 --with-http_image_filter_module=dynamic \
 --with-http_mp4_module \
 --with-http_perl_module=dynamic \
 --with-http_random_index_module \
 --with-http_secure_link_module \
 --with-http_sub_module \
 --with-http_xslt_module=dynamic \
 --with-mail=dynamic \
 --with-mail_ssl_module \
 --with-stream=dynamic \
 --with-stream_ssl_module \
 --add-dynamic-module=modules/nginx-auth-pam \
 --add-dynamic-module=modules/nginx-cache-purge \
 --add-dynamic-module=modules/nginx-dav-ext-module \
 --add-dynamic-module=modules/ngx-fancyindex \
 --add-dynamic-module=modules/nchan \
 --add-dynamic-module=modules/nginx-upload-progress \
 --add-dynamic-module=modules/nginx-upstream-fair \
 --add-dynamic-module=modules/ngx_http_substitutions_filter_module 

## Modules ci-dessous sont intégrés à openresty ##
# --add-dynamic-module=modules/headers-more-nginx-module \
# --add-dynamic-module=modules/nginx-echo \

# compilation installation
make && make install

# Configuration nginx pour le chargement des modules
mkdir -p /usr/local/openresty/nginx/modules-enabled
for file in $(find "/usr/local/openresty/nginx/modules/" -type f -name "*.so")
  do 
   tempfile="${file##*/}"
   prefix="50-mod"
	if [ $tempfile = "ndk_http_module.so" ]
	 then
	  prefix="10-mod"
	fi
   echo 'load_module "/usr/local/openresty/nginx/modules/'$tempfile'";' | tee  /usr/local/openresty/nginx/modules-enabled/$prefix-$tempfile-upload.conf

  done

# Création des liens pour le dossier /usr/local/openresty/bin/
for file in /usr/local/openresty/bin/*
  do 
   tempfile="${file##*/}"
   rest=${tempfile%.*}
        #convert $tempfile $rest".png"
	ln -s $file /usr/local/bin/$tempfile
   done



# Création systemd openresty.service
cat > /etc/systemd/system/openresty.service << EOF
# Stop dance for OpenResty
# A modification of the Nginx systemd script
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the Nginx process.
# If, after 5s (--retry QUIT/5) OpenResty is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if OpenResty is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# Nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
Description=A dynamic web platform based on Nginx and LuaJIT.
After=network.target

[Service]
Type=forking
PIDFile=/run/openresty.pid
ExecStartPre=/usr/local/openresty/bin/openresty -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/local/openresty/bin/openresty -g 'daemon on; master_process on;'
ExecReload=/usr/local/openresty/bin/openresty -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/openresty.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
EOF

# le fichier de configuration /usr/local/openresty/nginx/conf/nginx.conf
rm /usr/local/openresty/nginx/conf/nginx.conf
cat > /usr/local/openresty/nginx/conf/nginx.conf << EOF
user www-data;
worker_processes  auto;
pid /run/openresty.pid;

include ../modules-enabled/*.conf;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    variables_hash_max_size 2048;
    
    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;

    keepalive_timeout  65;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    access_log /var/log/openresty/access.log;
    error_log /var/log/openresty/error.log;

    gzip  on;
    gzip_disable "msie6";

    include ../sites/*.conf;
}
EOF

mkdir -p /var/log/openresty
systemctl daemon-reload
systemctl start openresty
systemctl enable openresty

# Ensuite, créez le nouveau répertoire de sites que nous avons spécifié dans la ligne d’ include , il contient les fichiers de configuration conf des sites virtuels VHOST
mkdir -p /usr/local/openresty/nginx/sites
# Création du dossier  contenant les fichiers html, php, etc...
mkdir -p /usr/local/openresty/nginx/www/

# PHP7.3
apt-get -y install apt-transport-https lsb-release ca-certificates
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
apt update
# Le paquet php7.3-mcrypt est inexistant (si besoin ,installer php7.1-mcrypt)
apt install php7.3 php7.3-fpm php7.3-mysql php7.3-curl php7.3-json php7.3-gd php7.3-tidy php7.3-intl php7.3-imagick php7.3-xml php7.3-mbstring php7.3-zip -y

# Contenu fichier /usr/local/openresty/nginx/sites/default.conf
# back slash pour prise en compte request_filename
cat > /usr/local/openresty/nginx/sites/default.conf << EOF
server {
    listen 80;
    listen [::]:80;
    root /usr/local/openresty/nginx/www/ ;
        location ~ \.php$ {
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           fastcgi_pass unix:/run/php/php7.3-fpm.sock;   # PHP7.3
           fastcgi_index index.php;
           include fastcgi_params;
           fastcgi_param SCRIPT_FILENAME \$request_filename;
        }
}
EOF

# DÉPLACEMENT fichier index.html
mv /usr/local/openresty/nginx/html/index.html /usr/local/openresty/nginx/www/

# test php
echo "<?php phpinfo(); ?>" > /usr/local/openresty/nginx/www/info.php

# Réinitialiser
systemctl daemon-reload
# Relancer les services
systemctl restart openresty php7.3-fpm 

# Installer MariaDb
apt install mariadb-server -y
# Générer un mot de passe pour mysql
echo $(head -c 12 /dev/urandom | openssl enc -base64) > /etc/mysql/mdp
# fichier sql
cat > /tmp/mysql_secure.sql << EOF
GRANT ALL ON *.* TO 'admin'@'localhost' IDENTIFIED BY '$(cat /etc/mysql/mdp)' WITH GRANT OPTION;
FLUSH PRIVILEGES; /* Applique les changements effectués précédemment concernant la gestion des droits */
EOF

# Exécuter la requête sql
mysql -uroot < /tmp/mysql_secure.sql

clear
echo "**************************************"
echo "Versions Nginx OpenSSL MariaDB et PHP"
echo "**************************************"
openresty -v
openssl version
mysql --version
echo "Mot de passe MySql/MariaDB : /etc/mysql/mdp"
php -v

echo "********** FIN EXECUTION SCRIPT ************"

```

Le rendre exécutable  
`chmod +x compil`  


Exécution   
`./compil`  




## Script téléchargeable pour compiler OpenResty

Ce script bash compile et installe openresty (nginx + openssl TLSv1.3 + modules dynamiques) ,le  service  nginx (systemd) pour le démarrage, PHP7.3 et mariadb. [Téléchargement](/files/debian-compilation-openresty(nginx+lua+tls1.3)-php7.3-MariaDB.sh)  
Après téléchargement passer en mode super utilisateur (su ou sudo)  

Exécuter les instructions suivantes pour lancer la compilation

```
sudo -s             # Passer en mode super utilisateur (su ou sudo)  
wget -O compil.sh "https://yann.cinay.eu/files/debian-compilation-openresty(nginx+lua+tls1.3)-php7.3-MariaDB.sh" # Télécharger le script
chmod +x compil.sh  # Rendre le script exécutable  
./compil.sh     # Exécuter le script

```

Patienter de 5 à 10 minutes...   
Résultat de la compilation

```
**************************************
Versions Nginx OpenSSL MariaDB et PHP
**************************************
nginx version: openresty/1.15.8.1
OpenSSL 1.1.1c  28 May 2019
mysql  Ver 15.1 Distrib 10.3.17-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2
Mot de passe MySql/MariaDB : /etc/mysql/mdp
PHP 7.3.4-2 (cli) (built: Apr 13 2019 19:05:48) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.4, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.3.4-2, Copyright (c) 1999-2018, by Zend Technologies
********** FIN EXECUTION SCRIPT ************
```

Tester avec le lien de type http://votre-ip/info.php

## Liens

* [Nginx Mainline,Stable et Legacy](http://nginx.org/en/download.html)
* [How to Install PHP 7.3 on Debian 9](https://www.rosehosting.com/blog/how-to-install-php-7-3-on-debian-9/)
* [Nginx et Lua, découverte d’OpenResty](https://jolicode.com/blog/nginx-et-lua-decouverte-d-openresty)
* [OpenResty installation](https://openresty.org/en/installation.html)
* [How to Enable TLS 1.3 in Nginx](https://www.howtoforge.com/how-to-enable-tls-13-in-nginx/)
* [How to Enable HTTP/2 in Nginx](https://www.howtoforge.com/how-to-enable-http-2-in-nginx/)
* [How to Build Nginx from source on Debian 9](https://www.howtoforge.com/how-to-build-nginx-from-source-on-debian-9/)
* [How to Use the OpenResty Web Framework for Nginx on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-use-the-openresty-web-framework-for-nginx-on-ubuntu-16-04)  
