+++
title = 'GitLabYunohostDebianJessieServer'
date = 2018-11-23 00:00:00 +0100
categories = debian
+++
### gitlab 

Installer une application personnalisée **Multi custom webapp** <https://github.com/YunoHost-Apps/multi_webapp_ynh>   
Libellé pour Multi custom webapp : **Gitlab cinay**  
Choisissez un domaine pour votre Webapp : **gitlab.cinay.pw**  
Choisissez un chemin pour votre Webapp : **/**  (il ne sera plus possible d'ajouter quoique ce soit à ce domaine)  
Choisissez l'utilisateur YunoHost associé : **cinay spm**  
Créer une base de données? : **Non** (ne pas cocher la case)  
Est-ce un site public ? : **Oui** (cocher la case)  
Vous ne pourrez pas installer d'autres applications sur map.cinay.pw. Continuer ?  **OK**  

Dossier root web **/var/www/webapp_cinay/gitlab.cinay.pw/**

Installation gitlab avec utilisation du serveur nginx existant sur yunohost  
Télécharger Gitlab Debian Jessie [Gitlab-ce APT/YUM repository for GitLab Community Edition packages](https://packages.gitlab.com/gitlab/gitlab-ce)  

```
#curl -LJO https://packages.gitlab.com/gitlab/gitlab-ce/packages/debian/jessie/gitlab-ce_10.0.0-ce.0_amd64.deb/download
#sudo dpkg -i gitlab-ce_10.0.0-ce.0_amd64.deb  # patienter de longues minutes !!!
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo apt-get install gitlab-ce=10.4.3-ce.0 # patienter de longues minutes !!!
```

Configuration , on n'utilise pas le serveur nginx embarqué dans gitlab-ce  
`sudo nano /etc/gitlab/gitlab.rb`  

```
external_url 'https://gitlab.cinay.pw'

nginx['enable'] = false
web_server['external_users'] = ['www-data']
```

Valider la configuration  
`sudo gitlab-ctl reconfigure`  
Ajout www-data au groupe gitlab-www  
`sudo usermod -aG gitlab-www www-data`  

Modifier les fichiers de configuration nginx  
Ajouter au début du fichier **/etc/nginx/conf.d/gitlab.cinay.pw.conf**  

```
upstream gitlab-workhorse {
  server unix:/var/opt/gitlab/gitlab-workhorse/socket fail_timeout=0;
}
```

Effacer puis recréer le fichier de configuration du site **/etc/nginx/conf.d/gitlab.cinay.pw.d/webapp_gitlab.cinay.pw.conf**  

```
  location / {
    client_max_body_size 0;
    gzip off;

    ## https://github.com/gitlabhq/gitlabhq/issues/694
    ## Some requests take more than 30 seconds.
    proxy_read_timeout      300;
    proxy_connect_timeout   300;
    proxy_redirect          off;

    proxy_http_version 1.1;

    proxy_set_header    Host                $http_host;
    proxy_set_header    X-Real-IP           $remote_addr;
    proxy_set_header    X-Forwarded-Ssl     on;
    proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
    proxy_set_header    X-Forwarded-Proto   $scheme;
    proxy_pass http://gitlab-workhorse;
  }
```

Vérifier  
`sudo nginx -t`  
Relancer nginx  
`sudo systemctl reload nginx`  

A la première connexion au site <https://gitlab.cinay.pw> , il faut renseigné le mot de passe “admin”  
Cliquer sur **Register** et créer un utilisateur  
Full name : cinay spm  
Username : cinay  
Email : cinay@cinay.pw  
Password : xxxxxxxxx  

