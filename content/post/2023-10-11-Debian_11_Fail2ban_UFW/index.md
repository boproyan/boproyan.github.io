+++
title = 'Installer et configurer Fail2ban + UFW sur Debian 11/12'
date = 2023-10-11 00:00:00 +0100
categories = outils
+++
![Fail2ban](fail2ban.png)

*Fail2ban est un logiciel de prévention des intrusions qui protège les serveurs informatiques principalement des attaques par force brute, en interdisant les mauvais agents utilisateurs, en interdisant les scanners d'URL, et bien plus encore. Fail2ban y parvient en lisant les journaux d'accès/erreurs de votre serveur ou de vos applications web. Fail2ban est codé dans le langage de programmation python*


## Conditions préalables

*    Système d'exploitation : Debian 11 Bullseye 
*    Compte utilisateur : Un compte utilisateur avec un accès sudo ou root 
*    Paquets requis : wget

Mettez à jour votre système d'exploitation Debian 

    sudo apt update && sudo apt upgrade -y

Le tutoriel utilise la commande sudo et suppose que vous avez le statut sudo.

Pour vérifier le statut sudo sur votre compte : `sudo whoami`

## Installer Fail2ban

Par défaut, Fail2ban est inclus dans le dépôt Bullseye de Debian 11. 

    sudo apt install fail2ban

Par défaut, fail2ban après l'installation doit être lancé et activé.

    sudo systemctl status fail2ban

```
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/lib/systemd/system/fail2ban.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-09-27 10:47:45 CEST; 1min 49s ago
       Docs: man:fail2ban(1)
    Process: 46176 ExecStartPre=/bin/mkdir -p /run/fail2ban (code=exited, status=0/SUCCESS)
   Main PID: 46177 (fail2ban-server)
      Tasks: 5 (limit: 4698)
     Memory: 15.9M
        CPU: 198ms
     CGroup: /system.slice/fail2ban.service
             └─46177 /usr/bin/python3 /usr/bin/fail2ban-server -xf start

sept. 27 10:47:45 ouestyan.fr systemd[1]: Starting Fail2Ban Service...
sept. 27 10:47:45 ouestyan.fr systemd[1]: Started Fail2Ban Service.
sept. 27 10:47:45 ouestyan.fr fail2ban-server[46177]: Server ready
```

Si votre service fail2ban n'est pas activé, exécutez les commandes suivantes pour le démarrer et, si vous le souhaitez, l'activer par défaut au démarrage du système 

    sudo systemctl start fail2ban
    sudo systemctl enable fail2ban

Vérifiez la version de fail2ban 

    fail2ban-client --version

Sortie : **Fail2Ban v0.11.2**

## Configurer Fail2ban

Après avoir terminé l'installation, nous devons maintenant procéder à l'installation et à la configuration de base.  
Fail2ban est livré avec deux fichiers de configuration qui sont situés dans **/etc/fail2ban/jail.conf** et The default Fail2ban **/etc/fail2ban/jail.d/defaults-debian.conf**. <u>Ne modifiez pas ces fichiers</u>.  
`Les fichiers de configuration originaux seront remplacés dans toute mise à jour de Fail2ban`{: .prompt-warning }

Création des copies se terminant par **.local** au lieu de **.conf** car Fail2ban lira toujours les fichiers .local en premier avant de charger .conf s'il n'en trouve pas.

    sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

Ouvrir le fichier de configuration pour le modifier

    sudo nano /etc/fail2ban/jail.local

On va passer en revue certains paramètres que vous pouvez utiliser ou modifier à votre guise. Notez que la plupart des paramètres sont commentés, le tutoriel décommentera les lignes en question ou modifiera les lignes existantes dans les paramètres de l'exemple.  

### Debian 12 - Configuration

Configuration `/etc/fail2ban/jail.local`

```
[DEFAULT]
# Debian 12 has no log files, just journalctl
backend = systemd
logtarget = SYSTEMD-JOURNAL
bantime = 720m # How long to block an abusive IP
findtime = 120m # Time period to check the connections
maxretry = 3 # Within the above time period, block the abusive IP if the number of the abusive IP connections reaches the maxretry
banaction = ufw
banaction_allports = ufw
destemail = example@example.com
sender = example@example.com
ignoreip = 127.0.0.1/8 ::1 192.168.0.0/24 # Ignore these IP, Hosts, IP ranges during operation

[sshd]
# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
enabled = true
port    = 55215
logpath = %(sshd_log)s
backend = %(sshd_backend)s
bantime = 60h
maxretry = 3
```

