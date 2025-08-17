+++
title = 'subsonic-yunohost'
date = 2019-09-06 00:00:00 +0100
categories = yunohost
+++
### Subsonic (audio.cinay.xyz)

![](subsonic-logo.png){:width="80"}  
*Application dédiée musique, serveur java*

Créer domaine et certificat Let's Encrypt  
Installation de l'application "Multi webapp for YunoHost" sur le domaine **audio.cinay.xyz**  

    yunohost app install https://github.com/YunoHost-Apps/multi_webapp_ynh

```
WARNING! Installing 3rd party applications may compromise the integrity and security of your system. You should probably NOT install it unless you know what you are doing. Are you willing to take that risk? [Y/N] : Y
Available domains:
- cinay.xyz
- map.cinay.xyz
- blog.cinay.xyz
- audio.cinay.xyz
- liens.cinay.xyz
- static.cinay.xyz
Choose a domain for your Webapp (default: cinay.xyz): audio.cinay.xyz
Choose a path for your Webapp (default: /site): /
Available users:
- yannick
Choose the YunoHost user: yannick
Create a database? [yes | no] (default: no): 
Is it a public website ? [yes | no] (default: no): yes
Info: [....................] > Retrieve arguments from the manifest
Info: [#...................] > Check if the app can be installed
Info: [##..................] > Store settings from manifest
Info: [#####...............] > Setup SSOwat
Info: [######..............] > Create final path
Info: The directory /var/www/webapp_yannick already exist, do not recreate it.
Info: [#######.............] > Create a dedicated user
Info: [##########..........] > Configure php-fpm
Warning: Reload the service php7.0-fpm
Info: [###########.........] > Configure nginx
Warning: Reload the service nginx
Info: [############........] > Reload nginx
Info: [####################] > Installation completed
Success! The SSOwat configuration has been generated
Success! Installation complete
```

Installer java

	sudo apt install openjdk-8-jre

Installer subsonic , [télécharger la version en cours](http://www.subsonic.org/pages/download.jsp)

    wget -O subsonic-6.1.5.deb https://s3-eu-west-1.amazonaws.com/subsonic-public/download/subsonic-6.1.5.deb

Installer

	sudo dpkg -i subsonic-6.1.5.deb

Modifier le paramètrage **/etc/default/subsonic**

```
SUBSONIC_ARGS="--port=8090 --max-memory=200"
SUBSONIC_USER=debadm
```

Relancer subsonic

	sudo systemctl restart subsonic

Créer le fichier de configuration nginx

	nano /etc/nginx/conf.d/audio.cinay.xyz.d/webapp_audio.cinay.xyz_.conf

```
location / {
	#alias /var/www/webapp_yannick/audio.cinay.xyz_/ ;
	if ($scheme = http) {
		rewrite ^ https://$server_name$request_uri? permanent;
	}

 	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_redirect off;
 	proxy_set_header Host $http_host;
 	proxy_pass http://localhost:8090;

	# Include SSOWAT user panel.
	include conf.d/yunohost_panel.conf.inc;
}
```

Vérifier et recharger nginx

    sudo nginx -t
	sudo systemctl reload nginx

Accès <https://audio.xoyize.xyz>

>NE PAS OUBLIER : **admin** Autoriser l'accès à ces dossiers de médias **Music**