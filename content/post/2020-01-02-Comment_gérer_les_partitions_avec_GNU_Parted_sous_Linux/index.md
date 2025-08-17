+++
title = 'Comment gérer les partitions avec GNU Parted sous Linux'
date = 2020-01-02 00:00:00 +0100
categories = ['cli']
+++
### Objectif

Apprendre à gérer des partitions à l'aide du gestionnaire de partitions GNU parted sous Linux.

### Exigences

*  Autorisations root 

### Conventions

* `#` - nécessite que la commande linux donnée soit exécutée avec les privilèges root soit directement en tant qu'utilisateur root, soit en utilisant la commande `sudo`
* `$` - commande linux donnée à exécuter en tant qu'utilisateur normal non privilégié 

### Introduction

La gestion des partitions est l'une des tâches les plus essentielles et les plus dangereuses à effectuer lorsque vous travaillez avec des systèmes d'exploitation. Dans ce tutoriel, nous nous concentrerons sur l'utilisation de GNU parted et verrons comment nous pouvons l'utiliser pour créer, redimensionner et supprimer des partitions de l'interface de ligne de commande. **Parted** peut fonctionner à la fois en mode interactif et non interactif, ce dernier étant particulièrement utile lorsque nous voulons automatiser certaines opérations ou lorsque les commandes doivent s'exécuter dans un contexte sans surveillance, peut-être dans un script ou dans un fichier kickstart .

### Initialisation d'un périphérique avec une table de partition

Le périphérique sur lequel je vais travailler dans ce tutoriel, c'est **/dev/sde** :  
La première chose que nous voulons faire est de laisser parted afficher l'état actuel de ce lecteur.  
Pour fonctionner en interactive mode nous devons lancer parted avec les autorisations root, en passant comme argument à la commande, le chemin du périphérique sur lequel nous voulons opérer, dans ce cas:

    $ sudo parted /dev/sde

L'invite séparée sera ouverte:

```
Utilisation de /dev/sde
Bienvenue sur GNU Parted ! Tapez « help » pour voir la liste des commandes.
(parted)                                                                  
```

A ce stade, comme l'a suggéré à l' écran, nous pouvons saisir `help` , pour recevoir une liste des commandes disponibles. Dans ce cas, au fait, nous voulons visualiser l'état actuel du lecteur, donc nous utiliserons la commande `print` :

```
Erreur: /dev/sde : étiquette de disque inconnue
Modèle : Samsung Flash Drive FIT (scsi)                                   
Disque /dev/sde : 32,1GB
Taille des secteurs (logiques/physiques) : 512B/512B
Table de partitions : unknown
Drapeaux de disque : 
```

Comme vous pouvez le voir, puisque `/dev/sde` ne contient pas de table de partition, parted nous montre juste des informations sur le modèle de disque, la taille totale et la taille du secteur. Pour pouvoir utiliser le disque, nous devons l'initialiser, nous devons donc créer une table de partition dessus. La commande qui nous a permis de le faire est `mklabel` . Si nous ne spécifions pas le type de table de partition que nous voulons créer, **parted** nous le demandera à l'invite:

```
(parted) mklabel                                                          
Nouveau type d’étiquette de disque ? msdos          
```

Dans ce cas, nous créons une table de partition **msdos** traditionnelle. Les autres valeurs valides sont "aix", "amiga", "bsd", "dvh", "gpt", "loop", "mac", "pc98" et "sun". Comme indiqué précédemment, nous aurions également pu spécifier le type de table de partition comme argument de la commande mklabel:

    (parted) mklabel msdos

Ceci est très similaire à la commande que nous voulons utiliser si nous voulons effectuer la même tâche, mais de manière non interactive. Si la commande doit s'exécuter dans un contexte sans surveillance, nous devons également fournir l'option -s , (abréviation de --script ): ce faisant, nous serons sûrs que l'intervention de l'utilisateur n'est jamais demandée:

    $ sudo parted -s /dev/sde mklabel msdos

### Création d'une partition

Maintenant, créons notre première partition sur l'appareil: nous devons fournir le partition type , en choisissant entre primaire ou étendu, le type de système de fichiers (facultatif), le point de départ de la partition et le point de fin de la partition. Encore une fois, si elles ne sont pas fournies directement, ces valeurs seront demandées de manière interactive. La commande pour créer une partition est mkpart :

