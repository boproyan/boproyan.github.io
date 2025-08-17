+++
title = 'Debian-Stretch-Xfce-eeePC-1001HA'
date = 2018-11-23 00:00:00 +0100
categories = []
+++
2017-06-14-Debian-Stretch-Xfce-eeePC-1001HA
========================

---
layout: article
title: Debian 9 (stretch) XFCE sur portable Asus eeepc1001ha (noir)
toc: true
ref: (falcutatif)
date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23
tags: [archlinux]
lang: fr
description:  Debian 9 (stretch) XFCE sur portable Asus eeepc1001ha (noir)
---

# eeePC 1001HA (Noir) 

fichier : 2017-06-14-Mint-Xfce-eeePC-1001HA.md

# Matériel

  * CPU: Intel Atom N270 @ 1.60 GHz    
  * RAM: `2 GB DDR2 533 MHz SODIMM`, 1 slot  
  * Graphics: Intel GMA950, integrated in the northbridge  
  * Sound: Intel HD Audio, two integrated speakers, microphone, headphones out, microphone in  
  * Chipset: Intel 945GSE northbridge, ICH7 southbridge  
  * Hard disk: `SSD Crucial 512 GB`   
  * Display: 10.1", LED backlight, resolution: 1024x600  
  * Wired network: Atheros AR8132 10/100 Fast Ethernet  
  * Wireless: Ralink RT3090 802.11n, mini PCIe card  


---
layout: article
title:  Asus eeepc900a (blanc) Debian 9 stretch 32bits + XFCE
toc: true
ref: (falcutatif)
date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23
tags: [debian]
lang: fr
description:  Debian 9 (stretch) XFCE sur Portable eeepc900a (blanc)
---

# Ordinateur portable eeepc900a

image_tag src="/images/eeepc900a-blanc.png" width="150" %}

## Matériel

 * Processeur : Intel® Atom™ N270 de 1,6 Ghz ;
 * Ethernet : Atheros AR8216 (10/100 Mbit/s) ;
 * Wifi : Atheros AR5007EG (802.10b/g) ;
 * SSD 64GB SATA Mini PCIE Kingspec SSD Only for ASUS Eee PC 1000 S101 900 901 900A ;
 * Mémoire vive : 2 Go DDR2 667
 * Wifi : Atheros AR5007EG (802.11b/g) ;
 * Carte graphique : Intel GMA 900 ;
 * Carte son : Realtek ALC 662 ;
 * WebCam : 0,3 Megapixels ;
 * Écran :
   * Diagonale : 8,9 pouces,
   * Résolution : 1024x600,
   * Format : 16/10.
 * Périphériques de sortie ;
   * 3 Port USB,
   * 1 Port VGA,
   * Lecteur de Cartes mémoires SD/SDHC.
 * Batterie :
   * 6600 mAh, 3 ~ 4 heures


## Debian 9 (stretch) sur eeepc900a

hostname : eeepc-900a
root : ytreu49
eeepc : eeepc49
disque ssd 64GB
ram 2Go

### Installation

* Environnement de bureau Debian
* Xfce
* serveur d'impression
* serveur SSH
* Utilitaires usuels du système

### Première connexion 

Relever adresse IP , ouvrir un terminal

	ip addr    # 192.168.0.41

Se connecter via SSH depuis un autre poste du réseau

	ssh eeepc@192.168.0.41

Installer **sudo** et modifier **/etc/sudoers** pour accès root sans mot de passe utilisateur **eeepc**  

	su
	apt install sudo
	echo "eeepc     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers


