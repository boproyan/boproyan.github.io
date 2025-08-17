+++
title = 'Debian Python version 3 par défaut'
date = 2021-09-24 00:00:00 +0100
categories = python
+++
## Python version 3 par défaut

![python](python-logo.png){:width="50"}

Pour changer la version de python à l’échelle du système, nous allons utiliser la commande `update-alternatives` en tant qu’utilisateur root.  
Pour visualiser toutes les alternatives disponibles de python :

    update-alternatives --list python

*update-alternatives: error: no alternatives for python*

Le message d’erreur ci-dessus indique qu’aucune alternative de python n’a été reconnue par update-alternatives.  

Liste des versions installées

    ls -l /usr/bin/python*

```
lrwxrwxrwx 1 root root       7 Mar  4  2019 /usr/bin/python -> python2
lrwxrwxrwx 1 root root       9 Mar  4  2019 /usr/bin/python2 -> python2.7
-rwxr-xr-x 1 root root 3689352 Oct 11  2019 /usr/bin/python2.7
lrwxrwxrwx 1 root root       9 Mar 26  2019 /usr/bin/python3 -> python3.7
-rwxr-xr-x 2 root root 4877888 Jan 22  2021 /usr/bin/python3.7
```

Pour cette raison, nous devons mettre à jour notre tableau des alternatives et inclure à la fois python2.7 et python3.7 :

    sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1

*update-alternatives: using /usr/bin/python2.7 to provide /usr/bin/python (python) in auto mode*

    sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.7 2

*update-alternatives: using /usr/bin/python3.7 to provide /usr/bin/python (python) in auto mode*

L’option --install prend plusieurs arguments à partir desquels il sera capable de créer un lien symbolique. Le dernier argument spécifié défini la priorité, si aucune sélection manuelle alternative n’est donnée, l’option avec le numéro de priorité le plus élevé sera exécutée. Dans notre cas, nous avons défini une priorité 2 pour /usr/bin/python3.7 et de ce fait /usr/bin/python3.7 a été définie comme la version python par défaut par update-alternatives.

    python --version

*Python 3.7.3*

Installer pip3

    sudo apt install python3-venv python3-pip
