+++
title = 'Guide d'installation Nextcloud 17 et plus (Debian / NGINX)'
date = 2020-01-20 00:00:00 +0100
categories = nextcloud
+++
## Guide d'installation Nextcloud 17 et plus (Debian / NGINX)

[Nextcloud 17 installation guide and more (Debian/NGINX)](https://www.c-rieger.de/nextcloud-installation-guide-debian/) de [Carsten Rieger](https://www.c-rieger.de/author/criegerde/) · Publié 2. août 2019 · Mis à jour 30. septembre 2019

En suivant ce guide, vous pourrez installer et configurer Nextcloud 17 en fonction de Debian 9.x Stretch ou de Debian 10.x Buster, NGINX 1.17.x, TLSv1.3, PHP 7.3, MariaDB 10.4, Redis, fail2ban, firewall (ufw). ) et obtiendra une note A + de Nextcloud et de Qualys SSL Labs. Nous allons demander et mettre en œuvre votre / vos certificat (s) SSL à partir de Let's Encrypt. Il vous suffit de modifier les valeurs marquées en rouge ( YOUR.DEDYN.IO , 192.168.2.x , ssh port 22 ) en ce qui concerne votre environnement!

### Pré-requis

De mon point de vue, les exigences de ce guide peuvent être considérées comme faibles: il suffit de

*    fournir un serveur 64 bits (par exemple, Intel NUC),
*    Transférez deux ports (80 et 443) depuis Internet (votre routeur, par exemple FritzBox ou Speedport) vers votre serveur interne Nextcloud
*    et installez le système d'exploitation Debian Stretch 9.x (64Bit). 


## 1. Installez NGINX

Préparez votre serveur pour l'installation elle-même:

    su - 
    (ou sudo -s) 

    apt install curl gnupg2 git lsb-release ssl-cert ca-certificates apt-transport-https tree locate software-properties-common dirmngr screen htop net-tools zip unzip curl ffmpeg ghostscript libfile-fcntllock-perl -y

Ajouter de nouveaux référentiels de logiciels

    cd /etc/apt/sources.list.d

    echo "deb [arch=amd64] http://nginx.org/packages/mainline/debian $(lsb_release -cs) nginx" | tee nginx.list 
    echo "deb [arch=amd64] https://packages.sury.org/php/ $(lsb_release -cs) main" | tee php.list
    echo "deb [arch=amd64] http://mirror2.hs-esslingen.de/mariadb/repo/10.4/debian $(lsb_release -cs) main" | tee mariadb.list

Téléchargez les clés requises pour faire confiance à toutes les nouvelles sources:

    curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
    wget -q https://packages.sury.org/php/apt.gpg -O- | apt-key add -
    apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8

Mettez à jour votre serveur, générez des certificats auto-signés et installez nginx:

    apt update && apt upgrade -y && apt install ssl-cert -y && make-ssl-cert generate-default-snakeoil
    apt remove nginx nginx-extras nginx-common nginx-full -y --allow-change-held-packages


Assurez-vous qu'Apache (2) ne fonctionne pas, sinon NGINX ne démarrera pas car le port requis (: 80) serait utilisé par Apache (2):

    systemctl stop apache2.service && systemctl disable apache2.service
    apt install nginx -y && systemctl enable nginx.service

### Changer la configuration de NGINX

Attention remplacer 192.168.2.0/24 et 192.168.2.1 apr vote ip/24 et gateway

    mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak && vi /etc/nginx/nginx.conf

Collez les lignes suivantes:

```
user www-data;
worker_processes auto;
pid /var/run/nginx.pid;
events {
 worker_connections 1024;
 multi_accept on;
 use epoll;
}
http {
  server_names_hash_bucket_size 64;
		upstream php-handler {
	  		server unix:/run/php/php7.3-fpm.sock;
		}
	set_real_ip_from 127.0.0.1;
	set_real_ip_from 192.168.2.0/24;
	real_ip_header X-Forwarded-For;
	real_ip_recursive on;
	include /etc/nginx/mime.types;
	#include /etc/nginx/proxy.conf;
	#include /etc/nginx/ssl.conf;
	#include /etc/nginx/header.conf;
	#include /etc/nginx/optimization.conf;
	default_type application/octet-stream;
	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log warn;
	sendfile on;
	send_timeout 3600;
	tcp_nopush on;
	tcp_nodelay on;
	open_file_cache max=500 inactive=10m;
	open_file_cache_errors on;
	keepalive_timeout 65;
	reset_timedout_connection on;
	server_tokens off;
	resolver 192.168.2.1 valid=30s;
	#resolver 127.0.0.53 valid=30s; is recommended but reuqires a valid resolver configuration
	resolver_timeout 5s;
	include /etc/nginx/conf.d/*.conf;
}
```

### Redémarrez NGINX

    service nginx restart 

Créer des dossiers et appliquer des autorisations

    mkdir -p /var/nc_data /var/www/letsencrypt
    chown -R www-data:www-data /var/nc_data /var/www
    chown -R www-data:root

## 2. Installez PHP

    apt install php7.3-fpm php7.3-gd php7.3-mysql php7.3-curl php7.3-xml php7.3-zip php7.3-intl php7.3-mbstring php7.3-json php7.3-bz2 php7.3-ldap php-apcu imagemagick php-imagick -y

Génial, PHP 7.3 est déjà installé. Vérifiez vos paramètres de fuseau horaire

    date 

et si nécessaire le régler correctement

    timedatectl set-timezone Europe/Paris 

Configurer PHP


```
cp /etc/php/7.3/fpm/pool.d/www.conf /etc/php/7.3/fpm/pool.d/www.conf.bak
cp /etc/php/7.3/cli/php.ini /etc/php/7.3/cli/php.ini.bak
cp /etc/php/7.3/fpm/php.ini /etc/php/7.3/fpm/php.ini.bak
cp /etc/php/7.3/fpm/php-fpm.conf /etc/php/7.3/fpm/php-fpm.conf.bak

sed -i "s/;env\[HOSTNAME\] = /env[HOSTNAME] = /" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/;env\[TMP\] = /env[TMP] = /" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/;env\[TMPDIR\] = /env[TMPDIR] = /" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/;env\[TEMP\] = /env[TEMP] = /" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/;env\[PATH\] = /env[PATH] = /" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/pm.max_children = .*/pm.max_children = 240/" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/pm.start_servers = .*/pm.start_servers = 20/" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/pm.min_spare_servers = .*/pm.min_spare_servers = 10/" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/pm.max_spare_servers = .*/pm.max_spare_servers = 20/" /etc/php/7.3/fpm/pool.d/www.conf
sed -i "s/;pm.max_requests = 500/pm.max_requests = 500/" /etc/php/7.3/fpm/pool.d/www.conf

sed -i "s/output_buffering =.*/output_buffering = 'Off'/" /etc/php/7.3/cli/php.ini
sed -i "s/max_execution_time =.*/max_execution_time = 1800/" /etc/php/7.3/cli/php.ini
sed -i "s/max_input_time =.*/max_input_time = 3600/" /etc/php/7.3/cli/php.ini
sed -i "s/post_max_size =.*/post_max_size = 10240M/" /etc/php/7.3/cli/php.ini
sed -i "s/upload_max_filesize =.*/upload_max_filesize = 10240M/" /etc/php/7.3/cli/php.ini
sed -i "s/max_file_uploads =.*/max_file_uploads = 100/" /etc/php/7.3/cli/php.ini
sed -i "s/;date.timezone.*/date.timezone = Europe\/\Berlin/" /etc/php/7.3/cli/php.ini
sed -i "s/;session.cookie_secure.*/session.cookie_secure = True/" /etc/php/7.3/cli/php.ini

sed -i "s/memory_limit = 128M/memory_limit = 512M/" /etc/php/7.3/fpm/php.ini
sed -i "s/output_buffering =.*/output_buffering = 'Off'/" /etc/php/7.3/fpm/php.ini
sed -i "s/max_execution_time =.*/max_execution_time = 1800/" /etc/php/7.3/fpm/php.ini
sed -i "s/max_input_time =.*/max_input_time = 3600/" /etc/php/7.3/fpm/php.ini
sed -i "s/post_max_size =.*/post_max_size = 10240M/" /etc/php/7.3/fpm/php.ini
sed -i "s/upload_max_filesize =.*/upload_max_filesize = 10240M/" /etc/php/7.3/fpm/php.ini
sed -i "s/max_file_uploads =.*/max_file_uploads = 100/" /etc/php/7.3/fpm/php.ini
sed -i "s/;date.timezone.*/date.timezone = Europe\/\Berlin/" /etc/php/7.3/fpm/php.ini
sed -i "s/;session.cookie_secure.*/session.cookie_secure = True/" /etc/php/7.3/fpm/php.ini
sed -i "s/;opcache.enable=.*/opcache.enable=1/" /etc/php/7.3/fpm/php.ini
sed -i "s/;opcache.enable_cli=.*/opcache.enable_cli=1/" /etc/php/7.3/fpm/php.ini
sed -i "s/;opcache.memory_consumption=.*/opcache.memory_consumption=128/" /etc/php/7.3/fpm/php.ini
sed -i "s/;opcache.interned_strings_buffer=.*/opcache.interned_strings_buffer=8/" /etc/php/7.3/fpm/php.ini
sed -i "s/;opcache.max_accelerated_files=.*/opcache.max_accelerated_files=10000/" /etc/php/7.3/fpm/php.ini
sed -i "s/;opcache.revalidate_freq=.*/opcache.revalidate_freq=1/" /etc/php/7.3/fpm/php.ini
sed -i "s/;opcache.save_comments=.*/opcache.save_comments=1/" /etc/php/7.3/fpm/php.ini

sed -i "s/;emergency_restart_threshold =.*/emergency_restart_threshold = 10/" /etc/php/7.3/fpm/php-fpm.conf
sed -i "s/;emergency_restart_interval =.*/emergency_restart_interval = 1m/" /etc/php/7.3/fpm/php-fpm.conf
sed -i "s/;process_control_timeout =.*/process_control_timeout = 10s/" /etc/php/7.3/fpm/php-fpm.conf

sed -i "s/09,39.*/# &/" /etc/cron.d/php
(crontab -l ; echo "09,39 * * * * /usr/lib/php/sessionclean 2>&1") | crontab -u root -

cp /etc/ImageMagick-6/policy.xml /etc/ImageMagick-6/policy.xml.bak
sed -i "s/rights\=\"none\" pattern\=\"PS\"/rights\=\"read\|write\" pattern\=\"PS\"/" /etc/ImageMagick-6/policy.xml
sed -i "s/rights\=\"none\" pattern\=\"EPI\"/rights\=\"read\|write\" pattern\=\"EPI\"/" /etc/ImageMagick-6/policy.xml
sed -i "s/rights\=\"none\" pattern\=\"PDF\"/rights\=\"read\|write\" pattern\=\"PDF\"/" /etc/ImageMagick-6/policy.xml
sed -i "s/rights\=\"none\" pattern\=\"XPS\"/rights\=\"read\|write\" pattern\=\"XPS\"/" /etc/ImageMagick-6/policy.xml
```

