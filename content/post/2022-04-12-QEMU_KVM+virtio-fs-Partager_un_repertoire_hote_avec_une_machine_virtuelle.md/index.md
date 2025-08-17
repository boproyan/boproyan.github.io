+++
title = 'QEMU/KVM + virtio-fs - Partager un répertoire hôte avec une machine virtuelle'
date = 2022-04-12 00:00:00 +0100
categories = ['kvm']
+++
![Qemu](Qemu_logo_blanc.png){:height="50"} ![KVM](kvm-logo.png){:height="50"}   

* [QEMU/KVM + virtio-fs - Sharing a host directory with a virtual machine](https://www.tauceti.blog/posts/qemu-kvm-share-host-directory-with-vm-with-virtio/)
* [Sharing files with Virtiofs](https://libvirt.org/kbase/virtiofs.html)

Avant virtio-fs, si quelqu'un voulait partager des fichiers entre une machine virtuelle (VM) et l'hôte sur lequel la VM fonctionne, il n'y avait pas beaucoup d'options qui fonctionnaient très bien en termes de performances. Parmi les autres options, il y avait principalement SMB-Share (Samba), NFS ou virto-9p. Comme nous l'avons déjà mentionné, elles ont toutes en commun de ne pas être très rapides. L'une des raisons est qu'ils dépendent tous d'éléments liés au réseau qui nuisent aux performances.

### virtio-fs

virtio-fs est conçu pour offrir une sémantique de système de fichiers local et des performances.   
virtio-fs profite de la co-localisation de la machine virtuelle avec l'hyperviseur pour éviter les surcharges associées aux systèmes de fichiers réseau.   
virtio-fs utilise FUSE comme base. Contrairement à FUSE traditionnel où le démon du système de fichiers s'exécute dans l'espace utilisateur, le démon de virtio-fs s'exécute sur l'hôte.   
Un périphérique VIRTIO transporte les messages FUSE et fournit des extensions pour des fonctionnalités avancées non disponibles dans FUSE traditionnel.  

Généralités

*    Les invités ont un accès complet uid/gid au répertoire partagé.
*    Les invités n'ont pas d'accès en dehors du répertoire partagé
*    Utilisez un système de fichiers dédié pour le répertoire partagé afin d'éviter l'épuisement des inodes ou d'autres attaques par déni de service.
*    Le répertoire parent du répertoire partagé doit avoir des permissions rwx------ pour empêcher les non-propriétaires d'accéder à des fichiers non fiables.
*    Monter le répertoire partagé nosuid,nodev sur l'hôte


Les prérequis pour utiliser virtio-fs 

*    La VM invitée doit exécuter un noyau Linux >= 5.4 
*    Sur l'hôte, QEMU 5.0 et libvirt 6.2 doivent être installés 




### Partage répertoire hôte avec un invité

Ajoutez les éléments XML de domaine suivants pour partager le répertoire hôte `/srv/media` avec l'invité

```xml
<domain>
  ...
  <memoryBacking>
    <source type='memfd'/>
    <access mode='shared'/>
  </memoryBacking>
  ...
  <devices>
    ...
    <filesystem type='mount' accessmode='passthrough'>
      <driver type='virtiofs'/>
      <source dir='/srv/media'/>
      <target dir='media_tag'/>
    </filesystem>
    ...
  </devices>
</domain>
```

N'oubliez pas les éléments `<memoryBacking>`. Ils sont nécessaires pour la connexion **vhost-user** avec le démon **virtiofsd**.

Notez que malgré son nom, le `target dir` est une chaîne arbitraire appelée `media_tag` qui est utilisée à l'intérieur de l'invité pour identifier le système de fichiers partagé à monter. Il n'est pas nécessaire qu'elle corresponde au point de montage souhaité dans l'invité.
{: .prompt-info }

Démarrez l'invité et montez le système de fichiers

    sudo mount -t virtiofs media_tag /mnt/media

Pour un montage permanent, modifier `/etc/fstab`

    media_tag /mnt/media virtiofs rw,_netdev 0 0

Rechargement

    sudo mount -a

Note : ceci nécessite le support de virtiofs dans le noyau de l'invité (Linux v5.4 ou plus).
{: .prompt-warning }