Télécharger les images  **~/Images/yannick/eeepc/** pour personnalisation XFCE

```
archlinux1024x600.jpg
awesome-freevector-1024x600.jpg
black-background-1024x600.jpg
black_lipstick_monk_1024x600ym.png
carbon_material_dark_eeepc900a_1024x600.jpg
triangle_inverted_black_white_92770_1024x600.jpg
yannick53x64.png
yannick-green.png
yannick-green.svg
carbon.tga
debian_dark_wallpapers_hd_1080.jpg
```

Copier les images dans les bons dossiers  

```
cd ~
sudo mv ./{debian_dark_wallpapers_hd_1080.jpg,black_lipstick_monk_1024x600ym.png,black-background-1024x600.jpg,carbon_material_dark_eeepc900a_1024x600.jpg} /usr/share/backgrounds/
sudo mv ./{archlinux1024x600.jpg,triangle_inverted_black_white_92770_1024x600.jpg} /usr/share/backgrounds/xfce/
sudo mv ./{yannick53x64.png,yannick-green.png,yannick-green.svg} /usr/share/pixmaps/
```

### Grub

Image de grub

	sudo apt install grub2-splashimages
	sudo mv carbon.tga /usr/share/images/grub/

Ecran de la page de démarrage **grub**  

	sudo nano /etc/default/grub

```
GRUB_BACKGROUND="/usr/share/images/grub/carbon.tga"  
```

Reconfigurer grub pour la prise en charge de l'image  

	sudo update-grub

>NOTE : conversion jpg en tga ,paquet **imagemagick** installé  
`convert image.jpg image.tga` 



## Xfce

### Personnaliser xfce

compléments  menu libre et whisker

	sudo apt install menulibre xfce4-whiskermenu-plugin

#### icônes ,thèmes et styles

Installer des icônes et thèmes  
Pour le thème du bureau, **Numix**

```
# Installation des pré-requies
sudo apt-get install gtk2-engines-murrine murrine-themes
# Installation de Numix
sudo apt install numix-gtk-theme numix-icon-theme moka-icon-theme
# Installation de Numix-circle
wget https://github.com/numixproject/numix-icon-theme-circle/archive/master.zip
unzip master.zip
sudo mv numix-icon-theme-circle-master/Numix-Circle /usr/share/icons/
sudo mv numix-icon-theme-circle-master/Numix-Circle-Light /usr/share/icons/
rm -r numix-icon-theme-circle-master/
rm master.zip
sudo gtk-update-icon-cache /usr/share/icons/Numix-Circle
sudo gtk-update-icon-cache /usr/share/icons/Numix-Circle-Light
```

Menu Paramètres --> Apparence  

  * Style    : Xfce-flat  
  * Icônes   : Numix Circle
  * Polices  : Droid Sans Fallback 10
  * Rendu
     * activer l'anti-crénélage 
     * Lissage **Léger**
     * Ordre de sous-pixellisation : **RVB**
  * DPI
     * Paramètre DPI personnalisé : **90** 

Menu Paramètres --> Gestionnaire de fenêtres -> Style : **Numix**  

#### tableau de bord

Modification du **tableau de bord** , clic-droit --> Tableau de bord --> Préférences de tableau de bord  
Tableau de bord 1  

  * Onglet **Eléments** 
      * Ajout **Menu whisker** icône */usr/share/pixmaps/yannick-green.svg* (supprimer "Menu des applications")  
      * Ajout **Mise à jour météo** ,Greffon PulseAudio  
      * Ajout **Captures d'écran** 
      * Ajout **Afficher le bureau**
      * Ajout **Moniteur de batteries**
      * Modifier **Horloge** par défaut avec Affichage date et heure, **format personnalisé** : %e %b %Y %R  
      * Positionner **Boutons d'action** dans apparence , actions `Déconnexion` (décocher tout le reste) 
      * Supprimer changeur espace de travail 
  * Onglet **Affichage**
      * Verrouiller tableau de bord
      * Masquer automatiquement le tableau de bord **Toujours**  
      * Taille d'une ligne (pixels) **25**   

Tableau de bord 2 à **supprimer**


Après validation tableau de bord, clic droit sur icône capture écran puis  Propriétés -> Zone à capturer : Sélectionner une zone

image_tag src="/images/xfce-tableau-bord1.png" width="300" %}



### Lightdm (display manager)

Utilisé avec Debian stretch  
Fichiers de configuration (.conf) sous **/etc/lightdm**  

Modification fond d'écran et avatar (icône utilisateur à la connexion)  

    nano /etc/lightdm/lightdm-gtk-greeter.conf

```
[greeter]
background=/usr/share/backgrounds/eeepc.jpg
user-background=/usr/share/backgrounds/yannick53x64.png
```

Pour activer la liste des utilisateurs , modifier le fichier par défaut **/usr/share/lightdm/lightdm.conf.d/01_debian.conf** et positionner *greeter-hide-users = false*

