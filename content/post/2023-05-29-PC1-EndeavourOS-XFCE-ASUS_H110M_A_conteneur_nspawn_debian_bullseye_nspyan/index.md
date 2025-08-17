+++
title = 'PC1 Ordinateur Bureau EndeavourOS xfce --> conteneur nspawn debian bullseye nspyan'
date = 2023-05-29 00:00:00 +0100
categories = ['systemd']
+++
*systemd-nspawn peut être utilisé pour exécuter une commande ou un système d'exploitation dans un espace de noms léger. Il est plus puissant que chroot car il virtualise entièrement la hiérarchie du système de fichiers, ainsi que l'arborescence des processus, les différents sous-systèmes IPC et le nom de l'hôte et du domaine.*

![](container.png){:width="150"}


## Création conteneur systemd-nspawn

![](minitour_ecran_clavier.png){:width="150"}  
[Description matériel mini tour PC1](/posts/Description_materiel_minitour_PC1/)

*Créer un environnement Debian bullseye avec un conteneur nommé nspyan dans une Mini tour PC1 EndeavourOS XFCE ([Archlinux --> Container systemd-nspawn](/posts/systemd-nspawn/))*  

### Prérequis

#### Installer les dépendances debian

Installez **debootstrap**, ainsi que **debian-archive-keyring** ou **ubuntu-keyring**, ou les deux, en fonction de la distribution choisie.

    yay -S debootstrap debian-archive-keyring

**Assurez-vous que le paquetage *dbus* ou *dbus-broker* est installé sur le système de conteneurs.**
{: .prompt-info }

    yay -Ss dbus

```
core/dbus 1.14.6-2 (304.7 KiB 911.4 KiB) (Installé)
```

### Container invité debian

![bullseye](debian11-logo.png){:height="30"} 


#### Installer le système d'exploitation invité debian

Créer un dossier pour les containers

    mkdir -p $HOME/virtuel/nspawn

Configurer l'environnements Debian 

```shell
sudo debootstrap --arch amd64  --include=systemd-container,systemd,ssh,locales,dbus bullseye  $HOME/virtuel/nspawn/nspbullseye https://deb.debian.org/debian
```

Le container nspbullseye est créé

```
I: Base system installed successfully.
```

#### Lien /var/lib/machines 

Lien /var/lib/machines avec $HOME/virtuel

    sudo rmdir /var/lib/machines
    sudo ln -s $HOME/virtuel/nspawn /var/lib/machines

#### Créer mot de passe root dans système d'exploitation invité

    sudo systemd-nspawn -D /var/lib/machines/nspbullseye -U --machine nspbullseye

*Cela permet d'obtenir un shell root dans le système d'exploitation invité*

Définir le mot de passe du super-utilisateur :

    passwd  # root49

Autoriser la connexion extérieure :

    echo 'pts/1' >> /etc/securetty

Se déconnecter de l'OS invité (Press Ctrl+AltGr-] three times within 1s to kill container.)

#### Démarrer la nouvelle machine invitée

Lancement du conteneur debian nspbullseye

```shell
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

On veut utiliser une connexion réseau de type **macvlan** liée à l'interface **enp0s31f6** 
Créer le fichier

    sudo mkdir -p /etc/systemd/nspawn/
    sudo nano /etc/systemd/nspawn/nspbullseye.nspawn

```
[Network]
MACVLAN=enp0s31f6
```

## Connexion hôte --> Conteneur

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
        Machine ID: 53fa5f7e23114069aab0959a5832ddb1
           Boot ID: 24512f52be2d49f89c71b4f8ccf7a0f8
    Virtualization: systemd-nspawn
  Operating System: Debian GNU/Linux 11 (bullseye)
            Kernel: Linux 6.3.4-arch1-1
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

<https://docs.fedoraproject.org/en-US/fedora-server/containerization/systemd-nspawn-setup/>  
<https://wiki.debian.org/nspawn>

Si vous devez définir une adresse ipv4 statique pour l'interface macvlan.  
adresse de passerelle 192.168.0.254  
adresse ipv4 à 192.168.0.210/24  
Utiliser le contenu suivant pour le fichier de configuration du réseau

    nano /etc/systemd/network/mvenp0s31f6.network

```
[Match]
Name=mv-enp0s31f6

[Link]
ARP=True

[Network]
DHCP=no

# IPv4 static configuration, no DHCP configured!
Address=192.168.0.210/24
Gateway=192.168.0.254
```

On peut modifier directement le fichier `/virtuel/nspawn/nspbullseye/etc/systemd/network/mvenp0s31f6.network`

Après avoir créé le fichier, vous devrez redémarrer le service systemd-networkd dans le container

    systemctl restart systemd-networkd

Après avoir créé le fichier, se déconnecter `Ctrl+Alt ]]]`  

redémarrer le container

```shell
sudo systemctl restart nspdebian
```

Vérification

    machinectl

```
MACHINE     CLASS     SERVICE        OS     VERSION ADDRESSES     
nspbullseye container systemd-nspawn debian 11      192.168.0.210…

