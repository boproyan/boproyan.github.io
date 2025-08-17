+++
title = 'SSHFS pour monter des dossiers distants dans le système de fichier (ssh + fuse) et autofs'
date = 2021-02-20 00:00:00 +0100
categories = ['cli', 'ssh']
+++
# SSHFS 

*outil permettant d'utiliser le protocole ssh comme un système de fichiers*

## Liens

[SSHFS](https://fr.wikipedia.org/wiki/Secure_shell_file_system) permet d'utiliser un [serveur ssh](https://doc.ubuntu-fr.org/ssh) afin de monter des dossiers distants disponibles dans le système de fichier grâce à [fuse](https://fr.wikipedia.org/wiki/Filesystem_in_Userspace).
Il ne nécessite côté serveur que openssh-server et côté client fuse openssh-client sshfs et éventuellement tor.


* [[Tuto/HowTo] Configurer et monter SSHFS sécurisé via utilisateur dédié côté serveur](https://www.0rion.netlib.re/forum4/viewtopic.php?f=68&t=339&p=847#p847)
* [[Tuto/HowTo] [GNU/Linux] Montage SSHFS cross-canal (Lan, Wan, Tor, etc)](https://www.0rion.netlib.re/forum4/viewtopic.php?f=68&t=339&p=847#p893)
 * [Man Page adduser [EN]](http://linux.die.net/man/8/adduser)
 * [[Tuto/HowTo] [GNU/Linux] SSH](https://192.168.1.65/forum4/viewtopic.php?f=40&t=295)
 * [[Tuto/HowTo] Monter dossier distant sur Raspberry Pi & Ubuntu/Debian](https://192.168.1.65/forum4/viewtopic.php?f=68&t=288#p740)
 * [[Tuto/HowTo] Garder un contrôle discret sur son serveur avec tor et ssh](https://192.168.1.65/forum4/viewtopic.php?f=45&t=159)
 * [LinuxTricks - Autour du protocole SSH, utiliser le canal de communication](http://www.linuxtricks.fr/wiki/autour-du-protocole-ssh-utiliser-le-canal-de-communication#paragraph_sshfs-systeme-de-fichiers-sur-ssh)
 * [LinuxFR Journal - SSHFS est un vrai système de fichiers en réseau](https://linuxfr.org/users/elessar/journaux/sshfs-est-un-vrai-systeme-de-fichiers-en-reseau)
 * [Automounting remote shares via autofs](https://whattheserver.com/automounting-remote-shares-via-autofs/)

## Options sshfs

```
Les options du SSHFS :

-p PORT
    équivalent à "-o port=PORT". 
-C

équivalent à "-o compression=yes".
-F ssh_configfile
    spécifie un autre fichier de configuration ssh 
-1

équivalent à "-o ssh_protocol=1".
-o reconnect
    se reconnecter au serveur 
-o delay_connect
    retarder la connexion au serveur 
-o sshfs_sync
    écritures synchrones 
-o no_readahead
    lectures synchrones (pas de readahead spéculatif) 
-o sshfs_debug
    imprimer quelques informations de débogage 
-o cache=BOOL
    activer la mise en cache {yes, no} (par défaut : yes) 
-o cache_timeout=N
    définit le délai d'attente pour les caches en secondes (par défaut : 20) 
-o cache_X_timeout=N
    définit le délai d'attente pour le cache {stat,dir,link}. 
-o workaround=LIST
    liste de contournements séparés par deux points 
    none
        aucune solution de contournement n'est possible
    all
        toutes les solutions de contournement possibles 
    [no] renommer 
        Correction du renommage d'un fichier existant (par défaut : off)
    [no]nodelaysrv 
        set nodelay tcp flag in ssh (default : off)
     [no]truncate  
        correction de la troncature des anciens serveurs (par défaut : off)
    [no]buflimit 
        correction d'un bogue de remplissage de la mémoire tampon du serveur (par défaut : on)

-o idmap=TYPE
    la cartographie des ID d'utilisateurs/de groupes (user/group), les types possibles sont 
    none
         pas de traduction de l'espace d'identification (par défaut)
    user
         ne traduire que l'UID de l'utilisateur connecté

    file
         traduire les UID/GID en se basant sur le contenu des uidfile et gidfile 
-o uidfile=FILE
    fichier contenant le nom d'utilisateur:uid mappings for idmap=file 
-o gidfile=FILE
    fichier contenant le nom de groupe : gid mappings for idmap=file 
-o nomap=TYPE
    avec idmap=file, comment gérer les cartographies manquantes 
    ignore 
         ne pas refaire de cartographie
    error
        retourner une erreur (par défaut) 

-o ssh_command=CMD
    exécuter CMD au lieu de "ssh". 
-o ssh_protocol=N
    protocole ssh à utiliser (par défaut : 2) 
-o sftp_server=SERV
    chemin d'accès au serveur ou sous-système sftp (par défaut : sftp) 
-o directport=PORT
    se connecter directement au PORT en contournant ssh -o esclave communiquer sur le réseau stdin et stdout bypassing 
-o transform_symlinks
    transformer les liens symboliques absolus en liens symboliques relatifs 
-o follow_symlinks
    suivre les liens symboliques sur le serveur 
-o no_check_root
    ne pas vérifier l'existence d'un "dir" sur le serveur 
-o password_stdin
    lire le mot de passe depuis stdin (uniquement pour pam_mount !) 
-o SSHOPT=VAL
    Options ssh (voir man ssh_config) 
```

## Configuration rapide client serveur

*Note : Les paquets “openssh-client” et “fuse-utils” sont requis, mais il sont déjà installés par défaut*

### client

Il faut disposer d'un serveur ssh focntionnel

### serveur

Installer sshfs

    sudo apt update && sudo apt install sshfs # debian
    sudo pacman -S sshfs  # archlinux/manjaro

### Test rapide

Sur le système client, créer un répertoire dans lequel va être monté le système de fichiers :

    mkdir /home/utilisateur/Test

Monter un répertoire distant (ici ~/Public):

    sshfs utilisateur@ip_distante:/home/utilisateur/Public /home/utilisateur/Test

Le mot de passe de l'utilisateur est alors demandé.

Ouvrir l gestionnaire de fichiers pour accèder à vos fichiers distants. 

Enfin, penser à démonter :

    fusermount -u /home/utilisateur/Test

### SSHFS avec clés et port différent

`sshfs -oIdentityFile=<clé privée> utilisateur@domaine.tld:<dossier distant> <dossier local> -C -p <port si différent de 22>`

### SSHFS fstab

Comment puis-je monter de façon permanente le système de fichiers distant en mettant à jour /etc/fstab ?

Pour les montages permanents, vous devez créer une connexion basée sur les clés ssh

    ssh-keygen -t rsa
    ssh-copy-id -i ~/.ssh/id_rsa.pub vivek@server1.cyberciti.biz

Maintenant, éditez le fichier **/etc/fstab**

    sudo nano /etc/fstab

La syntaxe est :

    userNameHere@FQDN_OR_IP_HERE:/path/to/source/  /local/mountdir/  fuse.sshfs  defaults,_netdev  0  0

Ajoutez l'entrée suivante au bas du fichier fstab :

    user@192.168.1.142:/ /mnt/server1

Un autre exemple avec des options supplémentaires :

    user@192.168.1.142:/ /mnt/server1 fuse defaults,idmap=user,allow_other,reconnect,_netdev,users,IdentityFile=/path/to/.ssh/keyfile 0 0

Option de montage à la demande avec **systemd** :

    USERNAME@HOSTNAME_OR_IP:/REMOTE/DIRECTORY  /LOCAL/MOUNTPOINT  fuse.sshfs x-systemd.automount,_netdev,user,idmap=user,transform_symlinks,identityfile=/home/USERNAME/.ssh/id_rsa,allow_other,default_permissions,port=22,uid=USER_ID_N,gid=USER_GID_N 0 0
    vivek@server1.cyberciti.biz:/project/www/ /mnt/server1  fuse.sshfs noauto,x-systemd.automount,_netdev,users,idmap=user,IdentityFile=/home/vivek/.ssh/id_rsa,allow_other,reconnect 0 0

Descripton des options

*    **user@192.168.1.142** : Serveur distant avec sshd
*    **fuse** : Type de système de fichiers.
*    **idmap=user** : Ne traduit que l'UID de l'utilisateur connecté.
*    **allow_other** : Autorise l'accès aux autres utilisateurs.
*    **reconnect** : Se reconnecter au serveur.
*    **_netdev** : Le système de fichiers réside sur un périphérique qui nécessite un accès au réseau (utilisé pour empêcher le système de tenter de monter ces systèmes de fichiers tant que le réseau n'a pas été activé sur le système).
*    **users** : Permet à chaque utilisateur de monter et de démonter le système de fichiers.
*    **IdentityFile=/chemin/vers/.ssh/keyfile** : fichier de clés SSH.

### Autorisations

**"utilisateur" et/ou "root" des supports FUSE**

**Accès utilisateur**  

* Exécuter `sshfs` (ou toute autre commande de montage FUSE) avec l’option `-o allow_other`

**Accès root**  
Il y a des endroits que même les root ne peuvent pas atteindre. L'un de ces endroits est un volume monté sur FUSE. Je l'ai découvert en essayant d'accéder à un répertoire monté sshfs en tant que root et je me suis vu refuser la permission.

    # ls /home/user/dossier

```
ls: impossible d'accéder à '/home/user/dossier': Permission non accordée
```

Pour contourner ce problème, faites ce qui suit (ce que j'ai appris en me renseignant sur la défaillance du serveur) :


1. Ajoutez `user_allow_other` au fichier **/etc/fuse.conf**  
2. Exécutez `sshfs` (ou toute autre commande de montage FUSE) avec l'option `-o allow_root`  

En cas d'erreur "Permission denied" 

    sudo gpasswd -a "$USER" fuse

## autofs

[Automatisation des actions à distance via les autofs](/posts/autofs/#automatisation-via-autofs)