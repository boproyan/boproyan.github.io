+++
title = 'MULTIBOOT USB 32Go (EFI+GPT et BIOS+GPT/MBR)'
date = 2023-01-19 00:00:00 +0100
categories = ['outils']
+++
*Un lecteur USB multiboot permettant de démarrer plusieurs fichiers ISO Archlinux, Debian, Manjaro, PartedMagic, Tails, etc...*

- [Création USB multiboot par script](#création-usb-multiboot-par-script)
    - [Le script](#le-script)
    - [Utilisation du script](#utilisation-du-script)
- [Création USB multiboot "manuelle"](#création-usb-multiboot-manuelle)
    - [Clé USB universelle bootable MBR et UEFI](#clé-usb-universelle-bootable-mbr-et-uefi)
    - [Systèmes de fichier ,montage ,grub](#systèmes-de-fichier-montage-grub)
- [Ajout des distributions sur le lecteur USB](#ajout-des-distributions-sur-le-lecteur-usb)
    - [Manjaro](#manjaro)
    - [Kali](#kali)
    - [Partition magic](#partition-magic)
    - [Debian](#debian)
- [Démarrez un ISO avec MEMDISK (facultatif)](#démarrez-un-iso-avec-memdisk-facultatif)
- [Test périphérique USB avec QEMU](#test-périphérique-usb-avec-qemu)
- [Annexe](#annexe)
    - [Obtenir des fichiers amorçables ISO](#obtenir-des-fichiers-amorçables-iso)
    - [Liens](#liens)


## Création USB multiboot par script

### Le script

*Copie du script git https://github.com/aguslr/multibootusb*

Script shell **makeUSB.sh**  

<details>
<summary>(Afficher/Cacher) <b>makeUSB.sh</b></summary>

{% highlight shell %}  
#!/bin/sh

# Description : Script pour préparer une clé USB multiboot

# Afficher le numéro de ligne lors de l'exécution par bash -x makeUSB.sh
[ "$BASH" ] && \
    export PS4='    +\t $BASH_SOURCE:$LINENO: ${FUNCNAME[0]:+${FUNCNAME[0]}():}'

# Exit if there is an unbound variable or an error
set -o nounset
set -o errexit

# Valeurs par défaut
scriptname=$(basename "$0")
hybrid=0
clone=0
eficonfig=0
interactive=0
data_part=2
data_fmt="vfat"
data_size=""
efi_mnt=""
data_mnt=""
data_subdir="boot"
repo_dir=""
tmp_dir="${TMPDIR-/tmp}"

# Afficher l'utilisation
# 	  -c, --clone                   Clone le dépôt Git sur le périphérique
showUsage() {
	cat <<- EOF
	Script to prepare multiboot USB drive
	Usage: $scriptname [options] device [fs-type] [data-size]

	 device                         Dispositif à modifier (par exemple /dev/sdb)
	 fs-type                        Type de système de fichiers pour la partition de données [ext3|ext4|vfat|ntfs].
	 data-size                      Taille de la partition de données (par exemple, 5G)
	  -b, --hybrid                  Créer un MBR hybride
	  -e, --efi                     Activer la compatibilité EFI
	  -i, --interactive             Lance gdisk pour créer un MBR hybride
	  -h, --help                    Affiche ce message
	  -s, --subdirectory <NAME>     Spécifier un sous-répertoire de données (par défaut : "boot")
	  Exemple : $scriptname -b -e /dev/sde
	EOF
}

# Nettoyer en quittant
cleanUp() {
	# Changement de propriétaire des fichiers
	{ [ "$data_mnt" ] && \
	    chown -R "$normal_user" "${data_mnt}"/* 2>/dev/null; } \
	    || true
	# Démontez tout
	umount -f "$efi_mnt" 2>/dev/null || true
	umount -f "$data_mnt" 2>/dev/null || true
	# Suppression des points de montage
	[ -d "$efi_mnt" ] && rmdir "$efi_mnt"
	[ -d "$data_mnt" ] && rmdir "$data_mnt"
	[ -d "$repo_dir" ] && rmdir "$repo_dir"
	# Exit
	exit "${1-0}"
}

# # Vérifiez que le lecteur USB n'est pas monté
unmountUSB() {
	umount -f "${1}"* 2>/dev/null || true
}

# Trappe les signaux kill (SIGHUP, SIGINT, SIGTERM) pour effectuer un nettoyage et quitter la machine
trap 'cleanUp' 1 2 15

# Afficher l'aide avant de vérifier la présence de root
[ "$#" -eq 0 ] && showUsage && exit 0
case "$1" in
	-h|--help)
		showUsage
		exit 0
		;;
esac

# Vérifier la présence de root
if [ "$(id -u)" -ne 0 ]; then
	printf 'Ce script doit être exécuté en tant que root. Utilisation de sudo...\n' "$scriptname" >&2
	exec sudo -k -- /bin/sh "$0" "$@" || cleanUp 2
fi

# Récupérer l'utilisateur original
normal_user="${SUDO_USER-$(who -m | awk '{print $1}')}"

# Vérifier les arguments
while [ "$#" -gt 0 ]; do
	case "$1" in
		-b|--hybrid)
			hybrid=1
			;;
		-c|--clone)
			clone=1
			;;
		-e|--efi)
			eficonfig=1
			data_part=3
			;;
		-i|--interactive)
			interactive=1
			;;
		-s|--subdirectory)
			shift && data_subdir="$1"
			;;
		/dev/*)
			if [ -b "$1" ]; then
				usb_dev="$1"
			else
				printf '%s: %s périphérique non valide.\n' "$scriptname" "$1" >&2
				cleanUp 1
			fi
			;;
		[a-z]*)
			data_fmt="$1"
			;;
		[0-9]*)
			data_size="$1"
			;;
		*)
			printf '%s: %s argument non valide.\n' "$scriptname" "$1" >&2
			cleanUp 1
			;;
	esac
	shift
done

# Vérifier les arguments requis
if [ ! "$usb_dev" ]; then
	printf '%s: Aucun périphérique fourni.\n' "$scriptname" >&2
	showUsage
	cleanUp 1
fi

# Vérifier le binaire d'installation de GRUB
grub_cmd=$(command -v grub2-install) \
    || grub_cmd=$(command -v grub-install) \
    || cleanUp 3

# Démonter le périphérique
unmountUSB "$usb_dev"

# Confirmez le périphérique
printf 'Etes-vous sûr de vouloir utiliser %s? [y/N] ' "$usb_dev"
read -r answer1
case "$answer1" in
	[yY][eE][sS]|[yY])
		printf 'CELA SUPPRIMERA TOUTES LES DONNÉES. Vous êtes sûr ? [y/N] '
		read -r answer2
		case $answer2 in
			[yY][eE][sS]|[yY])
				true
				;;
			*)
				cleanUp 3
				;;
		esac
		;;
	*)
		cleanUp 3
		;;
esac

# Imprimer toutes les étapes
set -o verbose

# Suppression des partitions
sgdisk --zap-all "$usb_dev"

# Créer une table de partition GUID
sgdisk --mbrtogpt "$usb_dev" || cleanUp 10

# Créer une partition de démarrage BIOS (1M)
sgdisk --new 1::+1M --typecode 1:ef02 \
    --change-name 1:"BIOS boot partition" "$usb_dev" || cleanUp 10

# Créer une partition système EFI (50M)
[ "$eficonfig" -eq 1 ] && \
    { sgdisk --new 2::+50M --typecode 2:ef00 \
    --change-name 2:"EFI System" "$usb_dev" || cleanUp 10; }

# Définir la taille de la partition de données
[ -z "$data_size" ] || \
    data_size="+$data_size"

# Définit les informations de la partition de données
case "$data_fmt" in
	ext2|ext3|ext4)
		type_code="8300"
		part_name="Linux filesystem"
		;;
	msdos|fat|vfat|ntfs)
		type_code="0700"
		part_name="Microsoft basic data"
		;;
	*)
		printf '%s: %s est un type de système de fichiers invalide.\n' "$scriptname" "$data_fmt" >&2
		showUsage
		cleanUp 1
		;;
esac

# Créer une partition de données
sgdisk --new ${data_part}::"${data_size}": --typecode ${data_part}:"$type_code" \
    --change-name ${data_part}:"$part_name" "$usb_dev" || cleanUp 10

# Démonter le périphérique
unmountUSB "$usb_dev"

# Configuration interactive ?
if [ "$interactive" -eq 1 ]; then
	# Créer manuellement un MBR hybride
	# https://bit.ly/2z7HBrP
	gdisk "$usb_dev"
elif [ "$hybrid" -eq 1 ]; then
	# Créer un MBR hybride
	if [ "$eficonfig" -eq 1 ]; then
		sgdisk --hybrid 1:2:3 "$usb_dev" || cleanUp 10
	else
		sgdisk --hybrid 1:2 "$usb_dev" || cleanUp 10
	fi
fi

# Définir l'indicateur de démarrage pour la partition de données
sgdisk --attributes ${data_part}:set:2 "$usb_dev" || cleanUp 10

# Démonter le périphérique
unmountUSB "$usb_dev"

# Effacer la partition de démarrage du BIOS
wipefs -af "${usb_dev}1" || true

# Formatage de la partition système EFI
if [ "$eficonfig" -eq 1 ]; then
	wipefs -af "${usb_dev}2" || true
	mkfs.vfat -v -F 32 "${usb_dev}2" || cleanUp 10
fi

# Effacer la partition de données
wipefs -af "${usb_dev}${data_part}" || true

# Formatage de la partition de données
if [ "$data_fmt" = "ntfs" ]; then
	# Utiliser le format rapide mkntfs
	mkfs -t "$data_fmt" -f "${usb_dev}${data_part}" || cleanUp 10
else
	mkfs -t "$data_fmt" "${usb_dev}${data_part}"    || cleanUp 10
fi

# Démonter le périphérique
unmountUSB "$usb_dev"

# Créer des répertoires temporaires
efi_mnt=$(mktemp -p "$tmp_dir" -d efi.XXXX)   || cleanUp 10
data_mnt=$(mktemp -p "$tmp_dir" -d data.XXXX) || cleanUp 10
repo_dir=$(mktemp -p "$tmp_dir" -d repo.XXXX) || cleanUp 10

# Monter la partition système EFI
[ "$eficonfig" -eq 1 ] && \
    { mount "${usb_dev}2" "$efi_mnt" || cleanUp 10; }

# Monter la partition de données
mount "${usb_dev}${data_part}" "$data_mnt" || cleanUp 10

# Installer GRUB pour EFI
[ "$eficonfig" -eq 1 ] && \
    { $grub_cmd --target=x86_64-efi --efi-directory="$efi_mnt" \
    --boot-directory="${data_mnt}/${data_subdir}" --removable --recheck \
    || cleanUp 10; }

# Installer GRUB pour BIOS
$grub_cmd --force --target=i386-pc \
    --boot-directory="${data_mnt}/${data_subdir}" --recheck "$usb_dev" \
    || cleanUp 10

# Installer GRUB de secours
$grub_cmd --force --target=i386-pc \
    --boot-directory="${data_mnt}/${data_subdir}" --recheck "${usb_dev}${data_part}" \
    || true

# Créer les répertoires nécessaires
mkdir -p "${data_mnt}/${data_subdir}/isos" || cleanUp 10

#if [ "$clone" -eq 1 ]; then
	# Cloner le dépôt Git
#	(cd "$repo_dir" && \
#		git clone https://github.com/aguslr/multibootusb . && \
#		# Déplacer tous les fichiers et dossiers visibles et cachés, sauf '.' et '...'.
#		for x in * .[!.]* ..?*; do if [ -e "$x" ]; then mv -- "$x" \
#			"${data_mnt}/${data_subdir}"/grub*/; fi; done) \
#	    || cleanUp 10
#else
	# Copier les fichiers
	cp -R ./boot/grub/mbusb.* "${data_mnt}/${data_subdir}"/grub*/ \
	    || cleanUp 10
	# Copier la configuration GRUB
	cp ./boot/grub/grub.cfg "${data_mnt}/${data_subdir}"/grub*/ \
	    || cleanUp 10
	# Copier les thèmes GRUB
	cp -R ./boot/grub/themes/* "${data_mnt}/${data_subdir}"/grub/themes/ \
	    || cleanUp 10
#fi

# Renommer la configuration d'exemple
#( cd "${data_mnt}/${data_subdir}"/grub*/ && cp grub.cfg.example grub.cfg ) \
#    || cleanUp 10

# Télécharger memdisk
syslinux_url='https://www.kernel.org/pub/linux/utils/boot/syslinux/syslinux-6.03.tar.gz'
{ wget -qO - "$syslinux_url" 2>/dev/null || curl -sL "$syslinux_url" 2>/dev/null; } \
    | tar -xz -C "${data_mnt}/${data_subdir}"/grub*/ --no-same-owner --strip-components 3 \
    'syslinux-6.03/bios/memdisk/memdisk' \
    || cleanUp 10

# Nettoyer et quitter
cleanUp
exit 0
{% endhighlight %}

</details>

Le rendre exécutable


    chmo +x makeUSB.sh

### Utilisation du script

Il suffit de l'exécuter en tant que root:

```sh
./makeUSB.sh -b -e <device>
```

**-b** pour créer un hybride MBR et **-e** pour la compatibilité UEFI  
Où `<device>` est le nom du périphérique USB (par exemple */dev/sd*).   
Exécutez `mount` pour obtenir cette information.
Autre possibilité , juste après avoir inséré la clé USB , exécuter `dmesg` dans une fenêtre terminal :

    [ 2541.013465] sd 4:0:0:0: [sde] Attached SCSI removable disk

Dans notre exemple la clé est sous **/dev/sdd**

** AVERTISSEMENT **: Cela efface toutes les données du périphérique, alors assurez-vous de choisir le bon.


## Création USB multiboot "manuelle"

### Clé USB universelle bootable MBR et UEFI

[Hybrid UEFI GPT + BIOS GPT/MBR boot(archlinux wiki)](https://wiki.archlinux.org/index.php/Multiboot_USB_drive#Hybrid_UEFI_GPT_.2B_BIOS_GPT.2FMBR_boot)

Cette configuration est utile pour créer une clé USB universelle, bootable partout. Tout d'abord, vous devez créer une table de partitions GPT sur votre périphérique. Vous avez besoin d'au moins 3 partitions:

* Une partition d'amorçage BIOS (type EF02) qui doit doit avoir une taille de 1 Mo
* Une partition système EFI (type EF00 avec un système de fichiers FAT32) au minimum 50 Mo
* Votre partition de données (utilisez un système de fichiers pris en charge par GRUB) peut prendre le reste de l'espace de votre lecteur

Ensuite, vous devez créer une table de partition hybride MBR, car définir l'indicateur de démarrage sur la partition de protection MBR peut ne pas être suffisant.

Mode su

    sudo -s

Identifier la clé USB par dmesg (exemple /dev/sdd

    dmesg

```
    [25430.573773] sd 4:0:0:0: [sdd] Attached SCSI removable disk
```

La clé est sur /dev/sdd  

On va initialiser la clé USB

    gdisk /dev/sdd

```
GPT fdisk (gdisk) version 1.0.4

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): x

Expert command (? for help): z
About to wipe out GPT on /dev/sdd. Proceed? (Y/N): Y
GPT data structures destroyed! You may now partition the disk using fdisk or
other utilities.
Blank out MBR? (Y/N): Y
```

Lancer gdisk pour préparer la clé USB

    gdisk /dev/sdd

```
GPT fdisk (gdisk) version 1.0.1

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): Y

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-30282974, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-30282974, default = 30282974) or {+-}size{KMGTP}: +1M
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): EF02
Changed type of partition to 'BIOS boot partition'

Command (? for help): n
Partition number (2-128, default 2): 
First sector (34-30282974, default = 4096) or {+-}size{KMGTP}: 
Last sector (4096-30282974, default = 30282974) or {+-}size{KMGTP}: +50M
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): EF00
Changed type of partition to 'EFI System'

Command (? for help): n
Partition number (3-128, default 3): 
First sector (34-30282974, default = 106496) or {+-}size{KMGTP}: 
Last sector (106496-30282974, default = 30282974) or {+-}size{KMGTP}: 
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 0700
Changed type of partition to 'Microsoft basic data'

Command (? for help): r

Recovery/transformation command (? for help): h     # h --> make hybrid MBR

WARNING! Hybrid MBRs are flaky and dangerous! If you decide not to use one,
just hit the Enter key at the below prompt and your MBR partition table will
be untouched.

Type from one to three GPT partition numbers, separated by spaces, to be
added to the hybrid MBR, in sequence: 1 2 3
Place EFI GPT (0xEE) partition first in MBR (good for GRUB)? (Y/N): N

Creating entry for GPT partition #1 (MBR partition #1)
Enter an MBR hex code (default EF):  
Set the bootable flag? (Y/N): N

Creating entry for GPT partition #2 (MBR partition #2)
Enter an MBR hex code (default EF): 
Set the bootable flag? (Y/N): N

Creating entry for GPT partition #3 (MBR partition #3)
Enter an MBR hex code (default 07): 
Set the bootable flag? (Y/N): Y

Recovery/transformation command (? for help): x  # x --> extra functionality (experts only)

Expert command (? for help): h                   # h --> recompute CHS values in protective/hybrid MBR

Expert command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sdd.
The operation has completed successfully.

```

Structure de la clé USB

```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdd           8:48   1  29,3G  0 disk 
├─sdd1        8:49   1     1M  0 part 
├─sdd2        8:50   1    50M  0 part 
└─sdd3        8:51   1  29,2G  0 part 
```

### Systèmes de fichier ,montage ,grub

**Systèmes de fichier** 

```
pacman -S dosfstools            # pour les systèmes de fichier vfat
mkfs.vfat -F 32 /dev/sdd2       # FAT32 pour EFI
mkfs.vfat -F 32 /dev/sdd3       # FAT32 partition principale
```

Tout d'abord, vous devez monter la partition EFI System (Ex: /mnt/multiboot/efi) et la partition de données (Ex: /mnt/multiboot/data) de votre clé USB (EX: /dev/sdd).  

**montage** 

```shell
mkdir -p /mnt/multiboot/efi
mount /dev/sdd3 /mnt/multiboot
mount /dev/sdd2 /mnt/multiboot/efi
```

**grub**  
Vous pouvez maintenant installer GRUB pour prendre en charge EFI + GPT et BIOS + GPT/MBR. La configuration GRUB (--boot-directory) peut être conservée au même endroit.   

Installation GRUB pour EFI

    grub-install --target=x86_64-efi --efi-directory=/mnt/multiboot/efi --boot-directory=/mnt/multiboot/boot --removable --recheck /dev/sdd

Installation GRUB pour BIOS

    grub-install --target=i386-pc --boot-directory=/mnt/multiboot/boot --recheck /dev/sdd

La partition de données est montée sur **/mnt/multiboot** et le dossier multiboot **iso/multibootusb/**   
Copiez les fichiers GRUB nécessaires:

```
# répertoire nommé *isos* pour les fichiers ISO
mkdir -p /mnt/multiboot/boot/isos
#mkdir -p /mnt/multiboot/boot/grub/grub.d
# Copie des fichiers GRUB
cp -rf iso/multibootusb/boot/grub/mbusb.d /mnt/multiboot/boot/grub/
cp iso/multibootusb/boot/grub/grub.cfg /mnt/multiboot/boot/grub/
# Le fichier commun à toutes les distributions
cp iso/multibootusb/boot/grub/mbusb.cfg /mnt/multiboot/boot/grub/
# les thèmes
cp -rf iso/multibootusb/boot/grub/themes /mnt/multiboot/boot/grub/
# splash.png
cp iso/multibootusb/boot/grub/splash.png /mnt/multiboot/boot/grub/

```

Le fichier **/mnt/multiboot/boot/grub/grub.cfg**

```
# Config for GNU GRand Unified Bootloader (GRUB)

insmod font
if loadfont unicode ; then
  if keystatus --shift ; then true ; else
    if [ x"${grub_platform}" = xefi ]; then
      insmod efi_gop
      insmod efi_uga
      insmod videotest
      insmod videoinfo
    else
      insmod vbe
      insmod vga
    fi
    insmod gfxterm
    insmod gfxmenu
    set gfxmode=auto
    set gfxpayload=auto
    terminal_output gfxterm
    if terminal_output gfxterm ; then true ; else
      terminal gfxterm
    fi
  fi
fi

# Timeout for menu
# set timeout=30

insmod png
set theme=${prefix}/themes/Virtualfuture/theme.txt

# Load MBUSB configuration
# Partition holding files
probe -u $root --set=rootuuid
set imgdevpath="/dev/disk/by-uuid/$rootuuid"
export imgdevpath rootuuid

# Custom variables
set isopath="/boot/isos"
export isopath
export theme

# Load modules
insmod regexp
insmod all_video

# MultiBoot USB menu
  # Warning for 32-bit systems
  if ! cpuid -l; then
    clear
    echo "This is a 32-bit computer."
    echo "You won't get to use 64-bit software on it."
    echo
    echo -n "To continue, press ESC or wait 10 seconds... "
    sleep --interruptible 10
    echo
    echo
  fi

  # Load custom configuration
  if [ -e "$prefix/mbusb.cfg.local" ]; then
    source "$prefix/mbusb.cfg.local"
  fi

  # Load configuration files
  echo -n "Loading configuration files... "
  for cfgfile in $prefix/mbusb.d/*.d/*.cfg; do
    source "$cfgfile"
  done

# Grub options
submenu "GRUB2 options ->" {
  menuentry "List devices/partitions" {
    ls -l
    sleep --interruptible 9999
  }

  menuentry "Enable GRUB2's LVM support" {
    insmod lvm
  }

  menuentry "Enable GRUB2's RAID support" {
    insmod dm_nv
    insmod mdraid09_be
    insmod mdraid09
    insmod mdraid1x
    insmod raid5rec
    insmod raid6rec
  }

  menuentry "Enable GRUB2's PATA support (to work around BIOS bugs/limitations)" {
    insmod ata
    update_paths
  }

  menuentry "Enable GRUB2's USB support *experimental*" {
    insmod ohci
    insmod uhci
    insmod usbms
    update_paths
  }

  menuentry "Mount encrypted volumes (LUKS and geli)" {
    insmod luks
    insmod geli
    cryptomount -a
  }

  menuentry "Enable serial terminal" {
    serial
    terminal_input --append serial
    terminal_output --append serial
  }
}

# Reboot
menuentry "Reboot" {
  reboot
}
# Poweroff
menuentry "Poweroff" {
  halt
}
```

## Ajout des distributions sur le lecteur USB

Le fichier commun tous les OS **mbusb.cfg**

	/mnt/multiboot/boot/grub/mbusb.cfg

```
# Partition holding files
probe -u $root --set=rootuuid
set imgdevpath="/dev/disk/by-uuid/$rootuuid"
export imgdevpath rootuuid

# Custom variables
set isopath="/boot/isos"
export isopath

# Load modules
insmod regexp
insmod all_video

# MultiBoot USB menu
submenu "Multiboot ->" {
  # Warning for 32-bit systems
  if ! cpuid -l; then
    clear
    echo "This is a 32-bit computer."
    echo "You won't get to use 64-bit software on it."
    echo
    echo -n "To continue, press ESC or wait 10 seconds... "
    sleep --interruptible 10
    echo
    echo
  fi

  # Load custom configuration
  if [ -e "$prefix/mbusb.cfg.local" ]; then
    source "$prefix/mbusb.cfg.local"
  fi

  # Load configuration files
  echo -n "Loading configuration files... "
  for cfgfile in $prefix/mbusb.d/*.d/*.cfg; do
    source "$cfgfile"
  done
}
```

### Manjaro

Créer le dossier 

    mkdir -p /mnt/multiboot/boot/grub/mbusb.d/manjaro.d/

Créer le fichier de configuration

    nano /mnt/multiboot/boot/grub/mbusb.d/manjaro.d/generic.cfg

```
for isofile in $isopath/manjaro-*.iso; do
  if [ -e "$isofile" ]; then
    regexp --set=isoname "$isopath/(.*)" "$isofile"
    submenu "$isoname ->" "$isofile" {
      iso_path="$2"
      loopback loop "$iso_path"
      probe --label --set=cd_label (loop)
      menuentry "Start Manjaro Linux" {
        bootoptions="img_dev=$imgdevpath img_loop=$iso_path earlymodules=loop misobasedir=manjaro misolabel=$cd_label nouveau.modeset=1 i915.modeset=1 radeon.modeset=1 logo.nologo overlay=free lang=fr_FR keytable=fr quiet splash showopts"
        linux (loop)/boot/vmlinuz-* $bootoptions
        initrd (loop)/boot/initramfs-*.img
      }
      menuentry "Start (non-free drivers)" {
        bootoptions="img_dev=$imgdevpath img_loop=$iso_path earlymodules=loop misobasedir=manjaro misolabel=$cd_label nouveau.modeset=0 i915.modeset=1 radeon.modeset=0 nonfree=yes logo.nologo overlay=nonfree lang=fr_FR keytable=fr quiet splash showopts"
        linux (loop)/boot/vmlinuz-* $bootoptions
        initrd (loop)/boot/initramfs-*.img
      }
    }
  fi
done
```

Copier le fichier iso manjaro dans le dossier `/mnt/multiboot/boot/isos/`

### Kali

Créer le dossier 

    mkdir -p /mnt/multiboot/boot/grub/mbusb.d/kali.d/

Créer le fichier de configuration

    nano /mnt/multiboot/boot/grub/mbusb.d/kali.d/generic.cfg

```
for isofile in $isopath/kali-linux-*.iso; do
  if [ -e "$isofile" ]; then
    regexp --set=isoname "$isopath/(.*)" "$isofile"
    submenu "$isoname ->" "$isofile" {
      iso_path="$2"
      loopback loop "$iso_path"
# Live boot
menuentry "Live system (amd64)" --hotkey=l {
	linux	(loop)/live/vmlinuz-5.18.0-kali5-amd64 boot=live components quiet splash noeject findiso=${iso_path}
	initrd	(loop)/live/initrd.img-5.18.0-kali5-amd64
}
if [ "${grub_platform}" = "efi" ]; then
menuentry "Live system (amd64 fail-safe mode)" {
	linux	(loop)/live/vmlinuz-5.18.0-kali5-amd64 boot=live components noeject memtest noapic noapm nodma nomce nolapic nosmp nosplash vga=normal
	initrd	(loop)/live/initrd.img-5.18.0-kali5-amd64
}
else
menuentry "Live system (amd64 fail-safe mode)" {
	linux	(loop)/live/vmlinuz-5.18.0-kali5-amd64 boot=live components noeject memtest noapic noapm nodma nomce nolapic nomodeset nosmp nosplash vga=normal
	initrd	(loop)/live/initrd.img-5.18.0-kali5-amd64
}
fi

menuentry "Live system (amd64 forensic mode)" {
	linux (loop)/live/vmlinuz-5.18.0-kali5-amd64 boot=live components quiet splash noeject findiso=${iso_path} noswap noautomount
	initrd (loop)/live/initrd.img-5.18.0-kali5-amd64
}
menuentry "Live system with USB persistence  (check kali.org/prst)" {
	linux (loop)/live/vmlinuz-5.18.0-kali5-amd64 boot=live components quiet splash noeject findiso=${iso_path} persistence
	initrd (loop)/live/initrd.img-5.18.0-kali5-amd64
}
menuentry "Live system with USB Encrypted persistence" {
	linux (loop)/live/vmlinuz-5.18.0-kali5-amd64 boot=live components quiet splash noeject findiso=${iso_path} persistent=cryptsetup persistence-encryption=luks persistence
	initrd (loop)/live/initrd.img-5.18.0-kali5-amd64
}

    }
  fi
done
```

Modifier la configuration pour prise en charge langue FR par défaut

    sed -i 's/hostname=kali/hostname=kali lang=fr_FR.UTF-8 locales=fr_FR.UTF-8 keyboard-layouts=fr keyboard-model=pc105/g' /mnt/multiboot/boot/grub/mbusb.d/kali.d/generic.cfg

Log et mot de passe : kali

### Partition magic

Montage fichier iso sur tmp

    mkdir -p tmp
    sudo mount -o loop -t iso9660 pmagic_2019_12_24.iso tmp/

Copie du répertoire **pmagic** sur la racine de la clé USB

    sudo cp -r tmp/pmagic/ /mnt/multiboot/ && sync
    sudo umount tmp

Remplacer le fichier **/mnt/multiboot/boot/grub/mbusb.d/pmagic.d/generic.cfg**

```
submenu "Pmagic -->" {
 set default_settings="edd=on keymap=fr-latin1 fr_FR"
 set live_settings="boot=live eject=no"
 set linux_64="/pmagic/bzImage64"
 set linux_32="/pmagic/bzImage"
 set initrd_img="/pmagic/initrd.img /pmagic/fu.img /pmagic/m64.img"
 set initrd_img32="/pmagic/initrd.img /pmagic/fu.img /pmagic/m32.img"
 set message="Chargement kernel et initramfs. Patienter SVP..."

 menuentry "Pmagic - Default settings 64 (Runs from RAM)"{
	echo $message
	search --set -f $linux_64
	linux $linux_64 $default_settings
	initrd $initrd_img
 }

 menuentry "Pmagic - Default settings 32 (Runs from RAM)"{
	search --set -f $linux_32
	linux $linux_32 $default_settings
	initrd $initrd_img32
 }

 menuentry "Pmagic - Live with default settings 64"{
	echo $message
	search --set -f $linux_64
	linux $linux_64 $default_settings $live_settings
	initrd $initrd_img
 }
 menuentry "Pmagic - Live with default settings 32"{
	search --set -f $linux_32
	linux $linux_32 $default_settings $live_settings
	initrd $initrd_img32
 }

 menuentry "Pmagic - Clonezilla 64"{
	echo $message
	search --set -f $linux_64
	linux $linux_64 $default_settings clonezilla=yes
	initrd $initrd_img
 }
 menuentry "Pmagic - Clonezilla 32"{
	search --set -f $linux_32
	linux $linux_32 $default_settings clonezilla=yes
	initrd $initrd_img32
 }
}
```


### Debian 

On monte l'image iso debian

    sudo mount -t iso9660 -o loop,ro iso/debian/bullseye/firmware-11.2.0-amd64-netinst.iso /mnt/iso

Copier le dossier 

    sudo rsync -avz --exclude "debian" /mnt/iso /mnt/multiboot/debian

Créer le fichier **/mnt/multiboot/boot/grub/mbusb.d/debian.d/generic.cfg**

```
submenu "Debian 11 + Firmware -->" {

  menuentry --hotkey=g 'Graphical install' {
    set background_color=black
    linux    /debian/install.amd/vmlinuz vga=788 --- quiet 
    initrd   /debian/install.amd/gtk/initrd.gz
  }
  menuentry --hotkey=i 'Install' {
    set background_color=black
    linux    /debian/install.amd/vmlinuz vga=788 --- quiet 
    initrd   /debian/install.amd/initrd.gz
  }
}
```

## Démarrez un ISO avec MEMDISK (facultatif)

[Using Syslinux's MEMDISK][usingmemdisk], , un fichier ISO peut être chargé directement en mémoire (si le système en a assez), ce qui permettra de démarrer certaines images ISO non prises en charge.

Pour obtenir le fichier binaire de MEMDISK, vous pouvez installer [syslinux]() à l'aide du gestionnaire de paquets de votre système et le trouver dans `/usr/lib/syslinux/memdisk` ou `/usr/lib/syslinux/bios/memdisk`, suivant votre distribution.

    sudo pacman -S syslinux # le fichier /usr/lib64/syslinux/bios/memdisk

Alternativement, vous pouvez télécharger l'archive officielle depuis [kernel.org](https://www.kernel.org/), dans ce cas, vous trouverez le binaire  `/bios/memdisk/memdisk`.

Une fois que vous avez le fichier, il suffit de le copier dans votre partition de données:

```sh
cp -f /usr/lib64/syslinux/bios/memdisk /mnt/multiboot/boot/grub/
```

Démontage

	sudo umount /mnt/multiboot
	sudo umount /mnt/multiboot/efi


## Test périphérique USB avec QEMU

Pour tester le lecteur USB nouvellement créé dans un environnement virtuel, exécutez:

	sudo qemu-system-x86_64 -enable-kvm -rtc base=localtime -m 2G -vga std -drive file=/dev/sdd,readonly=on,cache=none,format=raw,if=virtio

Remplacer */dev/sdd* par le nom du périphérique USB  (par exemple. */dev/sdb,/dev/sdc,etc...*).  
Exécutez `lsblk` pour obtenir cette information.  



## Annexe

### Obtenir des fichiers amorçables ISO

Vous pouvez télécharger des fichiers ISO à partir de ces sites ,dans le dossier `<mountpoint>/boot/isos`:

* **[Antergos](https://antergos.com/)**:un système d'exploitation moderne, élégant et puissant basé sur une des meilleures distributions Linux disponibles, Arch Linux.
* **[Arch Linux](https://www.archlinux.org/)**: une distribution légère et flexible de Linux® qui tente de rester simple.
* **[AVG Rescue CD](http://www.avg.com/ww-en/rescue-cd-business-edition)**:un outil pour réparer les pannes du système et permettre à vos systèmes de fonctionner à pleine capacité.
* **[BackBox](https://backbox.org/)**: une distribution basée sur Ubuntu développée pour effectuer des tests de pénétration et des évaluations de sécurité.
* **[CentOS](http://www.centos.org/)**: un effort de logiciel libre axé sur la communauté visant à fournir un écosystème open source robuste.
* **[Clonezilla Live](http://clonezilla.org/clonezilla-live.php)**: une petite distribution GNU/Linux pour le "clonage" des  ordinateurs x86/amd64 (x86-64).
* **[Debian](https://www.debian.org/)**: un système d'exploitation gratuit (OS) pour votre ordinateur.
* **[elementary OS](https://elementary.io/)**: un remplacement rapide et ouvert pour Windows et OS X.
* **[Fedora](https://getfedora.org/)**: un système d'exploitation poli et facile à utiliser.
* **[Gentoo Linux](https://www.gentoo.org/)**: une distribution Linux souple et basée sur les sources.
* **[GParted Live](http://gparted.sourceforge.net/livecd.php)**: outil de partitionnement sur une distribution GNU/Linux.
* **[Grml Live Linux](https://grml.org/)**: un système live bootable basé sur Debian qui comprend une collection de logiciels GNU/Linux spécialement pour les administrateurs système.
* **[Hiren's BootCD](http://www.hirensbootcd.org/)**: une trousse de premiers soins pour votre ordinateur.
* **[Kali Linux](https://www.kali.org/)**: une distribution Linux dérivée de Debian conçue pour le forensic numérique et les tests de pénétration.
* **[Linux Mint](https://linuxmint.com/)**: une distribution basée sur Ubuntu dont le but est de fournir une expérience complète en incluant des plugins de navigateur, des codecs multimédia, la prise en charge de la lecture de DVD, Java et autres Composants.
* **[Manjaro Linux](https://manjaro.org/)**: une distribution Linux conviviale basée sur le système d'exploitation Arch développé indépendamment.
* **[openSUSE](https://www.opensuse.org/)**: un projet et une distribution Linux sponsorisés par SUSE Linux GmbH et d'autres sociétés.
* **[Parabola GNU/Linux-libre](https://www.parabola.nu/)**: un effort communautaire pour fournir un système d'exploitation entièrement libre simple et léger.
* **[Parted Magic](http://partedmagic.com/)**: une solution complète de gestion des disques durs et plus encore.
* **[Slax Linux](http://www.slax.org/)**: un système d'exploitation Linux moderne, portable, petit et rapide avec une approche modulaire et un design exceptionnel.
* **[SliTaz](http://www.slitaz.org/)**: un système d'exploitation sécurisé et performant utilisant le noyau Linux et le logiciel GNU.
* **[SystemRescueCd](http://www.sysresccd.org/)**: une solution de secours de système Linux pour administrer ou réparer votre système et vos données après une panne.
* **[Tails](https://tails.boum.org/)**: un système d'exploitation en direct, qui vise à préserver votre vie privée et l'anonymat.
* **[Trisquel GNU/Linux](https://trisquel.info/)**:  un système d'exploitation entièrement gratuit pour les particuliers, les petites entreprises et les centres éducatifs.
* **[Ubuntu](http://www.ubuntu.com/)**: une plateforme logicielle open source qui va du cloud, au smartphone.
* **[Void](http://www.voidlinux.eu/)**: un système d'exploitation à usage général, basé sur le noyau Linux® monolithique.

Vous pouvez obtenir des noyaux iPXE à partir de ces sites Web (sauvegardez sur `<mountpoint>/boot/krnl`):

* **[boot.rackspace.com](http://boot.rackspace.com/)**: une collection de scripts iPXE qui vous permettent de rapidement redémarrer les systèmes d'exploitation, les utilitaires et d'autres outils très facilement.
* **[netboot.xyz](https://netboot.xyz/)**: un moyen de sélectionner les différents installateurs ou utilitaires du système d'exploitation à partir d'un emplacement dans le BIOS sans avoir besoin d'aller chercher le support pour exécuter l'outil.


### Liens

* [How to Boot ISO Files From GRUB2 Boot Loader](https://www.linuxbabe.com/desktop-linux/boot-from-iso-files-using-grub2-boot-loader)
* [memdisk](http://www.syslinux.org/wiki/index.php?title=MEMDISK)
* [syslinux](http://www.syslinux.org/)
* [kernel.org](https://www.kernel.org/pub/linux/utils/boot/syslinux/)
* [qemu](http://qemu.org/)
* [usingmemdisk](https://wiki.archlinux.org/index.php/Multiboot_USB_drive#Using_Syslinux_and_memdisk)
* [multipass-usb](https://github.com/Thermionix/multipass-usb)
* [multiboot-usb](http://www.circuidipity.com/multi-boot-usb.html)
* [multibootusb](http://www.panticz.de/MultiBootUSB)
* [loop-boot](http://forums.kali.org/showthread.php?1025-Grub2-Loop-Boot-Solution)
* [grml-usb-stick](http://www.gtkdb.de/index_7_2627.html)
* [sgdisk](http://www.rodsbooks.com/gdisk/sgdisk.html)
* [hybridmbr](http://www.rodsbooks.com/gdisk/hybrid.html)
* [efi+bios](https://wiki.archlinux.org/index.php/Multiboot_USB_drive#Hybrid_UEFI_GPT_.2B_BIOS_GPT.2FMBR_boot)

