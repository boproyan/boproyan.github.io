+++
title = 'cwwk KVM - Serveur Debian 12 (collabora online)'
date = 2025-01-13 00:00:00 +0100
categories = ['virtuel']
+++
*Comment installer Debian 12 sur un serveur KVM grâce à l'utilisation de l'une des images cloud officielles de Debian* 

## Debian cloud images

![debian-cloud](debian-cloud.png){:height="50"}  
*Si vous exploitez un nuage privé ou une plateforme de virtualisation fonctionnant avec KVM, comme OpenStack et oVirt. La manière la plus idéale de faire tourner une machine virtuelle Debian 12 est d'utiliser une image de nuage. Dans ce blog, nous vous montrons comment télécharger l'image officielle du nuage Debian 12 et créer une instance de machine virtuelle à partir de celle-ci sur l'hyperviseur KVM.* 

* **generic** : Doit fonctionner dans n'importe quel environnement
* **genericcloud** : devrait fonctionner dans n'importe quel environnement virtualisé. Il est plus petit que generic car il exclut les pilotes pour le matériel physique.
* **nocloud** : Principalement utile pour tester le processus de construction lui-même. N'a pas installé cloud-init, mais permet à l'utilisateur de se connecter en tant que root sans mot de passe.

### Prérequis

Les packages cloud-utils et whois 

    sudo apt update && sudo apt install cloud-utils whois -y

**cloud-utils** est nécessaire pour exécuter la commande **cloud-localds** , pour créer l'image ISO cloud init ultérieurement et valider la configuration cloud-init.  
**whois** pour exécuter la commande **mkpasswd** ultérieurement si nous voulons l'utiliser pour générer la version hachée de notre mot de passe pour la configuration **cloud-init**.

L'extension php8.x-yaml donne accès au standard de sérialisation **YAML Ain't Markup Language (YAML)**.  
L'analyse syntaxique et la production de données sont effectuées par la bibliothèque » LibYAML. 

    sudo apt install php8.3-yaml

Le paquet **libguestfs-tools**, qui contient l'utilitaire **virt-sysprep**

    sudo apt install libguestfs-tools

### Télécharger Image cloud debian

Télécharger l'image **debian bookworm generic** dans le dossier `/srv/kvm/libvirt/images/` en la renommant debian-12.qcow2

```shell
wget https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2 -O /srv/kvm/libvirt/images/debian-12.qcow2
```

### Choisir un nom

Tout d'abord, choisissez un nom pour votre nouveau VPS. Je crée toujours un invité « modèle », à partir duquel je le clone ensuite pour créer les autres invités

VM_NAME="vm-debian12"

### Augmenter la taille du disque

L'image qcow2 que nous avons extraite était destinée à une très petite machine virtuelle qui manquera rapidement d'espace si vous commencez à l'utiliser pour quoi que ce soit. Je recommanderais d'augmenter la taille du disque à la taille souhaitée pour la machine virtuelle. Dans ce cas, je règle 20 Go.

```
DESIRED_SIZE=8G

sudo qemu-img resize \
  debian-12.qcow2 \
  $DESIRED_SIZE
```

### Créer un fichier de configuration cloud

J'ai découvert que beaucoup de gens avaient des fichiers cloud-config et qu'il était frustrant d'essayer de les mélanger/associer et d'avoir des problèmes avec le hachage de mon mot de passe. Je n'aimais pas non plus devoir faire attention à ne pas casser le formatage/l'indentation YAML requis, etc. Au final, j'ai créé un script PHP que je pouvais facilement modifier afin de générer mon fichier cloud-config.cfg en utilisant quelques paramètres simples que j'ai définis en haut.

Téléchargez le script 

    wget https://files.programster.org/tutorials/cloud-init/create-debian-12-kvm-guest-from-cloud-image/generate.php -O generate-cloud-cfg.php

Remplissez les paramètres en haut du script `generate-cloud-cfg.php`

```php
<?php

# Fill in these settings as appropriate to you.

# Set the hostname for the guest.
$hostname = "vm-debian12";

# Specify your username and password
$username = "userone";
$password = "xxxxx";

# Set a random salt string here for password hashing
$randomSaltString = '-WMCNJgVCyDdKqwFHG_.';

# Specify the public keys of the private-keys you wish to login with.
$sshPublicKeys = [
    'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMUoTI1oGwsS69C1V2pXh7WNsrOTnExil8kyFBjX4oVv yann@PC1',
];

##### End of settings. Do not edit below this line #######
##########################################################
```

Exécutez ensuite le script pour générer votre **cloud-init.cfg** 

    php generate-cloud-cfg.php

Ce script nécessite que vous ayez installé l'extension PHP YAML .

J'ai vu quelques exemples de fichiers qui contenaient autoinstall. Cela semble être lié à l'installation automatisée et peut être spécifique à Ubuntu. Quoi qu'il en soit, cloud-init ignore simplement cette clé et nous n'en avons de toute façon pas besoin.

Le tableau des utilisateurs contient souvent « par défaut » en plus dans les exemples que j'ai vus, mais cela génère un utilisateur debian supplémentaire dont je n'ai pas besoin ou que je ne souhaite pas avoir.

Vous pouvez utiliser cloud-init schema --config-file cloud-init.cfg le fichier de configuration d'initialisation cloud que vous avez généré pour valider.

### Créer le fichier ISO d'initialisation du cloud

Créez le fichier ISO à partir du fichier de configuration cloud que nous venons de créer :

```bash
sudo cloud-localds \
  cloud-init.iso \
  cloud-init.cfg
```

Créer l'invité

Vous pouvez maintenant enfin exécuter la commande pour créer l’invité :

