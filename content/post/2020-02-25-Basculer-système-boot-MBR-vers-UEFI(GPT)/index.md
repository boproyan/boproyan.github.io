+++
title = 'Ordinateur Bureau PC1 démarrage UEFI (GPT)'
date = 2020-02-25 00:00:00 +0100
categories = ['archlinux']
+++
## Basculer d'un système de boot MBR vers UEFI (GPT)

**Il faut 2 disques SATA pour réaliser cette opération**

Toutes les commandes en mode su  

### Changer de disque de boot

Actuellement le boot se fait sur  `SATA6G_1` de la carte mère : SATA6G_1 CR120BX300SSD1 (120.0GB)  
Un second disque est présent sur `SATA6G_2` de la carte mère : Crucial_CT512MX100SSD1 (512.1GB)  

Ce disque est formaté LVM EXT4  


	[root@yannick-pc yannick]# pvs

```
  PV         VG         Fmt  Attr PSize    PFree  
  /dev/sda1  ssd-vga    lvm2 a--  <111,79g <51,79g
  /dev/sdb1  ssd-512    lvm2 a--  <476,94g 100,00g
```

La partion root archlinux est monté sur le volume logique root de ssd-vga

	mount |grep mapper

```
/dev/mapper/ssd--vga-root on / type ext4 (rw,relatime)
```

On va créer une partition root de 60G sur le volume ssd-512 avec système de fichiers EXT4

	lvcreate -n root -L 60G ssd-512
	mkfs.ext4 /dev/ssd-512/root

On "boot" sur une clé USB contenant "partition magic"  
Création des dossiers "root" et montage

	mkdir /mnt/{vga,512}
	mount /dev/ssd-vga/root /mnt/vga
	mount /dev/ssd-512/root /mnt/512