```
# Debian specific defaults
#
# - use lightdm-greeter session greeter, points to the etc-alternatives managed
# greeter
# - hide users list by default, we don't want to expose them
# - use Debian specific session wrapper, to gain support for
# /etc/X11/Xsession.d scripts

[Seat:*]
greeter-session=lightdm-greeter
greeter-hide-users=false
session-wrapper=/etc/X11/Xsession

```

Relancer lightdm

    sudo service lightdm restart


### Applications et outils

	sudo apt install terminator filezilla nmap minicom zenity dnsutils curl git

Pour la compilation (Facultatif)

    sudo apt install build-essential

Créer un dossier .ssh pour stocker les différentes clés et un dossier pour la base des mots de passe (keepassx) synchronisée sur le cloud    

	cd ~ && mkdir .ssh && mkdir .keepassx

La clé keepassx `yannick.key` est stockée dans le dossier **.ssh/**  

Dans les **Applications favorites**  onglet Utilitaires -> Emulateur de terminal */usr/bin/terminator "%s"*

#### keepassx

Avec debian , keepassx2 est installé par défaut  , on veut la version 0.4

```
cd ~
wget http://ftp.fr.debian.org/debian/pool/main/k/keepassx/keepassx_0.4.3+dfsg-0.1+deb8u1_i386.deb
sudo dpkg -i keepassx_0.4.3+dfsg-0.1+deb8u1_i386.deb
```

#### owncloud client

Pour Debian 9.0 (stretch)

	sudo apt install owncloud-client

Paramétrage  

  * menu --> Lancer "Owncloud..."
  * Adresse du serveur : **https://xeuyakzas.xyz/owncloud**  
  * Nom d'utilisateur : xeuyak  
  * Mot de passe : xxxxx  
  * Sauter les dossiers à synchroniser
  * Trousseau de clés ,mot de passe connexion utilisateur   

Sélectionner l'icône **xeuyak** puis **Ajouter une synchronisation de dossier**   


  * Dossier local **/home/eeepc/.keepassx** (valider par clic droit , option "Afficher les fichiers cachés")  
  * Répertoire distant : **.keepassx**  
  * Cliquer sur **Ajouter une synchronisation**  

>**BUG** : l'icône owncloud est présent mais invisible dans la barre des tâches

On peut lancer le gestionnaire de mot de passe **keepassx**  , base **~∕.keepassx/yannickdb.kdb** et clé **~/.ssh/yannick.key**  

#### Impression

Imprimante HP officejet 6700  
Si l'imprimante est connectée directement à votre système ou si vous avez accès à une imprimante réseau IPP 

Menu -> configuration de l'impression -> Ajouter une imprimante  
Imprimante réseau -> Hp Officejet 6700... -> AppSocket/HP JetDirect

#### Thunar (Recherche)

Gestionnaire de fichier **Thunar**

  * Edition --> Préférence ,ongle **Comportement** *Ouvrir le dossier dans un nouvel onglet*

image_tag src="/images/thunar-prefer1.png" width="300" %}

Fonction recherche ,installer les logiciels

    sudo apt install catfish mlocate

  * Edition --> Configurer les actions personnalisées ,ajouter (+)

image_tag src="/images/thunar-prefer2.png" width="300" %}image_tag src="/images/thunar-prefer3.png" width="300" %}

## Chiffrement (eCryptfs)

**Ecryptfs ** est un outil pour créer un dossier privé (**~/Private**), *chiffré et inaccessible aux autres utilisateurs* , il est destiné à contenir tous les fichiers "sensibles" que vous pourriez avoir : vos fichiers contenant des mots de passe, les données confidentielles relatives à vos comptes bancaires, vos emails...  

**A-Installer ecryptfs**  

Installer le paquet **ecryptfs-utils**.  

	sudo apt install ecryptfs-utils

Une fois ces paquets installés, vous devez charger le module **ecryptfs**

	sudo modprobe ecryptfs


**B-Configuration**  

Générer les fichiers   

	ecryptfs-setup-private

