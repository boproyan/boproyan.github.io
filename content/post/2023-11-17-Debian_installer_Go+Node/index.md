+++
title = 'Debian installer go + nodejs'
date = 2023-11-17 00:00:00 +0100
categories = go node
+++
*Installer Go et NodeJs sur debian*

- [Go](#go)
    - [Installer la dernière version de Go](#installer-la-dernière-version-de-go)
    - [Version installée](#version-installée)
- [Installer Node.js sur Debian 12, 11 ou 10 via NodeSource](#installer-nodejs-sur-debian-12-11-ou-10-via-nodesource)
    - [Ajouter le PPA de NodeSource sur Debian](#ajouter-le-ppa-de-nodesource-sur-debian)
    - [Installer Node.js sur Debian via la commande APT](#installer-nodejs-sur-debian-via-la-commande-apt)
- [Installer Node.js sur Debian 12, 11 ou 10 via le NVM](#installer-nodejs-sur-debian-12-11-ou-10-via-le-nvm)
    - [Installer le NVM sur Debian](#installer-le-nvm-sur-debian)
    - [Installer Node.js via la commande NVM sur Debian](#installer-nodejs-via-la-commande-nvm-sur-debian)
        - [Liste des versions de Node.js disponibles](#liste-des-versions-de-nodejs-disponibles)
        - [Installer une version de Node.js via la commande NVM](#installer-une-version-de-nodejs-via-la-commande-nvm)
        - [Vérifier l'installation de Node.js](#vérifier-linstallation-de-nodejs)
        - [Passer d'une version de Node.js à l'autre](#passer-dune-version-de-nodejs-à-lautre)
    - [Commandes supplémentaires de Node.js sur Debian 12, 11 ou 10](#commandes-supplémentaires-de-nodejs-sur-debian-12-11-ou-10)
        - [Supprimer Node.js de Debian](#supprimer-nodejs-de-debian)
        - [Désinstaller Node.js installé via NVM](#désinstaller-nodejs-installé-via-nvm)

## Go

![golang](golang-color-icon2.png){:width="100"} 

### Installer la dernière version de Go

Go installation (Debian) , installer la dernière version de Go &rarr; <https://golang.org/dl/>

```bash
cd ~
wget https://golang.org/dl/go1.15.7.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.15.7.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin"  >> ~/.bashrc
source ~/.bashrc
```

### Version installée

    go version

```
go version go1.15.7 linux/amd64
```


## Installer Node.js sur Debian 12, 11 ou 10 via NodeSource

![nodejs](Node_logo.png){:width="100"} 

Si vous souhaitez installer une version plus récente de Node.js, vous pouvez utiliser le PPA NodeSource. Cette méthode vous permet de choisir une version spécifique de Node.js et vous assure d'obtenir la dernière version.

### Ajouter le PPA de NodeSource sur Debian  

Pour ajouter le PPA NodeSource à votre système

Pour une version précise

    curl -fsSL https://deb.nodesource.com/setup_<version>.x | sudo -E bash -

Remplacez <version> par la version majeure de Node.js souhaitée. Cette commande téléchargera et exécutera un script qui ajoutera le PPA NodeSource à votre système et mettra à jour votre liste de paquets

Dernier dépôt stable

    curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash -

Dépôt de support à long terme LTS

    curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo bash -

### Installer Node.js sur Debian via la commande APT

Avec l'ajout du PPA NodeSource, vous pouvez maintenant installer Node.js en utilisant la commande suivante 

    sudo apt install nodejs

Cette commande installera la version spécifiée de Node.js ainsi que les dépendances nécessaires.  
Une fois l'installation terminée, vous pouvez vérifier la version installée en exécutant la commande suivante

    node -v

Cela affichera la version installée de Node.js sur votre système.

## Installer Node.js sur Debian 12, 11 ou 10 via le NVM

![nodejs](Node_logo.png){:width="100"} 

Une autre façon d'installer Node.js est d'utiliser le gestionnaire de versions de Node (NVM).  
Cette méthode vous permet de gérer plusieurs versions de Node.js sur votre système, ce qui facilite le passage d'une version à l'autre pour différents projets.

### Installer le NVM sur Debian

Pour installer NVM, exécutez la commande suivante

    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash

Cette commande télécharge et exécute le script d'installation du NVM. Une fois l'installation terminée, vous devez redémarrer votre terminal ou exécuter la commande suivante pour charger le NVM 

    source ~/.bashrc

### Installer Node.js via la commande NVM sur Debian

Dans cette étape, nous allons installer Node.js en utilisant le gestionnaire de versions de Node (NVM).   Le NVM vous permet de gérer plusieurs versions de Node.js sur votre système, ce qui facilite le passage d'une version à l'autre pour différents projets.

#### Liste des versions de Node.js disponibles

Avant d'installer une version spécifique de Node.js, vous pouvez vérifier les versions disponibles en exécutant 

    nvm ls-remote

Cette commande affiche une liste de toutes les versions de Node.js disponibles. Elle vous aide à identifier la version que vous souhaitez installer, comme la dernière version LTS ou un numéro de version spécifique.

Extrait

```
        v20.7.0
        v20.8.0
        v20.8.1
        v20.9.0   (Latest LTS: Iron)
        v21.0.0
        v21.1.0
        v21.2.0
```

#### Installer une version de Node.js via la commande NVM

Une fois le NVM installé, vous pouvez maintenant installer la version souhaitée de Node.js en exécutant la commande suivante :

    nvm install <version>

Remplacez `<version>` par la version spécifique que vous souhaitez installer. Par exemple, si vous souhaitez installer la version 19.9.0 de Node.js, vous devez exécuter la commande suivante

    nvm install 19.9.0

Cette commande téléchargera et installera la version spécifiée de Node.js

#### Vérifier l'installation de Node.js

Pour vérifier la version installée de Node.js, exécutez la commande suivante :

    node -v

Cela affichera la version installée de Node.js sur votre système.

#### Passer d'une version de Node.js à l'autre

Le NVM vous permet de passer facilement d'une version de Node.js à l'autre. Pour passer d'une version de Node.js à l'autre, utilisez la commande suivante :

    nvm use <version>

Remplacez `<version>` par la version vers laquelle vous souhaitez passer. Par exemple, si vous souhaitez passer à la version 18.16.0 de Node.js, vous devez exécuter la commande suivante

    nvm use 18.16.0

Cette commande définira la version spécifiée comme la version active de Node.js pour la session en cours. 

Pour faire d'une version spécifique de Node.js la version par défaut pour les nouvelles sessions de terminal, utilisez la commande 

    nvm alias default <version>

Remplacez `<version>` par le numéro de version souhaité. Par exemple, pour définir la version 18.16.0 de Node.js comme version par défaut, vous devez exécuter la commande suivante :

    nvm alias default 18.16.0

### Commandes supplémentaires de Node.js sur Debian 12, 11 ou 10

#### Supprimer Node.js de Debian

Supprimer Node.js installé à partir du dépôt Debian ou d'un PPA

Si vous avez installé Node.js à partir du dépôt Debian ou d'un PPA, vous pouvez le désinstaller en utilisant le programme apt. Tout d'abord, expliquons la commande :

sudo apt remove nodejs

Cette commande supprimera Node.js ainsi que les fichiers de configuration associés. Elle vous demandera de confirmer la suppression, et après confirmation, elle procédera à la désinstallation.

#### Désinstaller Node.js installé via NVM

Si vous avez installé Node.js à l'aide du gestionnaire de versions de Node (NVM), vous pouvez le désinstaller en suivant les étapes suivantes :

Tout d'abord, vérifiez la version actuelle de Node.js installée en exécutant la commande :

    nvm current

Cette commande affichera la version active de Node.js sur votre système.

Avant de désinstaller la version actuelle de Node.js, vous devez désactiver le NVM en exécutant la commande 

    nvm deactivate

Cette commande déchargera la version active de Node.js de votre session actuelle.

Maintenant, exécutez la commande suivante pour désinstaller une version spécifique de Node.js installée à l'aide du NVM 

    nvm uninstall <version>

Remplacez `<version>` par le numéro de la version que vous souhaitez désinstaller. Par exemple, si vous souhaitez désinstaller la version 19.9.0 de Node.js, vous devez exécuter la commande suivante

    nvm uninstall 19.9.0

Cette commande supprimera la version de Node.js spécifiée de votre système.