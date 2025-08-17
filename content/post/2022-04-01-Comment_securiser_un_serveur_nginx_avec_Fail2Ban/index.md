+++
title = 'Comment sécuriser un serveur nginx avec Fail2Ban'
date = 2022-04-01 00:00:00 +0100
categories = ['nginx']
+++
![](nginx-logo.png) ![](fail2ban.png)  

* [How to Secure an nginx Server with Fail2Ban](https://snippets.aktagon.com/snippets/554-How-to-Secure-an-nginx-Server-with-Fail2Ban)
* [Fail2ban Config with Nginx and SSH ](https://gist.github.com/JulienBlancher/48852f9d0b0ef7fd64c3)
* [THow to secure Nginx with Fail2ban from botnet attack](https://ep.gnt.md/index.php/how-to-secure-nginx-with-fail2ban-from-botnet-attack/)
* [Debian 10 Buster : sécuriser votre serveur avec Fail2ban](https://www.geek17.com/fr/content/debian-10-buster-securiser-votre-serveur-avec-fail2ban-111)
* [Secure Your Linux Server With Fail2Ban ](https://linuxhandbook.com/fail2ban-basic/)

## Prérequis

Serveur nginx installé  

Installer fail2ban

    sudo apt update
    sudo apt install fail2ban

Vérifier son fonctionnement

    sudo systemctl status fail2ban

```
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2022-04-01 13:36:42 CEST; 23s ago
       Docs: man:fail2ban(1)
    Process: 4000 ExecStartPre=/bin/mkdir -p /run/fail2ban (code=exited, status=0/SUCCESS)
   Main PID: 4004 (fail2ban-server)
      Tasks: 5 (limit: 9370)
     Memory: 18.4M
        CPU: 166ms
     CGroup: /system.slice/fail2ban.service
             └─4004 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

avril 01 13:36:42 think systemd[1]: Starting Fail2Ban Service...
avril 01 13:36:42 think systemd[1]: Started Fail2Ban Service.
avril 01 13:36:42 think fail2ban-server[4004]: Server ready
```

## Exigences en matière de sécurité

*    Bloquer toute personne essayant d'exécuter des scripts (.pl, .cgi, .exe, etc.)
*    Bloquer toute personne essayant d'utiliser le serveur comme un proxy.
*    Bloquer toute personne qui ne s'authentifie pas à l'aide de l'authentification de base de nginx.
*    Bloquer toute personne qui ne s'authentifie pas en utilisant la page de connexion de notre application.
*    Bloquer les mauvais bots
*    Limiter le nombre de connexions par session

Nous voulons une solution légère et facile à utiliser et nous pouvons répondre à toutes ces exigences avec fail2ban et nginx.

### Configuration de Fail2ban

Après installation, tous les fichiers de configurations seront logés dans le répertoire /etc/fail2ban
Passons à la configuration proprement dite…  
Le fichier de configuration principal est le jail.conf mais nous n’allons pas l’utiliser directement car ce fichier est souvent altéré après les mises à niveau. Pour cela nous allons faire une copie de ce fichier et le nommer jail.local avec la commande ci-après: cp jail.conf jail.local (en étant dans le répertoire /etc/fail2ban)  

    cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

Nous allons à présent définir nos options dans le fichier jail.local  
Les options à définir sont en dessous de la section [DEFAULT] (la section qui vient après [INCLUDES] )

*    ignoreip = 127.0.0.1/8 ::1 192.168.0.0/24
*    bantime = 600
*    findtime = 300
*    maxretry = 10

**ignoreip**: définit les adresses ip ou plage d’adresse à ignorer ou tolérer lors du fonctionnement de fail2ban   
**bantime**: définit le temps durant lequel une adresse ip doit être bloqué   
**findtime**: c’est le laps de temps pendant lequel on considère les occurences (au delà du findtime, on repart à 0)  
**maxretry**: c’est le nombre de tentatives infructueuses que peut faire une adresse ip  

En résumé, en langage humain, nos options définies peuvent être traduite de la sorte: Toute adresse ip faisant 10 requêtes infructueuses en 5 minutes (300 sec) sera bannie pendant 10 minutes (600 sec) sauf l’adresse de loopback (127.0.0.1 ou ::1) et les adresses du réseau 192.168.0.0/24.

Ouvrir le fichier en édition

    nano /etc/fail2ban/jail.local

Modifier les lignes suivantes

```
# "bantime" is the number of seconds that a host is banned.
bantime  = 600

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 300

# "maxretry" is the number of failures before a host get banned.
maxretry = 10
```

Et ajouter les lignes suivantes en fin de fichier

```
[ssh]
enabled  = true
port     = 55145
filter   = sshd
logpath  = /var/log/auth.log

[ssh-ddos]
enabled  = true
port     = 55145
filter   = sshd-ddos
logpath  = /var/log/auth.log

#
# HTTP servers
#

[nginx-auth]
enabled = true
filter = nginx-auth
action = iptables-multiport[name=NoAuthFailures, port="http,https"]
logpath = /var/log/nginx/*error*.log

[nginx-login]
enabled = false
filter = nginx-login
action = iptables-multiport[name=NoLoginFailures, port="http,https"]
logpath = /var/log/nginx/*access*.log
 
[nginx-badbots]
enabled  = true
filter = apache-badbots
action = iptables-multiport[name=BadBots, port="http,https"]
logpath = /var/log/nginx/*access*.log
maxretry = 1
 
[nginx-proxy]
enabled = true
action = iptables-multiport[name=NoProxy, port="http,https"]
filter = nginx-proxy
logpath = /var/log/nginx/*access*.log
maxretry = 0

[nginx-dos]
enabled  = true
port     = http
filter   = nginx-dos
logpath  = /var/log/nginx/*access*.log
findtime = 120
maxretry = 200
```

### Configuration des filtres

Les fichiers de configuration de filtre sont stockés dans `/etc/fail2ban/filter.d/` 

```
cat > /etc/fail2ban/filter.d/nginx-auth.conf << EOF
#
# Auth filter /etc/fail2ban/filter.d/nginx-auth.conf:
#
# Blocks IPs that makes too much accesses to the server
#
[Definition]

failregex = ^<HOST> -.*"(GET|POST).*HTTP.*"

ignoreregex =
EOF

cat > /etc/fail2ban/filter.d/nginx-dos.conf << EOF
#
# Ddos filter /etc/fail2ban/filter.d/nginx-dos.conf:
#
# Block IPs trying to ddos the server.
#
#
[Definition]

failregex = ^<HOST> -.*"(GET|POST).*HTTP.*"

ignoreregex =
EOF

cat > /etc/fail2ban/filter.d/nginx-login.conf << EOF
#
# Login filter /etc/fail2ban/filter.d/nginx-login.conf:
#
# Blocks IPs that fail to authenticate using web application's log in page
#
# Scan access log for HTTP 200 + POST /sessions => failed log in
#
[Definition]

failregex = ^<HOST> -.*POST /wp-login.php.* HTTP/1\.." 200

ignoreregex =
EOF

cat > /etc/fail2ban/filter.d/nginx-noscript.conf << EOF
# 
# Noscript filter /etc/fail2ban/filter.d/nginx-noscript.conf:
#
# Block IPs trying to execute scripts such as .php, .pl, .exe and other funny scripts.
#
# Matches e.g.
# 192.168.1.1 - - "GET /something.php
#
[Definition]

failregex = ^<HOST> -.*GET.*(\.php|\.asp|\.exe|\.pl|\.cgi|\scgi)

ignoreregex =
EOF

cat > /etc/fail2ban/filter.d/nginx-proxy.conf << EOF
#
# Proxy filter /etc/fail2ban/filter.d/nginx-proxy.conf:
#
# Block IPs trying to use server as proxy.
#
# Matches e.g.
# 192.168.1.1 - - "GET http://www.something.com/
#
[Definition]

failregex = ^<HOST> -.*GET http.*

ignoreregex =
EOF

cat > /etc/fail2ban/filter.d/sshd-ddos.conf << EOF
[Definition]

# Option:  failregex
# Notes.:  regex to match the password failures messages in the logfile. The
#          host must be matched by a group named "host". The tag "<HOST>" can
#          be used for standard IP/hostname matching and is only an alias for
#          (?:::f{4,6}:)?(?P<host>[\w\-.^_]+)
# Values:  TEXT
#
failregex = sshd(?:\[\d+\])?: Did not receive identification string from <HOST>$

# Option:  ignoreregex
# Notes.:  regex to ignore. If this regex matches, the line is ignored.
# Values:  TEXT
#
ignoreregex = 
EOF
```

Après les modifications, relancer fail2ban

    systemctl restart fail2ban

Tester les règles fail2ban

    fail2ban-client -d

```
[...]
['start', 'sshd']
['start', 'ssh']
['start', 'ssh-ddos']
['start', 'nginx-auth']
['start', 'nginx-badbots']
['start', 'nginx-proxy']
['start', 'nginx-dos']
```

Statut fail2ban

    systemctl status fail2ban

```
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2022-04-01 16:32:09 CEST; 1h 32min ago
       Docs: man:fail2ban(1)
    Process: 1006 ExecStartPre=/bin/mkdir -p /run/fail2ban (code=exited, status=0/SUCCESS)
   Main PID: 1013 (fail2ban-server)
      Tasks: 9 (limit: 9370)
     Memory: 25.0M
        CPU: 7.686s
     CGroup: /system.slice/fail2ban.service
             └─1013 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

avril 01 16:32:09 think fail2ban-server[1013]: 2022-04-01 16:32:09,492 fail2ban.configreader   [1013]: WARNING >
avril 01 16:32:09 think fail2ban-server[1013]: 2022-04-01 16:32:09,493 fail2ban.configreader   [1013]: WARNING >
avril 01 16:32:09 think fail2ban-server[1013]: 2022-04-01 16:32:09,494 fail2ban.configreader   [1013]: WARNING >
avril 01 16:32:09 think fail2ban-server[1013]: 2022-04-01 16:32:09,494 fail2ban.configreader   [1013]: WARNING >
avril 01 16:32:09 think fail2ban-server[1013]: 2022-04-01 16:32:09,494 fail2ban.jailreader     [1013]: WARNING >
avril 01 16:32:09 think fail2ban-server[1013]: 2022-04-01 16:32:09,494 fail2ban.configreader   [1013]: WARNING >
avril 01 16:32:09 think fail2ban-server[1013]: 2022-04-01 16:32:09,494 fail2ban.configreader   [1013]: WARNING >
avril 01 16:32:09 think fail2ban-server[1013]: 2022-04-01 16:32:09,494 fail2ban.configreader   [1013]: WARNING >
avril 01 16:32:09 think fail2ban-server[1013]: 2022-04-01 16:32:09,494 fail2ban.jailreader     [1013]: WARNING >
avril 01 16:32:09 think fail2ban-server[1013]: Server ready
```

Aperçu du log : /var/log/fail2ban.log

```
2022-04-01 18:07:23,389 fail2ban.server         [1753]: INFO    Starting Fail2ban v0.11.2
2022-04-01 18:07:23,389 fail2ban.observer       [1753]: INFO    Observer start...
2022-04-01 18:07:23,395 fail2ban.database       [1753]: INFO    Connected to fail2ban persistent database '/var/lib/fail2ban/fail2ban.sqlite3'
2022-04-01 18:07:23,395 fail2ban.jail           [1753]: INFO    Creating new jail 'sshd'
2022-04-01 18:07:23,402 fail2ban.jail           [1753]: INFO    Jail 'sshd' uses pyinotify {}
2022-04-01 18:07:23,405 fail2ban.jail           [1753]: INFO    Initiated 'pyinotify' backend
2022-04-01 18:07:23,407 fail2ban.filter         [1753]: INFO      maxLines: 1
2022-04-01 18:07:23,424 fail2ban.filter         [1753]: INFO      maxRetry: 10
2022-04-01 18:07:23,424 fail2ban.filter         [1753]: INFO      findtime: 300
2022-04-01 18:07:23,424 fail2ban.actions        [1753]: INFO      banTime: 600
2022-04-01 18:07:23,424 fail2ban.filter         [1753]: INFO      encoding: UTF-8
2022-04-01 18:07:23,429 fail2ban.filter         [1753]: INFO    Added logfile: '/var/log/auth.log' (pos = 203485, hash = c86a8eadbc83bb0d37ead2f0ca726dffcfac9021)
2022-04-01 18:07:23,430 fail2ban.jail           [1753]: INFO    Creating new jail 'ssh'
2022-04-01 18:07:23,430 fail2ban.jail           [1753]: INFO    Jail 'ssh' uses pyinotify {}
2022-04-01 18:07:23,432 fail2ban.jail           [1753]: INFO    Initiated 'pyinotify' backend
2022-04-01 18:07:23,432 fail2ban.filter         [1753]: INFO      maxLines: 1
2022-04-01 18:07:23,433 fail2ban.filter         [1753]: INFO      maxRetry: 10
2022-04-01 18:07:23,433 fail2ban.filter         [1753]: INFO      findtime: 300
2022-04-01 18:07:23,433 fail2ban.actions        [1753]: INFO      banTime: 600
2022-04-01 18:07:23,433 fail2ban.filter         [1753]: INFO      encoding: UTF-8
2022-04-01 18:07:23,433 fail2ban.filter         [1753]: INFO    Added logfile: '/var/log/auth.log' (pos = 0, hash = c86a8eadbc83bb0d37ead2f0ca726dffcfac9021)
2022-04-01 18:07:23,434 fail2ban.jail           [1753]: INFO    Creating new jail 'ssh-ddos'
2022-04-01 18:07:23,434 fail2ban.jail           [1753]: INFO    Jail 'ssh-ddos' uses pyinotify {}
2022-04-01 18:07:23,436 fail2ban.jail           [1753]: INFO    Initiated 'pyinotify' backend
2022-04-01 18:07:23,437 fail2ban.filter         [1753]: INFO      maxRetry: 10
2022-04-01 18:07:23,437 fail2ban.filter         [1753]: INFO      findtime: 300
2022-04-01 18:07:23,437 fail2ban.actions        [1753]: INFO      banTime: 600
2022-04-01 18:07:23,437 fail2ban.filter         [1753]: INFO      encoding: UTF-8
2022-04-01 18:07:23,437 fail2ban.filter         [1753]: INFO    Added logfile: '/var/log/auth.log' (pos = 0, hash = c86a8eadbc83bb0d37ead2f0ca726dffcfac9021)
2022-04-01 18:07:23,438 fail2ban.jail           [1753]: INFO    Creating new jail 'nginx-auth'
2022-04-01 18:07:23,438 fail2ban.jail           [1753]: INFO    Jail 'nginx-auth' uses pyinotify {}
2022-04-01 18:07:23,440 fail2ban.jail           [1753]: INFO    Initiated 'pyinotify' backend
2022-04-01 18:07:23,441 fail2ban.filter         [1753]: INFO      maxRetry: 10
2022-04-01 18:07:23,441 fail2ban.filter         [1753]: INFO      findtime: 300
2022-04-01 18:07:23,441 fail2ban.actions        [1753]: INFO      banTime: 600
2022-04-01 18:07:23,441 fail2ban.filter         [1753]: INFO      encoding: UTF-8
2022-04-01 18:07:23,442 fail2ban.filter         [1753]: INFO    Added logfile: '/var/log/nginx/error.log' (pos = 0, hash = e2b8399106db15b3fc49fcac8b9fe0c2bbd8a83c)
2022-04-01 18:07:23,442 fail2ban.jail           [1753]: INFO    Creating new jail 'nginx-badbots'
2022-04-01 18:07:23,443 fail2ban.jail           [1753]: INFO    Jail 'nginx-badbots' uses pyinotify {}
2022-04-01 18:07:23,445 fail2ban.jail           [1753]: INFO    Initiated 'pyinotify' backend
2022-04-01 18:07:23,450 fail2ban.filter         [1753]: INFO      maxRetry: 1
2022-04-01 18:07:23,450 fail2ban.filter         [1753]: INFO      findtime: 300
2022-04-01 18:07:23,450 fail2ban.actions        [1753]: INFO      banTime: 600
2022-04-01 18:07:23,450 fail2ban.filter         [1753]: INFO      encoding: UTF-8
2022-04-01 18:07:23,450 fail2ban.filter         [1753]: INFO    Added logfile: '/var/log/nginx/access.log' (pos = 59288, hash = 4b448d01a20ff30dda4e7ae67e66cfc071c65c84)
2022-04-01 18:07:23,451 fail2ban.jail           [1753]: INFO    Creating new jail 'nginx-proxy'
2022-04-01 18:07:23,451 fail2ban.jail           [1753]: INFO    Jail 'nginx-proxy' uses pyinotify {}
2022-04-01 18:07:23,453 fail2ban.jail           [1753]: INFO    Initiated 'pyinotify' backend
2022-04-01 18:07:23,454 fail2ban.filter         [1753]: INFO      maxRetry: 0
2022-04-01 18:07:23,454 fail2ban.filter         [1753]: INFO      findtime: 300
2022-04-01 18:07:23,454 fail2ban.actions        [1753]: INFO      banTime: 600
2022-04-01 18:07:23,454 fail2ban.filter         [1753]: INFO      encoding: UTF-8
2022-04-01 18:07:23,454 fail2ban.filter         [1753]: INFO    Added logfile: '/var/log/nginx/access.log' (pos = 59288, hash = 4b448d01a20ff30dda4e7ae67e66cfc071c65c84)
2022-04-01 18:07:23,455 fail2ban.jail           [1753]: INFO    Creating new jail 'nginx-dos'
2022-04-01 18:07:23,455 fail2ban.jail           [1753]: INFO    Jail 'nginx-dos' uses pyinotify {}
2022-04-01 18:07:23,457 fail2ban.jail           [1753]: INFO    Initiated 'pyinotify' backend
2022-04-01 18:07:23,457 fail2ban.filter         [1753]: INFO      maxRetry: 200
2022-04-01 18:07:23,458 fail2ban.filter         [1753]: INFO      findtime: 120
2022-04-01 18:07:23,458 fail2ban.actions        [1753]: INFO      banTime: 600
2022-04-01 18:07:23,458 fail2ban.filter         [1753]: INFO      encoding: UTF-8
2022-04-01 18:07:23,458 fail2ban.filter         [1753]: INFO    Added logfile: '/var/log/nginx/access.log' (pos = 0, hash = 4b448d01a20ff30dda4e7ae67e66cfc071c65c84)
2022-04-01 18:07:23,459 fail2ban.jail           [1753]: INFO    Jail 'sshd' started
2022-04-01 18:07:23,461 fail2ban.jail           [1753]: INFO    Jail 'ssh' started
2022-04-01 18:07:23,463 fail2ban.jail           [1753]: INFO    Jail 'ssh-ddos' started
2022-04-01 18:07:23,464 fail2ban.jail           [1753]: INFO    Jail 'nginx-auth' started
2022-04-01 18:07:23,465 fail2ban.jail           [1753]: INFO    Jail 'nginx-badbots' started
2022-04-01 18:07:23,466 fail2ban.jail           [1753]: INFO    Jail 'nginx-proxy' started
2022-04-01 18:07:23,467 fail2ban.jail           [1753]: INFO    Jail 'nginx-dos' started
```

