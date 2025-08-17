+++
title = 'autofs'
date = 2021-02-20 00:00:00 +0100
categories = application
+++
## autofs

*Autofs est un démon de service qui monte et remonte automatiquement tous les partages distants sshfs, NFS et autres types de partages pour vous, à la demande. Chaque fois que le point de montage est accédé, il remonte le partage distant s'il n'est pas déjà monté. C'est très efficace et bien plus efficace que de faire des montages fstab personnalisés qui peuvent ne pas être fiables si votre machine subit des problèmes de réseau et ne se rétablit pas.*

Installation

	sudo pacman -S autofs       # archlinux/manjaro
    sudo apt-get install autofs # debian/ubuntu

Modification hosts

	sudo nano /etc/hosts

192.168.0.45	shuttle

Les montages NFS de shuttle  

```
[yann@e6230 ~]$ showmount -e shuttle
Export list for shuttle:
/media/video                 192.168.0.0/24
/media/yanplus/BiblioCalibre 192.168.0.0/24
/media/yanplus/Musique       192.168.0.0/24
/media/yanplus/devel         192.168.0.0/24
```

Découverte "Auto"

AutoFS offre une nouvelle façon de découvrir et de monter automatiquement des partages sur des serveurs distants . Pour activer la détection automatique et le montage de partages réseau de tous les serveurs accessibles sans autre configuration, vous aurez besoin de vérifier ou d'ajouter la ligne suivante au fichier **/etc/autofs/auto.master**:  

  sudo nano /etc/autofs/auto.master

```
  /net -hosts
```

Chaque nom d'hôte doit être résolu, par exemple le nom d'une adresse IP dans **/etc/hosts** ou via DNS 
  sudo nano /etc/hosts 
le serveur est à l'adresse 192.168.0.12 et on l'appelle cubieboard

Modifier le **hosts** pour ajouter le nom du serveur nfs et son adresse IP

	sudo nano /etc/hosts


```
[...]
192.168.0.45	shuttle
[...]
```

Après relance 

   sudo systemctl restart autofs

On découvre les partages de **shuttle**

  ls -l /net/shuttle


```
total 0
dr-xr-xr-x 4 root root 0 24 juin  16:37 media
```
   
  ls -l /net/shuttle/media/yanplus

```
total 0
drwxr-xr-x 2 root root 0 24 juin  16:48 BiblioCalibre
drwxr-xr-x 2 root root 0 24 juin  16:48 Musique
drwxr-xr-x 2 root root 0 24 juin  16:48 devel
```
   
Lancement manuel

	sudo systemctl start autofs

Status

	sudo systemctl status autofs

```
● autofs.service - Automounts filesystems on demand
   Loaded: loaded (/usr/lib/systemd/system/autofs.service; disabled; vendor pres
   Active: active (running) since Sat 2017-06-24 16:26:49 CEST; 12s ago
  Process: 3901 ExecStart=/usr/bin/automount $OPTIONS --pid-file /run/autofs.pid
 Main PID: 3903 (automount)
    Tasks: 6 (limit: 4915)
   CGroup: /system.slice/autofs.service
           └─3903 /usr/bin/automount --pid-file /run/autofs.pid

juin 24 16:26:49 e6230 systemd[1]: Starting Automounts filesystems on demand...
juin 24 16:26:49 e6230 systemd[1]: Started Automounts filesystems on demand.
```

Pour la prise en compte autofs au démarrage

	sudo systemctl enable autofs

Créer un lien dans le dossier $HOME

	ln -s /net/shuttle/media/yanplus $HOME/media

## Automatisation via autofs

*Automatisation des actions à distance via les autofs*

Une fois installé, nous avons configuré notre support de base par défaut pour toutes les actions qui allaient être ajoutées.  
Toutes les actions ajoutées apparaissent sous le "/mnt" comme un sous-dossier.

Pour configurer cela, j'ai donc ajouté la ligne ci-dessous au bas du fichier situé ici `/etc/auto.master`  

*Veuillez noter : si vous souhaitez que ce fichier soit exécuté et monté sous un utilisateur sudo, les uid et gid, alias userid et groupid, devront être ajustés pour correspondre aux vôtres.*  
ouvrir et modifier le fichier

    sudo nano /etc/auto.master

La ligne ci-dessous suppose que le fichier sera monté sous l'utilisateur root.

    /mnt /etc/auto.sshfs uid=0,gid=0,--timeout=30,--ghost

Maintenant que nous avons la configuration de base du chemin par défaut, nous pouvons nous concentrer sur l'ajout des points de montage. 

