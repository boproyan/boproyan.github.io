+++
title = 'Aide mémoire Borg Backup'
date = 2024-01-08 00:00:00 +0100
categories = ['borgbackup']
+++
### Initialisation dépôt borg

Avant de lancer notre première sauvegarde, il faut créer un repository (dépôt).

Dépôt non chiffré

Pour créer un repository sans chiffrement

    borg init -e none /backup/borg

Pour créer un dépôt distant à travers SSH, on utilisera la syntaxe suivante (avec destination = 192.168.91.215 et port SSH 4022) 

    borg init -e none ssh://user@192.168.91.215:4022/backup/user

Dépôt chiffré

Pour protéger le repository avec une passphrase 

    borg init -e  repokey /backup/borg

la passphrase sera saisie 2 fois

C'est le couple passphrase + clé qui vous permet de déchiffrer le contenu, sauvegarder cette clé.

Si on veut utiliser un dépôt distant à travers SSH, on utilisera la syntaxe suivante (avec destination = 192.168.91.215 et port SSH 4022) 

    borg init -e repokey ssh://user@192.168.91.215:4022/backup/user

### Créer une sauvegarde

commande `borg create` 

On se positionnera dans le dossier concerné avant  

    cd /home/user
    borg create --stats -C zstd,10 /backup/borg::20230119 .

Explications 

* `--stats` :  afficher des statistiques 
* `-C` : utiliser la compression 
* `/backup/borg::20230119` : Le chemin du dépôt, suivi de `::` et du nom de la sauvegarde
* `.` : Le dossier à sauvegarder (ici le dossier courant) On peut évidemment mettre plusieurs dossiers, comme avec la commande tar.
* `--progress` :afficher la progression de la création de la sauvegarde.
* `--exclude` : exclure des dossiers (peut être utilisé plusieurs fois)

Exemple avec progress et 2 dossiers exclus :

    borg create --progress -C zstd,10 --exclude '.cache' --exclude 'Téléchargements' /backup/borg::20230119 .

Avec un dépôt (repository) chiffré, la passphrase sera demandée.

Avec l'option stats, type de sortie après la sauvegarde 

```
------------------------------------------------------------------------------
Repository: /backup/borg
Archive name: 20230119
Archive fingerprint: 10855358175d9bb31f8b52dee8c13c438d3392581ee15f44b8fbeebbb8872b59
Time (start): Thu, 2023-01-19 19:01:32
Time (end):   Thu, 2023-01-19 19:05:33
Duration: 4 minutes 1.15 seconds
Number of files: 26354
Utilization of max. archive size: 0%
------------------------------------------------------------------------
                     Original size    Compressed size  Deduplicated size
This archive:              4.31 GB            2.40 GB            2.23 GB
All archives:              4.31 GB            2.40 GB            2.23 GB
                       Unique chunks     Total chunks
Chunk index:                   22656            27245
----------------------------------------------------------------------
```

Ne pas utiliser deux fois la même commande, chaque nom de sauvegarde doit être unique !

Si on veut utiliser un dépôt distant à travers SSH, on utilisera la syntaxe suivante (avec destination = 192.168.21.105 et port SSH 22) :

    borg create --stats -C zstd,10 ssh://user@192.168.21.105:22/backup/user::20230119 .

### Lister les backups

Pour visualiser les backups sur un repository, commande  `borg list` 

    borg list /backup/borg/

Résultat commande

    20230119          Thu, 2023-01-19 19:01:32 [10855358175d9bb31f8b52dee8c13c438d3392581ee15f44b8fbeebbb8872b59]

Si dépôt chiffré, la passphrase vous sera demandée.

Dépôt distant à travers SSH, on utilisera la syntaxe suivante (avec destination = 192.168.21.105 et port SSH 22) :

    borg list ssh://user@192.168.21.105:22/backup/user

### Lister le contenu d'un backup

Visualiser le contenu d'un backup sur un dépôt, commande `borg list` dépôt suivi de deux fois deux points 

    borg list /backup/borg::20230119

Ce qui sort une liste, un extrait (avec les permissions à la ls -l ) :

