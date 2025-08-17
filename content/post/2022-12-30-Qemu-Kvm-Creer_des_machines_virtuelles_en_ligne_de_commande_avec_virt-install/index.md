+++
title = 'QEMU/KVM - Créer des machines virtuelles en ligne de commande avec virt-install'
date = 2022-12-30 00:00:00 +0100
categories = ['kvm']
+++
Vous pouvez utiliser la commande `virt-install` pour créer des machines virtuelles et installer le système d'exploitation sur ces machines virtuelles à partir de la ligne de commande.  
`virt-install` peut être utilisé de manière interactive ou dans le cadre d'un script pour automatiser la création de machines virtuelles.  
Si vous utilisez une installation graphique interactive, vous devez avoir installé `virt-viewer` avant d'exécuter virt-install. En outre, vous pouvez lancer une installation sans surveillance de systèmes d'exploitation de machines virtuelles en utilisant virt-install avec des fichiers kickstart. 

![Qemu](Qemu_logo_blanc.png){:height="40"}  ![KVM](kvm-logo-blanc.png){:height="40"}

- [virt-install](#virt-install)
    - [Options](#options)
        - [Stockage de l'invité](#stockage-de-linvité)
        - [Méthode d'installation des invités](#méthode-dinstallation-des-invités)
    - [Installer une machine virtuelle](#installer-une-machine-virtuelle)
        - [A partir d'une image ISO](#a-partir-dune-image-iso)
        - [Importation d'une image de machine virtuelle](#importation-dune-image-de-machine-virtuelle)
        - [A partir du réseau](#a-partir-du-réseau)
        - [A l'aide de PXE](#a-laide-de-pxe)
        - [Avec Kickstart](#avec-kickstart)
    - [Configurer le réseau de la machine virtuelle invitée pendant la création de l'invité](#configurer-le-réseau-de-la-machine-virtuelle-invitée-pendant-la-création-de-linvité)
        - [Réseau par défaut avec NAT](#réseau-par-défaut-avec-nat)
        - [Réseau ponté avec DHCP](#réseau-ponté-avec-dhcp)
        - [Réseau ponté avec une adresse IP statique](#réseau-ponté-avec-une-adresse-ip-statique)
        - [Aucun réseau](#aucun-réseau)


Remarque : Il se peut que vous ayez besoin des privilèges de l'utilisateur **root** pour que certaines commandes virt-install puissent être exécutées avec succès. 
{: .prompt-info }

## virt-install

### Options

L'utilitaire `virt-instal`l utilise un certain nombre d'options de ligne de commande. Cependant, la plupart des options de virt-install ne sont pas nécessaires. 
Les principales options requises pour les installations de machines invitées virtuelles sont : 

```
--name      
    Le nom de la machine virtuelle.  
--memory
    La quantité de mémoire (RAM) à allouer à l'invité, en MiB. 
```

#### Stockage de l'invité

Utilisez l'une des options de stockage invité suivantes : 

```
--disque 
    Les détails de la configuration du stockage pour la machine virtuelle.
    Si vous utilisez l'option  --disk none , la machine virtuelle est créée sans espace disque 
--filesystem
    Le chemin d'accès au système de fichiers pour la machine virtuelle invitée. 
```

#### Méthode d'installation des invités

Utilisez l'une des méthodes d'installation suivantes : 

```
--location
    L'emplacement du support d'installation. 
--cdrom 
    Le fichier ou le périphérique utilisé comme périphérique CD-ROM virtuel.
    Il peut s'agir du chemin d'accès à une image ISO ou d'une URL permettant de récupérer ou d'accéder à une image ISO de démarrage minimal.
    Toutefois, il ne peut s'agir d'un périphérique CD-ROM ou DVD-ROM hôte physique. 
--pxe 
    Utilise le protocole de démarrage PXE pour charger le disque RAM et le noyau initial
    afin de démarrer le processus d'installation de l'invité. 
--import 
    Saute le processus d'installation du système d'exploitation et construit un invité autour d'une image disque existante.
    Le périphérique utilisé pour le démarrage est le premier périphérique spécifié par l'option disk ou filesystem. 
--boot 
    La configuration de démarrage de la VM après l'installation. 
    Cette option permet de spécifier l'ordre des périphériques de démarrage, de démarrer de façon permanente à partir du noyau et de l'initrd avec des arguments facultatifs pour le noyau et d'activer un menu de démarrage BIOS. 
```

Pour voir une liste complète des options, entrez la commande suivante

    virt-install --help

Pour voir une liste complète des attributs d'une option, entrez la commande suivante : 

    virt install --option= ?

La page de manuel virt-install documente également chaque option de commande, les variables importantes et les exemples. 
Avant d'exécuter virt-install, vous devrez peut-être aussi utiliser **qemu-img** pour configurer les options de stockage. Pour obtenir des instructions sur l'utilisation de qemu-img, consultez le [Chapitre 14, Utilisation de qemu-img (en)](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/virtualization_deployment_and_administration_guide/index#chap-Using_qemu_img). 

### Installer une machine virtuelle 

#### A partir d'une image ISO

L'exemple suivant installe une machine virtuelle à partir d'une image ISO : 

```
virt-install \ 
  --name guest1-rhel7 \ 
  --memory 2048 \ 
  --vcpus 2 \ 
  --disk size=8 \ 
  --cdrom /path/to/rhel7.iso \ 
  --os-variant rhel7 
```

L'option `--cdrom /path/to/rhel7.iso` indique que la machine virtuelle sera installée à partir de l'image CD ou DVD à l'emplacement spécifié. 

#### Importation d'une image de machine virtuelle

L'exemple suivant importe une machine virtuelle à partir d'une image de disque virtuel : 

```
virt-install \ 
  --name guest1-rhel7 \ 
  --memory 2048 \ 
  --vcpus 2 \ 
  --disk /path/to/imported/disk.qcow2 \ 
  --import \ 
  --os-variant rhel7 
```

L'option --import indique que la machine virtuelle sera importée à partir de l'image de disque virtuel spécifiée par l'option `--disk /path/to/imported/disk.qcow2`. 

#### A partir du réseau

L'exemple suivant installe une machine virtuelle à partir d'un emplacement réseau : 

```
virt-install \ 
  --name guest1-rhel7 \ 
  --memory 2048 \ 
  --vcpus 2 \ 
  --disk size=8 \ 
  --location http://example.com/path/to/os \ 
  --os-variant rhel7 
```

L'option `--location http://example.com/path/to/os` indique que l'arbre d'installation se trouve à l'emplacement réseau spécifié. 

#### A l'aide de PXE

Lors de l'installation d'une machine virtuelle à l'aide du protocole de démarrage PXE, l'option --network spécifiant un réseau ponté et l'option --pxe doivent être spécifiées. 
L'exemple suivant installe une machine virtuelle à l'aide de PXE : 

```
virt-install \ 
  --name guest1-rhel7 \ 
  --memory 2048 \ 
  --vcpus 2 \ 
  --disk size=8 \ 
  --network=bridge:br0 \ 
  --pxe \ 
  --os-variant rhel7 
```

#### Avec Kickstart

L'exemple suivant installe une machine virtuelle à l'aide d'un fichier kickstart : 

```
virt-install \ 
  --name guest1-rhel7 \ 
  --memory 2048 \ 
  --vcpus 2 \ 
  --disk size=8 \ 
  --location http://example.com/path/to/os \ 
  --os-variant rhel7 \
  --initrd-inject /path/to/ks.cfg \ 
  --extra-args="ks=file:/ks.cfg console=tty0 console=ttyS0,115200n8" 
```

Les options `initrd-inject` et `extra-args` spécifient que la machine virtuelle sera installée à l'aide d'un fichier Kickstarter. 

### Configurer le réseau de la machine virtuelle invitée pendant la création de l'invité

Lors de la création d'une machine virtuelle invitée, vous pouvez spécifier et configurer le réseau pour la machine virtuelle.  
Cette section fournit les options pour chacun des principaux types de réseau de la machine virtuelle invitée. 

#### Réseau par défaut avec NAT

Le réseau par défaut utilise le commutateur de réseau virtuel NAT (Network Address Translation) de libvirtd. Pour plus d'informations sur la NAT, consultez la [Section 6.1, "Traduction d'adresse réseau (NAT) avec libvirt" (en)](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/virtualization_deployment_and_administration_guide/index#sect-Network_configuration-Network_Address_Translation_NAT_with_libvirt). 

Avant de créer une machine virtuelle invitée avec le réseau par défaut NAT, assurez-vous que le paquet `libvirt-daemon-config-network` est installé. 

Pour configurer un réseau NAT pour la machine virtuelle invitée, utilisez l'option suivante pour virt-install : 

    --network default

Remarque
Si aucune option de réseau n'est spécifiée, la machine virtuelle invitée est configurée avec un réseau par défaut avec NAT.
{: .prompt-info }

#### Réseau ponté avec DHCP

Lorsqu'il est configuré pour un réseau ponté, l'invité utilise un serveur DHCP externe.  
Cette option doit être utilisée si l'hôte a une configuration réseau statique et que l'invité a besoin d'une connectivité entrante et sortante complète avec le réseau local (LAN).  
Elle doit être utilisée si une migration en direct sera effectuée avec la machine virtuelle invitée.

Pour configurer un réseau ponté avec DHCP pour la machine virtuelle invitée, utilisez l'option suivante : 

    --network br0

Remarque
Le pont doit être créé séparément, avant d'exécuter virt-install. Pour obtenir des détails sur la création d'un pont réseau, consultez la Section 6.4.1, "Configurer un réseau ponté sur un hôte Red Hat Enterprise Linux 7".
{: .prompt-info }

#### Réseau ponté avec une adresse IP statique

La mise en réseau pontée peut également être utilisée pour configurer l'invité afin qu'il utilise une adresse IP statique.  

Pour configurer un réseau ponté avec une adresse IP statique pour la machine virtuelle invitée, utilisez les options suivantes : 

```
--network br0 \
--extra-args "ip=192.168.1.2::192.168.1.1:255.255.255.0:test.example.com:eth0:none"
```

Pour plus d'informations sur les options d'amorçage par le réseau, consultez le [Guide d'installation de Red Hat Enterprise Linux 7 (en)](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-anaconda-boot-options.html#sect-boot-options-installer). 

#### Aucun réseau

Pour configurer une machine virtuelle invitée sans interface réseau, utilisez l'option suivante : 

    --network=none





