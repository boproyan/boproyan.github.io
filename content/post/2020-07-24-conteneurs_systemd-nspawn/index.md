+++
title = 'Mise en place de conteneurs systemd-nspawn'
date = 2020-07-24 00:00:00 +0100
categories = ['virtuel']
+++
![](Linux_Containers_logo.png){:width="100"}

## Conteneurs systemd-nspawn

*Depuis un certain temps déjà, les conteneurs font l'objet d'un grand intérêt. Souvent considérés comme des "VM light" (virtual machine light), ils permettent aux utilisateurs d'exécuter des services dans des environnements dédiés, en les isolant les uns des autres et du système hôte.  
Comment faire fonctionner un simple conteneur en utilisant les outils fournis par systemd.*

### Comment les conteneurs fonctionnent-ils ?

La meilleure façon d'imaginer ce que font les conteneurs est de penser en termes d'espaces de noms. Il existe plusieurs espaces de noms dans votre système auxquels vous pouvez restreindre l'accès. Examinons certains d'entre eux :  

* **Système de fichiers**. Une des étapes les plus importantes pour isoler un service est de restreindre l'accès au système de fichiers. Les conteneurs fournissent une fonctionnalité de type **"chroot"**, vous permettant de spécifier une racine de système de fichiers arbitraire. Cela signifie que vous pouvez décider quels programmes vous voulez installer dans votre conteneur de façon totalement indépendante du système hôte. Cependant, cela signifie également que vous devez mettre en place une seconde arborescence de système d'exploitation pour le conteneur contenant tous les outils dont un système d'exploitation a besoin pour fonctionner. Mais c'est facile, comme nous le verrons plus tard. En fin de compte, vous aurez une hiérarchie de système de fichiers Unix typique à l'intérieur de votre répertoire racine spécifié, contenant /bin, /etc, /home, /usr et ainsi de suite.
* **Les ID de processus (PID)**. Lorsque vous exécutez `ps -ef`, vous voyez chaque processus s'exécuter sur le système, avec leurs PID uniques. Cependant, nous ne voulons pas que quelque chose s'exécute dans le conteneur pour voir les processus de l'hôte ou d'autres conteneurs. En outre, nous voulons qu'un processus dans le conteneur puisse avoir par exemple le PID 1 (init), même si nous avons déjà un PID 1 sur l'hôte.
* **Les ID utilisateur (UID)**. Chaque utilisateur d'un système possède un ID utilisateur. L'utilisateur root a généralement l'ID 0, alors que les ID utilisateur normaux commencent à 1000. L'espacement des noms d'utilisateur est surtout une précaution ; si quelqu'un est en mesure de pénétrer dans le système hôte à partir de votre conteneur, il n'aura pas l'UID d'un utilisateur existant sur l'hôte. Sans espacement des noms d'utilisateur, par exemple, l'utilisateur root dans un conteneur apparaîtra également dans le système hôte sous l'UID 0. L'activation de cette fonctionnalité ajoute un nombre arbitraire élevé à tous les UID à l'intérieur d'un conteneur, de sorte que l'utilisateur root du conteneur (apparaissant comme UID 0 dans le conteneur) aura en fait l'UID 64789232 dans le système hôte (par exemple).
* **Interfaces réseau**. L'exécution de `ip addr` sur l'hôte vous montre toutes les interfaces réseau disponibles sur votre machine. Exécuter la même commande dans un conteneur vous donnera exactement le même résultat, ce qui signifie que le conteneur a les mêmes privilèges d'accès au réseau que l'hôte. Pour éviter cela, nous pouvons restreindre l'accès du conteneur aux interfaces réseau de l'hôte et ajouter un pont pour le conteneur afin de lui permettre de continuer à accéder à l'internet et au réseau local.

>Tous les espaces de noms sont gérés par le noyau Linux. Il est possible d'activer ou de désactiver le support de ces espaces de noms au moment de la compilation du noyau. Si certains d'entre eux ne fonctionnent pas pour vous, peut-être ont-ils été désactivés ou votre noyau est trop vieux.

