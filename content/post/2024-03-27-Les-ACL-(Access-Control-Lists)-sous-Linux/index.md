+++
title = 'Les ACL (Access Control Lists) sous Linux'
date = 2024-03-27 00:00:00 +0100
categories = ['cli']
+++
*Les droits standards et les droits étendus sont des fonctionnalités intéressantes mais qui ne s’applique que pour un seul utilisateur ou un seul groupe. Comment définir des permissions spécifiques, voire différents, pour d’autres utilisateurs ou groupes que les propriétaires ? Les ACLs (Access control lists) offrent une réponse à cette question.*

![](acl-logo.png)

## ACL - Listes contrôle accès

*Vous avez surement déjà été confronté à un problème de permissions de fichiers et dossiers, par exemple vous voulez créer un dossier partagé, mais vous ne souhaitez que certains sous-dossiers ne soient accessibles qu'à certains utilisateurs, modifiables par d'autres, etc...   
Les Access Control Lists (ACL) vous permettent de créer des permissions à la carte, toutes les combinaisons sont possibles.*

###  Installation

*Les droits standards et les droits étendus sont des fonctionnalités intéressantes mais qui ne s’applique que pour un seul utilisateur ou un seul groupe. Comment définir des permissions spécifiques, voire différents, pour d’autres utilisateurs ou groupes que les propriétaires ? Les ACLs offrent une réponse à cette question.*

Il existe deux commandes essentielles : l'une pour manipuler l'ACL d'un fichier (`setfacl`) et l'autre pour la consulter (`getfacl`). Les commandes traditionnelles `chmod` et `chown` ne peuvent accéder aux ACL.

Ces deux commandes nécessitent, sous Debian (et distributions dérivées, comme Knoppix ou Ubuntu), l'installation du paquetage « acl ».  
Pour l'installer : 

    sudo apt install acl 

Pour les distributions à base de RedHat (donc aussi Fedora, Mandriva), il faut installer les paquetages acl.*.rpm et libacl1.*.rpm (leur nom contient leur numéro de version).

si la partition concernée par le partage est de type ext4 le support des acl est actif par défaut: l'option de montage "acl" a été remplacée par "noacl", qui devient donc celle à utiliser si on veut… désactiver le support des acl
{: .prompt-info }

**Autres Systèmes fichiers**  
Quand le noyau est disposé à gérer les ACL, on doit préparer les partitions montées dans un système de fichiers adapté (par exemple, il est exclu de vouloir utiliser ces permissions avec du vfat).

Montage et démontage à la volée, il faut monter les partitions voulues avec l'option acl. Par exemple : 

    mount -t ext3 -o defaults,acl /dev/hda2/ /var/www/ 

Si la partition est déjà montée, on peut modifier ces paramètres à la volée : 

    mount -o remount,acl /var/www/

L'inscription dans `/etc/fstab` des options de gestion des ACL est recommandée quand leur utilisation est régulière.   
Par exemple, notre même couple partition / point de montage serait déclaré ainsi : 

    /dev/hda2 /var/www ext3 defaults,acl 0 0

À chaque montage automatique des partitions, le support des ACL sera activé. 

### Explications

l'attribution des droits se fait grâce à la commande **setfacl**, la lecture des droits avec **getfacl**

Ainsi les deux commandes suivantes sont équivalentes :

    chmod u=rw fichier
    setfacl -m u::rw

Un fichier dont les ACL auront été spécifiés verra s'ajouter un **+** à la fin de la liste des droits avec la commande :

    ls -l fichier

**getfacl** permet d'afficher l'ensemble des permissions définies :

    getfacl fichier

```
# file: fichier
# owner: utilisateur
# group: utilisateur
user::rwx
user:utilisateur1:rw-
user:utilisateur2:r--
group::r--
mask::rwx
other::---
```

ici on peut voir que le propriétaire du fichier (utilisateur) a les droits rwx, utilisateur1 rw- et utilisateur2 r--, les autres utilisateurs n'ont aucun droit

`getfacl --omit-header ...` supprime de l'affichage les 3 premières lignes, le nom du fichier, le propriétaire et le groupe.

    mkdir dir
    ls -ld dir

