+++
title = 'Grub "Configure GRUB2 Boot Loader settings", clavier FR et un mot de passe'
date = 2019-12-19 00:00:00 +0100
categories = ['cli']
+++
# Grub

## [Configure GRUB2 Boot Loader settings-Lien HS](/files/html/Configure GRUB2 Boot Loader settings.htm)

## Clavier FR

Par défaut ,le clavier est qwerty , on le passe en azerty  
Tout d'abord, vérifiez que vous utilisez GRUB 2 (GRUB 0.x fonctionne différemment).

    grub-install --version
        grub-install (GRUB) 2.04


Générer un fichier de disposition de clavier GRUB. Ci-dessous se trouve la commande pour un clavier français. Pour les autres langues, vérifiez /usr/share/X11/xkb/symboles/.  
Le choix du nom de fichier n'est pas important (vous pouvez changer bepo).

    sudo grub-kbdcomp -o /boot/grub/bepo.gkb fr

Editez **/etc/default/grub** avec les droits root 

```
#GRUB_HIDDEN_TIMEOUT=0
GRUB_TERMINAL_INPUT="at_keyboard".
```

Editez **/etc/grub.d/40_custom** avec les droits root 

```
#!/bin/sh
exec tail -n +3 +3 0

claviers insmod
keymap /boot/grub/bepo.gkb
```

Enfin :

    sudo update-grub

>Remarque : Oubliez immédiatement l'utilisation de la touche Shift pour afficher le menu GRUB ! 

## Mot de passe

>Il faut impérativement avoir validé le clavier FR afin de pouvoir saisir correctement le mot de passe sous grub

Pour protéger l’accès de grub avec un mot de passe, il faut un utilisateur auquel associer ce mot de passe. Dans le tuto ce sera donc “yann“.
Cet utilisateur est complétement indépendant des utilisateurs crées sur votre système d’exploitation. Il ne sera associé qu’à GRUB.

Création du mot de passe , il faut être **root**, ou utiliser **sudo**  
Pour créer un mot de passe hashé, on utilise la commande suivante,

    sudo grub-mkpasswd-pbkdf2

```
Entrez le mot de passe : 
Entrez de nouveau le mot de passe : 
Le hachage PBKDF2 du mot de passe est grub.pbkdf2.sha512.10000.847E40FD9C433D29D2585405E4439783E1B2F213BA4947BCF6379548133CDC5C39FC247F8DF3D060E7AD550A38EF96778B3D7725952F500EA84B81647A304C72.FD0F52EC6D1F15A141915EE191D84450159DBEF4BFFFD9368F709BBC83BE3ACD7A903729E9EAB344DF7DB8B8692F4F2F0605B2C4790B55E1B75C8F9CA9E5EA78
```

Une fois le mot de passe crée, on édite le fichier **/etc/grub.d/00_header**, afin de lui ajouter ceci en fin de fichier 

```
cat << EOF
set superusers="yann"
password_pbkdf2 hedi grub.pbkdf2.sha512.10000.847E40FD9C433D29D2585405E4439783E1B2F213BA4947BCF6379548133CDC5C39FC247F8DF3D060E7AD550A38EF96778B3D7725952F500EA84B81647A304C72.FD0F52EC6D1F15A141915EE191D84450159DBEF4BFFFD9368F709BBC83BE3ACD7A903729E9EAB344DF7DB8B8692F4F2F0605B2C4790B55E1B75C8F9CA9E5EA78
EOF
```

Et enfin, pour prendre en compte le changement

    sudo update-grub

Voilà, vous n’avez plus qu’à redémarrer.

    sudo systemctl reboot

