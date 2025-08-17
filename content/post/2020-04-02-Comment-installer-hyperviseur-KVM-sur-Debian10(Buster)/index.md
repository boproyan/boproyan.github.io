+++
title = 'Comment installer l'hyperviseur KVM sur Debian 10 (Buster)'
date = 2020-04-02 00:00:00 +0100
categories = ['virtuel']
+++
Comment installer le serveur de virtualisation d'hyperviseur KVM sur Debian 10 (Buster). KVM (Kernel-based Virtual Machine) est une solution de virtualisation complète open source pour les systèmes Linux fonctionnant sur du matériel x86 avec des extensions de virtualisation (Intel VT ou AMD-V).


# KVM/QEMU (virtualisation linux) sur un serveur debian buster

* [How To Install KVM Hypervisor on Debian 10 (Buster)](https://computingforgeeks.com/how-to-install-kvm-virtualization-on-debian/)
* [Installing KVM on Debian 10](https://linuxhint.com/install_kvm_debian_10/)
* [VirtualNetworking (libvirt virsh)](https://wiki.libvirt.org/page/VirtualNetworking)
* [Qemu (archlinux)](https://wiki.archlinux.fr/Qemu)
* [La Gestion réseau dans une machine virtuelle](https://chrtophe.developpez.com/tutoriels/gestion-reseau-machine-virtuelle/#L4-2-2)

## KVM supporté par le CPU ?

Exécutez la commande *egrep* suivante pour vérifier que **Intel VMX** ou **AMD SVM** est supporté sur votre CPU 

    egrep --color 'vmx|svm' /proc/cpuinfo

vmx (Intel) ou svm (Amd) doit apparaître d'une autre couleur dans le résultat 


## Installer KVM/QEMU sur le serveur

![KVM](kvm-logo.png){:width="80"} ![Qemu](qemulogo.png)  

On utilise ssh pour se connecter au serveur  
Installation, exécuter la commande suivante

    sudo apt install qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils libguestfs-tools genisoimage virtinst libosinfo-bin

Chargez et activez le module vhost_net

    sudo modprobe vhost_net
    lsmod | grep vhost

```
vhost_net              24576  0
tun                    49152  1 vhost_net
vhost                  49152  1 vhost_net
tap                    28672  1 vhost_net
```

Vérifier si le service **libvirtd** est lancé et activé (enabled)

    sudo systemctl status libvirtd

```
● libvirtd.service - Virtualization daemon
   Loaded: loaded (/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-12-12 11:28:02 CET; 2 days ago
     Docs: man:libvirtd(8)
           https://libvirt.org
 Main PID: 759 (libvirtd)
    Tasks: 17 (limit: 32768)
   Memory: 34.7M
   CGroup: /system.slice/libvirtd.service
           └─759 /usr/sbin/libvirtd

déc. 12 11:28:00 xoyize.xyz systemd[1]: Starting Virtualization daemon...
déc. 12 11:28:02 xoyize.xyz systemd[1]: Started Virtualization daemon.
```


### Ajout utilisateur au groupe libvirt

Si vous voulez que l'utilisateur normal/régulier puisse gérer les machines virtuelles. Ajouter l'utilisateur $USER à libvirt et libvirt-qemu en utilisant la commande *usermod*

    sudo adduser $USER libvirt
    sudo adduser $USER libvirt-qemu

Recharger l'adhésion à un groupe avec l'aide de la commande *newgrp*

    newgrp libvirt
    newgrp libvirt-qemu

Vérifiez votre appartenance à un groupe à l'aide de la commande *id*

    $ id

```
gid=64055(libvirt-qemu) groupes=64055(libvirt-qemu),118(libvirt)
```

Veuillez noter que vous devez utiliser une des commandes suivantes pour vous connecter au serveur KVM

    virsh --connect qemu:///system

```
Welcome to virsh, the virtualization interactive terminal.

Type:  'help' for help with commands
       'quit' to quit

virsh # 
```

    virsh --connect qemu:///system list --all  # avec la commande list par exemple