1 machines listed.
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
2: mv-enp0s31f6@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 4a:db:85:cd:5f:df brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.0.210/24 brd 192.168.0.255 scope global mv-enp0s31f6
       valid_lft forever preferred_lft forever
    inet6 2a01:e0a:9c8:2080:48db:85ff:fecd:5fdf/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 86388sec preferred_lft 86388sec
    inet6 fe80::48db:85ff:fecd:5fdf/64 scope link 
       valid_lft forever preferred_lft forever
```

### Outils, utilisateur, SSH, dossiers

Modifier le mot de passe  root : `passwd`  

#### Outils 

Installer les outils

```shell
apt install ssh nano sudo netcat-openbsd rsync curl tmux jq figlet git \
dnsutils wget tree iptables lsof rsync net-tools
```

#### Utilisateur

Création  Utilisateur nspyan

    useradd -m -d /home/nspyan/ -s /bin/bash nspyan
    passwd nspyan

Accès sudo

    echo "nspyan     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/20-nspyan
    chmod 440 /etc/sudoers.d/20-nspyan

Locales 

    timedatectl

```
               Local time: Sat 2023-05-27 11:56:54 CEST
           Universal time: Sat 2023-05-27 09:56:54 UTC
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

Se reconnecter en utilisateur nspyan  
Créer les dossiers 

    mkdir $HOME/{scripts,media}

#### Bind, Partage de dossier

Ajouter un montage bind ,ouvrir le fichier `/etc/systemd/nspawn/nspbullseye.nspawn`

Et ajouter en fin de fichier dans la rubrique `[Files]` , les dossiers avec la syntaxe  
`Bind=/chemin/local:/chemin/container`

Il faut également avoir `PrivateUsers=no` pour accéder aux dossiers partagés avec les mêmes droits `nsypyan nspyan` au lieu de `nobody nogroup` (accès lecture uniquement)

```
[Exec]
Boot=on
PrivateUsers=no

[Network]
MACVLAN=enp0s31f6

[Files]
Bind=/srv/media:/home/nspyan/media
Bind=/home/yann/scripts:/home/nspyan/scripts
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

#### SSH clé

![ssh](ssh_logo.png){:height="50"}  
**connexion avec clé**  
<u>ordinateur de bureau</u>
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) nommé **nspbullseye** pour une liaison SSH avec le serveur KVM.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/nspbullseye

Envoyer la clé publique dans le conteneur nspbullseye

    ssh-copy-id -i ~/.ssh/nspbullseye.pub nspyan@192.168.0.210 

<u>conteneur nspbullseye</u>
On se connecte au conteneur nspbullseye 

    ssh nspyan@192.168.0.210

Modifier la configuration serveur SSH  

    sudo nano /etc/ssh/sshd_config 

les paramètres à modifier

```conf
Port 55210
PasswordAuthentication no 
```

Relancer openSSH  

    sudo systemctl restart sshd

On est bien en écoute sur le port 55210

```
nspyan@nspbulls:~$ sudo netstat -plunt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      39/systemd-resolved 
tcp        0      0 0.0.0.0:5355            0.0.0.0:*               LISTEN      39/systemd-resolved 
tcp        0      0 0.0.0.0:55210           0.0.0.0:*               LISTEN      182/sshd: /usr/sbin 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      69/nginx: master pr 
tcp6       0      0 :::5355                 :::*                    LISTEN      39/systemd-resolved 
tcp6       0      0 :::55210                :::*                    LISTEN      182/sshd: /usr/sbin 
tcp6       0      0 :::80                   :::*                    LISTEN      69/nginx: master pr 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           39/systemd-resolved 
udp        0      0 0.0.0.0:5355            0.0.0.0:*                           39/systemd-resolved 
udp6       0      0 :::5355                 :::*                                39/systemd-resolved 
```

Accès depuis le poste distant avec la clé privée  après avoir ouvert le port

    ssh -p 55210 -i ~/.ssh/nspbullseye nspyan@192.168.0.210

```
Linux nspbulls 6.3.4-arch1-1 #1 SMP PREEMPT_DYNAMIC Wed, 24 May 2023 17:44:00 +0000 x86_64
     _       _     _               _  _                
  __| | ___ | |__ (_) __ _  _ _   / |/ |               
 / _` |/ -_)| '_ \| |/ _` || ' \  | || |               
 \__,_|\___||_.__/|_|\__,_||_||_| |_||_|               
                  _           _  _                     
  _ _   ___ _ __ | |__  _  _ | || | ___ ___  _  _  ___ 
 | ' \ (_-<| '_ \| '_ \| || || || |(_-</ -_)| || |/ -_)
 |_||_|/__/| .__/|_.__/ \_,_||_||_|/__/\___| \_, |\___|
           |_|                               |__/      
