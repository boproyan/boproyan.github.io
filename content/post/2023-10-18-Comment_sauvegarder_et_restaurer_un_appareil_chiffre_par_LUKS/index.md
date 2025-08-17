+++
title = 'Comment sauvegarder et restaurer un appareil chiffré par LUKS'
date = 2023-10-18 00:00:00 +0100
categories = chiffrement
+++
*LUKS (Linux Unified Key Setup) est la norme de facto pour activer le chiffrement de disque sous Linux . Il facilite la compatibilité entre les distributions et assure une gestion sécurisée des mots de passe de plusieurs utilisateurs. LUKS chiffre les données au niveau du bloc de disque, permettant ainsi aux utilisateurs de déployer n'importe quel système de fichiers au-dessus du périphérique de bloc chiffré* 

- [LUKS](#luks)
    - [Installer LUKS](#installer-luks)
    - [Chiffrer le volume USB avec LUKS](#chiffrer-le-volume-usb-avec-luks)
    - [Dump en-tête LUKS](#dump-en-tête-luks)
    - [Ajouter une clé de sauvegarde](#ajouter-une-clé-de-sauvegarde)
    - [Sauvegarde et restauration en-tête clé LUKS](#sauvegarde-et-restauration-en-tête-clé-luks)
    - [Conclusion](#conclusion)

## LUKS

Le chiffrement LUKS utilise un en-tête pour stocker les métadonnées d'un appareil. L'en-tête est généralement placé au début de la partition chiffrée ou du périphérique de bloc brut et contient des informations précieuses telles que le nom et le mode de chiffrement, les emplacements de clé, SALT et des données supplémentaires utilisées pour chiffrer et déchiffrer le périphérique.

L'oubli d'une phrase secrète ou d'un mot de passe sur un appareil chiffré LUKS2 entraîne une perte de données. Un volume entièrement chiffré peut entraîner un échec de démarrage puisque le déchiffrement n'est pas possible sans la phrase secrète. Il n'existe actuellement aucun moyen de récupérer une phrase secrète oubliée à partir d'un appareil chiffré LUKS2. La clé est chiffrée et stockée dans l'en-tête du volume.

Il est prudent de <u>**créer une sauvegarde de votre en-tête LUKS**</u> en cas de problème, comme un en-tête corrompu ou une panne matérielle.

Dans ce didacticiel

1. installer LUKS et chiffrer un volume USB amovible. 
2. comment sauvegarder et restaurer l'en-tête du volume LUKS.

### Installer LUKS

Pour démonstration, nous installerons LUKS sur Ubuntu 22.04. Nous chiffrerons ensuite une clé USB amovible, puis sauvegarderons et restaurerons l’en-tête.  
Ouvrez le terminal, mettez à jour les listes de packages et installez le package cryptsetup comme suit :

    sudo apt update
    sudo apt install cryptsetup

### Chiffrer le volume USB avec LUKS

Connecter un volume USB externe au système Linux.  

    lsblk

```
[...]
sdd                    8:48   1  14,3G  0 disk  
[...]
```

Dans notre cas, le système de fichiers de l'appareil est identifié comme `/dev/sdd`

Ensuite, effacez le système de fichiers de votre appareil à l'aide de la wipefs commande comme indiqué 

    sudo wipefs -a /dev/sdd

REMARQUE : une extrême prudence doit être prise lors de l’exécution de cette commande. Assurez-vous de vérifier le nom/chemin du lecteur deux fois avant d'appuyer sur ENTRÉE. Une erreur peut détruire votre disque principal, entraînant des données irrécupérables
{: .prompt-warning }

Une fois le système de fichiers nettoyé, procédez et chiffrez le volume amovible comme indiqué :

    sudo cryptsetup luksFormat /dev/sdd

Cette commande écrasera toutes les données de votre volume. Pour continuer l'opération, tapez «YES» en majuscule et appuyez sur ENTER. Lorsque vous y êtes invité, fournissez la phrase secrète et confirmez-la.
 
Pour confirmer que l'appareil a été chiffré, accédez-y comme indiqué. Le usb-chiffre paramètre est simplement une étiquette qui peut être n'importe quelle valeur arbitraire.

    sudo cryptsetup luksOpen /dev/sdd usb-chiffre

La commande ci-dessus crée le mappeur suivant : `/dev/mapper/usb-chiffre`  
Vous pouvez le confirmer en exécutant la commande : `ls -l /dev/mapper/usb-chiffre`
 
Ensuite, vous pouvez créer un système de fichiers sur le volume. Nous avons sélectionné le système de fichiers EXT4 sur le lecteur ; n'hésitez pas à spécifier un système de fichiers différent.

    sudo mkfs.ext4 /dev/mapper/usb-chiffre -L usb-chiffre

Créez un point de montage et montez le disque comme suit pour accéder et stocker les données sur le disque.

    sudo mkdir /mnt/usb-chiffre
    sudo mount /dev/mapper/usb-chiffre  /mnt/usb-chiffre

### Dump en-tête LUKS

Utilisez la syntaxe suivante pour afficher tout le contenu de l'en-tête et vider les informations d'en-tête LUKS :
    sudo cryptsetup luksDump DEVICE

Dans la commande suivante, se trouve le périphérique chiffré /dev/sdd

    sudo cryptsetup luksDump /dev/sdd

Cela renseigne les informations suivantes sur le volume chiffré

```
LUKS header information
Version:       	2
Epoch:         	3
Metadata area: 	16384 [bytes]
Keyslots area: 	16744448 [bytes]
UUID:          	9a078732-c8e5-4f6c-a0f1-fc2659ccc5ab
Label:         	(no label)
Subsystem:     	(no subsystem)
Flags:       	(no flags)

Data segments:
  0: crypt
	offset: 16777216 [bytes]
	length: (whole device)
	cipher: aes-xts-plain64
	sector: 512 [bytes]

Keyslots:
  0: luks2
	Key:        512 bits
	Priority:   normal
	Cipher:     aes-xts-plain64
	Cipher key: 512 bits
	PBKDF:      argon2id
	Time cost:  9
	Memory:     1048576
	Threads:    4
	Salt:       dd f8 ab 9e 69 50 6b 8a 8e 96 57 44 bb c6 32 d7 
	            07 21 bf 02 7f ad 22 20 0f 3f f3 e9 4e f2 e1 eb 
	AF stripes: 4000
	AF hash:    sha256
	Area offset:32768 [bytes]
	Area length:258048 [bytes]
	Digest ID:  0
Tokens:
Digests:
  0: pbkdf2
	Hash:       sha256
	Iterations: 116198
	Salt:       e6 79 80 90 e4 27 22 dd 15 85 d9 8d ee 96 69 82 
	            a6 9f 01 e6 1c 93 f6 a2 16 57 47 96 c7 92 8f 86 
	Digest:     58 29 d7 d1 fa 9c 53 41 48 fb 54 49 71 2b 00 d4 
	            6e bb 22 d3 a0 8d f1 57 84 b1 a9 da 2d ac 77 c8 
```

Seul l'emplacement de clé 0 (Keyslots) est utilisé, car nous n'avons ajouté qu'une seule phrase secrète lors du cryptage de l'appareil.  
Les volumes LUKS chiffrés autorisent un total de huit emplacements de clé ou phrases secrètes pour les disques chiffrés. Cela permet aux utilisateurs d'ajouter des phrases secrètes ou des clés de sauvegarde.

Nous ajouterons une clé de secours, ce qui entraînera la création d'un emplacement de clé supplémentaire.

### Ajouter une clé de sauvegarde

Ensuite, ajoutez une clé de sauvegarde en utilisant le paramètre **luksAddKey** comme indiqué pour créer un emplacement de clé supplémentaire :

    sudo cryptsetup luksAddKey --key-slot 1 /dev/sdd

Sachez que vous serez invité à saisir la phrase secrète de l'emplacement de clé 0 avant d'en ajouter une nouvelle.

```
[yann@yann-eos ~]$ sudo cryptsetup luksAddKey --key-slot 1 /dev/sdd
ATTENTION: Le paramètre --key-slot est utilisé pour le nouveau numéro de l'emplacement de clé.
Entrez une phrase secrète existante : 
Entrez une nouvelle phrase secrète pour l'emplacement de clé : 
Vérifiez la phrase secrète : 
```

Videz à nouveau l'en-tête LUKS. Vous verrez un emplacement pour clé supplémentaire, comme indiqué ci-dessous.

    sudo cryptsetup luksDump /dev/sdd

```
LUKS header information
Version:       	2
Epoch:         	4
Metadata area: 	16384 [bytes]
Keyslots area: 	16744448 [bytes]
UUID:          	9a078732-c8e5-4f6c-a0f1-fc2659ccc5ab
Label:         	(no label)
Subsystem:     	(no subsystem)
Flags:       	(no flags)

Data segments:
  0: crypt
	offset: 16777216 [bytes]
	length: (whole device)
	cipher: aes-xts-plain64
	sector: 512 [bytes]

Keyslots:
  0: luks2
	Key:        512 bits
	Priority:   normal
	Cipher:     aes-xts-plain64
	Cipher key: 512 bits
	PBKDF:      argon2id
	Time cost:  9
	Memory:     1048576
	Threads:    4
	Salt:       dd f8 ab 9e 69 50 6b 8a 8e 96 57 44 bb c6 32 d7 
	            07 21 bf 02 7f ad 22 20 0f 3f f3 e9 4e f2 e1 eb 
	AF stripes: 4000
	AF hash:    sha256
	Area offset:32768 [bytes]
	Area length:258048 [bytes]
	Digest ID:  0
  1: luks2
	Key:        512 bits
	Priority:   normal
	Cipher:     aes-xts-plain64
	Cipher key: 512 bits
	PBKDF:      argon2id
	Time cost:  9
	Memory:     1048576
	Threads:    4
	Salt:       c1 11 4c b9 08 10 63 3a 29 c1 f0 56 d7 d0 8d d7 
	            f4 17 7f ed 36 68 3e fd aa f8 59 4d af 4f d3 6a 
	AF stripes: 4000
	AF hash:    sha256
	Area offset:290816 [bytes]
	Area length:258048 [bytes]
	Digest ID:  0
Tokens:
Digests:
  0: pbkdf2
	Hash:       sha256
	Iterations: 116198
	Salt:       e6 79 80 90 e4 27 22 dd 15 85 d9 8d ee 96 69 82 
	            a6 9f 01 e6 1c 93 f6 a2 16 57 47 96 c7 92 8f 86 
	Digest:     58 29 d7 d1 fa 9c 53 41 48 fb 54 49 71 2b 00 d4 
	            6e bb 22 d3 a0 8d f1 57 84 b1 a9 da 2d ac 77 c8 
```

Il y a maintenant deux phrases secrètes (au cas où vous oublieriez la première). Vous pouvez toujours utiliser le second pour le cryptage.

### Sauvegarde et restauration en-tête clé LUKS

Toutes les données seront définitivement perdues si l'en-tête d'un volume LUKS est endommagé, sauf si vous disposez d'une sauvegarde d'en-tête. De plus, si l'emplacement de clé est endommagé, il ne peut être restauré qu'à partir d'une sauvegarde d'en-tête.

Il est fortement conseillé de créer une sauvegarde du fichier d'en-tête. Vous pouvez le faire en utilisant la syntaxe suivante où DEVICE est le volume de bloc et le chemin du fichier /path/of/file dans lequel l'en-tête sera sauvegardé :

    sudo cryptsetup luksHeaderBackup DEVICE --header-backup-file /path/of/file

La commande a le format suivant :

    sudo cryptsetup luksHeaderBackup /dev/sdd --header-backup-file /root/sdb-header.backup

Vous pouvez restaurer un en-tête corrompu à l'aide du paramètre **luksHeaderRestore** comme indiqué dans la commande ci-dessous :

    sudo cryptsetup luksHeaderRestore /dev/sdd –header-backup-file /root/sdb-header.backup

Notez que la procédure de restauration remplace tous les emplacements de clé. Cela implique que seules les phrases secrètes de la sauvegarde fonctionneront par la suite.
{: .prompt-info }

### Conclusion

La création d'un en-tête de périphérique est toujours recommandée comme mesure d'urgence au cas où l'en-tête actuel serait endommagé. Assurez-vous de stocker la sauvegarde de l'en-tête en toute sécurité sur un emplacement hors ligne. Le fichier doit être stocké en dehors de l’appareil chiffré, sinon vous ne pourrez pas le restaurer.

