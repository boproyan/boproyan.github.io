+++
title = 'Dell Latitude e6230 - EndeavourOS Plasma KDE chiffré'
date = 2025-08-07
categories = ['archlinux', 'chiffrement', 'lvm']
+++
*EndeavourOS est une distribution GNU/Linux basée sur Arch Linux*

![](EndeavourOS_Logo.png){:width="90"} ![Dell Latitude E6230](dell-latitude-e6230.png){:width="150"}  
[Portable Dell Latitude E6230 - matériel , documentation et bios](/posts/Dell_Latitude_E6230_Caracteristiques_generales_Documentation_et_Bios/)


## EndeavourOS Plasma KDE chiffrée


### EndeavourOS USB Live

*Création d'une clé USB EndeavourOS bootable*

Dans un terminal linux  
Télécharger le dernier fichier iSO : <https://endeavouros.com/latest-release/>  
**EndeavourOS_Mercury-Neo-2025.03.19.iso**

Vérifier checksum

```bash
sha512sum -c EndeavourOS_Mercury-Neo-2025.03.19.iso.sha512sum
```

Résultat de la commande ci dessus après quelques minutes  
*EndeavourOS_Mercury-Neo-2025.03.19.iso: Réussi*

Créer la clé bootable  
Pour savoir sur quel périphérique, connecter la clé sur un port USB d'un ordinateur et lancer la commande `sudo dmesg` ou `lsblk`  
Dans le cas présent , le périphérique USV est **/dev/sdc**

```bash
sudo dd if=EndeavourOS_Mercury-Neo-2025.03.19.iso of=/dev/sdc bs=4M
```

Brancher le réseau filaire sur le DELL  
Insérer la clé bootable EndeavourOS et démarrer le portable DELL , touche F12 pour la sélection   

Sur la page d'acceuil EndeavourOS  
Basculer en Français  

### Installer EndeavourOS

Insérer la clé USB EndeavourOS, basculer en FR et lancer l'installation   
Utilisation de Calamarès  
Installation "en ligne"  
Choix Plasma KDE (Paquets par défaut)  
Pas de firewall  
systemd-boot  
Chiffrer le disque  
Utilisateur yano  
Ordi e6230  
Mot passe utilisateur identique admin  

A la fin de l'installation  
Valider "Redémarrer maintenant"  et "Terminé"

### Premier démarrage

Au message "Please enter passphrase for disk endeavouros...", saisir la phrase mot de passe pour déchiffrer le disque  
Sur la page de connexion utilisateur yano, saisir le mot de passe  

### Mise à jour Système

Mode graphique  
![](plasma-kde01.png)

**Etablir une liaison temporaire SSH**

Pour un accès sur la machine via SSH depuis un poste distant  
Lancer et activer le service : `sudo systemctl enable sshd --now`  
Relever l'adresse ip de la machine : `ip a`  192\.168.0.21 dans notre cas

Se connecter depuis un poste distant `ssh yano@192.168.10.102`

### Utilisateur droits sudo

Modifier sudoers pour accès sudo sans mot de passe à l'utilisateur **yano**

```shell
su               # mot de passe root identique utilisateur
echo "yano     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/20-yano
```

### Activation SSH avec clés

**SSH avec clés**

**A - Poste appelant**  
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) nommé **e6230** pour une liaison SSH avec le portable E6230.

```
ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/e6230
```

Envoyer les clés depuis le poste distant

```
ssh-copy-id -i ~/.ssh/e6230.pub yano@192.168.10.102
```

On se connecte sur la machine

    ssh yano@192.168.0.22

**B - Dell Latitude e6230**  
Modification fichier configuration ssh sur le dell e6230 pour le port

```
sudo nano /etc/ssh/sshd_config
```

```
Port 56230
PasswordAuthentication	no
```

### Parefeu ufw

Installation et activation

```shell
yay -S ufw
sudo ufw enable
```

>`IL FAUT ACTIVER LE PORT 56230 DU PAREFEU UFW !`
{: .prompt-warning }

Ajouter le nouveau port 

```shell
sudo ufw allow 56230/tcp
sudo ufw reload
```

Redémarrer sshd

```bash
sudo systemctl restart sshd
```

Se connecter depuis le poste appelant

```
ssh yano@192.168.10.102 -p 56230 -i /home/yann/.ssh/e6230
```

