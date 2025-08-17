+++
title = 'Linux commande find'
date = 2020-08-26 00:00:00 +0100
categories = ['commande']
+++
# find

* [La commande find ou la puissance de la recherche](http://ubunteros.tuxfamily.org/spip.php?article172)
* [Rechercher des fichiers avec find](http://www.absolinux.net/tutos/ldcunix.html)
*  [Trier des fichiers selon une date avec FIND﻿](http://www.it-connect.fr/trier-des-fichiers-selon-une-date-avec-find%EF%BB%BF/)
*  [“find” : Quelques rappels](http://www.admin-linux.fr/?p=2691)
*  [A Unix/Linux “find” Command Tutorial ](http://content.hccfl.edu/pollock/Unix/FindCmd.htm)
*  [Mommy, I found it! -- 15 Practical Linux Find Command Examples (Part1)](http://www.thegeekstuff.com/2009/03/15-practical-linux-find-command-examples/)
*  [Daddy, I found it!, 15 Awesome Linux Find Command Examples (Part2)](http://www.thegeekstuff.com/2009/06/15-practical-unix-linux-find-command-examples-part-2/)


## Chercher remplacer une chaine

Sur linux, ne vous est-il jamais arrivé de vouloir chercher une chaine de caractères dans un dossier complet, autrement dit une multitude de fichiers ?
Et bien croyez-moi, si un jour vous devez chercher une adresse email, une adresse ip, un bout de code, ou autre, dans plusieurs milliers de fichiers, cette commande vous épargnera un gros travail manuel.

Placez vous dans le répertoire dans lequel vous souhaitez rechercher une chaine de caractères. Et tapez la commande suivante :

    find . -name "*" -type f -exec grep -Hn "trouvemoi" {} \;

En remplaçant trouvemoi par ce que vous cherchez.

Par exemple, si vous cherchez une adresse IP dans tous vos fichiers logs à la fois, vous ferez (ne tapez que ce qui se trouve après le #) :

    # cd /var/log
    # find . -name "*" -exec grep -Hn "192.168.0.1" {} \;

Alternative

    find . | xargs grep 'The Big Brains Company' -sl

Avec grep

    grep -lR "2016-05-09-website-response-time" media/devel/ouestline-jekyll_posts/
    media/devel/ouestline-jekyll_posts/2017-01-15-syntaxe-markdown.md
    ou
    cd media/devel/ouestline-jekyll_posts/
    grep -lR "2016-05-09-website-response-time" *

### Chercher une chaîne avec exclusion répertoire

    find . -type d \( -name _site  \) -prune -o -name "*" -type f -exec grep -Hn "resources/font-awesome" {} \;           # 1 répertoire
    find . -type d \( -name _site -o -name .jekyll-cache  \) -prune -o -name "*" -type f -exec grep -Hn "\.date" {} \;    # plusieurs répertoires

### Chercher dans des fichiers libreoffice odt

Chercher dans les fichiers type .odt dont le contenu est compressé. pour ce faire on utilisera la ligne de commandes en combinant find grep et unzip, pour cela adaptez cette commande à votre cas :

`find chemin/du/répertoire -name '*.odt' -exec sh -c 'unzip -c "{}" content.xml | grep -qi "motàchercher"' ";" -print`

Pour ce qui est des fichiers .pdf, la recherche se fera aussi en ligne de commande de la même façon qu'avec grep mais en installant auparavant le paquet pdfgrep. 

### Remplacer une chaine

Si vous utilisez Linux, vous aurez forcément besoin un jour de remplacer une chaine de caractères dans plusieurs fichiers, sans pour autant devoir tout faire manuellement.

Imaginons, par exemple, que vous souhaitiez remplacer une adresse email dans tous les fichiers .php d'un répertoire et de ses sous-répertoires. Si l'adresse à remplacer est "georges@6ma.fr", et que la nouvelle adresse est "michel@6ma.fr", voici la commande :

    find . -name "*.php" -type f -exec sed -i "s#georges@6ma.fr#michel@6ma.fr#g" {} \;

Remplacez *.php par l'extension que vous voulez. Et si vous souhaitez le faire dans tous les fichiers (toutes extensions), vous pouvez également faire "*"

    find . -name "*" -type f -exec sed -i "s#wlp2s0#wlan0#g" {} \;

Autre procédure

    find . -name "*" -type f -print |xargs sed -i 's/enp1s0/eth0/g'

## Exclusion de multiples répertoires

Linux Find Exclude Multiple Directories  
<u>Commande 1</u>

    find . -type d \( -name media -o -name images -o -name backups \) -prune -o -print

* `find .` &rarr; Trouver tous les fichiers dans ce répertoire (.)
* `-type d` &rarr; Répertoire ou dossier
* `-prune` &rarr; ignore le cheminement de la procédure de ...
* `\( -name media -o -name images -o -name backups \)` &rarr; Le `-o` signifie simplement **OU**
* `-o -print`  &rarr; Ensuite, si aucune correspondance n'est trouvée, imprimer les résultats, (élaguer les répertoires et imprimer les résultats restants)

Exemple : Je veux exclure les répertoires '_site' , '.jekyll-cache' et '.Trash-1000' de ma recherche 

    find /srv/basicblog/ -type d \( -name _site -o -name .jekyll-cache -o -name .Trash-1000  \) -prune -o -name "*" -type f -exec grep -Hn "/js/searchplus.js" {} \;

```
/srv/basicblog/_includes/default.html:95:   <script src="{{ site.BASE_PATH }}/js/searchplus.js"></script>
/srv/basicblog/README.md:1124:<script src="{{ site.BASE_PATH }}/js/searchplus.js"></script>
``` 

Comme toujours sous Linux, il existe de nombreuses façons d'obtenir le même résultat, et le patient peut préférer construire la commande find en utilisant l'attribut `path`. Veuillez noter que vous devrez peut-être spécifier le chemin avec un préfixe **"./"** et <u>sans barre oblique</u>.  
<u>Commande 2</u>

    find . -path './media' -prune -o -path './images' -prune -o -path './backups' -prune -o -print 

**Différence importante :**

* <u>Commande 1</u> va élaguer **TOUT** répertoire dans le chemin qui correspond au nom, tel que **"./media"** et **"./public/media"**
* <u>Commande 2</u> ne va élaguer que le chemin **"./media"**

## Checher les fichiers qui n'ont pas le critère demandé

Exemple: on veut chercher tous les fichiers `*.md` qui ne contiennent pas `tags:`   

    find . -name "*.md" -type f -exec grep -HEoc "tags:" {} \;  # affiche les noms de fichier avec le compteur d'occurence sur le critère

```bash
./2020-03-07-WireGuard-on-Linux-terminal(advanced).md:0
./2018-08-31-Minipaint-logiciel-dessin-auto-heberge.md:1
./2019-12-25-Installer-Ruby-avec-RVM.md:0
```

Le premier "tube (pipe)" `| grep :0$` filtre cette liste pour n'inclure que les lignes se terminant par `:0`

```bash
./2019-12-25-Bloquer_les_pubs_Pi-Hole_raspberry_et_routeur-freebox.md:0
./2019-12-25-raspberry-hotspot-wifi.md:0
./2019-12-25-_Olimex-A20-Olinuxino-MICRO-SPI.md:0
```

Le deuxième "tube (pipe)" `| sed 's/..$//'` supprime les deux derniers caractères de chaque ligne, ne laissant que les chemins d'accès aux fichiers.

```bash

./2019-12-25-acme-certificats-ssl-letsencrypt-via-api-ovh.md
./2019-12-25-utiliser-son-android-de-facon-plus-securisee.md
./2018-11-23-VPN-Connexions.md
```

## Supprimer des fichiers


*  [Linux or Unix find and remove files with one find command on fly](http://www.cyberciti.biz/faq/linux-unix-how-to-find-and-remove-files/)

Exemple : on veut supprimer tous les fichiers qui commence par "etrex" dans les répertoires et sous-répertoires du chemin courant. 

    find . -type f -name "etrex*" -exec rm -f {} \;

## Supprimer texte entre 2 balises (balises comprises)

Exemple A : Supprimer les balises javascript et le contenu dans les fichiers html

	:::bash
	#!/bin/bash
	# ALL HTML FILES
	FILES="*.html"
	# for loop read each file
	for f in $FILES
	do
	INF="$f"
	OUTF="$f.out.tmp"
	# replace javascript
	sed '/`<script type="text\/javascript"/,/<\/script>`/d' $INF > $OUTF
	/bin/cp $OUTF $INF
	/bin/rm -f $OUTF
	done


## Lister les fichiers récursivement par date de modification

trier les fichiers d’une arborescence récursivement, par ordre de date de modification. 

    find . -type f -printf '%TY-%Tm-%Td %TT %p\n' | sort -r

## Créer une playlist

`find /media/yanplus/Musique/ -iname "*.mp3" > /media/yanplus/Musique/yannick.m3u`

* [Find Files By Access, Modification Date / Time Under Linux or UNIX](http://www.cyberciti.biz/faq/howto-finding-files-by-date/)

## find by date and move files

	:::bash
	# example move files newer 11/01/2010 in "archives/" folder
	#
	touch --date "2010-01-11" /tmp/foo
	# find and move
	for file_erase in `find . -newer /tmp/foo`; do
	   echo -n "Move $file_erase : "
	   mv $file_erase 'archives/' 
	   echo "Done"
	done


