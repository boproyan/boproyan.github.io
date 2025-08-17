+++
title = 'Comment configurer et utiliser l’historique bash'
date = 2023-02-22 00:00:00 +0100
categories = ['bash']
+++
*L’historique permet de conserver les dernières commandes tapées dans un shell bash. C’est très utile pour retrouver certaines commandes oubliées, éviter de devoir les ré-écrire ou regarder rapidement ce qu’un autre administrateur a fait sur le système.*

- [history](#history)
    - [Introduction](#introduction)
    - [Utilisation](#utilisation)
        - [History - Reverse-Search](#history---reverse-search)
        - [!! La dernière commande](#-la-dernière-commande)
        - [! + Numero](#--numero)
        - [! + lettres](#--lettres)
    - [Effacer history](#effacer-history)
    - [Sauvegarder history](#sauvegarder-history)
    - [Importer une sauvegarde dans history](#importer-une-sauvegarde-dans-history)
    - [Réutiliser les arguments de la commande précédente](#réutiliser-les-arguments-de-la-commande-précédente)
    - [Affichage  alphabétique de l'historique des commandes](#affichage-alphabétique-de-lhistorique-des-commandes)
    - [Prise en charge historique, modification bashrc et inputrc](#prise-en-charge-historique-modification-bashrc-et-inputrc)
    - [Lister history](#lister-history)
    - [Rechercher dans l’historique Bash avec Ctrl+R](#rechercher-dans-l’historique-bash-avec-ctrlr)

## history 

![Texte alternatif](bash-logo-white.png){:height="100"}

*Copie du tutoriel Debian Facile [history](https://debian-facile.org/doc:programmation:bash:history)*

* [Comment utiliser les commandes et les extensions de l'historique Bash sur un VPS Linux](https://getdoc.wiki/Comment_utiliser_les_commandes_et_les_extensions_de_l%27historique_Bash_sur_un_VPS_Linux)
* [Comment configurer et utiliser efficacement l’historique bash](https://blog.madrzejewski.com/astuce-historique-bash-linux/)

### Introduction 

La commande history permet de visualiser l’ensemble des 500 dernières commandes que vous avez saisies dans votre console.

  * History dans le terminal <user> relève les commandes de l'user.
  * History dans le terminal <root> relève les commandes de root.

### Utilisation 

La commande s'utilise habituellement dans un terminal en tapant sur <key>Up</key> pour remonter le cours des commandes précédentes.

Nous pouvons également l'invoquer textuellement en tapant :

    history

et obtenir la liste numérotée des 500 dernières commandes lancées

```
  4 cat base.tex
  5 ls -al
  6 cd Desktop/
  7 cd Work/
  8 pwd
  9 history
```

#### History - Reverse-Search 

Terminal ouvert, en tapant : `C-r` nous passons en :
   
   (reverse-i-search)`reco': 

Et en tapant maintenant les premières lettres de la commande recherchée, celle-ci s'autocomplète intégralement.  
Pour remonter davantage dans l'historique sur le thème des premières lettres, il suffit de taper de nouveau sur `>C-r`  pour lister la suite des commandes similaires.

Astuce également valable dans un shell zsh. 

#### !! La dernière commande 

La commande `!!` permet de rappeler la dernière commande passée, tout comme `Up`

    !!
    
```
  4 cat base.tex
  5 ls -al
  6 cd Desktop/
  7 cd Work/
  8 pwd
  9 history
```

Normal, la dernière commande passée était bien `history`

#### ! + Numero 

La commande `!numéro` permet d’atteindre la commande à droite du numéro.

Essayez en tapant  `!8`, j’obtiens ici `pwd` qui est la huitième ligne de commande en mémoire dans **history**.

    !8<
    
``` 
  pwd
  /home/user
```

#### ! + lettres 

La commande `!lettres` permet d’atteindre la commande à droite du numéro  
    !ca
    
j’obtiens ici `cat base.tex` :

    !ca

``` 
  cat base.tex
  /home/cobex4
```

>Si on veut juste visualiser la commande sans l'executer, on ajoute `:p` après la ou les première lettre

    !ca:p
    cat base.tex 

### Effacer history 

Pour effacer l’historique, on utilise l’option `-c`

    history -c 
    history 

```
  4  history
  cobex4@pc:~>
```

### Sauvegarder history 

Pour sauvegarder votre historique dans un fichier .txt, faites simplement :

    history > history.txt 

Pour lire en console le fichier obtenu :

    more history.txt 

```
   1  ssh mattux@chubaka
   2  ssh mattux@r2d2
   3  su
   4  cd backup/
   5  mv .mozilla-thunderbird/ /home/mattux/
   6  mv .ssh/ /home/mattux/
   7  ls -la /home/mattux/
   8  cd ../.ssh/
   9  ls -l
   10  cp -r ../backup/.ssh/ /home/mattux/
   11  ls -l
```

Voilà votre fichier peut être trés long tout dépend du nombre de commandes que vous avez fait.

### Importer une sauvegarde dans history 

Par exemple, pour importer votre fichier **history.txt** précédemment 

    history -r history.txt 

>Cela va ajouter le contenu de **history.txt** dans le fichier **.bash_history** en cours, pas le remplacer. 

### Réutiliser les arguments de la commande précédente 

Il est possible de réutiliser les arguments de la commande précédente. Cela peut être utile en cas de faute de frappe par exemple.

Ainsi, on rappelle tous les arguments avec `!*`, le premier avec `!^`, le dernier avec `!$`, le nième avec `!!:n` et du nième au xième avec ''!!:n-x''.

Par exemple, je fais une faute de frappe dans une commande

    eco "Salut"  
       eco : commande introuvable

Pour corriger

    echo !*
    echo "Salut"
       Salut

Autre exemple, je me trompe d'option. Je voulais lister tous les fichiers installé par un paquet avec dpkg, mais j'utilise la mauvaise option

    dpkg -S libsdl2-image-dev

J'aurais du utiliser l'option **-L**. Pas besoin de tout retaper

    dpkg -L !$

Il est même possible de rappeler ces arguments en les modifiant. Imaginons que vous souhaitiez afficher le contenu de  *monFichierAvec1NomSuperLong.txt*

    cat monFichierAvec1NomSuperLong.txt
    
Ce n'est pas le bon fichier, vous vouliez en fait le contenu de *monFichierAvec2NomSuperLong.txt* : Pas besoin de tout retaper, vous pouvez réaliser l'opération avec

    cat !!:1:s/1/2/

Un poil d'explications :-p `!!:1` rappel l'argument 1 de la dernière commande, donc *monFichierAvec1NomSuperLong.txt*. On demande ensuite de substituer (**s**) la première occurrence de **1** pour la remplacer par **2**


### Affichage  alphabétique de l'historique des commandes 

Habituellement, nous pouvons **remonter l'historique** de nos commandes dans le terminal avec `Up`... Simple.  
Pourquoi ne pas remonter en utilisant les lettres alphatétiques débutant ces commandes 

Par exemple tu écris :

   g

puis tu utilises la touche `Up` et s'affichent à la suite toutes les commandes qui commencent par "g"  et qui se trouvent dans le **~/.bash_history**

### Prise en charge historique, modification bashrc et inputrc

Pour ce faire, éditer avec nano :

1- à la fin de `~/.bashrc`

```bash
# appel alphabétique commandes
shopt -s histappend
PROMPT_COMMAND='history -a'
```

2 - dans `~/.inputrc` (à créer au lieu d'utiliser `/etc/inputrc`)

```bash
# SANS la touche SHIFT
"\e[A": history-search-backward
"\e[B": history-search-forward

# AVEC la touche SHIFT
"\e[1;2A": history-search-backward
"\e[1;2B": history-search-forward

```

pour root effectuer les mêmes commandes (`/root/.bashrc` et `/root/.inputrc`)

POUR tous les utilisateurs ,y compris root et avec touche SHIFT

```bash
echo "
# appel alphabétique commandes
shopt -s histappend
PROMPT_COMMAND='history -a'" | tee -a $HOME/.bashrc

echo "
# appel alphabétique commandes
shopt -s histappend
PROMPT_COMMAND='history -a'" | sudo tee -a /root/.bashrc

echo '
# AVEC la touche SHIFT
"\e[1;2A": history-search-backward
"\e[1;2B": history-search-forward
' | sudo tee -a /etc/inputrc
```

Redémarrer le terminal la prise en compte

### Lister history 

Pour obtenir la liste historique du thème recherché, tapez :

    history | grep themerecherche 

### Rechercher dans l’historique Bash avec Ctrl+R 

Exemple :
Dans votre terminal si vous tapez `Ctrl + r`, vous obtiendrez :

```
(reverse-i-search)`':
```

Ensuite tapez un mot clé correspondant à ce que vous recherchez dans l’historique

```
(reverse-i-search)`': gcc -g test.c
```

Ci-dessus, on peut voir qu’il m’affiche une ligne commande entière (avec les arguments) qui contient l’expression « gcc ». Cette ligne de commande correspond à la dernière ligne de commande que j’ai tapé et qui contenait le mot clé gcc.

En appuyant une deuxième fois sur `Ctrl + R`, je remonte dans l’historique Bash enregistré. Il m’affiche :

```
(reverse-i-search)`': gcc test.c
```

Ci-dessus, il m’affiche une autre ligne de commande que j’ai tapé dernièrement et qui contient également l’expression « gcc ».

En appuyant successivement sur `CTRL + r` vous pouvez remonter dans l’historique des commandes.

Un autre exemple, si je tape `Ctrl +r vfat`, il m’affiche :

```
(reverse-i-search)`': cd /mnt/vfat
```

Il a trouvé la dernière commande que j’avais tapé et qui comprenait le mot clé vfat.

Si vous souhaitez sélectionner la ligne de commande qu’il vous propose, rien de plus simple, soit vous appuyez sur la touche entrée pour éxecuter la commande, soit vous l’éditer directement. Pour l’éditer, déplacer le curseur avec les flèches droite et gauche. Pour annuler appuyer sur `Ctrl +c`
