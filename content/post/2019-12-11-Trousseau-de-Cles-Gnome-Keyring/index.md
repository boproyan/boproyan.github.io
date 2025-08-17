+++
title = 'XFCE/GNOME Porte-clés ou trousseau (gnome-keyring)'
date = 2019-12-11 00:00:00 +0100
categories = ['archlinux', 'chiffrement']
+++
# GNOME/Porte-clés (GNOME/Keyring)

<https://wiki.archlinux.org/index.php/GNOME/Keyring>  
*Le porte-clés GNOME est "une collection de composants dans GNOME qui stockent des secrets, des mots de passe, des clés, des certificats et les mettent à la disposition des applications."*

## Installation

Lorsque vous utilisez GNOME, gnome-keyring est installé automatiquement en tant que partie du groupe gnome  
Sinon, installez le paquet **gnome-keyring**   
Installez **libsecret** pour permettre aux applications d'utiliser vos trousseaux de clés  
libgnome-keyring est obsolète, cependant, certaines applications peuvent l'exiger.

Les utilitaires supplémentaires liés au trousseau de clés GNOME comprennent:

*    secret-tool - Accédez au trousseau de clés GNOME (et à tout autre service implémentant l' API DBus Secret Service ) à partir de la ligne de commande (https://wiki.gnome.org/Projects/Libsecret)
*    gnome-keyring-query - Fournit un outil en ligne de commande simple pour interroger les mots de passe à partir du magasin de mots de passe du porte-clés GNOME. (utilise le porte-clés libgnome obsolète) 
*    gkeyring - Recherchez les mots de passe à partir de la ligne de commande. (utilise le porte-clés libgnome obsolète) 

## Gérer à l'aide de l'interface graphique

Vous pouvez gérer le contenu du trousseau de clés GNOME à l'aide de Seahorse. Installez- le avec le paquet **seahorse**

Il est possible de laisser le mot de passe du trousseau de clés GNOME vide ou de le changer.  
Dans seahorse, dans le menu déroulant "Affichage", sélectionnez "Par porte-clés". Sur l'onglet Mots de passe, faites un clic droit sur "Mots de passe: connexion" et choisissez "Changer le mot de passe".   Saisissez l'ancien mot de passe et laissez vide le nouveau mot de passe.   
Vous serez averti de l'utilisation d'un stockage non crypté; continuer en appuyant sur «Utiliser un stockage non sécurisé».

## Utiliser le trousseau de clés en dehors de GNOME

### Sans gestionnaire d'affichage

#### Connexion automatique

Si vous utilisez la connexion automatique, vous pouvez désactiver le gestionnaire de trousseaux en définissant un mot de passe vierge sur le trousseau de connexion.  
Remarque: les mots de passe sont stockés non chiffrés dans ce cas.  

#### Connexion à la console

Lorsque vous utilisez une connexion basée sur la console, le démon de trousseau de clés peut être démarré par **PAM** ou **xinitrc** .  
PAM peut également déverrouiller automatiquement le trousseau lors de la connexion.

#### Méthode PAM

Démarrez le démon **gnome-keyring-daemon** depuis **/etc/pam.d/login** :

Ajoutez `auth optional pam_gnome_keyring.so` à la fin de la section d' auth et `session optional pam_gnome_keyring.so auto_start` à la fin de la section de session .

    /etc/pam.d/login 

```
#%PAM-1.0
 
auth       required     pam_securetty.so
auth       requisite    pam_nologin.so
auth       include      system-local-login
auth       optional     pam_gnome_keyring.so
account    include      system-local-login
session    include      system-local-login
session    optional     pam_gnome_keyring.so auto_start
```

Pour SDDM , éditez plutôt le fichier de configuration **/etc/pam.d/sddm**

Ensuite, pour GDM , ajoutez le password optional pam_gnome_keyring.so à la fin de /etc/pam.d/passwd .

    /etc/pam.d/passwd 

```
#%PAM-1.0

#password	required	pam_cracklib.so difok=2 minlen=8 dcredit=2 ocredit=2 retry=3
#password	required	pam_unix.so sha512 shadow use_authtok
password	required	pam_unix.so sha512 shadow nullok
password	optional	pam_gnome_keyring.so
```

>Remarque:  
Pour utiliser le déverrouillage automatique, le <u>même mot de passe pour le compte d'utilisateur et le trousseau de clés</u> doivent être définis.  
Vous aurez toujours besoin du code dans **~/.xinitrc** ci-dessous pour exporter les variables d'environnement requises. 

#### Méthode xinitrc

Démarrez le démon gnome-keyring-daemon à partir de xinitrc :

    ~/.xinitrc

```
eval $(/usr/bin/gnome-keyring-daemon --start --components=pkcs11,secrets,ssh)
export SSH_AUTH_SOCK
```

### Avec un gestionnaire d'affichage

Lorsque vous utilisez un gestionnaire d'affichage, le trousseau de clés fonctionne immédiatement pour la plupart des cas. Les gestionnaires d'affichage suivants déverrouillent automatiquement le trousseau de clés une fois connecté:

*    GDM
*    LightDM
*    LXDM
*    SDDM 

>Pour GDM et LightDM, notez que le trousseau de clés doit être nommé **login** pour être automatiquement déverrouillé.

Pour activer le trousseau de clés pour les applications exécutées via le terminal, telles que SSH, ajoutez ce qui suit à votre ~/.bash_profile, ~/.zshenv, ou similaire:

    ~/.bash_profile

```
if [ -n "$DESKTOP_SESSION" ];then
    eval $(gnome-keyring-daemon --start)
    export SSH_AUTH_SOCK
fi
```

    ~/.config/fish/config.fish

```
if test -n "$DESKTOP_SESSION"
    set (gnome-keyring-daemon --start | string split "=")
end
```
