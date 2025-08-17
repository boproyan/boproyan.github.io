+++
title = 'Wifi iwd remplace wpa_supplicant'
date = 2020-08-04 00:00:00 +0100
categories = ['wifi']
+++
## iwd, le daemon Wi-Fi

*iwd vise à remplacer wpa_supplicant : [Présentation des Wireless Daemon sous Linux](https://www.linuxembedded.fr/2020/07/presentation-des-wireless-daemon-sous-linux/)  
[iwd (iNet wireless daemon)](https://wiki.archlinux.org/index.php/Iwd) est un démon sans fil pour Linux écrit par Intel. L'objectif principal du projet est d'optimiser l'utilisation des ressources en ne dépendant d'aucune bibliothèque externe et en utilisant plutôt les fonctionnalités fournies par le noyau Linux dans toute la mesure du possible.  
iwd peut fonctionner en mode autonome ou en combinaison avec des gestionnaires de réseau complets comme ConnMan, systemd-networkd et NetworkManager.*  

Regarder si [iwd est compatible](https://iwd.wiki.kernel.org/networkmanager) avec votre version de Network Manager (si vous l’utilisez).  
Ce n’est pas le cas pour Debian Buster mais c’est bon pour Ubuntu 20.04, Mint 20, Debian Bullseye (testing), Arch, openSUSE Leap 15.2... (au 04/08/20202)  

Il est extrêmement simple de tester. 

    sudo apt install iwd # debian ubuntu
    yay -S iwd           # archlinux Manjaro

ensuite on édite 

    sudo nano /etc/NetworkManager/NetworkManager.conf # on ajoute à la fin.

```ini
[device]
wifi.backend=iwd
```

On désactive le service wpa_supplicant 

    sudo systemctl stop wpa_supplicant 
    sudo systemctl mask --now wpa_supplicant 
    sudo systemctl daemon-reload

On lance le service iwd

    sudo systemctl start iwd
    sudo systemctl enable iwd

et on restart NetworkManager 

    sudo systemctl restart NetworkManager

Vous devriez avoir une demande de mot de passe pour le Wi-Fi sur lequel vous êtes, vous validez.  
La procédure chez Debian est [ici](https://wiki.debian.org/NetworkManager/iwd).

Si vous souhaitez revenir en arrière, enlevez les lignes ajoutées dans `/etc/NetworkManager/NetworkManager.conf`, `sudo systemctl stop iwd` , `sudo systemctl disable iwd` , `sudo systemctl unmask --now wpa_supplicant` , `sudo systemctl start wpa_supplicant` et `sudo systemctl restart NetworkManager`