### Augmenter le temps d'interdiction

Le premier paramètre que vous rencontrerez est l'incrément de temps d'interdiction. Vous devriez l'activer chaque fois que l'attaquant revient. Il augmentera le temps de bannissement, évitant à votre système de bannir constamment la même IP si vos durées de bannissement sont mineures ; par exemple, 1 heure, vous voudriez qu'elle soit plus longue si l'attaquant revient x5 fois.

Vous devez également définir un multiplicateur ou un facteur pour que la logique d'augmentation des bannissements fonctionne. Vous pouvez choisir n'importe lequel de ces facteurs ; cependant, dans notre guide, nous préférons les multiplicateurs, comme le souligne l'exemple ci-dessous, car vous pouvez définir des augmentations de temps d'interdiction personnalisées à votre convenance. Vous trouverez plus d'explications sur les mathématiques dans le set-up.

```
# "bantime.multipliers" used to calculate next value of ban time instead of formula, coresponding 
# previously ban count and given "bantime.factor" (for multipliers default is 1);
# following example grows ban time by 1, 2, 4, 8, 16 ... and if last ban count greater as multipliers count, 
# always used last multiplier (64 in example), for factor '1' and original ban time 600 - 10.6 hours
#bantime.multipliers = 1 2 4 8 16 32 64
# following example can be used for small initial ban time (bantime=60) - it grows more aggressive at begin,
# for bantime=60 the multipliers are minutes and equal: 1 min, 5 min, 30 min, 1 hour, 5 hour, 12 hour, 1 day, 2 day
#bantime.multipliers = 1 5 30 60 300 720 1440 2880
```

### UFW ou Iptables

```
#banaction = iptables-multiport
#banaction_allports = iptables-allports
banaction = ufw
banaction_allports = ufw
```

### Liste blanche d'IPs dans Fail2ban

Ensuite dans la liste, nous trouvons les options de liste blanche, décommentez ce qui suit et adressez toutes les adresses IP que vous voulez mettre sur la liste blanche.

    ignoreip = 127.0.0.1/8 ::1 192.167.5.5 # (exemple d'adresse IP)

Veillez à insérer un espace ou une virgule entre les adresses IP. Vous pouvez également mettre sur liste blanche des plages d'adresses IP.

```
# "ignoreip" can be a list of IP addresses, CIDR masks or DNS hosts. Fail2ban
# will not ban a host which matches an address in this list. Several addresses
# can be defined using space (and/or comma) separator.
ignoreip = 127.0.0.1/8 ::1 45.145.166.178 2a04:ecc0:8:a8:4567:4989:0:1
```

### Durée d'interdiction par défaut

Par défaut, la durée d'interdiction est de 10 minutes avec une recherche de 10 minutes et 5 tentatives. Une explication de ceci est que la prison Fail2ban avec filtrage bannira votre attaquant pendant 10 minutes après qu'il ait retenté la même attaque en 10 minutes (temps de recherche) x 5 fois (tentatives). Vous pouvez définir certains paramètres de bannissement par défaut ici.

Cependant, lorsque vous arrivez aux prisons, il est conseillé de définir des temps de bannissement différents car certains bannissements doivent automatiquement être plus longs que d'autres, y compris les tentatives qui doivent être plus ou moins longues.

```
# "bantime" is the number of seconds that a host is banned.
bantime  = 3600

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 3600

# "maxretry" is the number of failures before a host get banned.
maxretry = 6

# "maxmatches" is the number of matches stored in ticket (resolvable via tag <matches> in actions).
maxmatches = %(maxretry)s
```

### Courrier électronique avec Fail2ban

Remarque, par défaut, Fail2ban utilise le MTA sendmail pour les notifications par courriel. Vous pouvez changer cela pour la fonction mail, remplacer `mta = sendmail` par `mta = mail`

Vous devez sélectionner l'adresse e-mail qui recevra les notifications. Modifiez la directive destemail avec cette valeur. La directive sendername peut être utilisée pour modifier le champ "Sender" dans les emails de notification

