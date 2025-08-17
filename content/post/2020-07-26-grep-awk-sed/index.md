+++
title = 'grep - awk - sed'
date = 2020-07-26 00:00:00 +0100
categories = ['commande']
+++
## grep

*La commande grep chaîne fichier permet d'extraire de fichier toutes les lignes*

* [Recherche du mot "grep"](https://www.startpage.com/do/dsearch?query=linux+commande+grep&cat=web&pl=opensearch&language=francais)
* [Linux Grep OR, Grep AND, Grep NOT Operator Examples](http://www.thegeekstuff.com/2011/10/grep-or-and-not-operators/)
* [Commande grep : Rechercher une aiguille dans un système](https///wodric.com/commande-grep/)
* [How to use the most popular command in Unix - Grep ](http://www.codecoffee.com/tipsforlinux/articles/25.html)
* [Grep, rechercher de manière récursive un terme (ou plusieurs) dans tous les fichiers d’un répertoire](http://www.laurent-napias.com/post/2016/02/13/grep-rechercher-de-maniere-recursive-un-terme-ou-plusieurs-dans-tous-les-fichiers-dun-repertoire)

bash_completion est le MOTIF recherché et /usr l’endroit où on le recherche (ici dans le dossier /usr). C’est un exemple simple pour voir les différences entre les deux propositions, je vous invite à faire les tests ci-dessous.
    grep --color=auto -iRnH 'bash_completion' /usr # Proposition chez sois-net.fr

```
-i : Ignorer la casse aussi bien dans le MOTIF (dans notre exemple c’est bash_completion) que dans les fichiers
-R : Lire récursivement tous les fichiers à l’intérieur de chaque répertoire. Suivre tous les liens symboliques, contrairement à -r
-n : Préfixer chaque ligne de sortie par le numéro de la ligne dans le fichier. La numérotation commence à la ligne 1
-H	: Afficher le nom du fichier pour chaque correspondance. C’est le comportement par défaut quand la recherche est effectuée sur plusieurs fichiers (on peut donc s’en passer car c’est le comportement par défaut et qu’on utilise l’option -R…)
```

    grep --color=auto -RInis 'bash_completion' /usr # Ma proposition

```
-R	: Lire récursivement tous les fichiers à l’intérieur de chaque répertoire. Suivre tous les liens symboliques, contrairement à -r
-I	: Traiter un fichier binaire comme s’il ne contenait aucune correspondance (très utile mais là c’est un choix donc à vous de voir, je ne cherche jamais un MOTIF dans un fichier binaire. Avec cette option grep n’ira pas chercher notre MOTIF dans des fichiers binaires comme /usr/bin/ssh ou /usr/bin/top. J’ajoute qu’un fichier binaire est un fichier qui n’est pas un fichier texte par définition)
-n : Préfixer chaque ligne de sortie par le numéro de la ligne dans le fichier. La numérotation commence à la ligne 1
-i	: Ignorer la casse aussi bien dans le MOTIF que dans les fichiers
-s	: Ne pas afficher les messages d’erreur concernant les fichiers inexistants ou illisibles (très utile ça évite des messages comme grep: /usr/lib/firefox/hyphenation: Aucun fichier ou dossier de ce type ou encore quand on n’est pas root des messages comme grep: /usr/lib/ssl/private: Permission non accordée)
```
Le cas **--color=auto** ou **--color=always**

Avec **--color=always** si on chaîne la sortie vers une commande (via un pipe \|) ou si on redirige la sortie vers un fichier (via > ou >>) alors les caractères liés aux couleurs ne seront pas bien interprétés.

Avec **--color=auto**, les couleurs sont affichées sur la sortie sauf si on chaîne la sortie vers une commande ou si on redirige la sortie vers un fichier, on évite ainsi les caractères liés aux couleurs mal interprétés.

A noter que la plupart des utilisateurs voudront utiliser **less** ET avoir la **couleur**, on procèdera donc ainsi : 

    grep --color=always -RInis 'bash_completion' /usr | less -R

## Grep OR Operator

Use any one of the following 4 methods for **grep OR**. I prefer method number 3 mentioned below for grep OR operator.

### 1. Grep OR Using `\|`

If you use the `grep` command without any option, you need to use `\|` to separate multiple patterns for the or condition.

    grep 'pattern1\|pattern2' filename

For example, `grep` either Tech or Sales from the employee.txt file. Without the back slash in front of the pipe, the following will not work.

```
$ grep 'Tech\|Sales' employee.txt
100  Thomas  Manager    Sales       $5,000
200  Jason   Developer  Technology  $5,500
300  Raj     Sysadmin   Technology  $7,000
500  Randy   Manager    Sales       $6,000
```

### 2. Grep OR Using `-E`

`grep -E` option is for extended regexp. If you use the grep command with -E option, you just need to use `|` to separate multiple patterns for the or condition.

    grep -E 'pattern1|pattern2' filename

For example, grep either Tech or Sales from the employee.txt file. Just use the `|` to separate multiple OR patterns.

```
$ grep -E 'Tech|Sales' employee.txt
100  Thomas  Manager    Sales       $5,000
200  Jason   Developer  Technology  $5,500
300  Raj     Sysadmin   Technology  $7,000
500  Randy   Manager    Sales       $6,000
```

### 3. Grep OR Using `egrep`

egrep is exactly same as `grep -E`. So, use egrep (without any option) and separate multiple patterns for the or condition.

    egrep 'pattern1|pattern2' filename

For example, grep either Tech or Sales from the employee.txt file. Just use the | to separate multiple OR patterns.

```
$ egrep 'Tech|Sales' employee.txt
100  Thomas  Manager    Sales       $5,000
200  Jason   Developer  Technology  $5,500
300  Raj     Sysadmin   Technology  $7,000
500  Randy   Manager    Sales       $6,000
```

### 4. Grep OR Using `grep -e`

Using `grep -e` option you can pass only one parameter. Use multiple -e option in a single command to use multiple patterns for the or condition.

    grep -e pattern1 -e pattern2 filename

For example, grep either Tech or Sales from the employee.txt file. Use multiple -e option with grep for the multiple OR patterns.

```
$ grep -e Tech -e Sales employee.txt
100  Thomas  Manager    Sales       $5,000
200  Jason   Developer  Technology  $5,500
300  Raj     Sysadmin   Technology  $7,000
500  Randy   Manager    Sales       $6,000
```

## Grep AND

### 1. Grep AND using `-E ‘pattern1.*pattern2’`

There is no AND operator in grep. But, you can simulate AND using `grep -E` option.

    grep -E 'pattern1.*pattern2' filename
    grep -E 'pattern1.*pattern2|pattern2.*pattern1' filename

The following example will grep all the lines that contain both “Dev” and “Tech” in it (in the same order).

    $ grep -E 'Dev.*Tech' employee.txt
    200  Jason   Developer  Technology  $5,500

The following example will grep all the lines that contain both “Manager” and “Sales” in it (in any order).

    $ grep -E 'Manager.*Sales|Sales.*Manager' employee.txt

>Note: Using regular expressions in grep is very powerful if you know how to use it effectively.

### 2. Grep AND using Multiple grep command

You can also use multiple grep command separated by pipe to simulate AND scenario.

    grep -E 'pattern1' filename | grep -E 'pattern2'

The following example will grep all the lines that contain both “Manager” and “Sales” in the same line.

```
$ grep Manager employee.txt | grep Sales
100  Thomas  Manager    Sales       $5,000
500  Randy   Manager    Sales       $6,000
```

## Grep NOT

### 1. Grep NOT using `grep -v`

Using `grep -v` you can simulate the NOT conditions. -v option is for invert match. i.e It matches all the lines except the given pattern.

    grep -v 'pattern1' filename

For example, display all the lines except those that contains the keyword “Sales”.

```
$ grep -v Sales employee.txt
200  Jason   Developer  Technology  $5,500
300  Raj     Sysadmin   Technology  $7,000
400  Nisha   Manager    Marketing   $9,500
```

You can also combine NOT with other operator to get some powerful combinations.  
For example, the following will display either Manager or Developer (bot ignore Sales).

```
$ egrep 'Manager|Developer' employee.txt | grep -v Sales
200  Jason   Developer  Technology  $5,500
400  Nisha   Manager    Marketing   $9,500
```

## sed

[La commande sed pour les nazes](https://buzut.net/la-commande-sed-pour-les-nazes/)

```
sed [OPTION]... {script-only-if-no-other-script} [input-file]...

# Commandes :
sed -e --> commande uniligne
sed -f fichier --> passage de commande par un script
sed -i --> modifie directement le fichier source au lieu de rediriger vers la sortie standard
```


Un champ est défini par une expression rationnelle identifiée par les balises `\(` et `\)`

```
^ 	correspond au début d'une ligne (juste avant le premier caractère)
$ 	correspond à la fin d'une ligne (juste après le dernier caractère)
. 	correspond à n'importe quel caractère unique
* 	correspond à aucune ou plusieurs occurrences du caractère qui précède
[ ] 	correspond à n'importe lequel des caractères cités entre les crochets
&     prend la valeur du contenu du fichier avant modif = contenu fichier traité 

/./ 	Récupère toutes les lignes contenant au moins un caractère.
/../ 	Récupère toutes les lignes contenant au moins deux caractères.
/^#/  Récupère toutes les lignes commençant par un #.
/^$/  Récupère toutes les lignes vides.
/}^/   Récupère toutes les lignes finissant par un }.
/} *^/  Récupère toutes les lignes finissant par un } suivi ou non d'espaces.
/[abc]/  Récupère toutes les lignes contenant un des caractères a, b ou c.
/^[abc]/  Récupère toutes les lignes commençant soit par un a, soit un b, soit un c. 
```

Parcourir un fichier et afficher que certaines lignes :

	sed -n '10,12p' passwd

```
new:x:8:12:news:/etc/news:
ucp:x:13:11:uucp:/var/spool/uucp:/sbin/nologin
operato:x:10:0:operator:/root:/sbin/nologin
```

ici la ligne 10 à 12.

	sed -n '1p'

et la juste la première ligne.

Changer un mot par un autre dans vi avec sed :

	:%s/mot/mot1/g

dans le fichier ouvert tous les "mot" seront remplacés par "mot1"

Si on a besoin de remplacer une chaîne par une autre pour toute une liste de fichiers contenu dans un répertoire, on pourra utiliser ce petit script :

```bash
#!/bin/bash
for file in *.sh
do
echo “Traitement de $file …”
sed -e “s/chaine1/chaine2/g” “$file” > “$file”.tmp && mv -f “$file”.tmp “$file”
done
```

Ce script va afficher la liste de tous les fichiers dont l’extension est .sh et faire le remplacement.

Afficher que les lignes non commentées d'un fichier :

	cat fichier.conf | sed -e '/^#.*$/d'

Exemple ou l'on remplace le répertoire /root par /home :

	sed -e "s/\/root/\/home/g" fichier > fichier.tmp && mv -f fichier.tmp fichier 

Comme vous pouvez le voir vous êtes obligé d'échapper les caractères spéciaux.

Remplacer une chaîne par une autre pour une liste de fichiers dans un répertoire, on pourra utiliser ce petit script shell :

```bash 
#!/bin/bash
for file in *.txt
do
echo 'Traitement de $file ...'
sed -e 's/chaine1/chaine2/g' '$file' > '$file'.tmp && mv -f '$file'.tmp '$file'
done 
```

Mettre tout le contenu d'un fichier en majuscule :

	sed -i -e 's/.*/\U&/'

Mettre tout le contenu d'un fichier en minuscule :

	sed -i -e 's/.*/\L&/'

Ajouter un mot ou plus devant chaque ligne d'un fichier :

	sed -e 's/.*/mot ou plus &/' appli

Ajouter un mot ou plus après chaque ligne d'un fichier :

	sed -e 's/.*/& mot ou plus/' appli

Ajouter enable à la fin de la ligne comprenant le mot arg1 et arg2 :

	sed -i -e 's/^\(arg1.*arg2\).*$/&1 enable/g' Fichier.txt

Afficher la ligne 3 d'un fichier :

	sed -n 3p fichier.txt

Commenter un ligne contenant le mot httpd :

	sed -i -e 's/.*httpd.*/#$/g' fichier.txt

Insérer une nouvelle ligne sous/après une autre :  
le fichier contient :

```
lol
ola
mh
pou
```

on insert le mot : client après le mot : ola

	sed -i '/ola/a \client' fichier

Suppression d'une ligne commençant par le mot : football

	sed -i -e '/^football/d' fichier

Autre chose à savoir parce que bien pratique :  
Quand vous modifiez des fichiers il est recommandé de créer un fichier de backup au cas ou :

utiliser l'option `-i .bak`


## grep - awk - sed

### 1-Numeroter - compter - additionner - cumuler

Affiche le nombre de lignes (avec les lignes vides)

```
    sed -n -e '$=' in.txt
    awk 'END{print NR}' in.txt
    awk '{n++} END{print n}' in.txt
```

Affiche le nombre de lignes (sans les lignes vides)

```
    awk '/./ {print}' in.txt | wc -l
    grep "." in.txt | wc -l
```

Somme avec cumul de la colonne 1

```
    awk '{print (total +=$1)}' in.txt
```

Print le nombre de mots

```
    awk '{x=x+NF}END{print x}' in.txt
```

Compte nombre de lignes contenant le pattern 'titi'

```
    awk '/titi/{x+=1}END{print x}' in.txt
```

Print le numero de chaque ligne

```
    awk '{print NR,$0}' in.txt
    c=0; while read line; do ((c+=1)); echo $c $line; done < in.txt
```

compteur vertical

```
    for i in `seq 1 15`;do echo "$i";done
```

Compte le nombre de lignes vides d'un fichier

```
    awk '/^$/ {x += 1};END {print x }' in.txt
```

Compter le nombre de mots d'un fichier

```
    cat in.txt | wc -w
```

compter le nombre de lignes et de mots

```
    awk 'BEGIN{nl=0;nw=0} {nl++;nw+=NF} END {print "lines:",nl, "words:",nw}' in.txt
```

compter un caractere (ici: i)

```
    var=`cat in.txt` ; var="${var//[^i]/}" ; echo ${#var}
```

affiche le numero de ligne du pattern

```
    grep -n "pattern" in.txt
```

Compter le nombre d'occurrences de 'pattern'

```
    grep -c "pattern" in.txt
    awk '/pattern/ {n++} END {print n}' in.txt
```

Numeroter toutes les lignes contenant 'pattern' (affiche 1 numero par ligne)

```
    sed -n '/pattern/=' in.txt
```

Numeroter toutes les lignes entre 2 patterns (affiche 1 numero par ligne)

```
    sed -n '/pattern1/,/pattern2/{=;d;}' in.txt
```

Numeroter les lignes sans les lignes blanches

```
    nl in.txt
```

Numeroter les lignes avec les lignes blanches

```
    cat -n in.txt
    sed = in.txt | sed 'N; s/\n/\t/'
```

Numeroter les lignes

```
    awk '{print NR,$0}' in.txt
    while read line; do N=$((N+1)); echo "Line $N = $line"; done < in.txt
```

Ecrire le nom du fichier devant chaque ligne

```
    grep -H "pattern" in.txt
```

Somme et cumul d'une colonne ($2) , en utilisant 1 colonne clef ($1)

```
    awk '{arr[$1]+=$2} END {for(i in arr) {print i, arr[i]}}' in.txt | sort
```

### 2-Operations sur les champs : NF

Compter le nombre de champs de chaque ligne (separateur = ",")

```
    awk '{cnt=0 ; for(i=1; i<=NF; i++) {if($i != "") {cnt++}} {print NR " : "cnt" fields"}}' FS="," in.txt
```

Printer les 5 premiers caractères de toutes les lignes d'un fichier

```
    while read line;do echo ${line::5};done < in.txt
    while read line ; do echo $line | cut -c1-5 ; done < in.txt
```

Deleter les 5 premiers caracteres de toutes les lignes d'un fichier

```
    colrm 1 5 < in.txt
    awk 'sub("^.....", "")' in.txt
    while read line ; do echo $line | cut -c6- ; done < in.txt
```

Printer les 5 derniers caracteres de toutes les lignes

```
    sed 's/^.*\(.....\)$/\1/' in.txt
    sed 's/\(.*\)\(.\{5\}\)/\2/' in.txt
    while read line;do echo ${line: -5};done < in.txt
    awk '{print substr($0, length($0) - 4, length($0) ) }' in.txt
```

Deleter les 5 derniers caracteres de toutes les lignes

```
    awk 'sub(".....$", "")' in.txt
```

Supprimer un champ

```
    echo "data1 line1" | sed 's/.* //'
    echo "data1 line1" | sed -n 's/.* //;p'
    resultat: line1
```

Supprimer le dernier champ

```
    awk '{$NF=""; print $0}' in.txt
```

Champ tampon 'elapse' pour operations intermediaires

```
    awk '{elapse = $1/3600; if(elapse<8) print int($1/3600)}' in.txt
```

Dans champ 1, a la position 2, printer 3 caracteres

```
    awk '{print substr($1,2,3)}' in.txt
```

Print l'avant dernier champ ($NF-1) de chaque ligne

```
    awk '{print $(NF-1)}' in.txt
```

Print le nombre de champs uniquement

```
    awk '{print NF}' in.txt
```

Print le nombre maximum de champs

```
    awk '{print NF}' in.txt | sort -n | sed -n '$p'
```

Print le nombre minimum de champs

```
    awk '{print NF}' in.txt | sort -n | sed -n '1p'
```

Printer 2 colonnes en precisant le separateur

```
    awk -F'[ ]' '{print $2,$3}' in.txt
    cut -d ' ' -f2,3 in.txt
```

Printer du 5eme caractere au dernier (inclus) sur toutes les lignes d'un fichier

```
    cat in.txt | cut -c '5-'
```

Printer du 1er au 5eme caractere (inclus) sur toutes les lignes d'un fichier

```
    cat in.txt | cut -c '-5'
```

Printer du 3eme au 5eme caractere (inclus) et du 7eme au 9eme (inclus) sur toutes les lignes d'un fichier

```
    cat in.txt | cut -c '3-5,7-9'
```

Printer les lignes dont le nombre de champs est inferieur a 3

```
    awk 'NF<3' in.txt
```

Printer les lignes n'ayant qu'un seul champ

```
    awk '{if(NF == 1) {print}}' in.txt
```

tri de la 1ère colonne au 22eme caractere

```
    sort -k1.22
```

Print si longueur de 'colonne 1' >3 "ET" ou "OU" longueur de 'colonne 2' <5

```
    awk 'length($1)>3 && length($2)<5 {print}' in.txt ........... #ET
    awk 'length($1)>3 || length($2)<5 {print}' in.txt ........... #OU
```

Condition de print sur la longueur de colonne

```
    awk '{if(length($1)<2 && $1~/2/) {print $2} else {print $1}}' in.txt
```

### 3-operations sur les lignes : occurrences - digits - suppression - doubles - printer

Capturer la premiere occurrence d'une serie de lignes ayant meme pattern

```
    cat in.txt | sort -k1 | awk 'x !~ $1 ; {x = $1}'
    cat in.txt | sort -k1 | awk '!d[$1] {print} {d[$1]=1}'
    cat in.txt | sort -k1 | awk 'x[$1]++ {next} {print}'
    cat in.txt | sort -k1 | awk '!_[$1]++ {print $0 ; next} {next}'
```

Compter et marquer a la fin de la ligne les occurrences d'un unique pattern

```
    awk '/pattern/ {i=i+1} {print $0,i}' in.txt | awk '!a[NF]++ {print $0 ; next} {sub($NF,"") ; print}'
```

Compter les occurrences d'un pattern (total cumule)

```
    awk '/pattern/ {n++} END {print "pattern ecrit" n "fois"}' in.txt
```

Compter les occurrences d'un pattern (pour chaque ligne)

```
    awk -F "pattern" '{print NF-1}' in.txt
```

Remplace sur chaque ligne la 1ère occurrence de 't' par 'b'

```
    sed -e '1,$ s/t/b/1' in.txt
    while read line; do echo ${line/t/b}; done < in.txt
```

Remplacer la 2ème occurrence d'un pattern de la premiere ligne

```
    sed '0,/old/ s//new/2' in.txt
```

Remplacer la 2ème occurrence d'un pattern pour chaque ligne

```
    sed 's/old/new/2' in.txt
    sed '/old/ s//new/2' in.txt
    awk '{print gensub(/old/, "new", 2)}' in.txt
```

Print la 1ère occurrence d'un "pattern"

```
    grep -m1 "pattern" in.txt
    sed -n '/pattern/{p;q;}' in.txt
```

Printer les lignes dont les elements de la colonne 2 ont plus d'une occurrence

```
    awk 'FNR==NR && a[$2]++ {b[$2] ; next} $2 in b' in.txt in.txt
```

Printer les 4 premiers digits de chaque ligne

```
    cat in.txt | cut -c1-4
    for line in $(cat in.txt); do echo `expr "$line" : '\(....\)'`; done
    while read line; do echo `expr "$line" : '\(....\)'`; done < in.txt
```

Printer les 4 derniers digits de chaque ligne

```
    awk '{print substr($0, length($0)-3, length($0))}' in.txt
```

Deleter les 3 derniers digits de chaque ligne

```
    sed -n '1,$ s/...$//p' in.txt
```

Printer du 5ème digit au dernier de la ligne pour toutes les lignes

```
    while read line; do echo "substr($line,4)" | m4; done < in.txt
```

Printer 6 digits à partir du 2ème digit

```
    while read line; do echo `expr substr "$line" 2 6`; done < in.txt
```

Supprime de la ligne 4 a 7 (inclus) du fichier

```
    sed '4,7d' in.txt
```

Supprime les lignes contenant 'toto'

```
    sed '/toto/d' in.txt
    grep -v "toto" in.txt
```

Supprime 'toto' de la ligne 2 a 6

```
    sed -e '2,6 s/toto//g' in.txt
```

Supprime les lignes debutant par un chiffre (1 a 9)

```
    awk '$1 ~ /^[1-9]/ {next} {print}' in.txt
```

Supprime la ligne debutant par '@' et les 2 suivantes

```
    sed '/^@/ {N;N;d;}' in.txt
```

Supprimer la 1ere ligne , la derniere ligne ...

```
    sed '1d' in.txt ........... #supprime la premiere ligne
    sed '3d' in.txt ........... #supprime la ligne 3
    sed '$d' in.txt ........... #supprime la derniere ligne
```

Supprimer les lignes 1, 4, 7, 10.....

```
    sed -e '1~3d' in.txt
```

Supprimer 1 ligne toutes les 3 lignes

```
    sed '0~3d' in.txt
    sed 'n;n;d;' in.txt
```

Deleter les 2 dernieres lignes

```
    sed 'N;$!P;$!D;$d' in.txt
```

Deleter les 10 dernieres lignes

```
    sed -e :a -e '$d;N;2,10ba' -e 'P;D'
    sed -n -e :a -e '1,10!{P;N;D;};N;ba'
```

Printer les lignes uniques sans les doubles

```
    sort -u in.txt
    sort in.txt | uniq
    awk '!x[$0]++' in.txt
    awk '{ a[$1]++ } END {for (i in a) print i}' in.txt | sort
    sed '$!N; /^\(.*\)\n\1$/!P; D' in.txt .........consecutive lines
```

Printer les lignes doubles , deleter le reste

```
    awk 'x[$0]++' in.txt
    cat in.txt | uniq -d
    sed '$!N; s/^\(.*\)\n\1$/\1/; t; D' in.txt
```

Printer les lignes doubles (ou triples ...)

```
    awk 'FNR==NR && a[$0]++ {b[$0] ; next} $0 in b' in.txt in.txt
```

Printer uniquement la ligne 10

```
    sed '10q;d' in.txt
    sed '10!d' in.txt
    sed -n '10p' in.txt
    awk '{f[NR]=$0} END {print f[10]}' in.txt
    awk 'NR == 10 {print}' in.txt
```

Printer de la ligne 1 a 10

```
    sed 10q in.txt
    awk 'NR <=10{print}' in.txt
```

Printer de la ligne 3 a 5

```
    sed '3,5!d' in.txt
    sed -n '3,5p' in.txt
    awk 'NR >= 3 && NR <= 5' in.txt
    head -5 in.txt | tail -3
    sed -n '3{:a;N;5!ba;p}' in.txt
```

Printer la ligne 5 et 10 d'une serie de fichiers

```
    for i in fichiers*;do awk 'NR == 5;NR == 10 {print $0}' $i;done
```

Printer de la ligne 5 a 10 en numerotant

```
    awk 'NR == 5,NR == 10 {print NR" " $0}' in.txt
```

Printer la 1ere ligne à la place de la 3eme ligne

```
    sed -n -e '1h; 1!p; 3{g;p}' in.txt
```

Printer la 1ere ligne

```
    sed q in.txt
```

Printer la dernière ligne

```
    sed -n '$p' in.txt
    sed '$!d' in.txt
```

Printer les 2 dernieres lignes

```
    sed '$!N;$!D' in.txt
```

Printer les 10 dernieres lignes

```
    sed -e :a -e '$q;N;11,$D;ba' in.txt
```

Printer la 1ere et derniere ligne

```
    head -1 in.txt ; tail -1 in.txt
    sed -n '1p ; $p' in.txt
    awk 'NR==1 ; END {print}' in.txt
    sed q in.txt;sed '$!d' in.txt
    sed q in.txt;sed '$\!d' in.txt ..........#selon version de Linux
    IFS=$'\n';array=($(cat in.txt)); echo ${array[0]};sed '$!d' in.txt ........#en sous shell
```

Printer les lignes ayant moins de 6 caracteres

```
    sed '/^.\{6,\}/d' in.txt
```

Printer les lignes de 6 caracteres ou plus

```
    sed -n '/^.\{6\}/p' in.txt
```

Indexage du premier 't' lu pour chaque ligne

```
    while read line; do echo `expr index "$line" t`; done < in.txt
```

Longueur de chaque ligne (en nombre de digits) - voir si la version de Linux supporte : ' m4 '

```
    while read line; do echo "len($line)" | m4; done < in.txt
    while read line; do echo `expr length "$line"`; done < in.txt
```

Formater sur une meme ligne : une ligne paire a droite d'une ligne impaire

```
    cat in.txt | sed "N;s/\(.*\)\n\(.*\)/\1 \2/"
    cat in.txt | sed "N;s/\n/ /"
    cat in.txt | sed '$ !N; s/\n/ /'
```

Formater sur une meme ligne : une ligne impaire a droite d'une ligne paire

```
    cat in.txt | sed "N;s/\(.*\)\n\(.*\)/\2 \1/"
```

Affiche 1 ligne sur 2 (lignes 1, 3, 5...)

```
    sed 'n;d' in.txt
    sed -n 'p;n' in.txt
    sed -n '1,${p;n;}' in.txt
    sed '2~2d' in.txt
    awk 'FNR % 2' in.txt
    awk 'NR%2 {print}' in.txt
    awk 'NR%2 == 1' in.txt
```

Affiche 1 ligne sur 2 (lignes 2, 4, 6...)

```
    sed -n 'n;p' in.txt
    sed -n '1,${n;p;}' in.txt
    sed '1~2d' in.txt
    awk '!(FNR % 2)' in.txt
    awk '(NR+1)%2 {print}' in.txt
    awk 'NR%2 == 0' in.txt
```

Affiche 1 ligne sur 5 a partir de la ligne 3

```
    sed -n '3,${p;n;n;n;n;}' in.txt
```

Recherche de la ligne la plus longue

```
    awk '{ if ( length > L ) { L=length ; s=$0 } } END { print L,"\""s"\""}' in.txt
```

### 4-pattern

Capture d'un pattern dans 1 fichier

```
    grep 'pattern' in.txt
    grep -w 'pattern' in.txt
    awk '/pattern/' in.txt
    awk '$0 ~ /\ypattern\y/ {print}' in.txt
    sed -n '/pattern/p' in.txt
    sed '/pattern/ !d' in.txt
    grep '\<pattern\>' in.txt
```

Printer les lignes ne contenant que des chiffres

```
    sed -n '/^[[:digit:]]*$/p' in.txt
```

Printer les lignes ne contenant que des lettres

```
    sed -n '/^[[:alpha:]]*$/p' in.txt
```

Capture d'un pattern dans plusieurs fichiers

```
    grep "pattern" in*.txt
    for i in in*.txt;do seq=`ls $i`;awk '/pattern/ {print seq,$0}' seq=${seq} $i;done
```

Printer un paragraphe separe par des lignes blanches , contenant un pattern

```
    sed -e '/./{H;$!d;}' -e 'x;/pattern/!d;' in.txt
```

Printer un paragraphe separe par des lignes blanches , contenant pattern1 'et' pattern2

```
    sed -e '/./{H;$!d;}' -e 'x;/pattern1/!d;/pattern2/!d' in.txt
```

Printer un paragraphe separe par des lignes blanches , contenant pattern1 'ou' pattern2

```
    sed -e '/./{H;$!d;}' -e 'x;/pattern1/b' -e '/pattern2/b' -e d in.txt
```

Capturer dans la colonne 1 le pattern '2' et printer la colonne 3

```
    awk '$1 ~ /2/ {print $3}' in.txt
    awk '$1 == "2" {print $3}' in.txt
    awk '{if($1 ~ /t/){gsub(/t/, "z",$1);print $0}}' in.txt
    awk '$1 !~ /2/ {print $3}' in.txt ........... #syntaxe inverse
    awk '$1 \!~ /2/ {print $3}' in.txt ...........#syntaxe inverse (selon version Linux)
```

Supprimer les lignes contenant des patterns

```
    awk '$1 !~ /pattern1/ && $2 !~ /pattern2/ ' in.txt
    awk '$1 ~ /pattern1/ || $2 ~ /pattern2/ {next} {print}' in.txt
```

Capturer le caractère # à la 4ème position

```
    egrep '^.{3}#' in.txt
```

Capturer les lignes commencant par 1 espace ou plus, sans celles commencant par 2 espaces

```
    grep "[ ]\{1\}" in.txt | awk '$0 !~ /^ / {print}'
```

Capturer les lignes ayant 4 chiffres ou plus

```
    grep "[0-9]\{4\}" in.txt
```

Lire plusieurs patterns - (possibilite de lignes doubles)

```
    awk '/pattern1/ {print $0} /pattern2/ {print $0}' in.txt
    awk 'FNR==NR && a[$0]=/^t/ || a[$0]=/^d/ {b[$0] ; next} $0 in b' in.txt in.txt
```

utiliser 'egrep' si le pattern a plusieurs lignes

```
    egrep -a "MIN WORD 2|MIN WORD32" in.txt
```

ajouter antislash `\` pour le caractère special: `|`

```
    egrep "3 \|DISK|4 \| DISK" in.txt
```

pattern1 OR pattern2

```
    sed '/[pattern1pattern2]/!d' in.txt
    sed -e '/pattern1/b' -e '/pattern2/b' -e d in.txt
    awk '/pattern1|pattern2/' in.txt
    grep -E "pattern1|pattern2" in.txt
    grep -e pattern1 -e pattern2 in.txt
```

pattern1 AND pattern2

```
    sed '/a/!d; /b/!d' in.txt
    sed '/pattern1.*pattern2/!d in.txt
    awk '/pattern1.*pattern2/' in.txt
    awk '/pattern1/ && /pattern2/' in.txt
    grep -E 'pattern1.*pattern2' in.txt
```

NOT pattern1

```
    grep -v 'pattern1' in .txt
    awk '!/pattern1/' in.txt
    sed -n '/pattern1/!p' in.txt
```

Printer les 2 premieres occurrences du pattern

```
    grep -m2 "tata" in.txt
```

Substitution uniquement pour la 1ere occurrence

```
    sed '0,/tata/ s//zaza/' in.txt
```

Remplace 'old' par 'new' uniquement sur les lignes commencant par %%

```
    sed '/^%%/ s/old/new/g' in.txt
```

Remplace sur chaque ligne du début de la ligne au signe '=' par 'new'

```
    sed -e '1,$ s/^.*=/new/' in.txt
```

Supprimer la ligne contenant 'toto' entre 'titi' et 'tutu'

```
    sed -e '/titi/,/tutu/ {/toto/d}' in.txt
```

Capture entre 'toto' et 'tata' (attention si plusieurs occurrences du pattern)

```
    sed -n '/toto/,/tata/p' in.txt
    sed -e '/toto/,/tata/ !{/./d}' in.txt
    sed -e '/toto/,/tata/ \!{/./d}' in.txt
    perl -e "while(<>) {print if/toto/.../tata/}" in.txt
    awk "/toto/,/tata/" in.txt
```

Supprimer entre 'pattern1' et 'pattern2' (les patterns inclus)

```
    sed -e '/pattern1/,/pattern2/d' in.txt
```

Supprimer d'un debut de fichier a un pattern

```
    sed -e '1,/pattern/d' in.txt
```

Supprimer d'un pattern a la fin d'un fichier

```
    sed -e '/pattern/,$d' in.txt
```

Substitution entre 2 patterns

```
    sed '/titi/,/tata/ s/toto/zz/g' in.txt
```

Definir un pattern sur lequel la substitution ne se fera pas

```
    sed '/toto/!s/t/z/g' in.txt
```

Printer entre 'titi' et 'tutu'

```
    sed -n '/titi/{:a;N;/tutu/!ba;p;}' in.txt
```

Capture de 'titi' a 'tutu' de la ligne 3 a 10

```
    sed -n '3,10{/titi/,/tutu/p}' in.txt
```

Supprimer une ligne contenant un pattern + les 2 lignes suivantes

```
    sed '/pattern/,+2d' in.txt
```

Supprimer de la ligne 3 a la ligne contenant 'pattern'

```
    sed -e '3,/pattern/d' in.txt
```

Printer d'une ligne contenant 1 pattern jusqu'a la fin

```
    sed -n '/pattern/,$p' in.txt
    sed -n '/pattern/,EOF' in.txt
```

Si le pattern est une variable

```
    sed -n '/'$var'/p' in.txt
```

Ne pas selectionner les lignes contenant un pattern

```
    grep -v "pattern" in.txt
    awk '!/pattern/' in.txt
    sed '/pattern/d' in.txt
    sed -n '/pattern/!p' in.txt
    awk '$0 ~ /pattern/ {next} {print}' in.txt
```

A un pattern inserer une ligne

```
    sed -e '/pattern/ i\ligne ecrite avant le pattern' in.txt ............ (option 'i' : avant le pattern)
    sed -e '/pattern/ a\ligne ecrite apres le pattern' in.txt ............ (option 'a' : apres le pattern)
```

A un pattern inserer une ligne blanche

```
    sed -e '/pattern/ i\ ' in.txt ............ (option 'i' : avant le pattern)
    sed -e '/pattern/ a\ ' in.txt ............ (option 'a' : apres le pattern)
```

print 2 lignes après un pattern ( dans la colonne 1 )

```
    awk '/^pattern/ {c=2; next} c-->0' in.txt
    awk 'BEGIN {counter=0}; $1=="pattern" {counter=2; next}; counter>0 {counter--; print}' in.txt
```

print le pattern + 2 lignes après (After)

```
    grep -A2 "pattern" in.txt
    sed -n '/pattern/ {N;N;p;}' in.txt
```

delete le pattern + 2 lignes après

```
    sed '/pattern/ {N;N;d;}' in.txt
```

print le pattern + 2 lignes avant (Before)

```
    grep -B2 "pattern" in.txt
```

selectionner tous les caracteres (sans afficher les lignes blanches)

```
    grep "." in.txt
    awk "/./" in.txt
    sed -n '/./ {p;d}' in.txt
```

combiner motif et ligne

```
    sed '8,/fin/ s/toto/titi/g' in.txt
    sed '/debut/,$ s/toto/titi/g' in.txt
```

substituer tout un texte entre 2 motifs excluant les motifs

```
    sed '/titi/,/tutu/{/titi/b;/tutu/b;s/.*/SED/;}' in.txt
```

ne pas printer les lignes contenant 'tata'

```
    sed '/tata/d' in.txt
    sed -n '/tata/!p' in.txt
```

Printer les lignes precedant un pattern

```
    sed -n '/tata/{g;1!p;};h' in.txt
```

Printer les lignes suivant un pattern

```
    sed -n '/tata/{n;p;}' in.txt
```

Remembering a pattern ('_' is delimiter)

```
    echo "a line1" | sed 's_\([a-z]\)\([ ]\)\([a-z]*\)\([0-9]\)_\1\2\3 \4_'
    (result: a line 1)
```

### 5-Remplacer des lignes des chiffres ou des lettres

Exemple d'une suite d'actions entre 2 patterns

```
    sed -n '4,10 {/pattern1/,/pattern2/ {s/^0./y&/;/^$/d;s/m/w/g;p}}' in.txt
```

Supprimer tous les chiffres en gardant un tiret '-' a la place

```
    tr -d 0-9 < in.txt
```

supprimer toutes les lettres en gardant un tiret '-' a la place

```
    tr -d [a-zA-Z] < in.txt
```

Conserver les lignes contenant des chiffres

```
    sed -e '/[0-9]/!d' in.txt
    sed -e '/[0-9]/\!d' in.txt ........... (selon version Unix)
```

Changer un texte majuscule en minuscule

```
    awk '{print tolower($0)}' in.txt
    cat in.txt | tr -s A-Z a-z
```

Changer un texte minuscule en majuscule

```
    cat in.txt | tr -s a-z A-Z
```

Supprimer les lettres minuscules en fin de ligne

```
    awk '{ sub("[a-z]*$", ""); print }' in.txt
```

print le pattern sans distinguer majuscules ou minuscules

```
    grep -i "pattern" in.txt
```

Mettre en majuscule la 1ère lettre d'une phrase

```
    echo -e "texte ligne1\ntexte ligne2" | sed 's/^./\u&/'
```

Remplacer les lettres par une operande (ici: +)

```
    cat in.txt | tr '[:alpha:]' +
```

Remplacer les chiffres par une operande (ici: +)

```
    cat in.txt | tr '[:digit:]' +
```

Remplacer 1 caractere par un autre

```
    tr "t" "z" < in.txt
```

Suppression des sauts de lignes

```
    tr '\n' ' ' < in.txt
```

Supprimer la répétition de caractères

```
    echo "boonnjoouuur" | tr -s "onu"
```

Remplace le bloc ('t' suivi de 2 caracteres) par zorro en ligne 4

```
    sed -e '4s/t../zorro/g' in.txt
```

Inserer 1 caractere (1 point=1 caractere)

```
    ( ici l'insertion de 'Q' se fera apres les 2 premiers caractères )
    echo 'abcdef' | sed 's/^../&Q/' ........... #resultat : abQcdef
```

Remplacer en debut de chaine un nombre de points par une lettre

```
    echo 'abcdef' | sed 's/^../Q/' ........... #resultat : Qcdef
```

Uniquement le 2eme caractere 't' est remplace par 'z' en ligne 4

```
    sed -e '4s/t/z/2' in.txt
    awk 'NR==4 {print gensub(/t/,"z",2)}; NR!=4 {print}' in.txt
```

Remplace 't' par 'k' , et 'o' par 'l'

```
    sed 'y/to/kl/' in.txt
```

Effectue le remplacement des lignes 4 a 10 et n'ecrit dans 'out.txt' que celles modifiees

```
    sed -e '4,10 s/t/&zorro/gw out.txt' in.txt
```

Remplacer un pattern ('titi' remplace par: 'titi et tata')

```
    sed '/titi/ s//& et tata/g' in.txt
    sed 's/titi/titi et tata/g' in.txt
```

tr : remplacement dans un sous shell (taper: sh)

```
    a=abcdef; echo $a | tr f g ........... #resultat : abcdeg
    a=abcdef; echo ${a//f/g} ........... #resultat : abcdeg
    a=abcdef; echo $a | tr [ac] [xz] ........... #resultat : xbzdef
```

tr + option -d: effacement (utiliser un sous shell) (taper: sh)

```
    a=abcdef; echo $a | tr -d f ........... #resultat : abcde
    a=abcdef; echo ${a//f/} ........... #resultat : abcde
    a=abcdef; echo $a | tr -d [a-c] ........... #resultat : def
    a=abcdef; echo ${a//[a-c]/} ........... #resultat : def
    a=abcdef; echo $a | tr -d [ac] ........... #resultat : bdef
    a=abcdef; echo ${a//[ac]/} ........... #resultat : bdef
```

tr + option -c: inverse l'ensemble des caractères a detecter

```
    echo "acfdeb123" | tr -c b-d + ....... resultat: +c+d+b++++
```

Remplacer toutes les occurrences d'un caractère ou d'un pattern pour chaque ligne

```
    var="newpattern" ; awk '{gsub( /oldpattern/, "'"$var"'" )};1' in.txt
    var="newpattern" ; awk -v v="$var" '{gsub( /oldpattern/, v )}1' in.txt
```

Remplacer la 2eme occurrence d'un caractère ou d'un pattern pour chaque ligne

```
    awk '{print gensub(/old/, "new", 2) }' in.txt
```

Remplacer 'o' par 'zorro' sauf pour le pattern 'toto' ,

```
    sed -e '/toto/!s/o/zorro/' in.txt
```

Supprimer les 3 derniers caractères de la dernière ligne uniquement

```
    expr "$(cat in.txt)" : "\(.*\)...$"
```

Supprimer les 3 derniers caractères de chaque ligne

```
    awk 'sub( "...$", "" )' in.txt
```

Printer les 5 premiers caractères de la premiere ligne

```
    echo `cat in.txt`| cut -c1-5
    echo `expr "$(cat in.txt)" : '\(.....\)'`
```

Printer les 5 premiers caractères de toutes les lignes d'un fichier

```
    cat in.txt | cut -c1-5
    while read line;do echo ${line::5};done < in.txt
    while read line ; do echo $line | cut -c1-5 ; done < in.txt
```

Printer les 5 derniers caracteres de toutes les lignes d'un fichier

```
    while read line;do echo ${line: -5};done < in.txt
    sed 's/^.*\(.....\)$/\1/' in.txt
    sed 's/\(.*\)\(.\{5\}\)/\2/' in.txt
```

Substituer toto ou titi par tata

```
    sed 's/toto\|titi/tata/g' in.txt
```

### 6-Supprimer des lignes blanches ,espaces ,tabulations, contenu entre 2 balises (balises incluses)

Supprimer contenu entre 2 balises (balises incluses)  
Exemple ,supprimer le contenu entre `<ele>` et `</time>` d'un fichier nommé test.gpx

```
<trkpt lat="47.0134660788" lon="-1.1192788649"><ele>50.61</ele><time>2020-07-07T04:24:19Z</time></trkpt>
<trkpt lat="47.0134599600" lon="-1.1192838103"><ele>50.13</ele><time>2020-07-07T04:24:42Z</time></trkpt>
<trkpt lat="47.0134515781" lon="-1.1192932818"><ele>49.65</ele><time>2020-07-07T04:25:01Z</time></trkpt>
```

La commande

    sed "s/<ele>.*<\/time>//" test.gpx  # sed -i pour récréer le fichier

```
<trkpt lat="47.0134660788" lon="-1.1192788649"></trkpt>
<trkpt lat="47.0134599600" lon="-1.1192838103"></trkpt>
<trkpt lat="47.0134515781" lon="-1.1192932818"></trkpt>
```

Supprimer les lignes blanches (l'option '-i' reecrit directement dans le fichier)

```
    sed '/./!d' in.txt
    sed -i '/^$/d' in.txt ........... (l'option '-i' est a manier avec precaution)
```

Supprimer les lignes blanches repetees sauf la 1ere

```
    cat -s in.txt
```

Supprimer uniquement les lignes blanches du debut du fichier

```
    sed '/./,$!d' in.txt
```

Supprimer tout ce qui suit la 1ere ligne blanche

```
    sed '/^$/q' in.txt
```

Supprimer tout ce qui precede la 1ere ligne blanche

```
    sed '1,/^$/d' in.txt
```

Supprimer les lignes blanches entre 'tata' et 'route'

```
    sed -e '/tata/,/route/ {/^$/d}' in.txt
```

Remplacer 2 blancs (ou +) par 1 seul blanc

```
    sed 's/\ \ */\ /g' in.txt
```

Suppression des espaces et tabulations en debut et fin de ligne

```
    sed 's/^[ \t]*//;s/[ \t]*$//' in.txt
```

Suppression des lignes vides

```
    grep -v '^$' in.txt
    grep '.' in.txt
    sed -n '/^$/!p' in.txt
    awk NF in.txt
    awk '/./' in.txt
    sed '/^$/d' in.txt
    sed '/./!d' in.txt
    awk '/^$/ {next} {print}' in.txt
```

tr + option squeeze-repeats: efface tout sauf la première

```
    occurence d'une chaîne de caractères
    (utile pour supprimer plusieurs espaces blancs)
    echo "XXXXX" | tr --squeeze-repeats 'X' ....... resultat: X
```

Supprime tous les espaces au début de toutes les lignes

```
    sed 's/^ *//g' in.txt
```

Supprime tous les espaces à la fin de toutes les lignes

```
    sed 's/ *$//g' in.txt
```

Supprimer seulement la première ligne de chaque ensemble de lignes vides consecutives

```
    sed '/[0-9A-Za-z]/,/^$/{/^$/d}' in.txt
```

to join lines ( en deletant les lignes blanches )

```
    sed -e '/./!d' -e '$!N;s/\n/ /' in.txt
    sed -e '/./\!d' -e '$\!N;s/\n/ /' in.txt
    grep "." in.txt | sed '$\!N;s/\n/ /'
```

to join lines ( en gardant les lignes blanches )

```
    sed -e '/./!b' -e '$!N;s/\n/ /' in.txt
    sed -e '/./\!b' -e '$\!N;s/\n/ /' in.txt
    grep "." in.txt | sed '$\!N;s/\n/ /' | sed G
```

### 7-Inserer

Inserer une ligne blanche apres chaque ligne

```
    sed G in.txt
    sed 'a\ ' in.txt
```

Inserer une ligne de tirets toutes les 2 lignes

```
    sed 'n;a\----------' in.txt
```

Inserer une ligne blanche toutes les 3 lignes

```
    sed 'n;n;G;' in.txt
```

Inserer une ligne blanche apres chaque ligne sauf apres la ligne 3

```
    sed '3!G' in.txt
```

Inserer une ligne au debut, à la 3eme ligne et à la fin du fichier

```
    sed -e '1i \debut\ du\ traitement' in.txt
    sed -e '3i \ajout\ a\ la\ 3eme\ ligne' in.txt
    awk 'NR == 3 {print "line3"}1' in.txt
    sed -e '$a \fin\ du\ traitement' in.txt
```

Inserer une ligne à un pattern

```
    sed -e '/pattern/ i\ligne ecrite avant le pattern' in.txt ............ (option 'i' : avant le pattern)
    sed -e '/pattern/ a\ligne ecrite apres le pattern' in.txt ............ (option 'a' : apres le pattern)
```

Inserer une ligne blanche à un pattern

```
    sed -e '/pattern/ i\ ' in.txt ............ (option 'i' : avant le pattern)
    sed -e '/pattern/ a\ ' in.txt ............ (option 'a' : apres le pattern)
```

Changer la ligne si elle contient un pattern

```
    sed -e '/pattern/ c\new line' in.txt
```

Inserer **DELETED** et supprimer les lignes entre 2 patterns

```
    sed '/pattern1/,/pattern2/ c\**DELETED**' in.txt
```

Inserer la ligne 'line before' avant chaque ligne du fichier

```
    sed -e 'i\line before' in.txt
```

Inserer du texte avant une ligne matchée par un pattern

```
    sed -e '/pattern/ i\line before pattern' in.txt
```

Inserer une ligne blanche avant une ligne matchée par un pattern

```
    sed '/tata/{x;p;x}' in.txt
    sed -e '/pattern/ i\ ' in.txt
```

Inserer une ligne blanche après une ligne matchée par un pattern

```
    sed '/tata/G' in.txt
    sed -e '/pattern/ a\ ' in.txt
```

Inserer une ligne blanche avant et apres une ligne matchee par un pattern

```
    sed '/tata/{x;p;x;G}' in.txt
```

Inserer un fichier 'temp.txt'

```
    sed '1r temp.txt' < in.txt ..........après la 1ère ligne de 'in.txt'
    sed '/pattern/ r temp.txt' < in.txt .........apres le pattern
```

Inserer un 'blanc' devant toutes les lignes

```
    sed -e 's/^./ &/g' in.txt
    sed '{s_^_ _}' in.txt
```

Inserer un 'blanc' à toutes les fins de lignes

```
    sed -e 's/.$/& /g' in.txt
```

Inserer un 'blanc' apres tous les 't' (a droite) en ligne 4

```
    sed -e '4s/t/& /g' in.txt
    sed -e '4\!s/t/& /g' in.txt ........... #syntaxe inverse
```

Inserer au 5ème caractère après 3 caractères le signe # sur la ligne 2

```
    sed -r "2 ~ s/^(.{4})(.{3})/\2#/" in.txt
```

### 8-Remplacer un MOT par un AUTRE dans un ou plusieurs fichiers

REMPLACER UN MOT PAR UN AUTRE :

    sed -i 's/motachercher/nouveaumot/g' fichier.txt

REMPLACER UN MOT PAR UN AUTRE DE MANIERE RECURSIVE :

    find /home/mumbly/MONREP/sousrep -type f -exec sed -i 's/windows/linux/g' {} +

RECHERCHER UN MOT DANS TOUS LES REPERTOIRES ET SOUS-REPERTOIRES (insensible à la casse : majuscule/minuscule) :

    grep -i -l -r 'linux' /home/mumbly/MONREP/sousrep/ 

Comment chercher et remplacer dans plusieurs fichiers avec sed

Pour remplacer "echo htmlentities" par "highlight_string" dans tous les fichiers php du répertoire courant tapez ceci :

    sed -i 's/echo htmlentities/highlight_string/g' *.php

Alternative pour un chercher remplacer récursif :

    find . -name "*.php" -exec sed -i 's/echo htmlentities/highlight_string/g' {} \;


### 9-divers

Retirer les accents d'un texte

```
    cat non-ascii.txt | iconv -f utf8 -t ascii//TRANSLIT//IGNORE > ascii.txt
```

Printer les lignes d'un fichier2 qui ne sont pas dans le fichier1

```
    comm -23 file2.txt file1.txt 2>dev
    grep -vxFf file1.txt file2.txt
```

Printer un fichier

```
    cat in.txt
    sed '' in.txt
    sed ':' in.txt
```

Dupliquer toutes les lignes

```
    sed 'p' in.txt
```

printer 3 fois chaque ligne

```
    sed '{h;p;p}' in.txt
    while read line;do for i in `seq 1 3`;do echo $line;done;done < in.txt
```

Inverser l'ordre d'un fichier

```
    tac in.txt
    sed -n '1!G;h;$p' in.txt
    awk '{ a[i++]=$0 } END { for (j=i-1; j>=0; ) print a[j--] }' in.txt
```

Ecrire 1 mot par ligne (pour 5 mots: -n5)

```
    cat in.txt | xargs -n1
    awk '{i=1; while (i<=NF){print $i, " ";i++}}' in.txt
```

Ecrire un caractère par ligne

```
    while read line; do echo -n "$line" | dd cbs=1 conv=unblock 2>/dev/null; done < in.txt
```

Ecrire un fichier sur une ligne

```
    sed '{:a;$!N;s_\n_ _;ta}' in.txt
```

Joindre les lignes paires a la suite des lignes impaires

```
    sed '$!N; s/\n/ /g' in.txt
```

Transforme les lignes en colonnes

```
    awk '{printf "ligne%d: %s ",NR,$0>"z-cible"}' z-source
```

Ajouter en prefixe le nombre d'occurences des mots et trier

```
    cat in.txt | xargs -n1 | sort | uniq -c | sort -nr
```

Print les 10 premiers (ou derniers) caracteres d'un fichier

```
    head -10c in.txt ........... #les 10 premiers
    tail -10c in.txt ........... #les 10 derniers
```

Decoupe en n caracteres le fichier 'in.txt' (ici n=10)

```
    creation de fichiers: prefixe_outaa (outab..outac...)
    split -b 10 in.txt prefixe_out
```

Decoupe en n lignes le fichier 'in.txt' (ici n=5)

```
    creation de fichiers: prefixe_outaa (outab..outac...)
    split -l 5 in.txt prefix_out
    awk '{print >("prefix_out" int((NR+4)/5))}' in.txt
```

Trier un fichier dans un ordre numerique (-n); avec separateur (-t); colonne (-k); et place du caractere (.)

```
    cat in.txt | sort -n -t" " -k2.4
    dans un ordre decroissant (-r) et en retirant les doubles (-u) :
    cat in.txt | sort -r -u
```

Copier un fichier

```
    cp old_file new_file
    sed 'w new_file' old_file
```

Definir un marqueur en fin de ligne (ici ,) qui joindra la ligne suivante

```
    sed '/\,$/ {N; s_\,\n_ _}' in.txt
    sed -e :a -e '/\,$/N; s_\,\n_ _; ta' in.txt
```

Si 1 ligne se termine par ',' joindre la suivante a elle

```
    sed -e :a -e '/,$/N ; s#\n## ; ta' in.txt
```

Si 1 ligne commence par un signe egale '=' , l'ajouter a la ligne precedente et remplacer le signe egale '=' par un espace

```
    sed -e :a -e '$!N;s/\n=/ /;ta' -e 'P;D' in.txt
```

"Dispach" (inverse du "regroupe")

```
    split en couples: ['colonne 1'-'parties[k]']
    en 'k' fois nombre de lignes
    awk '{key=$1;$1="";n=split($0, parties, "[,]");for(k=1; k<=n; k++) print ""key" "1" "parties[k] ""}' in.txt| awk '{if($2>0) print $0}' | awk '{if($1 != key){key = $1} else {$2 += cum} cum=$2; print}' > out.txt
```

"Regroupe" (inverse du "dispach")

```
    (tous les elements semblables de la colonne 1
    sont regroupes sur 1 seule ligne)
    awk '{key=$1; $1=$2="";f[key]=f[key] s[key] $0;s[key]=","} END {for(key in f){gsub(/[[:space:]]/,"",f[key]);printf "%s %s\n",key,f[key]}}' in.txt| sort > out.txt
```

Encadre le premier nombre de la ligne avec des ** , ne printer que ces lignes

```
    sed -n "s/\([0-9][0-9]*\)/**\1**/p" in.txt
```

Printer les lignes avec 3 digits consecutifs

```
    sed -n '/[0-9]\{3\}/p' in.txt
```

Inserer un espace entre chaque lettre

```
    echo -e "bonjour"|sed 's/./& /g' ................resultat: b o n j o u r
    echo -e "bonjour"|sed -r 's/([^ ])/\1 /g'
    echo -e "bonjour"|sed 's/\([^ ]\)/\1 /g'
```

dirname ............(en sous shell)

```
    var="/home/Bureau/1/in.txt" ....... (en sous shell)
```

dirname $var ........... ----> /home/Bureau/1

```
    echo ${var%/*} ........... ----> /home/Bureau/1
```

basename ....... (en sous shell)

```
    basename $var ........... ----> in.txt
    echo ${var##*/} ........... ----> in.txt
```

while dans un sous shell (taper: sh)

```
    while read line; do echo -en "$line\n"; done < in.txt
    while read line; do echo -e $line; done < in.txt
```

'set' decoupe 1 variable en parametres positionnels dans un sous shell (taper: sh)

```
    string="a b:c def:g"; IFS=':'; set $string; echo "$1"
```

read ...$REPLY : equivalent de : head -1

```
    read -r
```

Contenu des dossiers sans les sous dossiers

```
    for fichier in *;do ls -al ;done
```

Contenu des dossiers avec les sous dossiers

```
    for fichier in *;do ls -al "$fichier";done
```

nombre de fichiers dans le repertoire courant (equivaut a: ls|wc -l)

```
    a=0;for i in *;do a=$(($a+1));done;echo nb=$a
```

apostrophe et guillemet

```
    echo -n "your name is: "; read name ........... #taper: toto
    echo 'hi $name' ........... #resultat : hi name
    echo "hi $name" ........... #resultat : hi toto
```

par defaut, le separateur est 'blanc' [Utiliser les parentheses () pour le sous shell]

```
    echo "moi et moi, lui, les autres" | (read x y ;echo $y) ........... (->lui les autres)
```

nouveau separateur

```
    IFS=",";echo "moi et moi, lui, les autres" | (read x y;echo $x) ........... (->moi et moi)
```

liste des variables d'environnement

```
    export
```

detruire la valeur d'une variable

```
    (utiliser un sous shell en tapant: sh + return)
    var=7 echo $var ........... -> 7
    unset var ........... -> nothing
```

Concatenation (utiliser un sous shell en tapant: sh + return)

```
    var=debut;echo ${var}ant ........... ---->debutant
```

Commande 'set' et 'shift' (utiliser un sous shell en tapant: sh + return)

```
    c="prof eleve classe note";set $c;echo $1 $2
    ........... --->prof eleve
    c="prof eleve classe note";shift;echo $1 $2
    ........... --->eleve classe
```

Commande 'eval' (utiliser un sous shell en tapant: sh + return)

```
    message="date d'aujourd'hui?";set $message;echo $# .......#resultat: 2
    message="date d'aujourd'hui?";set $message;echo $1 .......#resultat: date
    message="date d'aujourd'hui?";set $message;eval $1 .......#resultat: dim 20 jan..
```

### NOTE sed

>Les exemples de **sed** ne font que modifier l'affichage du fichier (sortie standard 1 = l'écran).  
Pour des modifications permanentes avec les anciennes versions (inférieure à 4) utiliser un fichier temporaire  
Pour GNU sed utiliser le paramètre **-i[suffixe]** (**--in-place[=suffixe]**), comme dans l'exemple suivant `sed -i".bak" '3d' monFichier.txt` qui aura pour effet, de ne produire aucun affichage sur la sortie standard, de modifier le fichier original *monFichier.txt* en supprimant la 3ème ligne et de créer un fichier de sauvegarde nommé *monFichier.txt.bak*   
