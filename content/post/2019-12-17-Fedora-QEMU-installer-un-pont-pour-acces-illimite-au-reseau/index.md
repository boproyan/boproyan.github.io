+++
title = 'KVM/QEMU Fedora ,installer un pont pour un accès illimité au réseau'
date = 2019-12-17 00:00:00 +0100
categories = ['virtuel']
+++
## Qemu

![qemu](Qemu_logo.png)

Article original ["QEMU : installer un pont pour un accès illimité au réseau"](https://doc.fedora-fr.org/wiki/QEMU_:_installer_un_pont_pour_un_acc%C3%A8s_illimit%C3%A9_au_r%C3%A9seau) issu de la communauté francophone de Fedora   

### Pré-requis

* Convention :
    * les lignes de commande qui débutent par le caractère # sont à saisir par le root
    * les lignes de commande qui débutent par le caractère $ sont à saisir par l’utilisateur 
* le paquetage bridge-utils : si absent, l’installer par la commande :  
`# yum install bridge-utils`  
* l’interface VTUN : normalement installée de base avec FC4.
* l’utilitaire sudo : normalement installé de base avec FC4
* les paquetages de QEMU maintenus par Thomas Chung :
    * suivre les liens disponibles sur http://fedoranews.org/tchung/qemu/ pour télécharger les rpms adaptés à notre version du noyau.
    * Nous avons besoin de kernel-module-qemu-??? et de qemu-???.
    * Ensuite, installer en une seule fois les 2 paquetages par la commande :  
`# rpm -ivh kernel-module-qemu-0.7.2-4.fc4.2.6.13_1.1532_FC4.i386.rpm qemu-0.7.2-4.fc4.2.6.13_1.1532_FC4.i386.rpm`

    * Cet exemple est fourni pour la version 0.7.2 de QEMU et le noyau 2.6.13_1.1352_FC4
    * Cette manipulation sera à refaire à chaque changement de version du noyau.
    * Avis aux amateurs et impatients, Thomas Chung explique aussi sur son site comment créer ses propres paquetages à partir des sources de Fabrice Bellard.

## L’environnement d’exploitation

### L’interface VTUN

* Pour communiquer avec le monde extérieur à travers le noyau de Fedora, un PC virtualisé par QEMU utilise une interface réseau VTUN, qui, une fois initialisée, s’appelle tun0 ou tun1 soit, génériquement, tunx.
* La machine virtuelle accède normalement au réseau via son périphérique Ethernet (émulé), QEMU fait alors apparaître les trames à la sortie de tunx et le noyau Linux les récupère pour routage (vers le monde extérieur) ou broutage (néologisme intéressant qui signifie « routage vers un autre port du pont »). Dans le sens inverse, tout ce qui concerne le PC émulé est dirigé sur l’interface tunx, et QEMU fait croire au PC virtuel que tout arrive sur son équipement réseau. Magique, n’est ce pas ?
* QEMU est serviable : il se charge de créer et de supprimer l’interface tunx mais nous laisse la possibilité de la paramétrer à travers le fichier /etc/qemu-ifup. En tant que root, rédigeons ce fichier de la façon suivante :

```bash
 #!/bin/bash
 sudo env-qemu add $1
```

(nous verrons pourquoi au paragraphe "Le script") puis, rendons-le exécutable par la commande :  

```bash
 # chmod a+x /etc/qemu-ifup
```

### Le pont

* Sur la machine hôte Fedora Core 4, nous allons installer un pont réseau dont la transparence permet notamment la transmission des trames de diffusion générale (broadcast) indispensables à certains protocoles comme DHCP (sinon, c’est le serveur DHCP intégré à QEMU qui répond) ou l’accès aux réseaux Windows, ....
* Un pont rassemble plusieurs interfaces réseau derrière un point de communication unique.
* Le pont ne connaît que le protocole ARP ce qui permet de faire transiter toute trame Ethernet quel que soit son contenu (IPV4, NETBEUI, IPX, ...) et sa destination :
    * le réseau local auquel le PC hôte est connecté, quelle que soit l’interface interne émettrice (un « port » du pont)
    * l’interface réseau du système hôte Fedora Core 4 sur sa propre adresse IP
    * l’interface réseau du PC virtuel disposant aussi de sa propre adresse IP

### La mémoire partagée

* La mémoire RAM de la machine virtuelle est prélevée par QEMU dans l’espace mémoire partagé mis à disposition par FC4 (voir paramétrage shm dans /etc/fstab).
* Les valeurs par défaut étant assez restrictives, le script d’initialisation met en place une réserve mémoire de 512 MO, et il est de votre responsabilité de vérifier que cette valeur - ou celle que vous choisirez - ne dépasse pas les capacités de votre machine.
* Pour obtenir les meilleures performances, n’utilisez que de la mémoire vive (éventuellement, investissez dans une barrette de RAM).

### Le script

* L’ensemble du paramétrage se fera à l’aide d’un script unique : /usr/local/bin/env-qemu.
* Nous confions à root le soin de le rédiger ainsi (là, on fait chauffer le copier-coller) :

```
#!/bin/bash

# Mise en place d'un environnement d'exécution d'une machine
# virtuelle sous QEMU pour partage de la connexion réseau
# avec une une machine hôte sous FC4.

# Valeurs par défaut (changez pour vos propres valeurs)
IP_ROUTEUR=192.168.1.1
IP_HOTE=192.168.1.31
IF_HOTE=eth0
PONT=lepont

# Fonction de mise en place du pont
start () {
        if [ -n "$1" ]; then
                IF_HOTE=$1
                shift
        fi
        set $(/sbin/ifconfig | grep $IF_HOTE) >/dev/null
        if [ -z "$1" ]; then
                echo "Interface réseau $IF_HOTE non trouvée."
                exit 1
        fi
        shift $(($# - 1))
        echo "Utilisation de l'interface" $IF_HOTE

        # Met en place l'adresse de la passerelle
        set $(/sbin/route | grep default) >/dev/null 2>&1
        if [ -n "$2" ]; then
                IP_ROUTEUR=$2
                shift $(($# - 1))
        fi
        echo "Adresse passerelle =" $IP_ROUTEUR

        #Récupère l'adresse IP de l'interface réseau
        set $(/sbin/ip addr show $IF_HOTE | grep inet) >/dev/null 2>&1
        if [ -n "$2" ]; then
                IP_HOTE=$2
        fi
        echo "Adresse du pont =" $IP_HOTE

        # Donne accès à tous sur l'interface VTUN
        /bin/chmod 666 /dev/net/tun

        # Création et paramétrage du pont
        /usr/sbin/brctl addbr $PONT
        /sbin/ifconfig $IF_HOTE 0.0.0.0
        /bin/sleep 2
        /usr/sbin/brctl addif $PONT $IF_HOTE
        /sbin/ifconfig $PONT $IP_HOTE
        /sbin/route add default gw $IP_ROUTEUR

        # Paramétrage de l'espace mémoire partagée
        /bin/umount /dev/shm
        /bin/mount -t tmpfs -o size=512m none /dev/shm
}

# Ajout d'une interface sur le pont en place
add () {
        /sbin/ifconfig $1 0.0.0.0
        /usr/sbin/brctl addif $PONT $1
}

# Arrêt et suppression du pont puis restauration du réseau initial
stop () {
        if [ -n "$1" ]; then
                IF_HOTE=$1
        fi
        TESTPONT=$(/sbin/ifconfig | grep $PONT)
        if [ -z "$TESTPONT" ]; then
                echo "Attention : pont réseau non trouvé. Vérifier la config réseau ..."
                exit 1
        fi
        /usr/sbin/brctl delif $PONT $IF_HOTE
        /sbin/ifconfig $PONT down
        /usr/sbin/brctl delbr $PONT
        /bin/umount /dev/shm
        /bin/mount /dev/shm
        /sbin/service network restart
}

# Point d’entrée du script
case $1 in
        start)
                start $2
        ;;
        stop)
                stop $2
        ;;
        add)
                add $2
        ;;
        *)
                echo $"Utilisation: env-qemu {start|add|stop} [interface]"
                exit 1
esac

exit 0
```


* sans oublier de le rendre exécutable par la commande :  

```bash
# chmod u+x /usr/local/bin/env-qemu
```

* A travers ce script, le pont peut être :
    * activé (« start ») avec mise en place d’un environnement d’exploitation,
    * modifié par ajout d’une interface (paramètre « add » qui sera utilisé par le script /etc/qemu-ifup),
    * arrêté (« stop ») avec remise en place de l’environnement initial.
* Par défaut, le script crée le pont sur l’interface eth0 : il est cependant possible de choisir toute autre interface opérationnelle en l’indiquant comme second paramètre. Exemple de commande :


```bash
 $ sudo env-qemu start wlan0
```

* Avec l’option « add », le script reçoit un second paramètre qui n’est autre que le nom de l’interface VTUN fraîchement créée par QEMU (exemple : tun0).

### Les droits d’exécution

* Il est nécessaire de donner aux utilisateurs le droit d’exécution pour l’interface VTUN. Le script /usr/local/bin/env-qemu s’en charge à travers la ligne :

```bash
 # Donne accès à tous sur l'interface VTUN
 /bin/chmod 666 /dev/net/tun
```

* L’utilisateur de QEMU doit aussi disposer des droits d’exécution sur le script /usr/local/bin/env-qemu : ce script appartenant à root et comportant des commandes qui lui sont propres, nous allons devoir utiliser sudo.
* A l’aide de l’outil graphique « system-config-users » également accessible par l’option « Utilisateurs et groupes » situé dans le menu « Paramètres de système », le root crée un groupe « qemu » et y insére les utilisateurs concernés.
* Ceci étant fait, nous indiquons à sudo que le groupe « qemu » a le droit d’exécuter le fameux script de root en insérant la ligne suivante dans le fichier /etc/sudoers (édition à faire sous root) :

```bash
 %qemu ALL=NOPASSWD:/usr/local/bin/env-qemu
```

## Lancement de QEMU

Roulements de tambours, nous arrivons maintenant au moment important.

### Démarrage manuel

Exemple : démarrer une session QEMU dans notre environnement d’exploitation sur mesure pour installation d’un OS sur un disque dur émulé.
Ce premier démarrage se fait en 4 temps :
*  création du fichier image du disque dur émulé (taille = 3 Go)
*  mise en place de l’environnement d’exploitation de QEMU
*  démarrage de la session QEMU avec les paramètres suivants :
    * le disque dur émulé est stocké dans le fichier dsl.dsk
    * le lecteur de CD-ROM émulé lit l’image ISO d’un CD d’installation de DSL (une distri Linux)
    * utilisation de l’horloge du PC hôte
    * simulation du boot du PC émulé depuis le lecteur de CD-ROM émulé
    * mémoire RAM du PC émulé = 416 Mo
* restauration de l’environnement initial quand la session QEMU est terminée
Voici la trace obtenue dans un terminal :

```bash
moi@tapioca ~]$ qemu-img create dsl.dsk 3G
Formating 'dsl.dsk', fmt=raw, size=3145728 kB
[moi@tapioca ~]$ sudo env-qemu start
Utilisation de l'interface eth0
Adresse passerelle = 192.168.1.1
Adresse du pont = 192.168.1.31/24
[moi@tapioca ~]$ qemu -hda dsl.dsk -cdrom dsl-1.5.iso -localtime -boot d -m 416
connected to host network interface: tun0
[moi@tapioca ~]$ sudo env-qemu stop
Arrêt de l'interface eth0 :                                [  OK  ]
Arrêt de l'interface de loopback :                         [  OK  ]
Activation de l'interface loopback :                       [  OK  ]
Activation de l'interface eth0 :                           [  OK  ]
[moi@tapioca ~]$
```

### Démarrage simplifié

* Pour soulager l’exploitation quotidienne, il est possible de rédiger (et personnaliser) un tout petit script.
* Chaque utilisateur concerné peut donc créer/adapter un fichier « emule_pc » dans son répertoire de base à partir de la trame suivante :

```bash
#!/bin/bash
sudo env-qemu start
qemu -hda ~/dsl.dsk -localtime -m 416
sudo env-qemu stop
```

* et, bien entendu, le rendre exécutable par la commande :

```bash
$ chmod u+x ~/emule_pc
```

* Installer le pont, le configurer, paramétrer la mémoire partagée, démarrer QEMU, tout remettre en l’état se fait désormais par la simple commande utilisateur :

```bash
$ ~/emule_pc
```

* Il est même possible de créer un lanceur personnalisé dans le tableau de bord de Gnome pour ne plus avoir à passer par la ligne de commande : bref, le luxe !

## Remarques

* QEMU ne fournit pas d’émulation pour l’imprimante.
    * Sur le PC émulé, avec un OS disposant du protocole IPP (voir le site de Microsoft pour W98SE et WME), il est aisé d’utiliser n’importe laquelle des imprimantes du réseau y compris celle du PC hôte, dès lors qu’elle est partagée sous CUPS.
    * Pour Windows, définir alors une imprimante réseau à l’adresse http://IP_du_PC_avec_imprimante:631/printers/nom_CUPS_de_l_imprimante
* Actuellement, l’émulation lecteur de CD-ROM de QEMU ne gère pas les supports gravés en multi-session.
* Toutes les interfaces réseau WiFi ne supportent pas le bridge : en ce qui me concerne, j‘ai utilisé sans difficulté une carte PCI avec puce et pilote Ralink RT2500.
* Les attributs du fichiers /dev/net/tun sont parfois modifiés sans que je sache vraiment pourquoi : QEMU ne peut donc plus créer le périphérique VTUN, laissant la machine émulée sans accès réseau. J’ai remarqué que cela ne se produisait pas quand les commandes s’enchaînaient assez vite : utilisez de préférence le script ~/emule-pc.

## Documentation

* [Le site de QEMU](http://fabrice.bellard.free.fr/qemu/) 
* [Le site de Thomas Chung](http://fedoranews.org/tchung/qemu/) 
* [Le site de l’interface VTUN](http://vtun.sourceforge.net/tun/) 
* [Le site de référence sur le bridge](http://bridge.sourceforge.net/) 
* [Pont filtrant et interaction avec IPTABLES](http://ebtables.sourceforge.net/br_fw_ia/br_fw_ia.html) 
