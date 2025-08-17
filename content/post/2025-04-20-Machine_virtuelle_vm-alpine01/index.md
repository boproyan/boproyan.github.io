+++
title = 'cwwk KVM Alpine Linux Navidrome (vm-alpine01)'
date = 2025-04-20 00:00:00 +0100
categories = ['virtuel']
+++
*Alpine Linux est une distribution Linux ultra-légère*

![](alpine-linux-logo.png){:width="300"}  

## Alpine Linux

*Création machine virtuelle Alpine  de type KVM avec 2 Go de RAM, 1 cœur de processeur et 5 Go de disque dur.*

### Créer vm-alpine sur un serveur

[Les dernières images Alpine Linux](https://alpinelinux.org/downloads/)  

Création d'une image virtuelle **vm-alpine01** sous le serveur Lenovo rnmkcy.eu  
On se connecte sur le serveur Lenovo en SSH, puis on exécute la commande suivante pour créer  une machine virtuelle Alpine avec 2 Go de RAM, 1 cœur de processeur et 5 Go de disque dur

```shell
sudo virt-install \sudo virt-install \
--osinfo alpinelinux3.17 \
--name vm-alpine01 \
--memory 2048 \
--vcpus 1 \
--cpu host \
--hvm \
--disk path=/srv/kvm/libvirt/images/vm-alpine01.qcow2,format=qcow2,size=5 \
--cdrom /home/yick/FreeUSB2To/iso/alpine-standard-3.21.0-x86_64.iso \
--network bridge=br0 \
--graphics vnc  
```

Note: La dernière version Alpine Linux est alpinelinux3.20  au 10/07/2024 mais KVM ne connait que alpinelinux3.17 (`sudo virt-install --osinfo list |grep alpine`)

Après exécution dans un terminal de la commande ci dessus, on arrive sur *En attente de fin d'installation*

### Configurer vm-alpine

`ATTENTION: Désactiver "Affichage VNC" des autres machines`{: .prompt-warning }

VNC, configuration xml

```xml
<graphics type="vnc" port="5900" autoport="yes" listen="127.0.0.1">
  <listen type="address" address="127.0.0.1"/>
</graphics>
```

Le serveur Lenovo n'a pas d'affichage, il faut créer un tunnel ssh depuis un terminal d'un poste client

    ssh -L 5900:127.0.0.1:5900 yick@192.168.0.205 -p 55205 -i /home/yann/.ssh/yick-ed25519

Puis lancer de ce même poste un client VNC  
![](alpine-linux02.png){:width="300"}  
la console s'affiche   
![](alpine-linux03.png)  

Une fois l'image ISO lancée, on arrive à un invite de connexion.   
Indiquez `root` comme nom d'utilisateur, aucun mot de passe ne vous sera demandé à cette étape.   

Le système est utilisable, mais on veut l'installer, ce qui passe par la commande suivante (clavier qwerty)

```
setup-alpine # saisir setup)qlpine
```

Une suite de questions :  
![](alpine-linux04.png)  
mot de passe root (rtyuiop)  
![](alpine-linux05.png)  
APK mirror...  
![](alpine-linux05a.png)  

Utilisateur alpi/alpi49 et suite...   
![](alpine-linux06.png)  

Relever l'adresse ip allouée : `ip a` --> 192.168.10.12  
Puis redémarrer : `reboot`  
La fenêtre vnc se ferme  

On n'a plus besoin de VNC, en mode graphique   
![](alpine-linux06a.png){:width="400"}  
![](alpine-linux06b.png){:width="400"}  
![](alpine-linux06c.png){:width="400"}  
Eteindre puis redémarrer la machine virtuelle 

### Explications sur la procédure

*Normalement, vous n'avez rien à faire, les paramètres par défaut doivent convenir. Mais si vous le désirez, vous pouvez les modifier pour utiliser une interface particulière, une IP fixe, un serveur proxy, etc.  
Une soixantaine de serveurs mirroir vous seront proposés pour télécharger les paquets. Choisissez un numéro dans la liste ou demandez au système de les tester et de sélectionner le plus rapide. Vous pouvez aussi modifier le fichier des sources. Il vous faudra ensuite choisir votre serveur SSH : OpenSSH, Dropbear ou aucun.* 

On termine par la méthode d'installation. Il en existe quatre : 

*    none : le système et ses données sont placés en RAM et seront perdus après le redémarrage
*    sys : le système et ses données sont placés sur un HDD/SSD
*    data : le système est placé en RAM, les données sur un HDD/SSD
*    lvm : utilisation de Logical Volume Manager, les deux choix précédents seront proposés (lvmsys, lvmdata)

Si vous stockez le système en mémoire, il faudra trouver un moyen de sauvegarder la configuration. Vous pourrez le faire uniquement depuis un lecteur de disquettes (!) ou une clé USB. Une fois le système installé, vous pourrez l'utiliser directement s'il est placé en mémoire ou redémarrer si vous avez opté pour un stockage classique.

Il n'est pas conseillé d'utiliser directement le compte root pour les actions du quotidien.  
Si utilisateur non créé dans la procédure d'installation, le créer avec son propre espace dans /home/ 

    adduser alpi

Vous pouvez utiliser l'utilisateur pour vous connecter via SSH (impossible avec le compte root)  

### Connexion vm-alpine via SSH

Sur un poste linux du réseau

    ssh alpi@192.168.10.12

Une fois connecté ,vous pouvez accéder au "root" de manière classique avec la commande :

    su -

Mise à jour

```shell
apk update
apk upgrade 
# Vous pouvez fusionner les deux lignes avec 
apk -U upgrade
```

Editeur nano (Vous pouvez aussi opter pour vi qui est nativement présent sur le système)

    apk add nano

### Réseau - IP statique

[How to configure static IP address on Alpine Linux](https://www.cyberciti.biz/faq/how-to-configure-static-ip-address-on-alpine-linux/)

Le fichier de configuration `/etc/network/interfaces`

    /etc/network/interfaces

```shell
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
  address 192.168.10.210
  gateway 192.168.10.1
```

Fichier de résolution dns

    /etc/resolv.conf

```
search yann.priv
nameserver 192.168.10.1
```

Les modifications apportées à /etc/network/interfaces peuvent être activées en exécutant

```shell
service networking restart 
```

`ATTENTION: Déconnexion SSH car changement adresse IP`{: .prompt-warning }

Connexion SSH avec IP 192.168.10.210

    ssh alpi@192.168.10.210

Message à la connexion SSH, `/etc/motd`

```
    _    _       _              _     _                   
   / \  | |_ __ (_)_ __   ___  | |   (_)_ __  _   ___  __ 
  / _ \ | | '_ \| | '_ \ / _ \ | |   | | '_ \| | | \ \/ / 
 / ___ \| | |_) | | | | |  __/ | |___| | | | | |_| |>  <  
/_/   \_\_| .__/|_|_| |_|\___| |_____|_|_| |_|\__,_/_/\_\ 
          |_|                _       _             ___  _ 
__   ___ __ ___         __ _| |_ __ (_)_ __   ___ / _ \/ |
\ \ / / '_ ` _ \ _____ / _` | | '_ \| | '_ \ / _ \ | | | |
 \ V /| | | | | |_____| (_| | | |_) | | | | |  __/ |_| | |
  \_/ |_| |_| |_|      \__,_|_| .__/|_|_| |_|\___|\___/|_|
 _   _             _     _    |_|                         
| \ | | __ ___   _(_) __| |_ __ ___  _ __ ___   ___       
|  \| |/ _` \ \ / / |/ _` | '__/ _ \| '_ ` _ \ / _ \      
| |\  | (_| |\ V /| | (_| | | | (_) | | | | | |  __/      
|_| \_|\__,_| \_/ |_|\__,_|_|  \___/|_| |_| |_|\___|      
 _  ___ ____    _  __    ___   _  ___   ____  _  ___      
/ |/ _ \___ \  / |/ /_  ( _ ) / |/ _ \ |___ \/ |/ _ \     
| | (_) |__) | | | '_ \ / _ \ | | | | |  __) | | | | |    
| |\__, / __/ _| | (_) | (_) || | |_| | / __/| | |_| |    
|_|  /_/_____(_)_|\___/ \___(_)_|\___(_)_____|_|\___/     

```

### OpenSSH avec clés

*Connexion ssh sur un autre port avec un jeu de clés*

Générer une paire de clé sur l'ordinateur de bureau PC1  
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec la machine virtuelle vm-alpine01

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/vm-alpine01
    chmod 600 ~/.ssh/vm-alpine01

Copier la clé publique `cat ~/.ssh/vm-alpine01.pub` dans le presse-papier

	ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHJXXXPhNI7WD1q9AJ70fnoI2OyNjReheLKR/hFhWaiz yann@PC1

On se connecte sur la machine virtuelle alpine linux "vm-alpine01" en proxy avec le serveur cwwk (192.168.0.205)

```bash
ssh -o ProxyCommand="ssh -W %h:%p -p 55205 -i /home/yann/.ssh/yick-ed25519 yick@192.168.0.205" alpi@192.168.10.210 -p 55110 -i /home/yann/.ssh/vm-alpine01
```

Créer le répertoire et ouvrir nouveau fichier

    mkdir -p $HOME/.ssh/
    nano $HOME/.ssh/authorized_keys

Coller le contenu du presse-papier , sauver le fichier et sortir

Modifier les droits

    chmod 600 $HOME/.ssh/authorized_keys

Passer en mode su

    su -

Modifier la configuration serveur SSH

    nano /etc/ssh/sshd_config

Modifier

```
Port = 55110
PasswordAuthentication no
```

Relancer le serveur

    service sshd restart

Test connexion

```bash
ssh -o ProxyCommand="ssh -W %h:%p -p 55205 -i /home/yann/.ssh/yick-ed25519 yick@192.168.0.205" alpi@192.168.10.210 -p 55110 -i /home/yann/.ssh/vm-alpine01
```

### sudo

Passer en root

    su -

**Ajout dépôt communauté**  
Editer la configuration des dépôts

    nano /etc/apk/repositories

Trouvez maintenant la ligne qui se termine dans **/community**  
Ensuite, retirez le `#` au début de la ligne.  
Le fichier résultant devrait ressembler à ceci

```
#/media/cdrom/apks
http://mirrors.ircam.fr/pub/alpine/v3.20/main
http://mirrors.ircam.fr/pub/alpine/v3.20/community
```

Installer sudo 

```bash
apk update
apk add sudo
```

Ajouter un utilisateur avec les privlèges root

```bash
echo "alpi     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/20-alpi
```

### Partage (virtiofs)

*Virtiofs est un système de fichiers partagé qui permet aux machines virtuelles d'accéder à une arborescence de répertoires sur l'hôte. Contrairement aux approches existantes, il est conçu pour offrir la sémantique et les performances d'un système de fichiers local.*

`Un arrêt puis un démarrage de la machine virtuelle est obligatoire`{: .prompt-warning }

Il faut valider **Enable shared memory**  
![](virtiofs01.png){:width="500"}  
Puis ajouter un **système de fichier**  
![](virtiofs02.png){:width="500"}  

Vue des éléments XML de domaine suivants pour partager le répertoire hôte /path avec l'invité

```xml
<domaine>
      ...
  <memoryBacking>
    <source type="memfd"/>
    <access mode="shared"/>
  </memoryBacking>
      ...
  <devices>
        ...
    <filesystem type="mount" accessmode="passthrough">
      <driver type="virtiofs"/>
      <source dir="/sharenfs"/>
      <target dir="sharenfs"/>
    </filesystem>
        ...
   </devices>
</domaine>
```

Créer dossier local en mode utilisateur

    mkdir -p ~/sharenfs

Le partage sur `/etc/fstab` en mode su

```
sharenfs   /home/alpi/sharenfs   virtiofs rw 0 0
```

monter

```
sudo mount -a
```

### Démarrage automatique VM

Démarrage auto  
![](virtiofs02a.png){:width="500"}  

### Modifier DNS

Ajout serveur DNS Unbound 192.168.0.205 (présent dans le serveur debian 12 cwwk rnmkcy.eu)  
dans le fichier `/etc/resolv.conf`

```
nameserver 192.168.0.205
nameserver 192.168.10.1
```

Outils Réseau

    apk update && apk add bind-tools

### Domaine et certificat alp01.home.arpa

Le paquet ca-certificats qui est commun dans chaque distribution Linux est absent de l'installation par défaut de Alpine, vous pouvez installer le paquet ca-certificats avec la commande suivante

```shell
sudo apk -U add ca-certificates
```

Les fichiers sont disponibles dans le dossier `~/sharenfs/rnmkcy/.mkcert/`

Nous recopions server.crt dans /usr/local/share/ca-certificats/ 

```shell
sudo mkdir -p /usr/local/share/ca-certificats/
sudo cp ~/sharenfs/rnmkcy/.mkcert/rootCA.pem /usr/local/share/ca-certificats/mkcert.crt
```

Enfin, nous pouvons ajouter le certificat à la collection ca-certificats en utilisant la commande update-ca-certificats:

```shell
sudo update-ca-certificates
```

Cette commande ajoute server.crt au /etc/ssl/certs/ca-certificats.crt.

Nous pouvons vérifier les mises à jour au bas du fichier.

```shell
sudo cp ~/sharenfs/rnmkcy/.mkcert/cwwk.home.arpa+1-key.pem /etc/ssl/private/alp01.home.arpa-key.pem 
sudo cp ~/sharenfs/rnmkcy/.mkcert/cwwk.home.arpa+1.pem  /etc/ssl/private/alp01.home.arpa.pem    
```

### Serveur web nginx

Installation nginx

    sudo apk add nginx

Nom utilisateur dans le fichier de configuration

    sudo nano /etc/nginx/nginx.conf

```
# /etc/nginx/nginx.conf

user alpi;
```

On crée ensuite le fichier de configuration du serveur web `/etc/nginx/http.d/default.conf`

    sudo nano 

Modifier le contenu du fichier par la configuration suivante

```
cat <<EOF >>/etc/nginx/http.d/default.conf
server {
  listen  80;
  server_name alp01.home.arpa;
  return 301 https://$server_name$request_uri;
}
server {
  listen *:443 ssl;
  server_name alp01.home.arpa;
  ssl_certificate /etc/ssl/private/alp01.home.arpa.pem;
  ssl_certificate_key /etc/ssl/private/alp01.home.arpa-key.pem;
    root /srv/www/;
    location /files {		
        autoindex on;	
    }
}
EOF

```

On crée ensuite les dossiers et fichiers nécessaires. Vous noterez que nous plaçons le contenu de nos sites web dans le dossier /srv/www/ ce qui n'est pas toujours l'emplacement utilisé par défaut. Il n'y a en effet pas consensus sur le « bon endroit » pour cela, mais la version 3.0 de la FHS (Filesystem Hierarchy Standard), qui fait référence en la matière, précise que /srv/ doit être utilisé pour les services diffusant des données, dont « www »


On modifie les dossiers créés et les librairies de nginx pour en donner la propriété à l'utilisateur ainsi qu'au groupe www-data, qui est celui en général en charge de l'accès aux serveurs web et à leurs données :

```
sudo adduser $USER www-data
sudo mkdir -p /srv/www/files/
sudo chown $USER:www-data -R /srv/www
sudo chown -R $USER:www-data /var/lib/nginx
echo "Serveur alp01.home.arpa !" > /srv/www/index.html
# Télécharger logo
wget https://static.rnmkcy.eu/alpine-linux-logo.png -O /srv/www/files/logo.png
```

Tout est prêt, on peut donc tester le fichier de configuration et lancer nginx s'il est valide 

```
sudo nginx -t
sudo rc-service nginx start
```

Les URL suivantes devraient fonctionner depuis votre machine (adaptez avec le hostname ou l'IP du serveur) 

```
https://alp01.home.arpa/
https://alp01.home.arpa/files/
https://alp01.home.arpa/files/logo.png
```

Vous pouvez ajouter nginx à la liste des services à lancer au démarrage de la machine 

```
sudo rc-update add nginx default
reboot
```

---

## Bash sur Alpine Linux (NON INSTALLE)

[How to install bash shell in Alpine Linux](https://www.cyberciti.biz/faq/alpine-linux-install-bash-using-apk-command/)  
Par défaut, **bash** n'est pas inclus avec BusyBox et Linux alpine.

**Installer un shell bash dans Alpine Linux**  
Voici comment installer un shell bash dans Alpine Linux en utilisant la commande apk.

Passer en mode root : `su -`

```bash
apk update
apk upgrade
apk add bash
```

Obtenez une liste des "shell" de connexion valides sur Linux Alpine 

    cat /etc/shells

Produit :

```
# valid login shells
/bin/sh
/bin/ash
/bin/bash
```

**documentation et auto complétion**  
Installer la documentation de bash : `apk add bash-doc`  
Installer l'auto complétion bash : `apk add bash-completion`

Vérifiez et appelez /etc/bash/bash_completion.sh 

```bash
source /etc/bash/bash_completion.sh
cat /etc/bash/bash_completion.sh
```

**Comment configurer bash comme shell login**

Pour utiliser bash  juste saisir : `bash`  
**Changer le root shell en bash**  
En mode su : `nano /etc/passwd`  
Remplacer `/bin/sh` par `/bin/bash` pour les utilisateurs root et alpi

```
root:x:0:0:root:/root:/bin/bash
alpi:x:1000:1000:alpi:/home/alpi:/bin/bash
```

Personnaliser le bash

```bash
cat > ~/.bashrc << EOF
alias update='apk update && apk upgrade'
export HISTTIMEFORMAT="%d/%m/%y %T "
export PS1='\u@\h:\W \$ '
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'
alias ls='ls --color=auto'
source /etc/bash/bash_completion.sh
export PS1="\[\e[31m\][\[\e[m\]\[\e[38;5;172m\]\u\[\e[m\]@\[\e[38;5;153m\]\h\[\e[m\] \[\e[38;5;214m\]\W\[\e[m\]\[\e[31m\]]\[\e[m\]\\$ "
EOF
```

Version bash : `bash --version`

```
GNU bash, version 5.2.26(1)-release (x86_64-alpine-linux-musl)
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>

This is free software; you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

## Docker (NON INSTALLE)

![](docker-logo-a.png)  

### Installation

[Install docker & docker-compose on Alpine Linux](https://geekscircuit.com/install-docker-docker-compose-on-alpine-linux/)

Passer en root

    su -

**Ajout dépôt communauté (si non effectué lors de l'installation)**  
Editer la configuration des dépôts

    nano /etc/apk/repositories

Trouvez maintenant la ligne qui se termine dans /community Ensuite, retirez le # au début de la ligne. Le fichier résultant devrait ressembler à ceci

```
#/media/cdrom/apks
http://mirrors.ircam.fr/pub/alpine/v3.20/main
http://mirrors.ircam.fr/pub/alpine/v3.20/community
```

**Installation docker**  
Installer docker et docker-compose

```bash
apk update
apk add docker docker-compose
```

Activer autostart sur boot en utilisant

```bash
rc-update add docker default
```

puis vous pouvez lancer le service docker en utilisant la commande

```bash
/etc/init.d/docker start
# ou
service docker start
```

## Site statique (NON INSTALLE)

### Ruby

![](ruby-logoa.png)

**Installation de Ruby avec RVM sur Alpine Linux ([Installing Ruby with RVM on Alpine Linux](https://btxx.org/posts/rvm/))**  
Passer en mode root : `su -`

Paquets de base

```bash
apk update
apk add curl gcc gnupg gpg dirmngr procps musl-dev linux-headers zlib zlib-dev openssl openssl-dev build-base
# libssl1.1
exit
```

En mode utilisateur  
Téléchargez ensuite la dernière version stable de rvm depuis Github, décompressez-la, placez-la dans le répertoire utilisateur approprié (~/.rvm) et installez toutes les bibliothèques requises :

```bash
curl -sSL https://github.com/rvm/rvm/tarball/stable -o rvm-stable.tar.gz
echo 'export rvm_prefix="$HOME"' > ~/.rvmrc
echo 'export rvm_path="$HOME/.rvm"' >> ~/.rvmrc
mkdir rvm && cd rvm
tar --strip-components=1 -xzf ../rvm-stable.tar.gz
./install --auto-dotfiles --autolibs=0
```

Faire le ménage et créer un lien vers **rvm** 

```bash
cd ../ && rm -rf rvm-stable stable.tar.gz rvm
source ~/.rvm/scripts/rvm
```

On va installer la version Ruby 3.1.4

    rvm install ruby-3.1.4

Patienter plusieurs minutes...

Vérification : `ruby -v`  
**ruby 3.1.4p223 (2023-03-30 revision 957bb7cb81) [x86_64-linux-musl]**

### Projet jekyll

On va dupliquer le dossier **yannstatic** existant sur PC1  en **alpistatic** sur vm-alpine01

####  Installer Jekyll et Bundler 

Vérifier les sources

```
localhost:~$ gem source
*** CURRENT SOURCES ***

# rien ci dessus ,on active une source
localhost:~$ gem source -a https://rubygems.org/
https://rubygems.org/ added to sources
# Vérification
localhost:~$ gem source
*** CURRENT SOURCES ***

https://rubygems.org/
```

installez Jekyll et Bundler 

    gem install jekyll bundler

Naviguez dans le répertoire de votre projet Jekyll et exécutez:

```bash
cd /home/alpi/alpistatic
```

Le fichier Gemfile

```
source "https://rubygems.org"

# gem "github-pages", group: :jekyll_plugins
gem "jekyll-text-theme"
gem "sass-embedded", "1.76.0"
```

Supprimer Gemfile.lock : `rm Gemfile.lock`

Bundle lit le fichier **Gemfile** 

    bundle install

`An error occurred while installing eventmachine (1.2.7), and Bundler cannot continue.`  
Il faut installer le paquet build-base 

GCC, qui représente GNU Compiler Collection, est un système de compilation libre et open-source produit par le projet GNU.  
Le moyen le plus rapide pour commencer avec GCC sur Alpine Linux est de lancer la commande suivante:

    sudo apk add build-base

build-base est un méta-package qui installera les paquets gcc, musl-dev et binutils (entre autres).

Info sur jekyll après installation : `bundle info jekyll`

```
  * jekyll (4.3.4)
	Summary: A simple, blog aware, static site generator.
	Homepage: https://jekyllrb.com
	Source Code: https://github.com/jekyll/jekyll
	Changelog: https://github.com/jekyll/jekyll/releases
	Bug Tracker: https://github.com/jekyll/jekyll/issues
	Path: /home/alpi/.rvm/gems/ruby-3.1.4/gems/jekyll-4.3.4
	Reverse Dependencies: 
		jekyll-feed (0.17.0) depends on jekyll (>= 3.7, < 5.0)
		jekyll-sitemap (1.4.0) depends on jekyll (>= 3.7, < 5.0)
		jekyll-text-theme (2.2.6) depends on jekyll (>= 3.6, < 5.0)
		jemoji (0.13.0) depends on jekyll (>= 3.0, < 5.0)
```

#### Création des liens

Création des liens sur les dossiers files , images , _posts et htmldoc

```bash
ln -s /home/alpi/statique/files /home/alpi/alpistatic/files
ln -s /home/alpi/statique/images /home/alpi/alpistatic/images
ln -s /home/alpi/statique/_posts /home/alpi/alpistatic/_posts
ln -s /home/alpi/statique/htmldoc /home/alpi/alpistatic/htmldoc
ln -s /home/alpi/statique/htmldoc/htmldoc.md /home/alpi/alpistatic/htmldoc.md
```

La structure des liens : `ls -l ~/alpistatic/ |grep ^l`

```
lrwxrwxrwx    1 alpi   alpi          28 Sep 18 19:43 _posts -> /home/alpi/statique/_posts
lrwxrwxrwx    1 alpi   alpi          27 Sep 18 19:43 files -> /home/alpi/statique/files
lrwxrwxrwx    1 alpi   alpi          29 Sep 18 19:46 htmldoc -> /home/alpi/statique/htmldoc
lrwxrwxrwx    1 alpi   alpi          40 Sep 18 19:44 htmldoc.md -> /home/alpi/statique/htmldoc/htmldoc.md
lrwxrwxrwx    1 alpi   alpi          28 Sep 18 19:43 images -> /home/alpi/statique/images
```

#### Génération dossier “static”


Dans le dossier **alpistatic**

```
cd $HOME/alpistatic
bundle exec jekyll build
```

C’est ça ! Maintenant si vous exécutez `bundle exec jekyll serve` vous trouverez votre projet Jekyll fonctionnant localement!

```
localhost:~/alpistatic$ bundle exec jekyll serve --source "$HOME/alpistatic"
Configuration file: /home/alpi/alpistatic/_config.yml
            Source: /home/alpi/alpistatic
       Destination: /home/alpi/alpistatic/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
                    done in 31.908 seconds.
 Auto-regeneration: enabled for '/home/alpi/alpistatic'
    Server address: http://127.0.0.1:4000
  Server running... press ctrl-c to stop.
```

Dans un terminal de PC1

    ssh -L 9500:localhost:4000 alpi@192.168.10.210

Dans un navigateur de PC1 : localhost:9500

#### Service gener_static

* [OpenRC](https://wiki.alpinelinux.org/wiki/OpenRC)
* [Working with OpenRC](https://docs.alpinelinux.org/user-handbook/0.1a/Working/openrc.html#_configuring_services)
* [How to Create and Manage a Service in an Alpine Linux Container](https://medium.com/@mfranzon/how-to-create-and-manage-a-service-in-an-alpine-linux-container-93a97d5dad80)

Créer un script `gener_static.sh`

    sudo nano /usr/local/bin/gener_static.sh

```
#!/bin/sh
echo $$ > /run/gener_static.pid  # Store the PID of this process
cd $HOME/alpistatic
bundle exec jekyll build --watch --incremental
```

Le rendre exécutable

    sudo chmod +x /usr/local/bin/gener_static.sh

### Augmenter taille disque virtuel

```
[yann@yann-pc1 ~]$ sudo virsh list
 ID   Nom           État
------------------------------------------
 1    vm-debian12   en cours d’exécution
 6    vm-alpine01   en cours d’exécution

[yann@yann-pc1 ~]$ sudo virsh shutdown vm-alpine01
Le domaine 'vm-alpine01' est en cours de fermeture

[yann@yann-pc1 ~]$ sudo virsh domblklist vm-alpine01
 Cible   Source
---------------------------------------------------
 vda     /home/yann/virtuel/KVM/vm-alpine01.qcow2


[yann@yann-pc1 ~]$ sudo qemu-img resize /home/yann/virtuel/KVM/vm-alpine01.qcow2 +15G
Image resized.

[yann@yann-pc1 ~]$ sudo qemu-img info /home/yann/virtuel/KVM/vm-alpine01.qcow2
image: /home/yann/virtuel/KVM/vm-alpine01.qcow2
file format: qcow2
virtual size: 20 GiB (21474836480 bytes)
disk size: 3.11 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: true
    refcount bits: 16
    corrupt: false
    extended l2: false
Child node '/file':
    filename: /home/yann/virtuel/KVM/vm-alpine01.qcow2
    protocol type: file
    file length: 5 GiB (5369757696 bytes)
    disk size: 3.11 GiB


[yann@yann-pc1 ~]$ sudo virsh start vm-alpine01
Domaine 'vm-alpine01' démarré

Connexion ssh alpi@192.168.10.210

# Outils
sudo apk add --no-cache cfdisk e2fsprogs-extra

# cfdisk pour redimensionner la partition vda3
localhost:~$ sudo cfdisk

sélectionner la partition à étendre puis resize write quit

Syncing disks.

# resize
localhost:~$ sudo resize2fs /dev/vda3
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/vda3 is mounted on /; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 3
The filesystem on /dev/vda3 is now 4838144 (4k) blocks long.

# vérification
localhost:~$ sudo df -H |grep vda3
/dev/vda3                18.1G      2.9G     14.4G  17% /


```

## Navidrome

![](navidrome-logo.png){:width="100"}  

*Comment installer Navidrome Music Server sur Alpine Linux. Navidrome est un serveur de musique libre et ouvert qui vous permet de diffuser votre collection de musique de n'importe où*

### Préalables

* Une machine virtuelle utilisant Alpine Linux Last
* Accès à l'interface en ligne de commande

### Installation

Assurez-vous que votre installation Alpine Linux est à jour 

```shell
sudo apk update && sudo apk update
```

Installez les dépendances Navidrome  

```shell
sudo apk add git sqlite flac ffmpeg 
```

Créer un utilisateur et groupe système

```shell
sudo adduser navidrome --system
sudo addgroup navidrome
```

Vérification utilisateur

```
/home/alpi # cat /etc/passwd |grep navidrome
navidrome:x:102:65533::/home/navidrome:/sbin/nologin
```

Créez un répertoire pour stocker l'exécutable Navidrome et de travail (même dossier) avec les permissions appropriées.

```shell
sudo mkdir -p /opt/navidrome
sudo chown navidrome:navidrome -R /opt/navidrome
```

Obtenez Navidrome

Télécharger la dernière version de la [page des versions](https://github.com/navidrome/navidrome/releases), extraire le contenu dans le répertoire exécutable, et définir les permissions pour les fichiers. (Remplacez l'URL ci-dessous avec celle de la page des versions) :

```shell
wget https://github.com/navidrome/navidrome/releases/download/v0.55.2/navidrome_0.55.2_linux_amd64.tar.gz -O Navidrome.tar.gz
sudo tar -xvzf Navidrome.tar.gz -C /opt/navidrome/
sudo chmod +x /opt/navidrome/navidrome
sudo chown -R navidrome:navidrome /opt/navidrome
```

Créer la configuration Navidrome, fichier nommé `/opt/navidrome/navidrome.toml` avec les paramètres suivants.

```
DefaultLanguage = "fr"
DataFolder = "/opt/navidrome/data"
MusicFolder = "/home/alpi/sharenfs/multimedia/Music/musicyan"
PlaylistsPath = "playlists"
AutoImportPlaylists = "true"
EnableInsightsCollector = "false"
```

Ensuite, nous devons générer un fichier **openrc**

```shell
sudo apk add navidrome-openrc
```

Modifier le fichier `/etc/init.d/navidrome` pour l'adapter à notre configuration

```shell
#!/sbin/openrc-run
supervisor=supervise-daemon

name=navidrome
description="Modern Music Server and Streamer compatible with Subsonic/Airsonic"

command=/opt/navidrome/navidrome
command_args="--nobanner --configfile /opt/navidrome/navidrome.toml"
command_user="navidrome:navidrome"
directory="/opt/$RC_SVCNAME"

extra_started_commands="scan"

start_pre() {
	[ -n "$error_log" ] && checkpath -f "$error_log" \
		-m 644 -o navidrome:navidrome || true
	[ -n "$ND_CACHEFOLDER" ] && checkpath -q -d \
		-m 0755 -o navidrome:navidrome "$ND_CACHEFOLDER" || true
}

scan() {
	${supervisor} "${SVCNAME}" --signal USR1
}

depend() {
	need net localmount
	after firewall
}
```

Ajout servcie au démarrage

```shell
rc-update add navidrome  
```

Démarrer Navidrome Music Serveur 

```shell
service navidrome start  
```

Navidrome Music Server devrait maintenant fonctionner.  

```
 * Caching service dependencies ...  [ ok ]
 * /var/log/navidrome.log: creating file
 * /var/log/navidrome.log: correcting owner
 * Starting navidrome ...            [ ok ]
```

Vous pouvez y accéder en ouvrant un navigateur Web et en naviguant sur `http://<server_ip>4533`.

## Maintenance

### Messagerie

[Alpine Linux - Relayer les e-mails vers un compte (msmtp, MailX, Sendmail)](/posts/Alpine_Linux-msmtp-mailx-sendmail/)

### Alpine Linux - Mises à jour automatique

Installer curl

    sudo apk add curl

Pour chaque nouveau serveur Alpine Linux, créer un script shell nommé apk-autoupgrade dans le dossier /etc/periodic/daily/ avecles permissions suivantes : 700

```shell
echo -e "#!/bin/sh\napk upgrade --update | sed \"s/^/[\`date\`] /\" >> /var/log/apk-autoupgrade.log" > /etc/periodic/daily/apk-autoupgrade && \
	chmod 700 /etc/periodic/daily/apk-autoupgrade
```

Si les tâches cron ne sont pas activées

```shell
rc-service crond start
rc-update add crond
```

Le script exécute la commande apk upgrade --update une fois par jour, apk par défaut ne demande jamais l’intervention de l’utilisateur. 

Additif pour notification ntfy

    nano /etc/periodic/daily/apk-autoupgrade

Ajouter ce qui suit en fin de ficheir

```
curl \
-H "X-Email: ntfy@cinay.eu" \
-H "Title: 💻 Alpine Linux Navidrome vm-alpine01 : Fin exécution script 'apk-autoupgrade'" \
-H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
-H prio:low \
-d "Navidrome vm-alpine01 192.168.10.210 
✔️ Fin exécution script /etc/periodic/daily/apk-autoupgrade" \
https://noti.rnmkcy.eu/yan_infos
```