```
drwxr-xr-x 2 root root 4096 Mar 12 13:54 dir
```

    getfacl --omit-header dir

```
user::rwx
group::r-x
other::r-x
```

Sans acl, la commande getfacl donne les mêmes informations que `ls -ld`

`setfacl -d ...` spécifie des acl par défaut, qui ne peuvent s'appliquer qu'aux dossiers.

    setfacl -m user:fdsadmin:rwx dir
    setfacl -d -m group:nasgrp:r-x dir
    getfacl --omit-header dir

```
user::rwx
user:fdsadmin:rwx
group::r-x
mask::rwx
other::r-x
default:user::rwx
default:group::r-x
default:group:nasgrp:r-x
default:mask::r-x
default:other::r-x
```

`getfacl --access --default ...` L'affichage précédent peut se décomposer en droits d'accès fichier, et en droits par défaut :

    getfacl --omit-header --access dir

```
user::rwx
user:fdsadmin:rwx
group::r-x
mask::rwx
other::r-x
```

    getfacl --omit-header --default dir

```
user::rwx
group::r-x
group:nasgrp:r-x
mask::r-x
other::r-x
```

Les **permissions effectives** sont affichées individuellement pour les utilisateurs ou les groupes qui subissent un droit de hiérarchie supérieure différent :

    chmod g-w dir
    ls -ld dir

```
drwxr-xr-x+ 2 root root 4096 Mar 12 13:54 dir
```

    getfacl --omit-header dir

```
user::rwx
user:fdsadmin:rwx               #effective:r-x
group::r-x
mask::r-x
other::r-x
default:user::rwx
default:group::r-x
default:group:nasgrp:r-x
default:mask::r-x
default:other::r-x 
```

Ici l'utilisateur fdsadmin qui avait pourtant les droits rwx s'est vu amputer du droit w supprimé au groupe.

autoriser à "utilisateur" la lecture et l'écriture sur "fichier"

    setfacl -m user:utilisateur:rw fichier
    setfacl -m u:utilisateur:rw fichier

La même commande est disponible pour les groupes. Il suffit de remplacer **u**/**user** par **g**/**group**

modifier les permissions de plusieurs utilisateurs/groupes sur "fichier" en même temps

    setfacl -m user:utilisateur:rwx,user:utilisateur2:r,group:groupe:rw fichier

définir l'accès en lecture par défaut pour "utilisateur" pour les nouveaux fichiers créés dans "dossier"

    setfacl -m d:u:utilisateur:r dossier

supprimer les ACL pour un utilisateur sur une arborescence

    setfacl -R -x user::nom_user repertoire_base_arborescence
    
supprimer les ACL sur un fichier/dossier

    setfacl -b fichier

### Commande setfacl

Le nom de la commande se comprend set file's ACL (« régler l'ACL du fichier »). Elle possède de nombreuses options dont il convient de prendre connaissance en consultant la page de manuel (`man setfacl`). La commande fonctionne bien sûr aussi de manière récursive (option -R) : 

    setfacl -Rm u:khadija:rw /var/www/ 

modifie l'ACL de tous les fichiers situés sous /var/www/ en attribuant une permission de lecture et d'écriture à l'utilisateur khadija.

**Ajouter des permissions**

La commande `setfacl -m u:khadija:rw /var/www/index.php` modifiera (`-m`) l'ACL de /var/www/index.php en attribuant à l'utilisateur (préfixe u:) khadija les droits rw et en lui refusant le droit d'exécution (qui n'a pas été mentionné dans la commande).

Les principaux paramètres à connaître sont :

*    préfixes :
      *  **u:** (droits pour un utilisateur, nommé ou désigné par son uid) ;
      *  **g:** (droits pour un groupe, nommé ou désigné par son gid) ;
      *  **o:** (droits pour other, le reste du monde) ; 
*    permissions : elles sont codées dans l'ordre `r, w` et `x` ou `X` (ce dernier représentant, comme avec chmod, le droit d'entrée dans les répertoires ou celui d'exécution pour les fichiers qui ont déjà un marqueur x).  
On les remplace par `-` pour une interdiction explicite. Ne pas mentionner un droit revient aussi à une interdiction :` setfacl -m u:khadija:w /var/www/index.php et setfacl -m u:khadija:-w- /var/www/index.php` reviennent au même.

