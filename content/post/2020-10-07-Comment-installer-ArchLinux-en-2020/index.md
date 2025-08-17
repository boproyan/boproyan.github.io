+++
title = 'Comment installer Archlinux en 2020'
date = 2020-10-07 00:00:00 +0100
categories = archlinux
+++
Les étapes d'installation peuvent différer à certains moments selon [si vous avez un système UEFI ou un système BIOS ancien](https://itsfoss.com/check-uefi-or-bios/ ). De nos jours, la plupart des nouveaux systèmes sont livrés avec l'**UEFI**.

Ecrit sur une base système **UEFI**, les étapes qui sont différentes (systèmes BIOS **non-UEFI**) seront mentionnées  
*Article original : [How to Install Arch Linux [Step by Step Guide]](https://itsfoss.com/install-arch-linux/) Last updated July 18, 2020 By Abhishek Prakash*
{: .prompt-info }


## Conditions requises pour l'installation d'Arch Linux 

- Une machine compatible x86_64 (c'est-à-dire 64 bits) 
- Au moins 512 Mo de RAM (2 Go recommandés) 
- Au moins 2 Go d'espace disque disponible (20 Go recommandés pour une utilisation de base avec un environnement de bureau) 
- Une connexion Internet active 
- Un lecteur USB d'une capacité de stockage minimale de 2 Go 
- Familiarité avec la ligne de commande Linux

Une fois que vous vous êtes assuré que vous avez tous les prérequis, procédons à l'installation d'Arch Linux.

## Télécharger l'ISO d'Arch Linux 

Vous pouvez télécharger l'ISO à partir du [site officiel](https://www.archlinux.org/download/ Télécharger Arch Linux) , des liens de téléchargement direct et de torrent sont disponibles.

## Créer un USB d'Arch Linux

Vous devrez créer une clé USB d'Arch Linux live à partir de l'ISO que vous venez de télécharger.

Linux, utiliser la commande `dd` (<https://linuxhandbook.com/dd-command/>)  pour créer un USB live. 

* Remplacez ''/path/to/archlinux.iso'' par le chemin où vous avez téléchargé le fichier ISO
* ''/dev/sdx'' par votre clé USB 

>Vous pouvez obtenir des informations sur votre lecteur en utilisant la commande `lsblk` (<https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s1-sysinfo-filesystems>)

    dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress && sync

## Démarrage avec USB 

Notez que dans certains cas, vous ne pourrez pas démarrer à partir d'un port USB live lorsque le démarrage sécurisé est activé. Si c'est votre cas, désactivez d'abord le démarrage sécurisé.

Une fois que vous avez créé un USB live pour Arch Linux, éteignez votre PC. Branchez votre USB et démarrez votre système. Pendant le démarrage, continuez à appuyer sur la touche F2, F10 ou F12 (selon votre système) pour accéder aux paramètres de démarrage.

Ici, choisissez de démarrer à partir d'une clé USB ou d'un disque amovible. Une fois que vous avez fait cela et que le système a démarré, sélectionnez **Boot Arch Linux (x86_64)**. Après diverses vérifications, Arch Linux démarrera à l'invite de connexion avec l'utilisateur root.

<u>La configuration par défaut du clavier dans la session live est US</u>.   
La plupart des claviers de langue anglaise fonctionneront parfaitement, il ne peut en être de même pour les claviers français, allemands et autres.   
La liste de tous les claviers pris en charge, `ls /usr/share/kbd/keymaps/**/*.map.gz`  
Changer la disposition pour celle qui vous convient en utilisant la commande `loadkeys`.   
Par exemple, si vous voulez un clavier allemand, `loadkeys de-latin1`, français `loadkeys fr`  

## Partitionner les disques 

Pour partitionner les disques, fdisk.
  
Utilisez cette commande pour lister tous les disques et partitions de votre système :

    fdisk -l

Votre disque dur doit porter l'étiquette `/dev/sda` ou `/dev/nvme0n1` (utiliser le label de disque approprié pour votre système)

Sélectionner le disque que vous allez formater et partitionner :

    fdisk /dev/sda

Supprimer toute partition existante sur le disque en utilisant la commande `d`  
Une fois que vous avez libéré tout l'espace disque, créer de nouvelles partitions avec la commande `n`

### Vérifiez si vous avez un système compatible UEFI 

Certaines étapes sont différentes pour les systèmes **UEFI** et **non UEFI**. Vous devez vérifier si vous avez un système compatible UEFI ou non. Utilisez la commande suivante : 

    ls /sys/firmware/efi/efivars

Si ce répertoire existe, vous disposez d'un système compatible avec l'UEFI. Vous devez suivre les étapes pour le système UEFI. Les étapes qui diffèrent sont clairement mentionnées.

### Créer une partition ESP (UEFI uniquement) 

Si vous avez un système UEFI, vous devez créer une partition EFI au début de votre disque. Sinon, sautez cette étape.

Lorsque vous entrez `n`, il vous sera demandé de choisir un numéro de disque, entrez `1`  
Restez avec la taille de bloc par défaut, lorsqu'il vous demande la taille de la partition, entrez `+512M` 

Une étape importante consiste à changer le type de partition EFI en **EFI System** (au lieu de Linux).  
Entrez `t` pour changer de type. Entrez `L` pour voir tous les types de partition disponibles, puis entrez le numéro correspondant au système EFI.

### Créer une partition racine 

Vous devez créer une partition racine **à la fois pour l'UEFI et LEGACY**

La pratique courante en matière de partitionnement était/est de créer des partitions racine, swap et home séparément. Vous pouvez simplement créer une seule partition racine et [https://itsfoss.com/create-swap-file-linux/ create a swapfile] et home sous le répertoire racine lui-même.

>Ainsi, dans cette approche, nous aurons une seule partition racine, pas de swap, pas de home.

Pendant que vous êtes dans la commande fdisk, appuyez sur `n` pour créer une nouvelle partition. Elle lui donnera automatiquement le numéro de **partition 2**. Cette fois, continuez à appuyer sur la touche entrée pour allouer <u>tout l'espace disque restant</u> à la partition racine.

Lorsque vous avez terminé le partitionnement du disque, entrez la commande `w` pour écrire les changements sur le disque et sortir de la commande `fdisk` 

## Créer le système de fichiers 

Les partitions de votre disque sont prêtes, créer un système de fichiers 

### Création d'un système de fichiers (UEFI)

Vous avez donc deux partitions de disque et

le premier est de type EFI. Créez une [FAT32 file system](https://en.wikipedia.org/wiki/File_Allocation_Table) dessus :

    mkfs.fat -F32 /dev/sda1

Créez maintenant un système de fichiers Ext4 sur la partition racine :

    mkfs.ext4 /dev/sda2

### Création d'un système de fichiers (non-UEFI) 

Pour les systèmes **non-UEFI**, vous n'avez qu'une seule partition racine. Il suffit donc de la rendre ext4:

    mkfs.ext4 /dev/sda1

## Réseau

### LAN

Si connexion via LAN  `ip a` pour récupérer l'adresse

### WiFi 

Vous pouvez vous connecter au WiFi de manière interactive en utilisant cet utilitaire utile appelé `iwctl`. Il suffit d'entrer cette commande et de suivre les étapes :

    iwctl

Vous devez pouvoir voir les connexions actives et vous y connecter à l'aide du mot de passe.  
Une fois que vous êtes connecté, vérifiez si vous pouvez utiliser Internet en utilisant la commande ping :

    ping google.com

Si vous obtenez des octets en réponse, vous êtes connecté. Utilisez Ctrl+C pour arrêter la réponse ping.

## Valider SSHD

Lancer le service SSH

    systemctl start sshd

Modifier le mot de passe

    passwd 

Connexion depuis un poste sur le réseau : `ssh root`

## Sélectionnez un miroir approprié 

C'est un gros problème pour l'installation d'Arch Linux. Si vous continuez à l'installer, vous pourriez trouver que les téléchargements sont beaucoup trop lents. Dans certains cas, il est si lent que le téléchargement échoue.

C'est parce que la liste des miroirs (située dans **/etc/pacman.d/mirrorlist**) contient un grand nombre de miroirs, mais pas dans le bon ordre. Le miroir du haut est choisi automatiquement et n'est pas toujours un bon choix.  
Heureusement, il existe 2 solutions à ce problème.  

Commencez par synchroniser le dépôt pacman afin de pouvoir télécharger et installer des logiciels :

    pacman -Syy

Faites une sauvegarde de la liste des miroirs (au cas où) :

    cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak

**Solution 1 (A privilégier)**  
Installer pacman-contrib et curl

    pacman -S pacman-contrib curl

Utilisation de **rankmirrors** pour tester le miroir le plus proche de chez vous. La commande ci-dessous permet de tester les miroirs en Belgique, France, Hollande et Allemagne et prendre les 5 plus rapides

    curl -s "https://www.archlinux.org/mirrorlist/?country=FR&country=NL&country=BE&country=GE&protocol=https&use_mirror_status=on" | sed -e 's/^#Server/Server/' -e '/^#/d' | rankmirrors -n 5 - > /etc/pacman.d/mirrorlist

**Solution 2**  
Maintenant, installez également un réflecteur que vous pourrez utiliser pour répertorier les miroirs frais et rapides situés dans votre pays :

    pacman -S reflector

Maintenant, obtenez la bonne liste de miroirs avec réflecteur et enregistrez-la dans la liste de miroirs. Vous pouvez changer le pays des États-Unis en votre propre pays.

    reflector -c "FR" -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist



## Installation d'Arch Linux 

Puisque vous avez tout préparé, il est temps d'installer enfin Arch Linux. Vous allez l'installer sur le répertoire racine, donc montez le d'abord.

la partition racine 

    mount /dev/sda1 /mnt    # NON efi
    swapon /dev/sda2        # NON efi
    mount /dev/sda2 /mnt    # efi

Une fois la racine montée, il est temps d'utiliser le merveilleux [pacstrap script](https://git.archlinux.org/arch-install-scripts.git/tree/pacstrap.in) pour installer tous les paquets nécessaires :

    pacstrap /mnt base base-devel linux linux-firmware nano

Il faudra un certain temps pour télécharger et installer ces paquets. Si les téléchargements sont interrompus, pas de panique. Vous pouvez exécuter la commande ci-dessus une fois de plus et le téléchargement reprendra.

J'ai ajouté l'éditeur de texte **Nano** à la liste car vous devrez modifier certains fichiers après l'installation.

## Configurer le système Arch installé 

Générer un fichier [fstab](https://en.wikipedia.org/wiki/Fstab) pour définir comment les partitions de disque, les périphériques de bloc ou les systèmes de fichiers distants sont montés dans le système de fichiers.

    genfstab -U /mnt >> /mnt/etc/fstab

Utilisez maintenant [arch-chroot](https://wiki.archlinux.org/index.php/Chroot#Using_arch-chroot) et entrez le disque monté comme racine. En fait, vous utilisez maintenant le système Arch Linux qui vient d'être installé sur le disque. Vous devrez faire quelques changements de configuration sur le système installé afin de pouvoir l'exécuter correctement lorsque vous démarrerez à partir du disque.

    arch-chroot /mnt

### Réglage du fuseau horaire 

Vous pouvez utiliser la commande timedatectl.  
Trouvez d'abord votre fuseau horaire :

    timedatectl list-timezones

Puis, configurez-le comme suit (remplacez Europe/Paris par le fuseau horaire de votre choix) :

    timedatectl set-timezone Europe/Paris

### Mise en place d'un "local" (fr_FR.UTF-8) 

C'est ce qui détermine la langue, la numérotation, la date et les formats de monnaie de votre système.  
Le fichier **/etc/locale.gen** contient tous les paramètres locaux et la langue du système dans un format commenté.  
Ouvrez le fichier en utilisant l'éditeur Nano, `nano /etc/locale.gen`, et décommentez (enlevez le # du début de la ligne) la langue que vous préférez (fr_FR.UTF-8 UTF-8).   

Générez maintenant la configuration locale dans le fichier du répertoire /etc en utilisant les commandes ci-dessous une par une :

    locale-gen

localisation française, le fichier /etc/locale.conf doit contenir la bonne valeur pour LANG

    echo LANG=fr_FR.UTF-8 > /etc/locale.conf  

Vous pouvez spécifier la locale pour la session courante (ça évitera des messages d'alerte par la suite) 

    export LANG=fr_FR.UTF-8

Les paramètres de la locale et du fuseau horaire peuvent être modifiés ultérieurement, même si vous utilisez votre système Arch Linux.

Si vous définissez la disposition du clavier, rendez les changements persistants dans **vconsole.conf** 

	nano /etc/vconsole.conf

Ajouter  

```
KEYMAP=fr-latin9
FONT=eurlatgr
```


### Configuration de noms d'hôtes

Créez le fichier de noms d'hôtes **/etc/hostname** et y ajouter l'entrée  
hostname est le nom de votre ordinateur sur le réseau.

Dans mon cas, je vais définir le nom d'hôte comme **myarch**.

    echo myarch > /etc/hostname

La partie suivante consiste à créer le fichier hôte :

    touch /etc/hosts

Et modifiez ce fichier **/etc/hosts** avec l'éditeur Nano, `nano /etc/hosts`, pour y ajouter les lignes suivantes (remplacez myarch par le nom d'hôte que vous avez choisi plus tôt) :

```
127.0.0.1 localhost
::1 localhost
127.0.1.1 myarch
```

### Mot de passe root 

Vous devez également définir le mot de passe du compte root à l'aide de la commande passwd :

    passwd

### Créer un utilisateur

Utilisateur + mot de passe

    useradd -m -d /home/userarch/ -s /bin/bash -c "Utilisateur arch" userarch
    passwd userarch  

### Installer SSH

Pour une connexion à partir d'un autre poste sur le réseau

    pacman -S openssh

## Installer le chargeur de démarrage Grub 

C'est l'une des étapes cruciales et elle diffère pour les systèmes UEFI et non UEFI. 

### Installer grub sur les systèmes UEFI. 

Assurez-vous que vous utilisez toujours de la racine d'arche. Installez les paquets nécessaires :

    pacman -S grub efibootmgr

Créez le répertoire où la partition EFI sera montée :

    mkdir /boot/efi

Maintenant, montez la partition ESP que vous aviez créée

    mount /dev/sda1 /boot/efi

Installez grub comme ceci :

    grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi

Un dernier pas :

    grub-mkconfig -o /boot/grub/grub.cfg

### Installer grub sur les systèmes non-UEFI 

Installez d'abord le paquet grub : 

    pacman -S grub

Et puis installez grub comme ceci (ne mettez pas le numéro de disque sda1, juste le nom de disque sda) :

    grub-install /dev/sda

Dernière étape :

    grub-mkconfig -o /boot/grub/grub.cfg

---

## Installer un environnement de bureau

La première étape consiste à installer l'environnement X. Tapez la commande ci-dessous pour installer le Xorg comme serveur d'affichage.

    pacman -S xorg xorg-server

Maintenant, vous pouvez installer l'environnement de bureau GNOME sur Arch Linux en utilisant :

    pacman -S gnome

La dernière étape consiste à activer le gestionnaire d'affichage GDM pour Arch. Je suggère également d'activer le gestionnaire de réseau

    systemctl start gdm.service
    systemctl enable gdm.service
    systemctl enable NetworkManager.service

Sortez maintenant du chroot en utilisant la commande exit :

    exit

Et ensuite, arrêtez votre système

    shutdown now

N'oubliez pas de retirer la clé USB avant de rallumer le système. Si tout se passe bien, vous devriez voir l'écran Grub puis l'écran de connexion à GNOME.

Je vous recommande les [choses essentielles à faire après l'installation](https://itsfoss.com/things-to-do-after-installing-arch-linux/) où vous trouverez les étapes à suivre pour installer divers autres environnements de bureau et en apprendre davantage sur le système d'exploitation. 


[How to Install Arch Linux [Step by Step Guide]](https://itsfoss.com/install-arch-linux/)