```
(parted) mkpart
Type de partition ?  primary/primaire/extended/étendue? primaire          
Type de système de fichiers ?  [ext2]?                                    
Début ? 1MiB                                                              
Fin ? 1025MiB     
```

Une chose qui doit être claire, c'est que même si **parted** demande un type de système de fichiers, il n'en créera jamais un sur la partition: les informations sont demandées juste pour définir le GUID (Global Unique Identifier) ​​de la partition.

Nous avons spécifié `1MiB` comme point de départ pour la partition, afin qu'elle démarre au secteur `2048` du disque (**1 secteur correspond à 512 octets, donc 2048 * 512 = 1048576 octets = 1 Mo**). Dans ce cas , nous aurions aussi pu utiliser `s` comme une unité, qui représente le `sector` , en fournissant directement le secteur pour la partition de départ.  
Le point de départ de la partition est très important pour l'alignement, mais nous verrons cela plus tard.

Puisque nous voulions une partition 1GiB (1024 MiB), nous avons spécifié 1025 MiB comme point de fin, puisque les partitions commencent à 1MiB. Dans le cas où nous voulions que la partition couvre tout l'espace disponible sur l'appareil, nous aurions pu fournir `100%` en valeur. Il est également important de noter que lors de la fourniture d'une partition, le point de départ ou de fin il est recommandé d'utiliser des **binary units** telles que `MiB` ou `GiB`. Lors de l'exécution en mode non interactif, la commande ci-dessus devient:

    $ sudo parted -s /dev/sde mkpart primary 1MiB 1025MiB

Si maintenant réexécutez la commande d'impression, nous pouvons voir la partition que nous venons de créer:

```
(parted) print                                                            
Modèle : Samsung Flash Drive FIT (scsi)
Disque /dev/sde : 32,1GB
Taille des secteurs (logiques/physiques) : 512B/512B
Table de partitions : msdos
Drapeaux de disque : 

Numéro  Début   Fin     Taille  Type     Système de fichiers  Drapeaux
 1      1049kB  1075MB  1074MB  primary  ext2                 lba
```

Le numéro de la partition, ses points de début et de fin ainsi que sa taille et son type sont affichés. Nous pouvons demander à parted d'utiliser une unité de mesure spécifique lors de l'affichage de ces informations. Disons par exemple que nous voulons utiliser `MiB` tant qu'unité: nous pourrions utiliser la commande `unit` pour le spécifier, puis relancer `print` :

```
(parted) unit MiB                                                         
(parted) print                                                            
Modèle : Samsung Flash Drive FIT (scsi)
Disque /dev/sde : 30592MiB
Taille des secteurs (logiques/physiques) : 512B/512B
Table de partitions : msdos
Drapeaux de disque : 

Numéro  Début    Fin      Taille   Type     Système de fichiers  Drapeaux
 1      1,00MiB  1025MiB  1024MiB  primary  ext2                 lba
```

Comme vous pouvez le voir, l'unité que nous avons spécifiée est maintenant utilisée.

### Vérification de l'alignement d'une partition

Comme nous l'avons dit précédemment, <u>l'alignement d'une partition est un facteur très important pour optimiser les performances</u>. En **parted** on peut vérifier deux types d'alignements, minimal et optimal .  

* En mode **minimal**, le programme vérifie que la partition respecte la valeur d'alignement minimale aux blocs physiques  
* En mode **optimal**, il vérifie si la partition est alignée sur un multiple de la taille du bloc physique, pour fournir des performances optimales. 

La commande à utiliser pour effectuer ces vérifications est `align-check` :

```
(parted) align-check                                                      
type d’alignement (min/opt)  [optimal]/minimal?                           
Numéro de partition ? 1                                                   
1 aligné(es)
```

Une fois la commande exécutée en mode interactif, nous sommes invités à fournir le type d'alignement que nous voulons vérifier (optimal est utilisé par défaut) et le numéro de partition (1). Dans ce cas, parted a confirmé que la partition est correctement alignée. La version non interactive de la commande est:

    $ sudo parted -s /dev/sde align-check optimal 1