On peut construire des commandes plus complexes en enchaînant les entrées dans l'ACL : 

    setfacl -m u:khadija:rw,g:site1:r--,o:--- /var/www/index.php 

définit des permissions dans l'ACL de `/var/www/index.php` pour l'utilisateur khadija, le groupe site1 et le reste du monde. 

Cette commande permet aussi de modifier les permissions classiques (et remplace dans ce cas `chmod`) : l'utilisateur, le groupe et le reste du monde initiaux du fichier sont simplement désignés par le préfixe (`u:, g:, o:`) suivi d'un nom vide `:` si un fichier **index.php** appartient à `luce:www-data` avec les droits `r--r-----`, pour donner à l'utilisateur et le groupe les droits en lecture et écriture il suffit d'une commande `setfacl -m u::rw,g::rw /var/www/index.php`  
Si l'utilisateur et le groupe possèdent déjà un droit qui ne serait pas mentionné dans la commande setfacl, ce droit sera annulé. Soit le fichier index.php avec les droits `rw-r-----` pour `luce:www-data`  
La commande `setfacl -m u::r,g::x index.php` modifiera les droits à `r----x---` pour pour `luce:www-data`

**Droits par défaut et héritage des droits étendus**

Les droits étendus d'un objet parent ne sont pas automatiquement hérités par les objets contenus. Par exemple, si un répertoire (`root:www-data, rwxr-x-r-x`) possède une ACL `u:luce:rwx`, un fichier créé à l'intérieur (ou déjà présent avant l'adjonction de l'ACL) ne reçoit pas cette ACL et ses droits sont ceux impliqués par l'[umask](https://lea-linux.org/documentations/Fstab) défini.

On peut modifier ce comportement en ajoutant, **aux répertoires seulement**, un attribut default, codé `d:`, qui se transmet à tous les fichiers créés dans le répertoire après l'ajout de l'ACL par défaut.  
Par exemple, `setfacl -m d:u:luce:rwX /var/www` donne à luce les droits de lecture et écriture (ainsi qu'« exécution » quand il s'agit de répertoires) pour tous les fichiers qui seront créés sous `/var/www` à partir de ce moment, jusqu'à ce que cette ACL « par défaut » soit annulé ou remplacé. 

**Retirer des permissions**

Pour annuler tout ou partie d'une ACL : 

    setfacl -b /var/www/index.php 

ôte tout le contenu de l'ACL du fichier, tandis que 

    setfacl -x u:khadija,g:site1 /var/www/index.php 

retire les permissions propres à khadija et au groupe site1.

Les permissions ACL par défaut d'un répertoire (`d:`) s'annulent par `setfacl -k`

**Le masque**

Le masque est une synthèse des valeurs les plus permissives que possède un fichier doté d'une ACL. Les droits de l'utilisateur fondamental ne sont cependant pas pris en compte.  
Le masque est calculé automatiquement : 

    chown luce:www-data index.php chmod 640 index.php 
    ls -l index.php

```
    -rw-r-----  1 luce www-data 5055 2005-10-16 18:53 index.php
```

    getfacl index.php

```
   # file: index.php
   # owner: luce
   # group: www-data
   user::rw-
   group::r--
   other::---
```

Ce fichier n'a pas d'ACL donc pas de masque.

    setfacl -m u:jean:rw,g:web:rw index.php getfacl index.php

```
   # file: index.php
   # owner: luce
   # group: www-data
   user::rw-
   user:jean:rw-
   group::r--
   group:web:rw-
   mask::rw-
   other::---
```

Maintenant que le fichier possède une ACL, il a reçu un masque : les permissions les plus élevées (utilisateur exclu) étant `rw`, c'est aussi la valeur du masque.

L'intérêt du masque est de pouvoir limiter d'un coup toutes les permissions d'un fichier (étendues ou non), sauf celles du propriétaire ; on utilise pour cela le préfixe `m:` suivi du droit maximal à accorder : 

    getfacl index.php

```
   # file: index.php
   # owner: luce
   # group: www-data
   user::rw-
   user:jean:rw-
   group::r--
   group:web:rw-
   mask::rw-
   other::---
```

    setfacl -m m:r index.php getfacl index.php

