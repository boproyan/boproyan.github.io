+++
title = 'Créer un service "Systemd Utilisateur"'
date = 2023-01-21 00:00:00 +0100
categories = ['systemd']
+++
*En plus de l'instance à l'échelle du système, systemd fournit des instances spécifiques à l'utilisateur qui permettent aux utilisateurs d'exécuter des services ou des applications en tant qu'eux-mêmes. Systemd prend en charge différents types d'unités, l'unité de service étant la plus courante, représentant un service ou un démon. Systemd est livré avec une variété de nouvelles approches, qui ciblent en particulier l'observabilité et le suivi du processus démarré. Les sorties de journal peuvent être envoyées à STDOUT/STDERR, qui seront récupérées par systemd et transmises au "journal".*

## Systemd Utilisateur

### Statut de l'utilisateur Systemd

La commande `systemctl --user status` affiche tous les processus gérés par systemd pour l'utilisateur actuel.
L'exemple suivant montre l'instance utilisateur systemd (init.scope) et un serveur memcached (mymemcached.service).

```shell
 systemctl --user status
● nine01-test
    State: running
     Jobs: 0 queued
   Failed: 0 units
    Since: Fri 2021-09-17 17:12:02 CEST; 4 months 10 days ago
   CGroup: /user.slice/user-33.slice/user@33.service
           ├─init.scope
           │ ├─1634 /lib/systemd/systemd --user
           │ └─1647 (sd-pam)
           └─mymemcached.service
             └─106393 /usr/bin/memcached -A -m 32 -p 11212 -u www-data -l 127.0.0.1 -P /home/www-data/memcached.pid
```

Si vous souhaitez utiliser la commande `systemctl` dans votre crontab ou dans des scripts, vous devrez définir la variable d'environnement **XDG_RUNTIME_DIR** :

    export XDG_RUNTIME_DIR=/run/user/$UID

### Statut d'un service spécifique

Vous pouvez obtenir plus d'informations sur l'état d'exécution d'un service particulier en ajoutant le nom du service à la commande status. L'état de l'unité, une liste des traitements et les dernières lignes du journal sont affichés :

```shell
 systemctl --user status mymemcached.service
● mymemcached.service - Custom memcached on port 11212
   Loaded: loaded (/home/www-data/.config/systemd/user/mymemcached.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2022-01-26 13:27:43 CET; 24h ago
     Docs: https://support.nine.ch/a/Ws7sDLPoiTo
 Main PID: 106393 (memcached)
   CGroup: /user.slice/user-33.slice/user@33.service/mymemcached.service
           └─106393 /usr/bin/memcached -A -m 32 -p 11212 -u www-data -l 127.0.0.1 -P /home/www-data/memcached.pid

Jan 26 13:27:43 nine01-test systemd[1634]: Started Custom memcached on port 11212.
```

### Afficher le journal d'un service

`journalctl` est utilisé pour afficher les journaux du journal. Le journal peut être filtré pour un service spécifique à l'aide du paramètre `--user-unit` :

    journalctl --user-unit=mymemcached.service

    Jan 26 13:27:43 nine01-test systemd[1634]: Started Custom memcached on port 11212.

### Activer un service

Pour s'assurer qu'un service est démarré lors du démarrage du système, il doit être ajouté au fichier  `default.target`. Notez que  `multi-user.target` car il n'est pas disponible dans l'instance utilisateur de systemd.

```shell
 systemctl --user enable mymemcached.service 
Created symlink from /home/www-data/.config/systemd/user/default.target.wants/mymemcached.service to /home/www-data/.config/systemd/user/mymemcached.service.
```

La commande systemctl --user is-enabledpeut être utilisée pour vérifier si un service a déjà été activé.

     systemctl --user is-enabled mymemcached.service 
    enabled

### Désactiver un service

     systemctl --user disable mymemcached.service 
    Removed symlink /home/www-data/.config/systemd/user/default.target.wants/mymemcached.service.

