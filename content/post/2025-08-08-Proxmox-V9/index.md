+++
title = 'Proxmox V9'
date = 2025-08-17
categories = ['debian']
+++
*Proxmox VE est une plate-forme open-source complète pour la virtualisation d'entreprise. Grâce à l'interface Web intégrée, vous pouvez facilement gérer les machines virtuelles, le stockage, la mise en réseau, ...*

![](proxmox-logo.png){: .normal}  

## Proxmox

*Proxmox V9.0: Modernisation générale de la base, avec un passage à Debian 13 (Trixie) et au noyau Linux 6.14.8 - 2 par défaut. Cette base s’accompagne de QEMU 10.0.2, de LXC 6.0.4 et d’OpenZFS 2.3.3.*

### Liens

*    Site Officiel de Proxmox VE : <https://www.proxmox.com/proxmox-ve> 
*    Documentation de Proxmox VE : <https://pve.proxmox.com/wiki/>
*    Projet GitHub de Proxmox VE : <https://github.com/proxmox>
* [Téléchargement des images ISO](https://www.proxmox.com/en/downloads)

### Installation via ISO

* Installation sur machine [Lenovo ThinkCentre M700 Tiny](/posts/Description_materiel_Lenovo_ThinkCentre_M700_Tiny_et_mise_a_jour_BIOS/)
* [Comment installer Proxmox VE 8 et créer des VMs ?](https://www.oameri.com/comment-installer-proxmox-ve-8-et-creer-des-vms/)
* [Installation Proxmox](https://jeromeinformatique.fr/installation-et-configuration-de-proxmox/)
* Proxmox ISO 
    * Fichier **proxmox-ve_9.0-1.iso** 
    * Version 9.0-1  
    * File Size  1.64 GB  
    * Last Updated  August 05, 2025   
    * SHA256SUM    228f948ae696f2448460443f4b619157cab78ee69802acc0d06761ebd4f51c3e

Installer ISO sur clé usb /dev/sdx 

    sudo dd if=proxmox-ve_9.0-1.iso of=/dev/sdx bs=4M

Démarrer la machine Lenovo sur la clé USB et procéder à l'installation de Proxmox (F12 pour accès menu démarrage)

Interface : eno1  
FQDN : pve.home.arpa   
IP address: 192.168.0.215/24  
Gateway: 192.168.0.205  
DNS: 192.168.0.205  

Redémarage à la fin de l'installation et accès lien <https://192.168.0.215:8006>

Pour accéder à l'interface, connectez-vous en tant que root et fournissez le mot de passe que vous définissez lors de l'installation de Proxmox.  
![](proxleno01.png){: width="300" .normal}

Une boîte de dialogue apparaît en disant qu'il n'y a pas d'abonnement valide pour le serveur. Proxmox offre un service complémentaire optionnel auquel vous pouvez vous abonner. Pour ignorer le message, cliquez sur OK.  
![](proxleno01a.png){: .normal}


### Accès proxmox via SSH

*SSH activé avec autorisation connexion root*

Accès SSH : `ssh root@192.168.0.215`

Changer le mot de passe root

```shell
passwd root
```

On désactive les dépôts entreprise 

```shell
# Désactivera message "no valid subscription" sur Proxmox
mv /etc/apt/sources.list.d/pve-enterprise.sources .
mv /etc/apt/sources.list.d/ceph.sources .
```

Lancer les mises à jour

```shell
apt update && apt upgrade -y
```  

### Locales

La commande localectl permet de gérer la langue et la disposition du clavier du système.

```shell
root@pve:~# localectl status
System Locale: LANG=en_US.UTF-8
    VC Keymap: (unset)         
   X11 Layout: fr
    X11 Model: pc105
```

*Le System Locale est la langue du système, VC Keymap est la disposition du clavier dans la VConsole (Les TTY), et X11 Layout la disposition du clavier dans l'interface graphique.*

Lister les locales: `localectl list-locales`  
Si `fr_FR.UTF-8` n'est pas présent il faut reconfigurer les locales  

```shell
dpkg-reconfigure locales
```

![](proxmox-locales.png){:width="350" .normal}  
![](proxmox-locales-a.png){:width="400" .normal}  

Vérifier  

```
root@pve:~# localectl list-locales
C.UTF-8
en_US.UTF-8
fr_FR.UTF-8

root@pve:~# localectl status
System Locale: LANG=fr_FR.UTF-8
    VC Keymap: (unset)         
   X11 Layout: fr
    X11 Model: pc105
```

Si **System Locale** n'est pas **fr_FR.UTF-8**

```shell
localectl set-locale fr_FR.UTF-8
```

### Proxmox clavier et langue

Langue ajouter `language: fr`  en fin du fichier `/etc/pve/datacenter.cfg`

```shell
keyboard: fr
language: fr
```

### Hostname

hostnamectl

```
 Static hostname: pve
       Icon name: computer-desktop
         Chassis: desktop 🖥️
      Machine ID: 7db95bb5bbad47bfa90043c0aa05d738
         Boot ID: 77fb1b06c0b045e2bb4d37c17b7087de
    Product UUID: 44b55a2c-9bc4-11e6-a167-917cd8ac1800
Operating System: Debian GNU/Linux 13 (trixie)        
          Kernel: Linux 6.14.8-2-pve
    Architecture: x86-64
 Hardware Vendor: Lenovo
  Hardware Model: ThinkCentre M700
 Hardware Serial: S4Z98432
Firmware Version: FWKTBCA
   Firmware Date: Mon 2021-12-27
    Firmware Age: 3y 7month 1w 4d                     
```

### Motd

*On se connecte en utilisateur leno sur proxmox via SSH*

Les outils

```shell
apt install jq figlet
```

Effacer et créer motd

    rm /etc/motd && nano /etc/motd

```
 _ __  _ _  ___ __ __ _ __   ___ __ __                              
| '_ \| '_|/ _ \\ \ /| '  \ / _ \\ \ /                              
| .__/|_|  \___//_\_\|_|_|_|\___//_\_\                              
|_|                 _                                               
 _ __ __ __ ___    | |_   ___  _ __   ___     __ _  _ _  _ __  __ _ 
| '_ \\ V // -_) _ | ' \ / _ \| '  \ / -_) _ / _` || '_|| '_ \/ _` |
| .__/ \_/ \___|(_)|_||_|\___/|_|_|_|\___|(_)\__,_||_|  | .__/\__,_|
|_|                                                     |_|         
 _  ___  ___     _   __  ___     __     ___  _  ___                 
/ |/ _ \|_  )   / | / / ( _ )   /  \   |_  )/ || __|                
| |\_, / / /  _ | |/ _ \/ _ \ _| () |_  / / | ||__ \                
|_| /_/ /___|(_)|_|\___/\___/(_)\__/(_)/___||_||___/                
```

### NFS Client

Configuration du client NFS sur Proxmox

```shell
apt install nfs-common # installé par défaut sur trixie
```

Point de montage et lien

```shell
mkdir -p /mnt/sharenfs
```

Ajouter les points de montage du serveur nfs: `nano /etc/fstab`
	
```
192.168.0.205:/sharenfs	/mnt/sharenfs nfs4 nofail,x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=10s,rsize=8192,wsize=8192 0 0
```

Rechargement et montage

```shell
systemctl daemon-reload && mount -a
```

### Connexion SSH avec clés

**connexion avec clé**  
<u>sur l'ordinateur de bureau</u>
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec le serveur.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/lenovo-ed25519

Envoyer les clés publiques sur le serveur KVM   

    ssh-copy-id -i ~/.ssh/lenovo-ed25519.pub root@192.168.0.215

<u>sur le serveur KVM</u>  
On se connecte  

    ssh root@192.168.0.215

Créer une configuration serveur SSH  `/etc/ssh/sshd_config.d/lenovo.conf`

```conf
Port = 55215
PasswordAuthentication no
ChallengeResponseAuthentication yes 
```

Relancer le serveur

    systemctl restart sshd

Test connexion depuis PC1

    ssh -p 55215 -i ~/.ssh/lenovo-ed25519 root@192.168.0.215

### pve.home.arpa

**Ajout autorité certification home.arpa**

```shell
cp /mnt/sharenfs/cwwk/homearpaCA.crt /usr/local/share/ca-certificates/
update-ca-certificates
```

**Nginx web**

Installation

```shell
apt update && apt install nginx
```

Ligne commentée `#include /etc/nginx/sites-enabled/*;` dans fichier `/etc/nginx/nginx.conf` 

**Domaine pve.home.arpa**  
Certificats **pve.home.arpa.crt** **pve.home.arpa.key** dans `/etc/ssl/private/`

```shell
cp /mnt/sharenfs/proxmox/pve.home.arpa.* /etc/ssl/private/
```

Créer un fichier de configuration nginx  
`/etc/nginx/conf.d/pve.home.arpa.conf`

```shell
upstream proxmox {
    server "pve.home.arpa";
}
 
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    rewrite ^(.*) https://$host$1 permanent;
}
 
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name _;
    ssl_certificate      /etc/ssl/private/pve.home.arpa.crt;
    ssl_certificate_key  /etc/ssl/private/pve.home.arpa.key;
    proxy_redirect off;
    location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass https://127.0.0.1:8006;
        proxy_buffering off;
        client_max_body_size 0;
        proxy_connect_timeout  3600s;
        proxy_read_timeout  3600s;
        proxy_send_timeout  3600s;
        send_timeout  3600s;
    }
}
```

Vérifier

```shell
nginx -t
```

Redémarrer le serveur

```shell
systemctl restart nginx
```

Accès <https://pve.home.arpa>

Une boîte de dialogue apparaît en disant qu'il n'y a pas d'abonnement valide pour le serveur. Proxmox offre un service complémentaire optionnel auquel vous pouvez vous abonner. Pour ignorer le message, cliquez sur OK.  
![](proxleno01a.png){: .normal}

>Cette pop-up est générée par du Javascript via le fichier : `/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js`  
Ce fichier est utilisé par Proxmox Virtual Environment (PVE) et Proxmox Backup Server (PBS).
{: .prompt-tip }

```shell
# Avant de modifier ce fichier, on en réalise une copie de sauvegarde :
cp /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js.bak
```

Dans le fichier `/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js` chercher la ligne contenant `.data.status.toLowerCase() !== 'active'`   

inverser la condition : `.data.status.toLowerCase() === 'active' `

Déconnectez-vous, et relancez pve-proxy (pour Proxmox Virtual Environment) et/ou proxmox-backup-proxy (pour Proxmox Backup Server) :

```shell
systemctl restart pveproxy.service
#systemctl restart proxmox-backup-proxy.service
```

En ajoutant le dépôt « no subscription », nous allons étendre cette liste.

Ce dépôt est un référentiel gratuit qui contient les mises à jour de Proxmox sans nécessiter d’abonnement. 

![](proxmox0001b.png){: .normal}


### Authentification 2FA

Ajout authentification à 2 facteurs  
![](proxmox0050.png) 

![](proxmox0051.png){:width="400" .normal}  
Renseigner Description et mot de passe et scanner le code avec application mobile AEGIS   

![](proxmox0052.png){:width="200" .normal}  ![](proxmox0053.png){:width="200" .normal}  

![](proxmox0051.png){:width="400" .normal}  
Saisir le code **329627** dans **Vérifier le code** puis cliquer sur **Ajouter**

![](proxmox0054.png)  
Authentification TOTP ajouter à l'utilisateur leno

### Web pve.home.arpa

Ensuite, vous pouvez accéder à votre serveur Proxmox à partir d'un navigateur: <https://pve.home.arpa>

![](proxmox0001.png){: .normal}  
![](proxmox0001a.png){: .normal}   

### Firefox et consoles NOVNC Proxmox

*corriger le layout clavier azerty*

Quand on utilise Firefox (ou ses forks), avec l'activation d'options renforçant la sécurité, quand on lance un bureau dans le navigateur web, on se retrouve avec un clavier en qwerty bien que le système hôte et le système invité sont bien en disposition azerty.

J'ai constaté ce problème avec :

- Proxmox et les consoles NoVNC

Le problème est lié au paramétrage de protection contre le **Fingerprinting**.

Deux solutions possibles :

- Désactiver le paramétrage **ResistFingerprinting**
- Ajouter des exceptions dans le **ResistFingerprinting**

Comme d'habitude avec Firefox, quand une option n'est pas disponible dans l'interface graphique, on va se rendre dans :

    about:config

Il est possible (mais c'est un peu brutal) de désactiver ResistFingerprinting en passant la clé suivante à false :

    privacy.resistFingerprinting

Le mieux, c'est de laisser cette clé à true et d'ajouter les sites problématiques dans la clé :

    privacy.resistFingerprinting.exemptedDomains

Ici, j'ai mis plusieurs domaines (séparés par virgule) :

    *.home.arpa, *.rnmkcy.eu

Ensuite, j'ai bien mon clavier en azerty (pas besoin de redémarrer Firefox) 

## Gérer la machine Lenovo Proxmox

### cwwk - Domaine pve.home.arpa

*Proxmox pve.home.arpa*

Générer les certificats du domaine local `pve.home.arpa`

```shell
# Machine cwwk
cd $HOME
./gencert.sh pve.home.arpa
# Copie et déplacement
sudo cp pve.home.arpa.crt /etc/ssl/private/
sudo cp pve.home.arpa.key /etc/ssl/private/
mv pve.home.arpa.crt $HOME/sharenfs/rnmkcy/cwwk/home/yick/CA/
mv pve.home.arpa.key $HOME/sharenfs/rnmkcy/cwwk/home/yick/CA/
```

Fichier de configuration proxy nginx `/etc/nginx/conf.d/pve.home.arpa.conf`

```
upstream pvex {
    server 192.168.0.215:8006;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  pve.home.arpa;

    ssl_certificate      /etc/ssl/private/pve.home.arpa.crt;
    ssl_certificate_key  /etc/ssl/private/pve.home.arpa.key;

   location / {
    proxy_pass https://pvex;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_buffering off;
   }

}
```

Vérification et rechargement nginx 

```shell
sudo nginx -t
sudo systemctl reload nginx
```

### Proxmox Pare-feu

* [Proxmox et son firewall intégré : simple, efficace, indispensable](https://belginux.com/proxmox-et-son-firewall-integre-simple-efficace-indispensable/)

**Restreindre l'accès de gestion (Webui & SSH)** à une adresse IP spécifique et bloquer l'accès local trop permissif
Pour ça nous allons seulement avoir à définir deux règles, au niveau Datacenter (Centre de données).

1. Autoriser une adresse spécifique
    * Interface : vmbr0 (à ajuster selon votre configuration réseau) 
    * Source : 192.168.0.0/24,92.168.10.0/24 # IP autorisé 
    * Destination : 192.168.0.215 # IP de l'hyperviseur 
    * Ports dest : 8006,55215,443 #8006, 22 ou autre port SSH 55215 et 443 pour le https
    * Protocole : TCP 
    * Action : ACCEPT 
    * ![](pve-parefeu01.png){:width="500" .normal}
2. Bloquer tout autre trafic
    * Interface : vmbr0 
    * Action : DROP 
    * Log level : Warning 
    * ![](pve-parefeu02.png){:width="500" .normal}

Cette règle doit être placée après la règle ACCEPT  
![](pve-parefeu03.png){: .normal}  
Une fois l'ordre correctement vérifier, les règles peuvent être activés.  
💡 Le paramètre Log level: Warning est optionnel, il permet de logger les tentatives bloquées. Consultables depuis l'interface Web, onglet "Firewall" > "Log" pour chaque niveau (Datacenter, hôte, VM).

**Activation du pare-feu**  
Par défaut le pare-feu est désactivé au niveau Datacenter. Pour qu'il fonctionne correctement, il faut passer l'option Firewall sur Yes à tous les niveaux concernés :

* Au niveau Datacenter, pour activer le pare-feu globalement sur votre infrastructure. 
* Au niveau de chaque hôte, pour activer le filtrage du trafic réseau sur ce serveur. 
* Au niveau de chaque VM ou conteneur (LXC), pour activer le filtrage spécifique à cette machine. 

![](pve-parefeu04.png){: .normal}

### Gestion distante démarrage et arrêt

*On veut démarrer ou éteindre, depuis le poste linux PC1, la machine distante Lenovo Proxmox (pve.rnmkcy.eu) via ssh et “wake on lan”*

[Démarrer ou éteindre une machine distante sur le réseau via ssh et "wake on lan"](/posts/demarrer_eteindre_une_machine_via_ssh_et_wake_on_lan/)

**Poste linux PC1 (archlinux/EndeavourOS)**  
Installer wake on lan

```shell
sudo apt install wakeonlan # Debian/Ubuntu 
yay -S wakeonlan # archlinux
```

Rechercher adresse Mac de la machine Lenovo Proxmox

```shell
ping -c3 pve.home.arpa && arp -n
```

![](wol01.png)

Lenovo Proxmox : `192.168.0.215            ether   00:23:24:c9:06:86`

* [PC1 - Script au démarrage](/posts/demarrer_eteindre_une_machine_via_ssh_et_wake_on_lan/#pc1---script-au-démarrage)
* [PC1 - Script à l’arrêt](/posts/demarrer_eteindre_une_machine_via_ssh_et_wake_on_lan/#pc1---script-à-larrêt)

### Importer un fichier ISO dans Proxmox

Afin de pouvoir installer les systèmes d’exploitation sur nos différentes machines virtuelles, nous devons au préalable télécharger les images système (ISO) et les importer dans Proxmox.

> Les fichiers ISO sont stockés dans le dossier `/var/lib/vz/template/iso/`
{: .prompt-tip }

Pour cela, Proxmox dispose d’un élément assez sympa je trouve, une sorte de banque où seront stockés toutes vos images. Procédez comme ceci :

Sélectionnez votre nœud puis le stockage (**local**, dans notre cas).  

Cliquez sur "ISO Images", puis sur le bouton "Upload" et recherchez l’image à importer sur votre disque local.  
![](proxmox0002.png){: .normal}  
![](proxmox0003.png){: .normal}  
Répéter l’opération autant de fois que nécessaire selon la quantité d’images ISO à importer, la seule limite c’est votre espace de stockage.  
Les fichiers ISO sont stockés dans le dossier `/var/lib/vz/template/iso/`

### Comment utiliser le disque EFI dans Proxmox

[How to Use EFI Disk in Proxmox?](https://www.vinchin.com/tech-tips/proxmox-efi-disk.html)

Ajouter EFI Disque via la ligne de commande

```shell
qm ensemble 100 --bios ovmf
qm set 100 --efidisk0 local-lvm:1,format=raw
# Remplacer 100 par votre ID VM, et local-lvm par votre nom de stockage
```

### Affecter un USB hôte à l'invité

[Comment activer l’USB Passthrough pour Home Assistant sous Proxmox ?](https://www.domo-blog.fr/comment-activer-usb-passthrough-proxmox-home-assistant-domotique/)

Voie simple: passez l'identifiant du périphérique USB à l'invité

Attribuer une clé Yubikey connectée à l'hôte à VM 103

    lsusb

```
Bus 001 Device 004: ID 1050:0407 Yubico.com Yubikey 4/5 OTP+U2F+CCID
```

Assignez-le à la VM en mode su par

```
qm set 103 -usb0 host=1050:0407
```

Réponse: `update VM 103: -usb0 host=1050:0407`

Eteindre la VM (en cours d'exécution) et relancer

### Sauvegarder et restaurer des machines virtuelles

[Comment sauvegarder et restaurer des machines virtuelles dans Proxmox](https://fr.linux-console.net/?p=29358)



## Machines virtuelles et conteneurs

### Création machine virtuelle

Pour créer une nouvelle machine virtuelle, il faut cliquer sur le bouton "Créer une VM" en haut à droite de l’interface.  
![](proxmox0007.png){: .normal}  
il faut nommer la machine 

Pour un ordonnancement correct, il sera utile de définir une nomenclature de nommage, je vous propose, ceci :

*    VM-[ID de la VM] – OS-Nom de la machine
*    CT-[ID du conteneur] – OS-Nom de la machine

![](proxmox0008.png){:width="500" .normal}  

Le "Resource Pool" ne sera utilisé que si vous avez plusieurs emplacements de stockage sur le Proxmox (pour de l’ordonnancement / backup, etc.).

Sélectionnez à présent l’ISO que vous souhaitez installer sur la machine et le type de système correspondant. Particularité pour une VM freeBSD, on sélectionnera « Other ».  
![](proxmox0009.png){:width="500" .normal}  
Sachez qu'il est aussi possible d’utiliser directement le lecteur de CD/DVD physique, voir une clé USB directement branchée au serveur.   

**Pour les conteneurs**,les templates préconfigurés sont téléchargeables directement via l’interface et il est demandé si vous souhaitez définir un mot de passe dès la création.

Sur l’écran suivant, on peut configurer certains aspects du système, en cochant « Advanced ». Ainsi, il sera possible de modifier le type de Firmware (BIOS ou UEFI), le type de disque (IDE, SCSI, SATA) et l’émulation SSD, le démarrage automatique, le type de CPU, etc.
![](proxmox0010.png){:width="500" .normal}  

L’étape suivante consiste à configurer le stockage de la machine virtuelle, avec le choix du disque dur, son type et sa taille.  
![](proxmox0011.png){:width="500" .normal}    

À présent, il s’agit de définir les spécifications du CPU, avec éventuellement la possibilité de modifier les vCPU (processeurs virtuels).  
![](proxmox0012.png){:width="500" .normal}  

Ensuite, nous définissons la quantité de mémoire RAM allouée à cette VM, il est alors possible d’allouer une quantité maximale et minimale (en **mode avancé**), permettant de limiter la monopolisation des ressources en fonction de l’utilisation de la machine.  
![](proxmox0013.png){:width="500" .normal}  

La partie Network est assez simple en soi sur une VM, on sélectionne l’interface (Bridge) sur laquelle on souhaite avoir la VM et éventuellement le tag du VLAN (VLAN Tag).  
![](proxmox0014.png){:width="500" .normal}  

Nous sommes à la dernière étape où nous avons le droit à un résumé. Si cela vous convient, cliquez sur **"Terminé"** pour créer la machine virtuelle. Cela n'installe pas le système d'exploitation dans la VM, mais la machine sera prête à l'installation.  
![](proxmox0015.png){:width="500" .normal}  
Notre VM est alors créé et nous pouvons à présent la retrouver dans notre node (partie de gauche de l'interface).  
![](proxmox0016.png)  

### Ajout EFI

[How to add an EFI disk in Proxmox?](https://www.vinchin.com/tech-tips/proxmox-efi-disk.html)

[OVMF/UEFI Boot Entries](https://pve.proxmox.com/wiki/OVMF/UEFI_Boot_Entries)

https://www.bgocloud.com/knowledgebase/95/making-your-first-virtual-machine-and-container-in-proxmox-ve.html

### Démarrage VM

Pour pouvoir lancer la VM nouvellement créée, il suffit de faire un clic droit sur l’icône de la machine dans le menu de gauche et de sélectionner "Démarrer"  
L’autre option est de la sélectionner la VM, comme nous venons de le faire, puis de sélectionner **"Démarrer"** en haut à droite de l’écran. D’ailleurs, ce menu comporte un bouton **"Plus"** qui permet de détruire une machine et son stockage associé, c'est-à-dire le disque virtuel. Ce menu permet aussi de créer un Template (c'est-à-dire un modèle de VM), que l'on pourra cloner à souhait (plutôt pratique).  
![](proxmox0017.png){: .normal}  

Une fois la machine démarrée, nous avons accès aux métriques en sélectionnant **"Résumé"** (charge CPU, RAM, espace de stockage, etc.).  
![](proxmox0018.png){: .normal}  

De la même façon, il est possible de suivre l’état de votre hyperviseur en sélectionnant : **"Centre de données"** puis **"Résumé"**.  
![](proxmox0019.png){: .normal}  

Pour accéder à la machine virtuelle ou au conteneur, il suffit de double-cliquer sur son icône dans le menu de gauche. Cette manipulation ouvre une fenêtre qui donne un accès à l’interface graphique de la machine virtuelle.  
![](proxmox0020.png)   
Sur la capture d'écran ci-dessus, on peut voir une flèche sur le côté gauche, celle-ci permet d’avoir accès à des options supplémentaires de la VM, telles que :

*    Mettre en plein écran
*    Activer une combinaison de touche (CTRL+ALT+SUPPR)
*    Démarrage, arrêt, rafraichir l’interface
*    Etc...

Note : sous Linux, si vous utiliser CTRL+W pour une recherche avec l'éditeur "nano", la fenêtre va se fermer sans pour autant éteindre la VM, il faut alors sélectionner [A], puis [CTRL], puis après votre touche [W] afin de permettre la recherche. Une fois le mot trouvé, désactivez la fonctionnalité.
{: .prompt-info }

![](proxmox0020a.png)   

Il ne reste plus qu'à procéder à l'installation du système d'exploitation, que ce soit du Linux ou du Windows !

### Erreurs

*VM quit/powerdown failed - got timeout*

**Erreur de délai d'attente**  

L'erreur de délai d'attente se produit lorsque la machine virtuelle est verrouillée ou que le processus est toujours en cours d'exécution en arrière-plan.

Si la machine virtuelle est verrouillée, nous déverrouillons la VM et l'arrêtons. 
Sinon, nous nous connectons au nœud hôte.  
Ensuite, nous trouvons le PID du processus de la machine en utilisant la commande.

    ps aux | grep "/usr/bin/kvm -id VMID"
    ps aux | grep "/usr/bin/kvm -id 100" # VMID=100 dans notre exemple

```
root       15847  0.5  0.6 3779756 49520 ?       Sl   09:58   0:10 /usr/bin/kvm -id 100 -name vm-001-debian11-yunotest -no-shutdown -chardev socket,id=qmp.......................
```

Une fois que nous avons trouvé le PID, nous tuons le processus en utilisant la commande.

    kill -9 15847  # PID=15847 dans notre exemple

**Arrêt en mode CLI**  
Nous avons rencontré de nombreuses instances qui arrêtent la machine virtuelle à partir de l'interface web pour effectuer les tâches.

Ainsi, nous arrêtons la machine virtuelle à partir du nœud. Pour arrêter la VM, il faut d'abord trouver le VMID.

Nous utilisons donc la commande suivante pour arrêter la machine virtuelle

    qm stop <VMID>

Ensuite, en rafraîchissant l'interface web, nous pouvons voir que la machine virtuelle est arrêtée.

**VM verrouillée**  
L'une des raisons les plus courantes pour lesquelles il est impossible d'arrêter une machine virtuelle est que celle-ci a été verrouillée. Cela se produit généralement lorsque nous essayons d'arrêter une machine virtuelle alors qu'une sauvegarde est en cours.

La machine virtuelle se verrouille alors pour terminer le processus de sauvegarde. Ainsi, nous pouvons attendre que le processus de sauvegarde supprime la VM. Sinon, nous pouvons déverrouiller la machine virtuelle et l'arrêter.

Pour déverrouiller la VM, nous nous connectons au nœud hôte.

Puis nous trouvons le VMID de la machine virtuelle en utilisant la commande

    cat /etc/pve/.vmlist

Une fois que nous avons obtenu le VMID, nous déverrouillons la VM à l'aide de la commande

    qm unlock <VMID>

Après avoir déverrouillé la machine virtuelle, nous pouvons supprimer la machine virtuelle à partir de l'interface web ou en utilisant le CLI.

## Création Conteneur

### Images des conteneurs

Les images de conteneurs, parfois également appelées -templates ou -Appliances, sont des archives tar qui contiennent tout pour exécuter un conteneur.

Proxmox VE lui-même fournit une variété de modèles de base pour [les distributions Linux les plus courantes](https://pve.proxmox.com/wiki/Linux_Container#pct_supported_distributions). Ils peuvent être téléchargés en utilisant l'utilitaire de ligne de commande GUI ou pveam (short for Proxmox VE Appliance Manager). En outre, les modèles de conteneur [TurnKey Linux](https://www.turnkeylinux.org/) sont également disponibles à télécharger.

La liste des modèles disponibles est mise à jour quotidiennement par l'intermédiaire de `pve-daily-update`.  
Vous pouvez également déclencher une mise à jour manuellement en exécutant en mode su :

```shell
pveam update
```

Pour voir la liste des images disponibles tourner:

```shell
pveam available
```

Vous pouvez restreindre cette grande liste en spécifiant la section qui vous intéresse, par exemple les images système de base  

```shell
pveam available --section system
```

Liste des images système disponibles

```
system          almalinux-9-default_20240911_amd64.tar.xz
system          alpine-3.19-default_20240207_amd64.tar.xz
system          alpine-3.20-default_20240908_amd64.tar.xz
system          alpine-3.21-default_20241217_amd64.tar.xz
system          archlinux-base_20240911-1_amd64.tar.zst
system          centos-9-stream-default_20240828_amd64.tar.xz
system          debian-11-standard_11.7-1_amd64.tar.zst
system          debian-12-standard_12.7-1_amd64.tar.zst
system          devuan-5.0-standard_5.0_amd64.tar.gz
system          fedora-41-default_20241118_amd64.tar.xz
system          fedora-42-default_20250428_amd64.tar.xz
system          gentoo-current-openrc_20250508_amd64.tar.xz
system          openeuler-24.03-default_20250507_amd64.tar.xz
system          openeuler-25.03-default_20250507_amd64.tar.xz
system          opensuse-15.6-default_20240910_amd64.tar.xz
system          rockylinux-9-default_20240912_amd64.tar.xz
system          ubuntu-20.04-standard_20.04-1_amd64.tar.gz
system          ubuntu-22.04-standard_22.04-1_amd64.tar.zst
system          ubuntu-24.04-standard_24.04-2_amd64.tar.zst
system          ubuntu-24.10-standard_24.10-1_amd64.tar.zst
system          ubuntu-25.04-standard_25.04-1.1_amd64.tar.zst
```

Au niveau graphique  
![](ct003.png){: .normal}  

![](ct004.png){:width="400" .normal}  

![](ct005.png){: .normal}  

### Alpine Linux

![](alpine-linux-logo.png){:width="250"}  

*Alpine Linux est une distribution Linux légère et sécurisée, conçue pour être utilisée dans des environnements exigeants en termes de performance et de sécurité. Elle est basée sur le noyau Linux et utilise le gestionnaire de paquets “apk” pour gérer les applications et les bibliothèques.*

Alpine Linux est particulièrement adaptée aux environnements de virtualisation, de conteneurisation et de cloud, grâce à sa taille réduite et à sa consommation faible en ressources. Elle est également utilisée dans de nombreux autres contextes, tels que les serveurs Web, les routeurs et les appareils embarqués.


*    Taille réduite : Alpine Linux est une distribution très légère, avec une image ISO de seulement 5 Mo, ce qui la rend idéale pour les environnements où l’espace disque est limité.
*    Consommation faible en ressources : Alpine Linux utilise peu de mémoire et de CPU, ce qui en fait une excellente option pour les environnements de virtualisation et de conteneurisation où les ressources sont partagées.
*    Temps de démarrage rapide : grâce à sa taille réduite et à sa consommation faible en ressources, Alpine Linux démarre rapidement et est prêt à être utilisé en quelques secondes.
*    Faible empreinte en termes de sécurité : Alpine Linux est conçue pour être sécurisée dès le départ, avec une base de code réduite et des paquets verrouillés par défaut. Cela réduit le risque de vulnérabilités et de menaces de sécurité.


Créer un jeu de clès SSH sur le poste PC1

```shell
ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/ct-alp01
```

La clé publique est disponible sous le dossier `~/.ssh/ct-alp01.pub` 

Les réglages généraux d'un conteneur incluent

* le nœud : le serveur physique sur lequel le conteneur va fonctionner
* ID CT: un numéro unique dans cette installation Proxmox VE utilisée pour identifier votre conteneur
* Nom d'hôte : le nom d'hôte du conteneur
* Resource Pool: un groupe logique de conteneurs et de VMs
* Mot de passe : le mot de passe "root" du conteneur
* SSH Public Key: une clé publique pour se connecter au compte racine sur SSH

Conteneur non privilégié : cette option permet de choisir au moment de la création si vous voulez créer un conteneur privilégié ou non privilégié.

Se connecter à l’interface de gestion de Proxmox VE  
Créer un conteneur  
![](ct001.png){: .normal}  

![](ct002.png){:width="400" .normal}  
renseigner **Nom d'hôte**, **Mot de passe** root (root49600), **Charger le fichier de clef SSH** (ct-alp01.pub)

![](ct002a.png){:width="400" .normal}  

![](ct002b.png){:width="400" .normal}  

![](ct002c.png){:width="400" .normal}  

![](ct002d.png){:width="400" .normal}  

![](ct002e.png){:width="400" .normal}  

![](ct002f.png){:width="400" .normal}  

![](ct002g.png){:width="400" .normal}  

![](ct006.png){: .normal} 

Vérification en mode console  
![](ct007.png){: .normal} 

En mode console

Mise à jour

```shell
apk -U upgrade
```

Créer un utilisateur

```shell
setup-user
```

Installer openssh

```shell
apk add openssh
```

Activer le service et lancer

```shell
rc-update add sshd
service sshd start
```

Connexion SSH

```shell
ssh -i ~/.ssh/ct-alp01 root@192.168.0.231
```

### Cockpit

*Cockpit est une interface graphique Web pour la gestion des serveurs Linux. Il permet aux utilisateurs d'effectuer des tâches telles que configurer les réseaux, gérer le stockage et surveiller les performances du système directement via un navigateur Web. Il s'intègre aux outils système existants, le rendant adapté à la fois aux débutants et aux administrateurs expérimentés.*

<https://community-scripts.github.io/ProxmoxVE/scripts?id=cockpit>  
Pour créer un nouveau Proxmox VE Cockpit LXC, lancez la commande ci-dessous dans Proxmox VE Shell.

```shell
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/cockpit.sh)"
```

Fin installation

![](cockpit-lxc.png)  

![](cockpit-lxc01.png)  


Location fichier de configuration : `/etc/cockpit/cockpit.conf`