```
sudo virt-install \
  --name vm-debian12 \
  --memory 1024 \
  --disk debian-12.qcow2,device=disk,bus=virtio \
  --disk cloud-init.iso,device=cdrom \
  --os-variant debiantesting \
  --virt-type kvm \
  --graphics none \
  --network bridge=br0 \
  --import
```

Après avoir exécuté la commande précédente, vous serez redirigé vers l'écran de connexion de la console, à partir duquel vous pourrez vous connecter avec le nom d'utilisateur  
![](cloud-init-debian.png)  

*on a un accès depuis la console par saisie utilisateur/Mot de passe* 

### Connexion console

Se connecter avec le nom d'utilisateur  
Relever adresse IP : 192.168.10.153

Définir un mot de passe pour l'utilisateur root.

```
sudo -s
passwd root
```

### Sortie Console 

Si vous devez sortir de la console, appuyez simplement sur `ctrl+ AltGr + ]` 

### Nettoyage

faire un nettoyage afin que les mots de passe bruts ne traînent pas.

    rm /srv/kvm/libvirt/images/cloud-init.*iso cloud-init.cfg*

## KVM debian virtuel vm-debian01 

### Réseau (netplan)

*netplan est utilisé comme gestionnaire réseau avec l'image debian cloud*

Pour désactiver les capacités de configuration réseau de cloud-init, écrivez un fichier `/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg` 

    echo "network : {config : disabled}" > /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg

Les périphériques du réseau

    ip link show

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:21:74:3b brd ff:ff:ff:ff:ff:ff
```

*[Netplan](https://netplan.io/) est un utilitaire qui permet de configurer facilement le réseau sous Linux.*

* [Gestion du réseau Linux avec Netplan](https://linux.goffinet.org/administration/configuration-du-reseau/gestion-du-reseau-linux-avec-netplan/)
* [How to configure IPv6 with Netplan](https://www.snel.com/support/how-to-configure-ipv6-with-netplan-on-ubuntu-18-04/)

Les fichiers de configuration sont dans le dossier `/etc/netplan`

```
90-default.yaml
```

On retire les fichiers de configuration originaux 

```
mkdir /etc/backup.netplan
mv /etc/netplan/* /etc/backup.netplan/
```

On propose cette configuration statique de l’interface **enp1s0**

```
bash -c 'cat << EOF > /etc/netplan/01-enp1s0.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      addresses:
        - 192.168.100.40/24
      nameservers:
        addresses:
        - 1.1.1.1
        - 9.9.9.9
        search: []
      routes:
      - to: default
        via: 192.168.100.1
EOF'
```

Droits

    chmod 600 /etc/netplan/01-enp1s0.yaml

Et on génère la configuration pour l’appliquer auprès du gestionnaire 

```
netplan generate
netplan apply
```

`Cannot call openvswitch: ovsdb-server.service is not running`  
Il s'agit juste d'un avertissement...

### Connexion ssh vm-debian12

Se connecter ssh sur l'adresse ip 192.168.100.40 avec la clé 

```bash
ssh -o ProxyCommand="ssh -W %h:%p -p 55205 -i /home/yann/.ssh/yick-ed25519 yick@192.168.0.205" userone@192.168.100.40 -p 22 -i /home/yann/.ssh/vm-debian01
```

## Débogage Astuces

### Modification des paramètres réseau

Modifier les paramètres réseau pour supprimer la restriction à une certaine adresse MAC statique, ce qui empêche le réseau de fonctionner sur les invités clonés, car ils obtiennent une nouvelle adresse MAC différente. Cela peut être fait en modifiant le fichier netplan à l'adresse :

    /etc/netplan/50-cloud-init.yaml

### Les clones obtiennent la même adresse IP DHCP

Debian 12 utilise désormais le concept d'identifiant de machine pour s'identifier auprès de votre serveur DHCP. Cela signifie que si vous créez un clone et ne réinitialisez pas l'identifiant de la machine, les deux invités s'identifieront de la même manière et recevront la même adresse IP du serveur DHCP. Pour résoudre ce problème, réinitialisez l'identifiant de votre machine lorsque vous créez un clone :

    sudo truncate -s 0 /etc/machine-id

Redémarrez ensuite. Votre serveur générera un nouvel ID de machine de remplacement au prochain démarrage.

### Nom d'hôte et fichier d'hôtes

Il semble que la définition du nom d'hôte dans la configuration cloud-init définira le nom d'hôte sous lequel la machine démarrera. Cependant, il se peut que l'entrée ne soit pas ajoutée au fichier /etc/hosts, vous souhaiterez donc peut-être le faire rapidement pour éviter que votre machine ne prenne inutilement du temps lors de l'exécution de certaines opérations. Vous pouvez également découvrir comment mettre à jour le fichier cloud-init.cfg pour mettre à jour correctement le fichier hosts à l'aide du module Etc hosts .
SSH ne fonctionne pas

Si vous constatez qu'Openssh ne fonctionne pas, vous devrez peut-être exécuter la commande suivante pour que votre serveur génère sa clé d'hôte afin que vous puissiez vous connecter au serveur via SSH. Plus d'informations ici .

    sudo dpkg-reconfigure openssh-server

Je vous suggère également de vérifier que vous êtes satisfait des paramètres de votre fichier /etc/ssh/sshd_config.

### Définir le fuseau horaire

Une autre chose que vous souhaiterez peut-être faire est de définir le fuseau horaire sur Europe/Paris

    sudo dpkg-reconfigure tzdata

```
Current default time zone: 'Europe/Paris'
Local time is now:      Thu Apr 10 18:08:48 CEST 2025.
Universal Time is now:  Thu Apr 10 16:08:48 UTC 2025.
```

## Collabora online

Installation et configuration : [Collabora](/posts/Collabora_Debian/)