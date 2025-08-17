+++
title = 'Linux commande "du" – Taille d’un répertoire et sous-répertoires'
date = 2021-05-24 00:00:00 +0100
categories = cli
+++
## Linux – Taille d’un répertoire et sous-répertoires, gros fichiers, etc.

Astuces pour récupérer la taille d’un répertoire et ses sous-répertoires

* Taille du répertoire home et de ses sous-répertoires :  
`command du -h --max-depth=1 /home/`  
* Les 10 plus gros fichiers :  
`command du -a /var | sort -nr | head -n 10`
* La même avec une meilleure lisibilité :  
`command du -hax /var | sort -hr | head -n 10`
