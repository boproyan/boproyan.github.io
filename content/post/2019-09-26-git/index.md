+++
title = 'Git'
date = 2019-09-26 00:00:00 +0100
categories = ['git']
+++
# Git

## Débuter avec Git 

* [Créer un dépôt](https://carlchenet.com/debuter-avec-git-creer-un-depot/)
* [Premier ajout de code](https://carlchenet.com/debuter-avec-git-premier-ajout-de-code/)
* [Un commit plus complexe](https://carlchenet.com/debuter-avec-git-partie-3-un-commit-plus-complexe/)
* [Les commits et les branches](https://carlchenet.com/debuter-avec-git-partie-4-les-commits-et-les-branches/)
* [Fusionner des branches](https://carlchenet.com/debuter-avec-git-partie-5-fusionner-des-branches/)
* [Une fusion de branches échoue](https://carlchenet.com/debuter-avec-git-partie-6-une-fusion-de-branches-echoue/)


## Commandes de base

Commande git | Description
--- | --- 
git clone URL | *Récupère un dépôt à partir d'une adresse URL*
git add fichier  | *Marque le fichier à consigner*
git add *  | *Marque tous les fichiers à consigner*
git commit -m "​description" | *Consigne les fichiers marqués dans le dépôt*
git status  | *Affiche l'état du dépôt local*
git tag étiquette | *Pose une étiquette de version sur les fichiers*
git reset --hard HEAD~1 | *Supprime la dernière  modification consignée (avant d'avoir fait un push)*
git log  | *Affiche l'historique descommandes (q pour sortir)*
git config --list  | *Affiche la liste des paramètres de git*
git config --global user.name NomUtilisateur  | *Enregistre le paramètre "nom utilis​ate​ur" du dépôt*
git config --global user.email email  | *Enregistre l'email de l'util​isateur du dépôt*
git branch testing | *Créer une nouvelle branche*
git checkout testing | *Cela déplace HEAD pour le faire pointer vers la branche testing*
git checkout -b nombranche | *Créer une branche et se positionner dessus*
git push origin nombranche | *Pousser la nouvelle branche sur un dépôt distant*
git checkout -b nombranchelocal origin/nombranchedistante | *Faire un checkout à partir d’une branche distante*
git branch -d nombranche | *Supprimer une branche localement*
git push origin :nombranche | *Propager la suppression sur le dépôt distant* 
git tag -a v1.4 -m 'ma version 1.4' | *Etiquette annotée*
git tag -a v1.4 | *Etiquette légère*
git show v1.4 | *visualiser les données de l’étiquette*
git push origin v1.5 | *Il faut explicitement pousser les étiquettes après les avoir créées localement.*
git push origin --tags | *pousser toutes les étiquettes*
git tag | *liste des étiquettes*

## Utilisation

### Utilisateur sur un ordinateur local  

Configuration globale **~/.gitconfig**

	git config --global user.name  "user"
	git config --global user.email "user@exemple.com"

Modifier l'éditeur de texte par défaut (vi) de git

	git config --global core.editor vim  # éditeur vim
	git config --global core.editor nano  # éditeur nano
	git config --global core.editor "atom --wait"  # éditeur atom

### Créer un dépôt local

Création dépôt  local **projet_local** 
  
	mkdir ~/chemin/projet_local

#### Initialisation d'un dépôt local 

	cd ~/chemin/projet_local

Copier dans le répertoire (dépôt local) fichiers et dossiers du projet  
Valider les changements

	git init      # Création des paramètres de configuration dans le dossier .git/
	git add .
	#Le dossier est donc ajouté au HEAD, mais pas encore dans votre dépôt distant.
	git commit -m "Initialisation"  

Vos changements sont maintenant dans le HEAD de la copie de votre dépôt local.  
Pour les envoyer à votre dépôt distant, exécutez la commande

	git remote add origin https://gitlab.monDepotGit.com/projet_local.git # création
	git push -u origin master   # utilisateur et mot de passe demandés

Remplacez **master** par la branche dans laquelle vous souhaitez envoyer vos changements.

### Connexion auto 

Si on utilise un projet GIT via une connexion http(s) nécessitant une authentification, elle est demandée à chaque **push** , **pull** et **commit** effectués.

Se rendre dans le répertoire d'un projet 

    cd /chemin/projet

Pour autoriser le stockage et l'accès des informations de connexion

    git config credential.helper store

Faire un premier push , utilisateur et mot de passe sont demandés et création du fichier **~/.git-credentials**

    git push

>Cette opération va générer un fichier **~/.git-credentials** contenant les paramètres de connexion

Pour les projets à suivre , il suffit d'autoriser dans le répertoire du projet l'accès aux informations de connexion (le fichier **~/.git-credentials** existe) 

    git config credential.helper store


### Création manuelle configuration  gitconfig

#### .gitconfig

Le fichier **~/.gitconfig **:

```
[user]
        name = monNom
        email = monEmail
[credential]
        helper = store
[http]
        sslVerify = false
        postBuffer = 524288000
```

- **[user]** contient les infos personnelles (nom, email etc etc ...)
- **[credential]** avec la variable `helper = store` pour autoriser la connexion auto 
-  **[http]** (facultatif) , la variable `sslVerify = false` permet de se connecter en https sur un dépot GIT avec un certificat non reconnu (auto-signé), `postBuffer = 524288000` permet de résoudre l'erreur `error: RPC failed; result=22, HTTP code = 411` lors des commits et autorise tous les projets locaux d'envoyer jusqu'à 500 Mo de données.

> Si le paramètre [credential] n'est pas ajouté manuellement , il faudra exécuter dans chaque répertoire des dépôts locaux : `git config credential.helper store` (pour autoriser la connexion auto) 

#### .git-credentials

Le fichier **~/.git-credentials** :  
Ce fichier contient l' **utilisateur** associé au **mot de passe**  renseignés dans le fichier **~/.gitconfig**  
Ce fichier doit être lisible uniquement par l'utilisateur concerné **(chmod 0600)**.

    https://utilisateur:mot_de_passe@gitlab.monDepotGit.com

Si **utilisateur** et/ou le **mot_de_passe** contient un des caractères spécifiques à la chaine de connexion (:/@), il est obligatoire de l'écrire en hexadécimal.

	@ = %40 en hexadécimal

Sinon, le simple fait de créer le fichier **~/.git-credentials** et de lui attribuer le bon mode **0600**, il sera automatiquement complété avec le bonne combinaison *user:password@adresseHttps* à la première utilisation de *git pull* ou *git push* .

### Erreurs

git push --set-upstream origin master

```
Énumération des objets: 145, fait.
Décompte des objets: 100% (145/145), fait.
Compression par delta en utilisant jusqu'à 4 fils d'exécution
Compression des objets: 100% (143/143), fait.
Écriture des objets: 100% (145/145), 24.66 MiB | 28.34 MiB/s, fait.
Total 145 (delta 2), réutilisés 0 (delta 0)
error: RPC failed; HTTP 413 curl 22 The requested URL returned error: 413
fatal: L'hôte distant a fermé la connexion de manière inattendue
fatal: L'hôte distant a fermé la connexion de manière inattendue
Everything up-to-date
```

Erreur : `error: RPC failed; HTTP 413 curl 22 The requested URL returned error: 413`  
Solution : Ouvrir le fichier de configuration nginx **nginx.conf** et d'ajouter ou modifier `client_max_body_size 50m ;` selon vos besoins  dans le bloc http. 

## Liens

* [git essentials – 1 – log](http://blog.xebia.fr/2015/11/13/git-essentials-1-log/)
* [git essentials – 2 – bisect](http://blog.xebia.fr/2016/01/18/git-essentials-2-bisect/)
* [git essentials – 3 – les alias](http://blog.xebia.fr/2016/04/06/git-essentials-3-les-alias/)
* [git essentials – 4 – rebase](http://blog.xebia.fr/2016/06/15/git-essentials-4-rebase/)
* [git essentials – 5 – visualiser les changements](http://blog.xebia.fr/2018/01/10/git-essentials-5-visualiser-les-changements/)

