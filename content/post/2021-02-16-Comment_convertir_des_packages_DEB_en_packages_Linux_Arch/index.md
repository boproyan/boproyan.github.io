+++
title = 'debtap ou comment convertir des packages deb en Linux Arch'
date = 2021-02-16 00:00:00 +0100
categories = ['archlinux', 'outils']
+++
*Convertir des packages DEB en packages Arch Linux (ex: mullvad vpn)*

### Installer Debtap

Pour cela, nous allons utiliser un utilitaire appelé «Debtap» . Il signifie DEB T o A rch (Linux) P ackage. Debtap est disponible dans AUR , vous pouvez donc l'installer à l'aide des outils d'assistance AUR tels que Yay .

Pour installer debtap à l'aide de yay, exécutez:

    yay -S debtap 

Et, assurez-vous que votre système Arch devrait avoir les packages **bash, binutils , pkgfile et fakeroot** installés.

Après avoir installé Debtap et toutes les dépendances mentionnées ci-dessus, exécutez la commande suivante pour créer / mettre à jour pkgfile et la base de données debtap.

    sudo debtap -u

>**Vous devez exécuter la commande ci-dessus au moins une fois.**  

En sortie ...

```
==> Synchronizing pkgfile database...
:: Updating 3 repos...
  download complete: core                 [   870,9 KiB   184K/s  2 remaining]
  download complete: extra                [     9,0 MiB   366K/s  1 remaining]
  download complete: community            [    20,0 MiB   681K/s  0 remaining]
:: download complete in 30,11s            <    29,8 MiB  1014K/s  3 files    >
:: waiting for 1 process to finish repacking repos...
==> Synchronizing debtap database...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 37.5M  100 37.5M    0     0   990k      0  0:00:38  0:00:38 --:--:-- 1117k
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  949k  100  949k    0     0   964k      0 --:--:-- --:--:-- --:--:--  963k
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  247k  100  247k    0     0   704k      0 --:--:-- --:--:-- --:--:--  704k
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 38.7M  100 38.7M    0     0  1141k      0  0:00:34  0:00:34 --:--:-- 1120k
==> Downloading latest virtual packages list...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   149    0   149    0     0    404      0 --:--:-- --:--:-- --:--:--   403
100 11890    0 11890    0     0  12107      0 --:--:-- --:--:-- --:--:-- 12107
==> Downloading latest AUR packages list...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  360k    0  360k    0     0   164k      0 --:--:--  0:00:02 --:--:--  164k
==> Generating base group packages list...
==> All steps successfully completed!
```


Maintenant, il est temps de convertir les packages.

### Convertir des packages DEB en packages Linux Arch à l'aide de Debtap

On prend comme exemple **mullvad** qui ne fournit que des paquets rpm ou deb...  
Désinstaller mullvad-vpn-bin : `yay -R mullvad-vpn-bin`  

```
vérification des dépendances…

Paquets (1) mullvad-vpn-bin-2019.10-1

Taille totale supprimée :  219,23 MiB

:: Voulez-vous désinstaller ces paquets ? [O/n] 
:: Traitement des changements du paquet…
Making sure the Mullvad VPN daemon is stopped & disabled...
Removed /etc/systemd/system/multi-user.target.wants/mullvad-daemon.service.
(1/1) désinstallation de mullvad-vpn-bin           [######################] 100%
-------------------------------------------------------------
Optionally remove logs & cache:
sudo rm -rf /var/log/mullvad-vpn/
sudo rm -rf /var/cache/mullvad-vpn/

Optionally remove config:
sudo rm -rf /etc/mullvad-vpn
-------------------------------------------------------------
:: Exécution des crochets de post-transaction…
(1/4) Reloading system manager configuration...
(2/4) Arming ConditionNeedsUpdate...
(3/4) Updating icon theme caches...
(4/4) Updating the desktop file MIME type cache...
```

Pour convertir n'importe quel package DEB, par exemple **MullvadVPN-2020.2_amd64.deb** , en package Arch Linux à l'aide debtap, procédez comme suit:

    debtap MullvadVPN-2020.2_amd64.deb

La commande ci-dessus convertira le fichier .deb donné en un package Arch Linux. Il vous sera demandé de saisir le nom du responsable du package et de la licence. Entrez-les et appuyez sur la touche ENTRÉE pour démarrer le processus de conversion.

La conversion du package prendra de quelques secondes à plusieurs minutes selon la vitesse de votre CPU. Prenez une tasse de café.

Un exemple de sortie serait:

```
==> Extracting package data...
==> Fixing possible directories structure differencies...
==> Generating .PKGINFO file...

:: Enter Packager name:
mullvad

:: Enter package license (you can enter multiple licenses comma separated):


*** Creation of .PKGINFO file in progress. It may take a few minutes, please wait...

==> Checking and generating .INSTALL file (if necessary)...

:: If you want to edit .PKGINFO and .INSTALL files (in this order), press (1) For vi (2) For nano (3) For default editor (4) For a custom editor or any other key to continue: 


==> Generating .MTREE file...

==> Creating final package...
==> Package successfully created!
==> Removing leftover files...

```

Si vous ne souhaitez pas répondre à des questions pendant la conversion du package, utilisez l'indicateur `-q` pour contourner toutes les questions, sauf pour la modification des fichiers de métadonnées.

    debtap -q MullvadVPN-2020.2_amd64.deb

Pour contourner toutes les questions (non recommandé cependant), utilisez l'indicateur `-Q`

    debtap -Q MullvadVPN-2020.2_amd64.deb

Une fois la conversion terminée, vous pouvez installer le package nouvellement converti à l'aide de «pacman» dans votre système Arch comme indiqué ci-dessous.

    sudo pacman -U mullvad-vpn-2020.2.0-1-x86_64.pkg.tar.xz 

Pour afficher la section d'aide, utilisez l'indicateur -h :

    debtap -h

```
Syntaxe: debtap [options] nom_fichier_package

 Options:

  -h --h -help --help Imprime ce message d'aide
  -u --u -update --update Mettre à jour la base de données de detteap
  -q --q -quiet --quiet Contourne toutes les questions, sauf pour l'édition des fichiers de métadonnées
  -Q --Q -Quiet --Quiet Contourner toutes les questions (non recommandé)
  -s --s -pseudo --pseudo Créer un package pseudo-64 bits à partir d'un package .deb 32 bits
  -w --w -wipeout --wipeout Supprimer les versions de toutes les dépendances, conflits, etc.
  -p --p -pkgbuild --pkgbuild Génère en outre un fichier PKGBUILD
  -P --P -Pkgbuild --Pkgbuild Génère un fichier PKGBUILD uniquement 
```
