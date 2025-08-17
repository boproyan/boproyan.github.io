+++
title = 'Dell_Latitude_e6230_activation_RFID-NFC_(Broadcom5880)'
date = 2019-11-07 00:00:00 +0100
categories = divers
+++
## Dell Latitude e6230 activation RFID-NFC (Broadcom 5880)

Article original :   
[Enabling Dell Latitude RFID/NFC (Broadcom 5880)](https://blog.g3rt.nl/enable-dell-nfc-contactless-reader.html)


### À propos de la solution de sécurité Dell

![Icône RFID de Dell sur le repose-mains Latitude E7240](20150717_dell_rfid_palmrest.jpg)

Les ordinateurs Dell Latitude que j’utilise, E7240 E6230 et E6530, affichent une icône sur le repose-mains indiquant qu’un lecteur sans contact (NFC / RFID) est présent. Cependant, dans le système d'exploitation, rien ne le montre. PCSC reconnaît le lecteur "Contacté"(Contacted), mais pas le "sans contact"(Contactless).

    pcsc_scan

```
Using reader plug'n play mechanism
Scanning present readers...
0: Broadcom Corp 5880 [Contacted SmartCard] (0123456789ABCD) 00 00
```

Dans cet article, je vais montrer les étapes à suivre pour inclure ...

```
1: Broadcom Corp 5880 [Contactless SmartCard] (0123456789ABCD) 01 00
```

Il semble que Dell ait coopéré avec Broadcom et créé une solution de sécurité appelée **ControlVault** pour le Broadcom Unified Security Hub (USH). Il s’agit d’une solution OEM exclusivement proposée par Dell.

### Outils de support DOS du package de support ControlVault

Heureusement, dans certains packages de prise en charge Windows destinés uniquement à la mise à niveau du micrologiciel Dell ControlVault fournis sur la page de support du modèle Latitude, j'ai repéré un document intéressant portant la mention «confidentiel» - à propos de l'utilisation de cet outil de diagnostic USH, ainsi qu'un outil plus récent, ushdiag.exe lui-même!

Quelques premières étapes de préparation:

1.    Téléchargez le [package de mise à niveau de ControlVault](https://www.helpjet.net/Fs-29857265-43688392-77304872-extract.html),  CV_WBF_Setup_Y2GT8_64bit_ZPE.exe
2.    Renommez le fichier pour qu'il porte l'extension .zip .
3.    Décompressez-le.
4.    Notez un dossier DOS dans le répertoire **/Dell ControlVault WBF Firmware/**

```
      DOS
     ├── DOS4GW.EXE
     ├── dosushdiag.pdf <- "Document d'architecture de clavier USH Broadcom"
     ├── errlvl.exe
     ├── release.txt
     ├── sleep.exe
     ├── ushdiag.exe <- là!
     └── ushfwumg.bat
```

### Outil de diagnostic USH Broadcom

Ce fichier PDF de Broadcom explique l'utilisation de l'outil ushdiag.exe .  
Le plus important est les options à fournir.

```
5.22 Device Enable (-de <devMask>)
This command will enable the specified devices.

<devMask>:

0: Smart Card:
1: Fingerprint:
2: RFID radio
3: CV Only Radio


5.23 Device Disable (-dd <devMask>)
This command will disable the specified devices.

<devMask>:

0: Smart Card:
1: Fingerprint:
2: RFID radio
3: CV Only Radio
```

### Lancer le lecteur flash USB DOS

Procédure :

1.    Téléchargez la dernière version du programme d'installation USB à partir de la page de [téléchargement gratuit de FreeDOS](http://www.freedos.org/download/)  
2.    Extraire l'archive, vous obtenez un fichier.img
3.    Déterminez de quel périphérique /dev/sdX est votre clé USB (utilisez fdisk -l pu dmesg)
4.    Ecrire l'image directement sur le périphérique bloc :  
`dd if=FD12FULL.img of=/dev/sdX status=progress` (où X est la lettre représentant votre clé USB en tant que périphérique bloc, ne pas écrire l'image sur une partition)
5.    Vérifiez deux fois que la copie de l'image a fonctionné :
      * fdisk -l (vous devriez voir une partition unique sur un disque DOS avec l'option bootable ("boot") activée)
6.    Montez la partition
      *  Effacer le dossier /FDSETUP/PACKAGES/ sur la clé pour récupérer de la place
      *  Copier les 2 fichiers suivants
          *  `sudo cp DOS/ushdiag.exe /mnt/usb/USHDIAG.EXE`
          *  `sudo cp DOS/DOS4GW.EXE /mnt/usb/`
 de matériel.
7.    Démonter et redémarrer. Faites le nécessaire (BIOS) pour démarrer à partir de la clé USB

Vous vous retrouverez maintenant dans l'environnement d'installation en direct de FreeDOS.

1.    Sélectionnez votre langue
2.    Vous serez invité à installer FreeDOS
      *  Sélectionnez "Non - Retour au DOS".
3.    Vous devriez voir une invite (C:\>)
4.    Exécutez dir  et vérifiez la présence des exécutables DOS4GW.EXE et USHDIAG.EXE

Vérifiez l'état actuel du périphérique USH:

```
FreeDOS C:\>ushdiag.exe -u -stat
[...]
Smart Card: Present; Enabled
Fingerprint: Present; Enabled
RFID Radio: Present; Enabled
RFID Lock: Disabled
CV Only Radio: Enabled
RFID AutoDetect Set
RFID Present Not Forced
WBDI: Enabled
RFID Block Mode: Unknown (CV Only Radio Mode Enabled)
```

Comme vous pouvez le constater, le mode CV uniquement est activé.

Désactivez maintenant le périphérique "CV-only" pour permettre un accès RFID CCID normal en fournissant un masque hexadécimal 8 . Cela provient du document trouvé précédemment en envoyant une commande de désactivation sur le champ de bits 3.

```
FreeDOS C:\>ushdiag.exe -u -dd 8
[...]
Disabled CV Only Radio Mode. waiting for USH to reset
[...]
RFID Lock: Disabled
CV Only Radio: Disabled
[...]
```

Redémarrez votre système et profitez du NFC/RFID ! 

    pcsc_scan

```
Using reader plug'n play mechanism
Scanning present readers...
0: Broadcom Corp 5880 [Contacted SmartCard] (0123456789ABCD) 00 00
1: Broadcom Corp 5880 [Contactless SmartCard] (0123456789ABCD) 01 00
```

Le lecteur NFC "Contactless Smartcard" est actif .


