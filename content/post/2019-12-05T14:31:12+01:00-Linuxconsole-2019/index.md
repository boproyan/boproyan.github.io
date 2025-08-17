+++
title = 'LinuxConsole 2019'
date = 2019-12-05T14:31:12+01:00 00:00:00 +0100
categories = cli
+++
LinuxConsole 2019 est disponible au téléchargement, plus d’un an après [la sortie de sa version précédente](https://linuxfr.org/news/linuxconsole-2018). 


Cette distribution GNU/Linux est particulièrement adaptée à une utilisation familiale. Elle se veut simple d’utilisation, tout en étant complète. On peut l’essayer via un média USB autonome, et aussi l’installer sur un disque dur. Elle est également utilisable sur des ordinateurs disposant de peu de ressources, grâce à un bureau assez léger ([MATE](https://mate-desktop.org/fr/)). Cette dépêche détaille les nouveautés de cette version et les façons de l’installer et de s’en servir.

Nouveautés 2019
---------------
- noyau 5.4.5 avec prise en charge native de l’exFAT ;
- construite avec [YourDistroFromScratch](https://bitbucket.org/yourdistrofromscratch/ydfs/src/master/) 2.7, en utilisant Docker ;
- ISO hybride ;
- outil de contrôle parental permettant de limiter le temps d’écran pour les enfants ;
- modularité ;
- environnement de bureau MATE 1.22 ;
- Chromium devient le navigateur Web par défaut ;
- Wine 4.0.3.

----

[linuxconsole.org](https://www.linuxconsole.org)
[Dépêche sur LinuxConsole 2018](https://linuxfr.org/news/linuxconsole-2018)

----

# Installation
## Installer LinuxConsole sur une clef USB
L’image ISO est maintenant « hybride », ce qui fait que l’on peut utiliser l’outil [Etcher](https://www.balena.io/etcher/) pour copier les images ISO. Il est également possible d’utiliser des logiciels comme [UNetbootin](https://unetbootin.github.io/) ou [Rufus](https://rufus.akeo.ie/) (ce dernier étant recommandé, car il gère l’écriture en mode hybride ou standard).
    
Une fois la clef USB générée, vous pouvez l’essayer ; et s’il y a des types de logiciels « particuliers » qui vous intéressent, vous pouvez télécharger des modules (au format SquashFS) depuis cet emplacement : [jukebox.linuxconsole.org/modules](http://jukebox.linuxconsole.org/modules/), puis les installer dans le dossier « modules » de la clef USB. Ceci illustre bien le côté « modulaire » de cette distribution.


## Installer LinuxConsole sur un disque dur
### 1 – Double amorçage avec Windows XP
Il est possible d’utiliser la version « 32 bits » pour facilement créer un double amorçage, à partir de Windows XP (sur des très vielles machines !). Pour cela, il suffit de télécharger l’[installateur](http://jukebox.linuxconsole.org/linuxconsole/wubi/linuxconsole.exe) au même endroit que l’image ISO 32 bits, et de lancer celui‑ci. Cet installateur est une divergence (_fork_) de [Wubi](https://doc.ubuntu-fr.org/wubi).

### 2 – Installation sur un disque dur vierge
Si le disque dur n’est pas vierge, vous pouvez utiliser l’outil « Gparted », disponible en mode autonome, pour effacer les partitions (il faudra redémarrer ensuite).
    
Pour installer LinuxConsole sur un disque vierge, il faut cliquer sur l’icône « Installer sur le disque dur » présente sur le bureau, puis créer une partition ext4 nommée « linuxconsole » via Gparted (après avoir créé la table de partitionnement au besoin). Le script installe le chargeur d’amorçage ou _boot loader_ (GRUB), le noyau, le système de fichiers d’initialisation (initramfs) et les modules. Cette opération est plutôt rapide (copie de quelques fichiers), vous pouvez ensuite redémarrer l’ordinateur.


### 3 – Double amorçage avec Ubuntu
Sur des ordinateurs récents, le double amorçage est assez délicat, c’est pour cela qu’il est recommandé d’installer Ubuntu en double amorçage avec Windows, dans un premier temps, puis lancer (en super‐utilisateur _root_) le script suivant :


## Procédure
### Détection de la partition à utiliser pour l’installation, on peut le faire en dur si nécessaire
    ROOTFS=`mount |grep ext4|cut – d' ' – f1|head – 1`
    # Label sur la partition
    e2label $ROOTFS linuxconsole || exit 1
    # ISO à utiliser
    ISO=linuxconsole-x86_64.iso
    # Test avant de continuer
    [! – e "$ISO" ] && echo "You need $ISO" && exit 1


### Création du dossier de montage temporaire
    install – d /media/iso
    mount $ISO /media/iso || exit 1


### Copie des fichiers sur le disque
    cp /media/iso/isolinux/kernel /boot/vmlinuz-4-linuxconsole || exit 1
    cp /media/iso/isolinux/initramfs /boot/initrd.img-4-linuxconsole || exit 1
    cp – fR /media/iso/modules/ / || exit 1


### Mise à jour du chargeur d’amorçage
    update-grub



### Active la persistance de données 
Voir le document dans `ydfs/start/restore-persistant-data`.
    
    touch /syslinux.cfg



### Démontage
    umount /media/iso
    
On aura ainsi un « amorçage de test ou _trial boot_ », avec Windows, Ubuntu et LinuxConsole.
    
_N. B. : les `/home` d’Ubuntu peuvent être partagés avec ceux de LinuxConsole ; pour cela, il suffit de créer un utilisateur avec le même nom._

Limiter le temps de connexion
===========
Pour ceux qui veulent limiter le temps de connexion pour des utilisateurs particuliers, un outil spécifique a été créé : `plogoff`. En tant que super‐utilisateur, il faut cliquer sur le menu « Configurer le temps de connexion des enfants », et vous serez alors invité à définir, en heures, le temps prévu, pour chacun des enfants (il faudra auparavant créer leurs comptes d’utilisateur).
    
Une fois le temps écoulé, la session se fermera automatiquement (avec des avertissements avant), il sera possible de se reconnecter, mais pour une minute seulement.


Modularité
==========
Les modules sont construits par type d’usage, depuis un fichier source. Ces fichiers sont [disponibles](http://jukebox.linuxconsole.org/modules/sources/). Des ajouts et mises à jour seront disponibles courant 2020. La construction de modules « personnalisés » est possible, avec l’outil [yourdistrofromscratch](https://bitbucket.org/yourdistrofromscratch).
    
Il y a donc aujourd’hui neuf modules disponibles :
    
- emulators ;
- fps ;
- games ;
- graphics ;
- multimedia ;
- music ;
- network ;
- office ;
- video.


Les jeux
--------
LinuxConsole a plusieurs jeux préinstallés. Pour en avoir plus, vous pouvez installer les modules « games » et « FPS ». Il est également possible d’installer des jeux (payants mais sans DRM) proposés par la plate‑forme _[Gog.com](https://www.gog.com/)_.


La musique
----------
Pour les amateurs de [[MAO]], LMMS a été compilé avec la gestion native des VST Windows, ce qui en fait un outil évolutif. De nombreuses extensions sont disponibles aux formats LV2, VSTL et DSSI. Des outils comme Ardour, zyn-fusion ou Helm sont également disponibles.


Installer des programmes supplémentaires
----------------------------------------
LinuxConsole est donc proposé avec un nombre de programmes limité, mais qui se veulent efficaces dans leur domaine. Si vous voulez installer un programme qui n’est pas disponible par défaut, ou dans les modules, il y a plusieurs possibilités :


### Wine
Il est possible d’installer des programmes Windows avec [Wine](https://www.winehq.org/). Par exemple, pour installer Scratch, il suffit de télécharger un [installateur](http://download.scratch.mit.edu/ScratchInstaller1.4.exe) et de le lancer avec Wine.

### opkg
Le gestionnaire de paquets de LinuxConsole est [opkg](https://openwrt.org/docs/guide-user/additional-software/opkg). Il permet d’installer des paquets optionnels, dont les sources sont dans [ce fichier](https://bitbucket.org/yourdistrofromscratch/ydfs/src/master/2.7/packages/list-opkg-linuxconsole) (qui devrait grossir au cours de l’année 2020).


Son utilisation est assez simple :
    
    opkg-cl update
    opkg-cl list
    opkg-cl install (nom du paquet)


### apt-get install
Un programme intermédiaire (_wrapper_) est disponible, pour installer des binaires de chez Debian (_Buster_). Ainsi, en faisant `apt-get install teeworlds` (en super‐utilisateur) et en acceptant les paquets proposés, Teeworlds sera installé avec un lien pour le lancer dans le menu.
