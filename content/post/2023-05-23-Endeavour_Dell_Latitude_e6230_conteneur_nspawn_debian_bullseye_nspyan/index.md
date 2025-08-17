+++
title = 'EndeavourOS Dell Latitude e6230 --> conteneur nspawn debian bullseye nspyan'
date = 2023-05-23 00:00:00 +0100
categories = ['laptop']
+++
*systemd-nspawn peut être utilisé pour exécuter une commande ou un système d'exploitation dans un espace de noms léger. Il est plus puissant que chroot car il virtualise entièrement la hiérarchie du système de fichiers, ainsi que l'arborescence des processus, les différents sous-systèmes IPC et le nom de l'hôte et du domaine.*

![](container.png){:width="150"}


## Portable Latitude e6230

![Dell Latitude E6230](dell-latitude-e6230.png){:width="150"}  

*Créer un environnement Debian bullseye avec un conteneur nommé nspyan dans un portable Dell Latitude e6230 EndeavourOS XFCE ([Archlinux --> Container systemd-nspawn](/posts/systemd-nspawn/))*  

### Création conteneur systemd-nspawn

#### Installer les dépendances debian

Installez **debootstrap**, ainsi que **debian-archive-keyring** ou **ubuntu-keyring**, ou les deux, en fonction de la distribution choisie.

    yay -S debootstrap debian-archive-keyring

**Assurez-vous que le paquetage *dbus* ou *dbus-broker* est installé sur le système de conteneurs.**
{: .prompt-info }

    yay -Ss dbus

```
core/dbus 1.14.6-2 (304.7 KiB 911.4 KiB) (Installé)
```

#### Installer le système d'exploitation invité debian

Créer un dossier pour les containers

    mkdir -p $HOME/virtuel

Configurer l'environnements Debian 

```shell
sudo debootstrap --arch amd64  --include=systemd-container,systemd,ssh,locales,dbus bullseye  $HOME/virtuel/nspbullseye https://deb.debian.org/debian
```

Le container nspbullseye est créé

```
I: Base system installed successfully.
```

#### Lien /var/lib/machines 

Lien /var/lib/machines avec $HOME/virtuel

    sudo rmdir /var/lib/machines
    sudo ln -s $HOME/virtuel/ /var/lib/machines

#### Créer mot de passe root dans système d'exploitation invité

    sudo systemd-nspawn -D /var/lib/machines/nspbullseye -U --machine nspbullseye

*Cela permet d'obtenir un shell root dans le système d'exploitation invité*

Définir le mot de passe du super-utilisateur :

    passwd  # root49

Autoriser la connexion extérieure :

    echo 'pts/1' >> /etc/securetty

Se déconnecter de l'OS invité (Ctrl-d)

#### Activer et démarrer la nouvelle machine invitée

Activation et lancement du conteneur debian nspbullseye

```shell
sudo machinectl enable nspbullseye
sudo machinectl start nspbullseye
```

Cela génére un lien symbolique avec le nom de la machine à partir d'un modèle **.nspawn** qui se trouve ici : `/usr/lib/systemd/system/systemd-nspawn@.service`   
Le lien `$MACHINE_NAME.nspawn` sera ajouté ici : `/etc/systemd/system/machines.target.wants/systemd-nspawn@$MACHINE_NAME.service`

Dans notre cas `/etc/systemd/system/machines.target.wants/systemd-nspawn@nspbullseye.service`

```
$ ls -la /etc/systemd/system/machines.target.wants/systemd-nspawn@nspbullseye.service
lrwxrwxrwx 1 root root 47 20 mai   14:50 /etc/systemd/system/machines.target.wants/systemd-nspawn@nspbullseye.service -> /usr/lib/systemd/system/systemd-nspawn@.service
```

#### Paramétrage réseau MacVLAN coté hôte

*MacVLAN est un module du noyau Linux. Sa fonction est de permettre la configuration de plusieurs adresses MAC sur la même carte réseau physique, c'est-à-dire plusieurs interfaces. Chaque interface peut être configurée avec sa propre IP.*

