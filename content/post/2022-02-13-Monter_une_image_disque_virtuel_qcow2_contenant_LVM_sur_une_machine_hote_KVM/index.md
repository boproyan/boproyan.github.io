+++
title = 'Comment monter une image disque virtuel qcow2 contenant LVM sur une machine hôte KVM'
date = 2022-02-13 00:00:00 +0100
categories = ['virtuel']
+++
![Qemu](qemu_logo_icon_170817.png) ![KVM](kvm-logo.png){:height="30"}  

*Question : J'ai une image disque de type qcow2 qui est utilisée par une de mes VM invitées sur QEMU/KVM. Je veux modifier le contenu de l'image disque sans allumer la VM, et pour cela j'ai besoin de monter l'image disque quelque part.  
Existe-t-il un moyen de monter une image disque qcow2 sous Linux ?* 

Lorsque vous exécutez une machine virtuelle (VM) invitée sur un hyperviseur, vous créez une ou plusieurs images de disque dédiées à la VM. En tant que volume de disque virtuel, une image de disque représente le contenu et la structure d'un périphérique de stockage (par exemple, un disque dur ou un lecteur flash) attaché à la VM. Si vous souhaitez modifier des fichiers dans l'image disque d'une VM sans mettre la VM sous tension, vous pouvez monter l'image disque. Vous pourrez alors modifier le contenu de l'image disque avant de la démonter.
Sous Linux, il existe plusieurs façons de monter une image disque, et différents types d'images disques nécessitent différentes approches.


## Méthode qemu-nbd

