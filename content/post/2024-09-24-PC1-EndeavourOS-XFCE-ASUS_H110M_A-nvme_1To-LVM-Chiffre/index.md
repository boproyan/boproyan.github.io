+++
title = 'Mini tour PC1 - EndeavourOS XFCE sur partition LVM chiffrée'
date = 2024-09-24 00:00:00 +0100
categories = ['archlinux', 'chiffrement', 'lvm']
+++
*EndeavourOS est une distribution GNU/Linux basée sur Arch Linux*  


* [Description matériel mini tour PC1](/posts/Description_materiel_minitour_PC1/)  

![](yannick.drawio.png)

## Création clé EndeavourOS USB Live

Télécharger le dernier fichier iSO <https://endeavouros.com/latest-release/>  
**EndeavourOS_Cassini_Nova-03-2023_R1.iso** et **EndeavourOS_Cassini_Nova-03-2023_R1.iso.sha512sum**  

Vérifier checksum

    sha512sum -c EndeavourOS_Cassini_Nova-03-2023_R1.iso.sha512sum

**EndeavourOS_Cassini_Nova-03-2023_R1.iso: Réussi**

Créer la clé bootable   
Pour savoir sur quel périphérique, connecter la clé sur un port USB d'un ordinateur et lancer la commande `sudo dmesg`  
Dans le cas présent , le périphérique est **/dev/sde**

    sudo dd if=EndeavourOS_Cassini_Nova-03-2023_R1.iso of=/dev/sde bs=4M

`Installer une distribution EndeavourOS sur une partition LVM est impossible avec l'outil "Calamarès"`{: .prompt-warning }

## Installation EndeavourOS XFCE sur partition LVM entièrement chiffrée

### Installation via USB LIVE

Démarrage avec la clé USB insérée dans le Mini tour PC1 et appui sur F12 pour un accès au menu    
Choisir UEFI specific storage 

Vous arrivez sur la page de sélection  
![](endos0001.png){:width="400"}  
Valider le choix par défaut  

Changer le clavier en FR  
![](endos0001a.png){:width="600"}  
![](endos0001b.png){:width="400"}  
![](endos0001c.png){:width="200"}  
Supprimer **English(US)** pour ne garder que **French** et **Close**   

Ouvrir un **Terminal Emulator** dans le live endeavour  
![](endos0001d.png){:width="600"}  

### Partionner un disque

en mode su

    sudo -s

Le disque : `lsblk`

```
nvme0n1               259:0    0 931,5G  0 disk 
```

On partitionne un disque en 3 avec `gdisk`

* Partition 1 : 512M EFI (code ef00) système de fichier FAT32
* Partition 2 : 920G LVM (code 8e00) système de fichier EXT4
* Partition restante pour Installation temporaire 

Zapper le disque,

(**Attention** Ceci effacera de manière irréversible toutes les données de votre disque, veuillez sauvegarder toutes les données importantes) :

```
sgdisk --zap-all /dev/nvme0n1
# OU
wipefs -a /dev/nvme0n1
```

Partitionnement du disque NVME 1To GPT + LVM

    gdisk /dev/nvme0n1

Format la partition EFI

    mkfs.fat -F32 /dev/nvme0n1p1

### Installer EndeavourOS sur une partition temporaire

Lancer l'installation  
![](endos0002.png){:width="600"}  


![](endos0003.png){:width="600"}  
Choix du "pas en ligne"

![](endos0004.png){:width="600"}  
Français  

Pour ne pas installer le parefeu firewalld  
![](endos0004a.png){:width="600"}  

![](endos0005.png){:width="600"}  

![](endos0006.png){:width="600"}  

![](endos0006n.png){:width="600"}  

![](endos0007n.png){:width="600"}  

![](endos0007n1.png){:width="600"}  


Laissez Calamares terminer l'installation. 


L'installation démarre  
![](endos0013.png){:width="600"}  
Installation en cours, patienter ...

  
![](endos0014.png){:width="600"}  
L'installation est terminée, cliquer "Redémarrer maintenant" et sur **Terminé**, oter la clé USB,  et redémarrer sur endeavour

Vérifiez si vous pouvez accéder au système crypté.
Vous devriez maintenant avoir un système crypté LUKS (sans les trucs amusants comme les volumes logiques, la partition /home séparée, etc.).  

`Réinsérer la clé USB et redémarrer dans l'environnement Live-Cd`{: .prompt-info }

Commuter le clavier en FR  
Ouvrir un terminal et basculer en mode su : `sudo -s`

**Facultatif**  
Pour un accès sur la machine via SSH depuis un poste distant  
Lancer le service : `sudo systemctl start sshd`  
Créer un mot de passe à liveuser : `passwd liveuser`
Relever l'adresse ip de la machine : `ip a`

### Convertir Déchiffrer et monter le système temporaire

Le système temporaire chiffré /dev/nvme0n1p3

Conversion chiffrement luks2

    cryptsetup convert /dev/nvme0n1p3 --type luks2

```
WARNING!
========
This operation will convert /dev/nvme0n1p3 to LUKS2 format.


Are you sure? (Type 'yes' in capital letters): YES
```

Confirmer par la saisie YES

Dans l'environnement live-CD, ouvrez un Terminal ,basulez en mode su et tapez (ou marquez et copiez la ligne avec ctrl-c et collez dans le terminal avec shift-ctrl-v ) …

```shell
cryptsetup luksOpen /dev/nvme0n1p3 crypttemp # saisir la phrase mot de passe de l'installation
mkdir -p /media/crypttemp
mount /dev/mapper/crypttemp /media/crypttemp 
```

Nos données d'installation temporaires sont désormais accessibles sous `/media/crypttemp` et peuvent être copiées sur le nouveau système que nous allons mettre en place dans les prochaines étapes.

### Configurer le nouveau système LVMonLUKS

Chiffrer la partition /dev/nvme0n1p2

```shell
cryptsetup luksFormat --type luks2 /dev/nvme0n1p2
```

Une demande de confirmation est exigée

```
WARNING!
========
This will overwrite data on /dev/nvme0n1p2 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/nvme0n1p2: 
Verify passphrase: 
```

