+++
title = 'Plymouth - Processus de démarrage graphique'
date = 2023-10-14 00:00:00 +0100
categories = ['archlinux']
+++
*[Plymouth](https://wiki.ubuntu.com/Plymouth) est une application qui permet d'afficher une animation graphique pendant le processus de démarrage du système.L'idée principale est de supprimer le scintillement ou le mur de textes défilants pendant le processus de démarrage de Linux en utilisant une belle animation graphique. Cela aide les utilisateurs en général à attendre que l'écran de connexion apparaisse pendant le démarrage du système. Ce programme dispose d'un mode texte de secours lorsqu'il ne parvient pas à charger l'animation graphique(Plymouth utilise principalement KMS pour afficher les graphiques, mais sur les systèmes UEFI, il peut utiliser le framebuffer EFI)*

- [Plymouth](#plymouth)
    - [Installation de Plymouth](#installation-de-plymouth)
        - [mkinitcpio](#mkinitcpio)
        - [dracut](#dracut)
    - [Configurer Plymouth](#configurer-plymouth)
        - [Changer le thème](#changer-le-thème)
        - [Installer un nouveau thème](#installer-un-nouveau-thème)
        - [Thème plymouth EndeavourOS](#thème-plymouth-endeavouros)
        - [Ajouter un délai](#ajouter-un-délai)
    - [Trucs et astuces](#trucs-et-astuces)
        - [Afficher les messages de démarrage](#afficher-les-messages-de-démarrage)
        - [Transition en douceur](#transition-en-douceur)
        - [Afficher aperçu du thème](#afficher-aperçu-du-thème)
        - [Image BGRT manquante](#image-bgrt-manquante)
        - [Ralentir le démarrage pour afficher l'animation complète](#ralentir-le-démarrage-pour-afficher-lanimation-complète)
    - [Dépannage](#dépannage)
        - [Désactiver avec les paramètres du noyau](#désactiver-avec-les-paramètres-du-noyau)
        - [Débogage](#débogage)
        - [L'invite de mot de passe ne se met pas à jour](#linvite-de-mot-de-passe-ne-se-met-pas-à-jour)
        - [L'affichage n'est pas centré](#laffichage-nest-pas-centré)

## Plymouth

### Installation de Plymouth

Le paquet **plymouth** se trouve dans l'Arch User Repository (AUR) et vous pouvez l'installer en utilisant yay.

    yay -S plymouth

L'étape suivante consiste à demander au Kernel de faire en sorte que le processus de démarrage soit silencieux (pas trop de messages) et d'afficher l'écran d'accueil (splash screen) qui présente le logo/animation de Plymouth.

Par défaut, Plymouth enregistre les messages de démarrage dans `/var/log/boot.log`, et n'affiche pas l'écran d'accueil graphique.

*    Si vous voulez voir l'écran de démarrage, ajoutez **splash** aux paramètres du noyau.
*    Si vous souhaitez un démarrage silencieux, ajoutez également **quiet**.
*    Si vous voulez désactiver la journalisation, ajoutez **plymouth.nolog**.


#### mkinitcpio

Ajoutez **plymouth** au tableau **HOOKS** dans `mkinitcpio.conf` 

    /etc/mkinitcpio.conf

    HOOKS=(... plymouth ...)

#### dracut

Après avoir installé Plymouth, dracut le détectera automatiquement et l'ajoutera à vos images initramfs. 

<u>Si la détection automatique échoue</u>, vous pouvez forcer dracut à inclure Plymouth en ajoutant la ligne suivante à votre configuration dracut :

    /etc/dracut.conf.d/myflags.conf

    add_dracutmodules+="plymouth"

Pour la prise en charge , il faut ajouter splash quiet au fichier `/etc/kernel/cmdline`

    nvme_load=YES nowatchdog rw splash quiet rd.luks.uuid=ea061853-5851-40d6-8dee-c7245a62da26 root=/dev/mapper/vg0-lvroot

### Configurer Plymouth

Les thèmes sant dans le dossier `/usr/share/plymouth/themes/`  
Plymouth peut être configuré dans le fichier `/etc/plymouth/plymouthd.conf`   
Vous pouvez voir les valeurs par défaut dans `/usr/share/plymouth/plymouthd.defaults`

#### Changer le thème

Le paquet installe plusieurs thèmes Plymouth. Vous pouvez les lister via la commande suivante.

    plymouth-set-default-theme -l

```
bgrt
details
fade-in
glow
kameleon-eos
script
solar
spinfinity
spinner
text
tribar
```

Plymouth définition des thèmes :

*    BGRT : Une variante de Spinner qui conserve le logo OEM si disponible (BGRT signifie Boot Graphics Resource Table)
*    Fade-in : "Thème simple qui apparaît et disparaît avec des étoiles scintillantes"
*    Glow : "Thème d'entreprise avec progression du démarrage sous forme de diagramme circulaire suivi d'un logo émergent lumineux"
*    Script : "Exemple de script plugin" (Malgré la description, cela semble être un thème de logo Arch assez sympa)
*    Solar : "Thème spatial avec une étoile bleue flamboyante violente"
*    Spinner : "Thème simple avec un spinner de chargement"
*    Spinfinity : "Thème simple qui montre un signe infini rotatif au centre de l'écran"
*    Tribar : "Thème mode texte avec barre de progression tricolore"
*    ( Texte : "Thème mode texte avec barre de progression tricolore")
*    ( Détails : "Thème de secours verbeux")

Par défaut, le thème bgrt est sélectionné. 

#### Installer un nouveau thème

Le thème peut être modifié en éditant le fichier de configuration, par exemple :

    /etc/plymouth/plymouthd.conf

    [Daemon]
    Theme=fade-in

ou par la commande ci-dessous.

    sudo plymouth-set-default-theme -R <nom du thème>

Par exemple, si vous souhaitez définir le thème bgrt, vous devez utiliser la commande suivante.

```bash
sudo plymouth-set-default-theme -R bgrt  # mkinitcpio
sudo plymouth-set-default-theme bgrt     # dracut 
# dracut
sudo reinstall-kernels
```

ATTENTION !!!  
Chaque fois qu’un thème est modifié, il faut le reconstruire initrd.  
L'option  `-R`  garantit que les les initramfs sont reconstruits  
Régénérez les initramfs manuellement si vous utilisez dracut.
{: .prompt-warning }

#### Thème plymouth EndeavourOS

Un thème Plymouth `plymouth-theme-endeavouros` qui utilise un fond d'écran et une palette de couleurs EndeavourOS. Supporte un dialogue de mot de passe, la progression du démarrage, l'état de capslock, etc.

**Installation**  

La façon recommandée d'utiliser ce thème dans EndeavourOS est de l'utiliser avec le paquet AUR `plymouth-git` qui est plus récent et a plus de fonctionnalités (y compris la prise en charge de l'état capslock).

Installer le thème et la version recommandée de Plymouth

    yay -Sy plymouth-git plymouth-theme-endeavouros

Ajouter le paramètre kernel "splash" pour que Plymouth soit affiché au démarrage

    sudo sed -i '$ s/$/ splash/' /etc/kernel/cmdline

Ainsi, Plymouth et le thème sont immédiatement installés dans votre initramfs pour le prochain démarrage. 

Les futures mises à jour du noyau incluront automatiquement Plymouth et ce thème.

    sudo reinstall-kernels

#### Ajouter un délai

Plymouth dispose d'une option de configuration pour retarder l'écran de démarrage :

    /etc/plymouth/plymouthd.conf

    [Daemon]
    ShowDelay=5

Sur les systèmes qui démarrent rapidement, vous ne verrez peut-être qu'un scintillement de votre thème de démarrage avant que votre DM ou votre invite de connexion ne soit prête. Vous pouvez définir `ShowDelay` un intervalle (en secondes) plus long que votre temps de démarrage pour éviter ce scintillement et afficher uniquement un écran vide. La valeur par défaut est 0 seconde, vous ne devriez donc pas avoir besoin de modifier cette valeur pour voir votre splash plus tôt lors du démarrage.

### Trucs et astuces

#### Afficher les messages de démarrage

Pendant le démarrage, vous pouvez passer aux messages de démarrage en appuyant sur la touche **Esc** 

#### Transition en douceur

GDM prend en charge une transition en douceur dès la sortie de la boîte.

Pour les autres gestionnaires d'affichage, vous pouvez obtenir une transition presque fluide avec l' extrait suivant pour `display-manager.service` :

    /etc/systemd/system/display-manager.service.d/plymouth.conf

```
[Unit]
Conflicts=plymouth-quit.service
After=plymouth-quit.service rc-local.service plymouth-start.service systemd-user-sessions.service
OnFailure=plymouth-quit.service

[Service]
ExecStartPre=-/usr/bin/plymouth deactivate
ExecStartPost=-/usr/bin/sleep 30
ExecStartPost=-/usr/bin/plymouth quit --retain-splash
```

#### Afficher aperçu du thème

Les thèmes peuvent être prévisualisés sans reconstruire initrd, appuyez sur Ctrl+Alt+F6 pour passer à un terminal texte, connectez-vous en tant que **root** et saisir :

```
plymouthd
plymouth --show-splash
```

Pour quitter l'aperçu, appuyez Ctrl+Alt+F6 à nouveau et saisir :

```
plymouth --quit
```

Changer l'image d'arrière-plan

Vous pouvez ajouter une image d'arrière-plan pour les thèmes en deux étapes (tels que spinner et bgrt). Placez simplement l'image souhaitée dans `/usr/share/plymouth/themes/spinner/background-tile.png`   N'oubliez pas de régénérer l'initrd une fois le thème changé.  

#### Image BGRT manquante

Si vous utilisez le thème BGRT mais que l'UEFI ne fournit pas de logo de fournisseur, vous pouvez placer une image de secours `/usr/share/plymouth/themes/spinner/bgrt-fallback.png` pour l'afficher à la place.

Vous pouvez également définir les paramètres suivants pour conserver l'arrière-plan du micrologiciel :

    /etc/plymouth/plymouthd.conf

    UseFirmwareBackground=true

#### Ralentir le démarrage pour afficher l'animation complète

Sur les systèmes avec un temps de démarrage très rapide, il peut être nécessaire d'ajouter un délai `plymouth-quit.service` avec un [extrait de code](https://wiki.archlinux.org/title/Drop-in_snippet) indiquant `ExecStartPre=/usr/bin/sleep 5` si l'affichage de l'animation entière est souhaité. Voir cet [article sur Reddit](https://old.reddit.com/r/archlinux/comments/u5fjbi/how_do_i_make_my_boot_time_slower/) 

Alternativement, il est possible d'utiliser un nouveau service systemd démarrant en même temps que plymouth et attendant toute la durée nécessaire à l'animation. Cette méthode garantira que des incohérences dans le temps de démarrage ne seront pas perçues, car il ne s'agit pas de temps ajouté après l'animation mais d'un délai s'exécutant pendant l'animation.

    /etc/systemd/system/plymouth-wait-for-animation.service

```
[Unit]
Description=Waits for Plymouth animation to finish
Before=plymouth-quit.service display-manager.service

[Service]
Type=oneshot
ExecStart=/usr/bin/sleep duration_of_your_animation

[Install]
WantedBy=plymouth-start.service
```

Activez ensuite le service.

>Remarque : Aucune des méthodes précédentes ne fonctionnera si vous démarrez Plymouth à partir d'initramfs.

### Dépannage

#### Désactiver avec les paramètres du noyau

Si vous rencontrez des problèmes lors du démarrage, vous pouvez désactiver temporairement Plymouth avec les paramètres du noyau suivants :

    plymouth.enable=0 disablehooks=plymouth

#### Débogage

Pour écrire la sortie de débogage dans `/var/log/plymouth-debug.log`, ajoutez le paramètre de noyau suivant :

    plymouth.debug

#### L'invite de mot de passe ne se met pas à jour

Lors de l'utilisation systemd à la place du hooks udev  dans Mkinitcpio , l'invite de mot de passe peut ne pas être mise à jour sur les thèmes qui le gèrent via les scripts Plymouth.

Vous pouvez essayer de passer à la version de développement plymouth-git AUR ou utiliser des substituts de [Mkinitcpio#Common hooks](https://wiki.archlinux.org/title/Mkinitcpio#Common_hooks) .

#### L'affichage n'est pas centré

Certains thèmes peuvent avoir du mal à centrer l'affichage lorsque plusieurs moniteurs sont activés lors du démarrage.

Vous pouvez utiliser [Kernel mode setting#Forcing modes](https://wiki.archlinux.org/title/Kernel_mode_setting#Forcing_modes) pour désactiver des moniteurs spécifiques.