Ouvrir un terminal

**Motd**

```bash
sudo nano /etc/motd
```

```text
 ___   ___  _     _      _           _    _  _             _      
|   \ | __|| |   | |    | |    __ _ | |_ (_)| |_  _  _  __| | ___ 
| |) || _| | |__ | |__  | |__ / _` ||  _|| ||  _|| || |/ _` |/ -_)
|___/ |___||____||____| |____|\__,_| \__||_| \__| \_,_|\__,_|\___|
       __  ___  ____  __                                          
 ___  / / |_  )|__ / /  \                                         
/ -_)/ _ \ / /  |_ \| () |                                        
\___|\___//___||___/ \__/                                         
 ___  ___   ___   ___  _                          _  __ ___   ___ 
| __|/ _ \ / __| | _ \| | __ _  ___ _ __   __ _  | |/ /|   \ | __|
| _|| (_) |\__ \ |  _/| |/ _` |(_-<| '  \ / _` | | ' < | |) || _| 
|___|\___/ |___/ |_|  |_|\__,_|/__/|_|_|_|\__,_| |_|\_\|___/ |___|
```

### Clé matérielle FIDO2

*Le déverrouillage se fait par saisie d'une phrase mot de passe, on peut ajouter des clés FIDO2 pour un déchiffrement sans mot de passe ([Using FIDO2 keys to unlock LUKS on EndeavourOS](https://forum.endeavouros.com/t/using-fido2-keys-to-unlock-luks-on-endeavouros/51111))*

Installer librairie libfido2 pour la prise en charge des clés Yubico et SoloKeys

```bash
sudo pacman -S libfido2
```

**Enroler clé USB FIDO2 YubiKey 5 NFC**  
![](yubikey5nfc.png){:height="100"}  

Vérifier que la YubiKey est insérée dans un port USB

Lister et enroler la yubikey

```bash
sudo systemd-cryptenroll --fido2-device=list
```

```text
PATH         MANUFACTURER PRODUCT              
/dev/hidraw4 Yubico       YubiKey OTP+FIDO+CCID
```

Vérifier que le disque chiffré est /dev/sda2 : `lsblk`

```
NAME                                          MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                                             8:0    0 447,1G  0 disk  
├─sda1                                          8:1    0     1G  0 part  /efi
└─sda2                                          8:2    0 446,1G  0 part  
  └─luks-d603c182-2d04-4a31-bde6-512ac2d18e7b 254:0    0 446,1G  0 crypt /
```

Vérifier que le chiffrement est luks2 : `sudo cryptsetup luksDump /dev/sda2`

```
LUKS header information
Version:       	2
[...]
```

**Enroler la clé pour le déverrouillage du disque chiffré /dev/sda2**

```
sudo systemd-cryptenroll --fido2-device=auto /dev/sda2
```

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

Si vous avez plusieurs clés répéter l'opération **Enroler la clé pour le déverrouillage du disque chiffré /dev/sda2**

**Prise en charge FIDO2 (crypttab)**  

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
luks-08d4dd05-387f-43e1-9f1f-5d33ea1e5b3b UUID=08d4dd05-387f-43e1-9f1f-5d33ea1e5b3b     none luks
```

Configurer `/etc/crypttab` pour la prise en charge FIDO2

```bash
sudo nano /etc/crypttab
```

Modifiez votre /etc/crypttab pour ajouter fido2-device=auto à chaque appareil crypté que vous avez enrôlé.

La quatrième colonne **luks** est remplacée par **luks,fido2-device=auto**

```
# <name>               <device>                         <password> <options>
luks-08d4dd05-387f-43e1-9f1f-5d33ea1e5b3b UUID=08d4dd05-387f-43e1-9f1f-5d33ea1e5b3b     none luks,fido2-device=auto
```

Sauvegarder et quitter.

Réinitialiser le noyau

```
sudo reinstall-kernels
```

### Passphrase de recouvrement (OPTION)

Les jetons et puces de sécurité FIDO2, PKCS#11 et TPM2 s'associent bien avec les clés de recouvrement : puisque vous n'avez plus besoin de taper votre mot de passe tous les jours, il est logique de vous en débarrasser et d'enregistrer à la place une clé de recouvrement à forte entropie que vous imprimez ou scannez hors écran et conservez dans un endroit physique sûr.  
Voici comment procéder :

