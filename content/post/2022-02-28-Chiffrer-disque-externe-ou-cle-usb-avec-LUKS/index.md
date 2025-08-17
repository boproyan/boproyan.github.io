+++
title = 'Chiffrer un disque dur externe (ou une clé USB) avec dm-crypt & LUKS'
date = 2022-02-28 00:00:00 +0100
categories = ['chiffrement']
+++
![](luks-logo-blanc.png)

## Chiffrer un disque dur externe (ou une clé USB) avec dm-crypt & LUKS

  * **LUKS** pour Linux Unified Key Setup est une norme de chiffrement de disque. On peut l'utiliser pour chiffrer une ou plusieurs partition et ainsi améliorer la sécurité de son système.
  * dm-crypt est le système utilisé par le noyau Linux pour la gestion des clefs et des disques. Son utilitaire cryptsetup permet de gérer les disques au format LUKS. <u>L'on parle donc de dm-crypt avec LUKS, habituellement simplifié à juste "LUKS".</u> 


[Partition chiffrée avec Cryptsetup](http://doc.ubuntu-fr.org/cryptsetup) .*Afin de protéger au mieux vos données personnelles, il peut être nécessaire de chiffrer vos partitions utilisateur. En effet, si via le système il est impossible d’accéder aux fichiers qui ne vous appartiennent pas, un simple passage sur un livecd permet d’accéder à n’importe quel fichier de votre système. Le chiffrement de partition avec **Cryptsetup** permet d’éviter ça.*

***LUKS** permet de chiffrer l'intégralité d'un disque de telle sorte que celui-ci soit utilisable sur d'autres plates-formes et distributions de Linux (voire d'autres systèmes d'exploitation). Il supporte des mots de passe multiples, afin que plusieurs utilisateurs soient en mesure de déchiffrer le même volume sans partager leur mot de passe.*



### 1-Installation paquet cryptsetup

Le paquet cryptsetup doit être installé debian/ubnuntu:

	sudo apt-get install cryptsetup

archlinux/manjaro ,installé par défaut sur manjaro

	yay -S cryptsetup

### 2-emplacement du disque

Méthode pour déterminer l’emplacement du disque dur externe/clé USB dans /dev. 

Brancher le périphérique USB , exécuter la commande :

    dmesg

```
[14408.326017] sd 11:0:0:0: [sdx] 7818184 512-byte logical blocks: (4.00 GB/3.73 GiB)
[14408.326633] sd 11:0:0:0: [sdx] Write Protect is off
[14408.326637] sd 11:0:0:0: [sdx] Mode Sense: 03 00 00 00
[14408.327270] sd 11:0:0:0: [sdx] No Caching mode page found
[14408.327275] sd 11:0:0:0: [sdx] Assuming drive cache: write through
[14408.332395]  sdx: sdx1
[14408.334889] sd 11:0:0:0: [sdx] Attached SCSI removable disk
```

Ici, [sdx] signifie que l’emplacement est /dev/sdx. 

### 3-Supprimer la table de partition 

On supprime la table de partition et on en créé une nouvelle avec l’utilitaire fdisk. 

    sudo fdisk /dev/sdx  # o + w

L’**option o** crée une nouvelle table vide de partitions DOS, **option w** écrit la table sur le disque et quitte.

### 4-Partionnement du disque

Utiliser *fdisk* ou *gparted* (archlinux graphique)  
*Si vous devez utiliser la clé USB sur des postes avec Windows, il faut réserver la première partition en la formatant en FAT32.*  

    sudo fdisk /dev/sdx  # n + w

Création d'une nouvelle partition **n** puis écriture de la table **w** et sortie.(dnas l'exemple ci-dessous , sde2 a été créé

```
Périphérique Amorçage    Début      Fin Secteurs Taille Id Type
/dev/sde1    *            2048 32767999 32765952  15,6G  c W95 FAT32 (LBA)
/dev/sde2             32768000 61439999 28672000  13,7G 83 Linux
```

### 5-Chiffrement partition

*Vous pouvez entrer la même phrase secrète pour les deux partitions. Par sécurité, générez une phrase complexe. Les applications Revelation, KeePassX & KeePass2 générent et stockent les mots de passe dans une base de données chiffrée.*

Chiffrer la partition (si plusieurs partitions, choisir celle qui est concernée sdx1, sdx2 , etc...)   

	sudo cryptsetup luksFormat /dev/sdx1

```
WARNING!
========
Cette action écrasera définitivement les données sur /dev/sdx1.

Are you sure? (Type uppercase yes): YES
Saisissez la phrase secrète : 
Vérifiez la phrase secrète : 
```

>La passphrase de déchiffrement ,phrase secrète ,est stockée dans un gestionnaire de mot de passe.


### 6-Formatage partition chiffrée

on doit utiliser la commande **cryptsetup luksOpen** pour déchiffrer la partition avant de la formater et lui attribuer un nom unique dans **/dev/mapper/**, par exemple LUKS01.  
On donne un label **ext4-chiffre** à la partition qui apparaîtra dans le gestionnaire de fichier une fois montée.

```
sudo cryptsetup luksOpen /dev/sdx1 LUKS01
    Saisissez la phrase secrète pour /dev/sdx1 : 

sudo mkfs.ext4 /dev/mapper/LUKS01 -L ext4-chiffre
[...]
```

>*Suite au formatage, le propriétaire de la racine du support chiffré est root. Un déplacement de fichier vers le support chiffré se fait par un `sudo cp`.* 

3 possibilités :  

1. changer le propriétaire `sudo chown -R user /run/media/user/nomDuDisque`  
2. donner les droits à l’utilisateur exécutant la commande de formatage `sudo mkfs.ext4 -E root_owner=$UID:$GID /dev/sdx`  
3. donner les droits à l’utilisateur exécutant la commande de formatage et un label `sudo mkfs.ext4 -E root_owner=$UID:$GID -L nomDisque /dev/mapper/nomDuDisque`

Exemple de formatage sur une clé avec 2 partitions (partition 1 FAT32)  

	sudo cryptsetup luksOpen /dev/sde2 CleChiffre
	sudo mkfs.ext4 -E root_owner=1000:100 -L CleChiffre /dev/mapper/CleChiffre

Statut

	sudo cryptsetup -v status CleChiffre

```
/dev/mapper/CleChiffre is active.
  type:    LUKS1
  cipher:  aes-xts-plain64
  keysize: 256 bits
  key location: dm-crypt
  device:  /dev/sde2
  sector size:  512
  offset:  4096 sectors
  size:    28667904 sectors
  mode:    read/write
Opération réussie.
```


### 7-retour en mode chiffré (fermeture)

L’initialisation est terminé ,retour en mode chiffré :

	sudo cryptsetup luksClose LUKS01


## Cli - Montage/Démontage 

*Montage/Démontage en ligne de commande*

Il est possible de déchiffrer et monter la partition manuellement en ligne de commande :

```
sudo cryptsetup luksOpen /dev/sdx1 lenomquevousvoulez        # Phrase secrète demandée
sudo mkdir -p /media/dechiffre
sudo mount /dev/mapper/lenomquevousvoulez /media/dechiffre
```

Le contenu est alors accessible dans /media/dechiffre.

Pour un retour en mode chiffré , il faut démonter et fermer :

```
sudo umount /media/dechiffre
sudo cryptsetup luksClose lenomquevousvoulez
```


## Gestion des phrases secrètes

*Gestion des phrases secrètes ,ajout ,suppression ,modification*

Il est possible d’utiliser plusieurs passphrases (jusqu’à 8) pour déchiffrer le même disque.

```
sudo cryptsetup luksAddKey /dev/sdx1
    Entrez une phrase de passe existante : 
    Entrez une nouvelle phrase secrète pour l'emplacement de clé : 
    Vérifiez la phrase secrète : 
```

Pour en supprimer une :

	sudo cryptsetup luksRemoveKey /dev/sdx1

Pour modifier une passphrase

	sudo cryptsetup luksChangeKey /dev/sdx1

## Etat partition LUKS 

Pour consulter l’état d’une partition LUKS :

	sudo cryptsetup luksDump /dev/sdx1

## Gestion en-tête LUKS 

L’en-tête LUKS est écrit au début du disque. L’écraser empêche définivement le déchiffrement de la partition.

Il est possible d’en faire une sauvegarde dans un fichier :

	sudo cryptsetup luksHeaderBackup /dev/sdx1 --header-backup-file fichier_sav

Et de les restaurer :

	sudo cryptsetup luksHeaderRestore /dev/sdx1 --header-backup-file fichier_sav

Il est possible de vérifier que le header que l’on souhaite restaurer est le bon

	sudo cryptsetup -v --header /chemin/vers/header open /dev/sdx test

```
Key slot 0 unlocked.
Command successful.
```

Pour supprimer l’en-tête (et donc rendre les données définitivement inaccessibles s’il n’y a pas de backup) :

	sudo cryptsetup luksErase /dev/sdx1

## Erreur système de fichiers "crypto_LUKS" inconnu

en mode su

Pour récupérer les données d’un disque chiffré avec Luks suite a l’erreur

```shell
mount /dev/sdc1 /mnt/usb
mount: /mnt/usb: type de système de fichiers « crypto_LUKS » inconnu.
```

Identifier la partition chiffrée puis déchiffrer a l’aide de la clé dans un conteneur que j’appelle ici **usbchiffre**

    lsblk

```
NAME                  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sdc                     8:32   1   3,7G  0 disk 
└─sdc1                  8:33   1   3,7G  0 part 
```

Montage

    cryptsetup open /dev/sdc1 usbchiffre

*Saisissez la phrase secrète pour /dev/sdc1 :*

Créer un répertoire et monter le conteneur dans ce point de montage

    mkdir -p /mnt/usb
    mount /dev/mapper/usbchiffre /mnt/usb

## Liens

* [[Tutoriel] Chiffrer un périphérique sous Linux](https://www.pofilo.fr/post/20181126-luks-cryptsetup/)
* [10 Linux cryptsetup Examples for LUKS Key Management (How to Add, Remove, Change, Reset LUKS encryption Key)](https://www.thegeekstuff.com/2016/03/cryptsetup-lukskey/)
* [How To Linux Hard Disk Encryption With LUKS [ cryptsetup encrypt command ]](https://www.cyberciti.biz/security/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/)