Redémarrez les deux, PHP et NGINX

    service php7.3-fpm restart && service nginx restart

## 3. Installez MariaDB

    apt update && apt install mariadb-server -y

Vérifiez la version de votre serveur de base de données:

    mysql --version

*mysql Ver 15.1 Distrib 10.4.x-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2* devrait apparaître.

### MariaDB sécurisé

    mysql_secure_installation
        Switch to unix_socket authentication [Y/n] N
        Enter current password for root (enter for none): <ENTER> or type the password Set root password? [Y/n] Y

*Si déjà défini lors de l'installation de MariaDB, il vous sera demandé si vous souhaitez modifier ou conserver le mot de passe.* 

```
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
``` 

### Configurer MariaDB

    service mysql stop
    mv /etc/mysql/my.cnf /etc/mysql/my.cnf.bak && vi /etc/mysql/my.cnf

Collez les lignes suivantes:

```
[client]
default-character-set = utf8mb4
port = 3306
socket = /var/run/mysqld/mysqld.sock

[mysqld_safe]
log_error=/var/log/mysql/mysql_error.log
nice = 0
socket = /var/run/mysqld/mysqld.sock

[mysqld]
basedir = /usr
bind-address = 127.0.0.1
binlog_format = ROW
bulk_insert_buffer_size = 16M
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
concurrent_insert = 2
connect_timeout = 5
datadir = /var/lib/mysql
default_storage_engine = InnoDB
expire_logs_days = 10
general_log_file = /var/log/mysql/mysql.log
general_log = 0
innodb_buffer_pool_size = 1024M
innodb_buffer_pool_instances = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 32M
innodb_max_dirty_pages_pct = 90
innodb_file_per_table = 1
innodb_open_files = 400
innodb_io_capacity = 4000
innodb_flush_method = O_DIRECT
key_buffer_size = 128M
lc_messages_dir = /usr/share/mysql
lc_messages = en_US
log_bin = /var/log/mysql/mariadb-bin
log_bin_index = /var/log/mysql/mariadb-bin.index
log_error=/var/log/mysql/mysql_error.log
log_slow_verbosity = query_plan
log_warnings = 2
long_query_time = 1
max_allowed_packet = 16M
max_binlog_size = 100M
max_connections = 200
max_heap_table_size = 64M
myisam_recover_options = BACKUP
myisam_sort_buffer_size = 512M
port = 3306
pid-file = /var/run/mysqld/mysqld.pid
query_cache_limit = 2M
query_cache_size = 64M
query_cache_type = 1
query_cache_min_res_unit = 2k
read_buffer_size = 2M
read_rnd_buffer_size = 1M
skip-external-locking
skip-name-resolve
slow_query_log_file = /var/log/mysql/mariadb-slow.log
slow-query-log = 1
socket = /var/run/mysqld/mysqld.sock
sort_buffer_size = 4M
table_open_cache = 400
thread_cache_size = 128
tmp_table_size = 64M
tmpdir = /tmp
transaction_isolation = READ-COMMITTED
user = mysql
wait_timeout = 600

[mysqldump]
max_allowed_packet = 16M
quick
quote-names

[isamchk]
key_buffer = 16M
```

Redémarrez et connectez-vous à MariaDB

    service mysql restart && mysql -uroot -p

### Créer base nextcloud

* la base de données **nextcloud**
* l'utilisateur **nextcloud**
* et son mot de passe **password**

```
CREATE DATABASE nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci; CREATE USER nextcloud@localhost identified by 'password'; GRANT ALL PRIVILEGES on nextcloud.* to nextcloud@localhost; FLUSH privileges; quit;
```

Vérifiez que le niveau d'isolement de la transaction a été défini sur READ_Commit et que le classement a été défini sur UTF8MB4 correctement:

    mysql -h localhost -uroot -p -e "SELECT @@TX_ISOLATION; SELECT SCHEMA_NAME 'database', default_character_set_name 'charset', DEFAULT_COLLATION_NAME 'collation' FROM information_schema.SCHEMATA WHERE SCHEMA_NAME='nextcloud'"


Si le résultat est "READ-COMMITTED" et "utf8mb4_general_ci" comme indiqué, poursuivez l'installation de redis.

## 4. Redis

*Redis, qui signifie Remote Dictionary Server (Serveur de dictionnaire à distance), est un système de stockage de données clé-valeur en mémoire, open source et rapide, pour une utilisation en tant que base de données, de cache, de courtier de messages et de file d'attente. Le projet a démarré lorsque Salvatore Sanfilippo, le développeur initial de Redis, a essayé d’améliorer la scalabilité de sa startup italienne. Redis offre désormais des temps de réponse inférieurs à la milliseconde permettant des millions de demandes par seconde pour des applications en temps réel dans les domaines du jeu, de la technologie publicitaire, des services financiers, des soins de santé et de l'Internet des objets. Redis est un choix populaire pour la mise en cache, la gestion de session, les jeux, les classements, l'analyse en temps réel, le géospatial, l'appel de voiture avec chauffeur, le chat/la messagerie, le streaming multimédia et les applications pub/sub.*

Installation

    apt update && apt install redis-server php-redis -y

Changer la configuration et l'appartenance à un groupe

```
cp /etc/redis/redis.conf /etc/redis/redis.conf.bak
sed -i "s/port 6379/port 0/" /etc/redis/redis.conf
sed -i s/\#\ unixsocket/\unixsocket/g /etc/redis/redis.conf
sed -i "s/unixsocketperm 700/unixsocketperm 770/" /etc/redis/redis.conf
sed -i "s/# maxclients 10000/maxclients 512/" /etc/redis/redis.conf
usermod -a -G redis www-data

cp /etc/sysctl.conf /etc/sysctl.conf.bak
sed -i '$avm.overcommit_memory = 1' /etc/sysctl.conf
```

Il est recommandé de redémarrer votre serveur une fois:

    shutdown -r now 

## 5. Nextcloud

Créez les fichiers de configuration, commencez par **/etc/nginx/conf.d/nextcloud.conf**

```
su -
[ -f /etc/nginx/conf.d/default.conf ] && mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.bak
touch /etc/nginx/conf.d/default.conf
nano /etc/nginx/conf.d/nextcloud.conf
```

ATTENTION remplacer **your.dedyn.io** par votre domaine  
Collez les lignes suivantes:

```
server {
	server_name your.dedyn.io;
	listen 80 default_server;
	listen [::]:80 default_server;
			location ^~ /.well-known/acme-challenge {
			proxy_pass http://127.0.0.1:81;
			proxy_set_header Host $host;
			}
			location / {
			return 301 https://$host$request_uri;
			}
}
server {
	server_name your.dedyn.io;
	listen 443 ssl http2 default_server;
	listen [::]:443 ssl http2 default_server;
	root /var/www/nextcloud/;
		location = /robots.txt {
			allow all;
			log_not_found off;
			access_log off;
		}
		location = /.well-known/carddav {
			return 301 $scheme://$host/remote.php/dav;
		}
		location = /.well-known/caldav {
			return 301 $scheme://$host/remote.php/dav;
		}
	#SOCIAL app enabled? Please uncomment the following row
	#rewrite ^/.well-known/webfinger /public.php?service=webfinger last;
	#WEBFINGER app enabled? Please uncomment the following two rows.
	#rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
	#rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json last;
	client_max_body_size 10240M;
		location / {
			rewrite ^ /index.php$request_uri;
		}
		location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
			deny all;
		}
		location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
			deny all;
		}
		location ^~ /apps/rainloop/app/data {
			deny all;
		}
		location ~ \.(?:flv|mp4|mov|m4a)$ {
			mp4;
			mp4_buffer_size 100M;
			mp4_max_buffer_size 1024M;
			fastcgi_split_path_info ^(.+?.php)(\/.*|)$;
			include fastcgi_params;
			include php_optimization.conf;
			fastcgi_pass php-handler;
			fastcgi_param HTTPS on;
		}
		location ~ ^\/(?:index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+).php(?:$|\/) {
			fastcgi_split_path_info ^(.+?.php)(\/.*|)$;
			include fastcgi_params;
			include php_optimization.conf;
			fastcgi_pass php-handler;
			fastcgi_param HTTPS on;
		}
		location ~ ^\/(?:updater|oc[ms]-provider)(?:$|\/) {
			try_files $uri/ =404;
			index index.php;
		}
		location ~ .(?:css|js|woff2?|svg|gif|map|png|html|ttf|ico|jpg|jpeg)$ {
			try_files $uri /index.php$request_uri;
			access_log off;
			expires 360d;
		}
}
```

Si vous souhaitez que votre Nextcloud s'exécute dans un sous-répertoire (sous-dossier) comme https://your.dedyn.io/ nextcloud, utilisez plutôt nextcloud.conf:

```
server {
	server_name your.dedyn.io;
	listen 80 default_server;
	listen [::]:80 default_server;
		location ^~ /.well-known/acme-challenge {
			proxy_pass http://127.0.0.1:81;
			proxy_set_header Host $host;
		}
		location / {
			return 301 https://$host$request_uri;
		}
}
server {
	server_name your.dedyn.io;
	listen 443 ssl http2 default_server;
	listen [::]:443 ssl http2 default_server;
	root /var/www/;
		location = /robots.txt {
			allow all;
			log_not_found off;
			access_log off;
		}
		location = /.well-known/carddav {
			return 301 $scheme://$host/nextcloud/remote.php/dav;
		}
		location = /.well-known/caldav {
			return 301 $scheme://$host/nextcloud/remote.php/dav;
		}
		#SOCIAL app enabled? Please uncomment the following row
		#rewrite ^/.well-known/webfinger /nextcloud/public.php?service=webfinger last;
		#WEBFINGER app enabled? Please uncomment the following two rows.
		#rewrite ^/.well-known/host-meta /nextcloud/public.php?service=host-meta last;
		#rewrite ^/.well-known/host-meta.json /nextcloud/public.php?service=host-meta-json last;
		client_max_body_size 10240M;
		location /nextcloud {
			rewrite ^ /nextcloud/index.php$request_uri;
		}
		location ~ ^/nextcloud/(?:build|tests|config|lib|3rdparty|templates|data)/ {
			deny all;
		}
		location ~ ^/nextcloud/(?:\.|autotest|occ|issue|indie|db_|console) {
			deny all;
		}
		location ^~ /nextcloud/apps/rainloop/app/data {
			deny all;
		}
		location ~ \.(?:flv|mp4|mov|m4a)$ {
			mp4;
			mp4_buffer_size 100M;
			mp4_max_buffer_size 1024M;
			fastcgi_split_path_info ^(.+?.php)(\/.*|)$;
			include fastcgi_params;
			include php_optimization.conf;
			fastcgi_pass php-handler;
			fastcgi_param HTTPS on;
		}
		location ~ ^\/nextcloud/(?:index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+).php(?:$|\/) {
			fastcgi_split_path_info ^(.+?.php)(\/.*|)$;
			include fastcgi_params;
			include php_optimization.conf;
			fastcgi_pass php-handler;
			fastcgi_param HTTPS on;
		}
		location ~ ^\/nextcloud/(?:updater|oc[ms]-provider)(?:$|\/) {
			try_files $uri/ =404;
			index index.php;
		}
		location ~ .(?:css|js|woff2?|svg|gif|map|png|html|ttf|ico|jpg|jpeg)$ {
			try_files $uri /nextcloud/index.php$request_uri;
			access_log off;
			expires 360d;
		}
}
```

Créer le letsencrypt.conf

    nano /etc/nginx/conf.d/letsencrypt.conf

Collez les lignes suivantes:

```
server {
server_name 127.0.0.1;
 listen 127.0.0.1:81 default_server;
 charset utf-8;
 location ^~ /.well-known/acme-challenge {
  default_type text/plain;
  root /var/www/letsencrypt;
 }
}
```

Créer le fichier ssl.conf

    nano /etc/nginx/ssl.conf

Coller les lignes suivantes

```
ssl_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;
ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
ssl_trusted_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;
#ssl_certificate /etc/letsencrypt/rsa-certs/fullchain.pem;
#ssl_certificate_key /etc/letsencrypt/rsa-certs/privkey.pem;
#ssl_certificate /etc/letsencrypt/ecc-certs/fullchain.pem;
#ssl_certificate_key /etc/letsencrypt/ecc-certs/privkey.pem;
#ssl_trusted_certificate /etc/letsencrypt/ecc-certs/chain.pem;
ssl_dhparam /etc/ssl/certs/dhparam.pem;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets off;
ssl_protocols TLSv1.3 TLSv1.2;
ssl_ciphers 'TLS-CHACHA20-POLY1305-SHA256:TLS-AES-256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384';
ssl_ecdh_curve X448:secp521r1:secp384r1:prime256v1;
ssl_prefer_server_ciphers on;
ssl_stapling on;
ssl_stapling_verify on;
```

Créer le proxy.conf

    nano /etc/nginx/proxy.conf

Coller les lignes suivantes

```
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-Host $host;
proxy_set_header X-Forwarded-Protocol $scheme;
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Forwarded-Port $server_port;
proxy_set_header X-Forwarded-Server $host;
proxy_connect_timeout 3600;
proxy_send_timeout 3600;
proxy_read_timeout 3600;
proxy_redirect off;
```

Créer le header.conf

  vi /etc/nginx/header.conf 

Coller les lignes suivantes

```
add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;";
add_header X-Robots-Tag none;
add_header X-Download-Options noopen;
add_header X-Permitted-Cross-Domain-Policies none;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "no-referrer" always;
#add_header Feature-Policy "accelerometer 'none'; autoplay 'self'; geolocation 'none'; midi 'none'; sync-xhr 'self' ; microphone 'self'; camera 'self'; magnetometer 'none'; gyroscope 'none'; speaker 'self'; fullscreen 'self'; payment 'none'; usb 'none'";
add_header X-Frame-Options "SAMEORIGIN";
``` 

Créer optimization.conf

    nano /etc/nginx/optimization.conf 

Coller les lignes suivantes

```
fastcgi_hide_header X-Powered-By;
fastcgi_read_timeout 3600;
fastcgi_send_timeout 3600;
fastcgi_connect_timeout 3600;
fastcgi_buffers 64 64K;
fastcgi_buffer_size 256k;
fastcgi_busy_buffers_size 3840K;
fastcgi_cache_key $http_cookie$request_method$host$request_uri;
fastcgi_cache_use_stale error timeout invalid_header http_500;
fastcgi_ignore_headers Cache-Control Expires Set-Cookie;
gzip on;
gzip_vary on;
gzip_comp_level 4;
gzip_min_length 256;
gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;
gzip_disable "MSIE [1-6]\.";
```

Créez le php_optimization.conf

    nano /etc/nginx/php_optimization.conf 

Coller les lignes suivantes

```
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
fastcgi_param PATH_INFO $fastcgi_path_info;
fastcgi_param modHeadersAvailable true;
fastcgi_param front_controller_active true;
fastcgi_intercept_errors on;
fastcgi_request_buffering off;
fastcgi_cache_valid 404 1m;
fastcgi_cache_valid any 1h;
fastcgi_cache_methods GET HEAD;
```

### Améliorer la sécurité

    openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096

S'il vous plaît soyez patient, cela prendra un certain temps en fonction de votre matériel.

### Redémarrez NGINX

    sed -i s/\#\include/\include/g /etc/nginx/nginx.conf && service nginx restart 

### Téléchargez et extrayez le logiciel Nextcloud, puis demandez vos certificats SSL à Let's Encrypt via acme:

    cd /usr/local/src
    wget https://download.nextcloud.com/server/releases/latest.tar.bz2
    tar -xjf latest.tar.bz2 -C /var/www && chown -R www-data:www-data /var/www/ && rm -f latest.tar.bz2