Puisque nous avons utilisé à nouveau l'indicateur `-s` , nous n'avons observé aucune sortie de la commande, mais nous pouvons savoir si elle a réussi, en vérifiant son code de sortie:

```
 $ echo $?
 0
```

Comme vous le savez, le `$?` variables contient la valeur de sortie de la dernière commande lancée, et comme il s'agit de `0` , nous savons que la commande elle-même a réussi. Lorsqu'elle ne fournit pas l'option -s , la commande renvoie le résultat de la vérification de manière similaire à ce qui se passe en mode interactif:

```
$ sudo parted /dev/sde align-check optimal 1
 1 aligné
```

### Redimensionner une partition

`Redimensionner une partition est une opération très dangereuse`, surtout si la partition contient déjà un système de fichiers. Sachez que lors de la modification de la taille d'une partition, `parted` <font color="red">`ne lui adaptera jamais le système de fichiers`</font>.  
Par conséquent, en particulier lors de la réduction, vous devez d'abord utiliser les outils dédiés pour redimensionner le système de fichiers utilisé. La commande utilisée pour effectuer une Redimensionnement de partition est `resizepart`  
Notre taille de partition est actuellement de 1 Gio; si par exemple, nous souhaitons l'étendre pour couvrir tout l'espace restant sur l'appareil, nous tapons:

```
(parted) resizepart                                                       
Numéro de partition ? 1                                                   
Fin ?  [1025MiB]? 100%            
```

Après avoir tapé la commande `resizepart` , **parted** nous a demandé de fournir le numéro de la partition et la valeur de sa nouvelle fin. Dans ce cas, nous avons fourni `100%` , ce qui est le moyen le plus court de s'assurer que tout l'espace restant sur l'appareil est couvert. La version non interactive de la commande est:

    sudo parted -s /dev/sde resizepart 1 100%

Où, encore une fois, 1 est le numéro de partition et 100% sa nouvelle valeur pour le point de fin de partition. Si nous exécutons `print` , nous pouvons avoir une confirmation que les modifications que nous avons apportées ont été appliquées:

```
(parted) print                                                            
Modèle : Samsung Flash Drive FIT (scsi)
Disque /dev/sde : 30592MiB
Taille des secteurs (logiques/physiques) : 512B/512B
Table de partitions : msdos
Drapeaux de disque : 

Numéro  Début    Fin       Taille    Type     Système de fichiers  Drapeaux
 1      1,00MiB  30592MiB  30591MiB  primary  ext2                 lba
```

La partition couvre désormais tout l'espace sur l'appareil.

### Supprimer une partition

La suppression d'une partition est tout aussi simple. Il est évident que nous devons effectuer une telle opération avec la plus grande attention. La commande à utiliser dans ce cas est `rm` :

```
(parted) rm                                                       
Numéro de partition ? 1                                                   
```

Encore une fois, comme nous n'avons pas fourni le numéro de partition directement, parted nous a invités à fournir les informations nécessaires. Nous aurions pu le fournir directement, en écrivant `rm 1` . Lors de l'exécution en mode non interactif, la commande devient:

    $ sudo parted -s /dev/sde rm 1

Comme prévu, après avoir exécuté la commande, la partition n'existe plus:

```
(parted) print                                                            
Modèle : Samsung Flash Drive FIT (scsi)
Disque /dev/sde : 30592MiB
Taille des secteurs (logiques/physiques) : 512B/512B
Table de partitions : msdos
Drapeaux de disque : 

Numéro  Début  Fin  Taille  Type  Système de fichiers  Drapeaux
```

### Conclusions

<font color="red"><b>La gestion des partitions est une tâche dangereuse qui doit être effectuée avec la plus grande attention.</b></font>  
Bien que de nombreux outils graphiques existent sur Linux pour accomplir les tâches nécessaires (le plus célèbre est probablement **Gparted** qui est basé sur **Parted** lui-même), nous avons parfois besoin de la simplicité et de la puissance de la ligne de commande. Dans de telles situations, se séparer est le bon outil. Comme toujours, il est toujours recommandé de consulter la page de manuel du programme. Amusez-vous et faites attention! 