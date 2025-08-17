+++
title = 'Bash commandes utiles'
date = 2020-10-29 00:00:00 +0100
categories = ['bash', 'cli']
+++
### Convertir un fichier WebP en JPG ou PNG

Le format WebP est un format d'image que l'on retrouve de plus en plus sur le net. Si pour une raison ou une autre vous désirez convertir un fichier .webp en .jpg ou en .png, voici les commandes à lancer dans un terminal :

    ffmpeg -i fichier-source.webp fichier-de-sortie.jpg

Ci-dessous un exemple avec un fichier se nommant "Julie.webp" que l'on veut en PNG

    ffmpeg -i Julie.webp Julie.png

L'excellente application "ffmpeg" doit bien sûr être installée sur votre système GNU/Linux.

### Convertir un fichier Wav en Mp3

Grâce au guillemets le nom du fichier peut contenir des apostrophes, des espaces etc.

    lame "nom du fichier.wav" "nom du fichier.mp3"

Ou si l'on veut suivre les options recommandées par les développeurs de lame :

    lame -V2 "mon fichier.wav" "mon fichier.mp3"

Il faut naturellement que le paquet "Lame" soit préalablement installée sur votre système GNU/Linux !  
Par défaut Lame encode à 128 kbps. L'option -V2 fixe l'encodage à ± 177 kilobits/seconde.

### Convertir plusieurs fichiers Wav d'un même dossier en fichiers Mp3

    for f in *.wav; do lame -V2 "$f" "`basename "$f" .wav`".mp3; done

Cette commande s'inspire des deux précédentes. Elle est à lancer depuis le dossier où se trouvent les fichiers Wav à convertir. Les fichiers Mp3 ainsi créés porteront le même nom que les Wav.

### Convertir plusieurs fichiers (stéréo) Wav d'un même dossier en fichiers Ogg

    for i in *.wav; do ffmpeg -ac 2 -channel_layout stereo -i "$i" -acodec libvorbis "${i%wav}ogg"; done;

Cette commande est à lancer depuis le dossier où se trouvent les fichiers Wav à convertir. Les fichiers Ogg ainsi créés porteront le même nom que les Wav originaux.

### Convertir plusieurs fichiers (stéréo) Flac d'un même dossier en fichiers Mp3

    for a in *.flac; do ffmpeg -i "$a" -qscale:a 0 "${a[@]/%flac/mp3}"; done

Cette commande est à lancer depuis le dossier où se trouvent les fichiers Flac à convertir. Les fichiers Mp3 ainsi créés porteront le même nom que les Flac originaux.

### Convertir tous les Wma (sans DRM) d'un répertoire en Wav

    for i in *.wma; do mplayer -ao pcm:file="${i%.wma}.wav" "$i"; done

"Mplayer" doit être installé pour lancer cette ligne de commande.

### Convertir tous les Wma (sans DRM) d'un répertoire en Mp3

    for i in *.wma; do mplayer -ao pcm:file="${i%.wma}.wav" "$i"; lame -h "${i%.wma}.wav" "${i%.wma}.mp3"; rm -f "${i%.wma}.wav"; done

Les paquets "Mplayer" et "Lame" doivent être installées pour lancer cette ligne de commande.

Remarquez que l'on passe par des fichiers temporaires WAV supprimés du répertoire en fin de boucle.  
L'option -h pour lame permet d'utiliser un bitrate de meilleure qualité, le traitement sera juste un plus long.

### Convertir tous les Wma (sans DRM) d'un répertoire en Ogg

    for i in *.wma; do mplayer -ao pcm:file="${i%.wma}.wav" "$i"; oggenc -m 256 "${i%.wma}.wav"; rm -f "${i%.wma}.wav"; done

"Mplayer" doit être installé pour lancer cette ligne de commande.

### Convertir tous les M4a d'un répertoire en Mp3

    for i in *.m4a; do ffmpeg -i "$i" -acodec libmp3lame -ac 2 -ab 128k "${i%m4a}mp3"; done

"Ffmpeg" doit être installé pour lancer cette ligne de commande. Vous pouvez également modifier la qualité (et du coup le poids) du mp3 en remplaçant 128k par 192k ou 320k dans cette ligne de commande.