Créez un utilisateur technique pour installer et renouveler vos certificats SSL

    adduser acmeuser
    usermod -a -G www-data acmeuser
    
Problème visudo

    visudo 

et ajoutez la ligne suivante à la fin du fichier

    acmeuser ALL=NOPASSWD: /bin/systemctl reload nginx.service

par exemple, redémarrer nginx sans mot de passe.

### Pour demander des certificats SSL à letsencrypt, il suffit d'installer acme et de demander votre / vos certificat (s) SSL:

    su - acmeuser
    curl https://get.acme.sh | sh
    exit

### Créez trois dossiers pour demander et stocker vos certificats SSL à (remplacez votre.dedyn.io):

Attention , remplacer **your.dedyn.io** par votre domaine

    sudo -s
    mkdir -p /var/www/letsencrypt/.well-known/acme-challenge /etc/letsencrypt/rsa-certs /etc/letsencrypt/ecc-certs
    chmod -R 775 /var/www/letsencrypt /etc/letsencrypt && chown -R www-data:www-data /var/www/ /etc/letsencrypt
    su - acmeuser
    acme.sh --issue -d your.dedyn.io --keylength 4096 -w /var/www/letsencrypt --key-file /etc/letsencrypt/rsa-certs/privkey.pem --ca-file /etc/letsencrypt/rsa-certs/chain.pem --cert-file /etc/letsencrypt/rsa-certs/cert.pem --fullchain-file /etc/letsencrypt/rsa-certs/fullchain.pem
    acme.sh --issue -d your.dedyn.io --keylength ec-384 -w /var/www/letsencrypt --key-file /etc/letsencrypt/ecc-certs/privkey.pem --ca-file /etc/letsencrypt/ecc-certs/chain.pem --cert-file /etc/letsencrypt/ecc-certs/cert.pem --fullchain-file /etc/letsencrypt/ecc-certs/fullchain.pem
    exit

### Appliquez les autorisations à l'aide d'un script permissions.sh :

    nano /root/permissions.sh 

Collez les lignes suivantes:

```
#!/bin/bash
find /var/www/ -type f -print0 | xargs -0 chmod 0640
find /var/www/ -type d -print0 | xargs -0 chmod 0750
chown -R www-data:www-data /var/www/
chown -R www-data:www-data /var/nc_data/
chmod 0644 /var/www/nextcloud/.htaccess
chmod 0644 /var/www/nextcloud/.user.ini
chmod 600 /etc/letsencrypt/rsa-certs/fullchain.pem
chmod 600 /etc/letsencrypt/rsa-certs/privkey.pem
chmod 600 /etc/letsencrypt/rsa-certs/chain.pem
chmod 600 /etc/letsencrypt/rsa-certs/cert.pem
chmod 600 /etc/letsencrypt/ecc-certs/fullchain.pem
chmod 600 /etc/letsencrypt/ecc-certs/privkey.pem
chmod 600 /etc/letsencrypt/ecc-certs/chain.pem
chmod 600 /etc/letsencrypt/ecc-certs/cert.pem
chmod 600 /etc/ssl/certs/dhparam.pem
exit 0
```

Exécutez le script:

    chmod +x /root/permissions.sh && /root/permissions.sh 

Modifiez le fichier ssl.conf et redémarrez NGINX:

    sed -i '/ssl-cert-snakeoil/d' /etc/nginx/ssl.conf
    sed -i s/\#\ssl/\ssl/g /etc/nginx/ssl.conf
    service nginx restart

Installez Nextcloud en mode silencieux

    su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ maintenance:install --database "mysql" --database-name "nextcloud" --database-user "nextcloud" --database-pass "nextcloud" --admin-user "YourNextcloudAdmin" --admin-pass "YourNextcloudAdminPasssword" --data-dir "/var/nc_data"'

Information

*    –Database-name “ nextcloud ”: comme défini ci-dessus lors de la création de la base de données
*    –Database-user “ nextcloud ”: comme défini ci-dessus lors de la création de l'utilisateur de la base de données
*    –Database-pass “ nextcloud ”: comme défini ci-dessus lors de la création du mot de passe de l'utilisateur
*    –Admin-user “ YourNextcloudAdmin ”: votre libre choix
*    –Admin-pass “ YourNextcloudAdminPasssword ”: votre libre choix 

    su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ config:system:set trusted_domains 0 --value=your.dedyn.io'
    su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ config:system:set overwrite.cli.url --value=https://your.dedyn.io'
    cp /var/www/nextcloud/config/config.php /var/www/nextcloud/config/config.php.bak

Développez votre fichier Nextcloud config.php:

```
sed -i 's/^[ ]*//' /var/www/nextcloud/config/config.php
sed -i '/);/d' /var/www/nextcloud/config/config.php

cat <<EOF >>/var/www/nextcloud/config/config.php
'activity_expire_days' => 14,
'auth.bruteforce.protection.enabled' => true,
'blacklisted_files' => 
array (
0 => '.htaccess',
1 => 'Thumbs.db',
2 => 'thumbs.db',
),
'cron_log' => true,
'enable_previews' => true,
'enabledPreviewProviders' => 
array (
0 => 'OC\\Preview\\PNG',
1 => 'OC\\Preview\\JPEG',
2 => 'OC\\Preview\\GIF',
3 => 'OC\\Preview\\BMP',
4 => 'OC\\Preview\\XBitmap',
5 => 'OC\\Preview\\Movie',
6 => 'OC\\Preview\\PDF',
7 => 'OC\\Preview\\MP3',
8 => 'OC\\Preview\\TXT',
9 => 'OC\\Preview\\MarkDown',
),
'filesystem_check_changes' => 0,
'filelocking.enabled' => 'true',
'htaccess.RewriteBase' => '/',
'integrity.check.disabled' => false,
'knowledgebaseenabled' => false,
'logfile' => '/var/nc_data/nextcloud.log',
'loglevel' => 2,
'logtimezone' => 'Europe/Berlin',
'log_rotate_size' => 104857600,
'maintenance' => false,
'memcache.local' => '\\OC\\Memcache\\APCu',
'memcache.locking' => '\\OC\\Memcache\\Redis',
'overwriteprotocol' => 'https',
'preview_max_x' => 1024,
'preview_max_y' => 768,
'preview_max_scale_factor' => 1,
'redis' => 
array (
'host' => '/var/run/redis/redis-server.sock',
# ATTENTION if you operate on Debian 9.x: 
# 'host' => '/var/run/redis/redis.sock',
'port' => 0,
'timeout' => 0.0,
),
'quota_include_external_storage' => false,
'share_folder' => '/Shares',
'skeletondirectory' => '',
'theme' => '',
'trashbin_retention_obligation' => 'auto, 7',
'updater.release.channel' => 'stable',
);
EOF
```

### Editer le Nextcloud .user.ini

    sudo -u www-data sed -i "s/output_buffering=.*/output_buffering='Off'/" /var/www/nextcloud/.user.ini
    service php7.3-fpm restart && service redis-server restart && service nginx restart


### Ajuster les applications Nextcloud

```
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ app:disable survey_client'
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ app:disable firstrunwizard'
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ app:enable admin_audit'
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ app:enable files_pdfviewer'
```

Optimisez votre Nextcloud avec deux scripts shell