```
sudo systemd-cryptenroll --recovery-key /dev/sda2
```

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

### Outil systemd-cryptenroll

**[systemd-cryptenroll](https://wiki.archlinux.org/title/Systemd-cryptenroll)**  
systemd-cryptenroll est un outil permettant d'enregistrer des jetons de sécurité matériels et des périphériques dans un volume crypté LUKS2, qui peuvent ensuite être utilisés pour déverrouiller le volume pendant le démarrage.  
systemd-cryptenroll permet d'enregistrer des cartes à puce, des jetons FIDO2 et des puces de sécurité Trusted Platform Module dans des périphériques LUKS, ainsi que des phrases de passe ordinaires. Ces périphériques sont ensuite déverrouillés par `systemd-cryptsetup@.service` à l'aide des jetons enregistrés. 

systemd-cryptenroll peut lister les keyslots d'un périphérique LUKS, de manière similaire à `cryptsetup luksDump`, mais dans un format plus convivial. 

    sudo systemd-cryptenroll /dev/sda2

Résultat pour disque déchiffrable avec une phrase et 3 clés FIDO2

```
SLOT TYPE    
   0 password
   1 fido2
   2 fido2
   3 fido2
```

### Historique ligne de commande  

Ajoutez la recherche d’historique de la ligne de commande au terminal  
Se connecter en utilisateur  
Tapez un début de commande précédent, puis utilisez shift + up (flèche haut) pour rechercher l’historique filtré avec le début de la commande.

```shell
# Global, tout utilisateur
echo '"\e[1;2A": history-search-backward' | sudo tee -a /etc/inputrc
echo '"\e[1;2B": history-search-forward' | sudo tee -a /etc/inputrc
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

### Réseau wifi

*Le portable DELL est connecté sur le réseau filaire 192.168.0.0/24 pour l'installation*

Paramétrage wifi pour une utilisation sur un autre réseau 192.168.10.0/24  
![](eos009.png){:width="400"}  

### Bluetooth

#### Souris Bluetooth Pebble

**Activer Bluetooth**  
Bluetooth n'est pas actif par défaut, en raison de plusieurs risques de sécurité et pour éviter une consommation d'énergie inutile.

Les packages nécessaires sont installés, mais ils sont désactivés.

Pour pouvoir utiliser Bluetooth, vous devez démarrer le service ou l'activer si vous avez besoin que Bluetooth soit exécuté à chaque démarrage :

```bash
sudo systemctl enable bluetooth --now # à activer par défaut, s'exécutera après chaque démarrage.
```

**Souris Bluetooth Pebble Mouse 2 M350s**  
Basculez entre 3 de vos dispositifs d'une simple pression sur le bouton Easy-Switch.  
Position 1 pour le portable DELL latitude e6230  
![](Peeble-MouseA.png){:height="400"}  
*Pour effacer une configuration existante de la souris bluetooth , garder enfoncer le bouton Easy-Switch jusqu'au clignotement rapide de la led*

Pour ajouter la souris bluetooth au portable DELL, clic droit sur l'icône bluetooth de la barre des tâches, Périphériques puis Rechercher et lorsque l'appareil est détecté , il faut l'appairer  
![](eos006.png){:width="300"}  

#### Ecouteurs bluetooth

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


### Mot de passe (keepassxc)

*On utilise une clé matérielle pour déverrouiller la base de mot de passe*

`La clé matériel utilisée pour la connexion doit être insérée`{: .prompt-tip }

Installer le gestionnaire de mot de passe **keepassxc**

```bash
yay -S keepassxc
```

Ajouter aux favoris "KeepassXC" et lancer l'application → **Ouvrir une base de données existante**  
Base de données --> Ouvrir une base de données (afficher les fichiers cachés) : **~/.keepassx/yannick_xc.kdbx** --> Ouvrir  
![](eos008.png){:width="400"}

Intégration navigateur  
![](keepassxc_browser.png){:width="400"}

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

### Flameshot (copie écran)

**Copie écran (flameshot)**  
[**Flameshot**](https://github.com/lupoDharkael/flameshot) c’est un peu THE TOOL pour faire des captures d’écrans

```
yay -S flameshot
```

Lancer l'application Flameshot et l'icône est visible dans la barre des tâches  
![](flameshot_e6230-1a.png){:width="300"}

Paramétrage de flameshot, clic droit sur icône , Configuration  
![](eos003.png){:width="300"}  
Paramétrage de flameshot  
![](flameshot01.png){:width="400"}  
Le chemin: /home/yano/media/statique/images

### scrpy émulation android

Utilise adb et le port USB

```
yay -S scrcpy
```

*Les icônes pour lancer l'application sont générés à l'installation*

![](eos004.png){:width="300"}  

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

### Plymouth

*[Plymouth - Processus de démarrage graphique](/posts/Plymouth_Processus_de_demarrage_graphique/)*

Installation

    yay -S plymouth plymouth-theme-endeavouros

Modifier image du thème endeavouros  
`sudo cp sharenfs/e6230/Fonds/wallpaper.png /usr/share/plymouth/themes/endeavouros/`
Ajout splash au fichier `/etc/kernel/cmdline`  
`nvme_load=YES nowatchdog splash rw rd.luks.uuid=c3f9cc28-3bb6...`  
Prise en compte

    sudo reinstall-kernels

### Navigateur Librewolf

[Navigateur LibreWolf](/posts/Navigateur_LibreWolf/)

Autoriser l'accès dans l'application keepassxc

### RustDesk

[Client Rustdesk Linux](/posts/RustDesk/#client-rustdesk-linux)

Installation

    yay -S rustdesk-bin

Ouvrir rustdesk et paraméter pour utilisation du serveur relais

### Gestionnaire de fichiers

*Double Commander est un gestionnaire de fichiers multiplateforme au source ouvert avec deux panneaux côte à côte.*

    yay -S doublecmd-qt6

### Alias (.bashrc)

Le fichier avec ses alias

```
alias audio='yt-dlp --extract-audio --audio-format m4a --audio-quality 0 --output "~/Musique/%(title)s.%(ext)s"'
alias audiomp3='yt-dlp --extract-audio --audio-format mp3 --audio-quality 0 --output "~/Musique/%(title)s.%(ext)s"'
alias borglist='$HOME/scripts/borglist.sh'
alias calibreraz='adb -s CNBT80D20191101145 shell -x rm /sdcard/Document/metadata.calibre'
alias certok='$HOME/scripts/ssl-cert-check'

alias etat='$HOME/scripts/etat_des_lieux_PC1.sh  > /mnt/sharenfs/rnmkcy/site/etat_pc1.txt'

alias iceyanwg="sh /mnt/sharenfs/pc1/scripts/wgiceyan.sh"
alias ipleak='curl https://ipv4.ipleak.net/json/'

alias nmapl='echo "sudo nmap -T4 -sP 192.168.0.0/24" && sudo nmap -T4 -sP 192.168.0.0/24'
alias nmap10='echo "sudo nmap -T4 -sP 192.168.10.0/24" && sudo nmap -T4 -sP 192.168.10.0/24'

alias orphelin='sudo pacman -Rsn $(pacman -Qdtq)'

alias rename='$HOME/scripts/remplacer-les-espaces-accents-dans-une-expression.sh'
alias service='systemctl --type=service'
alias sshm='$HOME/scripts/ssh-manager.sh'
alias ssl='$HOME/scripts/ssl-cert-check -f $HOME/scripts/ssldomains'

alias traduc='/usr/local/bin/trans'

alias wgiceyan='sh $HOME/scripts/wgiceyan.sh'

alias youtube='yt-dlp -f bv*+ba --output "~/Vidéos/%(title)s.%(ext)s"'
```

### Gestionnaire sshm

*Pour la gestion des serveurs distants via ssh avec clé*

Pour les serveurs concernés, générer des jeux de clés 

```shell
# Relais RustDesk + Nebula lighthouse:
# ssh yock@92.113.144.37 -p 55037 -i /home/yano/.ssh/e6230-vps-yock

# Stalwart Mail Server:
# ssh yockbis@92.113.144.40 -p 55040 -i /home/yano/.ssh/e6230-vps-yockbis

# Connexion Tunnel SSH pour wireguard islande:
# ssh -L 9500:localhost:8090 iceyan@185.112.146.46 -p 55046 -i /home/yano/.ssh/e6230-iceyan-vps

# Contabo Serveur messagerie maddy xoyize.xyz
# ssh xouser@158.220.91.148 -p 55148 -i /home/yano/.ssh/e6230-xoyize

ssh-keygen -t ed25519 -o -a 100 -f /home/yano/.ssh/e6230-vps-yock
ssh-keygen -t ed25519 -o -a 100 -f /home/yano/.ssh/e6230-vps-yockbis
ssh-keygen -t ed25519 -o -a 100 -f /home/yano/.ssh/e6230-xoyize
ssh-keygen -t ed25519 -o -a 100 -f /home/yano/.ssh/e6230-iceyan-vps
```

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
4. Créer le dossier .borg en mode su: `mkdir -p /root/.borg`

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

**Exécution script sauvegarde borg au boot après 3 min** `/etc/systemd/system/run-script-with-delay.service` et `/etc/systemd/system/run-script-with-delay.timer`  
[Exécution script au boot après 3 min](/posts/BorgBackup_vers-Boite_de_stockage/#exécution-script-au-boot-après-3-min)


## Nebula + Partage NFS

### Nebula

* [Nebula est un outil pour interconnecter de manière transparente des ordinateurs](/posts/Nebula/)
    * [Portable laptop DELL e6230](/posts/Nebula/#portable-laptop-dell-e6230)

### Partage NFS

* [NFS : partage réseau sécurisé et rapide](https://blog.stephane-robert.info/docs/services/stockage/nfs/)
* [How to auto-mount an NFS share using systemd](https://www.geraldonit.com/auto-mount-nfs-share-using-systemd/)

nfs est installé par défaut sur EndeavourOS

    sudo pacman -S nfs-utils

**Créer le point de montage /mnt/sharenfs**  

```shell
sudo mkdir -p /mnt/sharenfs  
sudo chown $USER:$USER -R /mnt/sharenfs  
ln -s /mnt/sharenfs $HOME/sharenfs
```

systemd mount fichier `/etc/systemd/system/mnt-sharenfs.mount`

```
[Unit]
Description=NFS Mount sharenfs au boot
DefaultDependencies=no
Conflicts=umount.target
After=network-online.target nebula.service
Before=umount.target

[Mount]
What=10.19.55.4:/sharenfs
Where=/mnt/sharenfs
Type=nfs
Options=defaults

[Install]
WantedBy=multi-user.target
```

Montage par systemd automount fichier `/etc/systemd/system/mnt-sharenfs.automount`

```
[Unit]
Description=Automount sharenfs

[Automount]
Where=/mnt/sharenfs

[Install]
WantedBy=multi-user.target
```

Rechargement système

```shell
sudo systemctl enable mnt-sharenfs.automount --now
sudo systemctl daemon-reload 
```

Status `systemctl status mnt-sharenfs.automount`

```
● mnt-sharenfs.automount - Automount sharenfs
     Loaded: loaded (/etc/systemd/system/mnt-sharenfs.automount; enabled; preset: disabled)
     Active: active (running) since Thu 2025-08-07 19:15:44 CEST; 4min 14s ago
 Invocation: 161adcd537044b3f9a82d9e95492f3fb
   Triggers: ● mnt-sharenfs.mount
      Where: /mnt/sharenfs

août 07 19:15:44 e6230 systemd[1]: Set up automount Automount sharenfs.
août 07 19:15:53 e6230 systemd[1]: mnt-sharenfs.automount: Got automount request for /mnt/sharenfs, triggered by 4837 (ls)
```

[Fix systemd ordering cycle with NFS automount, Nebula and local-fs.target](https://askubuntu.com/questions/1523234/fix-systemd-ordering-cycle-with-nfs-automount-nebula-and-local-fs-target)

## Développement

### Virtuel Virt-Manager

Script pour une installation complète

```shell
# Créez des groupes kvm et libvirt
sudo groupadd -f kvm  
sudo groupadd -f libvirt
# Ajouter l'utilisateur actuel
sudo usermod -a -G libvirt $USER
sudo usermod -aG kvm $USER

# Installation complète Virt-Manager
sudo pacman -Syu --needed virt-manager qemu-desktop libvirt edk2-ovmf dnsmasq vde2 bridge-utils iptables-nft dmidecode swtpm libguestfs guestfs-tools
# activer le service libvirtd
sudo systemctl enable --now libvirtd.service

# Modifier le fichier /etc/libvirt/libvirtd.conf
sudo sed -i -e 's/#unix_sock_group = "libvirt"/unix_sock_group = "libvirt"/g' /etc/libvirt/libvirtd.conf
sudo sed -i -e 's/#unix_sock_rw_perms = "0777"/unix_sock_rw_perms = "0770"/g' /etc/libvirt/libvirtd.conf
sudo sed -i -e 's/#unix_sock_ro_perms = "0777"/unix_sock_ro_perms = "0777"/g' /etc/libvirt/libvirtd.conf
sudo systemctl restart libvirtd.service

# Créer utilisateur système swtpm
sudo useradd -r swtpm
sudo mkdir -p /var/lib/swtpm-localca
sudo chown swtpm:swtpm /var/lib/swtpm-localca

# Ajout lignes suivantes au fichier /etc/libvirt/qemu.conf
sudo sed -i -e 's/#swtpm_user = "tss"/swtpm_user = "swtpm"/g' /etc/libvirt/qemu.conf
sudo sed -i -e 's/#swtpm_group = "tss"/swtpm_group = "swtpm"/g' /etc/libvirt/qemu.conf
# Utilisateur ID (1000)
sudo sed -i -e 's/#user = "libvirt-qemu"/user = "1000"/g' /etc/libvirt/qemu.conf
sudo sed -i -e 's/#group = "libvirt-qemu"/group = "1000"/g' /etc/libvirt/qemu.conf
```

>Un redémarrage complet de la machine est nécessaire
{: .prompt-tip }

Configurer le réseau virtuel par défaut actif  
![](vmm-e6230.png){:width="500" .normal}

### Go

Archlinux Go

    yay -S go
    go version

*go version go1.24.5 linux/amd64*

### NodeJS et nvm

Archlinux Node.js npm

    yay -S nodejs npm
    node --version && npm --version

v24.5.0  
11.5.2

## Optionnel

### Affichage SDDM

*SDDM (Simple Desktop Display Manager) est un gestionnaire d'affichage moderne, gratuit et open source disponible sur Linux et d'autres plateformes Unix comme FreeBSD. Il fonctionne à la fois avec X11 et Wayland, et est basé sur QtQuick, un framework pour créer des applications QML. SDDM permet un haut degré de personnalisation et, grâce à cela, de nombreux thèmes personnalisés sont disponibles.*

* [SDDM](https://wiki.archlinux.org/title/SDDM_(Fran%C3%A7ais))
* [How to customize the SDDM display manager on Linux](https://linuxconfig.org/how-to-customize-the-sddm-display-manager-on-linux)

### KDE Connect

*[KDE Connect](https://userbase.kde.org/KDEConnect/fr) est un projet qui permet à tous vos appareils de communiquer entre eux.([site de la communauté](https://community.kde.org/KDEConnect))*

Voici ce que KDE Connect peut faire:


*    Recevoir les notifications de votre téléphone sur votre ordinateur de bureau et répondre aux messages
*    Contrôler la lecture de la musique sur votre ordinateur à partir de votre téléphone
*    Utiliser votre téléphone comme télécommande pour votre ordinateur
*    Exécuter des commandes prédéfinies sur votre PC à partir des appareils connectés. 
*    Vérifier le niveau de batterie de votre téléphone depuis votre ordinateur
*    Faire sonner votre téléphone pour vous aider à le trouver
*    Partager des fichiers et des liens entre les deux appareils
*    Parcourir votre téléphone depuis votre ordinateur
*    Contrôler le volume de votre ordinateur en utilisant votre téléphone
*    Envoyer des SMS depuis votre ordinateur


Il est nécessaire de modifier le parefeu pour le fonctionnement de KDE Connect

```bash
sudo firewall-cmd --permanent --zone=public --add-service=kdeconnect
sudo firewall-cmd --reload
```

### Flatpak

*Flatpak offre une plate-forme universelle pour installer, gérer et désinstaller les logiciels sur toutes les distributions Linux.* 

* [Flatpak, un format de paquets universel](https://doc.ubuntu-fr.org/flatpak)
* [Flatpak : Télécharger et installer des applications sur Linux](https://www.malekal.com/flatpak-telecharger-installer-applications-linux/)

Installer flatpak

    yay -S flatpak