### Convertir un fichier Ogg en Mp3

    ffmpeg -i fichier.ogg -acodec libmp3lame fichier.mp3

"Ffmpeg" doit être installé pour lancer cette ligne de commande (qui fonctionne aussi pour les .oga).

### Convertir un fichier Amr en Mp3 (ici en 128k) avec Ffmpeg

    ffmpeg- i "piste audio.amr" -ab 128k "piste audio.mp3"

### Extraire l'audio de tous les fichiers Webm d'un dossier vers des Mp3

    find . -type f -iname "*.webm" -exec bash -c 'FILE="$1"; ffmpeg -i "${FILE}" -vn -ab 128k -ar 44100 -y "${FILE%.webm}.mp3";' _ '{}' \;

Cette commande va rechercher tous les fichiers .webm du dossier depuis lequel vous la lancez (et ses sous-dossiers) puis les extraire en .mp3, chacun gardera le même nom à l'exception de l'extension.

### Enregistrer un flux radio mp3 (stream)

Modifier par l'URL désirée !

    wget -c http://URL.mp3

Exemple (pour FIP) : wget -c http://mp3.live.tv-radio.com/fip/all/fiphautdebit.mp3

### Mettre plusieurs fichiers Mp3 bout à bout (concaténer) et enregistrer le tout dans un fichier Wav

    sox -S --interactive *.mp3 "nom fichier voulu.wav"

Il faut que "Sox" soit installé sur votre système et les librairies de sox (flac, wav, mp3... ) que vous désirez utiliser.

Cette commande va concaténer tous les fichiers mp3 contenus dans le dossier en cours (par ordre alphabétique) à la condition qu'ils aient tous le même nombre de canaux (tous mono ou tous stéréo).

>Option : `-S` pour ne pas effacer par erreur un fichier déjà existant => Tapez "y" pour effacer, ou "n" pour ne pas effacer, puis la touche [Entrée] pour valider.  
    Option : `--interactive` pour surveiller le processus en cours (voir le traitement des fichiers dans la console).

NB. Sox n'enregistre pas en mp3. Si vous désirez vraiment du mp3, il faudra passer par un autre format de sortie (wav, flac...) puis utiliser "Lame" pour encoder celui-ci en mp3.

### Sept instructions pour en savoir plus sur une commande ou une application

En modifiant X par le nom de la commande recherchée.

```
man X
whatis X
X -h
X --help
X -?
info X
about X
```

Exemple, afficher dans le terminal le fichier d'aide de l'application "lame" :

    lame --help

Pour sortir des pages d'un manuel depuis un terminal, utilisez la touche [q].

### Faire le tri dans une réponse du Terminal

On prend très vite goût au terminal, néanmoins lorsque l'on recherche une information particulière, celui-ci nous renvoie parfois des dizaines et des dizaines de lignes. L'astuce utiliser la commande "grep" derrière un pipe, le symbole | accessible avec la combinaison des touches du clavier[Alt Gr + 6], suivi de l'occurrence recherchée.

Exemple, recherche de toutes les lignes comportant "mp3" dans la documentation de "lame".

    man lame | grep mp3

