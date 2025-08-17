+++
title = 'Sauvegarde restauration système complet Linux avec Rsync ou snapshots (CYA)'
date = 2022-09-20 00:00:00 +0100
categories = rsync sauvegarde
+++
## Rsync

### Sauvegarde complète système Linux avec Rsync

Tout d'abord, insérez votre support de sauvegarde (clé USB ou disque dur externe).  
Ensuite, recherchez la lettre de lecteur à l'aide de la commande ' fdisk -l'  
Dans mon cas, mon identifiant de clé USB est **/dev/sdb1** 

Montez votre disque à n'importe quel endroit de votre choix. Je vais le monter sous **/mnt** 

    $ sudo mount /dev/sdb1 /mnt

Pour sauvegarder tout le système, il vous suffit d'ouvrir votre Terminal et d'exécuter la commande suivante en tant qu'utilisateur **root** 

    $ sudo rsync -aAXv / --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} /mnt

Cette commande sauvegardera l'intégralité du répertoire racine ( `/`), à l'exception des répertoires , `/dev`, `/proc`, `/sys`, `/tmp`, `/run`, `/mnt`, `/media` et `/lost+found` en enregistrant les données dans le dossier `/mnt`

Décomposons la commande ci-dessus et voyons ce que fait chaque argument.

*    **rsync** - Un utilitaire de copie de fichiers rapide, polyvalent, local et distant
*    **-aAXv** - Les fichiers sont transférés en mode "archive", ce qui garantit que les liens symboliques, les périphériques, les autorisations, les propriétaires, les heures de modification, les ACL et les attributs étendus sont conservés.
*    **/** - Répertoire des sources
*    **--exclude** - Exclut les répertoires donnés de la sauvegarde.
*    **/mnt** - C'est le dossier de destination de la sauvegarde.

Remarque importante : n'oubliez pas que vous devez exclure le répertoire de destination , s'il existe dans le système local. Cela évitera la boucle infinie.
{: .prompt-warning }

Si vous souhaitez conserver les liens physiques , incluez simplement **-Hflag** dans la commande ci-dessus. Veuillez noter qu'il consomme plus de mémoire.

    sudo rsync -aAXHv / --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} /mnt

### Restauration sauvegarde

Pour restaurer la sauvegarde , inversez simplement les chemins source et destination dans la commande ci-dessus.

N'oubliez pas que cela ne convient qu'aux systèmes locaux et autonomes. Si votre système est activement consulté par d'autres systèmes sur le réseau, ce n'est pas une meilleure solution.
{: .prompt-warning }

En effet, le contenu des systèmes peut être constamment mis à jour toutes les minutes et certains fichiers peuvent changer pendant le processus de rsync.

Supposons, par exemple, que lorsque rsync atteindra le fichier 2, le contenu du fichier précédent (fichier 1) pourrait être modifié. Cela vous laissera avec une erreur de dépendance lorsque vous devrez utiliser cette sauvegarde.

Dans de tels cas, une sauvegarde basée sur un instantané est la meilleure approche. Parce que le système sera "gelé" avant le démarrage du processus de sauvegarde et qu'il sera "dégelé" à la fin du processus de sauvegarde, de sorte que tous les fichiers sont cohérents.

## CYA - Utilitaire d'instantané et de restauration

**CYA** , signifie **C** over **Y** our **Assets**, est un instantané de système open source gratuit et un utilitaire de restauration pour tous les systèmes d'exploitation de type Unix qui utilisent le shell BASH. Cya est portable et prend en charge de nombreux systèmes de fichiers populaires tels que EXT2/3/4, XFS, UFS, GPFS, reiserFS, JFS, BtrFS et ZFS, etc.  
**Veuillez noter que Cya ne sauvegardera pas les données utilisateur réelles**  
<u>Il sauvegarde et restaure uniquement le système d'exploitation lui-même</u>.  
Cya est en fait un utilitaire de restauration du système.  
Par défaut, il sauvegardera tous les répertoires clés comme /bin/, /lib/, /usr/, /var/ et plusieurs autres. <u>Vous pouvez cependant définir vos propres répertoires et chemins de fichiers à inclure dans la sauvegarde, afin que Cya les récupère également</u>.  
De plus, il est possible de définir certains répertoires/fichiers à ignorer de la sauvegarde. Par exemple, vous pouvez ignorer /var/logs/ si vous ne consignez pas de fichiers. Cya utilise en fait la méthode de sauvegarde Rsync sous le capot. Cependant, Cya est un peu plus facile que Rsync lors de la création de sauvegardes continues.

Lors de la restauration de votre système d'exploitation, Cya restaurera le système d'exploitation en utilisant votre profil de sauvegarde que vous avez créé précédemment. Vous pouvez restaurer l'intégralité du système ou des répertoires spécifiques uniquement. Vous pouvez également accéder facilement aux fichiers de sauvegarde même sans restauration complète à l'aide de votre terminal ou de votre gestionnaire de fichiers.  
Une autre caractéristique notable est que nous pouvons générer un script de récupération personnalisé pour automatiser le montage de votre ou vos partitions système lorsque vous restaurez à partir d'un CD en direct, d'une clé USB ou d'une image réseau.  