### Fichiers Unit

Les fichiers Unit de l'instance utilisateur systemd sont stockés dans le répertoire  `~/.config/systemd/user`. Le nom du fichier définit le nom et le type de l'unité.  
Par exemple, le fichier `~/.config/systemd/user/mymemcached.service` définit une unité de service nommée "mymemcached".

La description de l'unité se compose de trois parties.

La section Unité contient des métadonnées sur l'unité.  
La deuxième section porte le nom du type d'unité, où Service est la plus courante.  
Cette section définit l'unité.  
La section Installer contient des instructions sur la façon d'activer/désactiver une unité.  

#### Simple Service

Cet exemple démarre un serveur memcached sur le port 11212.  

La section **Unit** définit uniquement la description et une URL (facultative) à des fins de documentation.  
La section **Service** définit la commande et les paramètres de démarrage et d'arrêt.  
La commande d'arrêt est facultative.  
Si systemd arrête un service, il exécutera d'abord la commande d'arrêt, puis tuera tous les processus restants de cette unité.

    ~/.config/systemd/user/mymemcached.service


```shell
[Unit]
Description=Custom memcached on port 11212
Documentation=https://support.nine.ch/a/Ws7sDLPoiTo

[Service]
ExecStart=/usr/bin/memcached -A -m 32 -p 11212 -u www-data -l 127.0.0.1 -P /home/www-data/memcached.pid

[Install]
WantedBy=default.target
```


#### Forking Service

Systemd déconseille les services qui se démonisent. Si votre application doit être démonisée, vous devez en tenir compte dans la configuration de l'unité en définissant le service **Type** sur **forking** dans la section **Service** . Dans ce cadre, il est recommandé de définir l'option **PIDFile** permettant d'indiquer à systemd quel processus est le processus principal.

Pour plus de commodité, les spécificateurs peuvent être utilisés pour créer des fichiers d'unité plus robustes et portables. Par exemple, `%h` est remplacé par le répertoire personnel de l'utilisateur et `%p` par le nom de l'unité. D'autres spécificateurs sont documentés dans la page de manuel `systemd.unit` 

    ~/.config/systemd/user/mymemcached.service

```shell
[Unit]
Description=Custom memcached on port 11212
Documentation=https://support.nine.ch/a/Ws7sDLPoiTo

[Service]
Type=forking
ExecStart=/usr/bin/memcached -A -m 32 -p 11212 -u www-data -l 127.0.0.1 -P %h/.%p.pid

[Install]
WantedBy=default.target
```

#### Template Service

Les modèles sont un outil puissant de systemd.  
Les unités dont le nom se termine par un `@` sont considérées comme une unité modèle. La configuration d'un modèle vous permet de lancer plusieurs instances similaires du même service avec différents ports ou sockets. La partie qui suit `@` est alors factorisée dans le spécificateur `%i` pour être utilisée dans le fichier unit.

Si vous collez l'exemple suivant dans un fichier nommé `mymemcached@.service`, vous pouvez en lancer plusieurs instances.

    .config/systemd/user/mymemcached@.service

```shell
[Unit]
Description=Custom memcached on port %i
Documentation=https://support.nine.ch/a/Ws7sDLPoiTo

[Service]
ExecStart=/usr/bin/memcached -A -m 32 -p %i -u www-data -l 127.0.0.1 -P %h/%p-%i.pid

[Install]
WantedBy=default.target
```

Par exemple, `mymemcached@7777` et `mymemcached@7778`d émarrerait un serveur memcached sur les ports 7777 et 7778 respectivement.