Vous pouvez également préciser une autre occurence (un équivalent de "ou", le fameux OR de nombreux langages de programmation), à l'aide d'un autre pipe que l'on échappe par un anti slash.
En effet dans de nombreux langages de programmation, dont le Bash, certains meta-caractères ont des fonctions bien précises, tels que ? + { | () - par exemple. Pour pouvoir les utiliser en dehors de leurs fonctions, il faut les échapper avec un anti slash que l'on place avant ceux-ci.

Exemple, recherche de toutes les lignes comportant "volume scaling" ou "WAV" dans la documentation de "lame" :

    man lame | grep 'volume scaling\|WAV'

Grep est sensible à la casse, si vous souhaitez annihiler ce comportement, utilisez l'option [-i].

Pour maintenant inverser la recherche et retirer les passages comportant une certaine occurence des réponse du terminal, l'option -v est toute désignée.

Exemple, recherche de toutes les lignes sauf celle se rapportant au "mono" dans la documentation de "lame" :

    man lame | grep -v mono

La commande "grep" est très utile et je vous invite à consulter son manuel pour utiliser toutes ses options et pourquoi pas à l'aide de grep lui même !

Exemple, recherche de l'option "-f" dans la documentation de grep :

    man grep | grep '\-f'

Un petit inconvénient néanmoins avec grep, c'est qu'elle ne retourne par défaut qu'une seule ligne. Pour pouvoir en obtenir un peu plus, l'option [-An] (A pour after et n pour nombre) va nous y aider.

Exemple, recherche dans la doc de "grep" de lignes où se trouve l'occurrence "NUM" suivies de trois autres lignes :

    man grep | grep -A3 NUM

### Sauvegarder le manuel d'une commande depuis le Terminal vers un fichier texte

Exemple pour l'utilisateur "Adam" voulant enregistrer le manuel de gs (ghostscript) sur son bureau :

    man gs | col -b > "/home/Adam/Bureau/Manuel_Ghostscript.txt"

'col' : Élimine les sauts de ligne arrière, l'option -b sert à ne pas émettre de retour chariot.

Cette commande s'avère en outre très pratique lorsque que l'on recherche dans un manuel une occurrence incluant des caractères spéciaux ($, &...), car ensuite en ouvrant son document texte dans Gedit (ou un autre éditeur), un [Ctrl+F] permet de trouver la réponse (sans avoir besoin d'échapper les caractères spéciaux).

### Trouver le "raccourci" qui lance un programme

    which nom_du_programme

which retourne le chemin complet d'un programme (le 1er avec le nom indiqué) trouvé dans le "path".

### Trouver un fichier

Recherche "monfichier" dans les répertoires de /home

    find /home -name monfichier

### Trouver un fichier.xyz

Recherche "Mon Fichier.xyz" dans les répertoires de /home

    find /home -name "Mon Fichier.xyz"

### Trouver tous les fichiers ayant une extension .x dans le dossier courant

    find . -name '*.x'

### Trouver tous les fichiers "html" dans les répertoires de /home

    find /home -name '*.html'

### Trouver tous les fichiers ".xcf" stockés dans un disque dur externe

Imaginons que le disque dur externe soit nommé "USB-Drive", le code sera :

    find /media/USB-Drive/ -name '*.xcf'

Notez bien que dans cette commande, il faut ajouter un slash [/] après le nom du disque dur externe.

### Effacer les traces des fichiers Flash après navigation internet

    rm -R ~/.adobe && rm -R ~/.macromedia

### Effacer les fichiers de la corbeille (si certains sont récalcitrants)

    sudo rm -R .local/share/Trash/*

Votre mot de passe ROOT sera demandé dans la console, tapez-le puis validez par la touche [Entrée].


### Effacer les miniatures (qui parfois prennent de la place)

    rm -r -f ~/.thumbnails/normal/*

### Vérifier si un fichier PDF est bien en CMJN

    identify -verbose mon-fichier.pdf | grep Colorspace

Si le terminal vous répond Colorspace: CMYK votre PDF est bien en CMJN.
Pour info "identify" fait partie du programme ImageMagick, un petit bijou pour le traitement d'images, normalement installé par défaut sous Ubuntu.

### Fusionner plusieurs documents PDF en un seul

    pdftk 1.pdf 2.pdf 3.pdf 4.pdf cat output unique.pdf

"pdftk" doit être installé pour lancer cette ligne de commande (pdftk est aussi un outil très pratique pour effectuer toutes sortes d'opérations sur des fichiers PDF).

### Convertir une vidéo Ogv en une vidéo Flv (Flash)

Dans cette ligne de commande, modifiez "input".ogv et "output".flv par le nom de votre vidéo.

    ffmpeg -i "input.ogv" -acodec libmp3lame -aq 4 -vcodec libx264 -vpre hq -crf 26 -wpredp 0 -threads 0 "output.flv"

### Convertir une vidéo Webm en une vidéo Mp4

Dans cette ligne de commande, modifiez "mavideo" par le nom de votre vidéo.

    ffmpeg -i mavideo.webm mavideo.mp4

### Convertir une vidéo Mp4 en une vidéo Avi

Dans cette ligne de commande, modifiez "mavideo" par le nom de votre vidéo.
Cette commande est bien utile si une vidéo ne passe pas sur TV (pourvue d'une entrée USB).

    ffmpeg -i mavideo.mp4 -codec copy mavideo.avi

### Convertir une vidéo Flv en une vidéo Avi

Modifiez "input.flv" et "output.avi" par le nom de votre vidéo.

    ffmpeg -i "input.flv" -f avi "output.avi"

### Retourner une vidéo Mov à 180°

Il arrive qu'une vidéo faite depuis un smartphone soit la tête en bas, pour y remédier voici la ligne de commande.

Modifiez "input.mov" et "output.mov" par le nom de votre vidéo.

    mencoder "input.mov" -oac pcm -ovc lavc -vf flip -o "output.mov"

L'application "Mencoder" doit être installée pour lancer cette ligne de commande.

### Droits (ou "Permissions") sur fichiers et dossiers

L'une des fondamentales du système GNU/Linux et donc d'Ubuntu est de fonctionner avec des droits sur les fichiers. Assurant de fait beaucoup plus de sécurité, puisque sur les répertoires "sensibles" par exemple, en tant que simple utilisateur, vous n'avez pas le droit de modifier ceux-ci (et donc un mauvais plaisantin non plus).

Cependant, il peut arriver de devoir modifier ce comportement (configuration d'un répertoire web local pour un serveur Apache et PHP, disque dur externe en NTFS, etc.).

La première chose à faire dans un tel cas est de voir comment sont attribués les droits sur le répertoire en question, avec la commande :

    ls -l nom_du_fichier

Sachant que sous GNU/Linux les dossiers sont considérés comme des fichiers !

Pour changer ces permissions, le plus simple est ensuite depuis Nautilus (le gestionnaire de fichiers),
de faire un clic droit sur le répertoire > Propriétés > Permissions


### Établir une liste des fichiers et des dossiers contenus dans un répertoire et la sauvegarder

Dans cet exemple, la liste sera enregistrée dans un fichier nommé "listing.txt"

    ls -a -R > listing.txt

>Un seul symbole "supérieur à" [>] permet de sauvegarder dans un fichier en effaçant les anciennes données.  
    Deux symboles "supérieur à" [>>] sauvegarde sur un fichier en conservant les anciennes données (enregistrant les nouvelles à la suite).  
    l'option -a : sert à lister les fichiers cachés  
    l'option -R : sert pour afficher de façon récursive les sous-répertoires.

Pour en savoir plus sur les options de la commande ls reportez-vous à son manuel :

    man ls

### Établir uniquement une liste des fichiers d'un dossier et la sauvegarder

Dans cet exemple, la liste sera enregistrée dans un fichier nommé "listing-fichiers.txt" et inventoriera tous les fichiers y compris ceux des sous-dossiers.

    find . -type f > listing-fichiers.txt

### Établir une liste des fichiers avec leur chemin d'accès

Dans cet exemple, la liste sera enregistrée dans un fichier nommé "liste-complete.txt"

    find "$PWD" -type f > liste-complete.txt

### Établir seulement une liste des images avec leur chemin d'accès

Dans cet exemple, la liste sera enregistrée dans un fichier nommé "liste-images.txt" et répertoriera seulement les fichiers .jpg, .gif, .png et .jpeg du dossier (et de ses sous-dossiers) depuis laquelle est lancée cette commande.

    find "$PWD" -type f -iregex ".*\.\(jpg\|gif\|png\|jpeg\)" > liste-images.txt

Mettre $PWD entre guillements permet d'éviter d'éventuels erreurs si les noms de vos dossiers ou fichiers comportent des espaces.

### Compter le nombre de fichiers compris dans un dossier

Le résultat englobera tous les fichiers y compris ceux des sous-dossiers (sans ajouter le nombre de dossiers eux-mêmes).

    find . -type f | wc -l

### Trier les lignes d'un fichier texte par ordre alphabétique

la commande "sort" trie toutes les lignes des fichiers indiqués ; avec l'option -o cette commande permet d'écrire dans un fichier de sortie et, si le fichier de sortie est également le fichier source, les lignes initiales du fichier source seront écrasées et remplacées par les lignes triées.
- Exemple pour trier alphabétiquement (a z) le fichier "listing.txt" et enregistrer le résultat sur lui-même :

    sort listing.txt -o listing.txt

- Pour trier à l'envers (z a), utilisez l'option -r :

    sort -r listing.txt -o listing.txt

### Ajouter un prefix "TRUC - " à tous les noms de fichiers

Dans cette commande, le prefixe "TRUC" sera ajouté devant chaque nom de fichier du dossier courant.

    rename 's/(.*)/TRUC - $1/' *

### Ajouter un prefix à tous les noms des MP3

Dans cette commande, le prefixe "DanceFloor - " sera ajouté devant chaque nom de fichier du dossier courant.

    rename 's/(.*)/DanceFloor - $1/' *.mp3

### Supprimer les 3 premiers caractères

Dans cette commande, les trois premiers caractères seront supprimés de tous les noms de fichiers du dossier courant.

    rename 's/.{3}(.*)/$1/' *

Modifiez {3} par {4} supprimera les quatre premiers caractères et ainsi de suite...

### Supprimer tous les termes "prefix" des noms de fichiers

Dans cette commande tous les noms de fichiers du dossier courant commençant par "prefix" seront écourtés de ce préfixe.

    for file in prefix*; do mv "$file" "${file#prefix}"; done;

Ici par exemple un fichier nommé "prefixmadame.txt" deviendra "madame.txt", tandis que celui nommé "monsieurprefix.txt" restera inchangé.

### Remplacer les underscores (_) par des espaces dans les noms des fichiers du dossier courant

"Billy_The_Kid.mp3" deviendrait donc "Billy The Kid.mp3".

    for file in *; do mv -- "$file" "${file//_/ }"; done;

### Remplacer une occurrence par une autre

Ici par exemple on va remplacer tous les "Amy" par des "Amy - Winehouse" dans les noms des fichiers du dossier courant.

    for file in *; do mv -- "$file" "${file//Amy/Amy Winehouse}"; done;

Par exemple "Amy - Rehab.mp3" deviendrait avec cette commande "Amy Winehouse - Rehab.mp3"

### Afficher l'espace disque utilisé et disponible pour chaque partition montée de façon compréhensive

Cette commande est utile pour surveiller par exemple la place qu'il reste en pourcentage pour la partition /boot

    df -h

Ainsi en lançant cette commande avant l'installation d'un nouveau kernel, vous pourrez contrôler la place qu'il reste sur la partition /boot et décider s'il vous faut nettoyer Ubuntu et plus précisément supprimer d'anciens noyaux, tâche que je préconise plutôt de faire avec Synaptic en utilisant les filtres de recherches.

### Déterminer la version du kernel actuellement installée

Pour connaître la version du kernel (noyau linux) actuellement installé sur votre machine, lancez dans le terminal, cette commande :

    uname -r

### Notabene

Dans la console, utilisez la flèche haute (ou basse) de votre clavier pour éviter de taper à nouveau une commande précédemment utilisée. En effet la console garde en mémoire, même après redémarrage de l'ordinateur, tout ce que vous y avez tapé, c'est idéal pour réutiliser fréquemment des instructions.

Parfois pour reprendre la main dans le terminal (avec des commandes affichant des résultats) plutôt que la touche [q], c'est la combinaison des touches [Ctrl+C] qu'il faut employer.

Certaines commandes nécessitent éventuellement votre mot de passe, lorsque vous le tapez dans le terminal il ne se passe rien, c'est normal (question de discrétion), tapez-le jusqu'à la fin puis pressez la touche [Entrée], La commande en question sera alors lancée !

Lorsque vous utilisez successivement plusieurs commandes, afin de retrouver une meilleure lisibilité, pour vider le terminal de toutes ses précédentes réponses, entrez simplement :

    clear

Si en revanche vous désirez effacer complètement toutes les commandes que vous avez lancées précédemment de la mémoire du Terminal, utilisez la commande :

    history -cw

Ou, pour supprimer toutes les commandes de l'historique de toutes les sessions exécutées au fil du temps, vous pouvez supprimer le contenu du fichier ".bash_history" avec la commande :

    cat /dev/null > ~/.bash_history