```
   # file: index.php
   # owner: luce
   # group: www-data
   user::rw-
   user:jean:rw-                   #effective:r--
   group::r--
   group:web:rw-                 #effective:r--
   mask::r--
   other::---
```

Les valeurs modifiées sont indiquées par le commentaire « effective: » suivi des permissions effectives après l'application du masque (ici, jean et web n'ont plus que le droit r, la situation reste la même pour www-data). 

### Commande getfacl

Cette commande suivie d'un nom de fichier affiche l'ACL de ce fichier (get file's ACL « récupérer l'ACL du fichier »). Par exemple : 

    getfacl /var/www

```
  # file: var/www
  # owner: root
  # group: www-data
  user::rwx
  user:luce:rwx
  group::rwx
  mask::rwx
  other::r-x
  default:user::rwx
  default:user:khadija:rwx
  default:group::rwx
  default:group:www-data:r-x
  default:mask::rwx
  default:other::r-x
```

On voit qu'outre les droits traditionnels attribués à `root:www-data` (droits indiqués après `user::` et `group::`), sont aussi définis :

*    des droits complets pour luce (`user:luce:rwx`) ;
*    une permission ACL par défaut donnant des droits complets à khadija sur tous les nouveaux fichiers créés sous /var/www/ (`default:user:khadija:rwx`) ;
*    une autre permission ACL par défaut donnant des droits de lecture et d'exécution au groupe www-data sur les mêmes fichiers (`default:group:www-data:r-x`).

Noter que `user::, group::` et `other::` représentent le triplet **utilisateur / groupe / reste du monde** des permissions classiques. Appliquer cette commande sur un fichier qui n'a pas d'ACL définie donne les mêmes informations que `ls -l`, dans un format différent : 

    setfacl -b index.php # retirer les ACL pouvant exister 
    ls -l index.php

```
  -rw-r-----  1 root www-data 5055 2005-10-16 18:53 index.php
```

    getfacl index.php

```
  # file: index.php
  # owner: root
  # group: www-data
  user::rw-
  group::r--
  other::---
```

### Commandes ls, cp et mv

Ces commandes doivent pouvoir lister, copier et déplacer les ACL en même temps que les fichiers. Pour les deux premières commandes, il faut préciser explicitement que l'on veut afficher/conserver les droits (ce qui est aussi le cas quand on ne travaille que sur les droits classiques) : `ls -l`, `cp -a`  
La commande `mv`, quant à elle, préserve toujours les droits.

Quand les droits étendus ne peuvent être conservés (déplacement ou copie vers un système de fichier qui n'est pas configuré pour les recevoir ou utilisation d'une version de cp trop ancienne), un message d'avertissement en informe l'utilisateur. Par exemple : 

    setfacl -m u:luce:rw index.php cp -a index.php /mnt/vfat

```
  cp: preserving permissions for `/mnt/vfat/index.php': Opération non supportée
```

Noter qu'un fichier comportant une ACL qu'on veut lister par `ls -l` n'affiche qu'un `+` à la suite de ses permissions.  
Seule la commande `getfacl`, pour l'instant, permet d'avoir connaissance du détail.  
Par exemple : 

    setfacl -m u:khadija:rw /var/www/index.php ls -l /var/www/index.php

```
  -rw-rw----+ 1 khadija www-data 5055 2005-10-16 18:53 /var/www/index.php