```shell
systemctl --user enable mymemcached@7778.service
Created symlink /home/www-data/.config/systemd/user/default.target.wants/mymemcached@7778.service → /home/www-data/.config/systemd/user/mymemcached@.service.

www-data@nine01-test:~ $ systemctl --user start mymemcached@7778.service

www-data@nine01-test:~ $ systemctl --user status mymemcached@7778.service
● mymemcached@7778.service - Custom memcached on port 7778
   Loaded: loaded (/home/www-data/.config/systemd/user/mymemcached@.service; indirect; vendor preset: enabled)
   Active: active (running) since Thu 2022-01-27 08:55:58 CET; 7s ago
     Docs: https://support.nine.ch/a/Ws7sDLPoiTo
 Main PID: 108919 (memcached)
   CGroup: /user.slice/user-33.slice/user@33.service/mymemcached.slice/mymemcached@7778.service
           └─108919 /usr/bin/memcached -A -m 32 -p 7778 -u www-data -l 127.0.0.1 -P /home/www-data/memcached-7778.pid

Jan 27 08:55:58 nine01-test systemd[1634]: Started Custom memcached on port 7778.
```

### Extraits de configuration avancés

#### Tâches avant et après le démarrage

Si votre service dépend de certaines étapes supplémentaires lors de la routine de démarrage ou d'arrêt, vous pouvez utiliser les options `ExecStart` et `ExecStop` en combinaison avec les suffixes `Pre` ou `Post`. Le paramètre peut être répété plusieurs fois pour plusieurs étapes et actions à exécuter.

En préfixant votre commande avec un `-`, vous pouvez indiquer à systemd d'ignorer les erreurs de cette commande. Systemd interrompra le processus de démarrage si vous ne spécifiez pas cet indicateur et qu'une erreur se produit au démarrage.

```shell
[Service]
ExecStartPre=/bin/mkdir -p /tmp/mytempdir
ExecStartPre=-/bin/false
ExecStartPost=/usr/bin/touch /tmp/mytempdir/started
ExecStopPost=/bin/rm -rf /tmp/mytempdir
```

#### Recharger

Si votre service est capable de recharger sa configuration, vous pouvez utiliser le paramètre `ExecReload` pour spécifier comment déclencher le rechargement.

Pour recharger le service, systemd exécute la commande donnée. La variable `$MAINPID` est définie sur l'ID de processus principal du service.

```shell
[Service]
ExecReload=/bin/kill -HUP $MAINPID
```

#### Redémarrage

Avec le paramètre **Restart** , vous pouvez spécifier si systemd doit redémarrer l'unité si elle se ferme de manière inattendue. La valeur par défaut pour Redémarrer est `no`. Le définir sur `always` fera que systemd redémarrera automatiquement le processus.

```shell
[Service]
Restart=always
```

Si vous ne surveillez pas le service d'une autre manière, il est fortement recommandé de définir `Restart=always`. Bien que cela n'empêche aucune interruption, l'impact sera moindre.

#### Variables d'environnement

Outre les fichiers de configuration, les applications peuvent également être configurées via des variables d'environnement. Systemd vous permet de définir un répertoire de travail ( `WorkingDirectory`) ainsi que des variables d'environnement ( `Environment`) dans la configuration de l'unité.

De plus, vous pouvez spécifier un fichier qui contient plus de variables d'environnement ( `EnvironmentFile`), pour faire partie de vos déploiements. Si vous souhaitez utiliser cette option, le fichier doit exister pour que systemd puisse démarrer le service. Préfixer la déclaration avec un `-` obligera systemd à ignorer l'option si le fichier n'existe pas. Les entrées `EnvironmentFile` doivent être dans un format `KEY=VALUE`.

```shell
[Service]
WorkingDirectory=/home/www-data/servicedir
Environment=ENV=production
EnvironmentFile=-/home/www-data/servicedir/.env
```

Liens