Copie de /mnt/vga vers /mnt/512

	cp -a /mnt/vga/* /mnt/512/

Redémarrer normalement (sans clé USB)

Monter le nouveau "root" sur /mnt/temp

	mount /dev/ssd-512/root /mnt/temp

Passage en chroot sur le nouveau "root"

```
mount /dev/mapper/ssd--512-root /mnt/temp
mount -t proc none /mnt/temp/proc
mount -o bind /dev /mnt/temp/dev
mount -t sysfs sys /mnt/temp/sys
chroot /mnt/temp/ /bin/bash
```

Relever le UUID de /dev/mapper/ssd--512-root

	blkid

```
/dev/mapper/ssd--512-root: UUID="1e532854-7ca5-45a9-85f6-0b5406da6646" BLOCK_SIZE="4096" TYPE="ext4"
```

Modifier le /etc/fstab pour que le montage du root soit sur l'UUID de  /dev/mapper/ssd--512-root

	nano /etc/fstab

```
[root@yannick-pc yannick]# cat /etc/fstab
# /dev/mapper/ssd--512-root
UUID=1e532854-7ca5-45a9-85f6-0b5406da6646	/		ext4		rw,relatime	0 1

# /dev/mapper/ssd--vga-root
#UUID=a6f5dd58-3970-4209-8412-6bfbf1356a03	/         	ext4      	rw,relatime	0 1
```

Installer le grub sur le disque physique qui contient LVM ssd-512 (/dev/sdb)

	grub-install --no-floppy --recheck /dev/sdb
	grub-mkconfig -o /boot/grub/grub.cfg

Reboot , changer le disque de démarrage sur le boot...  

>Le boot se fait maintenant sur `SATA6G_2` de la carte mère : Crucial_CT512MX100SSD1 (512.1GB)  

Vérification 

	mount |grep mapper

```
[...]
/dev/mapper/ssd--512-root on / type ext4 (rw,relatime)
[...]
```

Le disque 1 /dev/sda est libéré   
Supprimer les volumes logique,physique  

	lvremove /dev/ssd-vga/root
	vgremove ssd-vga
	pvremove /dev/sda1


## Arch Linux avec LVM sur disque partitionné gpt

* [Archlinux - Installation de base](https://wiki.archlinux.fr/Installation)


On "boot" sur une clé USB archlinux  

    loadkeys fr

### Vérifier le mode de démarrage  

Si le mode UEFI est activé sur une carte mère UEFI, Archiso démarrera Arch Linux en conséquence par le biais d'un système de démarrage. Pour vérifier cela, consultez le répertoire efivars :

    ls /sys/firmware/efi/efivars

Si le répertoire n'existe pas, le système peut être démarré en mode BIOS ou CSM. 

### Se connecter à l'internet  

Pour établir une connexion réseau, suivez les étapes suivantes :

    Assurez-vous que votre interface réseau est répertoriée et activée 

    ip a

### Mise à jour de l'heure système

Afin que pacman (et pacstrap) puisse vérifier la validité des paquets téléchargés, il est nécessaire que la date soit correcte. Dans le cas contraire, vous ne pourrez rien installer ! Vous pouvez vérifier la date et l'heure actuelle avec la commande suivante :

    timedatectl

Il se peut que l'horloge nécessite d'être réglée.

Utilisez la commande suivante pour synchroniser l'horloge système au réseau (si disponible). L'horloge matérielle (RTC, celle du BIOS) ne sera pas modifiée. http://man7.org/linux/man-pages/man1/timedatectl.1.html

    timedatectl set-ntp true

### Partitionnement des disques  

Sur /dev/sda, nous allons créer une nouvelle table de partition GPT pour le système UEFI en utilisant la commande parted. Ensuite, nous créerons les partitions nécessaires pour LVM.
Les étapes à suivre sont les suivantes :  

1. Créer la table de partition GPT
2. Créer une partition système EFI amorçable (taille recommandée : 512MiB)
3. Marquer la partition du système EFI avec le drapeau de démarrage
4. Créer une partition pour /boot
5. Marquer la partition restante pour LVM

La syntaxe pour créer une partition avec des parted est :

	(parted) mkpart part-type fs-type début fin

* part-type: primaire, étendue ou logique (significatif pour les tables de partition MBR uniquement)
* fs-type: Utilisé par parted pour définir le code d'un octet utilisé par les chargeurs de démarrage pour prévisualiser le type de données dans une partition. Il ne crée pas de système de fichiers.
* start: Début de la partition depuis le début du périphérique. Consiste en un nombre suivi d'une unité, par exemple 1MiB
* end: Fin de la partition à partir du début de l'appareil. 100% signifie que la partition se termine à la fin de l'appareil ( utiliser tout l'espace restant)

Commandes : 

	parted /dev/sda 

```
(parted) mklabel gpt
(parted) mkpart ESP fat32 1MiB 513MiB
(parted) set 1 boot on
(parted) name 1 efi
(parted) mkpart primary 513MiB 800MiB
(parted) name 2 boot
(parted) mkpart primary 800MiB 100%
(parted) name 3 lvm-partition
(parted) print
(parted) quit
```

Résultat

```
GNU Parted 3.3
Utilisation de /dev/sda
Bienvenue sur GNU Parted ! Tapez « help » pour voir la liste des commandes.
(parted) mklabel gpt                                                      
Avertissement: Le type du disque /dev/sda va être effacé et toutes les données vont être perdues. Voulez-vous continuer ?
Oui/Yes/Non/No? Yes                                                       
(parted) mkpart ESP fat32 1MiB 513MiB                                     
(parted) set 1 boot on                                                    
(parted) name 1 efi                                                       
(parted) mkpart primary 513MiB 800MiB                                     
(parted) name 2 boot                                                      
(parted) mkpart primary 800MiB 100%                                       
(parted) name 3 lvm-partition
(parted) print                                                            
Modèle : ATA CT120BX300SSD1 (scsi)
Disque /dev/sda : 120GB
Taille des secteurs (logiques/physiques) : 512B/4096B
Table de partitions : gpt
Drapeaux de disque : 

Numéro  Début   Fin    Taille  Système de fichiers  Nom            Drapeaux
 1      1049kB  538MB  537MB   fat32                efi            démarrage, esp
 2      538MB   839MB  301MB                        boot
 3      839MB   120GB  119GB                        lvm-partition

(parted) quit                                                             
Information: Ne pas oublier de mettre à jour /etc/fstab si nécessaire.
```

### Créer un volume Physique

Utilisez la commande pvcreate pour créer un volume physique sur la partition d'installation de votre système d'exploitation.

	pvcreate /dev/sda3

### Créer un groupe de Volume

Le groupe de volume est créé à partir du volume physique défini précédemment.

	vgcreate  ct120bx /dev/sda3

Avant de pouvoir installer avec LVM , il faut créer des volumes logiques

### Créer des volumes Logiques
Create Logical volumes for system, home and swap.

```
lvcreate -n root -L 60G ct120bx
```

Pour confirmer la création, utilisez la commande :

	lvs

```
  LV      VG         Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root    ct120bx    -wi-a-----   60,00g                                                    
```

### Les systèmes de fichiers

Nous avons besoin de systèmes de fichiers pour stocker les noyaux d'archives (fichiers de démarrage), l'installation du système d'exploitation, les fichiers des utilisateurs et la partition d'échange. 

```
mkfs.fat -F32 /dev/sda1
mkfs.ext2 /dev/sda2
mkfs.ext4 -L root /dev/ct120bx/root
```

On définit les partitions existantes et les nouvelles pour une copie

```
mkdir /mnt/{120,512}
mount /dev/ct120bx/root /mnt/120
mount /dev/ssd-512/root /mnt/512
rsync -av /mnt/512/boot/* /mnt/120/boot/
rm -rf /mnt/120/boot
mkdir -p /mnt/120/boot/efi

mount /dev/sda2 /mnt/120/boot
mount /dev/sda1 /mnt/120/boot/efi
rsync -av /mnt/512/boot/* /mnt/120/boot/
```

On démonte pour un remontage sur /mnt

    umount -R /mnt/120
    umount -R /mnt/512
    rm -r /mnt/{120,512}

Remontage sur /mnt

```
mount /dev/ct120bx/root /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda2 /mnt/boot
mount /dev/sda1 /mnt/boot/efi
```

### Configuration du système

Générer le /etc/fstab

    genfstab -U -p /mnt >> /mnt/etc/fstab

Chrooter dans le nouveau système :

    arch-chroot /mnt

### Installation bootloader GRUB

* [GRUB](https://wiki.archlinux.fr/GRUB)

Créer le répertoire /boot/efi/EFI

    mkdir -p /boot/efi/EFI

Installez l'application UEFI GRUB dans /boot/efi/EFI/arch_grub et ses modules dans /boot/grub/x86_64-efi à l'aide de :

    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub --recheck

Générer le fichier de configuration principal

    grub-mkconfig -o /boot/grub/grub.cfg

### Vérifier/Modifier le fichier fstab

    nano /etc/fstab

```
# /dev/mapper/ct120bx-root LABEL=root
UUID=1cb72c0a-defd-407d-9a74-3e6f6f2e8ca2	/         	ext4      	rw,relatime	0 1

# /dev/sda2
UUID=c34e370e-0bef-4b1b-847a-72516bc116dd	/boot     	ext2      	rw,relatime,stripe=4	0 2

# /dev/sda1
UUID=3D7B-0416      	/boot/efi 	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remount-ro	0 2

# /dev/mapper/ssd--512-homeb
UUID=a3271162-298f-45d9-82f2-e30478991079	/home     	ext4      	rw,relatime	0 2

# 
#UUID=c6ab468e-ed7b-491e-b7c3-010a70816d0c
/dev/mapper/vg--nas--one-yanplus	/home/yannick/media	ext4    	defaults	0 2

# 
#UUID=bfb64d50-9590-49e5-b3ef-afb8c9baba90
/dev/mapper/vg--nas--one-virtuel		/srv/virtuel    ext4    	defaults	0 2
```


### Sortie du chroot et redémarrage

    exit
    umount -R /mnt
    reboot

>Modifier l'ordre de démarrage dans le bios en basculant **SATA6G_1 CR120BX300SSD1 (120.0GB)** sur le **boot 1**  de la carte mère    

