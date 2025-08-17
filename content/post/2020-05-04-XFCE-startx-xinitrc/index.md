+++
title = 'XFCE startx xinitrc'
date = 2020-05-04 00:00:00 +0100
categories = xfce
+++
## startx

Pour démarrer un gestionnaire de fenêtre sans gestionnaire de connexion, le moyen le plus simple reste la commande startx, il faut lui indiquer quel gestionnaire lancer.

### Installation

Si pas de gestion xorg installée

    pacman -S xorg-xinit
    
Avec xfce installé et lightdm pour la connexion  
On désactive le gestionnaire de connexion pour qu'il ne se lance pas au prochain démarrage
    sudo sytemctl disable lightdm    

### Fichier de démarrage ~/.xinitrc 

Le fichier par défaut est **/etc/X11/xinit/xinitrc** qui lance twm, xterm et xclock par défaut (installer xorg-twm, xorg-xclock et xterm si vous prévoyez de vous en servir).

Pour votre utilisateur vous aurez besoin de recopier ce fichier dans son HOME et de le modifier:
 
    cp /etc/X11/xinit/xinitrc ~/.xinitrc

Le principe est simple, il suffit de conserver l'ensemble de son contenu et de commenter ou supprimer les lignes suivants *twm* jusqu'à la fin puis de rajouter ce dont vous avez besoin en dernier.

    ~/.xinitrc

```
# start some nice programs

if [ -d /etc/X11/xinit/xinitrc.d ] ; then
 for f in /etc/X11/xinit/xinitrc.d/?*.sh ; do
  [ -x "$f" ] && . "$f"
 done
 unset f
fi

#twm &
#xclock -geometry 50x50-1+1 &
#xterm -geometry 80x50+494+51 &
#xterm -geometry 80x20+494-0 &
#exec xterm -geometry 80x66+0+0 -name login

exec dbus-launch xfce4-session
```

## Démarrer xfce4 automatiquement

***Un arrêt ou redémarrage de la machine depuis un environnement graphique peut nécessiter de quitter l'environnement avant d'arrêter la machine.***

Créer un fichier **/etc/systemd/system/startx@.service**  

    /etc/systemd/system/startx@.service

```
[Unit]
Description=startx automatique pour l'utilisateur %I
After=graphical.target systemd-user-sessions.service

[Service]
User=%I
WorkingDirectory=%h
PAMName=login
Type=simple
ExecStart=/bin/bash -l -c startx

[Install]
WantedBy=graphical.target
```

Pour tester :

    systemctl start startx@utilisateur.service

Pour activer :

    systemctl enable startx@utilisateur.service

### Démarrer Xorg après identification 

Lancer automatiquement `startx` juste après vous être authentifié sur la console/tty, par exemple uniquement à partir du **tty1**. Ajouter le code suivant à la fin du fichier **$HOME/.bash_profile** :
 
    [[ $(tty) == '/dev/tty1' ]] && startx

Pour poser une question si on est dans une console et qu'aucune instance de X n'est détectée :

```bash
 if [[ -t 0 && $(tty) =~ /dev/tty ]] && ! pgrep -u $USER startx &> /dev/null;then
     echo "Aucune session X11 détectée, voulez vous en lancer une ? [O|n]"
     read -n 1 start_x
     if [[ $start_x == "n" ]];then
         echo "X11 ne sera pas lancé."
     else
 	  startx
     fi
 fi
```

Si aucune session X n'est démarrée, il vous sera alors proposer d'en lancer une lorsque vous vous identifiez. Saisir `n` pour l'empêcher ou appuyez sur n'importe quelle autre touche
