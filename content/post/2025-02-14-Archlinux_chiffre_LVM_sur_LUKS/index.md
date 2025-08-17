+++
title = 'Archlinux chiffré LVM/LUKS'
date = 2025-02-14 00:00:00 +0100
categories = archlinux
+++
*Installation Archlinux LVM/LUKS avec systemd-boot*

- <https://wiki.archlinux.org/index.php/Installation_guide>
- <https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system>

**Note**: Si vous voulez une configuration de chiffrement plus simple (avec LUKS seulement), vous pouvez plutôt utiliser l'installateur [archinstall](https://wiki.archlinux.org/title/Archiinstall) "guided" inclus avec Arch depuis avril 2021.
{: .prompt-info }

## USB

Télécharger Arch Linux. Préparer un support d'installation (Un lecteur USB est utilisé comme exemple ci-dessous).

Si vous avez téléchargé Arch Linux depuis un miroir, assurez-vous de vérifier la somme de contrôle du fichier

```shell
sha1sum file_name.iso
md5sum file_name.iso
```

Ce qui précède devrait donner des sommes contrôle à comparer aux sommes de contrôle officiels Arch Linux.  

Relever le nom de votre clé USB avec `lsblk`. Assurez-vous qu'elle n'est pas montée.

Pour monter la commande Arch ISO exécuter la commande suivante, en remplaçant `/dev/sdx` par votre lecteur, par exemple `/dev/sdb`. (ne pas ajouter de numéro de partition, donc n'utilisez pas quelque chose comme `/dev/sdb1`):

```shell
dd bs=4M if=path/to/archlinux-version-x86_64.iso of=/dev/sdx conv=fsync oflag=direct status=progress
```

## Préparation

Démarrer à partir du lecteur USB (Secure Boot dit être désactivé dans le BIOS si le démarrage à partir de l'USB échoue).

Si la police courante est illisible ou trop petite 

```shell
setfont sun12x22
```

Vérifiez si vous êtes en mode UEFI

```shell
ls /sys/firmware/efi/efivars
```

Si aucune erreur et que le répertoire existe, le système est démarré dans UEFI. Sinon redémarrer en mode UEFI.

Vérifiez qu'il y a une connexion Internet 

```shell
ping archlinux.org
```

Si vous devez vous connecter via Wi-Fi, utilisez `iwctl` (l'invite interactive pour `iwd`)

```shell
$ iwctl
[iwd]# device list
[iwd]# station DEVICE_NAME scan
[iwd]# station DEVICE_NAME get-networks
[iwd]# station DEVICE_NAME connect SSID
```

Mettre à jour l'horloge du système

```shell
timedatectl set-ntp true
```

Vous pouvez modifier `/etc/pacman.d/mirrorlist` si vous souhaitez modifier la liste des miroirs (et l'ordre de priorité) utilisés lors de l'installation des paquets. Il peut être intéressant de déplacer les miroirs géogrpahiquement les plus proches vers le haut du fichier. Ce fichier sera copié sur votre système final une fois l'installation terminée.

### Partitionnement

Obtenir le nom du disque 

```shell
lsblk
```

On a une réponse de du type `/dev/sda`

Effacer le disque à l'aide de l'outil shred

```shell
shred -v -n1 /dev/sdX
```

Partitionner le disque en utilisant `gdisk` 

```shell
gdisk /dev/sda
```

* La partition 1 doit être une partition de démarrage EFI (code: ef00) de 512 Mo. 
* La partition 2 devrait être une partition Linux LVM (8e00). 

La 2ème partition peut prendre le disque complet ou seulement une partie. N'oubliez pas d'écrire les modifications de la table de partition sur le disque à la fin de la configuration.

Une fois partitionné vous pouvez formater la partition de démarrage (la partition LVM doit être chiffrée avant qu'elle ne soit formatée)

```shell
mkfs.fat -F32 /dev/sda1
```

### Chiffrement

Premièrement lancer modprobe pour `dm-crypt`

```shell
modprobe dm-crypt
```

Chiffrer le disque

```shell
cryptsetup luksFormat /dev/sda2
```

Ouvrez le disque avec le mot de passe défini ci-dessus

```shell
cryptsetup open --type luks /dev/sda2 cryptlvm
```

Vérifiez que le disque lvm existe 

```shell
ls /dev/mapper/cryptlvm
```

Créer un volume physique

```shell
pvcreate /dev/mapper/cryptlvm
```

Créer un groupe de volume

```shell
vgcreate volume /dev/mapper/cryptlvm
```

Créer des partitions logiques

```shell
lvcreate -L20G volume -n swap
lvcreate -L40G volume -n root
lvcreate -l 100%FREE volume -n home
```

Formater le système de fichiers sur les partitions logiques

```shell
mkfs.ext4 /dev/volume/root
mkfs.ext4 /dev/volume/home
mkswap /dev/volume/swap
```

Monter les volumes et les systèmes de fichiers

```shell
mount /dev/volume/root /mnt
mkdir /mnt/home
mkdir /mnt/boot
mount /dev/volume/home /mnt/home
mount /dev/sda1 /mnt/boot
swapon /dev/volume/swap
```

## Installation

Installer le paquet de base, linux, firmware, lvm2 et utilitaires

```shell
pacstrap /mnt base base-devel linux linux-firmware lvm2 nano
```

Générer `fstab` 

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

`chroot` dans le système

```shell
arch-chroot /mnt
```

Définir l'heure locale (choisir une localité pertinente)

```shell
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
```

Réglez l'horloge

```shell
hwclock --systohc
```

Décommenter `fr_US.UTF-8 UTF-8` ou toutes les localisations dont vous avez besoin dans `/etc/locale.gen`  
Puis exécuter

```shell
locale-gen
```

Créer un fichier de configuration "locale"

```shell
locale > /etc/locale.conf
```

Définir la variable lang dans le fichier ci-dessus (Choisissez le code de langue qui vous intéresse)

```shell
LANG=fr_FR.UTF-8
```

Ajouter un nom d'hôte (tout nom d'hôte de votre choix comme une ligne dans le fichier. Par exemple, «monarch»

```shell
nano /etc/hostname
```

Mettre à jour le contenu de `/etc/hosts` (remplacer `myhostname` avec le nom d'hôte que vous avez utilisé ci-dessus)

```text
127.0.1.1   myhostname.localdomain  myhostname
```

Parce que notre système de fichiers est sur LVM, nous devrons activer les corrects  "hooks" **mkinitcpio**.  
Modifier le `/etc/mkinitcpio.conf`. Recherchez la variable HOOKS et mettez-la à jour pour ressembler à ce qui suit

```text
HOOKS=(base udev autodetect keyboard keymap modconf block encrypt lvm2 filesystems fsck)
```

Régénérer initramfs 

```shell
mkinitcpio -p linux
```

Installer un chargeur de démarrage (bootloader)

```shell
bootctl --path=/boot/ install
```

Créer un chargeur de démarrage. Modifier `/boot/loader/loader.conf`.  
Remplacer le contenu du fichier par :

```text
default arch
timeout 3
editor 0
```

`edito 0` garantit que la configuration ne peut pas être modifiée au démarrage.

Créer une entrée bootloader dans `/boot/loader/entries/arch.conf`

```text
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options cryptdevice=UUID={UUID}:cryptlvm root=/dev/volume/root quiet rw
```

Remplacer `{UUID}` par `/dev/sda2`.  
Afin d'obtenir l'UUID exécuter la commande suivante 

```shell
blkid
```

## Complément

Avant de terminer les étapes d'installation, vous pouvez installer des paquets supplémentaires pour la gestion utilisateur et réseau (ceux-ci sont inclus dans l'installateur mais ne sont normalement pas inclus dans l'installation elle-même):

```shell
sudo pacman -Syu sudo iw iwd dhcpcd
```

Mot de passe root

```shell
passwd
```

Sortie `chroot`:

```shell
exit
```

Tout démonter

```shell
umount -R /mnt
```

Redémarrage

```shell
reboot
```