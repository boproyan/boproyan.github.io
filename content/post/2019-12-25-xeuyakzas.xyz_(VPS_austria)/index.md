+++
title = 'xeuyakzas.xyz (VPS austria)'
date = 2018-11-23 00:00:00 +0100
categories = []
+++
xeuyakzas.xyz (VPS austria)
---
layout: article
title:  xeuyakzas.xyz (VPS austria)
toc: true
ref: (falcutatif)
date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23
tags: [yunohost]
lang: fr
description:  serveur VPS situé à Vienne Autriche
---

# xeuyakzas.xyz 

**xeu** :  
-*Code employé par la norme internationale ISO 639-3 pour désigner un langage parlé en Nouvelle-Guinée : le keoru-ahia.*  
-*Code employé par la norme internationale ISO 4217 pour désigner l'ancienne devise de la Communauté Européenne : l'Unité de Compte Européenne (ECU).La devise XEU est l'ancêtre de l'Euro.*

**yak** :  
*Le yak ou yack est une grande espèce de ruminant à longue toison de l'Himalaya.*

**zas** :  
*Zas est une commune de la province de la Corogne en Galice (Espagne).* 

## VPS Waveride Austria 

VPS **Debian 8.0 x86_64 minimal** au 26 septembre 2017  

  * Accès SSH: **ssh -p 48022 -i ~/.ssh/vps-austria yak@xeuyakzas.xyz** ou **sshm yak**
  * [Waveride VPS Control Panel](https://panel.waveride.at/)
  * Operating System 	Debian 8.0 x86_64 min
  * IPv6 Address 	1
  * Disk Space 	50 GB
  * Bandwidth 	1.95 TB
  * Memory 	4 GB
  * VSwap 	1 GB
  * Enable TUN/TAP pour le VPN ( **TUN/TAP ON** )
  * Hostname xeuyakzas
  * Modify root password
  * IPV4 address: **151.236.10.24**
  * IPv6 address: **2a03:f80:ed16:ca7:ea75:b12d:1e2:bdc7**
  * Certificats Let'sencrypt :  
      * xeuyakzas.xyz , static.xeuyakzas.xyz , cartes.xeuyakzas.xyz , wiki.xeuyakzas.xyz

### Première connexion SSH

Via SSH  
  `ssh root@151.236.10.24`  
Màj  
  `apt update && apt upgrade`  
Installer nano curl et tmux  
  `apt install nano curl tmux`  

### Création utilisateur 

Utilisateur **yak**  
  `useradd -m -d /home/yak/ -s /bin/bash yak`  
Mot de passe **yak**  
  `passwd yak`  
Déconnexion puis connexion ssh en mode utilisateur  
  `ssh yak@151.236.10.24`  
  
### SSH via clé 

Sur le poste distant , générer un jeu de clé  
  `$ ssh-keygen -t rsa -b 4096 -C vps-austria -f ~/.ssh/vps-austria`  
Copier la clé publique depuis le poste distant dans /home/$USER  
  `$ scp ~/.ssh/vps-austria.pub yak@151.236.10.24:/home/yak/`  

Copier le contenu de la clé publique dans /home/$USER/.ssh/authorized_keys  
  `$ cd ~`  
Sur le VPS ,créer un dossier .ssh  

```
  $ pwd  #pour vérifier que l'on est sous /home/$USER
  $ mkdir .ssh
  $ cat /home/$USER/vps-austria.pub >> /home/$USER/.ssh/authorized_keys
```

et donner les droits  
  `$ chmod 600 /home/$USER/.ssh/authorized_keys`  
effacer le fichier de la clé  
  `$ rm /home/$USER/vps-austria.pub`  
Modifier la configuration serveur SSH  
  `$ su ` # passage en superutilisateur  
  `# nano /etc/ssh/sshd_config`  

```
Port = 48022
PermitRootLogin no
PasswordAuthentication no
```

Exécution bash sur connexion ssh fichier **~/.ssh/rc **  

```
#!/bin/bash

#clear
PROCCOUNT=`ps -Afl | wc -l`  		# nombre de lignes
PROCCOUNT=`expr $PROCCOUNT - 5`		# on ote les non concernées
GROUPZ=`users`
ipinfo=$(curl -s ipinfo.io) 		# info localisation format json
publicip=$(echo $ipinfo | jq -r '.ip')  # extraction des données , installer préalablement "jq"
ville=$(echo $ipinfo | jq -r '.city')
pays=$(echo $ipinfo | jq -r '.country')
cpuname=`cat /proc/cpuinfo |grep 'model name' | cut -d: -f2 | sed -n 1p`

echo "\033[0m\033[1;31m"  
figlet "xeuyakzas.xyz"
echo "\033[0m"
echo "\033[1;35m  \033[1;37mHostname \033[1;35m= \033[1;32m`hostname`
\033[1;35m  \033[1;37mWired Ip \033[1;35m= \033[1;32m`ip addr show venet0 | grep 'brd\b' | awk '{print $2}' | cut -d/ -f1`
\033[1;35m    \033[1;37mKernel \033[1;35m= \033[1;32m`uname -r`
\033[1;35m    \033[1;37mDebian \033[1;35m= \033[1;32m`cat /etc/debian_version`
\033[1;35m    \033[1;37mUptime \033[1;35m= \033[1;32m`uptime | sed 's/.*up ([^,]*), .*/1/' | sed -e 's/^[ \t]*//'`
\033[1;35m       \033[1;37mCPU \033[1;35m= \033[1;32m`echo $cpuname`
\033[1;35m\033[1;37mMemory Use \033[1;35m= \033[1;32m`free -m | awk 'NR==2{printf "%s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }'`
\033[1;35m  \033[1;37mUsername \033[1;35m= \033[1;32m`whoami`
\033[1;35m  \033[1;37mSessions \033[1;35m= \033[1;32m`who | grep $USER | wc -l` 
\033[1;35m\033[1;37mPublic Ip  \033[1;35m= \033[1;32m`echo $publicip $pays`
\033[0m"
curl fr.wttr.in/$publicip?0
```

Relancer openSSH  
  `# systemctl restart sshd`  

Accès depuis le poste distant avec la clé privée  
  `$ ssh -p 48022 -i ~/.ssh/vps-austria yak@151.236.10.24`  

### sudo 

passage en superutilisateur et installation sudo   
   `$ su`   
   `apt install sudo`  
Configuration simplifiée ,ajouter utilisateur et ses droits en fin de fichier  
  `echo "yak     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers`  


### Fuseau/Locales

Fuseau horaire en mode console root  
  `# dpkg-reconfigure tzdata` 

```
  Geographic area: 8  # Europe
  Time zone: 37       # Paris

Current default time zone: 'Europe/Paris'
Local time is now:      Sat Aug 27 13:28:39 CEST 2016.
Universal Time is now:  Sat Aug 27 11:28:39 UTC 2016.
```

Locales  
  `dpkg-reconfigure locales`  

```
Generating locales (this might take a while)...
  fr_FR.UTF-8... done
Generation complete.
```

### ipinfo.io 

`curl ipinfo.io`  

```
{
  "ip": "151.236.10.24",
  "hostname": "xeuyakzas.xyz",
  "city": "",
  "region": "",
  "country": "AT",
  "loc": "48.2000,16.3667",
  "org": "AS57169 EDIS GmbH"
}
```

**48.2000,16.3667 -> Karlsplatz 1010 Vienne Autriche**


## YunoHost 

 * [Voir sur wiki]({{ site.url_wiki }}/doku.php?id=xeuyakzas#yunohost)

### Owncloud -> Nextcloud

 * Migration [**Yunohost/Owncloud** -> **Yunohost/Nextcloud**]({{ site.baseurl }}post_url 2016-11-16-yunohost-migrer-owncloud-vers-nextcloud %})

Les applications activées par l'administrateur :  

* Confirmée : Contacts ,Calendar
* Expérimentale : OwnNote ,QOwnNotesApi


### Certificats SSL letsencrypt 

 * [Yunohost , installer et renouveler les certificats Let’s encrypt (static.xeuyakzas.xyz)]({{ site.baseurl }}post_url 2017-08-31-Acme-Certficats-Serveurs %})  


