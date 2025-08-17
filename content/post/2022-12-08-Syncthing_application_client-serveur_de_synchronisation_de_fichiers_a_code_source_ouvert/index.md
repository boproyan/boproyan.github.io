+++
title = 'Syncthing outil réseau de synchronisation de fichiers peer-to-peer à code source ouvert'
date = 2022-12-08 00:00:00 +0100
categories = ['network']
+++
![Syncthing](syncthingnet-ar21.svg)  
*[Syncthing (site officiel)](https://syncthing.net/) est une application client/serveur de synchronisation de fichiers à code source ouvert, écrite en Go, mettant en œuvre son propre protocole d'échange de blocs, également libre. Toutes les communications de transit entre les nœuds de Syncthing sont cryptées en utilisant TLS, et tous les nœuds sont identifiés de manière unique avec des certificats cryptographiques.*

* [Wiki Archlinux - Syncthing](https://wiki.archlinux.org/title/syncthing)
* [Wiki Ubuntu - Syncthing](https://doc.ubuntu-fr.org/syncthing)
* [Install and Use Syncthing on Ubuntu 22.04|20.04|18.04](https://computingforgeeks.com/how-to-install-and-use-syncthing-on-ubuntu/)
* [Installation et configuration de Syncthing (it-connect)](https://www.it-connect.fr/installation-et-configuration-de-syncthing/)
* [How to Install Syncthing on Debian desktop/server](https://www.linuxbabe.com/debian/install-syncthing)

**Syncthing** est un outil de synchronisation de fichiers peer-to-peer open-source que vous pouvez utiliser pour synchroniser des fichiers entre plusieurs appareils (y compris un téléphone Android).
{: .prompt-info }

## Syncthing Archlinux

Installez le paquet syncthing

    sudo pacman -S syncthing  # Archlinux

Syncthing fournit une **Web-GUI** pour le contrôle et la surveillance. Des interfaces graphiques comme **Syncthing-GTK** et **Syncthing Tray** (fournis dans des paquets séparés) existent également.

### Démarrage automatique de Syncthing

Syncthing peut être installé soit en tant que service système systemd, soit en tant que service utilisateur systemd pour être exécuté automatiquement au démarrage.  

`Faire le choix entre A ou B`{: .prompt-info }

#### A-Service systemd système

Exécuter Syncthing en tant que service système assure qu'il est exécuté au démarrage même si l'utilisateur n'a pas de session active, il est destiné à être utilisé sur un serveur.  
Activez et démarrez le `syncthing@myusername.service` où `myusername` est le nom réel de l'utilisateur de Syncthing.

```shell
sudo systemctl enable syncthing@yann.service  # remplacer yann par votre nom utilisateur
sudo systemctl start syncthing@yann.service  # remplacer yann par votre nom utilisateur
sudo systemctl status syncthing@yann.service
```

Vérification

```
● syncthing@yann.service - Syncthing - Open Source Continuous File Synchronization for yann
     Loaded: loaded (/usr/lib/systemd/system/syncthing@.service; disabled; preset: disabled)
     Active: active (running) since Thu 2022-12-08 17:34:30 CET; 9s ago
       Docs: man:syncthing(1)
   Main PID: 4265 (syncthing)
      Tasks: 18 (limit: 19009)
     Memory: 40.5M
        CPU: 1.641s
     CGroup: /system.slice/system-syncthing.slice/syncthing@yann.service
             ├─4265 /usr/bin/syncthing serve --no-browser --no-restart --logflags=0
             └─4273 /usr/bin/syncthing serve --no-browser --no-restart --logflags=0

déc. 08 17:34:32 archyan syncthing[4265]: [JRGFW] INFO: Using discovery mechanism: IPv6 local multicast discovery on address [ff12::>
déc. 08 17:34:32 archyan syncthing[4265]: [JRGFW] INFO: TCP listener ([::]:22000) starting
déc. 08 17:34:32 archyan syncthing[4265]: [JRGFW] INFO: Loading HTTPS certificate: open /home/yann/.config/syncthing/https-cert.pem:>
déc. 08 17:34:32 archyan syncthing[4265]: [JRGFW] INFO: Creating new HTTPS certificate
déc. 08 17:34:32 archyan syncthing[4265]: [JRGFW] INFO: QUIC listener ([::]:22000) starting
déc. 08 17:34:32 archyan syncthing[4265]: [JRGFW] INFO: GUI and API listening on 127.0.0.1:8384
déc. 08 17:34:32 archyan syncthing[4265]: [JRGFW] INFO: Access the GUI via the following URL: http://127.0.0.1:8384/
déc. 08 17:34:32 archyan syncthing[4265]: [JRGFW] INFO: My name is "archyan"
déc. 08 17:34:32 archyan syncthing[4265]: [JRGFW] INFO: Ready to synchronize "Default Folder" (default) (sendreceive)
déc. 08 17:34:32 archyan syncthing[4265]: [JRGFW] INFO: Completed initial scan of sendreceive folder "Default Folder" (default)
lines 1-22/22 (END)
```

Note : Si un compte de service a été créé explicitement pour syncthing (e.g. via `useradd -r`) alors assurez-vous que l'utilisateur a un répertoire personnel valide sinon le service échouera immédiatement. Syncthing tente de mettre les fichiers de configuration dans `$HOME/.config/syncthing`
{: .prompt-info }

#### B-Service systemd utilisateur

Exécuter Syncthing comme un service utilisateur systemd assure que Syncthing ne démarre qu'après que l'utilisateur se soit connecté au système (par exemple, via l'écran de connexion graphique, ou ssh). Ainsi, le service utilisateur est destiné à être utilisé sur un ordinateur de bureau (multi-utilisateurs). 

Pour utiliser le service utilisateur, démarrez/activez l'unité utilisateur `syncthing.service` (avec le drapeau `--user`).

Le service utilisateur systemd Syncthing peut être démarré après le montage d'un périphérique spécifique (éventuellement crypté) et arrêté lorsque le périphérique a été démonté.  

Pour créer un service utilisateur dépendant d'un point de montage, après que le périphérique ait été monté, le nom de montage de systemd doit être identifié en exécutant la commande

    systemctl list-units -t mount

Ensuite, créez un nouveau service similaire à celui ci-dessous : 

    /home/$USER/.config/systemd/user/syncthing.service

```
[Unit]
Description=Syncthing
BindsTo=run-media-user-and-hash.mount

[Service]
ExecStart=/usr/bin/syncthing

[Install]
WantedBy=run-media-user-and-hash.mount
```

Le service ci-dessus doit être activé 

    systemctl enable --user syncthing.service

Il est également possible d'exécuter le service **systemd-user** au démarrage (c'est-à-dire sans se connecter) en utilisant le démarrage automatique des instances utilisateurs systemd 

    sudo loginctl enable-linger $USER

Un redémarrage de la machine est nécessaire...