[How to mount qcow2 disk image on Linux](https://www.xmodulo.com/mount-qcow2-disk-image-linux.html)

Une méthode pour monter une image disque qcow2 consiste à utiliser **qemu-nbd**, un outil en ligne de commande qui exporte une image disque en tant que périphérique de bloc réseau (nbd).  
    
### Montage d'une partition à partir d'une image qcow2

Toutes les commandes se font en mod `su`

Nous allons utiliser **qemu-nbd**, qui permet d'utiliser le protocole NBD (network block device) pour partager l'image disque.

    modprobe nbd max_part=16
    qemu-nbd --connect=/dev/nbd0 /path/to/qcow2/image

* <u>La première commande</u> charge le module du noyau nbd. L'option `max_part=N ` spécifie le nombre maximum de partitions que nous voulons gérer avec **nbd**. 
* <u>La seconde commande</u> exporte l'image disque spécifiée en tant que périphérique réseau (`/dev/nbd0`). En tant que périphérique réseau, vous pouvez utiliser `/dev/nbd0`, `/dev/nbd1`, `/dev/nbd2`, etc. selon ce qui est inutilisé.   

Quant à l'image disque, assurez-vous de spécifier son chemin complet.  
Par exemple, pour exporter l'image xenserver.qcow2 sous `/dev/nbd0` :

    qemu-nbd --connect=/dev/nbd0 /var/lib/libvirt/images/xenserver.qcow2

Après cela, les partitions de disque existantes dans l'image disque seront mappées vers `/dev/nbd0p1`, `/dev/nbd0p2`, `/dev/nbd0p3`, etc.

Pour vérifier la liste des partitions mappées par **nbd**, utilisez **fdisk** :

    fdisk /dev/nbd0 -l

Enfin, choisissez une partition (par exemple, `/dev/nbd0p1`) et montez-la sur un point de montage local (par exemple, `/mnt`).

    mount /dev/nbd0p1 /mnt

Vous pourrez maintenant accéder et modifier le contenu de la partition montée de l'image disque via le point de montage /mnt.

Une fois que vous avez terminé, démontez-le, et déconnectez l'image disque comme suit.

    umount /mnt
    qemu-nbd --disconnect /dev/nbd0

### Montage d'un disque virtuel contenant LVM

[How to Mount Guest Qcow2 Virtual disk Image containing LVM on KVM Host Machine](https://www.thegeekdiary.com/how-to-mount-guest-qcow2-virtual-disk-image-containing-lvm-on-kvm-host-machine/)

On charge le module pour gérer 16 partitions

    modprobe nbd max_part=16

On veut monter une image disque virtuelle contenant 2 partitions , part 1 - EFI et part 2 - LVM  
Le disque virtuel : **/home/yann/virtuel/KVM/endeavour-archlinux.qcow2**

exporter l'image endeavour-archlinux.qcow2 sous /dev/nbd0

    qemu-nbd --connect=/dev/nbd0 /home/yann/virtuel/KVM/endeavour-archlinux.qcow2

Liste des partitions 

    fdisk /dev/nbd0 -l

```
Disque /dev/nbd0 : 20 GiB, 21474836480 octets, 41943040 secteurs
Unités : secteur de 1 × 512 = 512 octets
Taille de secteur (logique / physique) : 512 octets / 512 octets
taille d'E/S (minimale / optimale) : 512 octets / 512 octets
Type d'étiquette de disque : gpt
Identifiant de disque : 7864DCE0-89B1-41F2-B530-EFDB179BBA95

Périphérique  Début      Fin Secteurs Taille Type
/dev/nbd0p1    2048   616447   614400   300M Données de base Microsoft
/dev/nbd0p2  616448 21587967 20971520    10G LVM Linux
```

Le périphérique /dev/nbd0p2, est LVM donc vous devrez localiser les nouvelles PVs/VGs et LVs dans la machine hôte KVM.

    pvscan

```
  PV /dev/sdb3        VG vg-nas-one      lvm2 [<3,64 TiB / 1,95 TiB free]
  PV /dev/nvme0n1p3   VG nvme1tb         lvm2 [930,71 GiB / 425,71 GiB free]
  PV /dev/nbd0p2      VG eos             lvm2 [<10,00 GiB / 1020,00 MiB free]
  PV /dev/sda1        VG ssd-512         lvm2 [<476,94 GiB / 0    free]
  Total: 4 [5,02 TiB] / in use: 4 [5,02 TiB] / in no VG: 0 [0   ]
```

Dans le cas où le PV/VG n'est pas vu, rafraîchissez le cache du volume physique pour que la machine hôte reconnaisse le nouveau PV avec la commande `pvscan --cache`

    vgscan

```
  Found volume group "vg-nas-one" using metadata type lvm2
  Found volume group "nvme1tb" using metadata type lvm2
  Found volume group "eos" using metadata type lvm2
  Found volume group "ssd-512" using metadata type lvm2
```

    lvscan

```
  ACTIVE            '/dev/vg-nas-one/sav' [500,00 GiB] inherit
  ACTIVE            '/dev/vg-nas-one/iso' [200,00 GiB] inherit
  ACTIVE            '/dev/vg-nas-one/lvm-sav' [1,00 TiB] inherit
  ACTIVE            '/dev/nvme1tb/rootnvme' [60,00 GiB] inherit
  ACTIVE            '/dev/nvme1tb/homenvme' [100,00 GiB] inherit
  ACTIVE            '/dev/nvme1tb/medianvme' [300,00 GiB] inherit
  ACTIVE            '/dev/nvme1tb/rooteos' [45,00 GiB] inherit
  inactive          '/dev/eos/rooteos' [9,00 GiB] inherit
  ACTIVE            '/dev/ssd-512/virtuel' [<476,94 GiB] inherit
```

Activer les OS invités VG

    vgchange -ay

```
3 logical volume(s) in volume group "vg-nas-one" now active
4 logical volume(s) in volume group "nvme1tb" now active
1 logical volume(s) in volume group "ssd-512" now active
1 logical volume(s) in volume group "eos" now active
```

    lvscan

```
  ACTIVE            '/dev/vg-nas-one/sav' [500,00 GiB] inherit
  ACTIVE            '/dev/vg-nas-one/iso' [200,00 GiB] inherit
  ACTIVE            '/dev/vg-nas-one/lvm-sav' [1,00 TiB] inherit
  ACTIVE            '/dev/nvme1tb/rootnvme' [60,00 GiB] inherit
  ACTIVE            '/dev/nvme1tb/homenvme' [100,00 GiB] inherit
  ACTIVE            '/dev/nvme1tb/medianvme' [300,00 GiB] inherit
  ACTIVE            '/dev/nvme1tb/rooteos' [45,00 GiB] inherit
  ACTIVE            '/dev/eos/rooteos' [9,00 GiB] inherit
  ACTIVE            '/dev/ssd-512/virtuel' [<476,94 GiB] inherit
```

Créer le dossier

    mkdir -p /mnt/vm-eos/root

Montage

    mount /dev/eos/rooteos /mnt/vm-eos/root

Liste

    ls /mnt/vm-eos/root/

```
bin  boot  dev	etc  home  lib	lib64  lost+found  mnt	opt  proc  root  run  sbin  srv  sys  tmp  usr	var
```

### Supprimez le pilote du noyau NBD

A la fin des opérations

    rmmod nbd