```

Avec `-rw-rw----+`, on sait que le fichier possède une ACL (+), sans en connaître les constituants.

### Sauvegarde des données

Sauvegarder des données dotées d'ACL nécessite :

*    l'utilisation d'un système de fichiers pour le stockage qui soit compatible ;
*    et l'utilisation d'un logiciel de sauvegarde qui soit tout autant compatible.

>À titre indicatif, `tar` et `cpio` et `rsync` ne le sont pas (à moins d'être patchés), `star` et `pax` le sont.

**Pour contourner le problème de sauvegarde**, il est possible d'écrire toutes les ACL dans un fichier qui servira de base à une restauration ultérieure : 

    getfacl --skip-base -R /dossier/dossier/ > fichier 

récupère les informations récursivement et les inscrit dans un simple fichier.  

La restauration se fait au moyen de 

    setfacl --restore=fichier  

Il faut, pour qu'elle fonctionne, se placer à la racine contenant l'arborescence, en raison de la notation relative des chemins (d'où le message `Removing leading '/' from absolute path names` que l'on peut souvent lire en tapant des commandes avec ces programmes).  
Le chemin d'un répertoire `/tmp/test` est enregistré comme `tmp/test` : on doit donc, pour restaurer, lancer la commande depuis la racine de `/tmp`, c'est-à-dire `/`

**Par exemple :** le répertoire `/tmp/test` contient trois fichiers à ACL.  
On sauvegarde les ACL avec 

    getfacl --skip-base -R /tmp/test > acl.acl 

Pour restaurer, on se place à la racine

    cd /
    # et on lance 
    setfacl --restore=acl.acl 

Si on avait lancé la commande depuis `/test`, setfacl aurait renvoyé les erreurs : setfacl: tmp/test: Aucun fichier ou répertoire de ce type `setfacl: tmp/test/a: Aucun fichier ou répertoire de ce type setfacl: tmp/test/b: Aucun fichier ou répertoire de ce type setfacl: tmp/test/c: Aucun fichier ou répertoire de ce type` 

### Compatibilité

Tous les utilitaires (sauvegarde, copie, déplacement de fichiers) ne sont pas nécessairement compatibles avec les ACLs. Il sera donc indiqué de sauvegarder les ACLs définies pour un dossier afin de les repousser sur une copie des fichiers.

Par exemple, on copie le répertoire /opt/partagedans opt/p2 avec récursion (option -R) :

    cp -R /opt/partage /opt/p2
    getfacl /opt/p2

```
getfacl : suppression du premier « / » des noms de chemins absolus
# file: opt/p2
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
```

Sauvegarde de l’ACL originale :

    getfacl -R /opt/partage > acls

```
getfacl : suppression du premier « / » des noms de chemins absolus
```

Adaptation des nouveaux droits :

    sed -i -e "s/opt\/partage/\/opt\/p2/g" acls
    getfacl /opt/p2

```
getfacl : suppression du premier « / » des noms de chemins absolus
# file: opt/p2
# owner: root
# group: root
user::rwx
user:alpha:r-x
group::r-x
group:omega:r-x
mask::r-x
other::r-x
default:user::rwx
default:user:alpha:r-x
default:group::r-x
default:mask::r-x
default:other::---
```

Restauration de l’ACL :

    setfacl --restore=acls

### Astuces

Afficher la version de la commande setfacl

    setfacl --version

ou

    apt-cache policy acl

 
Ajouter des ACL étendues sur un fichier/dossier à partir de la ligne de commande

    setfacl -m u:user:(rwx),g:group:(rwx) fichiers|dossiers

Exemple :  
Avant modification :

```
# file: file.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

Modification des ACL étendues :

    setfacl -m u:adminsys:rw,g:adminsys:r file.txt

Après modification :

```
# file: file.txt
# owner: root
# group: root
user::rw-
user:adminsys:rw-
group::r--
group:adminsys:r--
mask::rw-
other::r--
```

Remarque :  
Un fichier/dossier possédant des ACL étendues est identifié par un « + » lors de la commande ls :

```
    -rw-rw-r--+  1 root     root        0 nov.  19 23:46 file.txt
```

 
Ajouter des ACL étendues sur un fichier/dossier à partir d’un fichier de référence

    setfacl -M aclfilename fichiers|dossiers

Exemple :  
Avant modification :

```
# file: file.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

Modification des ACL étendues :

Contenu du fichier listing.acl :

```
    u:adminsys:rw
    g:adminsys:r
```

    setfacl -M listing.acl file.txt

Après modification :

```
# file: file.txt
# owner: root
# group: root
user::rw-
user:adminsys:rw-
group::r--
group:adminsys:r--
mask::rw-
other::r--
```

Remarque :  
Un fichier/dossier possédant des ACL étendues est identifié par un « + » lors de la commande ls :

```
    -rw-rw-r--+  1 root     root        0 nov.  19 23:46 file.txt