```
destemail = admin@example.com
sendername = Fail2BanAlerts
```

Dans le jargon de fail2ban, une "action" est la procédure suivie lorsqu'un client échoue trop souvent à l'authentification. L'action par défaut (appelée action_) consiste simplement à bannir l'adresse IP du port en question. Cependant, il existe deux autres actions prédéfinies qui peuvent être utilisées si vous avez configuré le courrier électronique.

Vous pouvez utiliser l'**action_mw** pour bannir le client et envoyer une notification par e-mail à votre compte configuré avec un rapport "whois" sur l'adresse incriminée. Vous pouvez également utiliser l'**action_mwl**, qui fait la même chose, mais inclut également les lignes du journal qui ont déclenché l'interdiction 

```
# Choose default action.  To change, just override value of 'action' with the
# interpolation to the chosen action shortcut (e.g.  action_mw, action_mwl, etc) in jail.local
# globally (section [DEFAULT]) or per specific section
action = %(action_mwl)s
```

### Configurer Fail2Ban Nginx

#### Configuration de Fail2Ban pour surveiller les journaux de Nginx

Si vous utilisez lz parefeu UFW , ajouter `banaction = ufw` aux différents "jail"
{: .prompt-warning }

Maintenant que vous avez mis en place certains des paramètres généraux de Fail2Ban, nous pouvons nous concentrer sur l'activation de certains jails spécifiques à Nginx qui surveilleront les journaux de notre serveur Web à la recherche de modèles de comportement spécifiques.

Chaque jail du fichier de configuration est marqué par un en-tête contenant le nom du jail entre crochets (chaque section, à l'exception de la section `[DEFAULT]`, indique la configuration d'un jail spécifique). Par défaut, seule la prison `[ssh]` est activée.

```
[sshd]

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
port    = 55178
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

Pour activer la surveillance des journaux des tentatives de connexion de Nginx, nous allons activer l'environnement jail `[nginx-http-auth]`. Modifiez la directive enabled de cette section pour qu'elle indique "true" 

    sudo nano /etc/fail2ban/jail.local

```
[nginx-http-auth]

enabled  = true
filter   = nginx-http-auth
port     = http,https
logpath  = /var/log/nginx/error.log
```

C'est le seul jail spécifique à Nginx inclus dans le paquet fail2ban d'Ubuntu. Cependant, nous pouvons créer nos propres jails pour ajouter des fonctionnalités supplémentaires. L'inspiration et certains détails d'implémentation de ces jails supplémentaires proviennent d'ici et d'ici.

Nous pouvons créer un jail [nginx-noscript] pour interdire les clients qui recherchent des scripts sur le site Web pour les exécuter et les exploiter. Si vous n'utilisez pas PHP ou tout autre langage en conjonction avec votre serveur web, vous pouvez ajouter cette prison pour interdire ceux qui demandent ces types de ressources :

```
[nginx-noscript]

enabled  = true
port     = http,https
filter   = nginx-noscript
logpath  = /var/log/nginx/access.log
maxretry = 6
```

Nous pouvons ajouter une section appelée [nginx-badbots] pour arrêter certains modèles de requête de robots malveillants connus :

```
[nginx-badbots]

enabled  = true
port     = http,https
filter   = nginx-badbots
logpath  = /var/log/nginx/access.log
maxretry = 2
```

Si vous n'utilisez pas Nginx pour fournir un accès au contenu Web dans les répertoires personnels des utilisateurs, vous pouvez interdire les utilisateurs qui demandent ces ressources en ajoutant un jail `[nginx-nohome]` :

```
[nginx-nohome]

enabled  = true
port     = http,https
filter   = nginx-nohome
logpath  = /var/log/nginx/access.log
maxretry = 2
```

Nous devrions interdire les clients qui tentent d'utiliser notre serveur Nginx comme un proxy ouvert. Nous pouvons ajouter un jail `[nginx-noproxy]` pour répondre à ces demandes :

```
[nginx-noproxy]

