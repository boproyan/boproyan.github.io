+++
title = 'Création rapide machine virtuelle KVM debian 12 nocloud'
date = 2024-03-14 00:00:00 +0100
categories = virtuel debian
+++
*La machine virtuelle basée sur le noyau (KVM) est un logiciel que vous pouvez installer sur des machines Linux physiques pour créer des machines virtuelles. Une machine virtuelle est une application logicielle qui agit comme un ordinateur indépendant au sein d'un autre ordinateur physique. *


## Serveur virtuel

### Debian 12 nocloud

*Modifier les paramètres puis exécuter et lancer l'installation ensuite*

Paramètres de la machine virtuelle

```shell
# Dossier contenant les images
export IMAGES="/srv/kvm/libvirt/images"
# Nom de la VM
export VM_NAME="debian-12-nocloud-amd64"
# disque , ex : 40G
export VM_ROOT_DISK_SIZE=10G
# Mémoire , ex: 1024, 2048, 4096
export MEMORY=2048
```

Lancer le bash suivant

```shell
# Télécharger dernière image nocloud debian12 amd64
wget -O $IMAGES/$VM_NAME.qcow2 https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-nocloud-amd64.qcow2

# Resize Debian 
qemu-img resize \
  $IMAGES/$VM_NAME.qcow2 \
  $VM_ROOT_DISK_SIZE

# virt-install
sudo virt-install \
    --memory $MEMORY \
    --vcpus 1 \
    --name $VM_NAME \
    --disk $IMAGES/$VM_NAME.qcow2,device=disk,bus=virtio,format=qcow2 \
    --os-variant debiantesting \
    --network bridge=br0 \
    --virt-type kvm \
    --graphics none \
    --import
```

Login root sans mot de passe

### OpenSSH

Par défaut openssh ne fonctionne pas

```
[FAILED] Failed to start ssh.servic…[0m - OpenBSD Secure Shell server.
```

Il faut regénérer les clés

    dpkg-reconfigure openssh-server

![](ssh-secure.png){:width=500}

Il faut activer authentification mot de passe

    nano /etc/ssh/sshd_config

```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication yes
#PermitEmptyPasswords no
```

Relancer

    systemctl restart sshd

### Créer mot de passe root

    pssswd

### Créer un utilisateur

Après s'être connecté en tant qu'utilisateur root, créer le premier utilisateur (mp uservm49) 

    useradd -m -d /home/userauth -s /bin/bash -c "GLAuth" -U userauth

Mot de passe uservm: uservm49

    passwd userauth

Ajout à sudoers

    echo "userauth     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/userauth

### Réseau statique

*netplan est utilisé comme gestionnaire réseau avec l'image debian cloud*

On retire les fichiers de configuration originaux 

    rm /etc/netplan/*

configuration statique de l’interface **enp1s0**

```
bash -c 'cat << EOF > /etc/netplan/01-enp1s0.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      addresses:
        - 192.168.0.218/24
      nameservers:
        addresses:
        - 1.1.1.1
        - 9.9.9.9
        search: []
      routes:
      - to: default
        via: 192.168.0.254
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

Pour vérification 

    ip a |grep "inet "

```
    inet 127.0.0.1/8 scope host lo
    inet 192.168.0.218/24 brd 192.168.0.255 scope global enp1s0
```

### OpenSSH + Clés

<u>sur l'ordinateur de bureau</u>  
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec le serveur.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/vm-glauth

Envoyer les clés publiques sur le serveur KVM   

    ssh-copy-id -i ~/.ssh/vm-glauth.pub userauth@192.168.0.218

<u>sur le serveur KVM</u>  
Modifier la configuration serveur SSH  

    nano /etc/ssh/sshd_config

Modifier

```conf
Port = 55218
PasswordAuthentication no
```

Relancer le serveur

    systemctl restart sshd

Test connexion depuis poste sur réseau

    ssh -p 55218 -i ~/.ssh/vm-glauth userauth@192.168.0.218

La partition n'a pas sa taille maximun

    fdisk /dev/vda

### Ajuster taille disque

Exécuter le commandes suivantes sous fdisk

```
d "ENTREE"
1 "ENTREE"
n "ENTREE"
1 "ENTREE"
"ENTREE"
"ENTREE"
w "ENTREE"
```

### Arrêt puis activation VM

On et en mode console, il faut stopper la vm

    systemctl poweroff

Activation au démarrage et lancement VM

```shell
sudo virsh autostart debian-12-nocloud-amd64
sudo virsh start debian-12-nocloud-amd64
```
