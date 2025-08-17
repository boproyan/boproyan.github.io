+++
title = 'Configurer la synchronisation de fichiers entre deux serveurs avec Unison'
date = 2022-12-25 00:00:00 +0100
categories = rsync
+++
*Comment configurer la synchronisation de fichiers entre deux serveurs Debian avec Unison qui est un outil de synchronisation de fichiers similaire à rsync, la grande différence est qu'il suit/synchronise les changements dans les deux sens, c'est-à-dire que les fichiers modifiés sur le serveur1 seront répliqués sur le serveur2 et vice versa.*

- [Unison](#unison)
    - [Remarque préliminaire](#remarque-préliminaire)
    - [Installer Unison](#installer-unison)
    - [Création d'une paire de clés privée/publique sur le serveur1](#création-dune-paire-de-clés-privéepublique-sur-le-serveur1)
    - [Lancer unison](#lancer-unison)
    - [Création d'une tâche Cron pour Unison](#création-dune-tâche-cron-pour-unison)
    - [Tester Unison](#tester-unison)
    - [Liens](#liens)
- [Unison pour une synchronisation de fichiers bidirectionnelle automatisée sous Linux](#unison-pour-une-synchronisation-de-fichiers-bidirectionnelle-automatisée-sous-linux)
    - [Debian](#debian)


## Unison

### Remarque préliminaire

Dans ce tutoriel, j'utiliserai les deux serveurs Debian suivants :

    server1.example.com avec l'adresse IP 192.168.0.100
    server2.example.com avec l'adresse IP 192.168.0.101

Je souhaite synchroniser le répertoire /var/www entre les deux serveurs. Je vais exécuter Unison en tant qu'utilisateur root dans ce didacticiel afin qu'Unison dispose des autorisations suffisantes pour synchroniser les autorisations des utilisateurs et des groupes.

Toutes les commandes de ce didacticiel sont exécutées en tant qu'utilisateur root. Connectez-vous aux deux serveurs sur le shell en tant que root et commencez par l'étape 2 " Installer Unison ".

### Installer Unison

Unison doit être installé sur server1 et server2 puisque nous nous connectons de server1 à server2 en utilisant SSH, nous avons également besoin des packages SSH et nano pour l'édition de fichiers sur le shell.

    sudo apt -y install unison  

### Création d'une paire de clés privée/publique sur le serveur1

Nous créons maintenant une paire de clés privée/publique sur server1.example.com

    ssh-keygen -t dsa

Il est important que vous n'entriez pas de phrase de passe, sinon la mise en miroir ne fonctionnera pas sans interaction humaine

nous copions notre clé publique dans server2.example.com

    ssh-copy-id -i $HOME/.ssh/id_dsa.pub root@192.168.0.101

Vérifiez maintenant sur le serveur2 si la clé publique du serveur1 a bien été transférée

    cat $HOME/.ssh/authorized_keys

```
cat $HOME/.ssh/authorized_keys
ssh-dss AAAAB3NzaC1kc3MAAAFq8WoGROgarBpCbQU3w2GUUnM= root@server1
```

### Lancer unison

serveur1 :  
Nous pouvons maintenant exécuter Unison pour la première fois pour synchroniser le répertoire /var/www   sur les deux serveurs. Sur le serveur1 , exécutez

    unison /var/www ssh://192.168.0.101//var/www

La sortie sera similaire à celle-ci - vous devrez peut-être répondre à quelques questions car c'est la première fois qu'Unison est exécuté

```
root@server1:/var/www# unison /var/www ssh://192.168.0.101//var/www
Contacting server...
Connected [//server1//var/www -> //server2//var/www]
Looking for changes
Warning: No archive files were found for these roots, whose canonical names are:
/var/www
//server2//var/www
This can happen either
because this is the first time you have synchronized these roots,
or because you have upgraded Unison to a new version with a different
archive format.

Update detection may take a while on this run if the replicas are
large.

Unison will assume that the 'last synchronized state' of both replicas
was completely empty. This means that any files that are different
will be reported as conflicts, and any files that exist only on one
replica will be judged as new and propagated to the other replica.
If the two replicas are identical, then no changes will be reported.

If you see this message repeatedly, it may be because one of your machines
is getting its address from DHCP, which is causing its host name to change
between synchronizations. See the documentation for the UNISONLOCALHOSTNAME
environment variable for advice on how to correct this.

Donations to the Unison project are gratefully accepted:
http://www.cis.upenn.edu/~bcpierce/unison

Press return to continue.[<spc>] <-- Press Enter

Waiting for changes from server
Reconciling changes

local server2
dir ----> example.com [f] <-- Press Enter
dir ----> example.de [f] <-- Press Enter

Proceed with propagating updates? [] <-- Enter "y"
Propagating updates


UNISON 2.48.4 started propagating changes at 13:24:01.10 on 05 May 2020
[BGN] Copying example.com from /var/www to //server2//var/www
[BGN] Copying example.de from /var/www to //server2//var/www
Shortcut: copied /var/www/example.de/web/index.html from local file /var/www/.unison.example.com.d3783bddaaf59b9ba4d2ed0433f9db63.unison.tmp/web/index.html
[END] Copying example.de
[END] Copying example.com
UNISON 2.48.4 finished propagating changes at 13:24:01.98 on 05 May 2020


Saving synchronizer state
Synchronization complete at 13:24:01 (2 items transferred, 0 skipped, 0 failed)
```

Vérifiez le répertoire /var/www sur le serveur1 et le serveur2 maintenant, et vous devriez constater qu'ils sont maintenant synchronisés.

Nous ne voulons pas exécuter Unison de manière interactive, nous pouvons donc créer un fichier de préférences ( /root/.unison/default.prf ) qui contient tous les paramètres que nous devrions autrement spécifier sur la ligne de commande

    nano /root/.unison/default.prf

```
# Racines de la synchronisation
root = /var/www
root = ssh://192.168.0.101//var/www

# Chemins à synchroniser
#path = current
#path = common
#path = .netscape/bookmarks.html

# Certaines expressions rationnelles spécifiant les noms et les chemins à ignorer
#ignore = Path stats    ## ignores /var/www/stats
#ignore = Path stats/*  ## ignores /var/www/stats/*
#ignore = Path */stats  ## ignores /var/www/somedir/stats, but not /var/www/a/b/c/stats
#ignore = Name *stats   ## ignores all files/directories that end with "stats"
#ignore = Name stats*   ## ignores all files/directories that begin with "stats"
#ignore = Name *.tmp    ## ignores all files with the extension .tmp

# Lorsqu'il est défini sur true, cet indicateur fait sauter l'interface utilisateur
# demander des confirmations sur les modifications non conflictuelles. (Suite
# précisément, lorsque l'interface utilisateur a fini de définir le
# direction de propagation pour une entrée et est sur le point de passer à la
# ensuite, il ignorera toutes les entrées non conflictuelles et passera
# directement au prochain conflit.)
auto=true

# Lorsque ceci est défini sur vrai, l'interface utilisateur ne demandera pas
# questions du tout. Les modifications non conflictuelles seront propagées ;
# conflits seront ignorés.
batch=true

# !Lorsque ce paramètre est défini sur true, Unison demandera un extra
# confirmation s'il apparaît que l'intégralité de la réplique a été
# supprimé, avant de propager le changement. Si l'indicateur de lot est
# également défini, la synchronisation sera abandonnée. Quand le chemin
# préférence est utilisée, la même confirmation sera demandée pour
# chemins de niveau supérieur. (Pour le moment, ce drapeau n'affecte que le
# interface utilisateur textuelle.) Voir aussi la préférence de point de montage.
confirmbigdel=true

# Lorsque cette préférence est définie sur true, Unison utilisera la
# heure de modification et longueur d'un fichier en tant que `pseudo inode
# nombre' lors de l'analyse des répliques pour les mises à jour, au lieu de lire
# le contenu complet de chaque fichier. Sous Windows, cela peut provoquer
# Unison manquera la propagation d'une mise à jour si l'heure de modification
# et la longueur du fichier sont tous deux inchangés par la mise à jour.
# Cependant, Unison n'écrasera jamais une telle mise à jour avec un
# changement de l'autre réplique, car elle fait toujours un coffre-fort
# vérifie les mises à jour juste avant de propager un changement. Ainsi, il est
# raisonnable d'utiliser ce commutateur sous Windows la plupart du temps
# et exécutez occasionnellement Unison une fois avec fastcheck défini sur false,
# si vous craignez qu'Unison n'ait oublié une mise à jour.
# La valeur par défaut de la préférence est auto, ce qui provoque
# Unison pour utiliser la vérification rapide sur les répliques Unix (là où c'est sûr)
# et vérification lente sur les répliques Windows. Pour l'arrière
# compatibilité, oui, non et défaut peuvent être utilisés à la place de
# vrai, faux et automatique. Voir la section "Vérification rapide" pour plus
# renseignements.
fastcheck=true

# Lorsque cet indicateur est défini sur true, les attributs de groupe du
# fichiers sont synchronisés. Que les noms de groupe ou le groupe
# les identifiants sont synchronisés en fonction des numéros de préférence.
group=true

# Lorsque cet indicateur est défini sur true, les attributs de propriétaire du
# fichiers sont synchronisés. Que le nom du propriétaire ou le propriétaire
# les identifiants sont synchronisés dépend de la préférence
# exttnumérids.
owner=true

# Inclure la préférence -prefer root oblige Unison à toujours
# résoudre les conflits en faveur de root, plutôt que de demander
# conseils de l'utilisateur. (La syntaxe de root est la même que pour
# la préférence racine, plus les valeurs spéciales plus récentes et plus anciennes.)
# Cette préférence est remplacée par la préférence preferpartial.
# Cette préférence ne doit être utilisée que si vous êtes sûr de connaître
# que fais tu!
prefer=newer

# Lorsque cette préférence est définie sur true, l'interface utilisateur textuelle
# n'imprimera rien du tout, sauf en cas d'erreurs.
# Définir le silence sur vrai définit automatiquement la préférence de lot
# à vrai.
silent=true

# Lorsque ce drapeau est défini sur vrai, les heures de modification des fichiers (mais pas
# répertoire modtimes) sont propagés.
times=true
```

Les commentaires doivent rendre le fichier explicite, à l'exception des directives de chemin . Si vous ne spécifiez aucune directive de chemin , les répertoires des directives racine seront synchronisés. Si vous spécifiez des directives de chemin , alors les chemins sont relatifs au chemin racine (par exemple , root = /var/www et path = current se traduit par /var/www/current ), et seuls ces sous-répertoires seront synchronisés, pas l'ensemble du répertoire spécifié dans la directive racine .

Vous pouvez en savoir plus sur les options disponibles en consultant la page de manuel d'Unison

    man unison

Maintenant que nous avons mis tous les paramètres dans un fichier de préférences (en particulier les directives root (et éventuellement les path )), nous pouvons exécuter Unison sans aucun argument

    unison

### Création d'une tâche Cron pour Unison

serveur1 :

Nous voulons automatiser la synchronisation, c'est pourquoi nous créons une tâche cron pour celle-ci sur server1.example.com :

    crontab -e

```
*/5 * * * * /usr/bin/unison &> /dev/null
```

Cela exécuterait Unison toutes les 5 minutes, adaptez-le à vos besoins 

    man 5 crontab

J'utilise le chemin complet vers unison ici ( /usr/bin/unison ) juste pour m'assurer que cron sait où trouver unison .  
Votre emplacement d' unisson peut différer 

    which unison

pour savoir où se trouve le vôtre.

### Tester Unison

Maintenant, je vais tester la synchronisation bidirectionnelle d'Unison pour voir si la configuration fonctionne parfaitement.

Exécutez la commande suivante sur server1 pour créer un fichier de test avec le contenu "Test 1":

Serveur1

    echo "Test 1" > /var/www/test.txt

Attendez maintenant au moins 5 minutes (car nous avons créé un cronjob qui s'exécute une fois toutes les 5 minutes).  

Exécutez ensuite sur le serveur2 

    cat /var/www/test.txt

pour afficher le contenu du fichier test.txt qui est "Test 1" à l'écran.

Copier des fichiers entre serveurs via SSH à l'unisson sur Debian

Exécutez maintenant cette commande sur le serveur2 qui met à jour le contenu de notre fichier de test sur "Test 2"

Serveur2

    echo "Test 2" > /var/www/test.txt

Et attendez au moins 5 minutes. Exécutez ensuite la commande cat sur le serveur 1 

Serveur1

    cat / var/www/test.txt

La sortie doit être  "Test 2"

### Liens

*    Unisson : http://www.cis.upenn.edu/~bcpierce/unison/
*    Debian : http://www.debian.org/

## Unison pour une synchronisation de fichiers bidirectionnelle automatisée sous Linux

[How to use Unison for automated two-way file synchronization on Linux (Ubuntu) and Windows and Android](https://tech.tiq.cc/2016/04/how-to-use-unison-for-automated-two-way-file-synchronization-on-linux-ubuntu-and-windows-and-android/)

Unison est un excellent outil pour la synchronisation bidirectionnelle.

Le transfert de fichiers se fera via une connexion SSH avec authentification par clé.

### Debian

Sur le client et le serveur Ubuntu , vous devez installer Unison. Le serveur doit évidemment exécuter un serveur ssh et vous avez besoin d'outils comme nano (éditeur), cron et ssh-keygen.

    sudo apt-get -y install unison

Générez une paire de clés ssh sans mot de passe sur le client et copiez la clé publique sur le serveur , afin qu'Unison puisse se connecter sans mot de passe.

```shell
ssh-keygen -t rsa -b 8192 -o -a 1000 -f ~/.ssh/id_rsa_sync [run this on client]
ssh-copy-id -i ~/.ssh/id_rsa_sync.pub -p 22 tiq@example.org [run this on client]
```

Le nom par défaut de la clé RSA SSH est "id_rsa", mais j'utilise une clé différente pour chaque serveur. Vous devrez peut-être désactiver "SSH Key Agent" dans "Startup Applications" sur Ubuntu 14.04 Desktop pour que cela fonctionne.

**Après avoir créé des dossiers de test sur le client et le serveur** , testez la commande Unison avec des dossiers vides , puis ajoutez des fichiers au client et au serveur pour voir si cela fonctionne.

    /usr/bin/unison -sshargs='-p 22 -i /home/tiq/.ssh/id_rsa_sync' -batch -auto -confirmbigdel=false /home/tiq/sync/ ssh://tiq@example.org//home/tiq/sync/

Cela synchronisera les dossiers /home/tiq/sync/ sur le client et le serveur.

Une fois que cela fonctionne, ajoutez une tâche cron :

    crontab -e

Si l'éditeur crontab par défaut n'est pas nano, vous pouvez le modifier une fois en tapant EDITOR=nano crontab -e, ou de façon permanente en tapant `export EDITOR=nano` 

La tâche cron pour synchroniser toutes les minutes ressemble à ceci :

```
*/1 * * * * /usr/bin/flock -n /tmp/synclock -c /home/tiq/tc/sync.sh
```

nous devons utiliser flock pour qu'Unison ne s'exécute pas lorsqu'il est déjà en cours d'exécution, sinon il se casse. Vous pouvez utiliser incron pour n'exécuter la commande de synchronisation que lorsqu'un fichier est ajouté ou mis à jour/modifié, mais pour une véritable synchronisation bidirectionnelle, vous devez évidemment demander en permanence au serveur des modifications, c'est pourquoi j'ai choisi une tâche cron exécutée toutes les minutes.

sync.sh contient `#!/bin/bash`, suivi de la commande Unison ci-dessus et d'un `chmod +x sync.sh` pour le rendre exécutable.