```

 
Ajouter des ACL étendues récursivement sur un dossier
A partir de la ligne de commande

    setfacl -Rm u:user:(rwx),g:group:(rwx) dossiers

A partir d’un fichier de référence

    setfacl -RM aclfilename dossiers

Remarque :  
Tous les fichiers et dossiers contenu dans le dossier spécifié en argument se verront affecter les mêmes ACL étendues.
Il est souvent plus cohérent d’attribuer des ACL différentes aux fichiers et aux dossiers, pour cela utiliser la commande find.

 
Ajouter des ACL étendues récursivement uniquement sur les fichiers
A partir de la ligne de commande

    find . -type f -print0 | xargs -0 setfacl -m u:user:(rwx),g:group:(rwx)

ou

    find . -type f -exec setfacl -m u:user:(rwx),g:group:(rwx) {} +

ou

    find . -type f -exec setfacl -m u:user:(rwx),g:group:(rwx) {} \;

A partir d’un fichier de référence

    find . -type f -print0 | xargs -0 setfacl -M aclfilename

ou

    find . -type f -exec setfacl -M aclfilename {} +

ou

    find . -type f -exec setfacl -M aclfilename {} \;

 
Ajouter des ACL étendues récursivement uniquement sur les dossiers
A partir de la ligne de commande

    find . -type d -print0 | xargs -0 setfacl -m u:user:(rwx),g:group:(rwx)

ou

    find . -type d -exec setfacl -m u:user:(rwx),g:group:(rwx) {} +

ou

    find . -type d -exec setfacl -m u:user:(rwx),g:group:(rwx) {} \;

A partir d’un fichier de référence

    find . -type d -print0 | xargs -0 setfacl -M aclfilename

ou

    find . -type d -exec setfacl -M aclfilename {} +

ou

    find . -type d -exec setfacl -M aclfilename {} \;

 
Supprimer toutes les ACL étendues sur un fichier/dossier

    setfacl -b fichiers|dossiers

 
Supprimer toutes les ACL étendues récursivement sur un dossier

    setfacl -Rb dossiers

Uniquement sur les fichiers contenus dans ce dossier

    find . -type f -print0 | xargs -0 setfacl -b

    find . -type f -exec setfacl -b {} +

    find . -type f -exec setfacl -b {} \;

Uniquement sur les sous-dossiers contenus dans ce dossier

    find . -type d -print0 | xargs -0 setfacl -b

    find . -type d -exec setfacl -b {} +

    find . -type d -exec setfacl -b {} \;

 
Supprimer des ACL sur un fichier/dossier
A partir de la ligne de commande

    setfacl -x u:user,g:group fichiers|dossiers

Exemple :  

    setfacl -x u:adminsys,g:adminsys file.txt

Récursivement : retirera toute ACL faisant référence à l’utilisateur ou au groupe spécifié (ne retournera pas d’erreur s’ils ne sont pas trouvés)

    setfacl -Rx u:user,g:group dossiers

A partir d’un fichier de référence

    setfacl -X aclfilename fichiers|dossiers

Exemple :  
Avec listing.acl :

    u:adminsys
    g:adminsys

    setfacl -X listing.acl file.txt

Récursivement : retirera toute ACL faisant référence à l’utilisateur ou au groupe spécifié dans le fichier (ne retournera pas d’erreur s’ils ne sont pas trouvés)

    setfacl -RX aclfilename dossiers

 
Restaurer les ACL d’une arborescence à partir d’un fichier de sauvegarde

L’emplacement de la restauration dépend du chemin spécifié lors de la sauvegarde.  
Si les chemins sont relatifs à la racine, alors il faudra se placer à la racine pour restaurer les permissions et droits.  
Si les chemins sont relatifs au dossier en question, alors il faudra se placer dans son répertoire parent.  
Si les chemins sont absolus alors la restauration pourra se faire depuis n’importe quel dossier.  
Cas d’un chemin relatif à un dossier :

```
# file: test/
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

# file: test//file.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

    setfacl --restore=filename

Exemple :  

    setfacl --restore=/home/adminsys/backupacl.acl

Cas d’un chemin relatif à la racine :

```
# file: home/adminsys/test/
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

# file: home/adminsys/test//file.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

    (cd / ; setfacl --restore=filename)

Exemple :  

    (cd / ; setfacl --restore=/home/adminsys/backupacl.acl)

Cas d’un chemin absolu

```
# file: /home/adminsys/test/
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

