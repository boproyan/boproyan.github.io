+++
title = 'VNC - x11vnc prise de contrôle bureau à distance'
date = 2022-10-19 00:00:00 +0100
categories = vnc
+++
*Il peut être utile de prendre la main à distance sur un PC équipé de GNU/Linux pour aider un de nos amis dans la détresse, ou pour accéder à un ordinateur de la maison constamment allumé, sans écran.*

## x11vnc 

### Installation

On va installer donc le paquet **x11vnc** qui permet à un utilisateur de se connecter sur sa machine à distance à la manière de RDP sur Windows (Connexion Bureau à distance).

Ubuntu et dérivés :

    sudo apt-get install x11vnc

Archlinux Manjaro

    sudo pacman -S x11vnc

Et voila, c'est installé !


### Utiliser x11vnc

Générer un mot de passe

Pour protéger la prise de main à distance, il est recommandé de créer un mot de passe pour permettre la prise de main (où ******** est le mot de passe) :

    x11vnc -storepasswd "*******" ~/.vnc_passwd


### Lancement Manuel

Pour lancer le serveur VNC, c'est en console, en session utilisateur :

    x11vnc -many -rfbauth ~/.vnc_passwd -xkb

Il est possible de placer dans son `.bashrc` un alias de manière à ne taper qu'un mot mnémotechnique :

    alias assistance='x11vnc -many -rfbauth /home/yann/.vnc_passwd -display :0 -listen localhost'

>Faire **Ctrl+C** pour arrêter le serveur vnc.


### Tunnel SSH

>Vous devez avoir installé et configuré SSH.

Utilisez le drapeau `-localhost` avec x11vnc pour qu'il se lie à l'interface locale. Une fois que c'est fait, vous pouvez utiliser SSH pour tunneliser le port ; puis, connectez-vous à VNC via SSH.

    ssh -t -L 5900:localhost:5900 remote_host 'x11vnc -localhost -display :0'
    # exemple avec clés et port différent de 22 
    ssh -f -L 5900:localhost:5900 -p 56230 -i /home/yannick/.ssh/e6230 yann@192.168.0.32 'x11vnc -localhost -display :0'

>(Vous devrez probablement fournir des mots de passe/phrases de passe pour vous connecter de votre emplacement actuel à votre compte Unix remote_host ; nous supposons que vous avez un compte de connexion sur remote_host et qu'il fait tourner le serveur SSH)

Puis, dans une autre fenêtre du terminal de votre machine actuelle, exécutez la commande :

    vncviewer -PreferredEncoding=ZRLE localhost:0

Si le curseur est mal affiché, c'est possible de mettre l'option `-cursor` à la ligne de commande x11vnc  
`Le programme écoute sur le port 5900. Il faut penser à ouvrir le parefeu sur ce port en TCP`{: .prompt-info }

**Comment configurer x11vnc pour accéder à un écran de connexion graphique ?**  
En supposant que vous utilisez **lightdm** pour la connexion, vous pouvez résoudre ce problème en démarrant x11vnc avec la commande :  
`sudo x11vnc -xkb -noxrecord -noxfixes -noxdamage -display :0 -auth /var/run/lightdm/root/:0 -usepw`

Exemple via le tunnel SSH :  
`ssh -f -L 5900:localhost:5900 -p 52022 marina@82.64.153.201 'sudo x11vnc -xkb -noxrecord -noxfixes -noxdamage -display :0 -auth /var/run/lightdm/root/:0 -usepw'`  
Mot de passe VNC dans `/root/.vnc/passwd`

Pour ne pas avoir à saisir le mot de passe de l'utilisateur distant, donner les droits sudo pour l'exécution du programme x11vnc dans le fichier `/etc/sudoers`  
Exemple pour un utilisateur nommé 'test'  
`test ALL=(ALL) NOPASSWD: /usr/bin/x11vnc`

### Script - Tunnel SSH

Ouverture distant et redirection du port 5900 via tunnel ssh, reprise de la main (`nohup`)   
On mémorise le processus ssh (`pidof`)  
Lancement de `vncviewer`   
En sortie du viewer, on tue le processus `ssh ...` avec `kill`

    nano vnc.sh 

```
#!/bin/bash

nohup ssh -f -L 5900:localhost:5900 -p 56230 -i /home/yannick/.ssh/e6230 yann@192.168.0.32 'x11vnc -localhost -display :0'
p=$(pidof -s ssh)
vncviewer -PreferredEncoding=ZRLE localhost:0
kill $p
echo "FIN"
 
exit 0
```

Droits en exécution

    chmod +x vnc.sh

## TigerVNC

[TigerVNC](https://wiki.archlinux.org/title/TigerVNC)

Implantation avec service utilisateur

Afin d'avoir un serveur VNC exécutant x0vncserver, qui est le moyen le plus simple pour la plupart des utilisateurs d'avoir rapidement accès à distance au bureau actuel, créez une unité systemd comme suit en remplaçant l'utilisateur et les options par ceux désirés :

    mkdir -p ~/.config/systemd/user
    nano ~/.config/systemd/user/x0vncserver.service

```
[Unit]
Description=Remote desktop service (VNC)

[Service]
Type=simple
#ExecStartPre=/bin/sh -c 'while ! pgrep -U "$USER" Xorg; do sleep 2; done'
ExecStart=/usr/bin/x0vncserver -rfbauth %h/.vnc/passwd

[Install]
WantedBy=default.target
```

La ligne ExecStartPre attend que Xorg soit démarré par ${USER}.

Pour vous connecter avec votre nom d'utilisateur et votre mot de passe, remplacez ExecStart par `/usr/bin/x0vncserver -PAMService=login -PlainUsers=${USER} -SecurityTypes=TLSPlain`

Start/enable le service x0vncserver.service   