```
Enter your login passphrase [eeepc]:    (mot de passe connexion utilisateur)  
Enter your mount passphrase [leave blank to generate one]:

************************************************************************
YOU SHOULD RECORD YOUR MOUNT PASSPHRASE AND STORE IT IN A SAFE LOCATION.
  ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase
THIS WILL BE REQUIRED IF YOU NEED TO RECOVER YOUR DATA AT A LATER TIME.
************************************************************************


Done configuring.

Testing mount/write/umount/read...
Inserted auth tok with sig [5cd18c8a613126ed] into the user session keyring
Inserted auth tok with sig [aec953a693d37047] into the user session keyring
Inserted auth tok with sig [5cd18c8a613126ed] into the user session keyring
Inserted auth tok with sig [aec953a693d37047] into the user session keyring
Testing succeeded.

Logout, and log back in to begin using your encrypted directory.

```

Entrez votre mot de passe de connexion, puis entrez un mot de passe de montage ou de le laisser vide pour générer automatiquement un sécurisé, déconnectez-vous et connectez-vous de nouveau et fait!   
Déplacez maintenant vos documents secrets vers le dossier **Private**.

Maintenant, le système de fichiers chiffré est monté, pour monter et démonter le répertoire privé crypté, utilisez les commandes ci-dessous.

	ecryptfs-umount-private  # to un mount
	ecryptfs-mount-private    # to mount it again

Pour crypter n'importe quel autre répertoire, déplacez simplement le répertoire vers le répertoire privé (**Private**), vous pouvez créer un lien vers ce répertoire pour un accès facile.

	mv ~/secret/ ~/Private/  # déplacer le dossier secret
	ln -s ~/Private/secret/ ~/secret/ # créer le lien symbolique pour un accès facile

**C-Sauvegarde clé (passphrase)** 

Dans le cas d'une génération automatique de la passphrase de montage , il faut la déchiffrer pour pouvoir la sauvegarder  

	ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase

Passphrase: `Mot de passe de connexion utilisateur`  
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX  
Enregistrez votre phrase secrète dans un lieu sûr, elle sera requise pour récupérer vos données ultérieurement.  

**D-éviter le montage auto de ecryptfs  lors de la connexion**

Par défaut, le dossier Privé (**Private**) est automatiquement monté après le login, pour éviter cette fonctionnalité ,passer l'argument **--noautoumount** lors de l'installation

	ecryptfs-setup-private --noautoumount

**E-Chiffrer le  répertoire /home**