# file: /home/adminsys/test//file.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

    setfacl --restore=filename

Exemple :  

    setfacl --restore=/home/adminsys/backupacl.acl

 
Définir des ACL par défaut sur un dossier

Tous les nouveaux fichiers créés dans ce dossier hériteront de ces ACL par défaut.

    setfacl -m d:u:username:{rwx},d:g:groupname:{rwx},d:o::{rwx} directory/

Exemple :  

    setfacl -m d:u:adminsys:rwx,d:g:adminsys:rwx,d:o::rwx reptest/

### Exemple Dossier multimédia

*Création dossier multimedia géré par les ACL*

    nano /home/yani/creation_dossier_multimedia.sh

<details>
<summary><b>Etendre Réduire</b></summary>
{% highlight shell %}  
#!/bin/bash

# Script de création des dossiers multimédia

GROUPE_MEDIA=multimedia
DOSSIER_MEDIA=/home/multimedia

## Création du groupe multimedia
sudo groupadd -f $GROUPE_MEDIA

## Création des dossiers génériques
sudo mkdir -p "$DOSSIER_MEDIA"
sudo mkdir -p "$DOSSIER_MEDIA/share"
sudo mkdir -p "$DOSSIER_MEDIA/share/Music"
sudo mkdir -p "$DOSSIER_MEDIA/share/Picture"
sudo mkdir -p "$DOSSIER_MEDIA/share/Video"
sudo mkdir -p "$DOSSIER_MEDIA/share/eBook"
sudo mkdir -p "$DOSSIER_MEDIA/share/Divers"

## Création des dossiers utilisateurs
while read user	#USER en majuscule est une variable système, à éviter.
do
		sudo mkdir -p "$DOSSIER_MEDIA/$user"
		sudo mkdir -p "$DOSSIER_MEDIA/$user/Music"
		sudo mkdir -p "$DOSSIER_MEDIA/$user/Picture"
		sudo mkdir -p "$DOSSIER_MEDIA/$user/Video"
		sudo mkdir -p "$DOSSIER_MEDIA/$user/eBook"
		sudo mkdir -p "$DOSSIER_MEDIA/$user/Divers"
		sudo ln -sfn "$DOSSIER_MEDIA/share" "$DOSSIER_MEDIA/$user/Share"
		# Création du lien symbolique dans le home de l'utilisateur.
		sudo ln -sfn "$DOSSIER_MEDIA/$user" "/home/$user/Multimedia"
		# Propriétaires des dossiers utilisateurs.
		sudo chown -R $user "$DOSSIER_MEDIA/$user"
done <<< "$(less /etc/passwd|grep "/bin/bash" |awk -F: '{ print $1}' | grep -Fv 'root')"# Liste les utilisateurs avec exclusion de root
# Le triple chevron <<< permet de prendre la sortie de commande en entrée de boucle.

## Application des droits étendus sur le dossier multimedia.
# Droit d'écriture pour le groupe et le groupe multimedia en acl et droit de lecture pour other:
sudo setfacl -RnL -m g:$GROUPE_MEDIA:rwX,g::rwX,o:r-X "$DOSSIER_MEDIA"
# Application de la même règle que précédemment, mais par défaut pour les nouveaux fichiers.
sudo setfacl -RnL -m d:g:$GROUPE_MEDIA:rwX,g::rwX,o:r-X "$DOSSIER_MEDIA"
# Réglage du masque par défaut. Qui garantie (en principe...) un droit maximal à rwx. Donc pas de restriction de droits par l'acl.
sudo setfacl -RL -m m::rwx "$DOSSIER_MEDIA"
{% endhighlight %}
</details>

Droits en exécution

    chmod +x /home/yani/creation_dossier_multimedia.sh

Exécuter le script

    .//home/yani/creation_dossier_multimedia.sh

Ajout utilisateur au groupe multimedia

    sudo usermod -a -G multimedia $USER

Ajout nextcloud au groupe multimedia

```shell
sudo usermod -a -G multimedia nextcloud
```

`Se reconnecter pour la prise en compte`{: .prompt-info }