Last login: Mon May 29 09:58:02 2023
nspyan@nspbulls:~$ 
```

### Démarrage auto container

Le fichier de configuration hôte

    /etc/systemd/nspawn/nspbullseye.nspawn

```
[Exec]
Boot=on
PrivateUsers=no

[Network]
MACVLAN=enp0s31f6

[Files]
Bind=/srv/media:/home/nspyan/media
Bind=/home/yann/scripts:/home/nspyan/scripts
```

Activation conteneur debian

    machinectl enable nspbullseye

Status et redémarrage

    machinectl status nspbullseye
    sudo systemctl restart systemd-nspawn@nspbullseye

### Connexion réseau entre host et container

[Docker Macvlan network inside container is not reaching to its own host](https://stackoverflow.com/questions/49600665/docker-macvlan-network-inside-container-is-not-reaching-to-its-own-host)

Avec un conteneur attaché à un réseau macvlan, vous constaterez que s'il peut contacter sans problème d'autres systèmes sur votre réseau local, <u>le conteneur ne pourra pas se connecter à votre hôte (et votre hôte ne pourra pas se connecter à votre conteneur)</u>. Il s'agit d'une limitation des interfaces macvlan :   
sans support spécial d'un commutateur réseau, votre hôte n'est pas en mesure d'envoyer des paquets à ses propres interfaces macvlan.

Heureusement, il existe une solution à ce problème : vous pouvez créer une autre interface macvlan sur votre hôte, et l'utiliser pour communiquer avec les conteneurs sur le réseau macvlan.

Nous créons une nouvelle interface macvlan sur l'hôte nommé **net-nsp** :

    ip link add net-nsp link enp0s31f6 type macvlan mode bridge

Nous devons maintenant configurer l'interface avec l'adresse que nous avons choisie et l'activer :

    ip addr add 192.168.0.215/27 dev net-nsp
    ip link set net-nsp up

La dernière chose à faire est d'indiquer à notre hôte d'utiliser cette interface pour communiquer avec les conteneurs. C'est relativement facile car nous avons restreint nos conteneurs à un sous-ensemble CIDR particulier du réseau local ; il suffit d'ajouter une route à cette plage comme ceci :

    ip route add 192.168.0.210/27 dev net-nsp

Avec cette route en place, votre hôte utilisera automatiquement l'interface net-nsp lorsqu'il communiquera avec des conteneurs sur le réseau

#### Connexion réseau persistante

On va créer un fichier de configuration NetworkManager

    /etc/NetworkManager/system-connections/net-nsp.nmconnection 

```
[connection]
id=net-nsp
uuid=b3944aaf-35b9-457b-8570-c938957f5501
type=macvlan
interface-name=net-nsp
timestamp=1685337859

[macvlan]
mode=2
parent=enp0s31f6
tap=true

[ipv4]
address1=192.168.0.215/32
method=manual
route1=192.168.0.210/32

[ipv6]
addr-gen-mode=default
method=auto

[proxy]
```

Les commandes nmcli ci dessous sont à tester si on ne veut pas utiliser le fichier de configuration

```
nmcli connection add type macvlan dev enp0s31f6 mode bridge tap yes ifname net-nsp con-name net-nsp ip4 192.168.0.215
nmcli connection modify example +ipv4.routes "192.168.0.210/24 192.168.0.215"
```


## Générateur de site statique

### Ruby Jekyll

#### Ruby

![](ruby-logo.png){:height="50"}  

Prérequis

    sudo apt install build-essential zlib1g-dev libtool libyaml-dev libssl-dev

[Installation Ruby (via rbenv) + Jekyll (générateur de site statique) sur Debian](/posts/Installation-Ruby-via-rbenv+Jekyll-sur-Debian/)

Récapitulatif des instructions

```
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
~/.rbenv/bin/rbenv init
echo 'eval "$(/home/nspyan/.rbenv/bin/rbenv init - bash)"' >> ~/.bashrc
source ~/.bashrc
# versions stables
rbenv install -l
```

`Ne pas installer de version ruby 3.2.x , incompatible avec Jekyll`

On installe la 3.0.6

    rbenv install 3.0.6

```
To follow progress, use 'tail -f /tmp/ruby-build.20230522204847.115121.log' or pass --verbose
Installing ruby-3.0.6...
Installed ruby-3.0.6 to /home/nspyan//.rbenv/versions/3.0.6

NOTE: to activate this Ruby version as the new default, run: rbenv global 3.0.6
```

Activation

    rbenv global 3.0.6

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