```
-rw-r--r-- user user  5258765 Tue, 2021-07-20 17:40:41 Téléchargements/anydesk_6.1.1-1_x86_64.rpm
drwxr-xr-x user user        0 Wed, 2018-12-26 18:58:19 Modèles
-rw-r--r-- user user   184252 Sat, 2023-01-07 11:24:16 Documents/Sans titre.png
-rw-r--r-- user user   238643 Sat, 2022-01-01 18:41:49 Documents/annee-2022-lt.jpg
-rw-r--r-- user user   258725 Thu, 2022-07-28 10:36:14 Documents/bp-remarque.png
```

Si backup dépôt chiffré, la passphrase vous sera demandée.

Si dépôt distant à travers SSH, on utilisera la syntaxe suivante (avec destination = 192.168.21.105 et port SSH 22) :

    borg list ssh://user@192.168.21.105:22/backup/user::20230119

### Informations sur un backup

Informations globales sur un backup, commande `borg info`  dépôt suivi de deux fois deux points 

    borg info /backup/borg::20230119

Résultat commande

```
Repository: /backup/borg
Archive name: 20230119
Archive fingerprint: 10855358175d9bb31f8b52dee8c13c438d3392581ee15f44b8fbeebbb8872b59
Comment: 
Hostname: fedora
Username: root
Time (start): Thu, 2023-01-19 19:01:32
Time (end): Thu, 2023-01-19 19:05:33
Duration: 4 minutes 1.15 seconds
Number of files: 26354
Command line: /usr/bin/borg create --stats -C zstd,10 /backup/borg::20230119 .
Utilization of maximum supported archive size: 0%
----------------------------------------------------------------------
                     Original size    Compressed size  Deduplicated size
This archive:              4.31 GB            2.40 GB            2.23 GB
All archives:              4.31 GB            2.40 GB            2.23 GB
                       Unique chunks     Total chunks
Chunk index:                   22656            27245
----------------------------------------------------------------------
```


Si dépôt distant à travers SSH, on utilisera la syntaxe suivante (avec destination = 192.168.21.105 et port SSH 22) :

    borg info ssh://user@192.168.21.105:22/backup/user::20230119

### Restaurer des éléments

La commande `borg extract` dépôt suivi de deux fois deux points 

Sans argument, extraction de toute la sauvegarde dans le dossier courant 

    borg extract --progress /backup/borg::20230119

L'option `--progress` permet de suivre borg dans l'extraction !

Récupérer que quelques éléments en les passant en argument (comme tar), ici le dossier Téléchargements par exemple 

    borg extract --progress /backup/borg::20230119 Téléchargements

Si vous restaurer en tant que root des fichiers d'un autre utilisateur, les données sont extraites dans un premier temps avec le propriétaire root et les droits sont appliqués après l'extraction de tous les fichiers.
{: .prompt-warning }

Si dépôt distant à travers SSH, on utilisera la syntaxe suivante (avec destination = 192.168.21.105 et port SSH 22) :

    borg extract --progress ssh://user@192.168.21.105:22/backup/user::20230119

### Monter une sauvegarde

Monter une sauvegarde sur un point de montage. On visualise la sauvegarde comme un dossier classique du système, et restaure les éléments par copier coller en ligne de commande ou graphique.  
Le montage se fait dans l'espace utilisateur. Pas besoin des droits root pour faire les opérations de montage et démontage.  
En créant un dossier dans /var/tmp nous avons également les droits en tant qu'utilisateur du système.
{: .prompt-info }

Créer un dossier temporaire 

    mkdir /var/tmp/borguser

On monte la sauvegarde avec la commande `borg mount` 

    borg mount /backup/borg::20230119 /var/tmp/borguser

On notera que les données sont montées en `lecture seule`{: .prompt-warning } comme l'indique la commande mount 

    borgfs on /var/tmp/borguser type fuse (ro,nosuid,nodev,relatime,user_id=1000,group_id=1000,default_permissions)

Les données de l sauvegarde sont accessibles en mode fichier, on peut donc les lister, les copier/coller, etc. :

    ls -l /var/tmp/borguser/

