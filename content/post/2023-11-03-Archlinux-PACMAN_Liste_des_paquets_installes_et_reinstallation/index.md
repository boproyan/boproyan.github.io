+++
title = 'PACMAN Créer une liste des paquets installés et les installer plus tard dans Arch Linux'
date = 2023-11-03 00:00:00 +0100
categories = ['archlinux']
+++
## Liste des paquets

### Différentes commandes

*    Liste des paquets installés explicitement: `pacman -Qe` 
*    Liste des paquets d'un groupe de paquets nommé group: `pacman -Sg group`
*    Liste des paquets "étrangers" (typiquement par téléchargement et installation manuels ou paquets retirés des dépôts): `pacman -Qm` 
*    Liste des paquets installés d'origine (depuis la base synchronisée): `pacman -Qn` 
*    Liste des paquets installés explicitement, d'origine (c.-à-d. présents dans la base synchronisée), qui ne sont pas des dépendances directes ni optionnelles: `pacman -Qent` 
*    Liste des paquets filtrée par expression régulière: `pacman -Qs regex` 
*    Liste des paquets filtrée par expression régulière avec sortie en format personnalisé: `expac -s "%-30n %v" regex` (requiert expac).


### Générer liste des paquets explicitement installés

Générons la liste des paquets explicitement installés en utilisant la commande :

    pacman -Qqe > pkglist.txt

Cette commande créera une liste des paquets explicitement installés dans l'ordre alphabétique et les enregistrera dans un fichier texte appelé "pkglist.txt".  

Explications

*    Q - Interroge la base de données des paquets. Cette option vous permet d'afficher les paquets installés et leurs fichiers, ainsi que d'autres méta-informations utiles sur les paquets individuels (dépendances, conflits, date d'installation, date de construction, taille).
*    q - Affiche moins d'informations pour certaines opérations de requête. Ceci est utile lorsque la sortie de pacman est traitée dans un script.
*    e - Liste les paquets explicitement installés qui ne sont requis par aucun autre paquet.
*    pkglist.txt - C'est le fichier de sortie dans lequel vous stockez la liste des fichiers installés.


Enregistrez le fichier "pkglist.txt" sur une clé USB ou dans un endroit sûr.


## Installer des paquets depuis une liste

Maintenant, formater et réinstaller le système. Après avoir réinstallé votre système, copiez le fichier "pkglist.txt" sur votre système nouvellement installé

### Installer tous les paquets depuis une liste

 et exécutez la commande suivante pour installer les paquets de la liste de sauvegarde.

    sudo pacman -S - < pkglist.txt

### Installer depuis une liste sans les paquets étrangers AUR 

Au cas où la liste de sauvegarde inclurait des paquets étrangers, tels que les paquets AUR, supprimez-les d'abord, puis installez le reste des paquets à l'aide de la commande :

    sudo pacman -S $(comm -12 <(pacman -Slq | sort) <(sort pkglist.txt))

La commande ci-dessus supprimera les paquets étrangers. Tapez 'y' et appuyez sur ENTER pour les supprimer. Enfin, tapez 'y' pour installer le reste des paquets de la liste.

Vous n'avez pas besoin d'installer tous les paquets un par un. Pacman lira la liste et installera les paquets qui y figurent.

### Installer les paquets non installés uniquement

Pour installer des paquets depuis une sauvegarde antérieure de la liste des paquets, tout en ne réinstallant pas ceux qui sont déjà installés et à jour, lancer:

    sudo pacman -S --needed - < pkglist.txt

### Supprimer les paquets non mentionnés dans la liste de sauvegarde

Pour supprimer tous les paquets sur votre système nouvellement installé qui ne sont pas mentionnés dans la liste de sauvegarde, exécutez :

    sudo pacman -Rsu $(comm -23 <(pacman -Qq | sort) <(sort pkglist.txt))

