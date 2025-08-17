+++
title = 'Archlinux - conteneurs LXC'
date = 2022-12-19 00:00:00 +0100
categories = ['virtuel']
+++
![lxc](Linux_Containers_logo.png){:width="100"}

*Les conteneurs Linux (LXC) sont une méthode de virtualisation au niveau du système d'exploitation pour exécuter plusieurs systèmes Linux isolés (conteneurs) sur un seul hôte de contrôle (hôte LXC). Il ne fournit pas de machine virtuelle, mais fournit plutôt un environnement virtuel qui a son propre CPU, mémoire, bloc d'E / S, réseau, etc. et le mécanisme de contrôle des ressources. Ceci est fourni par les espaces de noms et les fonctionnalités de cgroups dans le noyau Linux sur l'hôte LXC. Il est similaire à un chroot, mais offre beaucoup plus d'isolement.*

- [Liens](#liens)
- [Conteneurs "privilégiés" ou "non privilégiés"](#conteneurs-privilégiés-ou-non-privilégiés)
- [Installer LXC/Archlinux](#installer-lxcarchlinux)
    - [Logiciels requis](#logiciels-requis)
    - [Configuration du réseau hôte](#configuration-du-réseau-hôte)
        - [Utilisation d'un pont hôte](#utilisation-dun-pont-hôte)
        - [Utilisation d'un pont NAT](#utilisation-dun-pont-nat)
    - [Considérations relatives au pare-feu](#considérations-relatives-au-pare-feu)
        - [Exemple de règle iptables](#exemple-de-règle-iptables)
        - [Exemple de règle ufw](#exemple-de-règle-ufw)
    - [Exécution de conteneurs en tant qu'utilisateur non root](#exécution-de-conteneurs-en-tant-quutilisateur-non-root)
- [Création de conteneurs](#création-de-conteneurs)
    - [Chemins standards](#chemins-standards)
    - [Créer un conteneur Arch](#créer-un-conteneur-arch)
    - [Créer un conteneur pour une autre distribution (debian,ubuntu,etc...)](#créer-un-conteneur-pour-une-autre-distribution-debianubuntuetc)
- [Configuration des conteneurs](#configuration-des-conteneurs)
    - [Configuration de base avec mise en réseau](#configuration-de-base-avec-mise-en-réseau)
    - [Ressources système à virtualiser/isoler (/var/lib/lxc/CONTAINER_NAME/config)](#ressources-système-à-virtualiserisoler-varliblxccontainer_nameconfig)
- [Considérations sur le programme Xorg (facultatif)](#considérations-sur-le-programme-xorg-facultatif)
- [Considérations sur l'OpenVPN](#considérations-sur-lopenvpn)
- [Gestion des conteneurs](#gestion-des-conteneurs)
    - [Utilisation de base](#utilisation-de-base)
        - [Liste des conteneurs - lxc-ls -f](#liste-des-conteneurs---lxc-ls--f)
        - [Démarrer/Arrêter un conteneur LXC](#démarrerarrêter-un-conteneur-lxc)
        - [S'attacher à un conteneur](#sattacher-à-un-conteneur)
    - [Utilisation avancée](#utilisation-avancée)
        - [Clones LXC](#clones-lxc)
        - [Conversion d'un conteneur privilégié en un conteneur non privilégié](#conversion-dun-conteneur-privilégié-en-un-conteneur-non-privilégié)
        - [Exécution de programmes Xorg](#exécution-de-programmes-xorg)
- [Dépannage](#dépannage)
    - [Échec de la connexion à la racine](#échec-de-la-connexion-à-la-racine)
    - [Pas de connexion au réseau avec veth dans la configuration du conteneur](#pas-de-connexion-au-réseau-avec-veth-dans-la-configuration-du-conteneur)
    - [Error: unknown command (Erreur : commande inconnue)](#error-unknown-command-erreur-commande-inconnue)
    - [Error: Failed at step KEYRING spawning...](#error-failed-at-step-keyring-spawning)
- [LXC cli](#lxc-cli)
    - [Liste des commandes](#liste-des-commandes)
- [Installer les conteneurs Debian sur Archlinux en utilisant LXC](#installer-les-conteneurs-debian-sur-archlinux-en-utilisant-lxc)
    - [Installer un conteneur Debian](#installer-un-conteneur-debian)
    - [Configurer le réseau](#configurer-le-réseau)
    - [Configurer le mot de passe Debian](#configurer-le-mot-de-passe-debian)
    - [Installer openssh-server](#installer-openssh-server)
    - [Démarrer le conteneur Debian](#démarrer-le-conteneur-debian)
    - [Définir un IP statique pour le conteneur](#définir-un-ip-statique-pour-le-conteneur)
    - [Se connecter au conteneur](#se-connecter-au-conteneur)
- [Partager un répertoire de la machine hôte à un conteneur LXC](#partager-un-répertoire-de-la-machine-hôte-à-un-conteneur-lxc)
    - [Exemple](#exemple)

## Liens

* <https://linuxcontainers.org/fr/>
* [Archlinux linux containers](https://wiki.archlinux.org/index.php/Linux_Containers)
* [LXC 1.0 Blog Post Series](https://www.stgraber.org/2013/12/20/lxc-1-0-blog-post-series/)
* [LXD 2.0: Blog post series](https://stgraber.org/2016/03/11/lxd-2-0-blog-post-series-012/)
* [LXC@developerWorks](http://www.ibm.com/developerworks/linux/library/l-lxc-containers/)
* [LXC articles on l3net](http://l3net.wordpress.com/tag/lxc/)
* [LXC CLI Commands Cheet Sheat](https://cmdref.net/middleware/container/lxc/index.html)
* [Using command aliases in LXD to exec a shell](https://blog.simos.info/using-command-aliases-in-lxd-to-exec-a-shell/) 

* [LXC 1.0: Blog post series [0/10]](https://stgraber.org/2013/12/20/lxc-1-0-blog-post-series/)
    * [LXC 1.0: Your first Ubuntu container [1/10]](http://www.stgraber.org/2013/12/20/lxc-1-0-your-first-ubuntu-container/)
    * [LXC 1.0: Your second container [2/10]](http://www.stgraber.org/2013/12/21/lxc-1-0-your-second-container/)
    * [LXC 1.0: Advanced container usage [3/10]](http://www.stgraber.org/2013/12/21/lxc-1-0-advanced-container-usage/)
    * [LXC 1.0: Some more advanced container usage [4/10]](https://www.stgraber.org/2013/12/23/lxc-1-0-some-more-advanced-container-usage/)
    * [LXC 1.0: Container storage [5/10]](http://www.stgraber.org/2013/12/27/lxc-1-0-container-storage/)
    * [LXC 1.0: Security features [6/10]](http://www.stgraber.org/2014/01/01/lxc-1-0-security-features/)
    * [LXC 1.0: Unprivileged containers [7/10]](http://www.stgraber.org/2014/01/17/lxc-1-0-unprivileged-containers/)
    * [LXC 1.0: Scripting with the API [8/10]](http://www.stgraber.org/2014/02/05/lxc-1-0-scripting-with-the-api/)
    * [LXC 1.0: GUI in containers [9/10]](http://www.stgraber.org/2014/02/09/lxc-1-0-gui-in-containers/)
    * [LXC 1.0: Troubleshooting and debugging [10/10]](http://www.stgraber.org/2014/02/18/lxc-1-0-troubleshooting-and-debugging/)

Les alternatives d'utilisation des conteneurs sont systemd-nspawn , docker ou rkt AUR 

## Conteneurs "privilégiés" ou "non privilégiés"

Les LXC peuvent être configurés pour s'exécuter dans des configurations **privilégiées (privileged)** ou **non privilégiées (unprivileged)**   

En général, l'<u>exécution d'un conteneur **non privilégié** est considérée comme plus sûre</u> que l'exécution d'un conteneur privilégié , car les conteneurs non privilégiés ont un degré d'isolation accru en raison de leur conception. La clé de ceci est le mappage de l'UID racine dans le conteneur à un UID non root sur l'hôte, ce qui rend plus difficile pour un piratage à l'intérieur du conteneur d'avoir des conséquences sur le système hôte. En d'autres termes, si un attaquant parvient à s'échapper du conteneur, il ou elle devrait se retrouver avec des droits limités ou inexistants sur l'hôte.

Les packages de noyau Arch linux , linux-lts et linux-zen fournissent actuellement une prise en charge prête à l' emploi pour les conteneurs non privilégiés . De la même façon, avec le package renforcé Linux , les conteneurs non privilégiés ne sont disponibles que pour l'administrateur système; avec des modifications de configuration du noyau supplémentaires requises, car les espaces de noms des utilisateurs sont désactivés par défaut pour les utilisateurs normaux. Cet article contient des informations permettant aux utilisateurs d'exécuter l'un ou l'autre type de conteneur, mais des étapes supplémentaires peuvent être nécessaires pour utiliser des conteneurs non privilégiés .

**Un exemple pour illustrer les conteneurs non privilégiés**  
Pour illustrer la puissance du mappage d'UID, considérez la sortie ci-dessous d'un conteneur non privilégié en cours d'exécution. Nous y voyons les processus conteneurisés appartenant à l'utilisateur root du conteneur dans la sortie de ps : 

```
[root@unprivileged_container /]# ps -ef | head -n 5
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 17:49 ?        00:00:00 /sbin/init
root        14     1  0 17:49 ?        00:00:00 /usr/lib/systemd/systemd-journald
dbus        25     1  0 17:49 ?        00:00:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
systemd+    26     1  0 17:49 ?        00:00:00 /usr/lib/systemd/systemd-networkd
```

Sur l'hôte, cependant, ces processus "root" conteneurisés sont en fait exécutés en tant qu'utilisateur mappé (ID>100000), plutôt que l'utilisateur racine réel de l'hôte : 

```
[root@host /]# lxc-info -Ssip --name sandbox
State:          RUNNING
PID:            26204
CPU use:        10.51 seconds
BlkIO use:      244.00 KiB
Memory use:     13.09 MiB
KMem use:       7.21 MiB

[root@host /]# ps -ef | grep 26204 | head -n 5
UID        PID  PPID  C STIME TTY          TIME CMD
100000   26204 26200  0 12:49 ?        00:00:00 /sbin/init
100000   26256 26204  0 12:49 ?        00:00:00 /usr/lib/systemd/systemd-journald
100081   26282 26204  0 12:49 ?        00:00:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
100000   26284 26204  0 12:49 ?        00:00:00 /usr/lib/systemd/systemd-logind
```


## Installer LXC/Archlinux

### Logiciels requis

>L'installation de **lxc** et **arch-install-scripts** permettra au système hôte d'exécuter des lxcs **privilégiés**.

**Activer la prise en charge pour exécuter des conteneurs "non privilégiés" (facultatif)**  
Modifiez **/etc/lxc/default.conf** pour y ajouter les lignes suivantes:

```
lxc.idmap = u 0 100000 65536
lxc.idmap = g 0 100000 65536
```

Enfin, créez **/etc/subuid** et **/etc/subgid** pour contenir le mappage avec les paires uid/gid conteneurisées pour chaque utilisateur qui sera en mesure d'exécuter les conteneurs.  
L'exemple ci-dessous est simplement pour l'utilisateur root (et l'unité système systemd):

    /etc/subuid

```
root:100000:65536
```

    /etc/subgid

```
root:100000:65536
```

En outre, l'exécution de conteneurs non privilégiés en tant qu'utilisateur non privilégié ne fonctionne que si vous déléguez un cgroup à l'avance (le modèle de délégation cgroup2 applique cette restriction, pas liblxc). Utilisez la commande systemd suivante pour déléguer le cgroup : 

    systemd-run --unit=myshell --user --scope -p "Delegate=yes" lxc-start container_name


**Conteneurs non privilégiés sur des noyaux linux durcis et personnalisés**  
Les utilisateurs souhaitant exécuter des conteneurs **non privilégiés** sur Linux durci ou leur noyau personnalisé doivent effectuer plusieurs étapes de configuration supplémentaires.

Tout d'abord, un noyau est nécessaire qui prend en charge les espaces de noms d'utilisateurs (un noyau avec CONFIG_USER_NS). Tous les noyaux Arch Linux prennent en charge CONFIG_USER_NS. Cependant, en raison de problèmes de sécurité plus généraux, le noyau durci Linux est livré avec des espaces de noms d'utilisateurs activés uniquement pour l' utilisateur root . Il existe deux options pour y créer des conteneurs non privilégiés :

*    Démarrez les conteneurs non privilégiés uniquement en tant que root .
*    Activez le paramètre sysctlkernel.unprivileged_userns_clone pour permettre aux utilisateurs normaux d'exécuter des conteneurs non privilégiés. Cela peut être fait pour la session en cours avec sysctl kernel.unprivileged_userns_clone=1et peut être rendu permanent avec sysctl.d (5) .

### Configuration du réseau hôte

Les LXC prennent en charge différents types de réseaux virtuels et périphériques (voir lxc.container.conf (5) ). Un périphérique de pont sur l'hôte est requis pour la plupart des types de réseaux virtuels, comme illustré dans cette section.

Il y a plusieurs configurations principales à considérer:

1. Le <u>**pont hôte** (host bridge)</u> nécessite que le gestionnaire de réseau de l'hôte gère une interface de pont partagée. L'hôte et tout lxc se verront attribuer une adresse IP dans le même réseau (par exemple 192.168.1.x). Cela pourrait être plus simpliste dans les cas où l'objectif est de conteneuriser un service exposé au réseau comme un serveur Web ou un serveur VPN. L'utilisateur peut considérer le lxc comme simplement un autre PC sur le LAN physique et transférer les ports nécessaires dans le routeur en conséquence. La simplicité supplémentaire peut également être considérée comme un vecteur de menace supplémentaire, encore une fois, si le trafic WAN est acheminé vers le lxc, le fait qu'il s'exécute sur une plage distincte présente une surface de menace plus petite.
2. Le <u>**pont NAT** (NAT bridge)</u> ne nécessite pas le gestionnaire de réseau de l'hôte pour gérer le pont. lxc est livré avec lxc-netlequel crée un pont NAT appelé lxcbr0. Le pont NAT est un pont autonome avec un réseau privé qui n'est pas ponté vers le périphérique Ethernet de l'hôte ou vers un réseau physique. Il existe en tant que sous-réseau privé dans l'hôte.

#### Utilisation d'un pont hôte

Voir [Network Bridge](https://wiki.archlinux.org/index.php/Network_bridge)

#### Utilisation d'un pont NAT

Installez **dnsmasq** qui est une dépendance pour **lxc-net** et avant de démarrer le pont, créez d'abord un fichier de configuration pour celui-ci:

    /etc/default/lxc-net

```
# Laissez USE_LXC_BRIDGE à "true" si vous voulez utiliser lxcbr0 pour votre
# de conteneurs.  Mettez "false" si vous utilisez virbr0 ou un autre
# bridge, ou mavlan vers le NIC de votre hôte.
USE_LXC_BRIDGE="true"

# Si vous changez le LXC_BRIDGE pour autre chose que lxcbr0, alors
# vous devrez également mettre à jour votre /etc/lxc/default.conf ainsi que le
# configuration (/var/lib/lxc/<container>/config) pour tous les conteneurs
# déjà créé en utilisant la configuration par défaut pour refléter le nouveau pont
# nom.
# Si vous avez installé le démon dnsmasq, vous devrez également mettre à jour
# /etc/dnsmasq.d/lxc et redémarrez le démon dnsmasq à l'échelle du système.
LXC_BRIDGE="lxcbr0"
LXC_ADDR="10.0.3.1"
LXC_NETMASK="255.255.255.0"
LXC_NETWORK="10.0.3.0/24"
LXC_DHCP_RANGE="10.0.3.2,10.0.3.254"
LXC_DHCP_MAX="253"
# Décommentez la ligne suivante si vous souhaitez utiliser un fichier conf pour le lxcbr0
# dnsmasq.  For instance, you can use 'dhcp-host=mail1,10.0.3.100' to have
# container 'mail1' always get ip address 10.0.3.100.
#LXC_DHCP_CONFILE=/etc/lxc/dnsmasq.conf

# Décommentez la ligne suivante si vous voulez que le dnsmasq de lxcbr0 résolve le .lxc
# domaine.  Vous pouvez alors ajouter "server=/lxc/10.0.3.1" (ou votre $LXC_ADDR actuel)
# au fichier de configuration dnsmasq de votre système (normalement /etc/dnsmasq.conf,
# ou /etc/NetworkManager/dnsmasq.d/lxc.conf sur les systèmes qui utilisent NetworkManager).
# Une fois ces modifications effectuées, redémarrez les services lxc-net et network-manager.
# 'container1.lxc' sera alors résolu sur votre hôte.
#LXC_DOMAIN="lxc"
```

Si on veut pouvoir gérer des adresses ip statiques ,il faut réduire la plage dhcp  
Remplacer `LXC_DHCP_RANGE="10.0.3.2,10.0.3.254"` par `LXC_DHCP_RANGE="10.0.3.10,10.0.3.254"`  
Plage ip statique 10.0.3.3 à 10.0.3.10

>Conseil: assurez-vous que la plage IP du pont n'interfère pas avec le réseau local.

Ensuite, nous devons modifier le modèle de conteneur LXC afin que nos conteneurs utilisent notre pont:

    /etc/lxc/default.conf

```
lxc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
lxc.net.0.hwaddr = 00:16:3e:a2:cd:45
```

Créez éventuellement un fichier de configuration pour définir manuellement l'adresse IP de tout conteneur:

    /etc/lxc/dnsmasq.conf

```
dhcp-host=playtime,10.0.3.100
```

Maintenant, démarrez et activez lxc-net.service pour créer l'interface du pont.

    systemctl start lxc-net.service
    systemctl status lxc-net.service

```
● lxc-net.service - LXC network bridge setup
     Loaded: loaded (/usr/lib/systemd/system/lxc-net.service; disabled; vendor preset: disabled)
     Active: active (exited) since Wed 2020-04-29 12:13:10 CEST; 22s ago
       Docs: man:lxc
    Process: 23592 ExecStart=/usr/lib/lxc/lxc-net start (code=exited, status=0/SUCCESS)
   Main PID: 23592 (code=exited, status=0/SUCCESS)

avril 29 12:13:10 yannick-pc dnsmasq-dhcp[23619]: DHCP, plage d'adresses IP 10.0.3.2 -- 10.0.3.254, durée de b>
avril 29 12:13:10 yannick-pc dnsmasq-dhcp[23619]: DHCP, sockets bound exclusively to interface lxcbr0
avril 29 12:13:10 yannick-pc dnsmasq[23619]: Lecture de /etc/resolv.conf
avril 29 12:13:10 yannick-pc dnsmasq[23619]: utilise le serveur de nom 10.16.0.1#53
avril 29 12:13:10 yannick-pc dnsmasq[23619]: utilise le serveur de nom fdda:d0d0:cafe:1302::#53
avril 29 12:13:10 yannick-pc dnsmasq[23619]: utilise le serveur de nom 80.67.169.12#53
avril 29 12:13:10 yannick-pc dnsmasq[23619]: utilise le serveur de nom 80.67.169.40#53
avril 29 12:13:10 yannick-pc dnsmasq[23619]: utilise le serveur de nom fd0f:ee:b0::1#53
avril 29 12:13:10 yannick-pc dnsmasq[23619]: lecture /etc/hosts - 4 adresses
avril 29 12:13:10 yannick-pc systemd[1]: Finished LXC network bridge setup.
```

### Considérations relatives au pare-feu

Étant donné que le lxc fonctionne sur le sous-réseau 10.0.3.x, l'accès aux services tels que ssh, httpd, etc. devra être activement transmis au lxc. En principe, le pare-feu sur l'hôte doit transmettre le trafic entrant sur le port attendu du conteneur.

#### Exemple de règle iptables

Le but de cette règle est d'autoriser le trafic ssh vers le lxc:

```shell
# iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 2221 -j DNAT --to-destination 10.0.3.100:22
```

Cette règle transfère le trafic TCP provenant du port 2221 à l'adresse IP du lxc sur le port 22.  
Remarque: Assurez-vous d'autoriser le trafic sur 2221 / tcp sur l'hôte et d'autoriser le trafic 22 / tcp sur lxc.  

Pour ssh dans le conteneur depuis un autre PC sur le LAN, il faut ssh sur le port 2221 à l'hôte. L'hôte transmettra ensuite ce trafic au conteneur.

    $ ssh -p 2221 host.lan

#### Exemple de règle ufw

Si vous utilisez ufw , ajoutez ce qui suit au bas de /etc/ufw/before.rulespour rendre cela persistant:

    /etc/ufw/before.rules

```
*nat
:PREROUTING ACCEPT [0:0]
-A PREROUTING -i eth0 -p tcp --dport 2221 -j DNAT --to-destination 10.0.3.101:22
COMMIT
```

### Exécution de conteneurs en tant qu'utilisateur non root

Pour créer et démarrer des conteneurs en tant qu'utilisateur non root, une configuration supplémentaire doit être appliquée.

Créez le fichier usernet sous /etc/lxc/lxc-usernet. Selon la lxc-usernetpage de manuel, l'entrée par ligne est:

    user type bridge number

Configurez le fichier avec l'utilisateur devant créer des conteneurs. Le pont sera le même que celui défini dans /etc/default/lxc-net.

Une copie de la /etc/lxc/default.confest nécessaire dans le répertoire personnel de l'utilisateur non root, par exemple ~/.config/lxc/default.conf(créez le répertoire si nécessaire).

L'exécution de conteneurs en tant qu'utilisateur non root nécessite des +xautorisations sur ~/.local/share/. Apportez cette modification avec chmod avant de démarrer un conteneur.

## Création de conteneurs

Les conteneurs sont construits en utilisant lxc-create. Avec la sortie de lxc-3.0.0-1, l'amont a déprécié les modèles stockés localement.

### Chemins standards

Un mot rapide sur la façon dont LXC fonctionne habituellement et où il stocke ses fichiers :

* /var/lib/lxc (emplacement par défaut pour les conteneurs) 
* /var/lib/lxcsnap (emplacement par défaut pour les instantanés) 
* /var/cache/lxc (emplacement par défaut pour le cache du modèle) 
* $HOME/.local/share/lxc (emplacement par défaut pour les conteneurs non privilégiés) 
* $HOME/.local/share/lxcsnap (emplacement par défaut pour les instantanés non privilégiés) 
* $HOME/.cache/lxc (emplacement par défaut pour le cache de modèle non privilégié) 

Le chemin par défaut, également appelé **lxcpath**, peut être remplacé sur la ligne de commande avec l'option `-P` ou une fois pour toutes en définissant **"lxcpath = /new/path"** dans `/etc/lxc/lxc.conf` (ou `$HOME/.config /lxc/lxc.conf` pour les conteneurs non privilégiés).  
Le répertoire d'instantanés est toujours **"snap"** ajouté à **lxcpath** afin qu'il suive comme par magie lxcpath. Le cache de modèle est malheureusement codé en dur et ne peut pas être facilement déplacé sans compter sur des montages liés ou des liens symboliques.  
La configuration par défaut utilisée pour tous les conteneurs au moment de la création provient de 
/etc/lxc/default.conf (pas encore d'équivalent non privilégié).   
Les modèles eux-mêmes sont stockés dans /usr/share/lxc/templates.

### Créer un conteneur Arch

Pour créer un conteneur Arch, appelez comme ceci:

```shell
lxc-create -n playtime -t download -- --dist archlinux --release current --arch amd64
```

### Créer un conteneur pour une autre distribution (debian,ubuntu,etc...)

Pour les autres distributions, appelez comme ceci et sélectionnez les options dans les distributions prises en charge affichées dans la liste:

```shell
lxc-create -n playtime -t download

Setting up the GPG keyring
Downloading the image index

---
DIST	RELEASE	ARCH	VARIANT	BUILD
---
archlinux	current	amd64	default	20200429_04:18
archlinux	current	arm64	default	20200429_04:18
archlinux	current	armhf	default	20200429_04:18
debian	buster	amd64	default	20200429_05:24
debian	buster	arm64	default	20200429_05:24
debian	buster	armel	default	20200429_05:24
debian	buster	armhf	default	20200429_05:56
debian	buster	i386	default	20200429_05:24
debian	buster	ppc64el	default	20200429_05:24
debian	buster	s390x	default	20200429_05:24
debian	jessie	amd64	default	20200429_05:24
debian	jessie	armel	default	20200429_06:26
debian	jessie	armhf	default	20200429_05:24
debian	jessie	i386	default	20200429_05:24
ubuntu	bionic	amd64	default	20200429_07:43
ubuntu	bionic	arm64	default	20200429_08:39
ubuntu	bionic	armhf	default	20200429_07:42
ubuntu	bionic	i386	default	20200429_07:42
ubuntu	bionic	ppc64el	default	20200429_07:42
ubuntu	bionic	s390x	default	20200429_07:52
---

Distribution: 
debian
Release: 
buster
Architecture: 
amd64

Using image from local cache
Unpacking the rootfs

---
You just created a Debian buster amd64 (20200429_05:24) container.

To enable SSH, run: apt install openssh-server
No default root or user password are set by LXC.
```

**Astuce:** les utilisateurs peuvent éventuellement installer **haveged** et lancer haveged.service à éviter un blocage perçu pendant le processus de configuration en attendant que l'entropie du système soit amorcée. Sans cela, la génération de clés privées / GPG peut ajouter une longue attente au processus.  
**Astuce:** Les utilisateurs de Btrfs peuvent ajouter -B btrfspour créer un sous-volume Btrfs pour stocker les rootfs conteneurisés. Cela est pratique si vous clonez des conteneurs à l'aide de la lxc-clonecommande. Les utilisateurs ZFS peuvent utiliser -B zfs, en conséquence.  
**Note** : les utilisateurs souhaitant utiliser les anciens modèles peuvent les trouver dans lxc-templatesAUR ou bien les utilisateurs peuvent créer leurs propres modèles avec distrobuilderAUR.

## Configuration des conteneurs

Les exemples ci-dessous peuvent être utilisés aussi bien avec des conteneurs privilégiés que non privilégiés.  
Notez que pour les conteneurs non privilégiés, des lignes supplémentaires seront présentes par défaut qui ne sont pas montrées dans les exemples, y compris les valeurs lxc.idmap = u 0 100000 65536 et lxc.idmap = g 0 100000 65536 définies en option dans la section #Enable support to run unprivileged containers (optional).

### Configuration de base avec mise en réseau

Note : Avec la sortie de lxc-1:2.1.0-1, de nombreuses options de configuration ont été modifiées. Les conteneurs existants doivent être mis à jour ; les utilisateurs sont dirigés vers le tableau de ces changements dans les [notes de version v2.1](https://discuss.linuxcontainers.org/t/lxc-2-1-has-been-released/487).

### Ressources système à virtualiser/isoler (/var/lib/lxc/CONTAINER_NAME/config)

Les ressources système à virtualiser/isoler lorsqu'un processus utilise le conteneur sont définies dans **/var/lib/lxc/CONTAINER_NAME/config**  
Par défaut, le processus de création effectuera une configuration minimale sans support réseau. Vous trouverez ci-dessous un exemple de configuration avec mise en réseau fourni par `lxc-net.service` :

    /var/lib/lxc/playtime/config

```
# Template used to create this container: /usr/share/lxc/templates/lxc-archlinux
# Parameters passed to the template:
# For additional config options, please look at lxc.container.conf(5)

# Distribution configuration
lxc.include = /usr/share/lxc/config/common.conf
lxc.arch = x86_64

# Container specific configuration
lxc.rootfs.path = dir:/var/lib/lxc/playtime/rootfs
lxc.uts.name = playtime

# Network configuration
lxc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
lxc.net.0.hwaddr = ee:ec:fa:e9:56:7d
```

## Considérations sur le programme Xorg (facultatif)

Afin d'exécuter des programmes sur l'écran de l'hôte, certains montages de liaison doivent être définis afin que les programmes conteneurisés puissent accéder aux ressources de l'hôte. Ajoutez la section suivante dans **/var/lib/lxc/playtime/config** :

```
## for xorg
lxc.mount.entry = /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry = /dev/snd dev/snd none bind,optional,create=dir
lxc.mount.entry = /tmp/.X11-unix tmp/.X11-unix none bind,optional,create=dir,ro
lxc.mount.entry = /dev/video0 dev/video0 none bind,optional,create=file
```

Si une erreur de permission est toujours présente dans l'invité LXC, appelez `xhost +` dans l'hôte pour permettre à l'invité de se connecter au serveur d'affichage de l'hôte. Prenez note des problèmes de sécurité que pose l'ouverture du serveur d'affichage en procédant ainsi. En outre, ajoutez la ligne suivante **avant** les lignes de montage de liaison ci-dessus.

    lxc.mount.entry = tmpfs tmp tmpfs defaults

**Note :** Cela ne fonctionnera pas si vous utilisez des conteneurs non privilégiés.

## Considérations sur l'OpenVPN

Les utilisateurs souhaitant utiliser OpenVPN dans le conteneur sont priés de se diriger soit vers [OpenVPN (client) dans des conteneurs Linux](https://wiki.archlinux.org/index.php/OpenVPN_(client)_in_Linux_containers) et/ou [OpenVPN (serveur) dans des conteneurs Linux](https://wiki.archlinux.org/index.php/OpenVPN_(server)_in_Linux_containers). 

## Gestion des conteneurs

### Utilisation de base

#### Liste des conteneurs - lxc-ls -f

Pour répertorier tous les conteneurs LXC installés :

```shell
lxc-ls -f
```

#### Démarrer/Arrêter un conteneur LXC

Systemd peut être utilisé pour démarrer et arrêter les LXC via `lxc@CONTAINER_NAME.service`  
Activez `lxc@CONTAINER_NAME.service` pour qu'il démarre lorsque le système hôte démarre.

Les utilisateurs peuvent également démarrer/arrêter des LXC sans systemd.  
Démarrer un conteneur :

```shell
lxc-start -n CONTAINER_NAME
```

Arrêtez un conteneur :

```shell
lxc-stop -n CONTAINER_NAME
```

Pour se connecter à un conteneur :

```shell
lxc-console -n CONTAINER_NAME
```

Une fois connecté, traitez le conteneur comme n'importe quel autre système linux, définissez le mot de passe root, créez des utilisateurs, installez des paquets, etc.

#### S'attacher à un conteneur

Pour s'attacher à un conteneur :

```shell
lxc-attach -n CONTAINER_NAME --clear-env
```

Cela fonctionne à peu près de la même manière que la console lxc, mais cela provoque des démarrages avec une invite de la racine à l'intérieur du conteneur, contournant la connexion.  
**Sans le drapeau `--clear-env`, l'hôte passera ses propres variables d'environnement dans le conteneur (y compris $PATH, donc certaines commandes ne fonctionneront pas lorsque les conteneurs sont basés sur une autre distribution).**

### Utilisation avancée

#### Clones LXC

Les utilisateurs ayant besoin de gérer plusieurs conteneurs peuvent simplifier les tâches administratives (gestion des utilisateurs, mises à jour du système, etc.) en utilisant des instantanés. La stratégie consiste à mettre en place et à tenir à jour un seul conteneur de base, puis, si nécessaire, à le cloner (instantané). La puissance de cette stratégie est que l'espace disque et la surcharge du système sont vraiment minimisés puisque les instantanés utilisent un montage par superposition pour n'écrire que sur le disque, uniquement les différences de données. Le système de base est en lecture seule, mais les modifications dans les instantanés sont autorisées par les superpositions.

Note : les superpositions pour les conteneurs non privilégiés ne sont pas supportées dans le noyau principal actuel d'Arch Linux pour des raisons de sécurité.

Par exemple, configurez un conteneur comme indiqué ci-dessus. Nous l'appellerons "base" pour les besoins de ce guide. Créez maintenant 2 instantanés de "base" que nous appellerons "snap1" et "snap2" avec ces commandes :

```shell
lxc-copy -n base -N snap1 -B overlayfs -s
lxc-copy -n base -N snap2 -B overlayfs -s
```

Note : Si une IP statique a été définie pour le lxc "base", elle devra être modifiée manuellement dans la configuration pour "snap1" et pour "snap2" avant de les lancer. Si le processus doit être automatisé, un script utilisant sed peut le faire automatiquement bien que cela dépasse le cadre de cette section du wiki.

Les instantanés peuvent être lancés/arrêtés comme n'importe quel autre conteneur. Les utilisateurs peuvent éventuellement détruire les instantanés et toutes les nouvelles données qu'ils contiennent avec la commande suivante. Notez que le lxc "de base" sous-jacent n'est pas touché :

```shell
lxc-destroy -n snap1 -f
```

Des unités systémiques et des scripts de wrapper pour gérer les instantanés pour pi-hole et OpenVPN sont disponibles pour automatiser le processus dans les instantanés du service lxc.

#### Conversion d'un conteneur privilégié en un conteneur non privilégié

Une fois que le système a été configuré pour utiliser des conteneurs non privilégiés (voir, #Enable support to run unprivileged containers (optional)), nsexec-bzrAUR contient un utilitaire appelé uidmapshift qui est capable de convertir un conteneur privilégié existant en un conteneur non privilégié pour éviter une reconstruction totale de l'image.

Attention :  

*    Il est recommandé de sauvegarder l'image existante avant d'utiliser cet utilitaire !
*    Cet utilitaire ne changera pas les UIDs et GIDs dans ACL, les utilisateurs devront les changer manuellement !


Appelez l'utilitaire pour effectuer une conversion de la sorte :

```shell
uidmapshift -b /var/lib/lxc/foo 0 100000 65536
```

Des options supplémentaires sont disponibles en appelant simplement uidmapshift sans aucun argument.

#### Exécution de programmes Xorg

Soit attacher à ou SSH dans le conteneur cible et préfixer l'appel au programme avec le DISPLAY ID de la session X de l'hôte. Pour la plupart des configurations simples, l'affichage est toujours 0.

Un exemple d'exécution de Firefox à partir du conteneur dans l'affichage de l'hôte :

    $ DISPLAY=:0 firefox

Pour éviter de s'attacher ou de se connecter directement au conteneur, on peut aussi utiliser les éléments suivants sur l'hôte pour automatiser le processus :

```shell
lxc-attach -n playtime --clear-env -- sudo -u YOURUSER env DISPLAY=:0 firefox
```

## Dépannage

### Échec de la connexion à la racine

Si l'erreur suivante se présente lors de la tentative de connexion à l'aide de la console lxc :

    login: root
    Login incorrect

Et le journal du conteneur le montre :  
*pam_securetty(login:auth): access denied: tty 'pts/0' is not secure !*

Supprimez /etc/securetty[1] et /usr/share/factory/etc/securetty sur le système de fichiers du conteneur. Ajoutez-les éventuellement à `NoExtract` dans **/etc/pacman.conf** pour éviter qu'ils ne soient réinstallés.

Alternativement, créer un nouvel utilisateur dans lxc-attach et l'utiliser pour se connecter au système, puis passer en root.

```
# lxc-attach -n playtime
[root@playtime]# useradd -m -Gwheel newuser
[root@playtime]# passwd newuser
[root@playtime]# passwd root
[root@playtime]# exit
# lxc-console -n playtime
[newuser@playtime]$ su
```

### Pas de connexion au réseau avec veth dans la configuration du conteneur

Si vous ne pouvez pas accéder à votre LAN ou WAN avec une interface réseau configurée en tant que veth et configurée via /etc/lxc/containername/config. Si l'interface virtuelle se voit attribuer l'adresse IP et doit être connectée correctement au réseau.

    ip addr show veth0 
    inet 192.168.1.111/24

Vous pouvez désactiver toutes les formules IP statiques pertinentes et essayer de régler l'IP dans le conteneur amorcé, comme vous le feriez normalement.

Exemple de conteneur/configuration

```
...
lxc.net.0.type = veth
lxc.net.0.name = veth0
lxc.net.0.flags = up
lxc.net.0.link = bridge
...
```

Et ensuite attribuer l'IP par une méthode préférée à l'intérieur du conteneur, voir aussi Configuration du réseau#Gestion du réseau.

### Error: unknown command (Erreur : commande inconnue)

L'erreur peut se produire lorsqu'une commande de base (ls, cat, etc.) sur un conteneur attaché est tapée alors qu'une autre distribution Linux est conteneurisée par rapport au système hôte (par exemple, le conteneur Debian dans le système hôte Arch Linux). Lors de l'attachement, utilisez l'argument `--clear-env` :

```shell
lxc-attach -n container_name --clear-env
```

### Error: Failed at step KEYRING spawning...

Les services dans un conteneur non privilégié peuvent échouer avec le message suivant

```
some.service: Failed to change ownership of session keyring: Permission denied
some.service: Failed to set up kernel keyring: Permission denied
some.service: Failed at step KEYRING spawning ....: Permission denied
```

Créer un fichier /etc/lxc/unpriv.seccomp contenant

/etc/lxc/unpriv.seccomp

```
2
blacklist
[all]
keyctl errno 38
```

Ajoutez ensuite la ligne suivante à la configuration du conteneur après lxc.idmap

    lxc.seccomp.profile = /etc/lxc/unpriv.seccomp


## LXC cli

### Liste des commandes

```
lxc-ls
lxc-ls -f <- -f = --fancy 	
lxc-create -n CONTAINER -t XXXX -- --release X 	
lxc-start -n CONTAINER
lxc-start -n CONTAINER -d ← start background 	
lxc-stop -n CONTAINER
lxc-stop -k -n CONTAINER ← stop force 	
lxc-console -n CONTAINER  #	<Ctrl+a q> to exit
lxc-attach -n CONTAINER 	
lxc-info -n CONTAINER 	
lxc-destroy -n container1
lxc-destroy -n container1 -f 	
```

# Installer les conteneurs Debian sur Archlinux en utilisant LXC

Installez les paquets _lxc_ et _debootstrap_ :
```
# pacman -Sy lxc debootstrap 
```

### Installer un conteneur Debian

Je vais installer Debian 10. J'utiliserai donc la version _buster_.
Si vous voulez installer Debian 10, utilisez la version _précise_.

Je crée un conteneur LXC nommé _debian-10_ avec la version _buster_ et l'architecture _amd64_ :

```shell
lxc-create --name=debian-10 --template=download -- --dist debian --release buster --arch amd64


Setting up the GPG keyring
Downloading the image index
Downloading the rootfs
Downloading the metadata
The image cache is now ready
Unpacking the rootfs

---
You just created a Debian buster amd64 (20200429_05:24) container.

To enable SSH, run: apt install openssh-server
No default root or user password are set by LXC.
```

### Configurer le réseau

Installez le paquet _libvirt_ car nous avons besoin du programme _virsh_.
Nous aurons également besoin de _ebtables_ et _dnsmasq_ pour la mise en réseau NAT/DHCP par défaut.  [(*)](https://wiki.archlinux.org/index.php/libvirt#Server)

    pacman -Sy libvirt ebtables dnsmasq


Start and enable the _libvirtd_ daemon.

    systemctl start libvirtd
    systemctl enable libvirtd


Imprimez la configuration du réseau virtuel _default_ fournie par _libvirtd_.

    virsh net-dumpxml default

```xml
<network>
  <name>default</name>
  <uuid>6115b620-438d-44ad-9215-ce3ca396a890</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:df:da:3c'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```

Editez le fichier **/var/lib/lxc/debian-10/config** pour mettre en place la configuration réseau du conteneur :

```
# Network configuration
lxc.net.0.type = veth
lxc.net.0.flags = up
lxc.net.0.name = eth0
lxc.net.0.link = virbr0
```

Configurer l'interface _virb0_ :

    virsh net-start default

### Configurer le mot de passe Debian

    sudo lxc-attach -n debian-10
    passwd Debian

Tapez ctrl-d pour vous détacher de la session.

### Installer openssh-server

    sudo lxc-start -F -n debian-10

Connectez-vous sous le nom de "Debian" et installez le serveur openssh comme suit :

    sudo apt install openssh-server

### Démarrer le conteneur Debian

Démarrez le conteneur Debian comme suit :

    lxc-start --name debian-10

Obtenez l'adresse IP du conteneur Debian :

    lxc-ls -f debian-10 -F IPV4

Vous pouvez arrêter le conteneur de Debian comme suit :

    lxc-stop --name debian-10


### Définir un IP statique pour le conteneur

Modifier le réseau virtuel par défaut fourni par _libvirtd_ pour restreindre la plage du DHCP
de 192.168.122.2-192.168.122.254 à 192.168.122.10-192.122.254.

    virsh net-edit default

comme suit :

```xml
<network>
  <name>default</name>
  <uuid>bd881c3e-406f-40bb-84ed-b09e68f210e2</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:2d:23:30'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.10' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```

Redémarrez le démon _libvirtd_ :

    systemctl restart libvirtd

Arrêtez le conteneur _buster_ :  

    lxc-stop --name debian-10


Ajouter la ligne suivante dans le fichier **/var/lib/lxc/debian-10/config** :

    lxc.net.0.ipv4.address = 192.168.122.2/24

Redémarrez le conteneur _debian-10_.

### Se connecter au conteneur

    ssh Debian@192.168.122.2

Utilisez le mot de passe _Debian_ pour vous connecter.

## Partager un répertoire de la machine hôte à un conteneur LXC

**mycontainer**  est le nom du conteneur

* Connectez-vous au conteneur et créez un répertoire vide, ce sera le point de montage
* Déconnectez-vous et arrêtez le conteneur.
* Ouvrez le fichier de configuration de votre conteneur
    * Pour les conteneurs LXC ordinaires : `/var/lib/lxc/mycontainer/config`
    * Pour les conteneurs LXC *non privilégiés* : `$HOME/.local/share/lxc/mycontainer/config`.
* Ajouter une nouvelle ligne au-dessus de la directive "lxc.mount", qui suit le format ci-dessous. Remplacez les chemins appropriés si nécessaire :
    * `lxc.mount.entry = /chemin/vers/dossier/sur/hôte /chemin/vers/monter/point none bind 0 0`
    * Ces deux chemins sont **relatifs à la machine hôte**.
    * L'emplacement de la racine fs dans le conteneur peut être trouvé à l'adresse
        * Pour les conteneurs LXC normaux : `/var/lib/lxc/mycontainer/rootfs/`
        * Pour les conteneurs LXC *non privilégiés* : `$HOME/.local/share/lxc/mycontainer/rootfs``.

**Note** : Si l'utilisateur de l'hôte n'existe pas dans le conteneur, le conteneur sera quand même monté, mais avec `nobody:nogroup` comme propriétaire. Cela peut ne pas poser de problème, sauf si vous devez écrire dans ces fichiers, auquel cas vous devrez donner à tout le monde le droit d'écrire dans ce dossier. (i.e. `chmod -R go+w /dossier/to/share`)

### Exemple

Je veux partager `/home/media/foobar` avec mon conteneur *non privilégié* `debian-lxc`. Dans `debian-lxc`, je veux que ce dossier se trouve à `/mnt/partage`.

Dans le conteneur :

``` bash
$ cd /mnt
$ sudo mkdir partage
$ logout
```

Dans l'hôte, j'ajouterai la ligne suivante au-dessus de "lxc-mount" dans "/home/media/.local/share/lxc/debian-lxc/config" :

```
lxc.mount.entry = /home/media/foobar /home/julian/.local/share/lxc/debian-lxc/rootfs/mnt/partage none bind 0 0

```

Puis

"Bash
$ lxc-start -n debian-lxc -d
```

### Lecture complémentaire

* <https://wiki.debian.org/LXC>
