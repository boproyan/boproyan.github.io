+++
title = 'Installation Simplifiée Jekyll (générateur de site statique) sur Linux'
date = 2024-03-24 00:00:00 +0100
categories = jekyll
+++
*Jekyll  est un générateur de sites statiques (Static Site Generators - SSG) open source gratuit qui s’appuie sur le langage de programmation Ruby.*

## Jekyll

![](jekyll-300x133.png)  
[Tutoriel Jekyll : Comment créer un site web statique](https://kinsta.com/fr/blog/site-statique-jekyll/) 

### Installer les prérequis


Fedora

    sudo dnf install ruby ruby-devel openssl-devel redhat-rpm-config gcc-c++ @development-tools

RHEL8/CentOS8

    sudo dnf install ruby ruby-devel
    sudo dnf group install "Development Tools"

Debian

    sudo apt install ruby-full build-essential

Gentoo

    sudo emerge -av jekyll

ArchLinux

    sudo pacman -S ruby base-devel

OpenSUSE

    sudo zypper install -t pattern devel_ruby devel_C_C++
    sudo zypper install ruby-devel

### Installer Jekyll

![](jekyll-300x133.png){:height="30"}

Installez Ruby ![](ruby-logo.png){:height="20"} et les autres prérequis 

    sudo apt install ruby-full build-essential zlib1g-dev

Évitez d'installer les paquets RubyGems (appelés gems) en tant qu'utilisateur root. A la place, mettez en place un répertoire d'installation des gems pour votre compte utilisateur. Les commandes suivantes ajouteront des variables d'environnement à votre fichier ~/.bashrc pour configurer le chemin d'installation des gems 

```bash
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Enfin, installez Jekyll et Bundler :

    gem install jekyll bundler

Vérification

    jekyll -v

jekyll 4.3.3

Voilà, c'est fait ! Vous êtes prêt à utiliser Jekyll.
