+++
title = 'lsyncd rsync inotify ,synchronisation dossier temps réel entre plusieurs serveurs'
date = 2020-08-09 00:00:00 +0100
categories = ['inotify']
+++
## Lsyncd

* [How to setup lsyncd over SSH](https://www.keycdn.com/support/how-to-setup-lsyncd-over-ssh/)
* [How To Mirror Local and Remote Directories on a VPS with lsyncd](https://www.digitalocean.com/community/tutorials/how-to-mirror-local-and-remote-directories-on-a-vps-with-lsyncd)
* [Installation et configuration de lsyncd sur Debian](https://mnt-tech.fr/blog/installation-et-configuration-de-lsyncd-sur-debian/)

Lsyncd est un moyen de synchroniser automatiquement un répertoire local avec d'autres machines. Les fichiers sur la machine locale sont surveillés pour les changements toutes les quelques secondes et si des changements sont notés, ils sont alors répliqués et synchronisés sur le(s) serveur(s) distant(s). Cette solution de synchronisation est facile à installer, légère et ne limite pas les performances du système de fichiers local.

Par défaut, Lsync utilise Rsync pour répliquer les fichiers de la machine locale et ne transférera que les fichiers qui ont été modifiés. Il existe d'autres solutions sur le marché qui exécutent cette tâche, mais il s'agit d'une solution peu coûteuse et pratique pour synchroniser automatiquement les données, minimisant ainsi la perte de données et le temps passé à répliquer les fichiers d'une machine à l'autre.

Les 5 fonctions suivantes sont proposées :

* Vérifie ce qui doit être synchronisé de votre machine locale vers votre ou vos serveurs distants.
* Contrôles sur le vice versa
* Synchronisation de votre local vers votre distant
* Syncs vice-versa
* Peut éditer votre fichier "rsync includes".

L'utilisation de cette méthode vous permet de synchroniser en continu les données de votre machine locale vers la machine distante sans ajouter de stress supplémentaire sur le serveur. 

### Installation lsyncd

L'installation de Lsyncd sur votre serveur est un processus assez simple et facile. Utilisez les commandes suivantes pour l'installer sur votre système Debian ou Archlinux.

    sudo apt install lsyncd

Archlinux

    yaourt -S lsyncd

*L'exécution de ces commandes installera Lsyncd et inclura quelques exemples dans le répertoire **/usr/share/doc/lsyncd/examples**.Ces exemples peuvent être utilisés pour avoir une meilleure idée des configurations possibles lors de l'utilisation de cette méthode de synchronisation.*

Arrêt temporaire du service, pour la configuration 

    sudo systemctl stop lsyncd

### Mise en place de l'environnement

Répertoire local : **/srv/wikistatic/_site**  
La connexion SSH sur la machine distante se fait avec l'utilisateur **backupuser** et un jeu de clé **id_rsa**  

#### machine distante

Création répertoire avec les droits utilisateur 

    sudo mkdir /srv/dest
    sudo chown backupuser.backupuser -R /srv/dest

#### machine locale

Créer un répertoire de logs et quelques fichiers que **lsyncd** à utilisera

    sudo mkdir /var/log/lsyncd
    sudo touch /var/log/lsyncd/lsyncd.log
    sudo touch /var/log/lsyncd/lsyncd-status.log

Ensuite, nous pouvons créer le répertoire de configuration lsyncd 

    sudo mkdir /etc/lsyncd

Créer un fichier de configuration à l'intérieur de ce répertoire appelé "lsyncd.conf.lua"

    sudo nano /etc/lsyncd/lsyncd.conf.lua

```
settings {
    logfile = "/var/log/lsyncd/lsyncd.log",
    statusFile = "/var/log/lsyncd/lsyncd-status.log",
    statusInterval = 20
}
sync {
    default.rsync,
    source="/srv/wikistatic/_site",
    target="backupuser@193.70.43.101:/srv/dest",
    rsync = {
        archive = false,
        acls = false,
        compress = true,
        links = false,
        owner = false,
        perms = false,
        verbose = true,
        rsh = "/usr/bin/ssh -p 55027 -i /home/backupuser/.ssh/id_rsa -o StrictHostKeyChecking=no"
    }
}
```

### Utiliser lsyncd

Lancer manuellement

    lsyncd /etc/lsyncd/lsyncd.conf.lua

Lancer le service

    sudo systemctl start lsyncd

Statut

    sudo systemctl status lsyncd

```
● lsyncd.service - LSB: lsyncd daemon init script
   Loaded: loaded (/etc/init.d/lsyncd)
   Active: active (running) since mer. 2018-04-25 21:14:51 CEST; 27s ago
  Process: 9877 ExecStop=/etc/init.d/lsyncd stop (code=exited, status=0/SUCCESS)
  Process: 15464 ExecStart=/etc/init.d/lsyncd start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/lsyncd.service
           ├─15468 /usr/bin/lsyncd -pidfile /var/run/lsyncd.pid /etc/lsyncd/lsyncd.conf.lua
           ├─15469 /usr/bin/rsync --delete --ignore-errors -vzst --rsh=/usr/bin/ssh -p 55027 -i /home/b...
           └─15470 /usr/bin/ssh -p 55027 -i /home/backupuser/.ssh/id_rsa -o StrictHostKeyChecking=no -l...

avril 25 21:14:51 yanspm.com lsyncd[15464]: Starting synchronization daemon: lsyncd.
avril 25 21:14:51 yanspm.com systemd[1]: Started LSB: lsyncd daemon init script.
```

Vérifier l'opération 

    cat /var/log/lsyncd/lsyncd.log

```
[...]
sent 15,851,575 bytes  received 10,820 bytes  118,819.44 bytes/sec
total size is 21,886,154  speedup is 1.38
Wed Apr 25 21:17:04 2018 Normal: Startup of "/srv/wikistatic/_site/" finished.
```

### Lsyncd rotation des logs

Rotation des logs chaque semaine ,on conserve les 4 dernières rotations

    sudo nano /etc/logrotate.d/lsyncd

```
/var/log/lsyncd/*.log {
        weekly
        rotate 4
        delaycompress
        compress
        notifempty
        missingok
}
```

### Configuration d'une synchronisation locale

Donc, pour tester, essayons une synchronisation locale avec lsyncd.
Commençons par créer un «dossier source» contenant les fichiers à synchroniser.

    $ mkdir -p $HOME/unixmen/sync_source

Ensuite, nous allons créer un dossier de sauvegarde:

    $ mkdir $HOME/sync_backup

Juste pour le test, remplissez le répertoire source avec des fichiers nuls, en utilisant la commande tactile :

    touch $HOME/unixmen/sync_source/unixmentest{1..10}0

Maintenant, créez des fichiers de log et d’état:

```
# mkdir /var/log/lsyncd
# touch /var/log/lsyncd.{log,status}
```

Le fichier de configuration continue /etc/lsyncd :

    # mkdir /etc/lsyncd

Ici, nous allons créer un nouveau fichier de configuration:

    # nano /etc/lsyncd/lsyncd.conf.lua

Où nous mettrons le code suivant:

```
settings = {
logfile = "/var/log/lsyncd/lsyncd.log",
statusFile = "/var/log/lsyncd/lsyncd.status"
}

sync {
default.rsync,
source = "$HOME/unixmen/sync_source/",
target = "$HOME/sync_backup",
}
```

En ce moment, si vous regardez dans le répertoire de sauvegarde, vous le trouverez vide, car le service n'est pas en cours d'exécution.
Mais, pour résoudre ce problème, redémarrez simplement lsyncd avec les éléments suivants:

    # systemctl restart lsyncd

Il va commencer (et continuer) la synchronisation, même si vous ajoutez plus de fichiers sur sync_source. Cela fonctionne complètement en mode automatisé.


## Linux inotify rsync

* [Article original : Sync folders and files on Linux with rsync and inotify](https://bartsimons.me/sync-folders-and-files-on-linux-with-rsync-and-inotify/)  
* [Utilisation de inotifywait dans des scripts Shell](http://kerlinux.org/2010/08/utilisation-de-inotifywait-dans-des-scripts-shell/)
* [Synchroniser deux serveurs en temps réel avec inotify et rsync](http://www.casper-development.be/synchroniser-deux-serveur-en-temps-reel-avec-inotify-et-rsync/)
* [Unison (doc ubuntu)](https://doc.ubuntu-fr.org/unison)
* [Synchronisation de fichiers bidirectionnelle avec Unison](http://www.responsive-mind.fr/unison-synchronisation-fichiers/)

Vous avez deux clients et/ou serveurs ou plus. Ils contiennent des fichiers que vous voulez synchroniser automatiquement.  
Rsync + inotify (surveillance en temps réel de votre système de fichiers) sont la solution afin que vos fichiers puissent être synchronisés entre plusieurs machines.

### Prérequis sur les machines

* OpenSSH server
* Rsync
* Inotify

Installer toutes les dépendances sur les serveurs

    apt update && apt -y install openssh-server rsync inotify-tools # debian
    pacman -S openssh rsync inotify-tools # archlinux

### inotifywait

Aide

    inotifywait --help

```bash
inotifywait 3.20.2.1
Attendre un événement particulier sur un fichier ou un ensemble de fichiers.
Utilisation: inotifywait [ options ] file1 [ file2 ] [ file3 ] [ ... ]
Options:
	-h|--help     	Affiche ce texte d'aide.
	@<file>       	Exclude the specified file from being watched.
	--exclude <pattern>
	              	Exclure tous les événements sur les fichiers correspondant à 
	              	l'expression régulière étendue <modèle>.
	              	Seule la dernière option --exclude sera
	              	pris en considération.
	--excludei <pattern>
	              	Comme --exclure mais insensible à la casse.
	--include <pattern>
	              	Exclure tous les événements figurant dans les dossiers, sauf ceux
	              	correspondant à l'expression régulière étendue
	              	<pattern>.
	--includei <pattern>
	              	Comme --inclure mais sans tenir compte de la casse.
	-m|--monitor  	ne pas stopper l'exécution au premier évènement, poursuivre l'écoute ou jusqu'à ce que le --timeout expire.
	              	Sans cette option, inotifywait s'arrêtera après la réception d'un événement.
	-d|--daemon   	Identique à --monitor, sauf qu'il est exécuté en arrière-plan
	              	en enregistrant les événements dans un fichier spécifié par --outfile.
	              	Implique --syslog
	-r|--recursive	observer également (récursivement) les sous-répertoires
	--fromfile <file>
	              	Lire les fichiers à surveiller à partir de <file> ou `-' pour stdin.
	-o|--outfile <file>
	              	Imprimez les événements sur <file> plutôt que sur stdout.
	-s|--syslog   	Envoyer les erreurs à syslog plutôt qu'à stderr.
	-q|--quiet    	Imprimez moins (n'imprimez que les événements).
	-qq           	N'imprime rien (pas même les événements).
	--format <fmt>	Imprimer en utilisant un format spécifié de type printf
	              	chaîne de caractères (valeur par défaut : %w %,e %f) 
                        %f : le fichier a l'origine de l'évènement
                        %e (équivalent à %,e) : liste des évènements interceptés
                        %Xe : même chose que e, avec X au lieu d'une virgule, pour caractère de séparation entre chaque type d'évènements simultanés
                        %w : le fichier sur lequel a été mis en place l'observation
                        %T : les date et/ou heure de l'évènement (implique l'option --timefmt)
	--timefmt <fmt> chaîne de format compatible strftime à utiliser avec
	              	%T dans la chaîne de --format.
	-c|--csv      	Imprimer les événements au format CSV.
	-t|--timeout <seconds>
	              	Lors de l'écoute d'un seul événement, le temps mort après
	              	attendre un événement pendant <secondes> secondes.
	              	Si <secondes> est négatif, inotifywait ne s'arrêtera jamais.
	-e|--event <event1> [ -e|--event <event2> ... ]
		Ecoutez les événements spécifiques.  S'ils sont omis, tous les événements sont 
		écouté.

Statut de sortie:
	0 - Un événement que vous avez demandé à surveiller a été reçu.
	1 - Un événement que vous n'avez pas demandé à surveiller a été reçu
	      (généralement se supprimer ou se démonter), ou une erreur est survenue.
	2 - L'option --timeout a été donnée et aucun événement ne s'est produit
	      dans l'intervalle de temps spécifié.

Événements:
	access		le contenu du fichier d'accès ou du répertoire a été lu
	modify		le contenu d'un fichier ou d'un répertoire a été écrit
	attrib		attributs des fichiers ou des répertoires ont été modifiés
	close_write	fichier ou répertoire fermé, après avoir été ouvert en
	           	mode écriture
	close_nowrite	fermer le fichier ou le répertoire, quel que soit le mode de lecture/écriture
			    fichier ou répertoire ouvert
	close		fichier ou répertoire fermé, quel que soit le mode de lecture/écriture
	open		    fichier ou répertoire ouvert
	moved_to  	fichier ou répertoire déplacé vers le répertoire surveillé
	moved_from	fichier ou répertoire déplacé du répertoire surveillé
	move		    fichier ou répertoire déplacé vers ou depuis le répertoire surveillé
	move_self	Un fichier ou un répertoire surveillé a été déplacé.
	create		fichier ou répertoire créé dans le répertoire surveillé
	delete		fichier ou répertoire supprimé dans le répertoire surveillé
	delete_self	un fichier ou répertoire supprimé
	unmount		système de fichiers contenant un fichier ou un répertoire non monté
```

Test sur toutes modifications dans le dossier **$HOME/media/dplus/statique**

```bash
inotifywait -qrm --format "%w %:e %f %T" --timefmt "%T" $HOME/media/dplus/statique | while read file; do
    echo $file
done
```

Résulats

```bash
[yannick@yannick-pc ~]$ inotifywait -qrm --format "%w %:e %f %T" --timefmt "%T" $HOME/media/dplus/statique | while read file; do
>     echo $file
> done
/home/yannick/media/dplus/statique/_posts/ MODIFY 2018-04-30-sync-inotify-synchro-temps-reel-entre-serveurs.md 10:04:46
/home/yannick/media/dplus/statique/_posts/ OPEN 2018-04-30-sync-inotify-synchro-temps-reel-entre-serveurs.md 10:04:46
/home/yannick/media/dplus/statique/_posts/ MODIFY 2018-04-30-sync-inotify-synchro-temps-reel-entre-serveurs.md 10:04:46
/home/yannick/media/dplus/statique/_posts/ CLOSE_WRITE:CLOSE 2018-04-30-sync-inotify-synchro-temps-reel-entre-serveurs.md 10:04:46
/home/yannick/media/dplus/statique/_posts/ MODIFY 2020-08-08-PC1-ArchLinux-XFCE-ASUS-H110M-A.md 10:05:19
/home/yannick/media/dplus/statique/_posts/ OPEN 2020-08-08-PC1-ArchLinux-XFCE-ASUS-H110M-A.md 10:05:19
/home/yannick/media/dplus/statique/_posts/ MODIFY 2020-08-08-PC1-ArchLinux-XFCE-ASUS-H110M-A.md 10:05:19
/home/yannick/media/dplus/statique/_posts/ CLOSE_WRITE:CLOSE 2020-08-08-PC1-ArchLinux-XFCE-ASUS-H110M-A.md 10:0
```

Créons un dossier spécifique que nous voulons synchroniser 

    mkdir /opt/syncfiles

écrire un script qui passe par une boucle infinie avec inotifywait dedans :

```
#!/bin/bash

# Il est supposé s'exécuter sur rsync-host01, changer rsync-host02 en rsync-host01 pour créer un script qui est censé s'exécuter sur rsync-host02.

while true; do
  inotifywait -r -e modify,attrib,close_write,move,create,delete /opt/syncfiles
  rsync -avz -e "ssh -i /root/rsync-key -o StrictHostKeyChecking=no“  /opt/syncfiles/ root@rsync-host02:/opt/syncfiles/
done
```

>Si le port 22 n'est pas celui par défaut, ajouter l'option `-p N°Port` à la commande ssh

Enregistrer ce script dans le répertoire /opt sous le nom **file-sync.sh**



### Service systemd sync.service

Créer un fichier appelé **sync.service** qui peut s'arrêter, démarrer et réinitialiser le script à la demande ou sur des événements spécifiques comme un démarrage du système dans le répertoire */etc/systemd/systemd/system/system/* avec le contenu suivant 

```
[Unit]
Description = SyncService
After = network.target

[Service]
PIDFile = /run/syncservice/syncservice.pid
User = root
Group = root
WorkingDirectory = /opt
ExecStartPre = /bin/mkdir /run/syncservice
ExecStartPre = /bin/chown -R root:root /run/syncservice
ExecStart = /bin/bash /opt/file-sync.sh
ExecReload = /bin/kill -s HUP $MAINPID
ExecStop = /bin/kill -s TERM $MAINPID
ExecStopPost = /bin/rm -rf /run/syncservice
PrivateTmp = true

[Install]
WantedBy = multi-user.target
```

Droits du fichier de service et recharger le démon systemd

    chmod 755 /etc/systemd/system/sync.service  
    systemctl daemon-reload

Activer le service

    systemctl enable sync.service

Démarrer le service

    systemctl start sync.service

### Utilisation de clés pour SSH 

Deux alternatives, l'une (A) utilise SSH avec **ACCES root** et l'autre (B) utilise SSH avec **SANS ACCES root**

#### A-Utilisation de clés pour SSH avec ACCES root

Et pour un transfert de fichiers sécurisé, nous voulons un lien public-privé pour le lien de transfert que rsync utilise

    ssh-keygen -t rsa -f ~/rsync-key -N ''

Coller le résultat dans le fichier **~/.ssh/.ssh/authorized_keys** de vos serveurs de destination

    cat ~/rsync-key.pub

Retirer la clé publique pour des raisons de sécurité

    rm ~/rsync-key.pub

>Copier le contenu de la clé publique **rsync-key.pub** dans tous les fichiers **authorized_keys** de vos serveurs de destination.

#### B-Utilisation de clés pour SSH SANS ACCES root

>**A faire sur tous les serveurs distants**

Création utilisateur backupuser et d’un jeu de clé ssh

    sudo adduser backupuser  #mot de passe à saisir 

Cette commande va vous demander plusieurs informations et notamment un mot de passe à noter impérativement. N’hésitez pas à définir un mot de passe compliqué, d’environ 16 caractères avec des chiffres, des lettres majuscule/minuscule et quelques caractères spéciaux car vous n’aurez à le taper réellement qu’une fois.

Création dossier .ssh

    sudo mkdir /home/backupuser/.ssh/

Modifier les droits

    sudo chown backupuser. -R /home/backupuser/.ssh/
    sudo chmod 644 -R /home/backupuser/.ssh/

On se connecte en backupuser

    sudo su backupuser
    ssh-keygen -t rsa
    chmo 400 id_rsa*       #lecture seule pour utilisateur backupuser 

Edition fichier sudoers pour un accès root à l’exécution de rsync

    sudo -s

On donne à l'utilisateur backupuser les droits d'exécution en mode root de rsync  
Ajouter ligne suivante en fin du fichier **/etc/sudoers**

    echo "backupuser ALL=NOPASSWD: /usr/bin/rsync" >> /etc/sudoers


