+++
title = 'Asus-eeepc1001ha'
date = 2018-11-23 00:00:00 +0100
categories = []
+++
2017-05-30-Asus-eeepc1001ha
==========================

---
layout: article
title:  Asus eeepc1001ha (noir)
toc: true
ref: (falcutatif)
date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23
tags: [debian]
lang: fr
description:  Debian 9 stretch 32bits + XFCE sur Asus eeepc1001ha (noir)
---

# Ordinateur portable Asus eeepc1001ha

image_tag src="/images/eeepc1001ha.png" width="150" %}

## Matériel

  * CPU: Intel Atom N270 @ 1.60 GHz    
  * RAM: `2 GB DDR2 533 MHz SODIMM`, 1 slot  
  * Graphics: Intel GMA950, integrated in the northbridge  
  * Sound: Intel HD Audio, two integrated speakers, microphone, headphones out, microphone in  
  * Chipset: Intel 945GSE northbridge, ICH7 southbridge  
  * Hard disk: `SSD Crucial 512 GB`   
  * Display: 10.1", LED backlight, resolution: 1024x600  
  * Wired network: Atheros AR8132 10/100 Fast Ethernet  
  * Wireless: Ralink RT3090 802.11n, mini PCIe card  

# Debian 9 stretch 32bits + XFCE

Utilisation clé USB net-install Debian stretch avec le firmware wifi Ralink

hostname : eeepc1001ha
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

	ip addr    # 192.168.0.19

Se connecter via SSH depuis un autre poste du réseau

	ssh eeepc@192.168.0.19

Passage en mode super utilisateur su

    su


Forcer l'utilisation des serveurs IPV4 pour les dépôts (facultatif)

    nano /etc/apt/apt.conf.d/99force-ipv4
        Acquire::ForceIPv4 "true";

Mise à jour

    apt update && apt upgrade

Installer networkmanager + gnome

    apt install network-manager network-manager-gnome 

>**ATTENTION** le fichier **/etc/network/interfaces** ne doit pas contenir de ligne faisant référence à aux interfaces ethernet ou wifi car il y aura un problème avec networkmanager ["les réseaux filaires ne sont pas gérés"](https://wiki.debian.org/fr/NetworkManager#Les_r.2BAOk-seaux_filaires_ne_sont_pas_g.2BAOk-r.2BAOk-s)

Installer **sudo** et modifier **/etc/sudoers** pour accès root sans mot de passe utilisateur **eeepc**  

	su
	apt install sudo
	echo "eeepc     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers


### Personnaliser XFCE

* [Xfce4 Desktop Environment Customization](https://github.com/NicoHood/NicoHood.github.io/wiki/Xfce4-Desktop-Environment-Customization)
* [Embellir sa Debian et Xfce](https://blog.delort.email/embellir-sa-debian-et-xfce/)

Pour le thème du bureau, **Numix**

```
# Installation des pré-requies
sudo apt-get install gtk2-engines-murrine murrine-themes
# Installation de Numix
sudo apt install numix-gtk-theme numix-icon-theme
# Installation de Numix-circle
wget https://github.com/numixproject/numix-icon-theme-circle/archive/master.zip
unzip master.zip
sudo mv numix-icon-theme-circle-master/Numix-Circle /usr/share/icons/
sudo mv numix-icon-theme-circle-master/Numix-Circle-Light /usr/share/icons/
rm -r numix-icon-theme-circle-master/
sudo gtk-update-icon-cache /usr/share/icons/Numix-Circle
sudo gtk-update-icon-cache /usr/share/icons/Numix-Circle-Light
```

Menu --> Apparence  

  * Style    : Xfce-flat  
  * Icônes   : Numix
  * Polices  : Droid Sans Fallback 10
  * Rendu
     * activer l'anti-crénélage 
     * Lissage **Léger**
     * Ordre de sous-pixellisation : **RVB**
  * DPI
     * Paramètre DPI personnalisé : **90** 

Menu Paramètres --> Gestionnaire de fenêtres -> Style : **Numix**  

Modification du **tableau de bord** , clic-droit --> Tableau de bord --> Préférences de tableau de bord  
Tableau de bord 1  

  * Onglet **Eléments** 
      * Ajout **Menu whisker** (supprimer "Menu des applications")  
      * Ajout **Mise à jour météo** ,Greffon PulseAudio  
      * Ajout **Copie d'écran** , ajouter **Captures d'écran** 
      * Ajout **Afficher le bureau**
      * Ajout **Moniteur de batteries**
      * Modifier **Horloge** : Affichage date et heure, **format personnalisé** : %e %b %Y %R  
      * Positionner **Boutons d'action** dans apparence , actions `Déconnexion` (décocher tout le reste) 
  * Onglet **Affichage**
      * Verrouiller tableau de bord
      * Masquer automatiquement le tableau de bord **Toujours**  
      * Taille d'une ligne (pixels) **25**   

Tableau de bord 2 à **supprimer**


Après validation tableau de bord, clic droit sur icône capture écran puis  Propriétés -> Zone à capturer : Sélectionner une zone

image_tag src="/images/xfce-tableau-bord1.png" width="300" %}


Menu Paramètres --> Bureau  

  * Fonds d'écran image **/usr/share/backgrounds/**   
  * Icônes : Tout décocher dans **Icônes par défaut**  

Gestionnaire de fichier **Thunar**

  * Edition --> Préférence ,ongle **Comportement** *Ouvrir le dossier dans un nouvel onglet*

image_tag src="/images/thunar-prefer1.png" width="300" %}

Fonction recherche ,installer les logiciels

    sudo apt install catfish mlocate

  * Edition --> Configurer les actions personnalisées ,ajouter (+)

image_tag src="/images/thunar-prefer2.png" width="300" %}image_tag src="/images/thunar-prefer3.png" width="300" %}

Ajoutez la recherche d'historique de la ligne de commande au terminal.  
Tapez un début de commande précédent, puis utilisez shift + up (flèche haut) pour rechercher l'historique filtré avec le début de la commande.  

```
# Global, all users
echo '"\e[1;2A": history-search-backward' | sudo tee -a /etc/inputrc
echo '"\e[1;2B": history-search-forward' | sudo tee -a /etc/inputrc
```

Peut également être localisé pour l'utilisateur dans ~/.inputrc.

```
# Current user only
echo '"\e[1;2A": history-search-backward' >> ~/.inputrc
echo '"\e[1;2B": history-search-forward' >> ~/.inputrc
```

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

### OpenVPN et private Internet Access

* [OpenVPN et private Internet Access](post_url 2017-05-29-Debian-Connexion-Auto-Private-Internet-Access %})

### Fonds d'écran

Toutes les modifications se font en mode su

    sudo -s


#### Fond d'écran connexion (lightdm)

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
greeter-session=lightdm-greeter      # le greeter par défaut
greeter-hide-users=false             # chaque utilisateurs du système sera visible
session-wrapper=/etc/X11/Xsession    # démarrage de la session

```


Avatar (icône utilisateur à la connexion)   
L'image utilisateur **yannick53x64.png** est renommée **.face** dans le dossier **/home/$USER** (/home/eeepc dans notre cas)


    nano /etc/lightdm/lightdm-gtk-greeter.conf  # Modifier les lignes suivantes

```
default-user-image=/home/eeepc/.face
```

Relancer lightdm

    sudo service lightdm restart

#### Fond d'écran bureau

Menu Paramètres --> Bureau   
->Fond d'écran image **/usr/share/backgrounds/Debian-Netbook-1024x600.jpg**     