>**Note sur les conteneurs à traitement unique**  
Certaines personnes de la communauté Docker font la promotion du modèle "un processus par conteneur", ce qui signifie que chaque conteneur que vous mettez en place est destiné à exécuter exactement un processus à l'intérieur de celui-ci. Cependant, comme le suggère également cet article, il est bon d'avoir au moins un système d'initialisation minimal dans votre conteneur. Le problème est que les processus peuvent bifurquer() des processus enfants, et lorsqu'un parent n'attend pas correctement() ses enfants, le processus avec le PID 1 (le processus init) devient responsable du devoir de nettoyage. Mais s'il n'y a pas de processus init, ces processus enfants orphelins deviendront des zombies. Certains processus s'appuient sur l'init comme faucheur de zombies, par exemple les démons lors de la double bifurcation. Dans ce guide, nous aurons systématisé le système init à l'intérieur de notre conteneur.

## Créer un conteneur étape par étape

Je montrerai comment créer des conteneurs sur Debian, mais les étapes devraient être similaires pour les autres distributions Linux. Notez que vous avez besoin des privilèges de root pour exécuter les commandes des sections suivantes.

### Installation des paquets nécessaires

Nous avons besoin de quelques paquets pour installer et faire fonctionner le conteneur. 

    # apt install debootstrap systemd-container bridge-utils

*debootstrap* installe un système Debian minimal dans un répertoire personnalisé.  
*systemd-container* contient les outils systemd pour exécuter et configurer les conteneurs.  
*bridge-utils* permet de configurer facilement un pont pour donner au conteneur un accès au réseau.

### Configurer l'arborescence du système d'exploitation

Tout d'abord, nous devons mettre en place une arborescence d'OS dans un répertoire vide, qui servira de répertoire racine au conteneur. Pour ce faire, nous utiliserons *debootstrap*.  
systemd s'attend à ce que les conteneurs soient situés dans le répertoire **/var/lib/machines**. Ils peuvent se trouver ailleurs, mais certains outils ne reconnaîtront pas automatiquement le conteneur. Mettons en place un conteneur nommé *helloworld* :

    # mkdir -p /var/lib/machines/helloworld
    # debootstrap stable /var/lib/machines/helloworld http://deb.debian.org/debian/

Cela permettra d'installer tout ce qui est nécessaire pour faire fonctionner Debian dans notre nouveau conteneur (cela peut prendre un certain temps).  
Vous devez également vous assurer que seul root peut accéder au répertoire des machines :

    # chown root:root /var/lib/machines
    # chmod 700 /var/lib/machines

### Un aperçu des commandes Systemd de conteneurs 

Maintenant que nous avons mis en place le système d'exploitation, c'est le bon moment pour introduire les commandes que vous utiliserez pour contrôler votre conteneur.  
`systemd-nspawn` est utilisé pour démarrer directement un conteneur. Par défaut, il se contente d'engendrer un shell racine dans l'environnement du conteneur. L'utilisation du commutateur `-b` permet de démarrer le conteneur et de vous donner un shell de connexion. Cependant, dès que la commande se termine, le conteneur s'éteint également. Aussi, lorsque vous voulez utiliser l'espace de nommage utilisateur, je recommande fortement de toujours le lancer avec le commutateur `-U`, qui permet l'espace de nommage utilisateur. La première fois que vous faites cela, systemd ajustera toutes les permissions de fichiers à l'intérieur du conteneur, et désactiver l'espace de nommage utilisateur plus tard peut conduire à des erreurs très bizarres (et à des fichiers appartenant à nobody:nogroup). Enfin, avec le commutateur `-M nom_machine`, vous spécifiez le conteneur à lancer.  
`machinectl` est très similaire au `systemctl` de systemd, mais est utilisé spécifiquement pour travailler avec des conteneurs. En fait, les conteneurs peuvent être lancés, arrêtés et activés comme n'importe quel autre service systemd. Le fichier modèle du service se trouve dans **/lib/systemd/system/systemd-nspawn@.service**.  
Il vaut la peine d'y jeter un coup d'oeil, en particulier l'exécutable spécifié :

