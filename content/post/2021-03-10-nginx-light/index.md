+++
title = 'Nginx-light PHP 8'
date = 2021-03-10 00:00:00 +0100
categories = nginx
+++
## nginx-light

* [What is the difference between the core, full, extras and light packages for nginx?](https://askubuntu.com/questions/553937/what-is-the-difference-between-the-core-full-extras-and-light-packages-for-ngi)
* **nginx-light** is the lightest flavor of nginx available. It is in the Universe repository and you have to have that enabled to use it. It does not enable a large amount of the modules available in -core or -full. It also contains third-party modules. The modules available in it are as follows:
    * STANDARD HTTP MODULES: Core, Access, Auth Basic, Auto Index, Charset, Empty GIF, FastCGI, Gzip, Headers, Index, Log, Map, Proxy, Rewrite, Upstream.
    * OPTIONAL HTTP MODULES: Auth Request, Debug, Gzip Precompression, IPv6, Real Ip, SSL, Stub Status.
    * THIRD PARTY MODULES: Echo.

Supprimer toutes les installions existantes de nginx  
`sudo apt remove --auto-remove nginx nginx-light nginx-common`  
`sudo apt purge nginx nginx-light nginx-common`  
`sudo apt remove --purge $(dpkg -l | grep "^rc" | awk '{print $2}')`  
{: .prompt-warning }

### Nginx-light/Debian

    sudo apt install nginx-light

modifier le fichier `/etc/nginx/nginx.conf`  
On autorise tls1.2 et tls1.3 uniquement et ciphers off 

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
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

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1.2 TLSv1.3;
	ssl_prefer_server_ciphers off;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```

### Certificats SSL, Diffie Hellmann

[Serveur , installer et renouveler les certificats SSL Let's encrypt via Acme](/posts/Acme-Certficats-Serveurs/)   
Diffie-Hellman (facultatif) , générer le fichier dh2048.pem (en mode su):`openssl dhparam -out /etc/ssl/private/dh2048.pem -outform PEM -2 2048 && chmod 600 /etc/ssl/private/dh2048.pem`   

### Virtualhost default

Modifier le fichier `/etc/nginx/sites-enabled/default` , remplacer **exemple.fr** par le nom de domaine

```
server {
    listen 80;
    listen [::]:80;
    server_name exemple.fr;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name exemple.fr;
    ssl_certificate /etc/ssl/private/exemple.fr-fullchain.pem;
    ssl_certificate_key /etc/ssl/private/exemple.fr-key.pem;

    root /var/www/;
    index index/ index.htm index.nginx-debian/;

    # TLS 1.3 only
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;
 
    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;
 
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
 
    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /etc/ssl/private/exemple.fr-fullchain.pem;
 
    # replace with the IP address of your resolver
    resolver 127.0.0.1;

}
```

Vérifier

    sudo nginx -t

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Recharger nginx

    sudo systemctl reload nginx

### Autres configurations SSL

**Moderne (défaut)**

```
# generated 2020-12-27, Mozilla Guideline v5.6, nginx 1.17.7, OpenSSL 1.1.1d, modern configuration
# https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=modern&openssl=1.1.1d&guideline=5.6
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    ssl_certificate /path/to/signed_cert_plus_intermediates;
    ssl_certificate_key /path/to/private_key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    # modern configuration
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;

    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;

    # replace with the IP address of your resolver
    resolver 127.0.0.1;
}
```

**Intermediaire**

```
# generated 2020-12-27, Mozilla Guideline v5.6, nginx 1.17.7, OpenSSL 1.1.1d, intermediate configuration
# https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=intermediate&openssl=1.1.1d&guideline=5.6
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    ssl_certificate /path/to/signed_cert_plus_intermediates;
    ssl_certificate_key /path/to/private_key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    # curl https://ssl-config.mozilla.org/ffdhe2048.txt > /path/to/dhparam
    ssl_dhparam /path/to/dhparam;

    # intermediate configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;

    # replace with the IP address of your resolver
    resolver 127.0.0.1;
}
```

**Ancienne**

```
# generated 2020-12-27, Mozilla Guideline v5.6, nginx 1.17.7, OpenSSL 1.1.1d, old configuration
# https://ssl-config.mozilla.org/#server=nginx&version=1.17.7&config=old&openssl=1.1.1d&guideline=5.6
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    ssl_certificate /path/to/signed_cert_plus_intermediates;
    ssl_certificate_key /path/to/private_key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    # openssl dhparam 1024 > /path/to/dhparam
    ssl_dhparam /path/to/dhparam;

    # old configuration
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA256:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA;
    ssl_prefer_server_ciphers on;

    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;

    # replace with the IP address of your resolver
    resolver 127.0.0.1;
}
```

### Image sur la page d'accueil (facultatif)  

Déposer une image dans le dossier `/var/www/`  
Créer un fichier `/var/www//index/`  