Créer un groupe **spm** avec accès **Public**  (<https://gitlab.cinay.pw/spm>)   
Créer un projet **New Project** **wikistatic** dans le groupe spm avec accès **Public**  

**Gitlab Instructions en ligne de commande**

Configuration globale de Git

```
git config --global user.name "cinay spm"
git config --global user.email "cinay@cinay.pw"
```

Modifier ou créer **~/.git-credentials** pour un accès auto  
`https://cinay:Mot-de-passe@gitlab.cinay.pw`  
Il faut également ajouter en fin de fichier **~/.gitconfig** les 2 lignes suivantes  

```
[credential]
	helper = store
```

Créer un nouveau dépôt (repository)  

```
git clone http://gitlab.cinay.pw/spm/wikistatic.git
cd wikistatic
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

Dossier existant

```
cd existing_folder
git init
git remote add origin http://gitlab.cinay.pw/spm/wikistatic.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

Dépôt Git existant

Accès au dépôt local  
`cd existing_repo`  
Git sur distant  
`git remote add origin http://gitlab.cinay.pw/spm/wikistatic.git`  
si message **fatal: la distante origin existe déjà.** , exécuter : `git remote rm origin` puis réexécuter l'instruction précédente  
Mise à nivau git distant  
`git push -u origin --all `  
`git push -u origin --tags`  

### Site statique (jekyll)

[Ruby via compilation + serveur statique Jekyll sur debian Jessie ](https://static.ouestline.net/linux/2018/02/07/Ruby-Jekyll-serveur-statique.html)

Installer une application personnalisée **Multi custom webapp** <https://github.com/YunoHost-Apps/multi_webapp_ynh>   
Libellé pour Multi custom webapp : **Site statique**  
Choisissez un domaine pour votre Webapp : **static.cinay.pw**  
Choisissez un chemin pour votre Webapp : **/**  (il ne sera plus possible d'ajouter quoique ce soit à ce domaine)  
Choisissez l'utilisateur YunoHost associé : **cinay spm**  
Créer une base de données? : **Non** (ne pas cocher la case)  
Est-ce un site public ? : **Oui** (cocher la case)  
Vous ne pourrez pas installer d'autres applications sur map.cinay.pw. Continuer ?  **OK**  

Dossier root web **/var/www/webapp_cinay/static.cinay.pw/**

Installation site statique jekyll <https://gitlab.cinay.pw/spm/wikistatic>  
Si erreur installation **rmagick** `sudo apt-get install libmagick++-dev`  

Recréer le fichier de configuration du site **/etc/nginx/conf.d/static.cinay.pw.d/webapp_static.cinay.pw.conf**  

```
location / {
  proxy_pass http://127.0.0.1:4000;
}
```

Vérifier  
`nginx -t`  
Relancer nginx  
`systemctl reload nginx`  

Dans le cas d'une synchronisation des **posts** et **images** avec le support externe de **nextcloud** (Home), il faut modifier les liens sur les dossiers du site statique  

Actuellement :  
**/srv/wikistatic/_posts/** pour les publications au format **AAAA-MM-JJ-titre-de-la-publication.md**  
**/srv/wikistatic/images/** pour les images contenues dans les publications  

Utiliser les dossiers partagés de nextcloud :  
**/home/cinay/static/posts** et **/home/cinay/static/images**  
On arrête le serveur statique jekyll  
`sudo systemctl stop jekyll`  
On efface les dossiers actuels  
`sudo rm -r /srv/wikistatic/{_posts,images}`  
Création des liens avec les nouveaux dossiers  
`sudo ln -s /home/cinay/static/posts /srv/wikistatic/_posts`  
`sudo ln -s /home/cinay/static/images /srv/wikistatic/images`  
Relancer le serveur statique jekyll  
`sudo systemctl start jekyll`  
Le lien <https://static.cinay.pw>  
La mise à jour des applications "jekyll"  

```
sudo -s
cd /srv/wikistatic
bundle update
```

>Nous venons d'installer Jekyll qui génére le site statique et lance son propre serveur, **http://127.0.0.1:4000**  
Pour visualiser le site , on utilise un proxy nginx,  **proxy_pass http://127.0.0.1:4000;** 

**<u>Alternative : Utilisation du générateur jekyll et du serveur nginx (sans proxy)</u>**  

On modifie le  bash **/srv/wikistatic/start_jekyll.sh** pour exécuter uniquement le générateur **jekyll build**

    sudo nano /srv/wikistatic/start_jekyll.sh

```
#!/bin/sh
#lancement jekyll
cd /srv/wikistatic/
#bundle exec jekyll serve
bundle exec jekyll build --watch
```

Jekyll générera automatiquement le site statique dans le dossier **/srv/wikistatic/_site**  
Modification du ficher de configuration **/etc/nginx/conf.d/static.cinay.pw.d/webapp_static.cinay.pw.conf**  

    sudo nano /etc/nginx/conf.d/static.cinay.pw.d/webapp_static.cinay.pw.conf 

```
location / {
	alias /var/www/webapp_cinay/static.cinay.pw/ ;
	if ($scheme = http) {
		rewrite ^ https://$server_name$request_uri? permanent;
	}
	index index.html index.php ;
	try_files $uri $uri/ index.php;
	location ~ [^/]\.php(/|$) {
		fastcgi_split_path_info ^(.+?\.php)(/.*)$;
		fastcgi_pass unix:/var/run/php5-fpm-webapp_static.cinay.pw.sock;
		fastcgi_index index.php;
		include fastcgi_params;
		fastcgi_param REMOTE_USER $remote_user;
		fastcgi_param PATH_INFO $fastcgi_path_info;
		fastcgi_param SCRIPT_FILENAME $request_filename;
	}

	# Include SSOWAT user panel.
	include conf.d/yunohost_panel.conf.inc;
}
```

Dossier root web **/var/www/webapp_cinay/static.cinay.pw/**   , on efface ce répertoire  
`sudo rm -r /var/www/webapp_cinay/static.cinay.pw/`  
Créer un lien sur le dossier **/srv/wikistatic/_site** avec le root web de dev  
`sudo ln -s /srv/wikistatic/_site /var/www/webapp_cinay/static.cinay.pw`  
On recharge **nginx*  
`sudo systemctl reload nginx`  


### Shaarli

En mode su  
`yunohost app install https://github.com/YunoHost-Apps/shaarli_ynh`  

```
Domaines disponibles :
- cinay.pw
Choisissez un domaine pour votre Shaarli (default: cinay.pw) : 
Choisissez un chemin pour votre Shaarli (default: /shaarli) : 
Est-ce un site Shaarli public ? [Yes | No] (default: No) : Yes
Les nouveaux liens sont-ils privés par défaut ? [Yes | No] (default: Yes) : No
Cette instance est-elle privée ? [Yes | No] (default: Yes) : No
Propriétaire de l'instance Shaarli : xeuyak
Définissez le mot de passe de l'utilisateur Shaarli : 
Choissez un titre pour la page Shaarli (default: Shaarli) : 
Attention : « yunohost app checkurl » est déprécié et sera bientôt supprimé
Succès ! La configuration de SSOwat a été générée
Succès ! Installation terminée
```

Restauration base de données  
`cp site/data/datastore.php /var/www/shaarli/data/`  

Configuration  
date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23

Créer le lien firefox **Shaarli , ajouter une marque** pour la sauvegarde des marques  
`javascript:(function(){var%20url%20=%20location.href;var%20title%20=%20document.title%20||%20url;window.open('https://cinay.pw/shaarli/?post='%20+%20encodeURIComponent(url)+'&title='%20+%20encodeURIComponent(title)+'&description='%20+%20encodeURIComponent(document.getSelection())+'&source=bookmarklet','_blank','menubar=no,height=390,width=600,toolbar=no,scrollbars=no,status=no,dialog=1');})();`

Créer le lien firefox **Shaarli liens publiques cinay.pw**  <https://cinay.pw/shaarli/?do=tagcloud>  

javascript:(function(){var%20url%20=%20location.href;var%20title%20=%20document.title%20||%20url;window.open('https://cinay.pw/shaarli/?post='%20+%20encodeURIComponent(url)+'&title='%20+%20encodeURIComponent(title)+'&description='%20+%20encodeURIComponent(document.getSelection())+'&source=bookmarklet','_blank','menubar=no,height=390,width=600,toolbar=no,scrollbars=no,status=no,dialog=1');})();

### Dokuwiki

Installation par admin  
Installer le thème [bootstrap3](https://github.com/LotarProject/dokuwiki-template-bootstrap3/zipball/master) et valider le dans la configuration  
Restauration d'un dokuwiki contenu dans le dossier **dokuwiki**  
`sudo cp -r dokuwiki/* /var/www/dokuwiki/`  
Droits sur le dossier  
`sudo chown dokuwiki.root -R /var/www/dokuwiki/*`  
Suppression(config apache)  
`rm /var/www/dokuwiki/.htaccess.dist`  
Edition configuration **/var/www/dokuwiki/conf/dokuwiki.php** pour remettre les paramètres  
`sudo nano /var/www/dokuwiki/conf/dokuwiki.php`  

```php
...

$conf['start']       = 'start';           //name of start page
$conf['lang']        = 'fr';              //your language

...

/* Authentication Settings */
$conf['useacl']      = 1;                //Use Access Control Lists to restrict access?
$conf['openregister']= 0;
$conf['autopasswd']  = 1;                //autogenerate passwords and email them to user
$conf['authtype']    = 'authldap';      //which authentication backend should be used
$conf['passcrypt']   = 'sha1';           //Used crypt method (smd5,md5,sha1,ssha,crypt,mysql,my411)
$conf['defaultgroup']= 'user';           //Default groups new Users are added to
$conf['superuser']   = 'cinay';    //The admin can be user or @group or comma separated list user1,@group1,user2
$conf['manager']     = 'cinay';    //The manager can be user or @group or comma separated list user1,@group1,user2
$conf['profileconfirm'] = 1;             //Require current password to confirm changes to user profile
$conf['rememberme'] = 1;                 //Enable/disable remember me on login
$conf['disableactions'] = '';            //comma separated list of actions to disable
$conf['auth_security_timeout'] = 900;    //time (seconds) auth data is considered valid, set to 0 to recheck on every page view
$conf['securecookie'] = 1;               //never send HTTPS cookies via HTTP
$conf['remote']      = 0;                //Enable/disable remote interfaces
$conf['remoteuser']  = '!!not set !!';   //user/groups that have access to remote interface (comma separated)

/* LDAP Yunohost config */
$conf['plugin']['authldap']['server']      = 'localhost';
$conf['plugin']['authldap']['port']        = 389;
$conf['plugin']['authldap']['version']    = 3;
$conf['plugin']['authldap']['usertree']    = 'ou=users,dc=yunohost,dc=org';
$conf['plugin']['authldap']['userfilter']  = '(&(uid=%{user})(objectClass=posixAccount))';
...
```

Remplacer favicon et logo **/var/www/dokuwiki/lib/tpl/bootstrap3/images/**  

```
sudo cp images/yannick-green.ico /var/www/dokuwiki/lib/tpl/bootstrap3/images/favicon.ico
sudo cp images/yannick53x64.png /var/www/dokuwiki/lib/tpl/bootstrap3/images/logo.png
sudo cp images/yannick-green.png /var/www/dokuwiki/lib/tpl/bootstrap3/images/apple-touch-icon.png
```

### Diffie-Hellmann nginx

Générer une clé Diffie-Hellmann  
`openssl dhparam -out /etc/ssl/private/dh4096.pem -outform PEM -2 4096`  
Déplacer la clé dans le répertoire  
`sudo mv dh4096.pem /etc/ssl/private/`  
Droits pour root  
`sudo chmod 600 /etc/ssl/private/dh4096.pem`  
`sudo chown root.root /etc/ssl/private/dh4096.pem`  

Modifier les fichiers de configuration des différents domaines activer Diffie-Hellmann  
`    ssl_dhparam /etc/ssl/private/dh4096.pem;`  
 
```
sudo -s
sed -i 's:#ssl_dhparam /etc/ssl/private/dh2048.pem;:ssl_dhparam /etc/ssl/private/dh4096.pem;:g' /etc/nginx/conf.d/cinay.pw.conf
sed -i 's:#ssl_dhparam /etc/ssl/private/dh2048.pem;:ssl_dhparam /etc/ssl/private/dh4096.pem;:g' /etc/nginx/conf.d/gitlab.cinay.pw.conf
sed -i 's:#ssl_dhparam /etc/ssl/private/dh2048.pem;:ssl_dhparam /etc/ssl/private/dh4096.pem;:g' /etc/nginx/conf.d/static.cinay.pw.conf
sed -i 's:#ssl_dhparam /etc/ssl/private/dh2048.pem;:ssl_dhparam /etc/ssl/private/dh4096.pem;:g' /etc/nginx/conf.d/xoyize.xyz.conf
sed -i 's:#ssl_dhparam /etc/ssl/private/dh2048.pem;:ssl_dhparam /etc/ssl/private/dh4096.pem;:g' /etc/nginx/conf.d/map.cinay.pw.conf
sed -i 's:#ssl_dhparam /etc/ssl/private/dh2048.pem;:ssl_dhparam /etc/ssl/private/dh4096.pem;:g' /etc/nginx/conf.d/mulleryan.net.conf
```

Vérifier syntaxe  
`sudo nginx -t`  
Recharger nginx  
`sudo systemctl reload nginx`  

### Tiny Tiny RSS

Installation via interface admin web  
Ouvrir l'application et importer un fichier **opml** (sauvegarde des flux)

### Transmission

Installation via inerface admin web  

Configurer les utilisateurs et les autorisations

Il est recommandé que Transmission s'exécute sous son propre nom d'utilisateur pour des raisons de sécurité.  
Cela crée quelques problèmes avec l'accès aux fichiers et aux dossiers par Transmission ainsi que votre compte (supposons qu'il soit utilisateur).  

Ajoutez l'utilisateur courant au groupe **debian-transmission** et **www-data**:  
`sudo usermod -a -G debian-transmission $USER`  
`sudo usermod -a -G www-data $USER`  

>REMARQUE: lors de l'ajout d'un utilisateur à un nouveau groupe, l'utilisateur doit se déconnecter et se reconnecter pour qu'il prenne effet.  
Un redémarrage l'accomplira également.

### Dévelop - dev

Installer une application personnalisée **Multi custom webapp** <https://github.com/YunoHost-Apps/multi_webapp_ynh>   
Libellé pour Multi custom webapp : **Develop**
Choisissez un domaine pour votre Webapp : **dev.cinay.pw**  
Label : **Develop**  
Choisissez un chemin pour votre Webapp : **/**  (il ne sera plus possible d'ajouter quoique ce soit à ce domaine)  
Choisissez l'utilisateur YunoHost associé : **cinay spm**  
Créer une base de données? : **Non** (ne pas cocher la case)  
Est-ce un site public ? : **Oui** (cocher la case)  
Vous ne pourrez pas installer d'autres applications sur map.cinay.pw. Continuer ?  **OK**  

Dossier root web **/var/www/webapp_cinay/dev.cinay.pw/**  
Créer un lien sur le dossier **/srv/wikistatic/_site** avec le root web de dev  

### Searx

[Searx](https://searx.me/) est un métamoteur, il regroupe les résultats de plusieurs moteurs  de recherche tels que, entre autres, Google, Bing, Yahoo, Yandex, mais aussi Wikipedia, Reddit, et d’autres sites. On peut choisir les sources des résultats de recherche selon nos besoins et/ou envies, et paramétrer à peu près tout ce qu’on veut. C’est 100% open source, et il existe [plusieurs instances](https://github.com/asciimoo/searx/wiki/Searx-instances) à travers le monde.

En mode administrateur , installer application *searx**  