Pour configurer un répertoire de base chiffré sans aucun problème, déconnectez-vous de la session en cours, connectez-vous sur un autre  utilisateur  (c'est-à-dire en tant que root), installez **rsync** et **lsof** puis exécuter la commande en tant qu'utilisateur root `ecryptfs-migrate-home -u`

Un exemple avec Debian.

	sudo apt-get install lsof  # install lsof
	sudo apt-get install rsync # install rsync
	sudo ecryptfs-migrate-home -u b00m  # setup encrypted home, b00m is the username




## Paramétrage des navigateurs

Par défaut , sur une une installation Debian Jessie Xfce , le navigateur est **firefox ESR **   
Dans les **Applications favorites**  onglet internet -> Navigateur Web *Navigateur Sensible Debian*

### Firefox

Préférences:  

  * Au démarrage de firefox : **Afficher une page vide**  
  * Moteur de recherche par défaut : **DuckDuckGo**  
  * Contenu : **Polices et couleurs serif 12** (écran eeePC 1024x600)
  * Vie privée : **ne jamais conserver l'historique** (firefox doit redémarré)  

Modules:  

  * **Ghostery** , **Ublock Origin** et **TrackMeNot** (nécessite un redémarrage firefox)

Ajout de moteur de recherche:

**Quant** : rechercher dans “Modules” de firefox ,l'expression `qwant fr`. 
Sélectionner `Qwant pour firefox` , cliquer sur le bouton **Ajouter à Firefox** et il sera dans la liste des moteurs de recherche du navigateur.

**Ixquick** : rechercher dans “Modules” de firefox ,l'expression `ixquick fr`.  
Sélectionner `New Ixquick HTTPS -…` , cliquer sur le bouton **Ajouter à Firefox** et il sera dans la liste des moteurs de recherche du navigateur.

### Midori

>**Midori n'est pas installé par défaut dans Debian Jessie/Stretch**

Installation

    sudo apt install midori

Utiliser **Midori (Navigation privée)**

Dans les barres apparentes (clic droit en haut à gauche de l'écran), on laisse uniquement la **Barre de navigation**

Ajouter un moteur de recherche  
Attention , modifier sur **Midori** classique et non sur **Midori (navigation privée)** pour que les paramètres soient pris en compte.  

 * Cliquer sur l'icône dans le champ de recherche ,puis sur **Gestion des moteurs de recherche"
 * Cliquer sur **Ajouter**
    * Nom : ixquick
    * Description : 
    * Adresse https://www.ixquick.com/do/dsearch?cat=web&pl=opensearch&language=francais&query=
    * Identifiant : ix

Sélection ix et cliquer sur **Par défaut** et **Fermer**  
**ixquick** sera le moteur de recherche par défaut

## VPN

Il faut installer les modules  

	sudo apt install network-manager-openvpn network-manager-openvpn-gnome

Pour ajouter une connexion VPN à partir d'un fichier** .ovpn **  
Clic droit sur la connexion réseau dans la barre des tâches -> Modification des connexions -> Add -> Importer une configuration VPN enregistrée  

## Tor

Installation

    sudo apt install tor

Lancement

    sudo service tor start

Navigateur Firefox

Il faut paramétrer Firefox pour utiliser Tor  
Préférences -> Avancé , onglet Réseau et Paramètres  
Sélectionner **Configuration manuelle du proxy**  
Hôte SOCKS : **127.0.0.1**   Port : **9050**  
Pas de proxy pour : **localhost,127.0.0.1**  
Valider par clic sur OK

Vérification par lien <https://check.torproject.org/?lang=fr>  
**Félicitations. Ce navigateur est configuré pour utiliser Tor** est le message affiché dans le cas d'un fonctionnement correct du service tor

## VncClient

Installation client/serveur  

```
sudo pacman -S tigervnc
```

Lancer vncviewer dans un terminal avec le paramètre suivant **DotWhenNoCursor=1**  pour rendre visible le curseur de la souris

```
$ vncviewer DotWhenNoCursor=1 192.168.1.68::5900
```

## Arduino IDE

* <https://wiki.debian.org/fr/Arduino>

Installation IDE

Télécharger sur le site arduino , la version 32 ou 64 (ici 32) **arduino-1.8.3-linux32.tar.xz**  
Décompresser

    tar -xvf arduino-1.8.3-linux32.tar.xz
    cd arduino-1.8.3
    sudo -s
    ./install.sh

Il se lance , soit en double-cliquant sur son icône soit en ligne de commande :

    arduino

Connecter un interface USB/Serial ou une carte Arduino via USB

    dmesg |tail

```
[  770.085325] usb 3-2: Manufacturer: FTDI
[  770.085333] usb 3-2: SerialNumber: A9QTHNJB
[  770.219665] usbcore: registered new interface driver usbserial
[  770.219759] usbcore: registered new interface driver usbserial_generic
[  770.219855] usbserial: USB Serial support registered for generic
[  770.231657] usbcore: registered new interface driver ftdi_sio
[  770.231762] usbserial: USB Serial support registered for FTDI USB Serial Device
[  770.236185] ftdi_sio 3-2:1.0: FTDI USB Serial Device converter detected
[  770.236428] usb 3-2: Detected FT232RL
[  770.247477] usb 3-2: FTDI USB Serial Device converter now attached to ttyUSB0
```

Ici la carte apparaît sous le nom ttyUSB0 dans le dossier /dev et si j'en liste le contenu, je vois que la carte appartient au groupe **dialout**. 

    ls -l /dev/ttyUSB0
    crw-rw---- 1 root dialout 188, 0 juin   5 16:29 /dev/ttyUSB0

Ajout de l'utilisateur au groupe **dialout**

    sudo usermod -a -G dialout $USER   # Se déconnecter puis se reconnecter

Au lancement de **arduino** la "carte" sera visible dans « Outils > Port série ». 

Voir ce lien , [Arduino + ESP8266 Module Wifi](post_url 2017-03-16-Arduino-ESP8266-WIFI-Module %}) , pour ajouter les cartes wifi ESP8266  

