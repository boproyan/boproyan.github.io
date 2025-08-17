+++
title = 'Android adb'
date = 2020-03-04 00:00:00 +0100
categories = android
+++
## Adb

### Activer le mode débogage USB (appareil android)

* Depuis notre appareil Android allons sur **Paramètres**, si nous voyons pas le menu **“Options pour les développeurs”** nous allons devoir l'activer.
* Pour ce faire allons dans **Paramètres** –> **A propos du téléphone** ou **A propos de la tablette**
* Appuyons une dizaine de fois sur la partie **Numéro de build**.
Un message devrait nous indiquer que nous sommes maintenant un développeur !
* Nous pouvons activer le Débogage USB via **Paramètres** –> **Options pour les développeurs** –> et cochons **Débogage USB**

![alt text](/images/android-debug01.png "ADB Debug 1")  

![alt text](/images/android-debug02.png "ADB Debug 2")  

### Les outils adb Archlinux/manjaro

	yaourt -S android-tools

### Copier le contenu d'un dossier andoid vers le PC

[How do i adb pull ALL files of a folder present in SD Card](https://stackoverflow.com/questions/10050925/how-do-i-adb-pull-all-files-of-a-folder-present-in-sd-card)  
Exemple : copier tous les fichiers **.apk**

    adb shell find "/sdcard/Documents/Apk" -iname "*.apk" | tr -d '\015' | while read line; do adb pull "$line"; done;

### La reconnaissance de l'appareil  WileyFox Swift  

Connecter l'appareil sur l'ordinateur avec le cordon USB

	lsusb

	Bus 001 Device 018: ID 18d1:4ee7 Google Inc. 

ID 18d1 : Vendeur Google   
Déconnecter l'appareil

En mode root

	sudo -s


1. Créer le fichier : **/etc/udev/rules.d/51-android.rules**
2. Utilisez le format suivant pour ajouter chaque fournisseur au fichier:

	SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", MODE="0666", GROUP="users"

Dans ce cas le vendeur est google , MODE permissions lecture/écriture, and GROUP définit quel groupe Unix possède le nœud de périphérique.  
>Remarque: La syntaxe de la règle peut varier légèrement en fonction de votre environnement.            

	chmod a+r /etc/udev/rules.d/51-android.rules
	exit # sortie du mode root

Pour la prise en charge de la nouvelle règle

	sudo systemctl restart systemd-udevd.service

Reconnecter l'appareil sur l'ordinateur avec le cordon USB et saisir sur un terminal de l'ordinateur

	adb devices

```
List of devices attached
* daemon not running. starting it now on port 5037 *
* daemon started successfully *
d55de0a3	unauthorized
```

Il faut valider l'autorisation sur l'appareil android  
![alt text](/images/android-debug03.png "ADB Debug 3")

et relancer

	adb devices

```
List of devices attached
d55de0a3	device
```

L'appareil est connecté avec les autorisations nécessaires.

## Sauvegarde/Restauration

* [Dump et backup d'un appareil Android avec Debian et ses dérivés](http://debian-facile.org/utilisateurs:slyfox:tutos:dump-android)
* [Android : Utiliser adb pour faire une sauvegarde complète de son smartphone ou tablette sur PC sous GNU/Linux](https://memo-linux.com/android-utiliser-adb-pour-faire-une-sauvegarde-complete-de-son-smartphone-ou-tablette-sur-pc-sous-gnulinux/)

*Le dump ou nandroid est une copie brut d'un système. Il peut s'avérer fort utile en cas de plantage de d'un smartphone ou d'une tablette sous Android comme par exemple suite à une fausse manipulation ou dans le cas où l'appareil reste bloquée sur le logo de démarrage. Réinstaller ou flasher le firmware reste alors la seul manière de refaire fonctionner l'appareil...*

### Sauvegarder ses données avec adb 

La commande principale est :

	adb backup -f nom_fichier_sauvegarde.ab

>Si l'option '-f' n'est pas spécifiée, le fichier 'backup.ab' sera créé dans le répertoire courant où est exécutée la commande de sauvegarde ...

L'aide nous restitue :

```
  adb backup [-f <file>] [-apk|-noapk] [-obb|-noobb] [-shared|-noshared] [-all] [-system|-nosystem] [<packages...>]
                               - write an archive of the device's data to <file>.
                                 If no -f option is supplied then the data is written
                                 to "backup.ab" in the current directory.
                                 (-apk|-noapk enable/disable backup of the .apks themselves
                                    in the archive; the default is noapk.)
                                 (-obb|-noobb enable/disable backup of any installed apk expansion
                                    (aka .obb) files associated with each application; the default
                                    is noobb.)
                                 (-shared|-noshared enable/disable backup of the device's
                                    shared storage / SD card contents; the default is noshared.)
                                 (-all means to back up all installed applications)
                                 (-system|-nosystem toggles whether -all automatically includes
                                    system applications; the default is to include system apps)
                                 (<packages...> is the list of applications to be backed up.  If
                                    the -all or -shared flags are passed, then the package
                                    list is optional.  Applications explicitly given on the
                                    command line will be included even if -nosystem would
                                    ordinarily cause them to be omitted.)

```

Dans un terminal de l'ordinateur sur lequel est connecté l'appareil android via USB, lancer l'exécution de la sauvegarde

	adb backup -apk -shared -all -f /home/$USER/backup_android.ab

Un message s’affiche dans le terminal pour confirmer sur l’appareil :

	Now unlock your device and confirm the backup operation.

Confirmer sur l’appareil le lancement de la sauvegarde, il est possible de la chiffrer :  
![alt text](/images/adb-fullbackup.png "ADB Debug 2")  

#### Sauvegarder tout 

	adb backup -f nom_fichier_sauvegarde.ab -all

Cette commande implique la sauvegarde de toutes les applications installées, dont les applications systèmes. 

>Cette option intègre l'option '-system' !

#### Sauvegarde applications 

	adb backup -f nom_fichier_sauvegarde.ab -apk -obb

Cette commande sauvegarde les fichiers apk des applications installées.

>L'option par défaut est '-noapk' qui signifie que les fichiers apk ne seront pas sauvegardés ! <br />
L'option '-obb' implique la sauvegarde des fichiers relatifs aux applications installées, tels que fichiers de sauvegarde, de config, etc ... - par défaut, c'est l'option '-noobb' qui est active !

	adb backup -f nom_fichier_sauvegarde.ab package1 package2 package_n

Sauvegarde les noms des applications concernées !

#### Sauvegarde SD Carte 

	adb backup -f nom_fichier_sauvegarde.ab -shared

Sauvegarde le contenu de la SD Carte, ainsi que de tout répertoire de stockage partagé.

>Par défaut, c'est l'option '-noshared' qui est active et implique la non sauvegarde !
>>Attention : Il est bien sûr nécessaire d'avoir une SD Carte dans votre appareil ...

#### Sauvegarde Système 

	adb backup -f nom_fichier_sauvegarde.ab -system

>Par défaut, c'est l'option '-nosystem' qui est activée, ce qui a pour effet de ne pas inclure les applications systèmes !

#### Sauvegarde Pertinente 


Le moyen pertinent de sauvegarder tout correctement est, sans s'occuper du contenu de la SD Carte :

	adb backup -f nom_fichier_sauvegarde.ab -apk -obb -all

### Sauvegarde partitions 


L'outil 'adb' peut servir à sauvegarder indirectement les partitions de votre tablette ...
Pour cela, il faut télécharger l'outil [rkdump](http://files.androtab.info/rk2818/devel/rkdump_android.zip) !

Puis l'utiliser ainsi :

	adb push rkdump /data/ 
	adb shell chmod 0755 /data/rkdump 

Puis utiliser l'outil rkdump comme décrit dans son [tutoriel](http://androtab.info/rockchip/devel/rkutils/) ...


### Restaurer une sauvegarde avec adb

Pour lancer la restauration de la sauvegarde total de l’Android, saisir dans le terminal :

	adb restore nom_fichier_sauvegarde.ab

>Tout ce qui concerne le contenu du fichier de sauvegarde sera restauré !