Vous trouverez plus d'informations et d'options sur les unités dans la page de manuel [systemd.unit](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)  
Pour les unités de service, les pages de manuel [systemd.service](https://www.freedesktop.org/software/systemd/man/systemd.service.html) et [systemd.exec](https://www.freedesktop.org/software/systemd/man/systemd.exec.html) constituent un bon point de départ.

### Services utilisateurs au démarrage

La configuration par défaut de systemd-logind est conçue afin de s'assurer que les processus lancés par un utilisateur ne lui survivent pas. Du coup, cette instance est lancée lors du login, et coupée lors du log out *de toutes les sessions de l'utilisateur* (une seule instance est lancée même si vous vous connectez 5 fois à la machine).

Evidemment, ce n'est pas ce que l'on recherche ici, mais heureusement il est possible de dire à systemd que l'instance d'un utilisateur soit tout le temps présente, du boot au shutdown, ce qui permet d'avoir des processus (services) utilisateurs qui tournent sans avoir de session ouverte.

Il suffit de faire ceci (en root):

```
sudo loginctl enable-linger $USER
```

Redémarrer la machine...

## Exemple - Lancer VirtualBox dans tmux en ligne de commande

Ligne de commande

```bash
/usr/bin/tmux new-session -s virtuel -d /usr/bin/VBoxManage startvm 'Debian Stretch' --type headless
```

La commande `new-session` permet de créer une nouvelle session dédiée, dont le nom est `virtuel`. Le paramètre `-d` permet de détacher immédiatement la session tmux, et ensuite suit la commande à exécuter. 

### une instance "utilisateur" de systemd

Si tout le monde connaît le systemd qui tourne avec le PID 1, au niveau du système, on connait moins le systemd qui tourne avec un PID quelconque, et qui gère les services liés à un utilisateur donné.

[pam-systemd](https://www.freedesktop.org/software/systemd/man/pam_systemd.html) et [systemd-logind](https://www.freedesktop.org/software/systemd/man/systemd-logind.service.html) permettent à systemd d'instancier des *slices* pour les différents utilisateurs et des *scopes* pour les différentes sessions ([voir systemd user ,wiki Archlinux](https://wiki.archlinux.org/index.php/Systemd/User)).

Si vous exécutez `systemctl status`, vous pourrez retrouver ce type d'info dans l'arborescence:

```
   ├─user.slice
   │ └─user-1000.slice
   │   ├─user@1000.service
   │   │ ├─tmux-irc.service
   │   │ │ ├─5138 /usr/bin/tmux new-session -d -s irc /usr/bin/irssi
   │   │ │ └─5139 /usr/bin/irssi
   │   │ └─init.scope
   │   │   ├─18920 /lib/systemd/systemd --user
   │   │   └─18921 (sd-pam)

```

C'est relativement anodin au jour le jour (le plus souvent ça permet de s'assurer que rien ne subsiste une fois qu'un utilisateur se délogue), mais via ce mécanisme, systemd permet aussi à certains utilisateurs de fournir leurs propres *units*. C'est cela que nous allons utiliser pour notre session tmux.


### Créer une "unit" utilisateur pour le service

Il est possible de placer les *units* propres à un utilisateur dans le dossier `~/.config/systemd/user/`.  
Créer le dossier

    mkdir -p ~/.config/systemd/user

Le fichier unit `tmux-virtuel.service`:

    nano ~/.config/systemd/user/tmux-virtuel.service

```ini
[Unit]
Description=tmux instance with virtual running in it.

[Service]
ExecStart=/usr/bin/tmux new-session -d -s virtuel /usr/bin/VBoxManage startvm 'Debian Stretch' --type headless

Type=forking
Restart=always

[Install]
WantedBy=default.target
```

Une fois ce fichier placé au bon endroit, il suffit de reloader `systemd` puis on peut démarrer le service:

```
$ systemctl --user daemon-reload
$ systemctl start --user tmux-virtuel
$ systemctl status --user tmux-virtuel
```

Par contre si vous rebootez la machine, ça ne marche pas. En fait, l'instance utilisateur de systemd n'est créée que lorsqu'un utilisateur se logue. Donc, pour le coté "présence constante", ça n'est pas suffisant.

**Note:** Théoriquement, il me semble tmux devrait également être arrêté lors du log-out mais ce n'est pas le cas, en tout cas sous Debian.