*En un mot, CYA peut vous aider à restaurer votre système à l'état précédent lorsque vous vous retrouvez avec un système défectueux causé par une mise à jour logicielle, des changements de configuration et des intrusions/pirates, etc.*

### Installer CYA

L'installation de CYA est triviale. Tout ce que vous avez à faire est de télécharger le binaire Cya et de le mettre dans votre chemin système.

    $ git clone https://github.com/cleverwise/cya.git

Cela clonera la dernière version de cya dans un répertoire appelé cya dans votre répertoire de travail actuel.

Ensuite, copiez le binaire cya dans votre chemin ou où vous voulez.

    $ sudo cp cya/cya /usr/local/bin/

C'est aussi simple que cela. CYA est installé ! Maintenant, allons-y et créons des instantanés.

### Création d'instantanés

Avant de créer des instantanés/sauvegardes, créez un **script de récupération** 

    $ cya script

```bash
☀ Cover Your Ass(ets) v2.2 ☀

ACTION ⯮ Generating Recovery Script

Generating Linux recovery script ... 
Checking sudo permissions...
Complete

IMPORTANT: This script will ONLY mount / and /home. Thus if you are storing data on another mount point open the recovery.sh script and add the additional mount point command where necessary. This is also a best guess and should be tested before an emergency to verify it works as desired.
IMPORTANT : Ce script montera UNIQUEMENT / et /home. Ainsi, si vous stockez des données sur un autre point de montage, ouvrez le script recovery.sh et ajoutez la commande de point de montage supplémentaire si nécessaire. Ceci est également une meilleure estimation et doit être testé avant une urgence pour vérifier qu'il fonctionne comme souhaité.

‣ Disclaimer: CYA offers zero guarantees as improper usage can cause undesired results
‣ Notice: Proper usage can correct unauthorized changes to system from attacks
```

La commande ci-dessus créera un répertoire nommé "/home/cya/" et y enregistrera le fichier recovery.sh

    $ ls /home/cya/
    cya cya.conf points LAST_RUN recovery.sh

Enregistrez le fichier **recovery.sh** résultant sur votre <u>clé USB</u> que nous allons utiliser plus tard lors de la restauration des sauvegardes.  
Ce script vous aidera à configurer un environnement chrooté et à monter des lecteurs lorsque vous restaurerez votre système.

Maintenant, créons des instantanés.

Pour créer une sauvegarde progressive standard

    $ cya save

La commande ci-dessus conservera trois sauvegardes avant réécriture.  
Exemple de sortie

```
☀ Cover Your Ass(ets) v2.2 ☀

ACTION ⯮ Standard Backup

Checking sudo permissions...
[sudo] password for sk: 
We need to create /home/cya/points/1 ... done
Backing up /bin/ ... complete
Backing up /boot/ ... complete
Backing up /etc/ ... complete
.
.
.
Backing up /lib/ ... complete
Backing up /lib64/ ... complete
Backing up /opt/ ... complete
Backing up /root/ ... complete
Backing up /sbin/ ... complete
Backing up /snap/ ... complete
Backing up /usr/ ... complete
Backing up /initrd.img ... complete
Backing up /initrd.img.old ... complete
Backing up /vmlinuz ... complete
Backing up /vmlinuz.old ... complete
Write out date file ... complete
Update rotation file ... complete

‣ Disclaimer: CYA offers zero guarantees as improper usage can cause undesired results
‣ Notice: Proper usage can correct unauthorized changes to system from attacks
```

Vous pouvez afficher le contenu de l'instantané nouvellement créé, sous /home/cya/points/

    $ ls /home/cya/points/1/

```
bin cya-date initrd.img lib opt sbin usr vmlinuz
boot etc initrd.img.old lib64 root snap var vmlinuz.old
```

Pour créer une sauvegarde avec un nom personnalisé qui ne sera pas écrasé 

    $ cya keep name BACKUP_NAME

Remplacez BACKUP_NAME par votre propre nom.

Pour créer une sauvegarde avec un nom personnalisé qui écrasera

    $ cya keep name BACKUP_NAME overwrite

Pour créer une sauvegarde et l'archiver 

    $ cya keep name BACKUP_NAME archive

Cette commande stockera les sauvegardes à l'emplacement **/home/cya/archives**

Par défaut, CYA stockera sa configuration dans le répertoire **/home/cya/** et les instantanés avec un nom personnalisé seront stockés dans l'emplacement **/home/cya/points/BACKUP_NAME** . Nous pouvons modifier ces paramètres en modifiant le fichier de configuration CYA stocké dans **/home/cya/cya.conf**
{: .prompt-info }

