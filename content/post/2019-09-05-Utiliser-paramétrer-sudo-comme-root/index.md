+++
title = 'Utiliser et paramétrer sudo pour avoir les accès "root"'
date = 2019-09-05 00:00:00 +0100
categories = ['cli']
+++
### sudo

*sudo est une commande permettant à l'administrateur système d'accorder à certains utilisateurs (ou groupes d'utilisateurs) la possibilité de lancer une commande en tant qu'administrateur.*

Si non installé

	su                 # passer en administrateur
	apt install sudo   # installer sudo (debian)



### Utiliser sudo


sudo s'utilise en ligne de commande, dans un terminal. Il permet de prendre les droits root pour exécuter une commande.

Le mot de passe demandé est celui de l'utilisateur qui a saisi sudo, ici yann.
La commande sera exécutée si le mot de passe entré est correct et que l'utilisateur courant peut effectuer des tâches d'administration.

```bash
yann@linux: ~  $ ls /root
ls: impossible d ouvrir le répertoire /root: Permission non accordée
yann@linux: ~  $ sudo ls /root
[sudo] password for yann: 
script-de-root.sh
```

>Le mot de passe est mémorisé par défaut pour une durée de 15 minutes.  
Pendant ce laps de temps, on peut toujours saisir des commandes avec sudo mais le mot de passe ne sera pas demandé.

Pour "effacer" le mot de passe mémorisé

	sudo -k

Ouvrir une "session root" 

	sudo -i

`sudo -i` (mot de passe utilisateur demandé) identique à la commande `su -` (mot de passe root demandé) en terme de droits.

### Configurer sudo (/etc/sudoers)


Les autorisations pour utiliser sudo sont définies dans le fichier **/etc/sudoers**

	visudo              # Ouvrir en mode "root" et nous indiquera s'il y a une erreur de syntaxe
	nano /etc/sudoers   # alternative



Le fichier modifié par défaut est **/etc/sudoers**  (possibilité de créer un fichier dans le dossier /etc/sudoers.d)


Structure /etc/sudoers

```bash
utilisateur ALL = (utilisateur) commande1,commande2,...
%groupe ALL = (utilisateur) commande1,commande2,...
```


* **utilisateur** ,le nom  auquel le droit sudo s'appliquera 
* **%groupe** ,le nom du groupe auquel le droit sudo s'appliquera. Le nom du groupe doit donc être précédé d'un %.
* **ALL** désigne la ou les systèmes dans lesquels le droit sudo s'appliquera
     * **(utilisateur)** désigne l'utilisateur dont on prend les droits (**ALL** pour tous, **root** compris)
     * **commande**  pouvant être exécutées par l'utilisateur ou le groupe désigné en début de ligne. (si plusieurs commandes, les séparer par une virgule. Utiliser le chemin complet de la commande. (Si on ne connaît pas le chemin de la commande, utiliser which : which ls) Il est possible de mettre un point d'exclamation (!) devant la commande pour interdire à l'utilisateur de l'exécuter avec sudo.

>Un seul groupe ou utilisateur par ligne.


### Utiliser sudo sans mot de passe

On peut indiquer dans le fichier sudoers de ne pas utiliser de mot de passe pour certaines commandes (utile dans le cadre d'un script)  
Voici des exemples avec un utilisateur (fonctionne aussi avec les groupes) 

```bash
utilisateur ALL = commande1, NOPASSWD: commande2   # commande1 est autorisée via sudo avec un mot de passe, commande2 est autorisée via sudo sans demande de mot de passe.
backup      ALL=(ALL) NOPASSWD: /usr/sbin/rsync    # Autoriser backup à lancer la commande rsync avec sudo sans demande de mot de passe
yann        ALL=(ALL) NOPASSWD: ALL                # Autoriser yann à lancer toutes les commandes  sans demande de mot de passe
yann        ALL=(ALL) ALL                          # Autoriser l'utilisateur yann à exécuter toutes les commandes via sudo ave demande de mot de passe
yann ALL=(ALL) NOPASSWD: /usr/sbin/reboot          # sudo reboot ne demande pas le mot de passe
```