S'il s'agit d'un chemin basé sur **ssh/sshfs** pour un serveur distant, nous vous recommandons vivement de vous assurer que vous avez déjà configuré l'authentification de base par clé sans mot de passe entre les machines. Ceci est fortement recommandé afin qu'il n'y ait pas d'invite de mot de passe ou de mot de passe enregistré dans un fichier texte ou une commande locale pour des raisons de sécurité. Si cela n'est pas déjà fait, [veuillez consulter un tutoriel comme celui-ci pour obtenir cette configuration en premier lieu (en)](http://sshmenu.sourceforge.net/articles/key-setup.html).  
Veuillez vous assurer que lors de la génération de la paire de clés, le mot de passe est vide, c'est-à-dire qu'il faut appuyer deux fois sur la touche Entrée lorsqu'on vous demande le mot de passe plutôt que d'en spécifier un, et ce, par l'intermédiaire de la machine qui va accéder au partage à distance.
{: .prompt-warning }

Avec cette configuration, nous pouvons créer les points de montage.

Dans mon cas, je voulais monter 2 dossiers distants basés sur sshfs pour accéder aux sauvegardes et les stocker.

Les chemins d'accès aux dossiers :  
morgan  
rsyncnet  

Ces chemins finiront par devenir les chemins du dessous une fois montés.

    /mnt/morgan
    /mnt/rsyncnet

Il faudrait donc d'abord fabriquer la chaîne de montage de sshfs nécessaire pour les deux. Voir les exemples ci-dessous. Vous remplaceriez le nom devant et la section "Username@HostnameorIP:/remote/path" par vos informations de connexion sshfs.

    morgan  fstype=fuse,rw,nodev,nonempty,noatime,allow_other,max_read=65536 :sshfs#Username@HostnameorIP:/remote/path
    rsyncnet -fstype=fuse,rw,nodev,nonempty,noatime,allow_other,max_read=65536 :sshfs#Username@HostnameorIP:/remote/path


Une fois que vous avez créé les chaînes de montage, nous devons les ajouter au fichier "/etc/auto.sshfs". Créons ou modifions donc ce fichier et ajoutons la ou les lignes que vous avez créées ci-dessus. Nous allons utiliser nano pour cela via l'utilisateur root ou sudo, puis quitter et enregistrer.

    sudo nano /etc/auto.sshfs

Maintenant, si vous avez déjà fait monter ces chemins manuellement via fstab, vous voudrez les démonter avant de continuer.

    sudo umount /previous/mount/point

Nous devons maintenant recharger le service autofs pour qu'il voit les changements de configuration que nous avons apportés aux fichiers `/etc/auto.master` et `/etc/auto.sshfs`  

    sudo systemctl restart autofs 

Nous pouvons maintenant tester son fonctionnement en effectuant un "df -h" et voir si les supports sont actuellement là. Si ce n'est pas le cas, nous pouvons tester l'automount en passant au point de montage ou en essayant d'y accéder pour le démarrer. Dans mon cas, nous avons testé avec le premier point de montage /mnt/morgan

```
cd /mnt/morgan
ls -lah /mnt/morgan
```

Une fois que vous essayez d'y accéder, il se peut que la première fois qu'il se connecte, il se peut que vous soyez bloqué pendant une seconde, alors vous devriez voir une liste de tous les fichiers qui se trouvent dans ce chemin. Ensuite, pour voir s'il est monté, vous pouvez faire un autre "df -h" pour voir les points de montage actuels. Il devrait ressembler à quelque chose comme ceci.

```
root@coby:~# df -h
 Filesystem                                       Size  Used Avail Use% Mounted on
 udev                                              63G     0   63G   0% /dev
 tmpfs                                             13G  1.3G   12G  11% /run
 rpool/ROOT/pve-1                                 424G  276G  149G  65% /
 tmpfs                                             63G   37M   63G   1% /dev/shm
 tmpfs                                            5.0M     0  5.0M   0% /run/lock
 tmpfs                                             63G     0   63G   0% /sys/fs/cgroup
 rpool                                            149G     0  149G   0% /rpool
 rpool/ROOT                                       149G     0  149G   0% /rpool/ROOT
 rpool/data                                       149G     0  149G   0% /rpool/data
 tmpfs                                             13G     0   13G   0% /run/user/0
 /dev/fuse                                         30M   72K   30M   1% /etc/pve
 /dev/sdd                                         916G   80M  870G   1% /sdd
 user@hostname:/remote/path  708G  131G  577G  19% /mnt/morgan   <<<<<<<<<<<< Note the first mount
 user@hostname:                    1.1T  538G  563G  49% /mnt/rsyncnet  <<<<<Note the second mount also shows
```

Liens

* https://wiki.archlinux.org/index.php/Autofs
* https://linux.die.net/man/5/autofs
* https://help.ubuntu.com/community/Autofs
* https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/4/html/System_Administration_Guide/Mounting_NFS_File_Systems-Mounting_NFS_File_Systems_using_autofs.html