enabled  = true
port     = http,https
filter   = nginx-noproxy
logpath  = /var/log/nginx/access.log
maxretry = 2
```

Le fichier **jail.local** avec seulement les lignes modifiées

```
bantime.multipliers = 1 5 30 60 300 720 1440 2880
ignoreip = 127.0.0.1/8 ::1 45.145.166.178 2a04:ecc0:8:a8:4567:4989:0:1
ignorecommand =
bantime  = 3600
findtime  = 3600
maxretry = 5

destemail = vps@cinay.eu
sendername = Fail2BanAlerts
mta = mail

#banaction = iptables-multiport
#banaction_allports = iptables-allports
banaction = ufw
banaction_allports = ufw

# ban & send an e-mail with whois report and relevant log lines
# to the destemail.
action = %(action_mwl)s

# The simplest action to take: ban only
#action = %(action_)s



[nginx-noscript]
enabled  = true
port     = http,https
filter   = nginx-noscript
logpath  = /var/log/nginx/access.log
banaction = ufw
maxretry = 6

[nginx-badbots]
enabled  = true
port     = http,https
filter   = nginx-badbots
logpath  = /var/log/nginx/access.log
maxretry = 2

[nginx-nohome]
enabled  = true
port     = http,https
filter   = nginx-nohome
logpath  = /var/log/nginx/access.log
maxretry = 2

[nginx-noproxy]
enabled  = true
port     = http,https
filter   = nginx-noproxy
logpath  = /var/log/nginx/access.log
maxretry = 2
```

#### Ajout des filtres pour les prisons Nginx supplémentaires

Nous avons mis à jour le fichier **/etc/fail2ban/jail.local** avec quelques spécifications de jail supplémentaires pour correspondre et bannir un plus grand nombre de mauvais comportements. Nous devons créer les <u>fichiers de filtre</u> pour les prisons que nous avons créées. Ces fichiers de filtre spécifieront les modèles à rechercher dans les journaux de Nginx.

Le répertoire des filtres : `/etc/fail2ban/filter.d`

Nous voulons en fait commencer par ajuster le filtre d'authentification Nginx fourni au préalable pour qu'il corresponde à un modèle supplémentaire de journal d'échec de connexion.   

    sudo -s

Nous pouvons copier le fichier apache-badbots.conf pour l'utiliser avec Nginx. Nous pouvons utiliser ce fichier tel quel, mais nous allons le copier sous un nouveau nom pour plus de clarté. Cela correspond à la façon dont nous avons référencé le filtre dans la configuration de la prison :

    cp /etc/fail2ban/filter.d/apache-badbots.conf /etc/fail2ban/filter.d/nginx-badbots.conf

Ensuite, nous allons créer un filtre pour notre jail `[nginx-noscript]` 

Collez la définition suivante à l'intérieur. N'hésitez pas à ajuster les suffixes de script pour supprimer les fichiers de langue que votre serveur utilise légitimement ou pour ajouter des suffixes supplémentaires :

```bash
cat > /etc/fail2ban/filter.d/nginx-noscript.conf << EOF
[Definition]
failregex = ^<HOST> -.*GET.*(\.php|\.asp|\.exe|\.pl|\.cgi|\.scgi)
ignoreregex =
EOF
```

Ensuite, créez un filtre pour la prison `[nginx-nohome]`

Placez les informations de filtre suivantes dans le fichier :
/etc/fail2ban/filter.d/nginx-nohome.conf

```bash
cat > /etc/fail2ban/filter.d/nginx-nohome.conf << EOF
[Definition]
failregex = ^<HOST> -.*GET .*/~.*
ignoreregex =
EOF
```

Enfin, nous pouvons créer le filtre pour le jail `[nginx-noproxy]`

Cette définition de filtre correspondra aux tentatives d'utilisation de votre serveur comme proxy :
/etc/fail2ban/filter.d/nginx-noproxy.conf

```bash
cat > /etc/fail2ban/filter.d/nginx-noproxy.conf << EOF
[Definition]
failregex = ^<HOST> -.*GET http.*
ignoreregex =
EOF
```

### Activation de vos Jails Nginx

Pour mettre en œuvre vos modifications de configuration, vous devez redémarrer le service fail2ban. Vous pouvez le faire en tapant

    sudo systemctl restart fail2ban 

Le service devrait redémarrer et mettre en œuvre les différentes politiques de bannissement que vous avez configurées.
Obtenir des informations sur les prisons activées

Vous pouvez voir toutes vos prisons activées en utilisant la commande fail2ban-client :

    sudo fail2ban-client status

Vous devriez voir une liste de tous les jails que vous avez activés :

```
Status
|- Number of jail:	5
`- Jail list:	nginx-badbots, nginx-nohome, nginx-noproxy, nginx-noscript, sshd
```