Choisissez un mot de passe sécurisé ( <https://xkcd.com/936/> )

```shell
cryptsetup luksOpen /dev/nvme0n1p2 crypt
    Enter passphrase for /dev/nvme0n1p2:
pvcreate /dev/mapper/crypt
    Physical volume "/dev/mapper/crypt" successfully created.
vgcreate vg0 /dev/mapper/crypt
    Volume group "vg0" successfully created
```

Une bonne taille de départ pour le volume racine (lvroot) est d'environ 30 Go. Si vous envisagez d'utiliser ultérieurement un fichier d'échange résidant sur root, vous devez en tenir compte.  
Le redimensionnement ultérieur des volumes est assez facile, alors n'y réfléchissez pas trop.  
Vous pouvez attribuer tout l'espace libre restant au volume d'accueil,  
`lvcreate --extents 100%FREE vg0 -n lvhome`  
mais pour augmenter les volumes plus tard et pour les instantanés , il faut de l'espace vide à l'intérieur du groupe de volumes, donc je choisis généralement une taille pour lvhome qui laisse environ 30 Go d'espace inutilisé global dans le volume groupe (en supposant un lecteur de 500 Go, par exemple 500 – 0,512 – 40 – 430 = 29,488)

```shell
# 60G root dont 8 swapfile
lvcreate -L 60G vg0 -n lvroot   #  Logical volume "lvroot" created.
lvcreate -L 120G vg0 -n lvhome  #  Logical volume "lvhome" created.
```

Créez un système de fichiers ext4 sur les volumes logiques.

```shell
mkfs.ext4 -L root /dev/mapper/vg0-lvroot
mkfs.ext4 -L home /dev/mapper/vg0-lvhome
```

### Monter le nouveau système sur "mnt"

Monter le nouveau système sur `/mnt` pour les systèmes UEFI

```shell
mount /dev/mapper/vg0-lvroot /mnt
mkdir -p /mnt/home
mount /dev/mapper/vg0-lvhome /mnt/home
mkdir -p /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi
```

    lsblk

devrait maintenant fournir une sortie similaire à la suivante (ignorez les tailles, celles-ci proviennent d'une installation de test) …

pour les systèmes UEFI :

```
NAME             MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
nvme0n1              259:0    0 931.5G  0 disk  
├─nvme0n1p1          259:1    0   512M  0 part  /mnt/efi
├─nvme0n1p2          259:2    0   920G  0 part  
│ └─crypt            254:4    0   920G  0 crypt 
│   ├─vg0-lvroot     254:5    0    60G  0 lvm   /mnt
│   └─vg0-lvhome     254:6    0   120G  0 lvm   /mnt/home
└─nvme0n1p3          259:3    0    11G  0 part  
  └─crypttemp        254:3    0    11G  0 crypt /media/crypttemp
```

### Copier le système temporaire 

pour vider les nouveaux points de montage

    rsync -avA /media/crypttemp/ /mnt

*Veuillez patienter quelques minutes*

A la fin

```
sent 4,371,397,995 bytes  received 3,121,049 bytes  138,873,620.44 bytes/sec
total size is 4,358,374,092  speedup is 1.00
```

### Démonter le système temporaire

```shell
umount /media/crypttemp
cryptsetup luksClose crypttemp
```

### Ajouter un fichier de clé existant LUKS

Nous allons maintenant ajouter une deuxième clé saisie à la création chiffrement sur /dev/nvme0n1p2  
Nous ferons référence à cette clé à l'étape suivante.

    cryptsetup luksAddKey /dev/nvme0n1p2 /mnt/crypto_keyfile.bin  

Il faut saisir le phrase mot de passe

### Configurer "crypttab"

Configuration `/etc/crypttab`

    cryptsetup luksUUID /dev/nvme0n1p2

renvoie **2c8e7bb4-9286-47e9-8823-12b79bf2810c**  
Votre UUID sera différent, alors <u>**assurez-vous d'utiliser votre UUID à l'étape suivante !**</u>

    nano /mnt/etc/crypttab

contient une ligne non commentée commençant par `luks-`...  
Remplacez cette ligne par la suivante ; <u>**n'oubliez pas d' utiliser votre UUID**</u> 

    cryptlvm UUID=2c8e7bb4-9286-47e9-8823-12b79bf2810c /crypto_keyfile.bin luks

Sauvegarder et quitter.

### Basculer en chroot

Passer en chroot

    arch-chroot /mnt

### Configurer "fstab"

Configurer /etc/fstab

    blkid -s UUID -o value /dev/mapper/vg0-lvroot

renvoie l'UUID du volume racine :  **6727ede1-2ba5-45fb-9c5a-379c263ec078**.

    blkid -s UUID -o value /dev/mapper/vg0-lvhome

renvoie l'UUID du volume d'accueil : **52a1074e-72df-46d5-acf1-6553cdce1eaa**.

    nano /etc/fstab

contient une ligne commençant par `/dev/mapper/luks-`...  
**Supprimez** cette ligne et ajoutez ce qui suit (<u>**n'oubliez pas d' utiliser vos UUID**</u>) 

```
UUID=6727ede1-2ba5-45fb-9c5a-379c263ec078 / ext4 defaults,acl,noatime,discard 0 0
UUID=52a1074e-72df-46d5-acf1-6553cdce1eaa /home ext4 defaults,acl,noatime,discard 0 0
```

Sauvegarder et quitter.

### Ajout fichier échange

Utilisez dd pour créer un fichier d'échange de la taille de votre choix.  
Création d'un fichier d'échange de 8192 Mo (pour tous les systèmes de fichiers)

    dd if=/dev/zero of=/swapfile bs=1M count=8192 status=progress

Remplacez `count=8192` par la quantité de Mo que vous souhaitez installer pour l'utilisation du fichier d'échange :

    chmod 600 /swapfile

Pour donner au fichier d'échange des permissions de racine seulement.

    mkswap /swapfile

Pour faire du fichier un espace de pagination et enfin pour activer le fichier :

    swapon /swapfile

Modifier /etc/fstab pour activer le fichier d'échange 

    nano /etc/fstab

Ajoutez la ligne suivante…

```
/swapfile                                 none  swap defaults,pri=-2 0 0
```

Sauvegarder et quitter.

>Remarque : le fichier d'échange doit être spécifié par son emplacement sur le système de fichiers, et non par son UUID ou son LABEL.

pour vérifier :

    swapon --show

![Texte alternatif](swapon.png)

### Modifier les options du noyau


Dans **systemd-boot**, vous éditez le fichier d'entrée approprié qui se trouve sur votre partition EFI dans le répertoire `loader/entries`  
Chaque entrée est une option de démarrage dans le menu et chacune a une ligne appelée options. Vous pouvez modifier ces entrées directement, mais ces changements peuvent être écrasés lors de l'installation ou de la mise à jour de paquets.


Pour effectuer les changements, au lieu de modifier les entrées, modifiez le fichier `/etc/kernel/cmdline` qui est un fichier d'une ligne contenant une liste d'options du noyau.  

    nano /etc/kernel/cmdline

UUID de /dev/nvme0n1p2 : `blkid -s UUID -o value /dev/nvme0n1p2`

```
nvme_load=YES nowatchdog rw rd.luks.uuid=2c8e7bb4-9286-47e9-8823-12b79bf2810c root=/dev/mapper/vg0-lvroot
```

Exécutez ensuite `sudo reinstall-kernels` qui remplira les entrées et régénérera les initrds.

    reinstall-kernels

Si vous préférez utiliser les options actuelles du noyau du système en cours d'exécution, vous pouvez rm le fichier /etc/kernel/cmdline, puis exécuter sudo reinstall-kernels.

### Sortie du chroot et démontage

    exit
    umount -R /mnt

## Redémarrez sur le système LVM on LUKS chiffré

Oter la clé USB , redémarrer

    reboot

>FINI! Vous devriez maintenant avoir un système LVMonLUKS fonctionnel avec un volume logique séparé pour /home.

### Compléments

#### Accès sudo

Modifier sudoers pour accès sudo sans mot de passe à l'utilisateur yano

```
su               # mot de passe root identique utilisateur
echo "yann     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/20-yann
```

#### Historique de la ligne de commande  

Ajoutez la recherche d’historique de la ligne de commande au terminal  
Se connecter en utilisateur  
Tapez un début de commande précédent, puis utilisez shift + up (flèche haut) pour rechercher l’historique filtré avec le début de la commande.

```shell
# Global, tout utilisateur
echo '"\e[1;2A": history-search-backward' | sudo tee -a /etc/inputrc
echo '"\e[1;2B": history-search-forward' | sudo tee -a /etc/inputrc
```

#### Modification fstab

Ajout au fichier `/etc/fstab`  

```
# /dev/mapper/vg0-lvmedia
UUID=2b44ad46-3fd5-4f36-a503-b1e7507aa561 	/srv/media      ext4            rw,relatime     0 2

# /dev/mapper/ssd--512-virtuel
UUID=84bc1aa9-23ac-4530-b861-bc33171b7b42 	/virtuel    ext4    	defaults	0 2

# /dev/mapper/vg--nas--one-sav
UUID=c5b9eefc-1daa-4a0d-8a72-6169b3c8c91f       /sauvegardes	ext4            defaults        0 2

# /dev/vg-nas-one/iso - Volume logique 200G du disque 4To
UUID=58f4b6c7-3811-41d5-9964-f47ac32375f6	    /iso 	ext4		defaults	0 2

# LXC kvmdebeye dossiers partagés
#/srv/media /var/lib/lxc/kvmdebeye/rootfs/home/nspyan/media  none bind 0 0
#/home/yann/scripts /var/lib/lxc/kvmdebeye/rootfs/home/nspyan/scripts  none bind 0 0


#/dev/sdd1: LABEL="hdd500g" UUID="21b413f5-196e-49e2-995b-07eb39e32e23" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="ee37e746-3f0f-11ed-9462-066ab000c657"
UUID=21b413f5-196e-49e2-995b-07eb39e32e23 	/mnt/freebox	ext4	defaults	0 2

```

On monte les unités

    sudo mount -a

On modifie les droits /srv/media

    chown $USER:$USER -R /srv/media/

#### Images (fond d'écran + logos, connexion et grub)

    sudo cp /srv/media/dplus/images/yannick/plouzane-nb.jpg /usr/share/backgrounds/xfce/  # écran 2
    sudo cp /srv/media/dplus/images/yannick/Linux-Arch-1920x1080.jpg /usr/share/backgrounds/xfce/ # écran 1
    sudo cp /srv/media/dplus/images/yannick/yannick-green.png /usr/share/pixmaps/

les images de fond d'écran **/usr/share/backgrounds/xfce**   

	sudo cp /srv/media/dplus/images/Fonds/Linux-Arch-1920x1080.jpg /usr/share/backgrounds/xfce/ 
	sudo cp /srv/media/dplus/images/yannick/yannick-green.png /usr/share/pixmaps/ 

Ecran et logo pour lightdm de la page de connexion

	sudo cp /srv/media/dplus/images/Fonds/archlinux-lightdm.png /usr/share/backgrounds/
	sudo cp /srv/media/dplus/images/yannick/yannick53x64.png /usr/share/pixmaps/

### Partage disque

[Partage disque externe USB sur Freebox](/posts/Partage_disque_externe_USB_sur_Freebox/)

#### Disque freebox partagé FreeUSB2To

**FreeBox**  
HDD Mobile 2To connecté en USB sur la freebox  
Nom de partage : FreeUSB2To + EXT4 + vérification après formatage  
Partage windows activé : yannfreebox + mot de passe

**PC1**  
Partage linux samba : `sudo pacman -S cifs-utils` Installé par défaut   
Point de montage : `sudo mkdir /mnt/FreeUSB2To`   
Lien : `sudo ln -s /mnt/FreeUSB2To $HOME/FreeUSB2To`

Credential : /root/.smbcredentials avec 2 lignes  
username=XXXXXX  
password=XXXXXX  

Droits

```shell
sudo chown -R root:root /root/.smbcredentials
sudo chmod -R 600 /root/.smbcredentials
```

Les fichiers systèmes

```
# /etc/systemd/system/mnt-FreeUSB2To.mount 
[Unit]
  Description=cifs mount script
  Requires=network-online.target
  After=network-online.service

[Mount]
  What=//192.168.0.254/FreeUSB2To
  Where=/mnt/FreeUSB2To
  Options=credentials=/root/.smbcredentials,rw,uid=1000,gid=1000,vers=3.0
  Type=cifs

[Install]
  WantedBy=multi-user.target

# /etc/systemd/system/mnt-FreeUSB2To.automount 
[Unit]
  Description=cifs mount script
  Requires=network-online.target
  After=network-online.service

[Automount]
  Where=/mnt/FreeUSB2To
  TimeoutIdleSec=10

[Install]
  WantedBy=multi-user.target
```

Activation

    sudo systemctl enable mnt-FreeUSB2To.automount --now

#### Partage avec Lenovo serveur NFS


**PC1**  
Points de montage : `sudo mkdir /mnt/sharenfs`   
Lien : `sudo ln -s /mnt/sharenfs $HOME/sharenfs`

Ajouter les points de montage du serveur nfs au fichier `/etc/fstab`

```
# Les montage NFS du serveur lenovo 192.168.0.215
192.168.0.215:/ /mnt/sharenfs nfs4 nofail,x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=10s,rsize=8192,wsize=8192
```

Rechargement et montage

    sudo systemctl daemon-reload && sudo mount-a


#### Disque Lenovo partagé thinkshare (INACTIF)

**FreeBox**  
Nom de partage : thinkshare

**PC1**  
Partage linux samba : `sudo pacman -S cifs-utils` Installé par défaut   
Point de montage : `sudo mkdir /mnt/thinkshare`   
Lien : `sudo ln -s /mnt/thinkshare $HOME/thinkshare`

Credential : /root/.smbthink avec 2 lignes  
username=XXXXXX  
password=XXXXXX  

Droits

```shell
sudo chown -R root:root /root/.smbthink
sudo chmod -R 600 /root/.smbthink
```

Les fichiers systèmes

```
# /etc/systemd/system/mnt-thinkshare.mount 
[Unit]
  Description=cifs mount script
  Requires=network-online.target
  After=network-online.service

[Mount]
  What=//192.168.0.215/thinkshare
  Where=/mnt/thinkshare
  Options=credentials=/root/.smbthink,rw,uid=1000,gid=1000,vers=3.0
  Type=cifs

[Install]
  WantedBy=multi-user.target

# /etc/systemd/system/mnt-thinkshare.automount 
[Unit]
  Description=cifs mount script
  Requires=network-online.target
  After=network-online.service

[Automount]
  Where=/mnt/thinkshare
  TimeoutIdleSec=10

[Install]
  WantedBy=multi-user.target
```

Activation

    sudo systemctl enable mnt-thinkshare.automount --now

#### Dossiers et Liens

LVM , ajout media

    sudo lvcreate -L 500G vg0 -n lvmedia
    sudo mkfs.ext4 /dev/mapper/vg0-lvmedia
    sudo blkid -s UUID -o value /dev/mapper/vg0-lvmedia

renvoie l'UUID du volume racine :  **1ca4bfc7-3d31-4859-aeb3-656214fab490**.  
Ajouter au fstab  

```
# /dev/mapper/vg0-lvmedia
UUID=1ca4bfc7-3d31-4859-aeb3-656214fab490 	/srv/media      ext4            rw,relatime     0 2
```

Les dossiers et **Liens sur les autres unités** et les dossiers `.keepassx` , `Notes` , `scripts` `statique/images` et `statique/_posts` 

```
sudo mkdir -p /srv/media/{Notes,statique}  
sudo mkdir -p /srv/media/statique/{images,_posts}
sudo ln -s /srv/media $HOME/media
sudo mkdir -p /virtuel  
sudo ln -s /virtuel $HOME/virtuel 
mkdir -p ~/{.ssh,.keepassx}
```

Créer liens sharenfs

```bash
ln -s /mnt/sharenfs/pc1/.borg $HOME/.borg
ln -s /mnt/sharenfs/pc1/scripts $HOME/scripts
```

**Liens "statique" et "Notes"**  
les liens pour la rédaction des posts markdown et le dossier des fichiers

```bash
# Lien pour affichage des images avec éditeur Retext
sudo ln -s /srv/media/statique/images /images
# Lien pour les fichiers autres
sudo ln -s /srv/media/statique/files /files
```

les liens pour les images de Nextcloud Notes

    sudo mkdir /div
    sudo ln -s /srv/media/img /div/img


**Dossiers Documents et Musique**  
Supprimer les dossiers par défaut Documents et Musique et créer des liens 

```
# suppression Documents et Musique
sudo rm -r $HOME/Documents
sudo rm -r $HOME/Musique
#Création des liens
sudo ln -s /srv/media/Documents $HOME/Documents
sudo ln -s /mnt/sharenfs/Musique $HOME/Musique
```

## EndeavourOS XFCE

### Mise à jour EndeavourOS

#### Mode graphique  

![](eos-cassini-009.png){:width="400"}  
![](eos-cassini-009a.png){:width="400"}  
![](eos-cassini-009c.png){:width="300"}  

![](eos-cassini-010.png){:width="400"}  
![](eos-cassini-010a.png){:width="400"}  
![](eos-cassini-010b.png){:width="300"}  

![](eos-cassini-011.png){:width="400"}  
![](eos-cassini-011a.png){:width="400"}  
![](eos-cassini-011b.png){:width="300"}  
![](eos-cassini-011c.png){:width="400"}  

#### Résumé  

![](eos-welcome.png)

### Déverrouillage des volumes LUKS2

Description

* Slot 0 pour le déverrouillage du volume par saisie d'une phrase mot de passe. 
* Slot 1 et 2 pour le déverrouillage par des clés (2 ième clé en cas de perte ou casse) avec un appui sur une touche. 
* Slot 3 - Ajout d'une phrase mot de passe pour le recovery

Au final nous aurons 4 "slot" utilisés 

Installer librairie libfido2 pour la prise en charge des clés Yubico et SoloKeys

    pacman -S libfido2

#### Enroler clé USB YubiKey 5 NFC

![](yubikey5nfc.png){:height="150"}

Vérifier que la YubiKey est insérée dans un port USB

Lister et enroler la yubikey

    systemd-cryptenroll --fido2-device=list

```
PATH         MANUFACTURER PRODUCT              
/dev/hidraw5 Yubico       YubiKey OTP+FIDO+CCID
```

Enroler la clé pour le déverrouillage du disque chiffré nvme0n1p2

    sudo systemd-cryptenroll --fido2-device=auto /dev/nvme0n1p2

```
🔐 Please enter current passphrase for disk /dev/nvme0n1p2: *********************   
Requested to lock with PIN, but FIDO2 device /dev/hidraw5 does not support it, disabling.
Initializing FIDO2 credential on security token.
👆 (Hint: This might require confirmation of user presence on security token.)
Generating secret key on FIDO2 security token.
👆 In order to allow secret key generation, please confirm presence on security token.
New FIDO2 token enrolled as key slot 1.
```

Le **Y** de la clé se met à clignoter , il suffit de poser son doigt sur l'emplacement du **Y** pour le déverrouillage
{: .prompt-info }

Retirer la première clé et insérer la seconde clé USB YubiKey 5 NFC, puis exécuter la commande

    sudo systemd-cryptenroll --fido2-device=auto /dev/nvme0n1p2

```
🔐 Please enter current passphrase for disk /dev/nvme0n1p2: *********************   
Requested to lock with PIN, but FIDO2 device /dev/hidraw5 does not support it, disabling.
Initializing FIDO2 credential on security token.
👆 (Hint: This might require confirmation of user presence on security token.)
Generating secret key on FIDO2 security token.
👆 In order to allow secret key generation, please confirm presence on security token.
New FIDO2 token enrolled as key slot 2.
```

#### Enroler une passphrase de recouvrement

Les jetons et puces de sécurité FIDO2, PKCS#11 et TPM2 s'associent bien avec les clés de recouvrement : puisque vous n'avez plus besoin de taper votre mot de passe tous les jours, il est logique de vous en débarrasser et d'enregistrer à la place une clé de recouvrement à forte entropie que vous imprimez ou scannez hors écran et conservez dans un endroit physique sûr.  
Voici comment procéder :

    sudo systemd-cryptenroll --recovery-key /dev/nvme0n1p2

```
🔐 Please enter current passphrase for disk /dev/nvme0n1p2: ***********             
A secret recovery key has been generated for this volume:

    🔐 vbcrnbjn-vkrkihte-rctbufne-nlihihjl-tegudteu-rkjthcgd-hvhuvgik-rugeregh

Please save this secret recovery key at a secure location. It may be used to
regain access to the volume if the other configured access credentials have
been lost or forgotten. The recovery key may be entered in place of a password
whenever authentication is requested.
New recovery key enrolled as key slot 3.
```

Cette opération génère une clé, l'enregistre dans le volume LUKS2, l'affiche à l'écran et génère un code QR que vous pouvez scanner en dehors de l'écran si vous le souhaitez.  
La clé possède la plus grande entropie et peut être saisie partout où vous pouvez saisir une phrase d'authentification.  
C'est pourquoi il n'est pas nécessaire de modifier le fichier /etc/crypttab pour que la clé de récupération fonctionne.

#### Enroler une clé USB SoloKeys (OPTIONNEL)

![](solokeys.png)

Lister la clé

    systemd-cryptenroll --fido2-device=list

```
PATH         MANUFACTURER PRODUCT   
/dev/hidraw4 SoloKeys     Solo 4.1.5
```

Ajout de la solokeys

    sudo systemd-cryptenroll --fido2-device=auto /dev/nvme0n1p2

```
🔐 Please enter current passphrase for disk /dev/nvme0n1p2: ***********             
Requested to lock with PIN, but FIDO2 device /dev/hidraw1 does not support it, disabling.
Initializing FIDO2 credential on security token.
👆 (Hint: This might require confirmation of user presence on security token.)
Generating secret key on FIDO2 security token.
👆 In order to allow secret key generation, please confirm presence on security token.
New FIDO2 token enrolled as key slot 3.
```

Lors du boot , le **S** de la SoloKeys passe au ROUGE et il suffit d'appuyer sur le voyant pour qu'il repasse  au vert afin de lancer le processus de déchiffrement et finir le démarrage
{: .prompt-info }

#### Prise en charge YubiKey et SoloKey

Les options timeout de [crypttab](https://www.man7.org/linux/man-pages/man5/crypttab.5/)

```
timeout=
           Spécifie le délai d'attente pour la demande d'un mot de passe. Si aucune unité
           n'est spécifiée, l'unité utilisée est la seconde. Les unités prises en charge sont s, ms, us,
           min, h, d. Un délai de 0 permet d'attendre indéfiniment (valeur par défaut).

token-timeout=
           Spécifie le temps d'attente maximum pour que les dispositifs de sécurité configurés (c'est-à-dire FIDO2, PKCS#11, TPM2) apparaissent.
           configurés (c'est-à-dire FIDO2, PKCS#11, TPM2). Prend une valeur
           en secondes (mais d'autres unités de temps peuvent être spécifiées,
           voir systemd.time(7) pour les formats supportés). La valeur par défaut est 30s.
           Une fois le délai spécifié écoulé, l'authentification par
           mot de passe est tentée. Notez que ce délai s'applique à
           l'attente de l'apparition du dispositif de sécurité - il ne s'applique pas
           ne s'applique pas à la demande de code PIN pour le dispositif (le cas échéant)
           ou autre. Passez 0 pour désactiver le délai et attendre indéfiniment.
```

Configurer /etc/crypttab pour la prise en charge des clés

    sudo nano /etc/crypttab

```
# <name>               <device>                         <password> <options>
#cryptlvm UUID=2c8e7bb4-9286-47e9-8823-12b79bf2810c /crypto_keyfile.bin luks
cryptlvm UUID=2c8e7bb4-9286-47e9-8823-12b79bf2810c - fido2-device=auto,token-timeout=20s
```

Sauvegarder et quitter.

Réinitialiser

    sudo reinstall-kernels

`Redémarrer la machine`{: .prompt-info }

### Plymouth - Processus de démarrage graphique

[Plymouth - Processus de démarrage graphique](/posts/Plymouth_Processus_de_demarrage_graphique/)

### Pavé numérique au boot

**Avec un service systemd**

installer le paquetage AUR et en activer le service numLockOnTty

    yay -S systemd-numlockontty 
    sudo systemctl enable numLockOnTty --now

### Paramètres XFCE

#### Tableau de bord

On déplace le **tableau de bord** du bas vers le haut de l'écran  

Modification du **tableau de bord** , clic-droit &rarr; Tableau de bord &rarr; Préférences de tableau de bord &rarr; Eléments  

Affichage date et heure  
![](eos-cassini-012.png)  
ou **format personnalisé** dans **Horloge** : `%e %b %Y %R`  

Gestionnaire d'alimentation (Batterie et Branché)  
![](eos-cassini-013.png){:width="400"}  

#### Fond écran

Facultatif - Les fonds d'écran &rarr;  `/usr/share/endeavouros/backgrounds/`

#### LightDM

*Utilise `lightdm-slick-greeter` Un greeter basé sur GTK plus axé sur l'apparence que `lightdm-gtk-greeter`*

Les paramètres sont dans le fichier `/etc/lightdm/slick-greeter.conf` 

```
[Greeter]
background=/usr/share/endeavouros/backgrounds/endeavouros-wallpaper.png
draw-user-backgrounds=false
draw-grid=true
theme-name=Arc-Dark
icon-theme-name=Qogir
cursor-theme-name=Qogir
cursor-theme-size=16
show-a11y=false
show-power=false
background-color=#000000
```

Démarre auto ou pas de la session, modifier le fichier `/etc/lightdm/lightdm.conf` ', (début ligne sans ou avec commentaire `#`)

    sudo nano /etc/lightdm/lightdm.conf

```
[Seat:*]
autologin-user=yann
```

Ecran principal pour la fenêtre de connexion : [EndeavourOS XFCE - LightDM sur les systèmes multi-affichages](/posts/EndeavourOS_XFCE_-_Environnements_de_bureau_LightDM/)

Si vous changez l'image de fond, il désactiver draw-grid

```
background=/usr/share/endeavouros/backgrounds/light_sky_stars_85555_1366x768_yano.jpg
draw-grid=false
```

### Applications et Paquets 

On commence par tout ce qui est graphique : gimp, cups (gestion de l’imprimante) et hplip (si vous avez une imprimante scanner Hewlett Packard). Le paquet python-pyqt5 est indispensable pour l’interface graphique de HPLIP+scan. Webkigtk2 étant indispensable pour la lecture de l’aide en ligne de Gimp. outil rsync, Retext éditeur markdown, firefox fr, thunderbird, libreoffice, gdisk, bluefish, **Double Commander** , **Menulibre** pour la gestion des menus , outils android clementine    

```bash
yay -S cups system-config-printer gimp hplip libreoffice-fresh-fr thunderbird-i18n-fr jq figlet p7zip xsane tmux  calibre retext bluefish gedit doublecmd-gtk2 terminator filezilla minicom zenity android-tools yt-dlp qrencode zbar xclip nmap jre-openjdk-headless openbsd-netcat borg xterm gparted tigervnc xournalpp qbittorrent ldns

# Fuse borg mount
yay -S python-llfuse

# Scripts to aid in installing Arch Linux (ex: arch-chroot)
yay -S arch-install-scripts

# Autres avec compilation
yay -S freetube brave-nightly signal-desktop
```

Gestion des menus du bureau, construction du paquet avant installation

    yay -S menulibre 

Firefox en français

    yay -S firefox-i18n-fr

#### Tmux

<details>
<summary><b>Etendre Réduire fichier de configuration "~/.tmux.conf"</b></summary>

{% highlight text %} 
#Configuration de tmux
#Origine : http://denisrosenkranz.com
#Yannick juin 2017
# Copier/Coller par la souris se fait avec la touche "Shift" appuyée
 
##################################
#Changements des raccourcis claviers
##################################
#On change Control +b par Control +x
#set -g prefix C-x
#unbind C-b
#bind C-x send-prefix
 
#On utilise control + flèches pour naviguer entre les terminaux
bind-key -n C-right next
bind-key -n C-left prev
 
#on utilise alt + flèches our naviguer entre les panels
bind-key -n M-left select-pane -L
bind-key -n M-right select-pane -R
bind-key -n M-up select-pane -U
bind-key -n M-down select-pane -D
 
#On change les raccourcis pour faire du split vertical et horizontal
#On utilise la touche "|" (pipe) pour faire un split vertical
bind | split-window -h
#Et la touche "-" pour faire un split horizontal
bind - split-window -v
 
##################################
#Changements pratiques
##################################
#On permet l'utilisation de la souris pour changer de terminal et de panel
set -g mouse on

# Sélection zone par clic gauche souris (texte sélectionné sur fond jaune)
# Après relachement du clic , le texte sélectionné est copié dans le presse-papier 
# Le fond jaune disparaît
set-option -s set-clipboard off
# For emacs copy mode bindings
# Il faut installer l'utilitaire 'xclip' (sudo pacman -S xclip)
bind-key -T copy-mode MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel "xclip -selection clipboard -i"

#Les fenêtres commencent par 1 et non par 0
set -g base-index 1
 
##################################
#Changements visuels
##################################
#On met les panneaux non actif en gris
#set -g pane-border-fg colour244
#set -g pane-border-bg default
 
#On met le panneau actif en rouge
#set -g pane-active-border-fg colour124
#set -g pane-active-border-bg default
 
#On met la barre de status en gris
set -g status-fg colour235
set -g status-bg colour250
#set -g status-attr dim
 
# On surligne les fenêtres actives dans la barre de status en gris foncés
#set-window-option -g window-status-current-fg colour15
#set-window-option -g window-status-current-bg colour0
{% endhighlight %}

</details>

#### Minicom

Paramétrage de l'application terminale **minicom**

     sudo minicom -s

>Seul les paramètres à modifier sont cités

Configuration du port série  
![](minicom01.png)  
A -                             Port série : **/dev/ttyUSB0**   
F -              Contrôle de flux matériel : **Non**  
![](minicom02.png)  
Echap  
Enregistrer config. sous dfl  
![](minicom03.png)  
Sortir de Minicom  

#### Flameshot (copie écran)

**Copie écran (flameshot)**  
**[Flameshot](https://github.com/lupoDharkael/flameshot)** c’est un peu THE TOOL pour faire des captures d’écrans

    yay -S flameshot

Lancer l'application XFCE Flameshot et l'icône est visible dans la barre des tâches  
![](flameshot_e6230-1a.png){:width="300"}  

Paramétrage de flameshot, clic droit sur icône , Configuration   
![](flameshot_e6230-1b.png){:width="300"}  
Paramétrage de flameshot  
![](flameshot01.png){:width="300"}

#### scrpy émulation android

Utilise adb et le port USB

    yay -S scrcpy

`Ce qui suit n'est pas nécessaire car l'installation crée l'icône de lancement de l'application`{: .prompt-warning }

Créer le dossier 

    mkdir -p $HOME/.local/share/applications

Créer le fichier `$HOME/.local/share/applications/scrcpy-android.desktop` avec le contenu suivant

```
[Desktop Entry]
Version=1.1
Type=Application
Name=ScrCpy (Android)
Comment=Votre smartphone sur le bureau
Icon=phone
Exec=/usr/bin/scrcpy
Path=/home/yann
Actions=
Categories=Utility;X-XFCE;X-Xfce-Toplevel;
Terminal=false
StartupNotify=false
```

#### Client Nextcloud

Installation client nextcloud

    yay -S nextcloud-client libgnome-keyring gnome-keyring  

Démarrer le client nextcloud , après avoir renseigné l'url ,login et mot de passe pour la connexion  

Trousseau de clé avec mot de passe idem connexion utilisateur

Paramétrage

* Menu &rarr; Lancer **Client de synchronisation nextcloud**
* Adresse du serveur : <https://cloud.xoyaz.xyz>  
![](nextcloud_xfce01.png){:width="300"}  
* Nom d’utilisateur : yann
* Mot de passe : xxxxx  
![](nextcloud_xfce02.png){:width="200"}  
![](nextcloud_xfce03.png){:width="300"}  
![](nextcloud_xfce04.png){:width="200"}  
* Sauter les dossiers à synchroniser, Ignorer la configuration des dossiers
* Trousseau de clés = mot de passe connexion utilisateur  
![](nextcloud_xfce05.png){:width="400"}  
* Paramètres nextcloud  
![](e6230-nextcloud-a.png){:width="400"}

Saisir les différents dossiers à synhroniser  
![](e6230-nextcloud.png){:width="400"}

Au prochain redémarrage, il faudra confirmer le mot de passe du trousseau

#### Gestion mot de passe (keepassxc)

![](KeePassXC.png){:width="50"}  
Ajouter une synchronisation de dossier nextcloud : /home/yano/.keepassx (local) &rarr; Home/.keepasx (serveur)  
Télécharger la clé **yannick_keepassxc.key** dans **~/.ssh** 

```shell
scp -P 56230 -i ~/.ssh/e6230  ~/.ssh/yannick_keepassxc.key yano@192.168.0.20:/home/yano/.ssh/
```

Installer keepassxc 

    yay -S keepassxc

Ajouter aux favoris "KeepassXC" et lancer l'application &rarr; **Ouvrir une base de données existante**  
Base de données --> Ouvrir une base de données (afficher les fichiers cachés) : **~/.keepassx/yannick_xc.kdbx** --> Ouvrir  
![](e6230-keepassx01.png){:width="400"}

**Affichage &rarr; Thème** : Sombre  
**Affichage &rarr; Mode compact**  , un redémarrage de l'application est nécessaire  

#### SSHFS (facultatif)

![](sshfs-logo.png){:width="50"}  
*SSHFS sert à monter sur son système de fichier, un autre système de fichier distant, à travers une connexion SSH, le tout avec des droits utilisateur.*

Installer paquet SSHFS 

	sudo pacman -S sshfs 

sshfs est installé par défaut sur la distribution EndeavourOS
{: .prompt-warning }


Création des partages utilisés par sshfs (facultatif)

    mkdir -p $HOME/vps/{borgbackup,lxc,vdb,xoyaz.xyz,xoyize.xyz}

Exemple de montage manuel  
`sshfs -oIdentityFile=<clé privée> utilisateur@domaine.tld:<dossier distant> <dossier local> -C -p <port si dfférent de 22>`  

#### Thunderbird

Lancer thunderbird à l'ouverture de session xfce  
Paramètres &rarr; Session et démarrage &rarr; Démarrage automatique d'application  
![](thunderbird01.png){:width="300"}

Ajouter thunderbird aux favoris et lancer

**Comptes de messagerie**

* Paramètrer les différents compte de messagerie  
* Compte ProtonMail 
    1. Comment paramétrer Mozilla Thunderbird pour ProtonMail Bridge, ouvrir le lien suivant [Proton Mail](/posts/Proton_Mail/)
    * Paramétrer le compte de messagerie ProtonMail

*Si vous souhaitez que Thunderbird soit minimisé dans la zone de notification, vous devez installer une application indépendante pour déplacer Thunderbird dans la zone de notification chaque fois qu'il est minimisé. Sous Linux, je recommande KDocker (disponible dans de nombreuses distributions Linux).*

*KDocker est pratique lorsque vous souhaitez intégrer une application graphique dans la barre d'état système, étant donné que l'application en question ne dispose pas de sa propre fonctionnalité pour la placer dans la barre d'état système. Bien qu'elle n'ait pas été mise à jour depuis 2005, la dernière version publiée le 5 avril 2005 est suffisamment bonne et, selon le site officiel, elle fonctionne avec tous les gestionnaires de fenêtres conformes à la norme NET WM. Pour n'en citer que quelques-uns : KDE, GNOME, Xfce, Blackbox ou Fluxbox. Je ne l'ai utilisé que dans KDE 3.5.9, mais je suis sûr qu'il fonctionne bien dans les autres environnements de bureau aussi, si vous ne voulez pas utiliser une application d'ancrage native, comme ALLTray pour GNOME.*

* Paramètres &rarr; Modules complémentaires et thèmes
    * **Thèmes** : Activer **sombre**
    * **Extensions** : Installer **Minimize on Close** 

**Calendriers et contacts**

`Alt+m` pour afficher la bare de menu  
![](e6230-thunderbird02.png){:width="400"}

* **Calendrier**  
![](e6230-thunderbird03.png){:width="400"}  
![](e6230-thunderbird04.png){:width="400"}  
![](e6230-thunderbird05.png){:width="400"}  
Saisir le mot de passe  
![](e6230-thunderbird06.png){:width="400"}  
![](e6230-thunderbird07.png){:width="400"}  
* **Contacts**  
Outils &rarr; Carnet d'adresses  
![](e6230-thunderbird08.png){:width="400"}  
![](e6230-thunderbird09.png){:width="250"}  
![](e6230-thunderbird10.png){:width="250"}  
![](e6230-thunderbird11.png){:width="250"}  

#### Radio via internet (facultatif)

Installation au choix

    yay -S radiotray
    yay -S geocode-glib tuner-git

#### Gestionnaire de fichiers

*Double Commander est un gestionnaire de fichiers open source multiplateforme avec deux panneaux côte à côte. Il s'inspire de Total Commander*

Application GTK ou QT

    yay -S doublecmd-gtk2
    yay -S doublecmd-qt5

Les paramètres sont stockés dans le dossier `~/.config/doublecmd`

#### bashrc - alias et couleurs

Dans le fichier ~/.bashrc  

    alias

```
alias aide='xdg-open https://static.lxcdeb.local/aide-jekyll-text-theme/#autres-styles'
alias android='/home/yann/virtuel/KVM/bliss.sh'
alias audio='yt-dlp --extract-audio --audio-format m4a --audio-quality 0 --output "~/Musique/%(title)s.%(ext)s"'
alias audiomp3='yt-dlp --extract-audio --audio-format mp3 --audio-quality 0 --output "~/Musique/%(title)s.%(ext)s"'
alias borglist='/home/yann/scripts/borglist.sh'
alias certok='/home/yann/scripts/ssl-cert-check'
alias compress='/home/yann/scripts/compress'
alias dnsleak='/home/yann/scripts/dnsleaktest.py'
alias domain='/home/yann/scripts/domain-check-2.sh -f /home/yann/scripts/domain-list.txt'
alias findh='cat /home/yann/scripts/findhelp.txt'
alias homer='/home/yann/media/dplus/python-dev/homer/remoh.py'
alias ipleak='curl https://ipv4.ipleak.net/json/'
alias l='ls -lav --ignore=.?*'
alias ll='ls -lav --ignore=..'
alias ls='ls --color=auto'
alias mediasync='/home/yann/scripts/sav-yann-media.sh'
alias nmapl='sudo nmap -T4 -sP 192.168.0.0/24'
alias odt/='/home/yann/scripts/_odt/+index'
alias orphelin='sudo pacman -Rsn $(pacman -Qdtq)'
alias otp='/home/yann/scripts/generer-code-2fa-vers-presse-papier-toutes-les-30s.sh'
alias rename='/home/yann/scripts/remplacer-les-espaces-accents-dans-une-expression.sh'
alias service='systemctl --type=service'
alias sshm='/home/yann/scripts/ssh-manager.sh'
alias ssl='/home/yann/scripts/ssl-cert-check'
alias static='cd /home/yann/media/yannstatic; /home/yann/.rbenv/shims/bundle exec jekyll build -d /home/yann/media/yannstatic/static; cd ~'
alias status='/home/yann/scripts/status.sh'
alias toc='/home/yann/scripts/toc/toc.sh'
alias tocplus='/home/yann/scripts/toc/tocplus.sh'
alias traduc='/usr/local/bin/trans'
alias vncasus='sh /home/yann/scripts/vncasus.sh'
alias vncdell='sh /home/yann/scripts/vncdell.sh'
alias vncmarina='sh /home/yann/scripts/vncmarina.sh'
alias x96='adb connect 192.168.0.22:5555'
alias youtube='yt-dlp -f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best" --output "~/Vidéos/%(title)s.%(ext)s" --ignore-errors'
```

Couleurs

Générons un fichier de configuration par défaut en utilisant l' option –print-database de la commande dircolors

    dircolors --print-database > ~/.dir_colors

<details>
<summary><b>Etendre Réduire .dir_colors</b></summary>

{% highlight bash %}  
# Configuration file for dircolors, a utility to help you set the
# LS_COLORS environment variable used by GNU ls with the --color option.
# Copyright (C) 1996-2023 Free Software Foundation, Inc.
# Copying and distribution of this file, with or without modification,
# are permitted provided the copyright notice and this notice are preserved.
#
# The keywords COLOR, OPTIONS, and EIGHTBIT (honored by the
# slackware version of dircolors) are recognized but ignored.
# Global config options can be specified before TERM or COLORTERM entries
# ===================================================================
# Terminal filters
# ===================================================================
# Below are TERM or COLORTERM entries, which can be glob patterns, which
# restrict following config to systems with matching environment variables.
COLORTERM ?*
TERM Eterm
TERM ansi
TERM *color*
TERM con[0-9]*x[0-9]*
TERM cons25
TERM console
TERM cygwin
TERM *direct*
TERM dtterm
TERM gnome
TERM hurd
TERM jfbterm
TERM konsole
TERM kterm
TERM linux
TERM linux-c
TERM mlterm
TERM putty
TERM rxvt*
TERM screen*
TERM st
TERM terminator
TERM tmux*
TERM vt100
TERM xterm*
# ===================================================================
# Basic file attributes
# ===================================================================
# Below are the color init strings for the basic file types.
# One can use codes for 256 or more colors supported by modern terminals.
# The default color codes use the capabilities of an 8 color terminal
# with some additional attributes as per the following codes:
# Attribute codes:
# 00=none 01=bold 04=underscore 05=blink 07=reverse 08=concealed
# Text color codes:
# 30=black 31=red 32=green 33=yellow 34=blue 35=magenta 36=cyan 37=white
# Background color codes:
# 40=black 41=red 42=green 43=yellow 44=blue 45=magenta 46=cyan 47=white
#NORMAL 00 # no color code at all
#FILE 00 # regular file: use no color at all
RESET 0 # reset to "normal" color
FILE 01;33
DIR 01;34 # directory
LINK 01;36 # symbolic link. (If you set this to 'target' instead of a
 # numerical value, the color is as for the file pointed to.)
MULTIHARDLINK 00 # regular file with more than one link
FIFO 40;33 # pipe
SOCK 01;35 # socket
DOOR 01;35 # door
BLK 40;33;01 # block device driver
CHR 40;33;01 # character device driver
ORPHAN 40;31;01 # symlink to nonexistent file, or non-stat'able file ...
MISSING 00 # ... and the files they point to
SETUID 37;41 # file that is setuid (u+s)
SETGID 30;43 # file that is setgid (g+s)
CAPABILITY 00 # file with capability (very expensive to lookup)
STICKY_OTHER_WRITABLE 30;42 # dir that is sticky and other-writable (+t,o+w)
OTHER_WRITABLE 34;42 # dir that is other-writable (o+w) and not sticky
STICKY 37;44 # dir with the sticky bit set (+t) and not other-writable
# This is for files with execute permission:
EXEC 01;32
# ===================================================================
# File extension attributes
# ===================================================================
# List any file extensions like '.gz' or '.tar' that you would like ls
# to color below. Put the suffix, a space, and the color init string.
# (and any comments you want to add after a '#').
# Suffixes are matched case insensitively, but if you define different
# init strings for separate cases, those will be honored.
#
# If you use DOS-style suffixes, you may want to uncomment the following:
#.cmd 01;32 # executables (bright green)
#.exe 01;32
#.com 01;32
#.btm 01;32
#.bat 01;32
# Or if you want to color scripts even if they do not have the
# executable bit actually set.
#.sh 01;32
#.csh 01;32
# archives or compressed (bright red)
.tar 01;31
.tgz 01;31
.arc 01;31
.arj 01;31
.taz 01;31
.lha 01;31
.lz4 01;31
.lzh 01;31
.lzma 01;31
.tlz 01;31
.txz 01;31
.tzo 01;31
.t7z 01;31
.zip 01;31
.z 01;31
.dz 01;31
.gz 01;31
.lrz 01;31
.lz 01;31
.lzo 01;31
.xz 01;31
.zst 01;31
.tzst 01;31
.bz2 01;31
.bz 01;31
.tbz 01;31
.tbz2 01;31
.tz 01;31
.deb 01;31
.rpm 01;31
.jar 01;31
.war 01;31
.ear 01;31
.sar 01;31
.rar 01;31
.alz 01;31
.ace 01;31
.zoo 01;31
.cpio 01;31
.7z 01;31
.rz 01;31
.cab 01;31
.wim 01;31
.swm 01;31
.dwm 01;31
.esd 01;31
# image formats
.avif 01;35
.jpg 01;35
.jpeg 01;35
.mjpg 01;35
.mjpeg 01;35
.gif 01;35
.bmp 01;35
.pbm 01;35
.pgm 01;35
.ppm 01;35
.tga 01;35
.xbm 01;35
.xpm 01;35
.tif 01;35
.tiff 01;35
.png 01;35
.svg 01;35
.svgz 01;35
.mng 01;35
.pcx 01;35
.mov 01;35
.mpg 01;35
.mpeg 01;35
.m2v 01;35
.mkv 01;35
.webm 01;35
.webp 01;35
.ogm 01;35
.mp4 01;35
.m4v 01;35
.mp4v 01;35
.vob 01;35
.qt 01;35
.nuv 01;35
.wmv 01;35
.asf 01;35
.rm 01;35
.rmvb 01;35
.flc 01;35
.avi 01;35
.fli 01;35
.flv 01;35
.gl 01;35
.dl 01;35
.xcf 01;35
.xwd 01;35
.yuv 01;35
.cgm 01;35
.emf 01;35
# https://wiki.xiph.org/MIME_Types_and_File_Extensions
.ogv 01;35
.ogx 01;35
# audio formats
.aac 00;36
.au 00;36
.flac 00;36
.m4a 00;36
.mid 00;36
.midi 00;36
.mka 00;36
.mp3 00;36
.mpc 00;36
.ogg 00;36
.ra 00;36
.wav 00;36
# https://wiki.xiph.org/MIME_Types_and_File_Extensions
.oga 00;36
.opus 00;36
.spx 00;36
.xspf 00;36
# backup files
*~ 00;90
*# 00;90
.bak 00;90
.crdownload 00;90
.dpkg-dist 00;90
.dpkg-new 00;90
.dpkg-old 00;90
.dpkg-tmp 00;90
.old 00;90
.orig 00;90
.part 00;90
.rej 00;90
.rpmnew 00;90
.rpmorig 00;90
.rpmsave 00;90
.swp 00;90
.tmp 00;90
.ucf-dist 00;90
.ucf-new 00;90
.ucf-old 00;90
#
# Subsequent TERM or COLORTERM entries, can be used to add / override
# config specific to those matching environment variables.
{% endhighlight %}


</details>

Après avoir modifié le fichier de configuration pour personnaliser le jeu de couleurs

    echo "eval $(dircolors ~/.dir_colors)" >> ~/.bashrc

nous rechargeons le fichier pour appliquer les modifications 

    source ~/.bashrc

#### Imprimante et scanner

Prérequis , paquets **cups cups-filters cups-pdf system-config-printer hplip installés** (Pilotes HP pour DeskJet, OfficeJet, Photosmart, Business Inkjet et quelques modèles de LaserJet aussi bien qu'un certain nombre d'imprimantes Brother)...   

**Assurez-vous que les bons ports sont ouverts.**  
Assurez-vous que les ports 161 (udp et tcp), 162 (udp et tcp) et 9100 (udp et tcp) sont ouverts dans votre pare-feu. Si ces ports ne sont pas ouverts, HPLIP ne fonctionnera pas.


Ouvrir **pare-feu** , saisir le mot de passe  
passer en configuration permanente et activer les services ipp, ipp-client et mdns.  
Cette configuration sera permanente au fil des redémarrages.


On démarre le service cups

```shell
sudo systemctl start cups.service  # lancement cups
sudo systemctl enable cups.service  # activation cups
```

Installer graphiquement l'imprimante  
![](hp7510-00.png){:width="300"}  
![](hp7510-01.png)  
![](hp7510-02.png)  

![](hp7510-03.png){:width="300"}  
Pour contourner le problème , éditer le fichier `/etc/nsswitch.conf`  
Ajouter `mdns_minimal [NOTFOUND=return]` avant `resolve`   
`hosts: mymachines mdns_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] files myhostname dns`
{: .prompt-warning }

Après correctif  
![](hp7510-04.png){:width="300"}  

Avec cups : http://localhost:631/  
![](hp_cups.png)

Installation du scanner  
Coté logiciel il vous faudra **sane** et son interface graphique **xsane**, ainsi qu’éventuellement xsane-gimp le plugin pour gimp.

	yay -S xsane xsane-gimp 

Vérifier si le scaner est reconnu : `sudo scanimage -L`

```
device `escl:https://192.168.0.24:443' is a HP OfficeJet 7510 series [C22036] platen,adf scanner
device `hpaio:/net/officejet_7510_series?ip=192.168.0.24&queue=false' is a Hewlett-Packard officejet_7510_series all-in-one
```

Test scan, placer un original pour photocopie

    scanimage --device hpaio:/net/officejet_7510_series?ip=192.168.0.24 --format=png > test.png

#### Son Casque/HDMI

*Ceux qui travaillent à domicile sur une machine Linux ont peut-être déjà remarqué qu'il est difficile de passer d'un périphérique audio d'entrée à un périphérique audio de sortie. Vous pouvez vouloir passer rapidement d'un casque à un autre ou du haut-parleur au casque et vice versa. Ouvrir la boîte de dialogue des paramètres à chaque fois que vous voulez changer de périphérique audio n'est pas très productif. Certaines distributions de bureau comme Cinnamon fournissent des solutions prêtes à l'emploi pour changer le périphérique audio en quelques clics. Pour Gnome, il existe une extension qui active cette fonctionnalité. Cet article couvre ces options qui permettent de passer d'un périphérique audio à l'autre sans trop d'efforts.[Easily Switch Audio Devices on Linux](https://www.linuxedo.com/2021/11/easily-switch-audio-devices-on-linux/)*

**Toutes les distributions Linux**  
[Sound Switcher Indicator](https://github.com/yktoo/indicator-sound-switcher) est une application simple qui permet de changer l'audio d'entrée et de sortie à partir de l'icône de la barre des tâches. Contrairement aux deux premières options, cette application n'est pas limitée à une distribution particulière. Si vous n'aimez pas les longs noms de matériel, l'application propose également une option permettant de renommer les périphériques détectés.

Les utilisateurs d'Arch peuvent utiliser la commande suivante pour installer l'application

    yay -S indicator-sound-switcher

![](Switcher_Indicator01.png)   


![](Switcher_Indicator02.png) 


#### MariaDB - DBeaver

[MariaDB archlinux](/posts/MariaDB-sur-Debian-Stretch/#archlinux)

Résumé des commandes en mode su

```shell
pacman  -S mariadb
mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
systemctl enable mariadb --now
systemctl status --no-pager --full mariadb --full
```

Sécuriser

    sudo mysql_secure_installation

*DBeaver est basé sur le framework Eclipse, il est open source et il supporte plusieurs types de serveurs de bases de données comme : MySQL, SQLite, DB2, PostgreSQL, Oracle...*

    yay -S dbeaver

```
:: jdk-openjdk et jre-openjdk-headless sont en conflit. Supprimer jre-openjdk-headless ? [o/N] o
```

#### FreeTuxTv

*FreetuxTV est une application qui permet de regarder et enregistrer facilement les chaînes de télévision sous GNU/Linux et les chaînes de télévision de votre fournisseur d'accès internet.*

En mode su

Installation

    sudo pacman -S freetuxtv

Paramétrage du parefeu firewalld ([Configuration de firewalld pour le multicast VLC freebox](https://forums.fedora-fr.org/d/59161-configuration-de-firewalld-pour-le-multicast-vlc-freebox-de-chez-free))

Créer les services **mafreebox.xml** et **vlc.xml** pour firewalld

**mafreebox.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>mafreebox</short>
  <description>Permission pour mafreebox et vlc</description>
  <port protocol="udp" port=""/>
</service>
```

**vlc.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>vlc2</short>
  <description>Permission pour mafreebox et vlc</description>
  <port protocol="udp" port="15947"/>
  <destination ipv4="228.67.43.91"/>
</service>
```

Les ajouter à la **zone public** pour rendre ces règles permanentes

/etc/firewalld/zones/public.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="ipp-client"/>
  <service name="mdns"/>
  <service name="ipp"/>
  <service name="mafreebox"/>
  <service name="vlc"/>
  <forward/>
</zone>
```

Recharger le parefeu

    firewall-cmd --reload

#### Photini (Facultatif)

*Photini est un outil Linux qui permet de visualiser, mais également d'éditer les métadonnées de vos photos*

[Installation](https://photini.readthedocs.io/en/latest/other/installation/)

Installer et vérifier python et pip

```
python -V   # Python 3.11.5
sudo pacman -S python-pip
pip --version   # pip 23.2.1 from /usr/lib/python3.11/site-packages/pip (python 3.11)
```

Avant d'installer Photini, vous devez décider si vous l'installez pour un seul utilisateur ou pour plusieurs. Les installations multi-utilisateurs utilisent un environnement virtuel Python pour créer une installation autonome qui peut être facilement partagée. L'utilisation d'un environnement virtuel présente d'autres avantages, tels qu'une désinstallation facile, et vous pouvez donc également l'utiliser pour une installation mono-utilisateur.
{: .prompt-info }

**Environnement virtuel**  
Configurer avec le nom photini et création dans le répertoire personnel 

```
python -m venv photini --system-site-packages
source photini/bin/activate
python -m pip install -U pip
```

Notez que pip peut avoir besoin d'être mis à jour à nouveau à partir de l'environnement virtuel. L'option Linux / MacOS --system-site-packages rend les paquets installés avec le gestionnaire de paquets du système (par exemple PySide6 / PySide2 / PyQt6 / PyQt5) disponibles dans l'environnement virtuel. Vous devez rester dans cet environnement virtuel pendant l'installation et les tests de Photini.

**Installer Photini avec pip**

    pip install photini

Les dépendances optionnelles de Photini peuvent être incluses dans l'installation en les listant comme "extras" dans la commande pip. Par exemple, si vous souhaitez pouvoir télécharger sur Flickr et Ipernity

    pip install "photini[flickr,ipernity]"

`Notez que les noms des extras ne sont pas sensibles à la casse.`{: .prompt-info }

Nouveau dans la version 2023.7.0 : Vous pouvez installer toutes les dépendances optionnelles de Photini en ajoutant un "all extra". Vous pouvez également installer n'importe quel paquet Qt en tant qu'extra 

    pip  install "photini[all,pyqt6,pyside6]"

Lancez maintenant la commande `photini-configure` pour choisir le paquetage Qt à utiliser

    photini-configure

Liste des réponses aux questions

```
Which Qt package would you like to use?
  0 PyQt5 [installed, WebEngine not installed]
  1 PySide2 [not installed]
  2 PyQt6 [installed]
  3 PySide6 [installed]
Choose 0/1/2/3: 2
Would you like to upload pictures to Flickr? (y/n): n
Would you like to upload pictures to Google Photos? (y/n): n
Would you like to upload pictures to Ipernity? (y/n): n
Would you like to upload pictures to Pixelfed or Mastodon? (y/n): n
Would you like to check spelling of metadata? (y/n) [y]: n
Would you like to import GPS track data? (y/n) [y]: y
Would you like to make higher quality thumbnails? (y/n) [y]: n
Would you like to import pictures from a camera? (y/n): n
```

Tester l'installation : `python -m photini`  
![](photini01.png)

**Menu de démarrage / menu d'application**  
Bien que vous puissiez lancer Photini à partir d'un shell de commande, la plupart des utilisateurs préfèreront probablement utiliser le menu Démarrer / Application ou une icône sur le bureau. Ceux-ci peuvent être installés avec la commande `photini-post-install` 

**Utilisateurs supplémentaires**  
Si vous avez installé Photini dans un environnement virtuel, d'autres utilisateurs devraient pouvoir exécuter la commande photini en utilisant son chemin d'accès complet.(exemple utilisatrice sarah)

```
# dans le dossier home de sarah
sarah@mint:~$/home/yann/photini/bin/photini  
```

Ce n'est pas une façon très pratique de lancer Photini, donc la plupart des utilisateurs voudront l'ajouter à leur menu de démarrage ou d'application  

```
# dans le dossier home de sarah
sarah@mint:~$ /home/yann/photini/bin/photini-post-install
```

#### Météo Radar

![](wetteronline.png)  
<https://www.meteoetradar.com/meteo/beaupreau-en-mauges/9983268>

#### Flatpak (NON INSTALLE)

*Flatpak offre une plate-forme universelle pour installer, gérer et désinstaller les logiciels sur toutes les distributions Linux.* 

* [Flatpak, un format de paquets universel](https://doc.ubuntu-fr.org/flatpak)
* [Flatpak : Télécharger et installer des applications sur Linux](https://www.malekal.com/flatpak-telecharger-installer-applications-linux/)

Installer flatpak

    yay -S flatpak

Accédez à la plateforme flashhub : <https://flathub.org/home>  
Rechercher l'application "FreeTube"  
![](flatpak01.png)  
cliquez sur le bouton bleu **Install** pour télécharger le paquet `io.freetubeapp.FreeTube.flatpakref`  

Ouvrir un terminal et lancer la commande

    sudo flatpak install io.freetubeapp.FreeTube.flatpakref

![](flatpak02.png)  

`Les fichiers de configuration des logiciels installés ne sont pas dans les répertoires "classiques" ~/.config ou ~/.local, ils sont dans ~/.var`{: .prompt-warning }

`Les icônes et fichiers des applications Flatpak ne se trouvent pas dans /usr/share/, mais dans /var/lib/flatpak/exports/share/`{: .prompt-warning }

Autres commandes:  
Lister les applications installées `flatpak list`  
Afficher l'historique &rarr; `sudo flatpak history`  
Désinstaller un logiciel &rarr; `sudo flatpak uninstall <nom paquet>`, ex FreeTube &rarr; `sudo flatpak uninstall FreeTube`  
Dans certains cas, vous pouvez vous retrouver avec une installation corrompue d’un paquet &rarr; `sudo flatpak repair`

Désinstallation complète

    sudo flatpak uninstall --unused

#### VSCodium (NON INSTALLE)

*Si le code de Visual Studio est bien libre, l’application officielle de Microsoft ne l’est pas vraiment. C’est ici qu’entre en jeu VSCodium, une alternative à Visual Studio Code sous licence MIT, disponible pour de nombreux systèmes d’exploitation*

[VSCodium, éditeur de code source multiplateforme et multi langage](/posts/VSCodium/)

#### Synchro serveur dossier "BiblioCalibre"

Le but est de synchroniser le dossier **~/media/BiblioCalibre** avec le(s) serveur(s) web distant(s)  
Avec les unités de chemin, vous pouvez surveiller les fichiers et les répertoires pour certains événements. Si un événement spécifique se produit, une unité de service est exécutée, et elle porte généralement le même nom que l'unité de chemin
{: .prompt-info }


Nous allons surveiller dans le dossier */srv/media/statique/* toute modification du fichier **metadata.db** qui entrainera l'exécution d'un script

Dans le répertoire systemd utilisateur nous créons une unité de cheminement **media_BiblioCalibre_site.path**

    nano ~/.config/systemd/user/media_BiblioCalibre_site.path

```ini
[Unit]
Description=Surveiller metadata.db pour les changements

[Path]
PathChanged=/srv/media/BiblioCalibre/metadata.db
Unit=media_BiblioCalibre_site.service

[Install]
WantedBy=default.target
```

Dans la section `[Path]`, `PathChanged=` indique le chemin absolu du fichier à surveiller, tandis que `Unit=` indique l'unité de service à exécuter si le fichier change. Cette unité (**media_BiblioCalibre_site.path**) doit être lancée lorsque le système est en mode multi-utilisateur.

Ensuite, nous créons l'unité de service correspondante, **media_BiblioCalibre_site.service**, dans le répertoire `~/.config/systemd/user/`    
Si le fichier **metadata.db** change (c'est-à-dire qu'il est à la fois écrit et fermé), l'unité de service suivante sera appelée pour exécuter le script spécifié :

    nano ~/.config/systemd/user/media_BiblioCalibre_site.service

```ini
[Unit] 
Description="Exécute le script si metadata.db a été modifié."

[Service]
ExecStart=/home/yann/scripts/media_BiblioCalibre_site.sh

[Install]
WantedBy=default.target
```

Le script `media_BiblioCalibre_site.sh` lance une synchronisation locale distante via rsync ssh 
<details>
<summary><b>Etendre Réduire media_BiblioCalibre_site.sh</b></summary>  

{% highlight shell %}
#!/bin/bash

#++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# Modification mode rsync suivant serveur distant
#
# Chaque modification du fichier metadata.db dans le dossier local /srv/media/BiblioCalibre 
# déclenche une synchronisation du dossier local  avec le dossier distant '/sharenfs/multimedia/eBook/BiblioCalibre' 
# des serveurs VPS Yunohost
# le dossier local est également sauvegardé dans le dossier 'backup/datayan/static' de la boîte de stockage
#++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

# Fonction pour tester si le serveur est présent
# Host=$1 et Port=$2
# Réponse $?=0 -> OK  $?=1 -> NOK
host_ok () {
 nc -4 -d -z -w 1 $1 $2 &> /dev/null
}

synchro () {
# Synchronisation locale distante du dossier _site
host_ok $SERVER $PORT 
if [[ $? == 0 ]]
then



echo 'rsync -avz --progress --stats --human-readable --delete -e "ssh -p '$PORT' -i '$PRIVKEY'" '$REPLOC' '$USERDIS':'$REPDIS'/Divers/'
rsync -avz --progress --stats --human-readable --delete --rsync-path="$RSYNCMOD" -e "ssh -p $PORT -i $PRIVKEY" $REPLOC $USERDIS:$REPDIS/Divers/ > /dev/null

	 # Analyse résultat de la commande rsync
	 if [ ! $? -eq 0 ]; then 
		 #echo "Synchro $REPLOC avec $SERVER -> OK" | systemd-cat -t BiblioCalibre -p info 
		 #echo "Synchro $REPLOC avec $SERVER -> OK"
	 #else 
		 echo "Synchro $REPLOC avec $SERVER -> ERREUR" | systemd-cat -t BiblioCalibre -p emerg 
		 #echo "Synchro $REPLOC avec $SERVER -> ERREUR"
	 fi
else
    echo "Site $SERVER port $PORT Inaccessible !" | systemd-cat -t BiblioCalibre -p emerg
    #echo "Site $SERVER port $PORT Inaccessible !"
fi

}

#*******************************************************************
#
# DEPART SCRIPT
#
#*******************************************************************

# Tester la présence du fichier des serveurs distants
if [ ! -f /home/yann/scripts/serveurs.csv ]; then
    echo "Fichier serveurs.csv inexistant!" | systemd-cat -t BiblioCalibre -p emerg
    exit 1
fi

# Mesure temps exécution
begin=$(date +"%s")
echo "***DEPART*** Exécution script $0"
echo "***DEPART*** Exécution script $0" | systemd-cat -t BiblioCalibre -p info
#echo "Exécution script $0"

# Dossier local
REPLOC="/srv/media/BiblioCalibre" 

# Synchro serveurs
while IFS="," read -r SERVER REPDIS USERDIS PORT PRIVKEY RSYNCMOD LOCAL
do
  #echo " $SERVER $REPDIS $USERDIS $PORT $PRIVKEY $RSYNCMOD $LOCAL"
 
   if [[ "$SERVER" = "rnmkcy.eu" ]]; then
  	synchro
    echo "ssh $USERDIS -p $PORT -i $PRIVKEY 'sudo systemctl restart calibreweb'"
  	ssh $USERDIS -p $PORT -i $PRIVKEY 'sudo systemctl restart calibreweb'  
   fi
done < <(tail -n +2 /home/yann/scripts/serveurs.csv)

# Calcul et affichage temps exécution
termin=$(date +"%s")
difftimelps=$(($termin-$begin))
echo "***FIN*** $0 exécuté en $(($difftimelps / 60)) mn $(($difftimelps % 60)) s" | systemd-cat -t BiblioCalibre -p info
echo "***FIN*** $0 exécuté en $(($difftimelps / 60)) mn $(($difftimelps % 60)) s"

exit 0

{% endhighlight %}

</details>

Activer et lancer

    systemctl --user enable media_BiblioCalibre_site.path --now

Voir le fichier journal

    journalctl --user -f -u media_BiblioCalibre_site.service

```
juin 06 09:39:32 yann-pc1 systemd[1537]: Started "Exécute le script si metadata.db a été modifié.".
juin 06 09:39:32 yann-pc1 media_BiblioCalibre_site.sh[11100]: ***DEPART*** Exécution script /home/yann/scripts/media_BiblioCalibre_site.sh
juin 06 09:39:32 yann-pc1 media_BiblioCalibre_site.sh[11100]: rsync -avz --progress --stats --human-readable --delete -e "ssh -p 55215 -i /home/yann/.ssh/lenovo-ed25519" /srv/media/BiblioCalibre leno@192.168.0.215:/sharenfs/multimedia/Divers/
juin 06 09:39:33 yann-pc1 media_BiblioCalibre_site.sh[11100]: ***FIN*** /home/yann/scripts/media_BiblioCalibre_site.sh exécuté en 0 mn 1 s
juin 06 09:44:40 yann-pc1 systemd[1537]: Started "Exécute le script si metadata.db a été modifié.".
juin 06 09:44:40 yann-pc1 media_BiblioCalibre_site.sh[11278]: ***DEPART*** Exécution script /home/yann/scripts/media_BiblioCalibre_site.sh
juin 06 09:44:40 yann-pc1 media_BiblioCalibre_site.sh[11278]: rsync -avz --progress --stats --human-readable --delete -e "ssh -p 55215 -i /home/yann/.ssh/lenovo-ed25519" /srv/media/BiblioCalibre leno@192.168.0.215:/sharenfs/multimedia/Divers/
juin 06 09:44:41 yann-pc1 media_BiblioCalibre_site.sh[11278]: ssh leno@192.168.0.215 -p 55215 -i /home/yann/.ssh/lenovo-ed25519 'sudo systemctl restart calibreweb'
juin 06 09:44:42 yann-pc1 media_BiblioCalibre_site.sh[11278]: ***FIN*** /home/yann/scripts/media_BiblioCalibre_site.sh exécuté en 0 mn 2 s
```

On peut créer un accès graphique sur le poste archlinux 

    ~/.local/share/applications/suivi_BiblioCalibre_site.desktop

```
[Desktop Entry]
Version=1.1
Type=Application
Name=Synchro BiblioCalibre
Comment=synchro site rnmkcy.eu
Icon=xterm-color_48x48
Exec=xterm -rv -geometry 250x30+10+50 -T suivi_BiblioCalibre_site -e 'journalctl --user -u media_BiblioCalibre_site.service --no-pager; read -p "Touche Entrée pour sortir..."'
Actions=
Categories=Utility;
Path=
Terminal=false
StartupNotify=false
```

#### Synchro serveur dossier "scripts"

Le but est de synchroniser le dossier **~/scripts** avec le serveur distant rnmkcy.eu  
{: .prompt-info }


Nous allons surveiller dans le dossier **~/scripts/** toute modification qui entrainera l'exécution d'un script

Dans le répertoire systemd utilisateur nous créons une unité de cheminement **media_BiblioCalibre_site.path**

    nano ~/.config/systemd/user/home_scripts.path

```ini
[Unit]
Description=Surveiller dossier scripts

[Path]
PathChanged=/home/yann/scripts/
Unit=home_scripts.service

[Install]
WantedBy=default.target
```

Dans la section `[Path]`, `PathChanged=` indique le chemin absolu du fichier à surveiller, tandis que `Unit=` indique l'unité de service à exécuter si le fichier change.

Ensuite, nous créons l'unité de service correspondante, **home_scripts.service**, dans le répertoire `~/.config/systemd/user/`    
Si on modifie le contenu du dossier **/home/yann/scripts/**, l'unité de service suivante sera appelée pour exécuter le script spécifié :

    nano ~/.config/systemd/user/home_scripts.service

```ini
[Unit] 
Description="Modification dossier scripts"

[Service]
ExecStart=/home/yann/scripts/home_scripts.sh
```

On supprime les lignes de fin du fichier `home_scripts.service` pour éviter un lancement de la procédure au démarrage

```
[Install]
WantedBy=default.target
```

Le script `/home/yann/scripts/home_scripts.sh` lance une synchronisation locale distante via rsync ssh et envoie un message par ntfy

```bash
#!/bin/bash

#/usr/bin/curl  -H "Title: Modification dossier scripts" \
# -H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
# -H prio:high \
# -H tags:warning,parachute -d "Le dossier /home/yann/scripts a été modifié 
#Synchronistaion du dossier :
#rsync -av --delete /home/yann/scripts/* /mnt/sharenfs/rnmkcy/scripts/" \
#https://noti.rnmkcy.eu/notif_infos

echo "rsync -av --delete /home/yann/scripts/* /mnt/sharenfs/rnmkcy/scripts/" | systemd-cat -t scripts -p info
rsync -av --delete /home/yann/scripts/* /mnt/sharenfs/rnmkcy/scripts/
```

Le rendre exécutable

    chmod +x /home/yann/scripts/home_scripts.sh

Activer et lancer

    systemctl --user enable home_scripts.path --now

Pour visualiser les messages

    journalctl --user -t scripts --since today --no-pager

On peut créer un accès graphique sur le poste archlinux 

    ~/.local/share/applications/suivi_home_scripts.desktop

```
[Desktop Entry]
Version=1.1
Type=Application
Name=Synchro dossier scripts 
Comment=synchro scripts avec rnmkcy.eu
Icon=xterm-color_48x48
Exec=xterm -rv -geometry 150x40+100+150 -T suivi_home_scripts -e 'journalctl --user -t scripts --since today --no-pager; read -p "Touche Entrée pour sortir..."'
Actions=
Categories=Utility;
Path=
Terminal=false
StartupNotify=false
```

## Développement

### Python

#### Wing personal python IDE

**Wing personal python IDE** &rarr; [Téléchargement](https://wingware.com/downloads/wing-personal) 

```
# Décompression de la version téléchargée
tar xjvf wing-personal-9.1.2.0-linux-x64.tar.bz2
# Passage en root
sudo -s
# Lancement procédure installation
cd wing-personal-9.1.2.0-linux-x64
./wing-install.py
```

Déroulement de la commande 

```
Where do you want to install the support files for Wing Personal (default
     = /usr/local/lib/wing-personal9)? 
/usr/local/lib/wing-personal9 does not exist, create it (y/N)? y
Where do you want to install links to the Wing Personal startup scripts
     (default = /usr/local/bin)? 
[...]
Writing file-list.txt
Icon/menu install returned err=0
Done installing.  Make sure that /usr/local/bin is in your path and type
     "wing-personal9" to start Wing Personal.
```

Effacer les fichiers

```
# Suppression dossier et fichier
cd ..
rm -rf wing-personal*
# sortie root
exit
```

En cas d'erreur de lancement de l'application

/usr/local/lib/wing-personal9/bin/__os__/linux-x64/runtime-python3.10/bin/python3.10: error while loading shared libraries: libcrypt.so.1: cannot open shared object file: No such file or directory
{: .prompt-danger }

Installer la librairie

    sudo pacman -S libxcrypt-compat

#### Traitement des fichiers gpx

Les fichiers à traiter sont déposés dans le dossier `~/media/osm-new/file/tmp/`  
Ils sont déplacés après traitement dans le dossier `~/media/osm-new/file/`  
Le script python `~/media/osm-new/file/tracesgpxtable.py`

Créer un environnement virtuel **env-osm**

    python -m venv ~/media/dplus/python-dev/env-osm

Si vous répertoriez maintenant les fichiers figurant dans le répertoire python-dev  , vous verrez que vous avez créé un répertoire appelé env-osm avec tous les éléments pour une exécution python en autonomie  
![](venv01.png)

<u>Ligne de commande</u>  
Pour activer l'environnement virtuel python 

    source ~/media/dplus/python-dev/env-osm/bin/activate

À ce stade, votre terminal (selon celui que vous utilisez) ajoutera probablement le nom de votre environnement au début de chaque ligne de votre terminal, dans notre cas `(env-osm)`  
Pour désactiver l'environnement virtuel python , dans le dossier `~/media/dplus/python-dev/env-osm/` saisir `deactivate` et `(env-osm)` disparaît  

Aucun paquet n'est installé dans votre environnement virtuel. C'est le comportement par défaut lorsque vous créez un environnement virtuel.

Mise à niveau pip

    pip install --upgrade pip

Installer les modules pour le traitement des fichiers gpx

    pip install gpxpy geopy

<u>Wing personal (IDE python)</u>  
Ouvrir "wing personal" créer un nouveau projet "env-osm" répertoire `~/media/dplus/python-dev/env-osm`


### Go et NodeJS

* [Installer la dernière version de Go](/posts/Debian_installer_Go+Node/#installer-la-dernière-version-de-go)
    * go version go1.21.4 linux/amd64
* [Installer Node.js](/posts/Debian_installer_Go+Node/#installer-nodejs-sur-debian-12-11-ou-10-via-nodesource)
    * Now using node v20.10.0 (npm v10.2.3)  
Creating default alias: default -> 20.10.0 (-> v20.10.0)


## Générateur site statique

*Ensemble d'applications basé sur ruby et jekyll qui permet la génération de site statique à partir de fichiers markdown*

[Archlinux Ruby + Jekyll + générateur site statique](/posts/Archlinux_Ruby_Jekyll_site_statique/)

### Site statique en local (INACTIF)

Sur l'hôte PC1, on installe nginx (`yay -S nginx`) pour visualiser le site statique en local  
Modifier le fichier de configuration `/etc/nginx/nginx.conf`

```nginx
user yann;
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    types_hash_max_size 4096;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   /home/yann/media/yannstatic/_site;
            index  index/ index.htm;
        }

        #error_page  404              /404/;

        # redirect server error pages to the static page /50x/
        #
        error_page   500 502 503 504  /50x/;
        location = /50x/ {
            root   /usr/share/nginx/;
        }

    }

}
```

Lancer et activer nginx

    sudo systemctl start nginx
    sudo systemctl enable nginx

## Wireguard - Mullvad

### Wireguard

Installation

    sudo pacman -S wireguard-tools openresolv

Dossier /etc /wireguard   

#### Configuration wg-quick 

    sudo -s
    cd /etc/wireguard

Si le dossier `~/.local/share/gtkwg/mullvad_config_linux_all/` existe, on copie une configuration

    sudo cp ~/.local/share/gtkwg/mullvad_config_linux_all/fr-par-wg-003.conf /etc/wireguard/wg0.conf

Pour démarrer un tunnel avec un fichier de configuration `/etc/wireguard/wg0.conf`  

```
[Interface]
PrivateKey = 4Mcj/qVPOOWMwVD07bAefavy1kSSEAFnnOagiZzhgFk=
Address = 10.64.218.16/32,fc00:bbbb:bbbb:bb01::1:da0f/128
DNS = 10.64.0.1

[Peer]
PublicKey = cmqtm0PSjWUa4/0DKxdr0vQqf4nFVbENQDodarHc0hY=
AllowedIPs = 0.0.0.0/0,::0/0
Endpoint = 193.32.126.70:51820[root@yann-eos yann]# 
```

Lancement en mode su

    wg-quick up wg0

```bash
    # wg-quick up <nom de l'interface>
    wg-quick up wg0 # lancement

[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -6 address add fd18:2941:ae9:7d96::3/128 dev wg0
[#] ip -4 address add 10.14.94.3/32 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] resolvconf -a wg0 -m 0 -x
[#] wg set wg0 fwmark 51820
[#] ip -6 route add ::/0 dev wg0 table 51820
[#] ip -6 rule add not fwmark 51820 table 51820
[#] ip -6 rule add table main suppress_prefixlength 0
[#] ip6tables-restore -n
[#] ip -4 route add 0.0.0.0/0 dev wg0 table 51820
[#] ip -4 rule add not fwmark 51820 table 51820
[#] ip -4 rule add table main suppress_prefixlength 0
[#] sysctl -q net.ipv4.conf.all.src_valid_mark=1
[#] iptables-restore -n

    wg-quick down wg0 # arrêt
```

#### Application Mullvad

* [Archlinux Mullvad](/posts/Mullvad-2024/#archlinux-mullvad)
* [Utilisation application VPN Mullvad](/doc/Utilisation application VPN Mullvad/)

#### GtkWg (INACTIF)

Dossier : `mkdir -p ~/.local/share/gtkwg`  
Droits `chown $USER.$USER -R /usr/local/share/gtkwg/`  
Librairie : `yay -S libappindicator-gtk3`  
Les configurations mullvad `/usr/local/share/gtkwg/mullvad_config_linux_all/`   

Le fichier **desktop** dans `~/.local/share/applications/menulibre-gtkwg.desktop`

```
[Desktop Entry]
Version=1.0
Type=Application
Name=Wireguard
Comment=IconTray/Configuration
Icon=/usr/local/share/gtkwg/wireguard_icon.png
Exec=/usr/bin/python /usr/local/share/gtkwg/GtkWgTray.py
Path=/usr/local/share/gtkwg/
NoDisplay=false
Categories=Utility;
StartupNotify=false
Terminal=false
```

Le fichier de lancement

    /usr/local/share/gtkwg/GtkWgTray.sh

```shell
!/bin/bash

#resul=`systemctl is-active wg-quick@wg0`
#if ! [[ $resul == "active" ]]; then
#  sudo systemctl enable wg-quick@wg0
#fi
#sudo systemctl start wg-quick@wg0
cd /usr/local/share/gtkwg/
python GtkWgTray.py
```

xfce: "Menu -> Paramètres -> Session et démarrage", Démarrage automatique d'applications  
![](sup-wireguard.png){:width="400"}

`Remplacer "yann"  par votre nom d'utilisateur`{: .prompt-warning }


## Virtuel QEMU KVM VMM

### Installation Virt-Manager

* [EndeavourOS Virt-Manager Complete Edition (VMM KVM QEMU)](/posts/EndeavourOS-Virt-Manager_Complete_Edition/)

Les images disques

```
[yann@yann-pc1 ~]$ tree ~/virtuel/KVM/
/home/yann/virtuel/KVM/
├── bliss-install.sh
├── bliss.sh
├── debian-12-nocloud-amd64.qcow2
├── eos-lvm-luks.qcow2
├── vm-bullseye.qcow2
├── vm-debian12.qcow2
├── wineleven.qcow2
└── winten.qcow2
```

Les configurations

```
tree -L 2 /etc/libvirt/qemu
/etc/libvirt/qemu
├── archlinux.xml
├── autostart
├── networks
│   ├── autostart
│   ├── default.xml
│   └── host-bridge.xml
├── vm-bullseye.xml
├── vm-debian12.xml
├── win10.xml
└── win11.xml
```

### Réseau

[Hôte - NetworkManager Bridge Network br0 sur autre interface](/posts/EndeavourOS-Virt-Manager_Complete_Edition/#hôte---networkmanager-bridge-network-br0-sur-autre-interface)

### Erreur de redirection USB

ATTENTION!!!  
Si vous avez installé la gestion des lecteurs NFC (<a href="#lecture-nfc-usbrfid">Lecture NFC USB/RFID</a>), il ne sera pas possible de rediriger certains périphériques USB dans les machines virtuelles  
La solution par arrêt et désactivation du socket pcscd ne peut être définitive car il est utlisée par des applications comme KeepassXC et d'autres...
{: .prompt-warning }

L'erreur suivante est affichée  
![](kvm-erreur-usb-yubico.png){:width="300"}  
et dans les logs  

```
sept. 17 07:10:03 yann-pc1 kernel: input: Yubico YubiKey OTP+FIDO+CCID as /devices/pci0000:00/0000:00:14.0/usb1/1-3/1-3:1.0/0003:1050:0407.0015/input/input36
sept. 17 07:10:03 yann-pc1 kernel: hid-generic 0003:1050:0407.0015: input,hidraw0: USB HID v1.10 Keyboard [Yubico YubiKey OTP+FIDO+CCID] on usb-0000:00:14.0-3/input0
sept. 17 07:10:03 yann-pc1 kernel: hid-generic 0003:1050:0407.0016: hiddev96,hidraw1: USB HID v1.10 Device [Yubico YubiKey OTP+FIDO+CCID] on usb-0000:00:14.0-3/input1
sept. 17 07:10:03 yann-pc1 kernel: usb 1-3: usbfs: process 2152 (pcscd) did not claim interface 2 before use
```

Solution, arrêter et désactiver pcscd

```bash
sudo systemctl stop pcscd.socket
sudo systemctl disable pcscd.socket
```

Réactivation

```bash
sudo systemctl enable pcscd.socket --now
```

## Maintenance

### Mise à jour , si erreur de paquet ou signature PGP

En cas d'erreur de paquet ou signature PGP 

    sudo pacman -S endeavouros-keyring archlinux-keyring

`Redémarrer la machine`{: .prompt-info }

### Montage partition chiffrée LUKS2

Installer les outils archlinux

    pacman -S arch-install-scripts

```
cryptsetup luksOpen /dev/nvme0n1p2 crypt

mount /dev/mapper/vg0-lvroot /mnt
mkdir -p /mnt/home
mount /dev/mapper/vg0-lvhome /mnt/home
mkdir -p /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi

```

### Récupérer la partition temporaire (OPTION)

**Ajouter la partition temporaire à la partition LUKS**

Vous pouvez simplement reformater /dev/nvme0n1p3 et l'utiliser comme stockage non chiffré, mais ici, nous allons récupérer l'espace

**Redémarrez sur un environnement Live-Cd** 

Basculer le clavier en FR

**Supprimer sda3 (installation temporaire EndeavourOS)**

Pour supprimer /dev/nvme0n1p3

    sudo fdisk /dev/sda

Eentrez simplement les caractères ci-dessous dans l'ordre indiqué.

```
> p
> d
> 3 (delete partition 3)
> w (write changes to disk)
```

**Étendre /dev/nvme0n1p2 (partition LUKS) et le groupe de volumes**

    sudo fdisk /dev/sda

```
> d
> 2 (delete partition 2)
> n
> 2 (recreate partition 2)
>   (first sector is 'default'; press enter)
>   (last sector is 'default'; press enter)
> n (keep existing filesystem signature)
> w (write changes to disk)
```

Partition LUKS

```shell
sudo cryptsetup luksOpen /dev/nvme0n1p2 crypt
sudo cryptsetup resize crypt -v

sudo e2fsck -f /dev/mapper/vg0-lvroot
sudo e2fsck -f /dev/mapper/vg0-lvhome

sudo pvresize /dev/mapper/crypt
```

Le groupe de volumes vg0 contient maintenant l'espace que nous avons libéré en supprimant /dev/nvme0n1p3. Il a été ajouté en tant qu'espace libre pouvant être utilisé pour des instantanés ou une affectation future au volume racine

Oter la clé Live  
`Redémarrer la machine`{: .prompt-info }

### Pacman Hook Liste paquets installés 

*Ce hook sauvegardera une liste de vos paquets natifs et étrangers (AUR) installés. Cela garantit que vous aurez toujours une liste à jour de tous vos paquets que vous pourrez réinstaller.*

Prérequis pour la création du hook et des scripts, créez des sous-répertoires

    sudo mkdir -p /etc/pacman.d/{hooks,hooks.bin}

Créez le hook

    sudo nano /etc/pacman.d/hooks/50-pacman-list.hook

```
#/etc/pacman.d/hooks/50-pacman-list.hook
[Trigger]
Type = Package
Operation = Install
Operation = Upgrade
Operation = Remove
Target = *

[Action]
Description = Création liste des paquets installés
When = PostTransaction
Exec = /bin/sh -c '/usr/bin/pacman -Qqe > /mnt/FreeUSB2To/PC1_eos_pkg_list.txt'
```

Pour installer des paquets depuis une sauvegarde antérieure de la liste des paquets, tout en ne réinstallant pas ceux qui sont déjà installés et à jour, lancer:

    sudo pacman -S --needed - < PC1_eos_pkg_list.txt
    sudo pacman -S --needed $(comm -12 <(pacman -Slq | sort) <(sort pkglist.txt))

### Lecture NFC USB/RFID

[Lecteur USB/RFID SCL3711](/posts/Lecteur-USB-RFID(NFC)-SCL3711/#lecteur-usbrfid-scl3711)

Installer 

    yay -S ccid libnfc acsccid pcsclite pcsc-tools

Le paquet pcsclite contient un pcscd.socket qui fera démarrer le serveur pcscd lorsqu'un programme le demandera. Vous pouvez également démarrer/activer manuellement le service pcscd.

Après avoir installé libnfc, il est important de rebrancher votre lecteur de cartes car il est livré avec quelques règles udev et une liste noire de modules du noyau qui doivent être chargées avant de charger le pilote proprement dit.

Brancher votre lecteur RFID (si ça n’est pas déjà fait).

    sudo dmesg

![](2023-06-20_18-00.png)

USB &rarr; `Bus 001 Device 016: ID 04e6:5591 SCM Microsystems, Inc. SCL3711-NFC&RW`

Après insertion, archlinux va alors charger automatiquement en arrière plan des modules qui vont perturber les NFC Tools et provoquer une erreur : `error	libnfc.driver.pn53x_usb	Unable to set USB configuration (Device or resource busy)`{: .prompt-danger }

Les modules perturbateurs

    lsmod |grep pn533

```
pn533_usb              20480  0
pn533                  45056  1 pn533_usb
nfc                   135168  1 pn533
```

Pour décharger ces modules, toujours dans votre fenêtre du terminal, entrez les commandes suivantes.

```
sudo modprobe -r pn533_usb
sudo modprobe -r pn533
```

Pour être sûr que le lecteur fonctionne correctement avec les NFC Tools, poser un tag RFID sur le lecteur et lancer la commande suivante

    nfc-list

![](2023-06-20_18-12.png)

### Sauvegardes locales

[Sauvegardes locales avec systemd utilisateur service et timer](/posts/Sauvegardes_locales_avec_systemd_utilisateur_service_et_timer/)

La sauvegarde démarre 3 minutes après la mise sous tension de PC1

Les logs : `journalctl --user -u savyann.service`  
Liste des timers : `systemctl --user list-timers --all`

### Modifier veille écran (OPTION)

[Economiseur et veille écran XFCE XScreensaver](/posts/Economiseur-et-Veille-Ecran-XFCE-xscreensaver/)

### Arborescence dossiers et fichiers

![](tree_yann.png)

![](tree_media.png) ![](tree_media_dplus.png)

### Changer M.2 2280 NVMe

Remplacer M.2 2280 NVMe 1To par une 2To  
![](ssd_Fikwot_FN501_Pro.png)

Boot sur usb live EndeavourOS  

Passer en mode su

Les partitions du disque chiffré nvme0n1

```
nvme0n1              259:0    0 931.5G  0 disk  
├─nvme0n1p1          259:1    0   512M  0 part  
├─nvme0n1p2          259:2    0   920G  0 part  
│ └─crypttemp        254:3    0   920G  0 crypt 
│   ├─vg0-lvroot     254:4    0    70G  0 lvm   
│   ├─vg0-lvhome     254:5    0   120G  0 lvm   
│   └─vg0-lvmedia    254:6    0   600G  0 lvm   
└─nvme0n1p3          259:3    0    11G  0 part  
```

Déchiffrer la partition nvme0n1p2

    cryptsetup luksOpen /dev/nvme0n1p2 crypttemp

Créer et monter le système à sauvegarder sur /media

```shell
mkdir -p /media
mkdir -p /media/home
mkdir -p /media/efi
mount /dev/vg0/lvroot /media
mount /dev/vg0/lvhome /media/home
mount /dev/nvme0n1p1 /media/efi
```

Monter le système qui va recevoir la sauvegarde

```shell
mount /dev/vg-nas-one/sav /mnt
mkdir -p /mnt/pc1
mkdir -p /mnt/pc1/efi
mkdir -p /mnt/pc1/home
```

Sauvegarder le système actuel (racine,home et efi)

    rsync -avA /media/ /mnt/pc1

Patienter plusieurs minutes, suivant la taille

Arrêter la machine PC1

Remplacer la carte SSD M2  
Redémarrer la machine sur un USB Live EndeavourOS

Zapper le nouveau disque SSD M.2    

    sgdisk --zap-all /dev/nvme0n1

Partitionnement du disque NVME 2To GPT + LVM

    gdisk /dev/nvme0n1

Créer 2 partitions  
Partition 1 : 512M EFI (code ef00) système de fichier FAT32  
Partition 2 : le reste LVM (code 8e00) système de fichier EXT4  

Formater partition EFI

    mkfs.fat -F32 /dev/nvme0n1p1

Chiffrer la partition /dev/nvme0n1p2

    cryptsetup luksFormat --type luks2 /dev/nvme0n1p2

Ouvrir la partition chiffrée

    cryptsetup luksOpen /dev/nvme0n1p2 crypt

Créer LVM

    pvcreate /dev/mapper/crypt
    vgcreate vg0 /dev/mapper/crypt

Créer les volumes

```shell
lvcreate -L 60G vg0 -n lvroot   #  Logical volume "lvroot" created.
lvcreate -L 120G vg0 -n lvhome  #  Logical volume "lvhome" created.
```

Système de fichier

```shell
mkfs.ext4 -L root /dev/mapper/vg0-lvroot
mkfs.ext4 -L home /dev/mapper/vg0-lvhome
```

Monter le nouveau système sur /mnt 

```shell
mount /dev/mapper/vg0-lvroot /mnt
mkdir -p /mnt/home
mount /dev/mapper/vg0-lvhome /mnt/home
mkdir -p /mnt/efi
mount /dev/nvme0n1p1 /mnt/efi
```

Monter la sauvegarde sur media

    mount /dev/vg-nas-one/sav /media

Restaurer le système

    rsync -avA /media/pc1/ /mnt

Patienter plusieurs minutes, suivant la taille

Création volume logique LVM media et montage

```shell
lvcreate -L 800G vg0 -n lvmedia
mkfs.ext4 -L media /dev/mapper/vg0-lvmedia

mkdir -p /mnt/srv/media
mount /dev/vg0/lvmedia /mnt/srv/media
```

Restaurer la sauvegarde multimedia

    rsync -avA /media/pc1_20240201/media/srv/media/ /mnt/srv/media

Démonter le système de sauvegarde

    umount /media

Ajouter un fichier de clé existant LUKS

    cryptsetup luksAddKey /dev/nvme0n1p2 /mnt/crypto_keyfile.bin  

Il faut saisir le phrase mot de passe

Configuration /etc/crypttab

    cryptsetup luksUUID /dev/nvme0n1p2

Renvoie  UUID  ae37e59d-35f7-4920-8428-be8be8d15243

Modifier /mnt/etc/crypttab

Contenu

```
# <name>               <device>                         <password> <options>
cryptlvm UUID=ae37e59d-35f7-4920-8428-be8be8d15243 /crypto_keyfile.bin luks
```

Passer en chroot

    arch-chroot /mnt

Relever les UUID

    blkid -s UUID -o value /dev/mapper/vg0-lvroot

renvoie l’UUID du volume racine : 2a6cab35-6c52-4382-9aee-06a376a8acc0

    blkid -s UUID -o value /dev/mapper/vg0-lvhome

renvoie l’UUID du volume d’accueil : b4e52069-a8c9-459e-b39f-6ac1b682b0d6

    blkid -s UUID -o value /dev/mapper/vg0-lvmedia

renvoie l’UUID du volume media : 1ca4bfc7-3d31-4859-aeb3-656214fab490

    blkid -s UUID -o value /dev/nvme0n1p1

renvoie l’UUID du volume media : E5E4-A4AE


Configurer /etc/fstab

    nano /etc/fstab

```
UUID=E5E4-A4AE                            /efi           vfat    defaults,noatime 0 2
tmpfs                                     /tmp           tmpfs   defaults,noatime,mode=1777 0 0
UUID=2a6cab35-6c52-4382-9aee-06a376a8acc0 / ext4 defaults,acl,noatime,discard 0 0
UUID=b4e52069-a8c9-459e-b39f-6ac1b682b0d6 /home ext4 defaults,acl,noatime,discard 0 0
/swapfile                                 none  swap defaults,pri=-2 0 0

# /dev/mapper/vg0-lvmedia
UUID=86a7c58c-8f30-42e2-bd39-d1ae7464c837 	/srv/media      ext4            rw,relatime     0 2

# /dev/mapper/ssd--512-virtuel
UUID=84bc1aa9-23ac-4530-b861-bc33171b7b42       /virtuel    ext4        defaults        0 2

# /dev/mapper/vg--nas--one-sav
UUID=c5b9eefc-1daa-4a0d-8a72-6169b3c8c91f       /sauvegardes    ext4            defaults        0 2

# /dev/vg-nas-one/iso - Volume logique 200G du disque 4To
UUID=58f4b6c7-3811-41d5-9964-f47ac32375f6           /iso        ext4            defaults        0 2
```

options du noyau

```shell
blkid -s UUID -o value /dev/nvme0n1p2 # --> ae37e59d-35f7-4920-8428-be8be8d15243
```

Modifier /etc/kernel/cmdline 

    nano /etc/kernel/cmdline

```
nvme_load=YES nowatchdog rw rd.luks.uuid=ae37e59d-35f7-4920-8428-be8be8d15243 root=/dev/mapper/vg0-lvroot
```

Réinstaller noyau

    reinstall-kernels

Sortie du chroot , retirer la clé USB Live et reboot de la machine 

### Suppression disque HDD4To

*Ce disque de très haute capacité est très peu utilisé*

Démontage des partitions concernées

```shell
sudo umount /sauvegardes
sudo umount /iso
```

Structure LVM du disque concerné

* Physical volume : /dev/sdc3  
* Volume : vg-nas-one  
* Volumes logiques : 
    * iso
    * sav

Suppression des composants LVM du disque

```shell
sudo lvremove /dev/vg-nas-one/iso
sudo lvremove /dev/vg-nas-one/sav
sudo vgremove vg-nas-one
sudo pvremove /dev/sdc3
```

Supprimer les partitions dans le fichier /etc/fstab

```
# /dev/mapper/vg--nas--one-sav
UUID=c5b9eefc-1daa-4a0d-8a72-6169b3c8c91f       /sauvegardes    ext4            defaults        0 2

# /dev/vg-nas-one/iso - Volume logique 200G du disque 4To
UUID=58f4b6c7-3811-41d5-9964-f47ac32375f6           /iso        ext4            defaults        0 2
```

Eteindre la machine, retirer le disque HDD et redémarrer.

### Ajout disque LVM

#### SSD 120Go

Disque sda

    lsblk

```
NAME                 MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                    8:0    0 111,8G  0 disk  
```

**gdisk**

    sudo gdisk /dev/sda

o : nouvelle partion dos  
n : nouvelle partition typt LVM 8e00  

Format fichier ext4 

    sudo mkfs.ext4 /dev/sda1

**LVM** (Logical Volume Manager, ou gestionnaire de volumes logiques en français) permet la création et la gestion de volumes logiques sous Linux. L'utilisation de volumes logiques remplace en quelque sorte le partitionnement des disques.

Volume physique : `sudo pvcreate /dev/sda1`  
GroupevVolumes  : `sudo vgcreate ssd-120 /dev/sda1`  
Volume logique  : `sudo lvcreate -n lv120 -l +100%FREE ssd-120`  
Fichier ext4    : `sudo mkfs.ext4 /dev/ssd-120/lv120`  

Relever UUID `sudo blkid |grep lv120`

```
/dev/mapper/ssd--120-lv120: UUID="6b48e98c-9b85-461b-9371-040765aae682" BLOCK_SIZE="4096" TYPE="ext4"
```

Création point de montage

    sudo mkdir -p /mnt/ssd

Ajouter les lignes suivantes au fichier **/etc/fstab**  

```
# /dev/mapper/ssd--120-lv120
UUID=6b48e98c-9b85-461b-9371-040765aae682 	/mnt/ssd    ext4    	defaults	0 2
```

Rechargement et montage

    sudo systemctl daemon-reload
    sudo mount -a

Vérification : `df -h /mnt/ssd/`

```
Sys. de fichiers           Taille Utilisé Dispo Uti% Monté sur
/dev/mapper/ssd--120-lv120   110G    2,1M  104G   1% /mnt/ssd
```

Droits en écriture à l'utilisateur

    sudo chown $USER:$USER /mnt/ssd/

#### HDD 4To

Disque sdc

    lsblk

```
NAME                 MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sdc                    8:32   0   3,6T  0 disk  
```

**gdisk**

    sudo gdisk /dev/sdc

o : nouvelle partion dos  
n : nouvelle partition typt LVM 8e00  

Format fichier ext4 

    sudo mkfs.ext4 /dev/sdc1

**LVM** (Logical Volume Manager, ou gestionnaire de volumes logiques en français) permet la création et la gestion de volumes logiques sous Linux. L'utilisation de volumes logiques remplace en quelque sorte le partitionnement des disques.

Volume physique : `sudo pvcreate /dev/sdc1`  
GroupevVolumes  : `sudo vgcreate hdd-4to /dev/sdc1`  
Volume logique  : `sudo lvcreate -n lv4to -l +100%FREE hdd-4to`  
Fichier ext4    : `sudo mkfs.ext4 /dev/hdd-4to/lv4to`  

Relever UUID `sudo blkid |grep lv4to`

```
/dev/mapper/hdd--4to-lv4to: UUID="26466f5b-c1c0-45db-a9f2-25c0772468ff" BLOCK_SIZE="4096" TYPE="ext4"
```

Création point de montage

    sudo mkdir -p /mnt/hdd

Ajouter les lignes suivantes au fichier **/etc/fstab**  

```
# /dev/mapper/hdd--4to-lv4to
UUID=26466f5b-c1c0-45db-a9f2-25c0772468ff 	/mnt/hdd    ext4    	defaults	0 2
```

Rechargement et montage

    sudo systemctl daemon-reload
    sudo mount -a

Vérification : `df -h /mnt/hdd/`

```
Sys. de fichiers           Taille Utilisé Dispo Uti% Monté sur
/dev/mapper/hdd--4to-lv4to   3,6T    2,1M  3,4T   1% /mnt/hdd
```

Droits en écriture à l'utilisateur

    sudo chown $USER:$USER /mnt/hdd/

### Etat des liaux

Les disques

```
Disque /dev/sda : 111,79 GiB, 120034123776 octets, 234441648 secteurs
Modèle de disque : CT120BX300SSD1  
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 4096 octets
taille d'E/S (minimale / optimale) : 4096 octets / 4096 octets
Type d'étiquette de disque : gpt
Identifiant de disque : 34D2504E-806A-47F2-AFB8-B2014C54DEFB

Périphérique Début       Fin  Secteurs Taille Type
/dev/sda1     2048 234440703 234438656 111,8G LVM Linux


Disque /dev/sdb : 476,94 GiB, 512110190592 octets, 1000215216 secteurs
Modèle de disque : Crucial_CT512MX1
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 4096 octets
taille d'E/S (minimale / optimale) : 4096 octets / 4096 octets
Type d'étiquette de disque : dos
Identifiant de disque : 0x19dd6163

Périphérique Amorçage Début        Fin   Secteurs Taille Id Type
/dev/sdb1              2048 1000214527 1000212480 476,9G 8e LVM Linux


Disque /dev/nvme0n1 : 1,86 TiB, 2048408248320 octets, 4000797360 secteurs
Modèle de disque : FIKWOT FN501 Pro 2TB                    
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets
Type d'étiquette de disque : gpt
Identifiant de disque : 381BD67E-5015-4A43-BBF5-961489275B16

Périphérique     Début        Fin   Secteurs Taille Type
/dev/nvme0n1p1    2048    1050623    1048576   512M Système EFI
/dev/nvme0n1p2 1050624 4000796671 3999746048   1,9T LVM Linux
```

Structure des volumes : `lsblk`

```
NAME                 MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                    8:0    0 111,8G  0 disk  
└─sda1                 8:1    0 111,8G  0 part  
  └─ssd--120-lv120   254:1    0 111,8G  0 lvm   /mnt/ssd
sdb                    8:16   0 476,9G  0 disk  
└─sdb1                 8:17   0 476,9G  0 part  
  └─ssd--512-virtuel 254:0    0 476,9G  0 lvm   /virtuel
nvme0n1              259:0    0   1,9T  0 disk  
├─nvme0n1p1          259:1    0   512M  0 part  /efi
└─nvme0n1p2          259:2    0   1,9T  0 part  
  └─cryptlvm         254:2    0   1,9T  0 crypt 
    ├─vg0-lvroot     254:3    0    60G  0 lvm   /
    ├─vg0-lvhome     254:4    0   120G  0 lvm   /home
    └─vg0-lvmedia    254:5    0   800G  0 lvm   /srv/media
```

Les volumes LVM

```
[root@yann-pc1 yann]# pvs
  PV                   VG      Fmt  Attr PSize    PFree  
  /dev/mapper/cryptlvm vg0     lvm2 a--     1,86t 927,21g
  /dev/sda1            ssd-120 lvm2 a--  <111,79g      0 
  /dev/sdb1            ssd-512 lvm2 a--  <476,94g      0 
[root@yann-pc1 yann]# vgs
  VG      #PV #LV #SN Attr   VSize    VFree  
  ssd-120   1   1   0 wz--n- <111,79g      0 
  ssd-512   1   1   0 wz--n- <476,94g      0 
  vg0       1   3   0 wz--n-    1,86t 927,21g
[root@yann-pc1 yann]# lvs
  LV      VG      Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv120   ssd-120 -wi-ao---- <111,79g                                                    
  virtuel ssd-512 -wi-ao---- <476,94g                                                    
  lvhome  vg0     -wi-ao----  120,00g                                                    
  lvmedia vg0     -wi-ao----  800,00g                                                    
  lvroot  vg0     -wi-ao----   60,00g                                                    
```

Les point de montage : `mount |grep -E "/dev/mapper|/dev/sd|/dev/nvme"`

```
/dev/mapper/vg0-lvroot on / type ext4 (rw,noatime)
/dev/nvme0n1p1 on /efi type vfat (rw,noatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro)
/dev/mapper/ssd--512-virtuel on /virtuel type ext4 (rw,relatime)
/dev/mapper/ssd--120-lv120 on /mnt/ssd type ext4 (rw,relatime)
/dev/mapper/vg0-lvhome on /home type ext4 (rw,noatime)
/dev/mapper/vg0-lvmedia on /srv/media type ext4 (rw,relatime)
```

fstab

```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a device; this may
# be used with UUID= as a more robust way to name devices that works even if
# disks are added and removed. See fstab(5).
#
# <file system>             <mount point>  <type>  <options>  <dump>  <pass>
UUID=E5E4-A4AE                            /efi           vfat    defaults,noatime 0 2
tmpfs                                     /tmp           tmpfs   defaults,noatime,mode=1777 0 0
UUID=2a6cab35-6c52-4382-9aee-06a376a8acc0 / ext4 defaults,acl,noatime 0 0
UUID=b4e52069-a8c9-459e-b39f-6ac1b682b0d6 /home ext4 defaults,acl,noatime 0 0
/swapfile                                 none  swap defaults,pri=-2 0 0

# /dev/mapper/vg0-lvmedia
UUID=1ca4bfc7-3d31-4859-aeb3-656214fab490 	/srv/media      ext4            rw,relatime     0 2

# /dev/mapper/ssd--512-virtuel
UUID=84bc1aa9-23ac-4530-b861-bc33171b7b42 	/virtuel    ext4    	defaults	0 2

# Les montage NFS du serveur lenovo 192.168.0.215
192.168.0.215:/ /mnt/sharenfs nfs4 nofail,x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=10s,rsize=8192,wsize=8192

# /dev/mapper/ssd--120-lv120
UUID=6b48e98c-9b85-461b-9371-040765aae682 	/mnt/ssd    ext4    	defaults	0 2
```

Les liens

```
$HOME/.borg -> /mnt/sharenfs/pc1/.borg
$HOME/Documents -> /srv/media/Documents
$HOME/FreeUSB2To -> /mnt/FreeUSB2To
$HOME/media -> /srv/media
$HOME/Musique -> /mnt/sharenfs/Musique
$HOME/sharenfs -> /mnt/sharenfs
$HOME/virtuel -> /virtuel

/files -> /home/yann/media/statique/files
/images -> /home/yann/media/statique/images
```