```
-rw-rw-r--. 1 user user 3087 18 avril  2020  1
-rw-rw-r--. 1 user user 3518 18 avril  2020  2
-rw-rw-r--. 1 user user 3564 18 avril  2020  3
drwxr-xr-x. 1 user user    0  1 mai    2019  Bureau
drwxr-xr-x. 1 user user    0  4 janv. 18:23  Documents
drwxr-xr-x. 1 user user    0 26 oct.   2020  Images
drwxr-xr-x. 1 user user    0 26 déc.   2018  Modèles
drwxr-xr-x. 1 user user    0 26 oct.   2020  Musique
```

Pour le démontage , commande `borg umount`  

    borg umount /var/tmp/borguser

ou la commande native `fusermount` 

    fusermount -u /var/tmp/borguser

### Supprimer des backup

On exécute la commande `borg list`,  4 backups 

```
20230119         Thu, 2023-01-19 19:01:32 [10855358175d9bb31f8b52dee8c13c438d3392581ee15f44b8fbeebbb8872b59]
20230119-2       Thu, 2023-01-19 19:19:49 [cf1a5b5379437ee2a7a55c82325d976bd884d9d499e5e7486e776c8e95e7ae1e]
20230119-3       Thu, 2023-01-19 19:24:25 [e91320464b75d48bf72d6e1f14de0403df9cefb50d91a66a39a5d04585f9ac8b]
20230119-4       Thu, 2023-01-19 19:24:43 [9ba338b605faed78d75321dfc04c889be41f81b2d222d2cf77d67d8134767b30]
```


Pour supprimer un backup, on utilise la commande `borg delete` avec la même syntaxe que lors de la création avec les deux fois deux points :

    borg delete /backup/borg::20230119-2

On exécute de nouveau la commande `borg list` , le backup 2 est supprimé

```
20230119         Thu, 2023-01-19 19:01:32 [10855358175d9bb31f8b52dee8c13c438d3392581ee15f44b8fbeebbb8872b59]
20230119-3       Thu, 2023-01-19 19:24:25 [e91320464b75d48bf72d6e1f14de0403df9cefb50d91a66a39a5d04585f9ac8b]
20230119-4       Thu, 2023-01-19 19:24:43 [9ba338b605faed78d75321dfc04c889be41f81b2d222d2cf77d67d8134767b30]
```

les 3 autres sauvegardes sont bien intactes 

Si  dépôt distant à travers SSH, on utilisera la syntaxe suivante (avec destination = 192.168.21.105 et port SSH 22) :

    borg delete ssh://user@192.168.21.105:22/backup/user::20230119-2

Supprimer des archives ne libère pas réellement de la place.  
Compacter le dépôt avec la commande `borg compact` 

    borg compact /backup/borg

### Garder les X derniers backups

Pour garder que les X derniers backups (utile dans le cas d'une automatisation via script) commande `borg prune`

ici avec les 2 derniers 

    borg prune --keep-last 2 /backup/borg

Lister, il ne me reste que les 2 derniers 

```
20230119-3       Thu, 2023-01-19 19:24:25 [e91320464b75d48bf72d6e1f14de0403df9cefb50d91a66a39a5d04585f9ac8b]
20230119-4       Thu, 2023-01-19 19:24:43 [9ba338b605faed78d75321dfc04c889be41f81b2d222d2cf77d67d8134767b30]
```

Sachant que borg déduplique et compresse, on peut en garder plus !

Si dépôt distant à travers SSH, on utilisera la syntaxe suivante (avec destination = 192.168.21.105 et port SSH 22) :

    borg prune --keep-last 2 ssh://user@192.168.21.105:22/backup/user

Options

* `--keep-last 5` : Garder les 5 derniers backup
* `--keep-daily 5` : Garder les 5 derniers backups journaliers
* `--keep-monthly 5` : Garder les 5 derniers backups mensuels

### Vérifier le dépôt

Pour vériier la cohérence du dépôt, commande `borg check` et recherche d'éventuels problèmes  

    borg check --progress /backup/borg

### Compacter les données

A force de générer des sauvegardes, et d'en supprimer, on peut tenter de recompacter l'ensemble des données pour libérer un peu d'espace, commande `borg compact` 

    borg compact --progress /backup/borg

### Changer la passphrase

Changer une passphrase, commande `borg key` 

    borg key change-passphrase /backup/chiffre

L'ancienne clé vous sera demandée et deux fois la nouvelle :

```
Enter passphrase for key /backup/chiffre: 
Enter new passphrase: 
Enter same passphrase again: 
```