(a) mettre à jour votre environnement périodiquement [upgrade.sh](https://github.com/criegerde/install-nextcloud/blob/master/maintenance/debian/upgrade-debian.sh)

    nano /root/upgrade.sh

```
# Debian 9.x 10.x
#!/bin/bash
/usr/sbin/service nginx stop
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/updater/updater.phar'
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ status'
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ -V'
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ db:add-missing-indices'
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ db:convert-filecache-bigint'
sed -i "s/output_buffering=.*/output_buffering='Off'/" /var/www/nextcloud/.user.ini
chown -R www-data:www-data /var/www/nextcloud
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ app:update --all'
/usr/sbin/service php7.3-fpm restart
/usr/sbin/service nginx restart
exit 0
```

>(infos: [BigInt](https://docs.nextcloud.com/server/15/admin_manual/configuration_database/bigint_identifiers.html) , [indices manquants](https://docs.nextcloud.com/server/15/admin_manual/configuration_server/occ_command.html?highlight=add%20missing%20indices#add-missing-indices) ) 

(b) optimisez périodiquement votre Nextcloud [Texte du lien](https://github.com/criegerde/install-nextcloud/blob/master/maintenance/debian/optimize-debian.sh)

    nan /root/optimize.sh

```
#!/bin/bash
# ATTENTION if you operate on Debian 9.x: 
# redis-cli -s /var/run/redis/redis.sock <<EOF
redis-cli -s /var/run/redis/redis-server.sock <<EOF
FLUSHALL
quit
EOF
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ files:scan --all'
su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ files:scan-app-data'
exit 0
```

Sauvegarder les deux scripts et les marquer comme exécutables

    chmod +x /root/*.sh 

Émettez les deux fois:

    /root/upgrade.sh && /root/optimize.sh

Ajouter des tâches cron pour Nextcloud pour www-data et root

Pour www-data :

    crontab -u www-data -e

Coller les lignes suivantes

`*/5 * * * * php -f /var/www/nextcloud/cron.php > /dev/null 2>&1`

Pour root :

    crontab -e 

Collez les lignes suivantes:

`5 1 * * * /root/optimize.sh 2>&1` 

### Basculez Nextcloud pour utiliser cron.php

    su - www-data -s /bin/bash -c 'php /var/www/nextcloud/occ background:cron'

### Redémarrer tous les services

    service mysql restart && service php7.3-fpm restart && service redis-server restart && service nginx restart

### Connectez-vous à votre tout nouveau Nextcloud dans votre navigateur

https://your.dedyn.io/login

Si la vérification d'intégrité dans Nextcloud échoue, essayez de modifier le fichier config.php.

    sudo -u www-data sed -i "s/.*integrity.check.disabled.*/'integrity.check.disabled' => true,/g" /var/www/nextcloud/config/config.php 

Relancez le contrôle d'intégrité et redéfinissez la valeur sur 'false':

    sudo -u www-data sed -i "s/.*integrity.check.disabled.*/'integrity.check.disabled' => false,/g" /var/www/nextcloud/config/config.php


Actualisez le adminpanel (F5) et le message devrait disparaître!

## 6. Durcissez votre système avec fail2ban et ufw

Commencez par installer et configurer fail2ban, puis configurez le pare-feu ufw pour sécuriser Nextcloud.

### Installer et configurer fail2ban

    apt update && apt install fail2ban -y

Collez les lignes suivantes dans le filtre fail2ban pour Nextcloud (ou [téléchargez-les en tant que fichier txt](https://privat.c-rieger.de/s/88dQxRYYsoR3K2Y/download?path=%2F&files=nextcloud.txt) pour éviter les problèmes de code WordPress!):

    nano /etc/fail2ban/filter.d/nextcloud.conf 

```
[Definition]
failregex=^{"reqId":".*","remoteAddr":".*","app":"core","message":"Login failed: '.*' \(Remote IP: '<HOST>'\)","level":2,"time":".*"}$
            ^{"reqId":".*","level":2,"time":".*","remoteAddr":".*","app":"core".*","message":"Login failed: '.*' \(Remote IP: '<HOST>'\)".*}$
            ^.*\"remoteAddr\":\"<HOST>\".*Trusted domain error.*$
```

Collez les lignes suivantes dans la prison fail2ban pour Nextcloud:

    nano /etc/fail2ban/jail.d/nextcloud.local

```
[nextcloud]
backend = auto
enabled = true
port = 80,443
protocol = tcp
filter = nextcloud
maxretry = 3
bantime = 36000
findtime = 36000
logpath = /var/nc_data/nextcloud.log

[nginx-http-auth]
enabled = true
```

### Redémarrez le service fail2ban

    service fail2ban restart && fail2ban-client status nextcloud

### Configurez votre ufw (pare-feu simple)

    apt install ufw -y
    ufw allow 80/tcp && ufw allow 443/tcp && ufw allow 22/tcp && ufw logging medium
    ufw default deny incoming && ufw enable && service ufw restart

## 6.1 Renforcez votre Nextcloud avec Spamhaus Project et UFW

Si vous souhaitez éviter les «visiteurs non privilégiés», créez simplement le script /root/ufw-spamhaus.sh et bloquez-les directement par ufw.

    nano /root/ufw-spamhaus.sh

Collez les lignes suivantes:

```
#!/bin/bash
# Thanks to @ank0m
EXEC_DATE=`date +%Y-%m-%d`
SPAMHAUS_DROP="/usr/local/src/drop.txt"
SPAMHAUS_eDROP="/usr/local/src/edrop.txt"
URL="https://www.spamhaus.org/drop/drop.txt"
eURL="https://www.spamhaus.org/drop/edrop.txt"
DROP_ADD_TO_UFW="/usr/local/src/DROP2.txt"
eDROP_ADD_TO_UFW="/usr/local/src/eDROP2.txt"
DROP_ARCHIVE_FILE="/usr/local/src/DROP_$EXEC_DATE"
eDROP_ARCHIVE_FILE="/usr/local/src/eDROP_$EXEC_DATE"
# All credits for the following BLACKLISTS goes to "The Spamhaus Project" - https://www.spamhaus.org
echo "Start time: $(date)"
echo " "
echo "Download daily DROP file:"
wget -q -O - "$URL" > $SPAMHAUS_DROP
grep -v '^;' $SPAMHAUS_DROP | cut -d ' ' -f 1 > $DROP_ADD_TO_UFW
echo " "
echo "Extract DROP IP addresses and add to UFW:"
cat $DROP_ADD_TO_UFW | while read line
do
/usr/sbin/ufw insert 1 deny from "$line" comment 'DROP_Blacklisted_IPs'
done
echo " "
echo "Downloading eDROP list and import to UFW"
echo " "
echo "Download daily eDROP file:"
wget -q -O - "$eURL" > $SPAMHAUS_eDROP
grep -v '^;' $SPAMHAUS_eDROP | cut -d ' ' -f 1 > $eDROP_ADD_TO_UFW
echo " "
echo "Extract eDROP IP addresses and add to UFW:"
cat $eDROP_ADD_TO_UFW | while read line
do
/usr/sbin/ufw insert 1 deny from "$line" comment 'eDROP_Blacklisted_IPs'
done
echo " "
#####
## To remove or revert these rules, keep the list of IPs!
## Run a command like so to remove the rules:
# while read line; do ufw delete deny from $line; done < $ARCHIVE_FILE
#####
echo "Backup DROP IP address list:"
mv $DROP_ADD_TO_UFW $DROP_ARCHIVE_FILE
echo " "
echo "Backup eDROP IP address list:"
mv $eDROP_ADD_TO_UFW $eDROP_ARCHIVE_FILE
echo " "
echo End time: $(date)
```

Rendre le script exuable en émettant

    chmod +x /root/ufw-spamhaus.sh 

et configurez-le dans votre crontab pour qu'il soit émis automatiquement.

    (crontab -l ; echo "10 2 * * * /root/ufw-spamhaus.sh 2>&1") | crontab -u root -

Enfin effectuer une première exécution

    /root/ufw-spamhaus.sh

et de nombreuses règles UFW seront appliquées immédiatement. Soyez patient, cela peut prendre un certain temps.

## 7. Surveillez l'ensemble de votre système à l'aide de netdata

Démarrer le téléchargement netdata - le répertoire 'netdata' sera créé

```
apt install apache2-utils git gcc make autoconf automake pkg-config uuid-dev zlib1g-dev
cd /usr/local/src
git clone https://github.com/firehol/netdata.git --depth=1
cd netdata
```

Créez un fichier de mot de passe pour protéger netdata:

    htpasswd -c /etc/nginx/netdata-access YourName

Ensuite, exécutez le script netdata-installer.sh avec les privilèges root pour créer, installer et démarrer netdata.

    ./netdata-installer.sh

Netdata est déjà installé. Nous ferons de petits ajustements à la configuration de netdata:

    nano /etc/netdata/netdata.conf 

Dans un premier temps, nous modifions la valeur pour «historique», par exemple, 14400 (4 heures de conservation des données de graphique, utilise environ 60 Mo de RAM) dans la section [global]:

    history = 14400

Ensuite, nous modifions la liaison dans la section [web] en localhost (127.0.0.1) uniquement:

    bind to = 127.0.0.1

Enfin, nous améliorons les fichiers nextcloud.conf et nginx.conf pour inclure la configuration du serveur Web Netdata:

    nano /etc/nginx/conf.d/nextcloud.conf 

Collez les lignes comme indiqué ci-dessous dans le fichier nextcloud.conf:

```
...
location / {
 rewrite ^ /index.php$request_uri;
 }
location /netdata {
 return 301 /netdata/;
 }
 ### début insertion
 location ~ /netdata/(?<ndpath>.*) {
 auth_basic "Restricted Area";
 auth_basic_user_file /etc/nginx/netdata-access;
 proxy_http_version 1.1;
 proxy_pass_request_headers on;
 proxy_set_header Connection "keep-alive";
 proxy_store off;
 proxy_pass http://netdata/$ndpath$is_args$args;
 gzip on;
 gzip_proxied any;
 gzip_types *;
 }
 ### fin insertion
 location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
 deny all;
...
```

Créez le nouveau fichier /etc/nginx/conf.d/stub_status.conf:

    nano /etc/nginx/conf.d/stub_status.conf 

Collez toutes les lignes suivantes:

```
server {
listen 127.0.0.1:80 default_server;
server_name 127.0.0.1;
location /stub_status {
stub_status on;
allow 127.0.0.1;
deny all;
}
}
```

Enregistrez et quittez le fichier (: wq!) Et modifiez le fichier /etc/nginx/nginx.conf:

```
...
http {
 server_names_hash_bucket_size 64;
 upstream php-handler {
 server unix:/run/php/php7.3-fpm.sock;
 }
 ## début modif
 upstream netdata {
 server 127.0.0.1:19999;
 keepalive 64;
 }
 ## fin modif
...
```

Enregistrez et quittez le fichier (: wq!) Et vérifiez NGINX

    nginx -t

Si aucune erreur n'apparaît, redémarrez netdata et nginx

    service netdata restart && service nginx restart

et appelez netdata dans votre navigateur

  https: // votre.dedyn.io / netdata 

ou en tant que site externe dans votre Nextcloud.


## 8. Montez un espace de stockage supplémentaire sur votre Nextcloud

(a) …[using a NAS](https://www.c-rieger.de/nextcloud-installation-guide-debian/#storage01)

(b) …[using an external/additional HDD](https://www.c-rieger.de/nextcloud-installation-guide-debian/#storage02)

(c) … [using Nextclouds external storage app](https://www.c-rieger.de/nextcloud-installation-guide-debian/#storage03)

Vous pouvez améliorer votre Nextcloud avec des données de votre partage NAS ou d’un disque dur externe.

### (a) montez vos données NAS sur un utilisateur Nextcloud spécifique

Il est très simple de monter un partage NAS sur votre Nextcloud à l’aide de cifs. Commencez par installer cifs-utils:

    apt install cifs-utils -y 

Puis stockez vos informations d'identification dans un fichier spécial (par exemple, /root/.smbcredentials )

    nano /root/.smbcredentials

Ecrivez votre nom d'utilisateur et mot de passe:

```
username=NASuser
password=NASPassword
```

Enregistrez et quittez le fichier (: wq!) Et modifiez les autorisations en 0600:

    chmod 400 /root/.smbcredentials

Détectez l'ID de l'utilisateur Web (www-data) à l'aide de la commande id:

    id www-data

et gardez l’identifiant à l’esprit pour le réutiliser dans / etc / fstab :

    cp /etc/fstab /etc/fstab.bak
    vi /etc/fstab

Coller ce qui suit jusqu'à la fin du fstab

```
//<NAS>/<share> /var/nc_data/next/files cifs user,uid=33,rw,iocharset=utf8,suid,credentials=/root/.smbcredentials,file_mode=0770,dir_mode=0770 0 0
```

Veuillez remplacer **“//<NAS>/<share>“** par le suivant et, si nécessaire, l'uid = « 33 », puis essayez de monter votre NAS manuellement:

    mount //<NAS>/<share>/
    ou
    mount -a

Pour démonter votre NAS, lancez manuellement

    umount //<NAS>/<share>/
    ou
    umount -a 

Il sera nécessaire de numériser à nouveau vos données pour la première utilisation. Allez donc dans votre répertoire Nextcloud et exécutez les fichiers Nextclouds : recherchez l’utilisateur Nextcloud approprié (par exemple, next ) ou all ( –all ):

```
service nginx stop
cd /var/www/nextcloud
redis-cli -s /var/run/redis/redis.sock
FLUSHALL
quit
sudo -u www-data php occ files:scan --all -v
sudo -u www-data php occ files:scan-app-data -v
service nginx start
```

Après les fichiers Nextclouds: analyser toutes vos données NAS apparaîtra dans l'application de fichier Nextcloud.  
Le script de permissions <permission.sh> devrait être amélioré pour démonter et monter le nouveau partage NAS monté:

    nano /root/permissions.sh 

Ajoutez les lignes rouges au script existant:

```
#!/bin/bash
find /var/www/ -type f -print0 | xargs -0 chmod 0640
find /var/www/ -type d -print0 | xargs -0 chmod 0750
chown -R www-data:www-data /var/www/
umount //<NAS>/<share>
chown -R www-data:www-data /var/nc_data/
mount //<NAS>/<share>
chmod 0644 /var/www/nextcloud/.htaccess
chmod 0644 /var/www/nextcloud/.user.ini
chmod 600 /etc/letsencrypt/rsa-certs/fullchain.pem
chmod 600 /etc/letsencrypt/rsa-certs/privkey.pem
chmod 600 /etc/letsencrypt/rsa-certs/chain.pem
chmod 600 /etc/letsencrypt/rsa-certs/cert.pem
chmod 600 /etc/letsencrypt/ecc-certs/fullchain.pem
chmod 600 /etc/letsencrypt/ecc-certs/privkey.pem
chmod 600 /etc/letsencrypt/ecc-certs/chain.pem
chmod 600 /etc/letsencrypt/ecc-certs/cert.pem
chmod 600 /etc/ssl/certs/dhparam.pem
```

Veuillez substituer les `mount umount //<NAS>/<share>` en fonction de votre environnement, puis enregistrez et quittez (: wq!) Le fichier. A partir de maintenant, votre NAS sera toujours disponible dans Nextcloud pour l'utilisateur spécifique.

### (b) monter un disque dur externe sur votre Nextcloud

Nous préparons le nouveau lecteur '/dev/sda ' pour utilisation dans Nextcloud. Formatez-le avec un système de fichiers 'ext4' et montez-le de manière permanente avec une entrée dans / etc / fstab.

Arrêtez vos services de serveur (NGINX, PHP, MariaDB, Redis) et vérifiez la disponibilité du nouveau lecteur:

```
sudo -s
service nginx stop && service php7.3-fpm stop && service redis-server stop && service mysql stop
fdisk -l /dev/sda
```

Si disponible, créez une nouvelle partition avec la commande fdisk.

    fdisk /dev/sda

1.    Tapez 'o' pour créer une nouvelle table de partition.
2.    Tapez 'n' pour créer une nouvelle partition.
3.    Choisissez le type de partition principale, entrez 'p'.
4.    Numéro de partition - nous avons juste besoin de 1.
5.    Laissez toutes les valeurs par défaut sur le premier secteur et le dernier secteur - Appuyez sur Entrée.
6.    Tapez 'w' et appuyez sur Entrée pour écrire la partition. 

La partition ‘/dev/sda1’ a été créée, nous devons maintenant la formater en 'ext4' avec l'outil mkfs. Ensuite, vérifiez la taille du volume.

```
mkfs.ext4 /dev/sda1
fdisk -s /dev/sda1
```

Ensuite, créez un nouveau répertoire local "nc_data" et montez "/ dev / sda1" sur ce répertoire.

    mkdir -p /nc_data

Pour monter définitivement un nouveau disque, nous ajoutons la nouvelle configuration de montage au fichier fstab. Fstab ouvert avec vom:

    nano /etc/fstab

Collez la configuration ci-dessous à la fin du fichier.

    /dev/sda1     /nc_data     ext4     defaults     0     1

Enregistrer fstab et quitter:

Maintenant, montez le disque et assurez-vous qu'il n'y a pas d'erreur.

    mount -a
    df -h

Au moins, vous devez déplacer votre répertoire de données Nextcloud actuel vers le nouveau répertoire monté.

    chown -R www-data:www-data /nc_data
    rsync -av /var/nc_data/ /nc_data

et pointez-le dans le fichier config.php de Nextcloud.

    sudo -u www-data vi /var/www/nextcloud/config/config.php

Changer le répertoire de données

```
...
'datadirectory' => '/nc_data',
...
```

Enfin, redémarrez les services de votre serveur et effectuez une nouvelle analyse de fichiers:

```
service nginx stop && service php7.3-fpm restart && service redis-server restart && service mysql restart
cd /var/www/nextcloud
redis-cli -s /var/run/redis/redis.sock 
FLUSHALL
quit
sudo -u www-data php occ files:scan --all -v
sudo -u www-data php occ files:scan-app-data -v
service nginx restart
```

A partir de maintenant, vos données Nextcloud seront stockées sur votre disque dur externe.

### (c) Application de stockage externe Nextclouds

En tant qu'extension pour ( a ) et ( b ), vous pouvez activer l' [application de stockage externe](https://docs.nextcloud.com/server/15/admin_manual/configuration_files/external_storage_configuration_gui.html#configuring-external-storage-gui) et bénéficier de nombreux avantages:

*    les fichiers peuvent être créés, édités et supprimés des deux côtés: à l'intérieur et à l'extérieur de Nextcloud
*    vous êtes autorisé à monter des services et des périphériques de stockage externes en tant que périphériques de stockage Nextcloud secondaires
*    les utilisateurs sont autorisés à monter leurs propres services de stockage externe 

Si vous souhaitez utiliser Samba, émettez la déclaration suivante

    apt install php-smbclient smbclient -y

et redémarrez PHP

    service php7.3-fpm restart 

## 9. Installez msmtp pour envoyer des mails au serveur

(a) [configure fail2ban system-notification mails](https://www.c-rieger.de/nextcloud-installation-guide-debian/#fail2banmails)

(b) [install apticron and configure system update-notification mails](https://www.c-rieger.de/nextcloud-installation-guide-debian/#apticronmails)

Première installation msmtp

    sudo -s 
    apt update && apt upgrade -y && apt install msmtp msmtp-mta mailutils -y

Créez votre configuration: créez le fichier, puis collez et modifiez les lignes suivantes:

    nano /etc/msmtprc

```
defaults
port 587
tls on
tls_starttls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
#Your Mail:
account yourmail@domain.com
#Your SMTP-Server:
host smtp.domain.com
#Mails will be sent from:
from yourmail@domain.com
auth on 
#Your Mailaccount:
user yourmail@domain.com
#Your Password:
password yOUr-S3CrET
#Default Mailaccount:
account default: yourmail@domain.com
aliases /etc/aliases
# find out more about the configuration here: https://marlam.de/msmtp/msmtprc.txt
logfile /var/log/msmtp/msmtp.log
```

Définir la permission appropriée

    chmod 600 /etc/msmtprc

et testez votre configuration de serveur de courrier en émettant

    echo "Sending test mail..." | mail -s "Subject" yourmail@domain.com

Modifiez logrotate pour gérer correctement msmtp. Créer et éditer le fichier

    nano /etc/logrotate.d/msmtp

et collez toutes les lignes suivantes:

```
/var/log/msmtp/*.log {
rotate 12
monthly
compress
missingok
notifempty
}
```

Modifier PHP pour utiliser msmtp dans PHP

    nano /etc/php/7.3/fpm/php.ini

Définissez le sendmail_path comme suit

    sendmail_path = "/usr/bin/msmtp -t"

et redémarrez php

    service php7.3-fpm restart 

Définissez enfin vos alias de messagerie: ouvrez le fichier

    nano /etc/aliases

coller et modifier les lignes suivantes:

```
root: yourmail@domain.com
default: yourmail@domain.com
```

A partir de maintenant votre système est préparé pour envoyer des mails. Si vous voulez que fail2ban vous tienne au courant, suivez le chapitre suivant:

### (a) configurer les mails de notification du système fail2ban

Nous substituons l'utilisateur root dans fail2ban-config pour recevoir les mails d'état de fail2ban à l'avenir. Ces e-mails contiendront à la fois le statut fail2ban (arrêté / démarré) et, en cas d'échec de la connexion, l'adresse IP bannie. Editez le fichier de configuration fail2ban

    cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.conf.bak
    nano /etc/fail2ban/jail.conf

et remplacez au moins les paramètres **yourmail@domain.com, mail, action = %(action_mwl)s**  en fonction de votre système:

```
...
destemail = yourmail@domain.com
...
sender = yourmail@domain.com
...
mta = mail
...
# action = %(action_)s
action = %(action_mwl)s
...
```

Enregistrez et quittez (: wq!) La configuration fail2ban. Pour éviter (plusieurs) mails sur chaque redémarrage fail2ban, créez simplement un nouveau fichier et copiez-le comme indiqué ci-dessous:

    nano /etc/fail2ban/action.d/mail-buffered.local

Coller les lignes suivantes

```
[Definition]
actionstart =
actionstop =
```

Copier le fichier

```
cp /etc/fail2ban/action.d/mail-buffered.local /etc/fail2ban/action.d/mail.local
cp /etc/fail2ban/action.d/mail-buffered.local /etc/fail2ban/action.d/mail-whois-lines.local
cp /etc/fail2ban/action.d/mail-buffered.local /etc/fail2ban/action.d/mail-whois.local
cp /etc/fail2ban/action.d/mail-buffered.local /etc/fail2ban/action.d/sendmail-buffered.local
cp /etc/fail2ban/action.d/mail-buffered.local /etc/fail2ban/action.d/sendmail-common.local
```

Redémarrez le service fail2ban et vous serez (seulement) informé si Fail2ban a bloqué de nouvelles adresses IP

    service fail2ban restart

automatiquement

### (b) installer apticron et configurer les mails de notification de mise à jour du système

Si vous utilisez APTICRON, votre système peut également envoyer des courriels en cas de mises à jour système disponibles.

    apt install apticron -y

Après avoir installé APTICRON, vous devez modifier la configuration et substituer au moins votre adresse EMAIL, SYSTEM, NOTIFY_NO_UPDATES et CUSTOM_FROM.

    cp /etc/apticron/apticron.conf /etc/apticron/apticron.conf.bak
    nano /etc/apticron/apticron.conf

```
...
EMAIL="yourmail@domain.com"
...
SYSTEM="your.dedyn.io"
...
NOTIFY_HOLDS="1"
...
NOTIFY_NO_UPDATES="1"
...
CUSTOM_SUBJECT='$SYSTEM: $NUM_PACKAGES package update(s)'
...
CUSTOM_NO_UPDATES_SUBJECT='$SYSTEM: no updates available'
...
CUSTOM_FROM="yourmail@domain.com"
...
```

Pour lancer et vérifier APTICRON, appelez simplement

    apticron

et vous recevrez un email envoyé par APTICRON. Maintenant, vous êtes un peu plus en sécurité.

    cp /etc/cron.d/apticron /etc/cron.d/apticron.bak
    nano /etc/cron.d/apticron

`30 7 * * * root if test -x /usr/sbin/apticron; then /usr/sbin/apticron --cron; else true; fi`

Apticron va maintenant être exécuté par cron.d. Vous pouvez modifier l’heure de début, par exemple, tous les jours à 7 h 30.

## 10. Un deuxième facteur pour ssh (2FA - authentification à deux facteurs)

Les étapes suivantes concernent le système (critique) et ne sont recommandées que pour les utilisateurs avancés de Linux. Si la configuration de ssh échoue, vous ne pourrez plus vous connecter à votre système via ssh. La condition préalable requise est un serveur ssh sur lequel vous pouvez vous connecter à l'aide d'une clé privée / publique uniquement!

Installez le logiciel pour 2FA (authentification à deux facteurs) avec votre application OTP AUTH préférée.

    apt install libpam-google-authenticator -y

Quittez le shell racine et exécutez la commande suivante en tant que votre <nom de votre-utilisateur-ubuntu > et PAS en tant que root:

    exit
    google-authenticator
    
On vous demandera:

```
Do you want authentication tokens to be time-based (y/n) y

image

Do you want me to update your "~/.google_authenticator" file (y/n) y

Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y

By default, tokens are good for 30 seconds and in order to compensate for
possible time-skew between the client and the server, we allow an extra
token before and after the current time. If you experience problems with poor
time synchronization, you can increase the window from its default
size of 1:30min to about 4min. Do you want to do so (y/n) n

If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting (y/n) y
```

Revenez à la racine-Shell

    sudo -s 

Sauvegardez la configuration actuelle et configurez votre serveur ssh

    cp /etc/pam.d/sshd /etc/pam.d/sshd.bak
    nano /etc/pam.d/sshd

Changer le fichier à la mienne:

```
@include common-auth
@include common-password
auth required pam_google_authenticator.so
account required pam_nologin.so
@include common-account
session [success=ok ignore=ignore module_unknown=ignore default=bad] pam_selinux.so close
session required pam_loginuid.so
session optional pam_keyinit.so force revoke
@include common-session
session optional pam_motd.so motd=/run/motd.dynamic
session optional pam_motd.so noupdate
session optional pam_mail.so standard noenv # [1]
session required pam_limits.so
session required pam_env.so # [1]
session required pam_env.so user_readenv=1 envfile=/etc/default/locale
session [success=ok ignore=ignore module_unknown=ignore default=bad] pam_selinux.so open
``` 

Sauvegarder et quitter (: wq!) Le fichier.

S'il n'est pas déjà créé, créez d'abord votre clé RSA 4096 bits (SSH):

    cd ~
    ssh-keygen -q -f /etc/ssh/ssh_host_rsa_key -N '' -b 4096 -t rsa

Si vous êtes invité à écraser la clé existante, confirmez avec "Y". Ensuite, sauvegardez, éditez et changez votre configuration SSH en un exemple

    mv /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
    nano /etc/ssh/sshd_config
    
```
# Port 22
Port 1234 #your decision, but keep UFW in mind!
Protocol 2
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
UsePrivilegeSeparation yes
KeyRegenerationInterval 3600
ServerKeyBits 4096
SyslogFacility AUTH
LogLevel INFO
LoginGraceTime 30s
PermitRootLogin no
StrictModes yes
RSAAuthentication yes
PubkeyAuthentication yes
IgnoreRhosts yes
UseDNS yes
RhostsRSAAuthentication no
HostbasedAuthentication no
IgnoreUserKnownHosts yes
PermitEmptyPasswords no
MaxAuthTries 3
MaxSessions 3
ChallengeResponseAuthentication yes
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost no
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
Banner /etc/issue
AcceptEnv LANG LC_*
Subsystem sftp /usr/lib/openssh/sftp-server
UsePAM yes
AllowTcpForwarding no
AllowUsers ubuntuuser #<your-ubuntu-user-name> for e.g. putty or ssh native
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

Si vous avez changé le port ssh en 1234 , assurez-vous d'avoir modifié votre configuration ufw et ajustez le nom d'utilisateur dans 'AllowUsers ubuntuuser ' .

Collez votre clé publique dans le magasin de clés de < ubuntuuser > (le guide d'utilisation d' ubuntu):

    nano ~/.ssh/authorized_keys 

et définir les autorisations appropriées:

```
sudo chown -R ubuntuuser:ubuntuuser ~/.ssh
sudo chmod 700 ~/.ssh
sudo chmod 600 ~/.ssh/authorized_keys
```

Puis redémarrez votre serveur ssh

    service ssh restart

et reconnectez-vous à votre serveur en utilisant une nouvelle fenêtre de session . Ceci est votre dernier recours si vous avez mal configuré votre serveur ssh ;-). A partir de maintenant votre clé privée est nécessaire, vous serez invité à entrer votre mot de passe et enfin votre nouveau deuxième facteur.

image1  
Authentification par clé publique et mot de passe ssh-user

image2  
Code de vérification (OTP 2FA)

Démarrez votre application, par exemple OTP AUTH ou Google Authenticator, et lisez votre deuxième facteur pour accéder à votre serveur.

image3  
Connecté

Vous serez connecté en utilisant votre deuxième facteur.

## 11. Analysez votre serveur en utilisant logwatch

Première installation de logwatch

    apt update && apt install logwatch -y

puis copiez les fichiers de configuration par défaut dans le dossier logwatch:

```
cp /usr/share/logwatch/default.conf/logfiles/http.conf /etc/logwatch/conf/logfiles/nginx.conf
cp /usr/share/logwatch/default.conf/services/http.conf /etc/logwatch/conf/services/nginx.conf
cp /usr/share/logwatch/scripts/services/http /usr/share/logwatch/scripts/services/nginx
cp /usr/share/logwatch/default.conf/services/http-error.conf /etc/logwatch/conf/services/nginx-error.conf
cp /usr/share/logwatch/scripts/services/http-error /etc/logwatch/scripts/services/nginx-error
cp /etc/logwatch/conf/logfiles/nginx.conf /etc/logwatch/conf/logfiles/nginx.conf.org.bak
```

Editez le fichier /etc/logwatch/conf/logfiles/nginx.conf pour le mien

    nano /etc/logwatch/conf/logfiles/nginx.conf

Remplacez le fichier entier par:

```
########################################################
# Define log file group for NGINX
########################################################

# What actual file? Defaults to LogPath if not absolute path....
#LogFile = httpd/*access_log
#LogFile = apache/*access.log.1
#LogFile = apache/*access.log
#LogFile = apache2/*access.log.1
#LogFile = apache2/*access.log
#LogFile = apache2/*access_log
#LogFile = apache-ssl/*access.log.1
#LogFile = apache-ssl/*access.log
LogFile = nginx/*access.log
LogFile = nginx/*error.log
LogFile = nginx/*access.log.1
LogFile = nginx/*error.log.1

# If the archives are searched, here is one or more line
# (optionally containing wildcards) that tell where they are...
#If you use a "-" in naming add that as well -mgt
#Archive = archiv/httpd/*access_log.*
#Archive = httpd/*access_log.*
#Archive = apache/*access.log.*.gz
#Archive = apache2/*access.log.*.gz
#Archive = apache2/*access_log.*.gz
#Archive = apache-ssl/*access.log.*.gz
#Archive = archiv/httpd/*access_log-*
#Archive = httpd/*access_log-*
#Archive = apache/*access.log-*.gz
#Archive = apache2/*access.log-*.gz
#Archive = apache2/*access_log-*.gz
#Archive = apache-ssl/*access.log-*.gz
Archive = nginx/*access.log.*.gz
Archive = nginx/*error.log.*.gz

# Expand the repeats (actually just removes them now)
*ExpandRepeats

# Keep only the lines in the proper date range...
*ApplyhttpDate

# vi: shiftwidth=3 tabstop=3 et
```

Enregistrez et quittez (: wq!) Ce fichier et modifiez /etc/logwatch/conf/services/nginx.conf:

    cp /etc/logwatch/conf/services/nginx.conf /etc/logwatch/conf/services/nginx.conf.org.bak
    vi /etc/logwatch/conf/services/nginx.conf

Changez le nom de http en NGINX ou remplacez le fichier entier par le mien:

```
###########################################################################
# Configuration file for NGINX filter
###########################################################################

Title = "NGINX"

# Which logfile group...
LogFile = NGINX

# Define the log file format
#
# This is now the same as the LogFormat parameter in the configuration file
# for httpd. Multiple instances of declared LogFormats in the httpd
# configuration file can be declared here by concatenating them with the
# '|' character. The default, shown below, includes the Combined Log Format,
# the Common Log Format, and the default SSL log format.
#$LogFormat = "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"|%h %l %u %t \"%r\" %>s %b|%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"

# The following is supported for backwards compatibility, but deprecated:
# Define the log file format
#
# the only currently supported fields are:
# client_ip
# request
# http_rc
# bytes_transfered
# agent
#
#$HTTP_FIELDS = "client_ip ident userid timestamp request http_rc bytes_transfered referrer agent"
#$HTTP_FORMAT = "space space space brace quote space space quote quote"
# Define the field formats
#
# the only currently supported formats are:
# space = space delimited field
# quote = quoted ("..") space delimited field
# brace = braced ([..]) space delimited field

# Flag to ignore 4xx and 5xx error messages as possible hack attempts
#
# Set flag to 1 to enable ignore
# or set to 0 to disable
$HTTP_IGNORE_ERROR_HACKS = 0

# Ignore requests
# Note - will not do ANY processing, counts, etc... just skip it and go to
# the next entry in the log file.
# Note - The match will be case insensitive; e.g. /model/ == /MoDel/
# Examples:
# 1. Ignore all URLs starting with /model/ and ending with 1 to 10 digits
# $HTTP_IGNORE_URLS = ^/model/\d{1,10}$
#
# 2. Ignore all URLs starting with /model/ and ending with 1 to 10 digits and
# all URLS starting with /photographer and ending with 1 to 10 digits
# $HTTP_IGNORE_URLS = ^/model/\d{1,10}$|^/photographer/\d{1,10}$
# or simply:
# $HTTP_IGNORE_URLS = ^/(model|photographer)/\d{1,10}$

# To ignore a range of IP addresses completely from the log analysis,
# set $HTTP_IGNORE_IPS. For example, to ignore all local IP addresses:
#
# $HTTP_IGNORE_IPS = ^10\.|^172\.(1[6-9]|2[0-9]|3[01])\.|^192\.168\.|^127\.
#

# For more sophisticated ignore rules, you can define HTTP_IGNORE_EVAL
# to an arbitrary chunk of code.
# The default is not to filter anything:
$HTTP_IGNORE_EVAL = 0
# Example:
# $HTTP_IGNORE_EVAL = "($field{http_rc} == 401) && ($field{client_ip}=~/^192\.168\./) && ($field{url}=~m%^/protected1/%)"
# See the "scripts/services/http" script for other variables that can be tested.

# The variable $HTTP_USER_DISPLAY defines which user accesses are displayed.
# The default is not to display user accesses:
$HTTP_USER_DISPLAY = 0
# To display access failures:
# $HTTP_USER_DISPLAY = "$field{http_rc} >= 400"
# To display all user accesses except "Unauthorized":
# $HTTP_USER_DISPLAY = "$field{http_rc} != 401"

# To raise the needed level of detail for one or more specific
# error codes to display a summary instead of listing each
# occurrence, set a variable like the following ones:
# Raise 403 codes to detail level High
#$http_rc_detail_rep_403 = 10
# Always show only summary for 404 codes
#$http_rc_detail_rep_404 = 20

# vi: shiftwidth=3 tabstop=3 et
```

Enregistrez et quittez le fichier (: wq!) Et désactivez les fichiers de configuration apache par défaut:

    cd /usr/share/logwatch/default.conf/services
    mv http-error.conf http-error.conf.bak && mv http.conf http.conf.bak

Au moins, nous créons un cronjob pour envoyer le résultat de logwatch automatiquement:

    crontab -e 

Collez la ligne suivante:

`@daily /usr/sbin/logwatch --output mail --mailto your@mail.com --format html --detail high --range yesterday > /dev/null 2>&1`

Enregistrez et quittez crontab et vérifiez si logwatch est correctement configuré:

    /usr/sbin/logwatch --output mail --mailto your@mail.com --format html --detail high --range yesterday


Vous devriez recevoir un email de logwatch qui ressemble à ceci:

image

A partir de maintenant, vous recevrez des mails quotidiens contenant votre résumé système.

Profitez de vos données personnelles sur votre Nextcloud-Server sécurisé et sécurisé!  

## N'oubliez pas de sauvegarder votre Nextcloud

Retrouvez plus d'instructions ici: [Sauvegarde et restauration Nextcloud (en)](https://www.c-rieger.de/nextcloud-backup-and-restore/) 

## [Update Nextcloud-Lien HS](/files/html/Update Nextcloud.htm)



