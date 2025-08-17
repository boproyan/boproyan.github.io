+++
title = 'Dell Latitude E6230 - CachyOS BTRFS chiffré LUKS'
date = 2025-07-19
categories = ['archlinux', 'chiffrement']
+++
*CachyOS, une distribution Linux relativement nouvelle, vise à offrir des performances élevées et une expérience utilisateur fluide.*

![Dell Latitude E6230](dell-latitude-e6230.png){:width="150" .normal} [Portable Dell Latitude E6230 - matériel , documentation et bios](/posts/Dell_Latitude_E6230_Caracteristiques_generales_Documentation_et_Bios/)

## Description CachyOS 

* [Mon expérience avec CachyOS...](https://fr.a7la-home.com/cachyos-might-be-my-new-main-distro/)
* [CachyOS Wiki](https://wiki.cachyos.org/)

### Exigences du système

Avant de commencer avec les préparatifs d'installation, assurez-vous que l'ordinateur utilisé répond aux exigences du système nécessaire pour exécuter CachyOS. L'installateur utilise un processus d'installation en ligne de sorte qu'une connexion Internet stable et relativement rapide est obligatoire.

**Caractéristiques minimales recommandées**

* 8 Go RAM
* 50 Go d'espace de stockage (SSD/NVMe)
* CPU capable de x86-64-v3
* 50 Mbps ou une meilleure vitesse Internet
* NVIDIA GPU (900+ - par exemple: GTX 950), AMD +GCN 1.0 (par exemple: AMD R7 240) ou Intel (série intégrée HD Graphics ou plus). Série Arc)

**x86_64 Microarchitecture Level Support**

*    x86_64-v3 Compatible CPUs
    * Intel
         * Haswell and later generations (e.g., Broadwell, Skylake, Coffee Lake, etc)
    * AMD
         * Ryzen Series
*    x86_64-v4 Compatible CPUs
    * Intel
         * Knights Landing (Xeon Phi x200), Knights Mill (Xeon Phi x205), Skylake-SP, Skylake-X, Cannon Lake, Cascade Lake, Cooper Lake, Ice Lake, Rocket Lake, Tiger Lake and Sapphire Rapids
    * AMD
         * Zen4+ CPUs

### Gestionnaires de démarrage proposés

Pour offrir la meilleure expérience sur une gamme d'appareils, **CachyOS** propose actuellement les gestionnaires de démarrage suivants : **systemd-boot, rEFInd, GRUB et Limine**. Cet article wiki décrira le jeu de fonctionnalités de chaque gestionnaire de démarrage et inclut également nos recommandations pour lors de leur choix. Pour la configuration, veuillez consulter [Boot Manager Configuration](https://wiki.cachyos.org/configuration/boot_manager_configuration).

#### systemd-boot

Une partie de la famille systemd, systemd-boot a été créée pour être aussi simple que possible, donc il n'a de soutien que pour les systèmes basés sur l'UEFI. Cette conception simple et efficace garantit sa fiabilité et sa rapidité. Cependant, cela se fait au prix de fonctionnalités avancées prises en charge par d'autres gestionnaires de boot.

**Pour**

* Configuration très simple.
* Les entrées de démarrage sont séparées en plusieurs fichiers, ce qui facilite la gestion.

**Points négatifs**

* Manque de soutien adéquat pour le BIOS/MBR.
* Les os très nus conçoivent et manquent de tout type de thème ou de personnalisation.
* Config n'est pas généré automatiquement à moins que configuré pour le faire. CachyOS inclut **systemd-boot manager** pour offrir une configuration générée automatiquement.
* Seulement capable de lire les images de démarrage sur les systèmes de fichiers supportés par EFI (FAT, FAT16, FAT32).
* Impossible de trouver des images de démarrage sur des partitions autres que les siennes.
* Ne prend pas en charge correctement les retours instantanés de Btrfs en raison de l'exigence de stocker les images du noyau sur la partition de démarrage plutôt que le système de fichiers racine.

**Recommandation**

> Systemd-boot est le gestionnaire de démarrage recommandé et par défaut pour CachyOS. Choisissez celui-ci si vous n'êtes pas sûr.
{: .prompt-tip }

#### GRUB

GRUB est le plus ancien gestionnaire de démarrage disponible. Il a un très grand jeu de fonctionnalités, fonctionne sur presque toutes les machines et est le gestionnaire de démarrage Linux le plus couramment utilisé. Voici une liste de ses principaux avantages et inconvénients.

**Pour**

* Capable de lire les images de démarrage de presque tous les systèmes de fichiers Linux disponibles.
* Largement utilisé et très facile à trouver des informations en ligne.
* Capable de décrypter les partitions de démarrage chiffrées.
* Le seul chargeur de démarrage offert lui permettant de démarrer des machines BIOS.
* Ça semble daté. Cependant a un grand soutien thématique pour compenser.

**Points négatifs**

* "Bloated" en raison du besoin de prendre en charge beaucoup plus vieux matériel et nécessitant beaucoup de pilotes de système de fichiers.

**Recommandation**

* GRUB est le seul gestionnaire de démarrage qui supporte le cryptage de partition de démarrage (Différence par rapport au chiffrement du disque).

### Systèmes de fichiers

CachyOS offre 5 systèmes de fichiers (seulement XFS, BTRFS et EXT4 sont détaillés) pour permettre à l'utilisateur de choisir ce qui correspond le mieux à ses besoins. Voici les avantages, les inconvénients et les recommandations de chaque système de fichiers. Chaque système de fichiers est livré avec ses exigences/utilités préinstallées sur CachyOS.

*BTRFS est le système de fichiers par défaut et recommandé pour CachyOS. Choisissez-le si vous n'êtes pas sûr.*

#### XFS

XFS est un système de fichiers de revue créé et développé par Silicon Graphics, Inc. Il a été créé en 1993, porté à Linux en 2001, et est maintenant largement pris en charge par la plupart des distributions Linux.

**Pour**

* Fast, XFS a été initialement conçu avec la vitesse et l'évolutivité extrême à l'esprit.
* Fiable, XFS utilise plusieurs technologies pour prévenir la corruption des données.
* Résistant à la fragmentation en raison de son étendue et de sa stratégie de répartition retardée.

**Points négatifs**

* Il ne peut pas être réduit.

**Utilitaire espace utilisateur**

Le paquet contenant des outils d'espace utilisateur pour gérer les systèmes de fichiers XFS est `xfsprogs`.

**Recommandation**

> XFS est le système de fichiers recommandé pour les utilisateurs qui n'ont pas besoin de fonctionnalités avancées et veulent simplement un système de fichiers rapide et fiable.
{: .prompt-tip }

#### BTRFS

BTRFS est un système de fichiers moderne copy-on-write(COW) créé en 2007 et déclaré stable dans le noyau linux en 2013. Il est largement soutenu et est principalement connu pour son ensemble de fonctionnalités avancées.

**Pour**

* Compression transparente. BTRFS prend en charge la compression transparente des fichiers pour permettre des économies d'espace importantes sans intervention de l'utilisateur. Caché OS est livré avec la compression ZSTD réglée au niveau 3 par défaut.
* Fonctionnalité d'instantané. BTRFS exploite sa nature COW pour permettre la création d'instantanés de sous-volumes qui prennent très peu d'espace réel.
* Fonction sous-volume permettant un meilleur contrôle du système de fichiers.
* Peut grandir ou rétrécir.
* Très rapide développement.

**Points négatifs**

* Il faut parfois défragmenter ou équilibrer.
* Pire sur les entraînements de rotation en raison de la fragmentation susmentionnée.

**Utilitaire espace utilisateur**

Btrfs userspace utility package est `btrfs-progs`

Présentation du sous-volume

CachyOS fournit une mise en page en sous-volume de la boîte pour permettre une fonctionnalité d'instantané facile.


*    Subvol @ = /
*    Subvol @home = /home
*    Subvol @root = /root
*    Subvol @srv = /srv
*    Subvol @cache = /var/cache
*    Subvol @tmp = /var/tmp
*    Subvol @log = /var/log


**Recommandation**

>BTRFS est recommandé pour les utilisateurs qui veulent la fonctionnalité instantané / sauvegarde et la compression transparente.
{: .prompt-tip }

#### EXT4

EXT4 (quatrième système de fichiers étendu) est le système de fichiers Linux le plus couramment utilisé. EXT4 a été rendu stable dans le noyau linux en 2008.

**Pour**

* Très fréquent permettant un accès facile à de nombreuses ressources.
* Fiable. EXT4 a fait ses preuves en étant très fiable.
* Peut grandir ou rétrécir.

**Points négatifs**

* Construit sur une ancienne base de code.
* Manque de nombreuses fonctionnalités avancées que d'autres systèmes de fichiers offrent.

**Utilitaires d'espace utilisateur**

Le paquet à gérer ext4 est `e2fsprogs`

**Recommandation**

>EXT4 est recommandé pour les utilisateurs qui veulent le système de fichiers le plus simple et le plus utilisé.
{: .prompt-tip }

>Utilisez le système de fichiers par défaut BTRFS car il est considéré stable et a beaucoup de fonctionnalités soignées (snapshots, compression, etc). Utilisez XFS ou EXT4 pour un système de fichiers simple et rapide.
{: .prompt-info }

## CachyOS

![](cachyos-logo.png){: .normal}

## Création d'une Clé USB bootable CachyOS

*Le lecteur USB doit avoir au moins 8 Go d'espace disponible.*

Interface de ligne de commande (Linux)  
Branchez une clé USB.  
Détecter le lecteur USB branché en utilisant la commande suivante: `lsblk`

```
sdc                    8:32   1   7,5G  0 disk  
```

Copiez le contenu iso sur le lecteur USB branché en exécutant dansun terminal

```shell
# Remplacer <usbdrive> avec l'étiquette du lecteur.</usbdrive>
# sudo dd bs=4M if=full_iso_name.iso of=/dev/<usbdrive> status=progress oflag=sync
# Dans notre cas
sudo dd bs=4M if=cachyos-desktop-linux-250713.iso of=/dev/sdc status=progress oflag=sync
```

## Boot USB CachyOS

Brancher la clé USB bootable CachyOS sur le portable DELL   
Démarrer le portable DELL et appuyer sur F12 pour sélectionner USB EFI  
Lancer CachyOS  

![](CachyOS-01.png){:width="500" .normal}  
Basculer en **French** sur la page **Welcome to CachyOS**  

![](CachyOS-04.png){:width="500" .normal}  
Lancer l'installateur

![](CachyOS-05.png){:width="500" .normal}  
Bootloader: Systemd-boot  

![](cachyos004.png)  
![](cachyos005.png)  

![](CachyOS-06.png){: .normal}  
Tester le clavier

![](CachyOS-07.png){: .normal}  
Effacer le disque, btrfs et chiffrer le système

![](CachyOS-14.png){: .normal}  
![](CachyOS-15.png){: .normal}  
![](CachyOS-16.png){: .normal}  
![](CachyOS-17.png){: .normal}  
![](CachyOS-18.png){: .normal}  
![](CachyOS-19.png){: .normal}  
![](CachyOS-20.png){: .normal}  


Premier démarrage  
![](CachyOS-21.png){: .normal}  
![](CachyOS-22.png){: .normal}  
Saisir la phrase de déchiffrement du disque

![](CachyOS-23.png){: .normal}  

## CachyOS DELL Latitude

Après le redémarrage, on va paramétrer le portable DELL Latitude e6230

* Déplacement menu en haut de l'écran
* Activer Wifi **Tenda_YM_5G**
* UFW est le parefeu par défaut



### Utilisateur droits sudo

Modifier sudoers pour accès sudo sans mot de passe à l'utilisateur **yano**  

```shell
su               # mot de passe root identique utilisateur
echo "yano     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/20-yano
```

### Utilitaire yay

base-devel et git sont installés par défaut

Cloner yay

```shell
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

Vérification: `yay --version`

Supprimer le dossier yay

```shell
cd ..
sudo rm -r yay
```

### Activation SSH avec clés

**Portable DELL Latitude e6230**

1. Lancer et activer le service : `sudo systemctl enable sshd --now`  
3. Relever l'adresse ip de la machine : `ip a`  192.168.10.90 dans notre cas
4. Parefeu ouvrir le port 22/tcp : `sudo ufw allow ssh`

**A - Poste appelant**  

Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) nommé **e6230** pour une liaison SSH avec le portable E6230.

```
ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/e6230
```

Envoyer les clés depuis le poste distant

```
ssh-copy-id -i ~/.ssh/e6230.pub yano@192.168.10.90
```

Se connecter depuis un poste distant `ssh yano@192.168.10.90`

![](CachyOS-24.png)

**B - Dell Latitude e6230**  
Modification fichier configuration ssh sur le dell e6230 pour le port

```
sudo nano /etc/ssh/sshd_config
```

```
Port 56230
PasswordAuthentication	no
```

`IL FAUT ACTIVER LE PORT 56230/tcp sur le PAREFEU UFW !`{: .prompt-warning }

Ajouter le nouveau port

```shell
sudo ufw allow 56230/tcp
```

Redémarrer sshd

```bash
sudo systemctl restart sshd
```

Se connecter depuis le poste appelant

```
ssh yano@192.168.10.90 -p 56230 -i /home/yann/.ssh/e6230
```

### Créer les dossiers "utilisateur"

Créer les dossiers `.keepassx` , `Notes` , `scripts` `statique/images` et `statique/_posts`

```bash
mkdir -p ~/{.ssh,.keepassx,scripts,Private}
sudo mkdir -p /srv/media/statique/{images,_posts}
sudo chown $USER:$USER -R /srv/media 
mkdir -p /srv/media/Documents/Dossiers-Locaux-Thunderbird
mkdir -p /srv/media/Notes
mkdir -p ~/Private/.borg
# Lien media
ln -s /srv/media $HOME/media
# Lien pour affichage des images avec éditeur Retext
sudo ln -s /srv/media/statique/images /images
```

### Flameshot (copie écran)

**Copie écran (flameshot)**  
[**Flameshot**](https://github.com/lupoDharkael/flameshot) c’est un peu THE TOOL pour faire des captures d’écrans

```
yay -S flameshot
```

Lancer l'application **Flameshot** et l'icône est visible dans la barre des tâches  
![](flameshot_e6230-1a.png){:width="300"}

Paramétrage de flameshot, clic droit sur icône , Configuration  
![](eos003.png){:width="300"}  
Paramétrage de flameshot  
![](flameshot01.png){:width="400"}
![](eos003a.png){:width="400"}  

### Souris Bluetooth Pebble

**Souris Bluetooth Pebble Mouse 2 M350s**  
Basculez entre 3 de vos dispositifs d'une simple pression sur le bouton Easy-Switch.  
Position 1 pour le portable DELL latitude e6230  
![](Peeble-MouseA.png){:height="400"}  
*Pour effacer une configuration existante de la souris bluetooth , garder enfoncer le bouton Easy-Switch jusqu'au clignotement rapide de la led*

Pour ajouter la souris bluetooth au portable DELL, clic droit sur l'icône bluetooth de la barre des tâches, Périphériques puis Rechercher et lorsque l'appareil est détecté , il faut l'appairer  
![](eos006a.png){:width="300"}   
![](eos006b.png){:width="300"}   

### Ecouteurs bluetooth (INACTIF)

![](soundcore-liberty-air.png)  
Soundcore Liberty Air 2  

* [Test des Anker Soundcore Liberty Air 2](https://www.cnetfrance.fr/produits/test-anker-soundcore-liberty-air-2-39895873.htm)
* [Liberty Air 2 Support Videos](https://support.soundcore.com/s/product/a085g000000NlyiAAC/liberty-air-2)

Utiliser le gestionnaire bluetooth et la recherche  
![](bluetooth04.png){:width="400"}  

Lorsque le périphérique est détecté, il faut l'appairer, clic-droit --> Appairer  
Après appairage  
![](bluetooth05.png){:width="400"}  

`Pour que le périphèrique fonctionne correctement, il est IMPERATIF de redémarrer la machine`{: .prompt-info }

Après redémarrage, il faut séléctionner le profil audio  
![](bluetooth06.png){:width="500"}  

### Client Nextcloud

![](nextcloud-logo-a.png){:width="80"}

Installation client nextcloud

```
yay -S nextcloud-client 
```

Démarrer le client nextcloud , après avoir renseigné l'url <https://cloud.rnmkcy.eu> ,login et mot de passe pour la connexion

Trousseau de clé avec mot de passe idem connexion utilisateur

Paramétrage

* Menu → Lancer **Client de synchronisation nextcloud**
* Adresse du serveur : <https://cloud.xoyaz.xyz>  
  Se connecter avec un **mot de passe application nextcloud "Synchro DELL e6230"**
* Nom d’utilisateur : yann
* Mot de passe : xxxxx  
  ![](nextcloud_xfce02.png){:width="300"}  
  Puis saisir l'adresse : https://cloud.rnmkcy.eu  
  Le nagigateur s'ouvre sur l'adresse saisie   
  ![](nextcloud_xfce02a.png){:width="500"}  
  ![](nextcloud_xfce02b.png){:width="500"}  
  ![](nextcloud_xfce02c.png){:width="500"}    
* Sauter les dossiers à synchroniser, Ignorer la configuration des dossiers
  ![](nextcloud_xfce03.png){:width="400"}  
* Trousseau de clés = mot de passe connexion utilisateur  
  ![](nextcloud_xfce05.png){:width="400"}
* Paramètres nextcloud  
  ![](e6230-nextcloud-a.png){:width="400"}

Saisir les différents dossiers à synhroniser  
![](e6230-nextcloud-b.png){:width="400"}


### Clé matérielle FIDO2

*Le déverrouillage se fait par saisie d'une phrase mot de passe, on peut ajouter des clés FIDO2 pour un déchiffrement sans mot de passe ([Using FIDO2 keys to unlock LUKS on EndeavourOS](https://forum.endeavouros.com/t/using-fido2-keys-to-unlock-luks-on-endeavouros/51111))*

Installer librairie libfido2 pour la prise en charge des clés Yubico et SoloKeys

```bash
sudo pacman -S libfido2
```

Vérifier que le disque chiffré est /dev/sda2 : `lsblk`

```
NAME                                          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                                             8:0    0 111,8G  0 disk  
├─sda1                                          8:1    0     2G  0 part  /boot
└─sda2                                          8:2    0 109,8G  0 part  
  └─luks-17xxxxxxxxxxxxxxx7e1                 253:0    0 109,8G  0 crypt /var/tmp
                                                                         /var/cache
                                                                         /var/log
                                                                         /root
                                                                         /srv
                                                                         /home
                                                                         /
zram0                                         252:0    0  15,5G  0 disk  [SWAP]
```

Vérifier que le chiffrement est luks2 : `sudo cryptsetup luksDump /dev/sda2`

```
LUKS header information
Version:       	2
[...]
```

#### Enroler clé USB FIDO2 YubiKey 5 NFC

![](yubikey5nfc.png){:height="100"}  

Vérifier que la YubiKey est insérée dans un port USB

Lister la yubikey 

```bash
sudo systemd-cryptenroll --fido2-device=list
```

```text
PATH         MANUFACTURER PRODUCT              
/dev/hidraw4 Yubico       YubiKey OTP+FIDO+CCID
```

Clé yubikey "24 554 586" pour le déverrouillage du disque chiffré /dev/sda2**

```
sudo systemd-cryptenroll --fido2-device=auto /dev/sda2
```

Résulat de la commande précédente

```
🔐 Please enter current passphrase for disk /dev/sda2: *********************   
Requested to lock with PIN, but FIDO2 device /dev/hidraw4 does not support it, disabling.
Initializing FIDO2 credential on security token.
👆 (Hint: This might require confirmation of user presence on security token.)
Generating secret key on FIDO2 security token.
👆 In order to allow secret key generation, please confirm presence on security token.
New FIDO2 token enrolled as key slot 1.
```

Le **Y** de la clé se met à clignoter , il suffit de poser son doigt sur l'emplacement du **Y** pour le déverrouillage
{: .prompt-info }

Si vous avez plusieurs clés répéter l'opération **Enroler clé USB FIDO2 YubiKey 5 NFC**

Nous avons ajouté 3 clés yubikey FIDO2

1. yubikey "24 554 586"
2. yubikey "29 085 988"
3. yubikey "24 554 581"

Exemple  
![](CachyOS-27.png)

#### Prise en charge FIDO2 (crypttab)

Le fichier `/etc/crypttab` contient la liste des périphériques à déverrouiller automatiquement.  
Chaque ligne du fichier crypttab est de la forme :  
`<target name> <source device> <key file> <options>`

* `<target name>` : Nom à donner au mappage (/dev/mapper/name), dans le cas présent "secret"
* `<source device>` : l'identifiant du container luks, sous la forme UUID=
* `<key file>` : chemin absolu vers le ficher de phrase de passe. Si le déverrouillage doit s'effectuer par saisie d'un mot de passe, indiquer "none"
* `<options>` : liste d'options séparées par des virgules, par exemple luks, discard pour un chiffrage luks et autoriser l'utilisation de la commane fstrim ou discard au niveau du container. L'option keyscript= donne la possibilité d'exécuter un script ou une commande avec le chemin vers le fichier de passe de phrase (paramètre password précédent) fourni comme argument.

`/etc/crypttab` avant modification

```
# <name>               <device>                         <password> <options>
luks-17xxxxxxxxxxxxxxx7e1 UUID=17xxxxxxxxxxxxxxx7e1     none 
```

Modifier `/etc/crypttab` pour la prise en chrrge FIDO2 en ajoutant `fido2-device=auto`  

```bash
sudo nano /etc/crypttab
```

La quatrième colonne **luks** est remplacée par **luks,fido2-device=auto**

```
# <name>                  <device>                 <password> <options>
luks-17xxxxxxxxxxxxxxx7e1 UUID=17xxxxxxxxxxxxxxx7e1 - fido2-device=auto

```

Sauvegarder et quitter.

Réinitialiser le noyau

```shell
sudo mkinitcpio -P
```

`Redémarrer la machine`{: .prompt-info }

Fenêtre de saisie phrase pour déchiffrer le disque   
![](CachyOS-25.png){: .normal}  
Après validation, la fenêtre de connexion utilisateur s'ouvre  
![](CachyOS-25.png){: .normal}  

### Mot de passe (KeePassXC)

![](keepassxc_logo.png){:width="150" .normal}  
*On utilise une clé matérielle pour déverrouiller la base de mot de passe*

`La clé matériel utilisée pour la connexion doit être insérée`{: .prompt-tip }

Installer le gestionnaire de mot de passe **keepassxc**

```bash
yay -S keepassxc
```

Ouvrir "KeePassXC" --> Affichage  
**Affichage → Thème** : Sombre  
**Affichage → Mode compact**  , un redémarrage de l'application est nécessaire

Ajouter aux favoris "KeePassXC" et ouvrir l'application  
![](e6230-keepassxc01.png){:width="400"}  
Base de données --> Ouvrir une base de données (afficher les fichiers cachés) : **~/.keepassx/yannick_xc.kdbx** --> Ouvrir  
![](eos008.png){:width="400"}

Intégration navigateur  
![](keepassxc_browser.png){:width="400"}


### Gestionnaire fichiers

![](doublecmd-logo.png){:width="50" .normal}  
*Double Commander est un gestionnaire de fichiers open source multiplateforme avec deux panneaux côte à côte. Il s'inspire de Total Commander*

Application QT 

```bash
yay -S doublecmd-qt6
```

Ajouter l'application aux favoris  
Les paramètres sont stockés dans le dossier `~/.config/doublecmd/`

### Minicom

Installation

    yay -S minicom

Paramétrage de l'application terminale **minicom**

```
sudo minicom -s
```

> Seul les paramètres à modifier sont cités

Configuration du port série  
![](minicom01.png)  
A -                             Port série : **/dev/ttyUSB0**  
F -              Contrôle de flux matériel : **Non**  
![](minicom02.png)  
Echap  
Enregistrer config. sous dfl  
![](minicom03.png)  
Sortir de Minicom

### scrpy émulation android

Utilise adb et le port USB

```
yay -S scrcpy
```

*Les icônes pour lancer l'application sont générés à l'installation*

![](eos004.png){:width="300"}  

### Navigateur Librewolf

[Navigateur LibreWolf](/posts/Navigateur_LibreWolf/)

Autoriser l'accès dans l'application keepassxc

### RustDesk

[Client Rustdesk Linux](/posts/RustDesk/#client-rustdesk-linux)

Installation

    yay -S rustdesk-bin

Ouvrir rustdesk et paraméter pour utilisation du serveur relais

### Paquets supplémentaires

Lancer la commande

```bash
# qrencode zbar android-tools jq , installés par défaut
yay -S gimp libreoffice-fresh-fr figlet p7zip tmux calibre retext bluefish terminator filezilla borg yt-dlp xclip nmap tigervnc xournalpp tree 
```

Spécifique kde qt

```bash
yay -S qbittorrent partitionmanager 
```

### Historique de la ligne de commande  

Ajoutez la recherche d’historique de la ligne de commande au terminal  
Se connecter en utilisateur  
Tapez un début de commande précédent, puis utilisez shift + up (flèche haut) pour rechercher l’historique filtré avec le début de la commande.

```shell
# Global, tout utilisateur
echo '"\e[1;2A": history-search-backward' | sudo tee -a /etc/inputrc
echo '"\e[1;2B": history-search-forward' | sudo tee -a /etc/inputrc
```

## Sauvegardes

### Snapshots avec Btrfs

*Un snapshot est une copie instantanée d’un sous-volume à un moment donné, qui ne prend de l’espace que lorsqu’il y a des modifications. C’est l’outil idéal pour les sauvegardes, les tests, ou les restaurations rapides*

* [Réaliser une sauvegarde et la restaurer avec Btrfs (LIEN ARTICLE ORIGINAL)](https://connect.ed-diamond.com/Linux-Pratique/lp-114/realiser-une-sauvegarde-et-la-restaurer-avec-btrfs)
* [Réaliser une sauvegarde et la restaurer avec Btrfs](/posts/BTRFS-sauvegarde-restauration/)
* [Howto Btrfs](https://wiki.evolix.org/HowtoBTRFS)


>Un instantané ne fait en aucun cas office de sauvegarde.
{: .prompt-warning }


>Sauvegarde locale, c’est-à-dire sur un support physiquement différent, mais présent sur la machine, dans notre cas un disque dur USB partitionné en Btrfs.
{: .prompt-tip }

Créer un snapshot en lecture seule (idéal pour une sauvegarde) de /srv

```shell
sudo btrfs subvolume snapshot -r /srv /mnt/srv_snap_ro
```

Résulat:  
*Create readonly snapshot of '/srv' in '/mnt/srv_snap_ro'*

Pour exporter la sauvegarde, insérer une clé USB formatée btrfs
Relever le point de montage: `lsblk`

```
sdb                                           8:16   1  14,3G  0 disk  
└─sdb1                                        8:17   1  14,3G  0 part  /run/media/yano/USB-BTRFS
```

Supprimer un sous volume

```shell
sudo btrfs subvolume delete /mnt/srv_snap
```

La partition où l’on va sauvegarder (la clef USB) est montée dans `/run/media/yano/USB-BTRFS`

```shell
sudo btrfs send /mnt/srv_snap_ro | sudo btrfs receive /run/media/yano/USB-BTRFS/
```

Patienter quelques minutes...

Vérification

    ls /run/media/yano/USB-BTRFS/

```
srv_snap_ro
```

La commande `btrfs send` permet d’envoyer un sous-volume (en lecture seule) et la commande `btrfs receive` permet de l’écrire. Comme on le voit, il est très facile de copier un sous-volume (et donc tout son contenu à l’identique) d’un support physique à un autre. Le fait d’utiliser un pipe entre les deux commandes permet une très grande souplesse dans l’utilisation. On le verra lors de la sauvegarde à distance et sur un système de fichiers différent.

>Il arrive que l’on dispose d’un **disque déjà partitionné avec un autre système de fichiers que Btrf**s, dans ce cas-là, l’option `-f` de l’utilitaire btrfs send est prévue pour spécifier un **fichier de sortie** au lieu d’envoyer sur la sortie standard. **Le seul problème est que dans ce cas il n’est pas possible de parcourir le contenu de l’instantané.**
{: .prompt-warning }

Exemple avec une clé USB, formatée Extfat, montée sur `/run/media/yano/USB128G`

```shell
sudo btrfs send -f /run/media/yano/USB128G/srv_snap_ro.btrfs /mnt/srv_snap_ro
```

Patienter quelques minutes...

**Restauration**

Le dossier /mnt/btrfs recevra les restauration

>**Restauration** lorsque la sauvegarde est réalisée sur un système de fichiers Btrfs.  
{: .prompt-tip }

```shell
sudo btrfs send /run/media/yano/USB-BTRFS/srv_snap_ro | sudo btrfs receive /mnt/btrfs
```

Repasser le sous-volume en lecture/écriture.

```shell
sudo btrfs property set -tsubvol /mnt/btrfs/srv_snap_ro ro false
# en cas d'erreur
sudo btrfs property set -f -tsubvol /mnt/btrfs/srv_snap_ro ro false
```

>**Restauration** lorsque la sauvegarde est réalisée sur un système de fichiers autre que Btrfs.  
<u>Il faut restaurer tout le sous-volume</u>
{: .prompt-tip }

Le fichier à restaurer est sur une clé USB formatée Extfat  

    ls -la /run/media/yano/USB128G/

```
-rwxr-xr-x  1 yano yano 1405682551 25 juil. 14:17 srv_snap_ro.btrfs
```

On veut restaurer le fichier `srv_snap_ro.btrfs` sur `/mnt/btrfs/`

```shell
# Pour convertir le fichier en sous-volume
sudo btrfs receive -f /run/media/yano/USB128G/srv_snap_ro.btrfs /mnt/btrfs
```

Repasser le sous-volume en lecture/écriture.

```shell
sudo btrfs property set -ts /mnt/btrfs/srv_snap_ro ro false
```

>ERROR: cannot flip ro->rw with received_uuid set, use force option -f if you really want unset the read-only status. The value of received_uuid is used for incremental send, consider making a snapshot instead. Read more at btrfs-subvolume(8) and Subvolume flags.
{: .prompt-danger }

en cas d'erreur

```shell
sudo btrfs property set -f -ts /mnt/btrfs/srv_snap_ro ro false
```

**Résumé**

1. Créer un instantané en lecture seule dans btrfs.

    `btrfs subvolume snapshot -r /chemin/vers/le/sous-volume/monté /chemin/vers/l'instantané`

2. SI votre destination n'est PAS formatée avec BTRFS, alors votre commande send doit être :

    `sudo btrfs send /chemin/vers/l'instantané -f /media/@_basic_install.txt`

    L'option `"-f"` envoie le sous-volume à la destination sous forme de fichier **stdout**, c'est-à-dire ASCII. où **/media** est une clé USB montée formatée avec EXT4 ou FAT32, ou une destination réseau.

3. Pour le convertir à nouveau en sous-volume, on utiliserait :

    `sudo btrfs receive -f /media/@_basic_install.txt /mnt/instantanés`

4. Le sous-volume reçu sera défini en lecture seule. Utilisez la commande mv pour supprimer l'extension ".txt".

    `sudo mv /mnt/instantanés/@_basic_install.txt /mnt/instantanés/@_basic_install`

5. Pour ajouter l'attribut d'écriture, utilisez

    `sudo btrfs property set -ts @_basic_install ro false`


### BorgBackup

![](borg-logo-b.png){:height="60"}  
[Borg - Laptop Dell e6230](/posts/BorgBackup_vers-Boite_de_stockage/#borg---laptop-dell-e6230)

Le dossier `~/Private/.borg` contient les variables

Résumé des opérations

1. Créer une clé SSH pour l’authentification borg (en mode su)  
`ssh-keygen -t ed25519 -f /root/.ssh/id_borg_ed25519`
2. Ajouter clé publique `/root/.ssh/id_borg_ed25519.pub` au fichier authorized_keys de la boîte de stockage ([Modifier authorized_keys boîte stockage](/BorgBackup_vers-Boite_de_stockage/#modifier-authorized_keys))
3. Tester la connexion ssh à la boite de stockage  
`sftp -P 23 -i /root/.ssh/id_borg_ed25519 u326239@u326239.your-storagebox.de`
4. Créer le dossier .borg en mode su: `sudo mkdir -p /root/.borg`

```bash
# phrase dépôt borg
cat /home/yano/Private/.borg/e6230.passphrase > /root/.borg/e6230.passphrase
# ajout dépôt
cat /home/yano/Private/.borg/e6230.repository > /root/.borg/e6230.repository
```

Initialisation dépôt distant

```bash
export BORG_PASSPHRASE="$(cat /root/.borg/e6230.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
set BORG_REPOSITORY "$(cat /root/.borg/e6230.repository)"
borg init --encryption=repokey $BORG_REPOSITORY
```

Le résultat de la commande

```
IMPORTANT: you will need both KEY AND PASSPHRASE to access this repo!

Key storage location depends on the mode:
- repokey modes: key is stored in the repository directory.
- keyfile modes: key is stored in the home directory of this user.

For any mode, you should:
1. Export the borg key and store the result at a safe place:
   borg key export           REPOSITORY encrypted-key-backup
   borg key export --paper   REPOSITORY encrypted-key-backup.txt
   borg key export --qr/ REPOSITORY encrypted-key-backup/
2. Write down the borg key passphrase and store it at safe place.
```

Le fichier d'exclusion `cp /home/yano/Private/.borg/e6230.exclusions /root/.borg/e6230.exclusions`

```text
/proc
/sys
/dev
/media
/mnt
/cdrom
/tmp
/run
/var/tmp
/var/run
/var/cache
lost+found
/home/yano/.cache
/home/yano/.keepassx
/home/yano/Téléchargements
/home/yano/Musique
/home/yano/Vidéos
/home/yano/media
/home/yano/scripts
/home/yano/sharenfs
```

Créer le bash `/root/.borg/borg-backup.sh` et le rendre exécutable `chmod +x`

```shell
#!/bin/sh
#
# Script de sauvegarde.
#
# Envoie les sauvegardes sur un serveur distant, via le programme Borg.
# Les sauvegardes sont chiffrées
#

flag="/var/tmp/$(basename -- $0).flag"
if [ -e "$flag" ] ;then
echo "flag : $flag"
  if [ "$(date +%F)" == "$(date +%F -r $flag)" ]; then
   # script déjà exécuté 1 fois aujourd'hui, on sort
   echo "script $0 déjà exécuté ce jour($(date +%F))"
   echo "script $0 déjà exécuté ce jour($(date +%F))" | systemd-cat -t sauvegardes -p info
	# Envoi notification
	# DISPLAY=:0 notify-send "$0" "déjà exécuté ce jour\n($flag)" -i /root/.borg/information.png -t 10000
    curl \
    -H "X-Email: ntfy@cinay.eu" \
    -H "Title: DELL e6230 $0" \
    -H "Authorization: Bearer tk_xxxxxxxxxxxxxxxxxxxxxxxx" \
    -H prio:low \
    -H tags:information_source \
    -d "déjà exécuté ce jour ($flag)" \
    https://noti.rnmkcy.eu/yan_infos

   exit 0
  fi

fi

touch $flag

set -e
 
LOG_PATH=/var/log/borg-backup.log
 
export BORG_PASSPHRASE=$(cat /root/.borg/e6230.passphrase)
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
export BORG_RELOCATED_REPO_ACCESS_IS_OK=yes
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
BORG_REPOSITORY=$(cat /root/.borg/e6230.repository)

borg create \
-v --progress --stats \
--exclude-from /root/.borg/e6230.exclusions \
${BORG_REPOSITORY}::${BACKUP_DATE} \
/ \
>> ${LOG_PATH} 2>&1
 
# Nettoyage des anciens backups
# On conserve
# - une archive par jour les 7 derniers jours,
# - une archive par semaine pour les 4 dernières semaines,
# - une archive par mois pour les 6 derniers mois.
 
borg prune \
-v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 \
$BORG_REPOSITORY \
>> ${LOG_PATH} 2>&1

    curl \
    -H "X-Email: ntfy@cinay.eu" \
    -H "Title: DELL e6230 $0" \
    -H "Authorization: Bearer tk_xxxxxxxxxxxxxxxxxxxxxxxx" \
    -H prio:low \
    -H tags:information_source \
    -d "Fin sauvegarde borgbackup" \
    https://noti.rnmkcy.eu/yan_infos
```


Lancer la première sauvegarde (en mode su), lancer `tmux`

```
# exécuter script borgbackup
/root/.borg/borg-backup.sh
# sortie tmux : ctrl+b puis d
# retour tmux : tmux a
```

Lister les sauvegardes borgbackup `/home/yano/borg-e6230.sh`

```bash
#!/bin/bash

echo "Liste des sauvegardes borgbackup"
export BORG_RSH='ssh -i /home/yano/Private/.borg/e6230.borgssh'
export BORG_PASSPHRASE="$(cat /home/yano/Private/.borg/e6230.passphrase)"
export BORG_RELOCATED_REPO_ACCESS_IS_OK=yes
echo "borg list --short $(cat /home/yano/Private/.borg/e6230.repository)"
echo "Veuillez patitenter quelques instants..."
borg list --short "$(cat /home/yano/Private/.borg/e6230.repository)"
```

Le rendre exécutable: `chmod +x ~/borg-e6230.sh` puis exécution `./borg-e6230.sh`  
Résultat  

```
Liste des sauvegardes borgbackup
borg list --short ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/e6230
Veuillez patitenter quelques instants...
2025-07-21-17h11
```


Exécution script sauvegarde borg au boot après 3 min `/etc/systemd/system/run-script-with-delay.service` et `/etc/systemd/system/run-script-with-delay.timer`  
[Exécution script au boot après 3 min](/posts/BorgBackup_vers-Boite_de_stockage/#exécution-script-au-boot-après-3-min)

## Nebula + Partage NFS

### Nebula

* [Nebula est un outil pour interconnecter de manière transparente des ordinateurs](/posts/Nebula/)
    * [Portable laptop DELL e6230](/posts/Nebula/#portable-laptop-dell-e6230)

### Partage NFS

* [NFS : partage réseau sécurisé et rapide](https://blog.stephane-robert.info/docs/services/stockage/nfs/)
* [How to auto-mount an NFS share using systemd](https://www.geraldonit.com/auto-mount-nfs-share-using-systemd/)

Installer nfs

    sudo pacman -S nfs-utils

**Créer les points de montage**  

```shell
sudo mkdir -p /mnt/xoyizenfs  
sudo chown $USER:$USER -R /mnt/xoyizenfs  
ln -s /mnt/xoyizenfs $HOME/xoyizenfs

sudo mkdir -p /mnt/nebunfs  
sudo chown $USER:$USER -R /mnt/nebunfs  
ln -s /mnt/nebunfs $HOME/nebunfs

```

Ajouter les points de montage du serveur nfs au fichier `/etc/fstab`

```
# Les montage NFS cwwk réseau local nebula 10.19.55.4 
10.19.55.4:/nebunfs /mnt/nebunfs nfs4 nofail,x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=10s 0 0
# Les montage NFS xoyize réseau local nebula 10.19.55.5 
10.19.55.5:/sharenfs/xoyize /mnt/xoyizenfs nfs4 nofail,x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=10s 0 0
```

Rechargement et montage

```shell
sudo systemctl daemon-reload && sudo mount -a
```

[Fix systemd ordering cycle with NFS automount, Nebula and local-fs.target](https://askubuntu.com/questions/1523234/fix-systemd-ordering-cycle-with-nfs-automount-nebula-and-local-fs-target)

## Annexe

### Bash - Shell par défaut

[How to change my default shell in Linux using chsh](https://www.cyberciti.biz/faq/change-my-default-shell-in-linux-using-chsh/)

*Sur une unstallation KDE Plasma CachyOS, le shell par défaut est __fish__*

Lister les shell existants: `cat /etc/shells`

```
# Pathnames of valid login shells.
# See shells(5) for details.

/bin/sh
/bin/bash
/bin/rbash
/usr/bin/sh
/usr/bin/bash
/usr/bin/rbash
/usr/bin/systemd-home-fallback-shell
/usr/bin/git-shell
/usr/bin/fish
/bin/fish
/bin/zsh
/usr/bin/zsh
```

Le shell actuel: `printf "shell actuel - %s\n" "$SHELL"`

```
shell actuel - /bin/fish
```

**Passer en shell bash**  
Recherche chemin bash:  
`type -a bash`  
*bash is /usr/bin/bash*

Basculer  
`chsh -s /usr/bin/bash`

```
Modification d'interpréteur pour yano.
Mot de passe : 
L'interpréteur a été modifié.
```

Créer un acceuil sur connexion SSH en modifiant `/etc/motd` en su

```
 ____   _____  _      _       _            _    _  _               _        
|  _ \ | ____|| |    | |     | |     __ _ | |_ (_)| |_  _   _   __| |  ___  
| | | ||  _|  | |    | |     | |    / _` || __|| || __|| | | | / _` | / _ \ 
| |_| || |___ | |___ | |___  | |___| (_| || |_ | || |_ | |_| || (_| ||  __/ 
|____/ |_____||_____||_____| |_____|\__,_| \__||_| \__| \__,_| \__,_| \___| 
        __   ____   _____   ___                                             
  ___  / /_ |___ \ |___ /  / _ \                                            
 / _ \| '_ \  __) |  |_ \ | | | |                                           
|  __/| (_) |/ __/  ___) || |_| |                                           
 \___| \___/|_____||____/  \___/                                            
 _   ___  ____      _   __     ___     _   ___    ___    ___                
/ | / _ \|___ \    / | / /_   ( _ )   / | / _ \  / _ \  / _ \               
| || (_) | __) |   | || '_ \  / _ \   | || | | || (_) || | | |              
| | \__, |/ __/  _ | || (_) || (_) |_ | || |_| |_\__, || |_| |              
|_|   /_/|_____|(_)|_| \___/  \___/(_)|_| \___/(_) /_/  \___/               
```

### Liste token LUKS2 

Le déchiffrement du disque dur /dev/sda2 peut être réalisé de plusieurs manière

* Phrase mot de passe
* Clés FIDO2 (Solokeys et Yubico)
* Phrase de recouvrement

Chacune des possibilités est stockée dans un "SLOT" (8 max, de 0 à 7)  
Informations sur le chiffrement du volume : `sudo cryptsetup luksDump /dev/sda2`  

Tester les différentes clés enregistrées

```
# Phrase de passe
# Aucune clé Yubikeys 
sudo cryptsetup --verbose open --token-id=0 --test-passphrase /dev/sda2

Le jeton 0 a besoin d'une ressource supplémentaire qui est manquante.
Saisissez la phrase secrète pour /dev/sda2 : 
Emplacement de clé 0 déverrouillé.
Commande réussie.

# Yubico 24 554 586
sudo cryptsetup --verbose open --test-passphrase /dev/sda2

Asking FIDO2 token for authentication.
👆 Please confirm presence on security token to unlock.
Emplacement de clé 1 déverrouillé.
Commande réussie.

# Yubico 29 085 988
sudo cryptsetup --verbose open --test-passphrase /dev/sda2

Asking FIDO2 token for authentication.
👆 Please confirm presence on security token to unlock.
Emplacement de clé 2 déverrouillé.
Commande réussie.

# Yubico 24 554 581
sudo cryptsetup --verbose open --test-passphrase /dev/sda2

Asking FIDO2 token for authentication.
👆 Please confirm presence on security token to unlock.
Emplacement de clé 3 déverrouillé.
Commande réussie.
```

### CachyOS CachyOS-PKGBUILDS 

<https://github.com/CachyOS/CachyOS-PKGBUILDS/tree/master>

## Maintenance

### Système BTRFS

[Systeme de fichiers Btrfs](https://blog.stephane-robert.info/docs/admin-serveurs/linux/btrfs/)

### Analyser CachyOS

Caractéristiques de l'installation

Disque BTRFS Chiffré LUKS2  
systemd-cryptsetup@luks\x2d17ed40fd\x2d5601\x2d40a2\x2dbc1b\x2d9411540357e1.service loaded active Cryptography Setup for luks-17ed40fd-5601-40a2-bc1b-9411540357e1

Services actifs: `systemctl list-units --type=service`

Les services à noter

**systemd-boot**  
systemd-boot-random-seed.service Update Boot Loader Random Seed  
Outil: `bootctl`

**Page de connexion utilisateur**  
sddm.service            Simple Desktop Display Manager

**Réseau**  
NetworkManager.service  Network Manager   
Outil: `nmcli`

**Machines virtuelles**  
systemd-machined.service Virtual Machine and Container Registration Service  
Outils: `machinectl`

**Résolution DNS**  
systemd-resolved.service Network Name Resolution  
Outils: `resolvectl`

### Chroot BTRFS

1. Redémarrer sur la clé USB live
2. Basculer en French et ajouter clavier français dans le paramétrage
3. Ouvrir une console et lance le serveur SSH: `sudo systemctl start sshd`
4. Relever adresse: `ip a` --> 192.168.10.102
5. Cahnder mot de passe liveuser: `sudo passwd liveuser` --> rtyuiop

Se connecter depuis un poste sur le réseau

    ssh liveuser@192.168.10.102

Et exécuter les commandes suivantes

```
# passer en root
sudo -s
# déchiffrer le volume, saisir la phrase mot de passe ou par la clé Yubikey
cryptsetup luksOpen /dev/sda2 crypttemp

# https://www.arcolinuxd.com/arch-chroot-and-btrfs-use-the-power-of-arch-chroot-when-your-computer-crashes/
# Montage des dossiers BTRFS
mount /dev/mapper/crypttemp /mnt -o subvol=@
mount /dev/mapper/crypttemp /mnt/root -o subvol=@root
mount /dev/mapper/crypttemp /mnt/home -o subvol=@home
mount /dev/mapper/crypttemp /mnt/var/log -o subvol=@log
mount /dev/mapper/crypttemp /mnt/var/cache -o subvol=@cache
mount /dev/mapper/crypttemp /mnt/var/tmp -o subvol=@tmp
mount /dev/mapper/crypttemp /mnt/srv -o subvol=@srv

mount /dev/sda1 /mnt/boot

# Passage en chroot
arch-chroot /mnt
```

Modifier `/etc/mkinitcpio.conf`

Recréer le noyau

```shell
mkinitcpio -P
```

Sortie

```shell
exit
umount -R /mnt
```

Redémarrage

```shell
systemctl reboot
```