```
[Service]
ExecStart=/usr/bin/systemd-nspawn --quiet --keep-unit --boot --link-journal=try-guest --network-veth -U --settings=override --machine=%i
```

Comme vous pouvez le voir, il y a déjà un certain nombre d'arguments de ligne de commande spécifiés par défaut lorsque vous utilisez `machinectl` pour démarrer un conteneur. L'un d'entre eux est `--boot` (comme `-b`), de sorte que le conteneur démarre et ne se contente pas d'engendrer un shell root. `--network-veth` crée une interface ethernet virtuelle (sans passerelle) dans le conteneur. Notez que `-U` est spécifié, donc l'espacement des noms des utilisateurs sera activé.  
**/etc/systemd/nspawn** est un répertoire qui peut contenir des fichiers de configuration pour les paramètres spécifiques au conteneur. Tous les paramètres peuvent également être spécifiés via la ligne de commande `systemd-nspawn`, mais cela vous évite de les taper à chaque fois. En outre, les fichiers de configuration sont également honorés lorsque vous utilisez `machinectl` pour démarrer les conteneurs.  
Si le répertoire n'existe pas, il suffit de le créer en utilisant

    # mkdir /etc/systemd/nspawn

### Configuration du conteneur

Maintenant, nous avons mis en place une arborescence de systèmes d'exploitation et vous avez des connaissances de base sur les outils de Systemd.   
La première chose que nous faisons est de définir un mot de passe root, nous obtiendrons un shell root sans avoir à démarrer le conteneur :

    # systemd-nspawn -UM helloworld

Votre invite de commande devrait maintenant ressembler à ceci (ou du moins à un élément similaire) :

    root@helloworld:~# _

Définissez maintenant le mot de passe en utilisant

    # passwd

Saisissez votre mot de passe root et quittez à nouveau le conteneur :
    # exit

Voyons maintenant si nous pouvons démarrer le tout :

    # systemd-nspawn -UbM helloworld

Vous devriez voir les messages habituels de démarrage du système. Si tout fonctionne, après quelques secondes, vous devriez voir un shell de connexion. Connectez-vous avec root et le mot de passe que vous venez de définir.  
Ensuite, je vous recommande d'installer dbus. Il est nécessaire si vous voulez vous connecter à des conteneurs en cours d'exécution en utilisant `machinectl` sur l'hôte.

    # apt install dbus

Pour l'instant, le conteneur devrait voir les mêmes interfaces réseau que l'hôte. Vous pouvez utiliser `ip addr` pour vérifier cela. Nous mettrons en place un réseau Ethernet virtuel dans la prochaine section.   
Cependant, nous pouvons déjà dire au conteneur d'afficher automatiquement l'interface ethernet virtuelle :

    # echo 'auto host0' >> /etc/network/interfaces
    # echo 'iface host0 inet dhcp' >> /etc/network/interfaces

Vous devez également changer le nom d'hôte du conteneur, sinon il aura le même nom que l'hôte. Je l'appellerai **helloworld**, mais vous pouvez utiliser ce que vous voulez :

    # echo 'helloworld' > /etc/hostname

Bon, sortons du conteneur pour l'instant :

    # poweroff

### Configuration de l'hôte

Maintenant, essayons de mettre en place un réseau pour le conteneur. Pour y parvenir, nous devons d'abord configurer le système hôte de manière appropriée.

Dans ce guide, nous allons mettre en place un pont réseau sur l'hôte et connecter l'interface ethernet virtuelle du conteneur au pont. Cela signifie que l'hôte et le conteneur seront visibles sur le réseau en tant que deux machines différentes. Si vous connectez votre ordinateur à un routeur, celui-ci attribuera une adresse IP distincte au conteneur.

>Notez que cette méthode ne fonctionnera probablement pas pour vous si vous êtes sur un serveur avec une adresse IP statique. Dans ce cas, vous devrez mettre en place une NAT sur votre hôte ou une sorte de proxy, qui ne sera pas traitée ici.