**CYA ne sauvegardera pas les données de l'utilisateur par défaut** . Il ne sauvegardera que les fichiers système importants.  
Vous pouvez toutefois <u>inclure vos propres répertoires ou fichiers avec les fichiers système</u>.  
Par exemple, si vous vouliez ajouter le répertoire nommé **/home/sk/Downloads** dans la sauvegarde, éditer le fichier **/home/cya/cya.conf** 

    $ nano /home/cya/cya.conf

Définissez le chemin des données de votre répertoire que vous souhaitez inclure dans la sauvegarde comme ci-dessous.

    MYDATA_mybackup="/home/sk/Downloads/ /mnt/backup/sk/"

N'oubliez pas que les répertoires source et de destination doivent se terminer par une barre oblique à la fin.
{: .prompt-warning }

Selon la configuration ci-dessus, CYA copiera tout le contenu du répertoire **/home/sk/Downloads/** et l'enregistrera dans le répertoire **/mnt/backup/sk/** (en supposant que vous l'ayez déjà créé). Ici **mybackup** est le nom du profil, enregistrer et fermer le fichier.

Maintenant, sauvegardons le contenu du répertoire /home/sk/Downloads/. Pour ce faire, vous devez entrer le nom du profil (c'est-à-dire **mybackup** dans mon cas) avec la commande `cya mydata` comme ci-dessous 

    $ cya mydata mybackup

De même, vous pouvez inclure plusieurs données utilisateur avec des noms de profil différents. Tous les noms de profil doivent être uniques.

### Exclure les répertoires

Parfois, vous ne voudrez peut-être pas sauvegarder tous les fichiers système. Vous voudrez peut-être exclure certains éléments sans importance tels que les fichiers journaux. Par exemple, si vous ne souhaitez pas inclure les  répertoires **/var/tmp/** et **/var/logs/** , ajoutez ce qui suit dans le fichier **/home/cya/cya.conf** 

    EXCLUDE_/var/=”tmp/ logs/”

De même, vous pouvez spécifier un par un tous les répertoires que vous souhaitez exclure de la sauvegarde. Une fois terminé, enregistrez et fermez le fichier.

### Ajouter des fichiers spécifiques à la sauvegarde

Au lieu de créer une sauvegarde de tout le répertoire, vous pouvez inclure des fichiers spécifiques à partir d'un répertoire. Pour cela, ajoutez un à un le chemin de vos fichiers dans le fichier **/home/cya/cya.conf** 

    BACKUP_FILES="/home/sk/Downloads/ostechnix.txt"

### Restaurer votre système

Rappelez-vous, nous avons déjà créé un script de récupération nommé **recovery.sh** et l'avons enregistré sur une **clé USB** ?  
Nous en aurons besoin maintenant pour restaurer notre système défectueux.

Démarrez votre système avec n'importe quel CD/DVD amorçable en direct, clé USB.  
Le développeur de CYA recommande d'utiliser un environnement de démarrage en direct de la même version majeure que votre environnement installé ! Par exemple, si vous utilisez le système Ubuntu 18.04, utilisez le média en direct Ubuntu 18.04.

Une fois que vous êtes dans le système en direct, <u>montez le lecteur USB</u> qui contient le script **recovery.sh**. Une fois que vous avez monté le(s) lecteur(s), les répertoires **/** et **/home** de votre système seront montés dans le répertoire **/mnt/cya**  
Ceci est rendu très facile et géré automatiquement par le script **recovery.sh** pour les utilisateurs Linux.

Ensuite, démarrez le processus de restauration à l'aide de la commande :

    $ sudo /mnt/cya/home/cya/cya restore

Suivez simplement les instructions à l'écran. Une fois la restauration terminée, retirez le support en direct et démontez les disques et enfin, redémarrez votre système.

**Que faire si vous n'avez pas ou avez perdu le script de récupération ?**  
Pas de problème, nous pouvons toujours restaurer notre système défectueux.

Démarrez le média en direct. À partir de la session en direct, créez un répertoire pour monter le ou les lecteurs.

    $ sudo mkdir -p /mnt/cya

Ensuite, montez votre **/** et **/home** (si sur une autre partition) dans le répertoire /mnt/cya .

    $ sudo mount /dev/sda1 /mnt/cya
    $ sudo mount /dev/sda3 /mnt/cya/home

Remplacez **/dev/sda1** et **/dev/sda3** par vos partitions correctes (utilisez la commande `fdisk -l` pour trouver vos partitions).

Enfin, démarrez le processus de restauration à l'aide de la commande :

    $ sudo /mnt/cya/home/cya/cya restore

Une fois la récupération terminée, démontez toutes les partitions montées, supprimez le support d'installation et redémarrez votre système.

À ce stade, vous pourriez obtenir un système fonctionnel. J'ai supprimé certaines bibliothèques importantes du serveur Ubuntu 18.04 LTS. Je l'ai restauré avec succès à l'état de fonctionnement en utilisant l'utilitaire CYA.

