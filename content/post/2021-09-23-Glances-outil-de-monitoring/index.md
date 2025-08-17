+++
title = 'Glances outil de surveillance en temps réel des performances pour les systèmes d'exploitation basés sur Linux'
date = 2021-09-23 00:00:00 +0100
categories = ['outils']
+++
### Monitoring

[Glances - An eye on your system](https://github.com/nicolargo/glances)  
*Glances est un outil de surveillance multiplateforme qui vise à présenter une grande quantité d'informations de surveillance par le biais d'une interface basée sur curses ou sur le Web. L'information s'adapte dynamiquement en fonction de la taille de l'interface utilisateur.  
Il peut également fonctionner en mode client/serveur. La surveillance à distance peut se faire via un terminal, une interface Web ou une API (XML-RPC et RESTful). Les statistiques peuvent également être exportées vers des fichiers ou des bases de données temps/valeur externes.  
Glances est écrit en Python et utilise des bibliothèques pour récupérer les informations de votre système. Il est basé sur une architecture ouverte où les développeurs peuvent ajouter de nouveaux plugins ou modules d'exportation.*

**Script d'installation automatique de Glances**  
Pour installer les deux dépendances et la dernière version de Glances prête pour la production (aka branche master), il suffit d'entrer la ligne de commande suivante :

    curl -L https://bit.ly/glances | /bin/bash

Si parefeu, ouvrir un port d'écoute, exemple avec UFW 

    sudo ufw allow 55055/tcp  # glances 

**Utilisation**  
Pour le mode autonome 

    glances

Pour le mode serveur Web 

    glances -w

et entrez l'URL http://<ip>:61208 dans votre navigateur Web 

Pour le mode client/serveur 

    glances -s -p 55055    # côté serveur
    glances -c <ip>:55055  # côté client.

Vous pouvez également détecter et afficher tous les serveurs Glances disponibles sur votre réseau ou définis dans le fichier de configuration :

    glances --browser

Vous pouvez également afficher les statistiques brutes sur stdout 

    glances --stdout cpu.user,mem.used,load

```
cpu.user : 30.7
mem.used : 3278204928
load : {'cpucore' : 4, 'min1' : 0.21, 'min5' : 0.4, 'min15' : 0.27}
cpu.user : 3.4
mem.utilisé : 3275251712
load : {'cpucore' : 4, 'min1' : 0.19, 'min5' : 0.39, 'min15' : 0.27}
...
```

ou dans un format CSV grâce à l'option stdout-csv 

    glances --stdout-csv now,cpu.user,mem.used,load

```
now,cpu.user,mem.used,load.cpucore,load.min1,load.min5,load.min15
2018-12-08 22:04:20 CEST,7.3,5948149760,4,1.04,0.99,1.04
2018-12-08 22:04:23 CEST,5.4,5949136896,4,1.04,0.99,1.04
...
```

**Créer un service pour lancer le serveur web glances**

    sudo nano /etc/systemd/system/glancesweb.service

```
[Unit]
Description=outil de surveillance glances

[Service]
Type=simple
ExecStart=/usr/local/bin/glances -w


[Install]
WantedBy=multi-user.target
```

Démarrer et activer le service

    sudo systemctl start glancesweb.service && sudo systemctl enable glancesweb.service

Status

    systemctl status glancesweb.service

```
● glancesweb.service - outil de surveillance glances
   Loaded: loaded (/etc/systemd/system/glancesweb.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2021-09-23 13:15:12 CEST; 9s ago
 Main PID: 3755 (glances)
    Tasks: 1 (limit: 2359)
   Memory: 27.3M
   CGroup: /system.slice/glancesweb.service
           └─3755 /usr/bin/python /usr/local/bin/glances -w

sept. 23 13:15:12 debsrv systemd[1]: Started outil de surveillance glances.
```

**Proxy Nginx**  

Créer un proxy pour accès internet, exmple avec domaine **glances.rnmkcy.eu** et certificats  
Fichier de configuration nginx `glances.rnmkcy.eu.conf`

```
server {
    listen 80;
    listen [::]:80;
    server_name glances.rnmkcy.eu;
    return 301 https://$host$request_uri;
}
server {
    #listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name glances.rnmkcy.eu;
    ssl_certificate /etc/ssl/private/rnmkcy.eu-fullchain.pem;
    ssl_certificate_key /etc/ssl/private/rnmkcy.eu-key.pem;

    # TLS 1.3 only
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;
 
    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;
 
    location / {
	proxy_pass http://127.0.0.1:61208;
    }
    
    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
 
    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /etc/ssl/private/rnmkcy.eu-fullchain.pem;
 
    # replace with the IP address of your resolver
    resolver 1.1.1.1;

}
```