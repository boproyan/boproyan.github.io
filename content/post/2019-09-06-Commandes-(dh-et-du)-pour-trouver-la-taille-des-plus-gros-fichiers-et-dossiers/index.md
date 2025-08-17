+++
title = 'Commandes "dh" et "du" pour trouver la taille des plus gros fichiers et répertoires'
date = 2019-09-06 00:00:00 +0100
categories = ['cli']
+++
## Commandes dh du

### Espace utilisé

Si vous vous êtes posé la question de savoir quels fichiers prenaient le plus de place sur votre serveur, c’est que vous avez surement eu un problème d’espace disque sur votre disque dur...  
On passe en mode su pour un accès complet au(x) disque(s).  

Pour bien commencer, voici une commande pour lister l’espace (utilisé et restant) sur vos partitions 

    df -h

```
Filesystem      Size  Used Avail Use% Mounted on
udev            3.8G     0  3.8G   0% /dev
tmpfs           780M   77M  703M  10% /run
/dev/sda1        79G   60G   16G  80% /
tmpfs           3.9G   84K  3.9G   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
tmpfs           780M     0  780M   0% /run/user/1001
```

Le disque est utilisé à 80%...

Vous pouvez ensuite commencer à vous amuser à trouver vos plus gros fichiers...


### Taille des dossiers et fichiers

Le dossier courant : **/home/debadm**

Tailles des fichiers et sous-répertoires du dossier courant, dans l’ordre décroissant de taille:

    du -a --max-depth=1 | sort -nr

```
3496932	.
3274772	./kali-linux-2019-2-amd64-iso
179268	./archives
24252	./.bundle
9100	./.gem
7256	./gpx.tar.gz
1432	./knacss
740	./wikistatic2.0.tar.gz
28	./.ssh
24	./.bash_history
20	./.java
4	./videos
4	./temp
4	./start_basicblog.sh
4	./ssh_rc
4	./.profile
4	./.nano
4	./br0
4	./.bashrc
4	./.bash_logout
0	./.mysql_history
0	./media
```


Liste des 10 plus gros fichiers du répertoire courant et de ses sous-repertoires, dans l’ordre décroissant de taille:

    du -a | sort -nr | head -n 10

```
3496932	.
3274772	./kali-linux-2019-2-amd64-iso
3274764	./kali-linux-2019-2-amd64-iso/kali-linux-2019.2-amd64.iso
179268	./archives
144616	./archives/wikistatic_old
142868	./archives/wikistatic_old/_site
77728	./archives/wikistatic_old/_site/files
62932	./archives/wikistatic_old/_site/files/Tutoriel_Samsung_Galaxy_A5_2016-remplacer-la-batterie.mp4
57808	./archives/wikistatic_old/_site/images
31404	./archives/dokuwiki.sav
```

Taille du répertoire courant

    du -hs

```
3.4G	.
```

>la commande `du` accepte en premier paramètre le nom d’un dossier, exemple pour trouver la taille du répertoire home:

    du /home -hs

```
6.0G	/home
```

>Le paramètre `-h` donne un affichage de la taille plus facile à lire (par exemple en Go, Mo, Ko), mais ne fonctionne pas avec les commandes ci dessus qui utilisent le tri décroissant par taille, car dans un tri alphanumérique, 2M est plus grand que 1G…
