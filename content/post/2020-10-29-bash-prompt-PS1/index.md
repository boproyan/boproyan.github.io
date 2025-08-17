+++
title = 'bash prompt PS1'
date = 2020-10-29 00:00:00 +0100
categories = ['bash', 'cli']
+++
## bash prompt PS1=

[Bash tips: Colors and formatting (ANSI/VT100 Control sequences)-Lien HS](/files/html/BashColors.html)   
La variable de personnalisation du prompt sous Bash et ses dérivés est `PS1`  
Dans la plupart des distributions le prompt est initialisé dans **/etc/profile** ou dans les fichiers de configuration de bash (**/etc/bash/bashrc,/etc/bash.bashrc**)

Archlinux , Debian **/etc/bash.bashrc**

```
#
# /etc/bash.bashrc
#

# If not running interactively, don't do anything
[[ $- != *i* ]] && return

[[ $DISPLAY ]] && shopt -s checkwinsize

PS1='[\u@\H \W]\$ '

case ${TERM} in
  xterm*|rxvt*|Eterm|aterm|kterm|gnome*)
    PROMPT_COMMAND=${PROMPT_COMMAND:+$PROMPT_COMMAND; }'printf "\033]0;%s@%s:%s\007" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/\~}"'

    ;;
  screen*)
    PROMPT_COMMAND=${PROMPT_COMMAND:+$PROMPT_COMMAND; }'printf "\033_%s@%s:%s\033\\" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/\~}"'
    ;;
esac

[ -r /usr/share/bash-completion/bash_completion   ] && . /usr/share/bash-completion/bash_completion
```


La configuration utilisateur du propmt se fait dans **~/.bashrc** 

Les séquences d’échappement pouvant être utilisée pour le prompt sont principalement les suivantes :

* **\a** *alarme (le bip)* 
* **\\ ** *un back slash* 
* **\e[** *démarre une séquence de caractères non imprimable, interprété par le prompt (ex : définition de couleur, etc…)* 
* **\]** *fin de séquence non imprimable* 
* **\d** *la date au format “ ” (exemple : “Tue May 26?)* 
* **\H** *le nom de la machine* 
* **\h** *le nom de la machine avant le premier `.’* 
* **\$** *si l’UID est 0, affiche #, sinon, affiche $* 
* **\n** *retour chariot (saut/nouvelle ligne)* 
* **\nnn** *le caractère octal nnn* 
* **\s** *nom du shell, (équivaut à basename $0)* 
* **\#** *numéro de la commande saisie dans ce terminal* 
* **\@** *l’heure courante (format 12-heures am/pm)* 
* **\\!** *numéro de la commande dans l’historique* 
* **\T** *l’heure courante (format 12-heures HH:MM:SS)* 
* **\t** *l’heure courante (format 24-heures HH:MM:SS)* 
* **\u** *login utilisateur* 
* **\V** *la version de bash complète (ex : 2.00.0)* 
* **\v** *la version de bash (format court ex : 2.00)* 
* **\W** *répertoire courant (basename de \w)* 
* **\w** *répertoire courant (chemin complet).

L’application d’une couleurs est  définie par `\e[<code couleur>m`

Avec `<code couleur>`  une des valeurs suivante : 

    0;30 noir ; 1;30 gris ; 0;34 bleu ; 1;34 bleu clair ; 0;32 vert ; 1;32 vert clair ; 0;36 cyan ; 1;36 cyan clair ; 0;31 rouge ; 1;31 rouge clair ; 0;35 violet ; 1;35 rose ; 0;33 marron ; 1;33 jaune ; 0;37 gris clair ; 1;37 blanc ; 0 retour aux valeurs par défaut.

    echo -e '\e[0;30m noir \e[1;30m gris \e[0;34m bleu \e[1;34m bleu clair \e[0;32m vert \e[1;32m vert clair \e[0;36m cyan \e[1;36m cyan clair \e[0;31m rouge \e[1;31m rouge clair \e[0;35m violet \e[1;35m rose \e[0;33m marron \e[1;33m jaune \e[0;37m gris clair \e[1;37m blanc \e[0m retour aux valeurs par défaut'

![](bash-colors.png)

La couleur reste appliquée jusqu’à la demande de retour aux paramètres par défaut (`\e[0m`) ou la définition d’une autre couleur.

La couleur d’arrière plan peu être modifiée de la même manière que le texte avec des valeurs comprises entre 40 et 47 (représentant dans l’ordre noir, rouge, vert, jaune, bleu, rose, cyan et gris clair).

Par exemple :

    echo -e '\e[01;32m\e[44mHello world\e[0m'

affiche « Hello world » en vert sur fond bleu.![](bash-colors1.png)  
L’ensemble peut être raccourci en précisant en une seule instruction, la couleur de fond, celle du texte et éventuellement un (ou des) paramètre(s) d’action(s) :

    echo -e "\e[05;01;32;44mHello world\e[0m"

Affiche « Hello world » en vert (32), gras (01), clignotant (05) sur fond bleu (44).

### Prompt bash PS1

`PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\H: \w\a\]$PS1"`

La séquence **PS1** démarre par `\[` et se termine par `\]`