On veut utiliser une connexion réseau de type **macvlan** liée à l'interface **eno1** 
Créer le fichier

    sudo nano /etc/systemd/nspawn/nspbullseye.nspawn

```
[Network]
MACVLAN=eno1
```

## Conteneur Debian

![bullseye](debian11-logo.png){:height="30"} 

### Connexion hôte --> Conteneur

Se connecter au conteneur debian bullseye depuis l'hôte

    sudo machinectl login nspbullseye

Vous devez toujours vous connecter en tant que root (utilisateur inexistant à ce stade)

### Définir hostname

Editer /etc/hostname

    nano /etc/hostname # nspbulls

(utilisez un nom d'hôte différent de celui de votre ordinateur hôte, quelque chose qui permette de reconnaître cette machine invitée en dehors de votre machine hôte)

Modifier /etc/hosts

    nano /etc/hosts

Remplacez la première ligne par

```
#127.0.0.1 localhost <nouveau nom d'hôte ici>
127.0.0.1 localhost nspbulls
```

Mettre à jour le nom d'hôte sur la console :

```
# hostname <nouveau nom d'hôte>
hostname nspbulls
```

Vérification

    hostnamectl

```
   Static hostname: nspbulls
         Icon name: computer-container
           Chassis: container
        Machine ID: 700874f8300243e6bafdab5d68305a1c
           Boot ID: 30cb9689cac34a6791414cbdabda3904
    Virtualization: systemd-nspawn
  Operating System: Debian GNU/Linux 11 (bullseye)
            Kernel: Linux 6.3.2-arch1-1
      Architecture: x86-64
```

### Réseau invité statique

*Le conteneur va utiliser un réseau configuré avec systemd-networkd et une IP statique*  

#### Réseau systemd-networkd

Vérifiez si votre conteneur utilise systemd-networkd, vous pouvez vérifier si c'est le cas en exécutant la commande suivante

    systemctl status systemd-networkd

Si le service n'est pas activé et ne fonctionne pas, vous devez l'activer et le démarrer.

```shell
systemctl enable systemd-networkd.service
systemctl start systemd-networkd.service
```

#### Résolution des noms de domaine

Activer et démarrer le service systemd-resolved pour la résolution des noms de domaine 

```
systemctl enable systemd-resolved
systemctl start systemd-resolved
```

#### Adresse ipv4 statique

Si vous devez définir une adresse ipv4 statique pour l'interface macvlan.  
adresse de passerelle 192.168.0.254  
adresse ipv4 à 192.168.0.25/24  
Utiliser le contenu suivant pour le fichier de configuration du réseau

    nano /etc/systemd/network/mveno1.network

```
[Match]
Name=mv-eno1

[Network]
IPForward=yes
Address=192.168.0.25/24
Gateway=192.168.0.254
```

Après avoir créé le fichier, se déconnecter `Ctrl+Alt ]]]`  
redémarrer le container

```shell
sudo machinectl stop nspbullseye
sudo machinectl start nspbullseye
```

Se connecter

    sudo machinectl login nspbullseye

Vérification

    ip a

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: mv-eno1@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether ee:b8:50:73:d0:9f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.0.25/24 brd 192.168.0.255 scope global mv-eno1
       valid_lft forever preferred_lft forever
    inet6 fe80::ecb8:50ff:fe73:d09f/64 scope link 
       valid_lft forever preferred_lft forever
```

### Outils, utilisateur, SSH, dossiers

Modifier le mot de passe  root : `passwd`  

#### Outils 

Installer les outils

	apt install ssh nano sudo 
    apt install netcat-openbsd rsync curl tmux jq figlet git dnsutils wget tree iptables lsof rsync

#### Utilisateur

Création  Utilisateur nspyan

    useradd -m -d /home/nspyan/ -s /bin/bash nspyan
    passwd nspyan

Accès sudo

    echo "nspyan     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

Locales 

    timedatectl

```
               Local time: Mon 2023-05-22 18:09:35 CEST
           Universal time: Mon 2023-05-22 16:09:35 UTC
                 RTC time: n/a
                Time zone: Europe/Paris (CEST, +0200)
System clock synchronized: no
              NTP service: n/a
          RTC in local TZ: no
```

Motd

    rm /etc/motd && nano /etc/motd

```
     _       _     _               _  _                
  __| | ___ | |__ (_) __ _  _ _   / |/ |               
 / _` |/ -_)| '_ \| |/ _` || ' \  | || |               
 \__,_|\___||_.__/|_|\__,_||_||_| |_||_|               
                  _           _  _                     
  _ _   ___ _ __ | |__  _  _ | || | ___ ___  _  _  ___ 
 | ' \ (_-<| '_ \| '_ \| || || || |(_-</ -_)| || |/ -_)
 |_||_|/__/| .__/|_.__/ \_,_||_||_|/__/\___| \_, |\___|
           |_|                               |__/      

```


#### SSH clé et script

![ssh](ssh_logo.png){:height="50"}  
**connexion avec clé**  
<u>sur l'ordinateur de bureau</u>
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) nommé **nspawn-bulls** pour une liaison SSH avec le serveur KVM.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/nspawn-bulls

Envoyer la clé publique dans le conteneur nspbullseye

    ssh-copy-id -i ~/.ssh/nspawn-bulls.pub nspyan@192.168.0.25 

<u>sur le conteneur nspbullseye</u>
On se connecte au conteneur nspbullseye 

    ssh nspyan@192.168.0.25

Modifier la configuration serveur SSH  

    sudo nano /etc/ssh/sshd_config 

les paramètres à modifier

```conf
# Port 22
PasswordAuthentication no 
```

Relancer openSSH  

    sudo systemctl restart sshd

Accès depuis le poste distant avec la clé privée  après avoir ouvert le port

    ssh -i ~/.ssh/nspawn-bulls nspyan@192.168.0.25

#### Partage de dossier

Ajoute un montage bind de l'hôte dans le conteneur. Prend un chemin unique, une paire de deux chemins séparés par deux points, ou un triplet de deux chemins plus une chaîne d'options séparée par des deux points. Cette option peut être utilisée plusieurs fois pour configurer plusieurs montages bind. 

Modification du fichier de configuration `/etc/systemd/nspawn/lxcbulls.nspawn`  
Ajouter les lignes suivantes

```
[Files]
Bind=/home/yano/media:/home/nspyan/media
```

Relancer le container

```
sudo machinectl stop lxcbulls
sudo machinectl start lxcbulls
```

#### Historique des commandes bash

POUR tous les utilisateurs ,y compris root et avec touche SHIFT

```bash
echo "
# appel alphabétique commandes
shopt -s histappend
PROMPT_COMMAND='history -a'" | tee -a $HOME/.bashrc

echo "
# appel alphabétique commandes
shopt -s histappend
PROMPT_COMMAND='history -a'" | sudo tee -a /root/.bashrc

echo '
# AVEC la touche SHIFT
"\e[1;2A": history-search-backward
"\e[1;2B": history-search-forward
' | sudo tee -a /etc/inputrc
```

Redémarrer le terminal pour la prise en compte

### Générateur de site statique

#### Ruby

![](ruby-logo.png){:height="50"}  

Prérequis

    sudo apt install build-essential zlib1g-dev libtool libyaml-dev libssl-dev

[Installation Ruby (via rbenv) + Jekyll (générateur de site statique) sur Debian](/posts/Installation-Ruby-via-rbenv+Jekyll-sur-Debian/)

    rbenv install 3.2.2

```
To follow progress, use 'tail -f /tmp/ruby-build.20230522204847.115121.log' or pass --verbose
Installing ruby-3.2.2...
Installed ruby-3.2.2 to /home/nspyan//.rbenv/versions/3.2.2

NOTE: to activate this Ruby version as the new default, run: rbenv global 3.2.2
```

Activation

    rbenv global 3.2.2

#### Jekyll

![](jekyll-300x133.png){:height="50"}  

On va utiliser gem pour installer Bundler qui est un outil utilisé pour gérer les dépendances de Gem et Jekyll 

    gem install bundler

Installer jekyll

```shell
gem install jekyll  # Patienter quelques minutes
```

Version jekyll installée

    jekyll -v

jekyll 4.3.2