Tout d'abord, disons à systemd que notre conteneur utilisera désormais un pont réseau. Cette fonction est prise en charge directement par systemd ; il se chargera de connecter l'ethernet virtuel du conteneur à l'interface de la passerelle de l'hôte au démarrage du conteneur. Pour configurer cela, ouvrez **/etc/systemd/nspawn/helloworld.nspawn** avec votre éditeur de texte préféré et ajoutez les lignes suivantes :

    sudo nano /etc/systemd/nspawn/helloworld.nspawn

```
[Network]
VirtualEthernet=yes
Bridge=br0
```

Cela demandera à systemd de connecter l'ethernet virtuel *host0* du conteneur à l'interface réseau *br0* de l'hôte.

Nous avons maintenant une connexion entre le conteneur et l'hôte, mais nous devons encore créer et configurer le pont *br0* pour que le conteneur puisse réellement accéder à l'internet.   Assurez-vous que le transfert de paquets IPv4 est activé sur l'hôte : ouvrez le fichier **/etc/sysctl.conf** avec votre éditeur de texte préféré et cherchez **net.ipv4.ip_forward**. Assurez-vous qu'il est défini sur 1 et qu'il n'est pas commenté :

    net.ipv4.ip_forward=1

Dans la suite de ce guide, je suppose que votre interface réseau primaire sur l'hôte est appelée *eth0*. Si ce n'est pas le cas, remplacez *eth0* par le nom correct en conséquence.

Ouvrez maintenant **/etc/network/interfaces** et ajoutez les lignes suivantes :

```
auto br0
iface br0 inet dhcp
    bridge_ports eth0
```

Il devrait déjà y avoir une définition d'interface pour eth0 dans ce fichier. Assurez-vous qu'elle est réglée sur manuel en changeant sa définition en :

    iface eth0 inet manual

>Notez que si vous avez installé un gestionnaire de réseau ou quelque chose de similaire, cela peut interférer avec la configuration de notre réseau. Vous devrez probablement le désactiver ou configurer le gestionnaire de réseau pour qu'il ne touche pas les interfaces que nous utilisons ici.

Bien, maintenant, faites apparaître notre pont :

    # ifdown eth0
    # ifup br0

Utilisez l'adr ip pour voir si tout a fonctionné comme prévu. Le résultat doit être similaire à celui-ci (c'est moi qui souligne) :

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br0 state UP group default qlen 1000
    link/ether 08:00:27:3e:cf:c0 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global enp0s3
       valid_lft forever preferred_lft forever
3: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:3e:cf:c0 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe3e:cfc0/64 scope link 
       valid_lft forever preferred_lft forever
