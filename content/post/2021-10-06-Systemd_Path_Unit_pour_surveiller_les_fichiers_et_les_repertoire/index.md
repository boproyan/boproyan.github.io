+++
title = 'Systemd Path Unit pour surveiller les fichiers et les répertoires'
date = 2021-10-06 00:00:00 +0100
categories = ['systemd']
+++
## systemd.path

*Utilisation des unités de chemin systemd (Systemd Path Unit) pour surveiller les fichiers et les répertoires  
Les unités de chemin Systemd vous permettent de surveiller les fichiers et répertoires (chemins) pour un événement. Une fois l'événement déclenché, systemd peut exécuter un script via une unité de service.[Using systemd Path Units to Monitor Files and Directories](https://www.putorius.net/systemd-path-units.html)*

### Création d'une unité de chemin Systemd

Nous allons créer une unité de chemin pour synchroniser le dossier local **/srv/data/jekyll/jekyll-TeXt-theme/_site/** du serveur xoyize.xyz(srvxo)  avec le dossier distant **/var/www/webapp_yann/yann.cinay.eu_/** du serveur cinay.eu  
Afin de créer une unité de chemin fonctionnel, nous avons besoin de trois fichiers.

*    Un script pour la synchronisation avec `rsync`
*    Unité de service pour lancer le script lorsqu'un événement est observé
*    Unité de chemin pour surveiller le fichier ou le répertoire

### Créer un script pour la synchronisation

Script **synchro_site.sh** pour synchroniser le dossier local **/srv/data/jekyll/jekyll-TeXt-theme/_site/** du serveur xoyize.xyz(srvxo)  avec le dossier distant **/var/www/webapp_yann/yann.cinay.eu_/** du serveur cinay.eu

    nano /srv/data/jekyll/jekyll-TeXt-theme/synchro_site.sh

```bash
#!/bin/bash
rsync -avz --progress --stats --human-readable --delete --rsync-path="sudo rsync" -e "ssh -p 55034 -i /home/xoyi/.ssh/kvm-vps506197 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"  /srv/data/jekyll/jekyll-TeXt-theme/_site/* debian@cinay.eu:/var/www/webapp_yann/yann.cinay.eu_/ ; if [ $? -eq 0 ]; then echo "Synchronisation dossier _site/ xoyize.xyz -> cinay.eu OK   " | systemd-cat -t rsync_site -p info ; else echo "Synchronisation dossier _site/ xoyize.xyz -> cinay.eu ERREUR" | systemd-cat -t rsync_site -p emerg ; fi
# Envoi message
dt=$(date '+%H:%M:%S'); echo '' | mail -a "Content-Type: text/plain; charset=UTF-8" -s "Fin Synchronisation $dt" yak@xoyize.xyz
```

NOTE> Le tag `rsync_site` est utilisé lors de l'analyse des journaux

Rendre le script exécutable  

    chmod +x /srv/data/jekyll/jekyll-TeXt-theme/synchro_site.sh

Créer un fichier d'unité de service pour exécuter le script

L'unité de service sera responsable de l'exécution du script lorsque l'unité de chemin observe les modifications dans le dossier local sur le fichier feed.xml

    sudo nano /etc/systemd/system/synchro_site.service

```ini
[Unit] 
Description="Rsync local distant"
After=network.target

[Service]
ExecStart=/srv/data/jekyll/jekyll-TeXt-theme/synchro_site.sh
```

Il existe de nombreuses autres options pour une unité de service qui n'entrent pas dans le cadre de ce didacticiel.
Créer un fichier d'unité de chemin

Le dernier fichier nécessaire pour terminer notre configuration est le fichier d'unité de chemin lui-même. Le fichier d'unité doit être créé dans le même répertoire que son fichier de service correspondant, /etc/systemd/systemdans ce cas. Il doit également avoir le même nom, mais avec une extension .path.

    sudo nano /etc/systemd/system/synchro_site.path

```ini
[Unit]
Description=Modification dossier local _site

[Path]
PathModified=/srv/data/jekyll/jekyll-TeXt-theme/_site/feed.xml
Unit=synchro_site.service

[Install]
WantedBy=multi-user.target
```

Anatomie d'une unité de pathologie

Un systemd path unit prend l'extension .path, et il surveille un fichier ou un répertoire. Une unité .path appelle une autre unité (généralement une unité .service avec le même nom) lorsque quelque chose arrive au fichier ou au répertoire surveillé. Par exemple, si vous avez une unité .path picchanged pour surveiller l'instantané de votre webcam, vous aurez également une unité .service picchanged qui exécutera un script lorsque l'instantané sera écrasé.

Les unités de chemin contiennent une nouvelle section, [Path], avec quelques directives supplémentaires. Tout d'abord, vous avez les directives "what-to-watch-for" :

*    `PathExists=` vérifie si le fichier ou le répertoire existe. Si c'est le cas, l'unité associée est déclenchée.  
`PathExistsGlob=` fonctionne de manière similaire, mais vous permet d'utiliser le globbing, comme lorsque vous utilisez ls *.jpg pour rechercher toutes les images JPEG dans un répertoire. Cela vous permet de vérifier, par exemple, si un fichier avec une certaine extension existe.
*    `PathChanged=` surveille un fichier ou un répertoire et active l'unité configurée chaque fois qu'elle change. Il n'est pas activé à chaque écriture dans le fichier surveillé, mais seulement lorsqu'un fichier surveillé ouvert en écriture est modifié puis fermé. L'unité associée est exécutée lorsque le fichier est fermé.
*    `PathModified=`, en revanche, active l'unité lorsque quelque chose est modifié dans le fichier surveillé, avant même que vous ne fermiez le fichier.
*    `DirectoryNotEmpty=` fait ce qui est indiqué sur la boîte, c'est-à-dire qu'il active l'unité associée si le répertoire surveillé contient des fichiers ou des sous-répertoires.
* `Unit=` qui indique au `.path` quelle unité de service doit être activée, au cas où vous voudriez lui donner un nom différent de celui de votre unité `.path`
* `MakeDirectory=` peut être vrai ou faux (ou 0 ou 1, ou oui ou non) et crée le répertoire que vous voulez surveiller avant que la surveillance ne commence. Il est évident que l'utilisation de `MakeDirectory=` en combinaison avec `PathExists=` n'a pas de sens.  
Cependant, `MakeDirectory=` peut être utilisé en combinaison avec `DirectoryMode=`, que vous utilisez pour définir le mode (autorisations) du nouveau répertoire. Si vous n'utilisez pas `DirectoryMode=`, les permissions par défaut du nouveau répertoire sont `0755` 

### Vérifier la syntaxe des fichiers Unit

Vous pouvez utiliser la commande `systemd-analye` avec l'option verify pour vérifier l'exactitude de vos fichiers d'unité. Cela aidera à signaler les erreurs de syntaxe.

    sudo systemd-analyze verify /etc/systemd/system/synchro_site.*

Si vous rencontrez des erreurs fatales, elles seront surlignées en rouge.

### Démarrer unité de chemin systemd

les changements dans le dossier system

    sudo systemctl daemon-reload

Le démarrer avec systemctl, tout comme un service normal.

    sudo systemctl start synchro_site.path

Vérifier l'état 

    sudo systemctl status synchro_site.path

```
● synchro_site.path - Modification dossier local _site
   Loaded: loaded (/etc/systemd/system/synchro_site.path; disabled; vendor preset: enabled)
   Active: active (waiting) since Tue 2020-08-04 07:03:37 CEST; 10s ago

août 04 07:03:37 xoyize.xyz systemd[1]: Started Modification dossier local _site.
```

### Activer unit de chemin systemd au démarrage

L'activation de votre unité de chemin pour démarrer au démarrage se fait de la même manière que vous activez les services au démarrage , comme ceci:

    sudo systemctl enable synchro_site.path

### Test unité de chemin

Maintenant, nous avons notre nouvelle unité de chemin en cours d'exécution qui surveille le dossier local.   
Si des modifications sont apportées dans le dossier suite à une génération jekyll.   
L'unité de chemin détectera la modification du dossier, qui à son tour déclenchera l'unité de service.

Vérification des journaux de votre unité de chemin avec Journalctl

L'un des avantages de l'utilisation des unités de chemin (ou de minuterie) systemd est qu'elles fonctionnent comme des services normaux. Cela signifie que vous pouvez les contrôler avec systemctl et leurs journaux sont disponibles via `journalctl`.

Pour vérifier les journaux générés par votre unité de chemin 

    sudo journalctl -t rsync_site

```
-- Logs begin at Sun 2020-08-02 17:14:50 CEST. --
août 04 08:10:06 xoyize.xyz rsync_site[28585]: Synchronisation dossier _site/ xoyize.xyz -> cinay.eu OK
août 04 08:10:23 xoyize.xyz rsync_site[28598]: Synchronisation dossier _site/ xoyize.xyz -> cinay.eu OK
août 04 08:15:24 xoyize.xyz rsync_site[28674]: Synchronisation dossier _site/ xoyize.xyz -> cinay.eu OK
```

