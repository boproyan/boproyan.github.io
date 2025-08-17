+++
title = 'PACMAN gestionnaire de paquets archlinux'
date = 2022-12-03 00:00:00 +0100
categories = ['archlinux']
+++
- [Présentation de pacman](#présentation-de-pacman)
- [Configuration](#configuration)
    - [Options globales](#options-globales)
    - [Dépôts](#dépôts)
- [Utilisation](#utilisation)
    - [Synchronisation de la base de paquets](#synchronisation-de-la-base-de-paquets)
    - [Installation de paquets](#installation-de-paquets)
    - [Mise à jour des paquets](#mise-à-jour-des-paquets)
    - [Suppression de paquets](#suppression-de-paquets)
    - [Recherche](#recherche)
    - [Rétrograder des paquets](#rétrograder-des-paquets)
    - [Nettoyer le cache des paquets](#nettoyer-le-cache-des-paquets)
    - [Modification de la base de données](#modification-de-la-base-de-données)
    - [En vrac](#en-vrac)
    - [Une astuce totalement inutile et (peut-être) indispensable...](#une-astuce-totalement-inutile-et-peut-être-indispensable)
    - [Empêcher l'installation de locales](#empêcher-linstallation-de-locales)
    - [Récupérer les sources d'un paquet](#récupérer-les-sources-dun-paquet)
    - [Liste des paquets installés](#liste-des-paquets-installés)
    - [Installer des paquets depuis une liste](#installer-des-paquets-depuis-une-liste)
- [Problèmes](#problèmes)
    - [Crash de pacman durant une mise à jour](#crash-de-pacman-durant-une-mise-à-jour)
    - [Un paquet a été mis à jour mais pacman dit que le système est à jour](#un-paquet-a-été-mis-à-jour-mais-pacman-dit-que-le-système-est-à-jour)
    - ["Le paquet est introuvable"](#le-paquet-est-introuvable)
    - [Un même paquet se met à jour continuellement](#un-même-paquet-se-met-à-jour-continuellement)
    - [Équivalences entre Pacman et les autres gestionnaires de paquets](#équivalences-entre-pacman-et-les-autres-gestionnaires-de-paquets)
- [Pacman hook](#pacman-hook)
    - [Prérequis pour la création du hook et des scripts](#prérequis-pour-la-création-du-hook-et-des-scripts)
    - [Hook pour les sauvegardes Timeshift - 2 méthodes d'annulation](#hook-pour-les-sauvegardes-timeshift---2-méthodes-dannulation)
        - [Timeshift Pacman Hook](#timeshift-pacman-hook)
        - [Script de lancement de Timeshift](#script-de-lancement-de-timeshift)
        - [Script Killshot](#script-killshot)
    - [Hook pour créer une liste de tous les paquets installés](#hook-pour-créer-une-liste-de-tous-les-paquets-installés)
    - [Hook pour notifier les paquets orphelins](#hook-pour-notifier-les-paquets-orphelins)
        - [Hook pour les paquets orphelins - v.1](#hook-pour-les-paquets-orphelins---v1)
        - [Hook pour notifier les paquets orphelins - v.2](#hook-pour-notifier-les-paquets-orphelins---v2)
    - [Hook pour vérifier la présence de fichiers pacnew/pacsave](#hook-pour-vérifier-la-présence-de-fichiers-pacnewpacsave)
    - [Hook pour désactiver l'indexeur de fichiers baloo](#hook-pour-désactiver-lindexeur-de-fichiers-baloo)
    - [Hook pour mettre à jour grub/lsb-release](#hook-pour-mettre-à-jour-grublsb-release)
    - [Hook paccache qui supprime automatiquement le cache](#hook-paccache-qui-supprime-automatiquement-le-cache)


## Présentation de pacman 

[Archlinux - pacman Tips and tricks (Français)](https://wiki.archlinux.org/title/Pacman_(Fran%C3%A7ais)/Tips_and_tricks_(Fran%C3%A7ais))

**[pacman](http://archlinux.org/pacman)** est le gestionnaire de paquets
d'Archlinux. Il combine un ensemble d'outils binaires avec un système
relativement simple pour construire des paquets. (cf
[makepkg](https://wiki.archlinux.fr/Makepkg) et [ABS](https://wiki.archlinux.fr/Abs)). Le but étant de
facilement gérer les paquets, qu'ils proviennent des dépôts officiels
ou qu'ils soient compilés par l'utilisateur.

**pacman** permet de garder votre système à jour en synchronisant la
liste des paquets depuis un serveur puis de télécharger/installer les
nouveaux paquets, ainsi que leur dépendances, avec une simple commande.

## Configuration

**pacman** se configure à l'aide du fichier **/etc/pacman.conf** (cf.
page man:
[pacman.conf(5)](http://www.archlinux.org/pacman/pacman.conf.5.html)).

### Options globales 

Ces options se configurent sous la section **[options]**, quelques
exemples :

-   **Architecture** : si elle est définie, **pacman** n'installera que
    les paquets de cette architecture. Elle peut prendre la valeur
    *auto* (l'architecture sera définie par un appel à ). Elle permet
    aussi l'utilisation de la variable dans
    l'[URI](http://en.wikipedia.org/wiki/fr:URI) d'un
    [dépôt](https://wiki.archlinux.fr/Pacman#D.C3.A9p.C3.B4ts).
-   **IgnorePkg** : indique à **pacman** les paquets à ne pas mettre à
    jour.
-   **IgnoreGroup** : indique à **pacman** les groupes à ne pas mettre à
    jour.
-   **UseDelta** : cette option ne prend pas de paramètre et permet
    d'indiquer à **pacman** de télécharger les deltas de paquets s'ils
    sont disponibles. (cf. [Utilisation des deltas](https://wiki.archlinux.fr/Deltas)
-   **SigLevel** : Indique à comment gérer les signatures :
    [pacman-key](https://wiki.archlinux.fr/Pacman-key)
-   **Color** : la sortie de pacman en console sera colorisée.
-   **NoExtract** : Indique à de ne pas extraire certains fichiers (ex.
    [Empêcher l'installation de
    locales](#Empêcher_l'installation_de_locales "wikilink"))

### Dépôts

La syntaxe est simple :

`[nom_du_depôt]`  
`Server=miroir1_du_depôt`  
`Server=miroir2_du_depôt`  

cf. [Détails des dépôts](https://wiki.archlinux.fr/Depots).

## Utilisation

Pour une description complète, merci de vous référer à la page man :
[pacman(8)](http://www.archlinux.org/pacman/pacman.8.html). Certaines
options sont à prendre avec beaucoup de précaution, consultez [ces
quelques
conseils](https://wiki.archlinux.fr/Enhancing_Arch_Linux_Stability#.C3.89vitez_certaines_commandes_avec_pacman)
pour utiliser pacman au mieux.

Voici quelques exemples d'opérations :

### Synchronisation de la base de paquets 

Cette opération met à jour la liste des paquets disponibles sur les
[miroirs](https://wiki.archlinux.fr/Miroirs) :

`pacman -Sy`

### Installation de paquets  

Installation d'une liste de paquets :

`pacman -S paquet_1 paquet_2`

Si le paquet existe sous plusieurs dépôts, on peut éventuellement en
préciser un :

`pacman -S extra/paquet`

Si vous avez l'archive d'un paquet sur votre disque :

`pacman -U archive_du_paquet.pkg.tar.xz`

Ou même si vous avez le lien de l'archive, par exemple :

`pacman -U `[`https://www.archlinux.org/packages/core/i686/pacman/download/`](https://www.archlinux.org/packages/core/i686/pacman/download/)

Pour l'installation d'une liste de paquets depuis un fichier texte (un
paquet par ligne) :

`pacman -S - < pkglist.txt`

Il est possible aussi de faire ainsi si le nombre de paquets n'est pas
trop grand :

` pacman -S $(cat pkglist.txt)`

### Mise à jour des paquets 

Mise à jour suite à une synchronisation faite précédemment :

`pacman -Su`

Synchronisation puis, mise à jour :

`pacman -Syu`

Pour installer un nouveau paquet tout en mettant à jour le système :

`pacman -Syu paquet_1`

### Suppression de paquets 

`pacman -R paquet_1`

Pour garder un système propre, il faut aussi supprimer les dépendances
qui ne sont plus requises par aucun paquet :

`pacman -Rs paquet_1`

Par défaut, les fichiers de configuration modifiés sont sauvegardés avec
l'extension **.pacsave**. Pour ne pas les conserver :

`pacman -Rsn paquet_1`

### Recherche

Pour avoir une aide rapide :

`pacman -Q --help`  
`pacman -S --help`  
`pacman -R --help`  

Recherche d'un paquet parmi ceux installés :

`pacman -Qs paquet`

Recherche d'un paquet dans les dépôts :

`pacman -Ss paquet`

**pacman** peut effectuer une recherche avec des expressions régulières
:

Informations complètes sur un paquet installé (par exemple pour vérifier
si certaines des dépendances optionnelles peuvent vous apporter des
fonctionnalités supplémentaires):

`pacman -Qi paquet`

ou non:

`pacman -Si paquet`

Liste des fichiers d'un paquet installé :

`pacman -Ql paquet_1`

Savoir à quel paquet appartient l'un des fichiers du système:

`pacman -Qo /chemin/vers/le/fichier`

Liste des paquets n'appartenant à aucun dépôt configuré dans
pacman.conf :

`pacman -Qm`

Liste des paquets (dépendances) n'étant plus requis par le système (le
paquet qui a causé leur installation n'est plus présent) :

`pacman -Qdt`

### Rétrograder des paquets  

Il est parfois nécessaire de revenir temporairement à la version
précédente d'un paquet (régression, bug), différents moyens existent:
[Downgrade](https://wiki.archlinux.fr/Downgrade).

### Nettoyer le cache des paquets  

*pacman* stocke les paquets téléchargés ici et n'enlève pas
automatiquement les versions anciennes ou désinstallées. Il faut donc
vider régulièrement ce dossier pour éviter qu'il ne prenne une taille
démesurée.

La commande suivante de pacman supprime tous les paquest non-installés
du cache (anciennes versions ou non installées sur le système):

`pacman -Sc`

À cause de toutes les restrictions énoncées avant, il est recommandé
d'utiliser un script dédié au nettoyage du cache de pacman, afin de
contrôler plus finement les paquets à supprimer:

-   La commande *paccache*, fourni avec le paquet , supprime par défaut
    tous les paquets du cache sauf les trois dernières versions les plus
    récentes:

`paccache -r`

Cependant, *paccache* *ne* vérifiera *pas* si les paquets sont encore
installés sur le système et laissera donc les paquets non-installés dans
le cache. Pour supprimer du cache toutes les versions des paquets
non-installés, vous devez lancer la commande suivante dans un second
temps :

`paccache -ruk0 `

Voir pour plus d'options.

-   Vous pouvez également installer et utiliser les scripts alternatifs : [pkgcacheclean](https://aur.archlinux.org/packages/pkgcacheclean)(écrit en C)... :

`pkgcacheclean`

-   ...ou [pacleaner](https://aur.archlinux.org/packages/pacleaner)(écrit en python) depuis [AUR](https://wiki.archlinux.fr/AUR):

Pour vérifier les paquets à supprimer sauf les trois plus récents :

`pacleaner -m`

Pour vérifier les paquets non-installés :

`pacleaner -u`

Utilisez l'option --delete pour supprimer les paquets :

`pacleaner -m --delete`
`pacleaner -u --delete `

Les options par défaut peuvent être modifiées dans un fichier de
configuration (nombre de paquets à conserver, chemin d'accès vers le
cache...)

### Modification de la base de données 

Vous pouvez modifier certains éléments de la base de données des paquets
installés, notamment la raison d'installation d'un paquet (installés
en tant que dépendances ou explicitement) :

`pacman -D --asexplicit paquet_1 paquet_2`
`pacman -D --asdeps paquet_1 paquet_2`

### En vrac 

-   Téléchargement des paquets seulement:

`pacman -Sw paquet_1 paquet_2`

-   Vérifier qu'il ne manque pas de fichiers installés sur l'ensemble
    des paquets:

`pacman -Qqk`

### Une astuce totalement inutile et (peut-être) indispensable... 

Il existe une petite astuce avec Pacman que --- j'en suis sûr --- vous
allez vous empresser de tester. Éditez le fichier
"**/etc/pacman.conf**" et ajoutez l'option "**ILoveCandy**" dans la
rubrique en dessous de "**[options]**" :

`[options]`
`ILoveCandy`

Vous obtiendrez par la suite une sortie de Pacman similaire au jeu du
même nom lorsque vous mettrez votre système à jour.

### Empêcher l'installation de locales 

L'option du fichier est très utile si certaines parties de paquets vous
sont inutiles. Les locales sont un cas typique de ce genre de fichiers
dont on pourrait vouloir se passer.

Aucun des fichiers listés dans les règles de l'option n'est extrait
lors de l'installation d'un paquet. Le motif ***** est autorisé pour
désigner un ensemble de fichier (comme on le ferait dans un shell). Par
exemple désigne tous les fichiers dans le dossier ainsi que dans ses
sous-dossiers. Il est possible d'inverser le comportement d'une règle
en la préfixant d'un **!**. Cela annule les règles précedentes sur les
fichiers correspondant.

Donc, si on veut se passer de toutes les locales sauf celles
francofrançaises, on obient les règles suivantes à ajouter dans la
section de .

Les règles peuvent aussi être indiquées sur une seul ligne

### Récupérer les sources d'un paquet 

`git clone --branch packages/nom_du_paquet --single-branch `[`https://git.archlinux.org/svntogit/packages.git`](https://git.archlinux.org/svntogit/packages.git)` nom_du_paquet`

### Liste des paquets installés

Il peut être utile de conserver la liste de tous les paquets installés explicitement, par exemple pour sauvegarder un système ou une installation plus rapide sur un nouveau système:

```shell
$ pacman -Qqe > pkglist.txt
```


*    Avec l'option **-t**, les paquets déjà requis par d'autres paquets explicitement installés ne seront pas mentionnés. En cas de réinstallation depuis cette liste il seront bien installés, mais seulement en tant que dépendances.
*    Avec l'option **-n**, les paquets étrangers (ex. de AUR) seraient exclus de la liste.
*    Utiliser `comm -13 <(pacman -Qqdt | sort) <(pacman -Qqdtt | sort) > optdeplist.txt` pour créer aussi une liste des dépendances optionnelles installées qui pourront être réinstallées avec l'option --asdeps.
*    Utiliser `pacman -Qqem > foreignpkglist.txt` pour créer la liste des paquets AUR ou autres paquets étrangers, explicitement installés.

Pour garder à jour une liste de paquets installés explicitement (p. ex. en combinaison avec un fichier versionné dans` /etc/`), vous pouvez mettre en place un **hook** de pacman. Exemple:

```
[Trigger]
Operation = Install
Operation = Remove
Type = Package
Target = *

[Action]
When = PostTransaction
Exec = /bin/sh -c '/usr/bin/pacman -Qqe > /etc/pkglist.txt'
```

### Installer des paquets depuis une liste

Pour installer des paquets depuis une sauvegarde antérieure de la liste des paquets, tout en ne réinstallant pas ceux qui sont déjà installés et à jour, lancer:

    pacman -S --needed - < pkglist.txt

Cependant, il est probable que des paquets étrangers, tels ceux de AUR ou installés localement, soient présents dans la liste. Pour filtrer à partir de la liste les paquets étrangers, la ligne de commande précédente peut être enrichie comme suit :

    pacman -S --needed $(comm -12 <(pacman -Slq | sort) <(sort pkglist.txt))

Finalement, pour s'assurer que les paquets installés de votre système correspondent à la liste et supprimer tous les paquets qui n'y sont pas mentionnés :

    pacman -Rsu $(comm -23 <(pacman -Qq | sort) <(sort pkglist.txt))

## Problèmes

### Crash de pacman durant une mise à jour 

Dans le cas où pacman crashe durant une mise à jour, une suppression de
paquet ou encore une réinstallation d'un paquet, il faudra faire ces
différentes étapes :

-   Démarrer sur un iso d'installation ArchLinux
-   Monter votre système de fichier **racine** (, , , etc., s'ils
    existent)
-   Mettre à jour votre base de données pacman ainsi que mettre à jour
    vos paquets via la commande
-   Réinstaller le(s) paquet(s) cassé(s) via la commande

### Un paquet a été mis à jour mais pacman dit que le système est à jour 

Les miroirs de pacman ne sont pas directement synchronisé. Il faut
compter environ 24 heures avant de pouvoir recevoir la mise à jour tant
désirée.

### "Le paquet est introuvable" 

Premièrement, assurez vous que le paquet que vous tentez d'installer
existe réellement. Si ce paquet existe vraiment, vérifiez que
l'orthographe est correcte. Il se peut aussi que le paquet existe et
que l'orthographe est correcte mais que la liste de vos paquets est
obsolète. Dans ce cas, un simple **pacman -Syyu** fera amplement
l'affaire.

### Un même paquet se met à jour continuellement 

Ce problème est du a des entrées dupliquées dans
**/var/lib/pacman/local/** comme s'il y avait deux instances de linux.
La commande **pacman -Qi** donne la bonne version et la commande
**pacman -Qu** reconnait la plus vieille version **ET** essaye de la
mettre à jour. La solution est donc d'effacer la mauvaise entrée dans
**/var/lib/pacman/local/**

### Équivalences entre Pacman et les autres gestionnaires de paquets 

Si vous avez l'habitude d'utiliser Arch, Red Hat/Fedora,
Debian/Ubuntu, SUSE/openSUSE ou encore Gentoo, et que vous rencontrez
des difficultés à prendre en main le gestionnaire de paquets mais que
vous êtes à l'aise avec un autre, vous pouvez consulter la page [Pacman
Rosetta (en)](https://wiki.archlinux.org/index.php/Pacman_Rosetta), qui donne les commandes
équivalentes entre chacun des gestionnaires de paquets.

Si vous êtes très à l'aise avec Pacman mais que vous n'arrivez pas à
prendre d'autres gestionnaires de paquets en main,
[pacapt](https://github.com/icy/pacapt) est une solution.


## Pacman hook



Les "hooks" Pacman créés par les utilisateurs

* Les hooks de Pacman vous permettent d'automatiser toute fonction que vous souhaitez voir exécutée avant ou après une installation/désinstallation ou une mise à jour de Pacman.
* Les hooks inclus dans les paquets installés peuvent être trouvés dans /usr/share/libalpm/hooks.
* Les hooks créés par l'utilisateur doivent être placés dans /etc/pacman.d/hooks/
* Tous les scripts associés peuvent être placés dans /etc/pacman.d/hooks.bin/
    * Vous devrez créer ces répertoires, car ils ne sont pas créés automatiquement.
* Les noms de fichiers Hooks doivent avoir le suffixe .hook.
* Les hooks sont exécutés dans l'ordre alphanumérique de leur nom de fichier, - les hooks préfixés par un nombre inférieur sont prioritaires).

### Prérequis pour la création du hook et des scripts 

Créez des sous-répertoires pour le hook pacman et les scripts liés

    sudo mkdir -p /etc/pacman.d/{hooks,hooks.bin}

### Hook pour les sauvegardes Timeshift - 2 méthodes d'annulation 

#### Timeshift Pacman Hook

Ce hook déclenche une sauvegarde automatique de Timeshift avant d'installer une mise à jour du système. Il contient également plusieurs méthodes pour annuler la sauvegarde. Cette version inclut l'annulation automatique de la sauvegarde si une autre sauvegarde a été effectuée récemment dans un délai réglable. Elle intègre également une fonction qui permet l'annulation de toute sauvegarde avec une fenêtre de compte à rebours de 10 secondes (réglable) via le clavier. Pendant que le compte à rebours est actif, vous pouvez annuler n'importe quelle sauvegarde en appuyant sur la touche ENTER.

Il se peut que vous receviez un message d'erreur contextuel lorsqu'un instantané en temps partagé est annulé. Ce message d'erreur n'a pas d'importance car il ne fait que notifier l'échec de la sauvegarde programmée en temps partagé (ce qui était prévu).

Avec un éditeur de texte capable de gérer l'utilisateur root, créez le hook timeshift pacman 

    /etc/pacman.d/hooks/50-timeshift.hook

```shell
#/etc/pacman.d/hooks/50-timeshift.hook

[Trigger]
Operation = Upgrade
Type = Package
Target = *

[Action]
Description = pre-upgrade timeshift snapshot
When = PreTransaction
Exec = /bin/sh -c "sudo -Hiu user konsole -e /etc/pacman.d/hooks.bin/killshot.sh | /etc/pacman.d/hooks.bin/timeshift.sh"
```

Il est parfois difficile d'utiliser des variables dans les hooks pacman, il y a donc des portions de code dans lesquelles vous devrez insérer vos propres informations spécifiques.

    Exec = /bin/sh -c "sudo -Hiu user konsole -e

Dans la partie ci-dessus de la ligne "Exec", remplacez "user" par votre propre nom d'utilisateur. Remplacez " konsole " par le nom de l'émulateur de terminal que vous utilisez sur votre système. Tous les émulateurs de terminal ne se lancent pas correctement en utilisant le paramètre "-e" (execute). Le terminal gnome peut poser des problèmes d'exécution automatique avec ce script. Contrairement à de nombreux autres émulateurs de terminal, gnome-terminal n'accepte pas le paramètre "-e". Si quelqu'un connaît la meilleure méthode pour lancer automatiquement des scripts avec gnome-terminal, merci de poster l'info.

Ce script doit être utilisé avec un émulateur de terminal capable d'accepter le paramètre "-e" (execute) au lancement. Je ne sais pas exactement quels terminaux sont capables de cela, mais Yakuake et Konsole semblent bien gérer cette tâche. Vous devrez peut-être expérimenter en utilisant différents terminaux pour trouver la bonne méthode de lancement automatique du script via un hook pacman.

#### Script de lancement de Timeshift

Ce script lance une sauvegarde timeshift automatiquement à chaque fois qu'une mise à jour est lancée. Le script contient également une méthode pour annuler automatiquement toute sauvegarde si un autre instantané a été pris récemment. Cette fonction a pour but de limiter les instantanés répétés dans un court laps de temps. Les snapshots répétés deviennent une gêne importante lorsque vous rencontrez un problème de mise à niveau que vous devez déboguer.

Ce script annulera l'instantané si un instantané précédent a été pris dans la fenêtre de temps prédéterminée que vous sélectionnez. Pour modifier le délai, remplacez "-mmin -60" (1 heure) par une autre unité de temps. Si vous préférez que le déclencheur soit fixé à deux heures, il suffit de changer l'unité de temps en "-mmin -120".

Vous devrez remplacer le chemin "/run/media/user/1TB/timeshift/snapshots-ondemand" dans le script par l'emplacement réel de votre sauvegarde timeshift. La sauvegarde est stockée dans le sous-répertoire "/snapshots-ondemand" du répertoire principal de timeshift.

Le script déclenche également un petit message dans l'interface graphique lorsque la sauvegarde commence.

Créez le script pour lancer (ou annuler automatiquement) le snapshot timeshift 

    /etc/pacman.d/hooks.bin/timeshift.s

```shell
#!/bin/bash
#/etc/pacman.d/hooks.bin/timeshift.sh
find /run/media/user/1TB/timeshift/snapshots-ondemand -mmin -60 | grep $(date +%Y-%m-%d)
if [ $? -eq 0 ]; then
    echo "timeshift backup canceled, time threshold not met"
else
    sudo -Hiu user /bin/sh -c 'DISPLAY=:0 notify-send "Timeshift_Backup_Beginning"'
        echo " "
            echo "Initiating Pre-Update Timeshift Snapshot"
            echo " "
        echo "You have 10 seconds to cancel timeshift snapshot"
    /usr/bin/timeshift --create --comments "timeshift-pacman-hook-snapshot"
fi
exit
```

Sauvegardez le script, puis rendez le script exécutable

    sudo chmod +x /etc/pacman.d/hooks.bin/timeshift.sh

#### Script Killshot 

Ce script permet d'annuler manuellement l'instantané de timeshift par une intervention au clavier. Il fournit une fenêtre de compte à rebours de 10 secondes (réglable) pendant laquelle vous pouvez annuler l'instantané. Pendant que le compte à rebours est actif, vous pouvez annuler n'importe quelle sauvegarde en appuyant sur la touche ENTER.

Le nombre de secondes pendant lesquelles la fenêtre de compte à rebours est ouverte peut être ajusté dans cette ligne

    for (( i=9; i>0; i--)); do

Si vous souhaitez augmenter la fenêtre de temps pour l'annulation à 20 secondes, ajustez-la comme suit

    for (( i=20; i>0; i--)); do

Le script déclenche également un petit message contextuel de l'interface graphique lorsque la sauvegarde est interrompue.

Créez le script permettant d'annuler l'instantané en temps partagé par une intervention au clavier

    /etc/pacman.d/hooks.bin/killshot.sh

```shell
#!/bin/bash
#/etc/pacman.d/hooks.bin/killshot.sh
  echo "press any key to cancel timeshift snapshot"
      sleep 1
for (( i=9; i>0; i--)); do
    printf "\n$i seconds left to cancel snapshot  ... hit any key "
    read -s -n 1 -t 1 key
if [ $? -eq 0 ]; then
    sudo kill -2 `ps -ef|grep -i timeshift| grep -v grep| awk '{print $2}'`
        echo " "
            echo "aborting timeshift snapshot" 
      echo " "
   sudo -Hiu user /bin/sh -c 'DISPLAY=:0 notify-send "Timeshift_snapshot_aborted"'
fi
done
```

Sauvegardez le script, puis rendez-le exécutable

    sudo chmod +x /etc/pacman.d/hooks.bin/killshot.sh

C'est plus compliqué à écrire que les autres hooks timeshift, mais bien plus polyvalent


### Hook pour créer une liste de tous les paquets installés 

hook liste de sauvegarde du paquet Pacman :

Ce hook complète bien le hook timeshift ci-dessus dans toute stratégie de sauvegarde. Ce hook sauvegardera une liste de vos paquets natifs et étrangers (AUR) installés. Cela garantit que vous aurez toujours une liste à jour de tous vos paquets que vous pourrez réinstaller.

Pourquoi en avez-vous besoin, me direz-vous (si vous avez une sauvegarde timeshift) ? Si vous souffrez d'une panne matérielle catastrophique, vous apprécierez vraiment de pouvoir réinstaller automatiquement tous vos paquets préférés à partir d'une sauvegarde.

Créez le hook suivant 

    /etc/pacman.d/hooks/50-pacman-list.hook

```shell
#/etc/pacman.d/hooks/50-pacman-list.hook
[Trigger]
Type = Package
Operation = Install
Operation = Upgrade
Operation = Remove
Target = *

[Action]
Description = Create a backup list of all installed packages
When = PreTransaction
Exec = /bin/sh -c 'pacman -Qqen  > "/home/$USER/.cache/package_lists/$(date +%Y-%m-%d_%H:%M)_native.log"; pacman -Qqem > "/home/$USER/.cache/package_lists/$(date +%Y-%m-%d_%H:%M)_alien.txt" 2> /dev/null; exit'
```

Vous souhaiterez très probablement modifier le chemin où vous stockez vos listes de sauvegarde. L'ajout de plusieurs emplacements de sauvegarde est une meilleure assurance en cas de panne matérielle.

(edit) ci-dessous est un exemple de comment vous pourriez sauvegarder dans votre répertoire personnel, et aussi sur un disque de sauvegarde externe dans la même ligne "Exec"

```shell
Exec = /bin/sh -c 'pacman -Qqen  > "/home/$USER/$(date +%Y-%m-%d@%H:%M)_native.log"; pacman -Qqem > "/home/$USER/$(date +%Y-%m-%d@%H:%M)_alien.txt"; pacman -Qqem > "/run/media/$USER/Backup/package_lists/$(date +%Y-%m-%d@%H:%M)_native.log"; pacman -Qqem > "/run/media/$USER/Backup/package_lists/$(date +%Y-%m-%d@%H:%M)_alien.txt " 2> /dev/null; exit'
```

La meilleure pratique consiste souvent à remplacer "$USER" par votre propre nom d'utilisateur. Dans la plupart des cas, "$USER" fonctionnera correctement, mais dans certains cas, il peut insérer l'utilisateur root plutôt que votre propre nom d'utilisateur. Il est donc parfois préférable de s'assurer que votre nom d'utilisateur correct est inséré en remplacement de "$USER".

### Hook pour notifier les paquets orphelins 

#### Hook pour les paquets orphelins - v.1

Voici un autre hook très pratique pour le ménage.

Vous pouvez utiliser un hook pacman pour vous avertir lorsque des paquets orphelins sont créés.

Pour lister tous les paquets qui ne sont plus nécessaires comme dépendances, créez le hook suivant

    /etc/pacman.d/hooks/orphans.hook

``` 
#/etc/pacman.d/hooks/orphans.hook

[Trigger]
Operation = Install
Operation = Upgrade
Operation = Remove
Type = Package
Target = *

[Action]
Description = Orphaned package notification
When = PostTransaction
Exec = /usr/bin/bash -c "/usr/bin/pacman -Qtd || /usr/bin/echo '=> No orphans found.'"
```

Vous pourriez également faire en sorte que le hook supprime automatiquement les paquets orphelins, mais l'automatisation de cette fonction est un peu trop risquée.

#### Hook pour notifier les paquets orphelins - v.2 

``` 
[Trigger]
Operation = Install
Operation = Upgrade
Operation = Remove
Type = Package
Target = *
>
>
>
[Action]
Description = Checking for orphaned packages...
Depends = pacman
When = PostTransaction
Exec = /usr/bin/bash -c 'orphans=$(pacman -Qtdq); if [[ -n "$orphans" ]]; then echo -e "\e[1mOrphan packages found:\e[0m\n$orphans\n\e[1mPlease check and remove any no longer needed\e[0m"; fi'
```

### Hook pour vérifier la présence de fichiers pacnew/pacsave 

```
[Trigger]
Operation = Install
Operation = Upgrade
Operation = Remove
Type = Package
Target = *

[Action]
Description = Checking for .pacnew and .pacsave files...
When = PostTransaction
Exec = /usr/bin/bash -c 'pacfiles=$(pacdiff -o); if [[ -n "$pacfiles" ]]; then echo -e "\e[1m.pac* files found:\e[0m\n$pacfiles\n\e[1mPlease check and merge\e[0m"; fi'
```

### Hook pour désactiver l'indexeur de fichiers baloo 

La fonction de recherche de fichiers de Baloo a été considérablement améliorée dans KDE. Cependant, ce n'est toujours pas mon moteur de recherche préféré et je préférerais qu'il soit désactivé de façon permanente. Selon l'ArchWiki, baloo ne peut pas être désinstallé, et il peut être réactivé automatiquement lors d'une mise à jour du système. L'ArchWiki recommande de désactiver baloo après toute mise à jour du système si vous ne voulez pas qu'il fonctionne.

J'ai décidé de créer un hook pacman pour faire cela automatiquement après chaque mise à jour du système.

    /etc/pacman.d/hooks/disable-baloo.hook

```
[Trigger]
Type = Package
Operation = Upgrade
Target = /usr/bin/baloo_file

[Action]
Description = Disable baloo file indexer after every upgrade operation
When = PostTransaction
Exec = /bin/sh -c 'killall baloo_file ; mv /usr/bin/baloo_file /usr/bin/baloo_file.bak ; echo '#!/bin/sh' > /usr/bin/baloo_file'
```

### Hook pour mettre à jour grub/lsb-release 

J'ai quelques manjaro sur mon bureau, j'utilise un pacman hook de manjaro pour créer de bonnes entrées dans mon grub.
lancer le hook uniquement lorsque le fichier etc/lsb-release est mis à jour

    /etc/pacman.d/hooks/lsb-release.hook

```
[Trigger]
Operation = Install
Operation = Upgrade
Type = File
Target = etc/lsb-release

[Action]
Description = lsb-release change description for grub
When = PostTransaction
Exec = /usr/bin/sed -i 's/^DISTRIB_DESCRIPTION.*/DISTRIB_DESCRIPTION="Manjaro Cinnamon"/' /etc/lsb-release
```

### Hook paccache qui supprime automatiquement le cache

Supprimer cache des paquets installés, ne laisser que la dernière version

    /etc/pacman.d/hooks/paccache.hook

```
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *
[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -rk1
```

inclure les paquets désinstallés

```
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *
[Action]
Description = Cleaning pacman cache...
When = PostTransaction
Exec = /usr/bin/paccache -rk1 ; /usr/bin/paccache -ruk0
```