```

D'accord, nous devrions maintenant avoir un accès à Internet à la fois sur l'hôte et sur le conteneur. Vérifions cela :

```
$ ping google.com -c 4
# systemd-nspawn -UbM helloworld
$ ping google.com -c 4
```

Les deux pings devraient fonctionner. Sortons à nouveau du conteneur :

    # poweroff

### Conteneurs en marche en arrière-plan

Nous sommes maintenant en mesure de faire fonctionner des conteneurs et leur avons donné un accès à Internet via un pont réseau.  
Si vous prévoyez d'utiliser le conteneur, par exemple pour faire fonctionner un serveur web isolé à l'intérieur, vous souhaitez très probablement que le conteneur fonctionne en arrière-plan à tout moment, et aussi qu'il démarre automatiquement chaque fois que la machine hôte est (re)démarrée.

Comme je l'ai déjà mentionné dans cet article, les conteneurs systemd fonctionnent comme les autres services systemd, ce qui signifie que nous pouvons les démarrer, les arrêter, les activer ou les désactiver.

Pour démarrer un conteneur en arrière-plan :

    # machinectl start helloworld

Pour obtenir des informations sur le statut d'un conteneur en cours d'utilisation :

    # machinectl status helloworld

Pour obtenir un shell de connexion sur un conteneur en cours d'exécution (nécessite *dbus* sur l'hôte et le conteneur) :

    # machinectl login helloworld

Vous pouvez quitter la session (comme systemd vous en informe) en appuyant sur `Ctrl + Alt Gr + ]` 3 fois en une seconde.

Pour fermer un conteneur en cours d'exécution :

    # machinectl stop helloworld

Pour configurer un conteneur afin qu'il démarre à chaque fois que le système démarre :

    # machinectl enable helloworld

... et de désactiver à nouveau l'autodémarrage :

    # machinectl disable helloworld

### Commandes d'aide et liens

*    `man systemd-nspawn` pour les options de démarrage des conteneurs
*    `man machinectl` pour l'interaction hôte-conteneur et la gestion des conteneurs
*    `man systemd.nspawn` pour les options et la syntaxe du fichier de configuration
*    `man bridge-utils-interfaces` pour plus d'informations sur la configuration des ponts
* [Une présentation de systemd-nspawn](https://www.blog-libre.org/2020/04/17/une-presentation-de-systemd-nspawn/)
* [systemd-nspawn (archwiki)](https://wiki.archlinux.org/index.php/Systemd-nspawn)
* [Setting up containers with systemd-nspawn](https://medium.com/@huljar/setting-up-containers-with-systemd-nspawn-b719cff0fb8d)

## machinectl

### Une Debian buster avec machinectl

L’emplacement (**/var/lib/machines**) est primordial sans quoi machinectl ne pourra pas gérer le conteneur.

```
sudo apt install --no-install-recommends debootstrap systemd-container debian-archive-keyring
sudo debootstrap --arch amd64 --include=systemd-container --components=main,contrib,non-free buster /var/lib/machines/buster http://deb.debian.org/debian/
sudo machinectl clone buster buster1
screen -dmS buster1 sudo systemd-nspawn -bM buster1 # Un petit hack nettement plus simple/rapide que configurer au propre systemd-nspawn@.service pour avoir du réseau dans le conteneur avec machinectl start
sudo machinectl shell buster1
```

[machinectl](https://www.freedesktop.org/software/systemd/man/machinectl.html) va permettre de gérer les conteneurs et machines virtuelles (VM), voici les principales commandes utiles.

* `machinectl` ou `machinectl list` # Lister les conteneurs actifs (running/online)
* `machinectl list-images` # Montrer une liste des images des conteneurs présentes dans **/var/lib/machines/**
* `sudo machinectl start buster1` # Démarrer le conteneur buster1
* `sudo machinectl poweroff buster1` ou `sudo machinectl stop buster1` # Éteindre le conteneur buster1 proprement (shut down cleanly), stop est un alias de poweroff
* `sudo machinectl terminate buster1` # Éteindre le conteneur buster1 immédiatemment (immediately terminates a virtual machine or container, without cleanly shutting it down)
* `sudo machinectl clone buster buster1` # Cloner le conteneur buster en buster1
* `sudo machinectl remove buster1` # Supprimer le conteneur buster1
* `sudo machinectl clean --all` # Supprimer toutes les images des conteneurs
* `sudo machinectl login buster1` # Open an interactive terminal login session, à utiliser pour être connecté dans le conteneur, la commande exit déconnecte l’utilisateur mais ne le fait pas sortir du conteneur
* `sudo machinectl shell buster1` # Open an interactive shell session, l’intérêt est d’être connecté en root sans avoir besoin de se loguer (taper le mot de passe) par contre la commande exit fait sortir du conteneur. À utiliser pour lancer quelques commandes rapidement
* `sudo machinectl copy-to buster1 ~/Documents/proc.jpeg /root/image.jpg` # Copier un fichier dans le conteneur buster1
* `sudo machinectl copy-to buster1 ~/Images /root/Images` # Copier un dossier dans le conteneur buster1
* `sudo machinectl copy-from buster1 /root/xyz abc` # Récupérer le fichier /root/xyz du conteneur buster1 dans le dossier courant de l’hôte
* `sudo machinectl copy-from buster1 /root/xyz` # Récupérer le fichier /root/xyz du conteneur buster1 dans /root (par défaut) de l’hôte