### Mises à jour de sécurité Debian

[**Mises à jour de sécurité Debian**]({{ site.baseurl }}post_url 2016-12-17-Debian-ssmtp-Mise-A-jour-automatique-avec-Cron-APT %})

### Applications

**Dokuwiki** : DokuWiki est un wiki Open Source simple à utiliser et très polyvalent qui n'exige aucune base de données.  
**Nextcloud** : Consultez et partagez vos fichiers, agendas, carnets d'adresses, emails et bien plus depuis les appareils de votre choix, sous vos conditions  
**Rainloop** : Webmail léger multi-comptes  
**Tiny tiny RSS** : Un lecteur de flux en PHP et Ajax  
**Transmission** : Un client BitTorrent libre et rapide  
**PhpMyaAdmin** : Application web de gestion des bases de données MySQL  

#### Shaarli

En mode su  
`yunohost app install https://github.com/YunoHost-Apps/shaarli_ynh`  

```
Domaines disponibles :
- xeuyakzas.xyz
Choisissez un domaine pour votre Shaarli (default: xeuyakzas.xyz) : 
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
`javascript:(function(){var%20url%20=%20location.href;var%20title%20=%20document.title%20||%20url;window.open('https://xeuyakzas.xyz/shaarli/?post='%20+%20encodeURIComponent(url)+'&title='%20+%20encodeURIComponent(title)+'&description='%20+%20encodeURIComponent(document.getSelection())+'&source=bookmarklet','_blank','menubar=no,height=390,width=600,toolbar=no,scrollbars=no,status=no,dialog=1');})();`

Créer le lien firefox **Shaarli liens publiques xeuyakzas.xyz**  <https://xeuyakzas.xyz/shaarli/?do=tagcloud>  

#### StaticWebApp

En mode su  
`yunohost app install https://github.com/YunoHost-Apps/staticwebapp_ynh`  

```
Domaines disponibles :
- xeuyakzas.xyz
Choisissez un domaine pour votre Webapp (default: xeuyakzas.xyz) : 
Choisissez un chemin pour votre Webapp (default: /app) : /static
Est-ce un site public ? [0 | 1] (default: 1) : 
Attention : « yunohost app checkurl » est déprécié et sera bientôt supprimé
Succès ! La configuration de SSOwat a été générée
Succès ! Installation terminée
```

Redirection

	sudo nano /var/www/staticwebapp/index.html

```
<!DOCTYPE html>
<html>
<head>
   <!-- HTML meta refresh URL redirection -->
   <meta http-equiv="refresh" content="0; url=https://static.ouestline.net/">
</head>
<body>
   <p>Redirection vers <a href="https://static.ouestline.net/">static.ouestline.net</a></p>
</body>
</html>
```


