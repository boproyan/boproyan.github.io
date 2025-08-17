+++
title = 'EndeavourOS Dell Latitude e6230 --> conteneur nspawn debian bookworm nspyan'
date = 2023-09-02 00:00:00 +0100
categories = ['systemd']
+++
*systemd-nspawn peut être utilisé pour exécuter une commande ou un système d'exploitation dans un espace de noms léger. Il est plus puissant que chroot car il virtualise entièrement la hiérarchie du système de fichiers, ainsi que l'arborescence des processus, les différents sous-systèmes IPC et le nom de l'hôte et du domaine.*

![](container.png){:width="150"}


## Portable Latitude e6230

![Dell Latitude E6230](dell-latitude-e6230.png){:width="150"}  

*Créer un environnement Debian bookworm avec un conteneur nommé nspyan dans un portable Dell Latitude e6230 EndeavourOS XFCE ([Archlinux --> Container systemd-nspawn](/posts/systemd-nspawn/))*  

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
sudo debootstrap --arch amd64  --include=systemd-container,systemd,ssh,locales,dbus bookworm  $HOME/virtuel/nspbookworm https://deb.debian.org/debian
```

Le container nspbookworm est créé

```
I: Base system installed successfully.
```

#### Lien /var/lib/machines 

Lien /var/lib/machines avec $HOME/virtuel à créer si non existant

    sudo rmdir /var/lib/machines
    sudo ln -s $HOME/virtuel/ /var/lib/machines

#### Créer mot de passe root dans système d'exploitation invité

    sudo systemd-nspawn -D /var/lib/machines/nspbookworm -U --machine nspbookworm

*Cela permet d'obtenir un shell root dans le système d'exploitation invité*

Définir le mot de passe du super-utilisateur :

    passwd  # root49

Autoriser la connexion extérieure :

    echo 'pts/1' >> /etc/securetty

Se déconnecter de l'OS invité (Ctrl-d)

#### Activer et démarrer la nouvelle machine invitée

Activation et lancement du conteneur debian nspbookworm

```shell
sudo machinectl enable nspbookworm
sudo machinectl start nspbookworm
```

Cela génére un lien symbolique avec le nom de la machine à partir d'un modèle **.nspawn** qui se trouve ici : `/usr/lib/systemd/system/systemd-nspawn@.service`   
Le lien `$MACHINE_NAME.nspawn` sera ajouté ici : `/etc/systemd/system/machines.target.wants/systemd-nspawn@$MACHINE_NAME.service`

Dans notre cas `/etc/systemd/system/machines.target.wants/systemd-nspawn@nspbookworm.service`

```
$ ls -la /etc/systemd/system/machines.target.wants/systemd-nspawn@nspbookworm.service
lrwxrwxrwx 1 root root 47 20 mai   14:50 /etc/systemd/system/machines.target.wants/systemd-nspawn@nspbookworm.service -> /usr/lib/systemd/system/systemd-nspawn@.service
```

#### Paramétrage réseau MacVLAN coté hôte

*MacVLAN est un module du noyau Linux. Sa fonction est de permettre la configuration de plusieurs adresses MAC sur la même carte réseau physique, c'est-à-dire plusieurs interfaces. Chaque interface peut être configurée avec sa propre IP.*

https://web.archive.org/web/20190917181922/http://noyaudolive.net/2012/05/09/lxc-and-macvlan-host-to-guest-connection/

On veut utiliser une connexion réseau de type **macvlan** liée à l'interface **wlan0** 
Créer le fichier

    sudo nano /etc/systemd/nspawn/nspbookworm.nspawn

```
[Network]
MACVLAN=wlan0
```

## Conteneur Debian

![bookworm](debian12-logo.png){:height="30"} 

### Connexion hôte --> Conteneur

Se connecter au conteneur debian bookworm depuis l'hôte

    sudo machinectl login nspbookworm

Vous devez toujours vous connecter en tant que root (utilisateur inexistant à ce stade)

### Définir hostname

Editer /etc/hostname

    nano /etc/hostname # nspbook

nspbook
(utilisez un nom d'hôte différent de celui de votre ordinateur hôte, quelque chose qui permette de reconnaître cette machine invitée en dehors de votre machine hôte)

Modifier /etc/hosts

    nano /etc/hosts

Remplacez la première ligne par

```
#127.0.0.1 localhost <nouveau nom d'hôte ici>
127.0.0.1 localhost nspbook
```

Mettre à jour le nom d'hôte sur la console :

```
# hostname <nouveau nom d'hôte>
hostname nspbook
```

Vérification

    hostnamectl

```
 Static hostname: nspbook
       Icon name: computer-container
         Chassis: container ☐
      Machine ID: 0b28565cfda6436eab3a96051917a37c
         Boot ID: fb42a5bb82934d9bb75e4a6e0a11a691
  Virtualization: systemd-nspawn
Operating System: Debian GNU/Linux 12 (bookworm)  
          Kernel: Linux 6.4.11-arch2-1
    Architecture: x86-64
```

Après avoir créé le fichier, se déconnecter `Ctrl+Alt ]]]`  
redémarrer le container

```shell
sudo machinectl stop nspbookworm
sudo machinectl start nspbookworm
```

### Utiliser la mise en réseau de l'hôte

Pour <u>désactiver la mise en réseau privée</u> et la création d'un lien Ethernet virtuel utilisé par les conteneurs démarrés avec `machinectl`, ajoutez un fichier .nspawn avec l'option suivante :

/etc/systemd/nspawn/nom-du-conteneur.nspawn

```
[Network]
VirtualEthernet=no
```

Cette option remplacera l'option `-n/--network-veth` utilisée dans **systemd-nspawn@.service** et les conteneurs nouvellement démarrés utiliseront le mode réseau de l'hôte.

### Outils, utilisateur 

Modifier le mot de passe  root : `passwd`  

#### Outils 

Installer les outils

	apt install sudo 
    apt install netcat-openbsd rsync curl tmux jq figlet git

#### Utilisateur

Création  Utilisateur nspyan mp nspyan49

    useradd -m -d /home/nspyan/ -s /bin/bash nspyan
    passwd nspyan

Accès sudo

    echo "nspyan     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

Locales 

    timedatectl

```
               Local time: Thu 2023-08-24 10:47:43 CEST
           Universal time: Thu 2023-08-24 08:47:43 UTC
                 RTC time: n/a
                Time zone: Europe/Paris (CEST, +0200)
System clock synchronized: no
              NTP service: n/a
          RTC in local TZ: no
```

#### Partage de dossier

Ajoute un montage bind de l'hôte dans le conteneur. Prend un chemin unique, une paire de deux chemins séparés par deux points, ou un triplet de deux chemins plus une chaîne d'options séparée par des deux points. Cette option peut être utilisée plusieurs fois pour configurer plusieurs montages bind. 

Modification du fichier de configuration `/etc/systemd/nspawn/nspbookworm.nspawn`  
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