### Tester fail2ban - ssh  

Depuis un VPS, on va essayer de se connecter sur rnmkcy.eu via ssh

```
yann@xoyaz:~$ ssh riri@rnmkcy.eu -p 55215
riri@rnmkcy.eu: Permission denied (publickey).
yann@xoyaz:~$ ssh riri@rnmkcy.eu -p 55215
riri@rnmkcy.eu: Permission denied (publickey).
yann@xoyaz:~$ ssh riri@rnmkcy.eu -p 55215
riri@rnmkcy.eu: Permission denied (publickey).
yann@xoyaz:~$ ssh riri@rnmkcy.eu -p 55215
ssh: connect to host rnmkcy.eu port 55215: Connection refused
```

Comme vous pouvez le voir dans la sortie ci-dessus, après trois échecs consécutifs, Fail2Ban bloque activement la connexion SSH. Après trois échecs consécutifs, la connexion est interrompue et l'utilisateur est bloqué pendant la durée spécifiée. Si vous essayez de vous connecter à nouveau pendant la période de blocage, vous obtenez une erreur "Connexion refusée" et vous n'êtes pas en mesure d'établir une connexion SSH au serveur.

### Fail2ban-client

Maintenant que vous êtes opérationnel avec Fail2ban, vous devez connaître quelques commandes d'exploitation de base. Pour ce faire, nous utilisons la commande fail2ban-client. Vous devrez peut-être avoir les privilèges sudo, selon votre configuration.

#### Surveillance fail2ban-client

L'un des principaux avantages de Fail2Ban est qu'il vous permet de surveiller activement toutes les tentatives d'authentification qui ont échoué et les différentes adresses IP qui ont été bloquées. Ces informations vous aident à comprendre l'ampleur des attaques auxquelles vous êtes confronté et la géolocalisation des attaques en analysant l'origine des adresses IP.

Vous pouvez utiliser l'outil Fail2Ban-client pour vérifier l'état de Fail2Ban et des prisons actives. 

    sudo fail2ban-client status

```
Status
|- Number of jail:	1
`- Jail list:	sshd
```

Pour afficher l'état et les informations concernant une prison particulière comme sshd, vous pouvez utiliser la commande suivante 

    sudo fail2ban-client status sshd

```
Status for the jail: sshd
|- Filter
|  |- Currently failed:	0
|  |- Total failed:	3
|  `- Journal matches:	_SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned:	4
   |- Total banned:	4
   `- Banned IP list:	109.123.254.249 2a02:7b40:c3b5:f29c::1 2a02:c206:2108:3749::1 195.181.242.156
```

#### Bannir/Débannir adresse IP

Bannir une adresse IP  
`sudo fail2ban-client set nginx-badbots banip <adresse IP>`

Débannir une IP  
`sudo fail2ban-client set nginx-badbots unbanip <ip address>`

## Liens

* [Configuration of fail2ban with UFW](https://www.czeo.com/configure-fail2ban-with-ufw/)
* [Fail2Ban - Bannir automatiquement les intrus](https://www.linuxtricks.fr/wiki/fail2ban-bannir-automatiquement-les-intrus)
* [How To Protect an Nginx Server with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-an-nginx-server-with-fail2ban-on-ubuntu-14-04)
* [How to Install & Configure Fail2ban on Debian 11](https://www.linuxcapable.com/how-to-install-configure-fail2ban-on-debian-11/)
* [Using Fail2Ban for SSH Brute-force Protection](https://www.linode.com/docs/guides/how-to-use-fail2ban-for-ssh-brute-force-protection/)
* [Comment protéger SSH avec Fail2ban des attaques DoS / Bruteforce](https://www.malekal.com/comment-proteger-ssh-avec-fail2ban-des-attaques-dos-bruteforce/)
* [Installer et configurer Fail2ban + UFW sur Debian 11](/posts/Debian_11_Fail2ban_UFW/)

