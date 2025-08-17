+++
title = 'Mise-a-niveau-Shaarli-Yunohost'
date = 2019-08-23 00:00:00 +0100
categories = application
+++
2017-05-10-Mise-a-niveau-Shaarli-Yunohost
===============================


## Installer PHP Composer sur debian jessie

[How To Install and Use Composer on Debian 8](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-composer-on-debian-8)

Dépendances

    sudo apt-get install curl php5-cli git

Charger et installer **composer**

    php -r "copy('https://getcomposer.org/installer', '/tmp/composer-setup.php');"

Aller sur le site <https://composer.github.io/pubkeys.html> et copier la clé d'installation SHA384  
Coller la clé dans la commande

    php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('/tmp/composer-setup.php'); } echo PHP_EOL;"


Cette commande vérifie le hash du fichier que vous avez téléchargé avec le hash copié sur le site Web de Composer.  
Si correspondance, il va afficher **Installer verified**.  
Si cela ne correspond pas, il va afficher **Installer corrupt**, auquel cas il faut vérifier que vous avez copié la chaîne SHA-384 correctement.  
Ensuite, nous installerons Composer. Pour l'installer globalement sous **/usr/local/bin**, nous utiliserons l'indicateur --install-dir; --filename indique au programme d'installation le nom du fichier exécutable de Composer. Voici comment procéder dans une seule commande:

    sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer

Le message suivant s'affiche

```
All settings correct for using Composer
Downloading...

Composer (version 1.4.1) successfully installed to: /usr/local/bin/composer
Use it: php /usr/local/bin/composer
```

Vérifier 

     composer --version
        Composer version 1.4.1 2017-03-10 09:29:45


## Shaarli/Yunohost

[Sharli community](https://github.com/shaarli)  

**Shaarli** installé sur yunohost **xeuyakzas.xyz**  dans le dossier **/var/www/webapp_xeuyak/site/**  
Se connecter sur le serveur via **ssh**  
Sauvegarder site dans /tmp

    cp -a /var/www/webapp_xeuyak/site /tmp

Télécharger le latest shaarli

```
cd ~
mkdir shaarli
# clone the repository  
git clone https://github.com/shaarli/Shaarli.git -b master shaarli/
# install/update third-party dependencies
cd shaarli/
composer install --no-dev
```

Détails installation

```
Loading composer repositories with package information
Installing dependencies from lock file
Package operations: 11 installs, 0 updates, 0 removals
  - Installing erusev/parsedown (1.6.0): Downloading (100%)         
  - Installing psr/log (1.0.2): Downloading (100%)         
  - Installing pubsubhubbub/publisher (dev-master a5d6a0e): Cloning a5d6a0e1cc
  - Installing katzgrau/klogger (1.2.1): Downloading (100%)         
  - Installing shaarli/netscape-bookmark-parser (v2.0.1): Downloading (100%)         
  - Installing psr/http-message (1.0.1): Downloading (100%)         
  - Installing psr/container (1.0.0): Downloading (100%)         
  - Installing pimple/pimple (v3.0.2): Downloading (100%)         
  - Installing nikic/fast-route (v1.2.0): Downloading (100%)         
  - Installing container-interop/container-interop (1.2.0): Downloading (100%)         
  - Installing slim/slim (3.8.1): Downloading (100%)         
Generating autoload files
```

Mise en place du site **Shaarli**  

    cd ..
    sudo rm -r /var/www/webapp_xeuyak/site/*                        # On efface l'ancien site
    sudo cp -r shaarli/* /var/www/webapp_xeuyak/site/               # on copie le nouveau
    sudo cp -r /tmp/site/data/* /var/www/webapp_xeuyak/site/data/   # restauration des données
    sudo chown www-data.www-data -R /var/www/webapp_xeuyak/site/