```hmtl
<!DOCTYPE/>
/>
<head>
 <meta charset="UTF-8"> 
 <title>CX11</title>
<style type="text/css" media="screen" >
html { 
  margin:0;
  padding:0;
  background: url(wallpaper.jpg) no-repeat center fixed; 
  -webkit-background-size: cover; /* pour anciens Chrome et Safari */
  background-size: cover; /* version standardisée */
}
body { color: white; }
a:link {
  color: grey;
  background-color: transparent;
  text-decoration: none;
}
a:hover {
  color: red;
  background-color: transparent;
  text-decoration: underline;
}

</style>

</head>
<body>

<h1>Serveur exemple.fr</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>

</body>
</>
```

Lien https://exemple.fr

## Debian - PHP 8 

*Les paquets PHP8 ne sont pas disponibles dans les serveurs miroirs officiels de Debian 10 par défaut.*

### Dépôt sury.org

Mettre à jour le système :

    sudo apt update 
    sudo apt upgrade

Ajout du dépôt sury.org

    sudo -s

Pour installer la version de 8 de php,  ajouter le dépôt sury.

    apt install -y lsb-release apt-transport-https ca-certificates wget
    wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
    echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" |tee /etc/apt/sources.list.d/php.list

### Installer php8.0 

Mise à jour des dépôts :

    apt update

Installation de php8.0 et/ou php8.0-fpm  
paquet php8.0 :

    apt install php8.0 -y

paquet php8.0-fpm :

    apt install php8.0-fpm -y

### Installer php8.0 + extensions

Installer php8 et les extensions pour php 8.0 :

    apt install -y php8.0 php8.0-{cli,common,fpm,pdo,mysql,zip,mbstring,curl,xml,bcmath,imagick} -y

Vérification de la version de php :

    php -v

```
PHP 8.0.3 (cli) (built: Mar  5 2021 08:38:30) ( NTS )
Copyright (c) The PHP Group
Zend Engine v4.0.3, Copyright (c) Zend Technologies
    with Zend OPcache v8.0.3, Copyright (c), by Zend Technologies
```

### Activation compilateur JIT PHP 8.0

Mais nous devons encore activer notre compilateur JIT pour notre PHP 8.0, par défaut il est activé, mais  la taille du tampon est fixée à zéro.  
éditer la configuration de l'opcache.

    sudo nano /etc/php/8.0/cli/conf.d/10-opcache.ini

Puis ajouter ces deux lignes de codes `opcache.` en bas du fichier. Vous pouvez ajuster la taille du tampon JIT en fonction de votre mémoire RAM disponible. Vous pouvez également utiliser la notation suivante de la taille des fichiers.

*    M - Mégaoctet (ex. 512M)
*    G - Gigaoctet (ex. 2G)

```
; configuration for php opcache module
; priority=10
zend_extension=opcache.so

opcache.enable_cli=1
opcache.jit_buffer_size=512M
```

Redémarrer fpm PHP 8.0 pour refléter les changements.

    sudo service php8.0-fpm restart

Vérifier si la configuration est correctement définie, vérifiez les informations PHP.

    php -i | grep 'opcache\.jit'

Les valeurs doivent être les suivantes (Assurez-vous que opcache.jit est égal à tracing => tracing) :

```
opcache.jit => tracing => tracing
opcache.jit_bisect_limit => 0 => 0
opcache.jit_blacklist_root_trace => 16 => 16
opcache.jit_blacklist_side_trace => 8 => 8
opcache.jit_buffer_size => 512M => 512M
opcache.jit_debug => 0 => 0
opcache.jit_hot_func => 127 => 127
opcache.jit_hot_loop => 64 => 64
opcache.jit_hot_return => 8 => 8
opcache.jit_hot_side_exit => 8 => 8
opcache.jit_max_exit_counters => 8192 => 8192
opcache.jit_max_loop_unrolls => 8 => 8
opcache.jit_max_polymorphic_calls => 2 => 2
opcache.jit_max_recursive_calls => 2 => 2
opcache.jit_max_recursive_returns => 2 => 2
opcache.jit_max_root_traces => 1024 => 1024
opcache.jit_max_side_traces => 128 => 128
opcache.jit_prof_threshold => 0.005 => 0.005

```

### PHP Nginx

Vérifier le status fpm

    systemctl status php8.0-fpm

Modifier le fichier de configuration ,ex: `/etc/nginx/sites-enabled/default`  

```
server {
 
    # . . . other configuration
 
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.0-fpm.sock;
    }
}
```

Redémarrer le serveur nginx

    sudo systemctl restart nginx

Pour test, créer un fichier `phpinfo.php`

```php
<?phpinfo();?>
```