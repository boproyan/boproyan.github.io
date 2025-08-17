+++
title = 'Python - Création d'environnements virtuels VENV et "Wing Personnal" ,installer applis avec "pip"'
date = 2020-04-22 00:00:00 +0100
categories = python
+++
## Python venv

* [Python Doc FR](https://docs.python.org/fr/3.8/)
* [Wing Pro Python IDE Tutorial](https://wingware.com/doc/TOC)
* [Travaillez dans un environnement virtuel](https://openclassrooms.com/fr/courses/4425111-perfectionnez-vous-en-python/4463278-travaillez-dans-un-environnement-virtuel)

*Le module venv permet de créer des "environnements virtuels" légers avec leurs propres dossiers site, optionnellement isolés des dossiers site système. Chaque environnement virtuel a son propre binaire Python (qui correspond à la version du binaire qui a été utilisée pour créer cet environnement) et peut avoir sa propre liste de paquets Python installés dans ses propres dossiers site.*

### Dépendances

Vérifier ou installer python3

    sudo apt install python3 python3-venv

Ensuite, si vous prévoyez d'utiliser Virtualenv, installez-le aussi.

    sudo apt install virtualenv python3-virtualenv

### venv en ligne de commande

La création d'environnements virtuels est faite en exécutant la commande venv (fonctionnalité `venv` de Python 3 est intégrée)

    python3 -m venv /path/to/virtual/environment

Lancer cette commande crée le dossier du venv (en créant tous les dossiers parents qui n'existent pas déjà) et crée un fichier **pyenv.cfg** à l’intérieur de ce dossier avec une clé home qui pointe sur l'installation Python depuis laquelle cette commande a été lancée. Il crée aussi un sous-dossier **bin**  contenant une copie (ou un lien symbolique) du ou des binaire python (dépend de la plateforme et des paramètres donnés à la création de l'environnement). Il crée aussi un sous-dossier (initialement vide) **lib/pythonX.Y/site-packages**. Si un dossier existant est spécifié, il sera réutilisé.

Sur archlinux python version 3 par défaut

    python3 --version   # archlinux : saisir python qui est par défaut la version 3
        Python 3.7.3

Activation

    source /path/to/virtual/environment/bin/activate

Pour avoir le "prompt" suivant

    (environment) [~]$ 

Désactiver l'environnement

    deactivate

### Utiliser Virtualenv

Pour commencer, créez votre environnement avec la commande virtualenv. Vous devrez également lui dire d'utiliser Python 3 avec l'option -p.

    virtualenv -p python3 /path/to/virtual/environment

Cela prendra quelques secondes pour s'installer avec Pip et les autres paquets Python qu'il contient. Quand c'est fini, activez l'environnement.

    source your-project/bin/activate

Faites votre travail dans les répertoires de projets. Lorsque vous avez terminé, utilisez `deactivate` pour quitter l'environnement virtuel.

### "Wing Personnal Python IDE" - Utilisation d'un Virtualenv existant

Pour utiliser un virtualenv existant avec Wing, il suffit de définir l'exécutable Python dans les propriétés du projet de Wing (**Project Properties**)  sur le python dans votre répertoire virtualenv. Wing l'utilise pour déterminer l'environnement à utiliser pour l'analyse des sources et pour exécuter, tester et déboguer votre code.

La façon la plus simple de trouver la valeur correcte à définir est de lancer votre Python virtualenv à l'extérieur de Wing et d'exécuter ce qui suit de manière interactive :

```
(envgpx) [~]$ python
Python 3.7.3 (default, Mar 26 2019, 21:43:19) 
[GCC 8.2.1 20181127] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.executable
'/path/to/virtual/environment/bin/python'
>>> quit()
```

Utilisez le chemin complet qui s'affiche comme exécutable Python dans Wing.


### "Wing Personnal Python IDE" - Créer un nouveau Virtualenv

Si vous démarrez un nouveau projet à partir de zéro et que vous souhaitez créer un nouveau virtualenv pour le projet, sélectionnez Nouveau projet (**New Project**) dans le menu Projet (**Project**) et utilisez le bouton Créer un nouveau type de projet Virtualenv (**Create New Virtualenv**). Vous devrez entrer les valeurs suivantes :

* **Name** est le nom de votre répertoire virtualenv.
* **Parent Directory** est le répertoire où le répertoire virtualenv sera créé.
* **Python Executable** sélectionne l'installation de base de Python à utiliser. En Python 2, vous devez d'abord installer virtualenv dans le Python sélectionné, s'il n'est pas déjà présent.
* **Inherit global site-packages** contrôle si l'option ***--system-site-packages*** doit être utilisée lors de l'exécution de virtualenv. Une fois coché, le virtualenv pourra utiliser les paquets installés dans l'installation de base de Python. Sinon, il sera complètement isolé de l'installation de base, autre que son utilisation des bibliothèques standard de Python.
* **Auto-save project** contrôle si Wing enregistre automatiquement son fichier projet dans le répertoire virtualenv. Lorsqu'il est coché, le projet est nommé en utilisant le **Name** saisi ci-dessus plus**.wpr** et est stocké dans le niveau supérieur du répertoire virtualenv. Dans Wing Pro, qui sépare les données de projet partageables des données spécifiques à l'utilisateur, un deuxième fichier se terminant par**.wpu** sera également écrit.

Après avoir soumis le dialogue Nouveau projet (** New Project**), Wing créera le virtualenv, définira l'exécutable Python dans les propriétés du projet (**Project Properties**) sur le python dans le répertoire virtualenv, et ajoutera le répertoire virtualenv au projet.

Maintenant, l'analyse des sources, l'exécution, le débogage et les tests dans Wing utiliseront le nouveau virtualenv, tant que le projet que vous venez de créer est ouvert. Vous devrez redémarrer l'outil **Python Shell** dans Wing avant qu'il n'utilise le virtualenv nouvellement créé.

## Installation applications python via 'python setup' ou 'pip install'

En apparence, les deux font la même chose : en faisant soit `python setup.py install` ou `pip install <PACKAGE-NAME>` vous installerez votre paquet python pour vous, avec un minimum d'efforts.

Cependant, l'utilisation de **pip** offre quelques avantages supplémentaires qui le rendent beaucoup plus agréable à utiliser.

* **pip** téléchargera automatiquement toutes les dépendances d'un paquet pour vous. En revanche, si vous utilisez **setup.py**, vous devez souvent rechercher et télécharger manuellement les dépendances, ce qui est fastidieux et peut devenir frustrant.
* **pip** garde une trace des différentes métadonnées qui vous permettent de désinstaller et de mettre à jour facilement les paquets avec une seule commande : `pip uninstall <PACKAGE-NAME>` et `pip install --upgrade  <PACKAGE-NAME>`. En revanche, si vous installez un paquet à l'aide de setup.py, vous devez supprimer manuellement un paquet et le maintenir à la main si vous voulez vous en débarrasser, ce qui pourrait être potentiellement source d'erreurs.
* Vous n'avez plus besoin de télécharger manuellement vos fichiers. Si vous utilisez setup.py, vous devez visiter le site web de la bibliothèque, trouver où le télécharger, extraire le fichier, exécuter setup.py... En revanche, pip recherchera automatiquement dans l'index des paquets Python (PyPi) pour voir si le paquet existe à cet endroit, et téléchargera, extraira et installera automatiquement le paquet pour vous. À quelques exceptions près, presque toutes les bibliothèques Python réellement utiles peuvent être trouvées sur PyPi.
* **pip** vous permettra d'[installer facilement des "wheels"](https://stackoverflow.com/q/27885397/646543), ce qui est le nouveau standard de la distribution Python. Plus d'informations sur les roues.
    pip offre des avantages supplémentaires qui s'intègrent bien à l'utilisation de virtualenv, un programme qui vous permet d'exécuter plusieurs projets qui nécessitent des bibliothèques et des versions Python conflictuelles sur votre ordinateur. [Plus d'infos](http://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/).
* <u>pip est livré par défaut</u> avec Python à partir de Python 2.7.9 sur la série Python 2.x, et à partir de <u>Python 3.4.0</u> sur la série Python 3.x, ce qui le rend encore plus facile à utiliser.

>**Donc, en gros, utilisez pip. Il n'offre que des améliorations par rapport à l'utilisation de l'installation de python setup.py.**

Si vous utilisez une ancienne version de Python, que vous ne pouvez pas mettre à jour et que vous n'avez pas installé pip, vous pouvez trouver plus d'informations sur l'installation de pip sur les liens suivants :

* [Official instructions on installing pip for all operating systems](https://pip.pypa.io/en/latest/installing.html)

pip, en soi, ne nécessite pas vraiment de tutoriel. 90% du temps, la seule commande dont vous avez vraiment besoin est `pip install <PACKAGE-NAME>`. Cela dit, si vous souhaitez en savoir plus sur les détails de ce que vous pouvez faire exactement avec pip, voyez :

*    Guide de démarrage rapide ([Quickstart guide](https://pip.pypa.io/en/stable/quickstart.html))
*    Documentation officielle ([Official documentation](https://pip.pypa.io/en/stable/))

**Il est également généralement recommandé d'utiliser pip et virtualenv ensemble**. Si vous êtes un débutant en Python, je pense personnellement qu'il serait bien de commencer par utiliser pip et d'installer les paquets globalement, mais je pense que vous devriez éventuellement passer à l'utilisation de virtualenv au fur et à mesure que vous vous attaquez à des projets plus sérieux.

Si vous souhaitez en savoir plus sur l'utilisation conjointe de pip et de virtualenv, voir :


*    [Why you should be using pip and virtualenv](https://www.davidfischer.name/2010/04/why-you-should-be-using-pip-and-virtualenv/)
*    [A non-magical introduction to Pip and Virtualenv for Python beginners](https://www.davidfischer.name/2010/04/why-you-should-be-using-pip-and-virtualenv/)
*    [Virtual Environments](http://docs.python-guide.org/en/latest/dev/virtualenvs/)
