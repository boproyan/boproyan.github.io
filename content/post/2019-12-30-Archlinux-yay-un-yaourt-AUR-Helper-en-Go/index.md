+++
title = 'Archlinux "yay" un autre "yaourt" - Un AUR Helper écrit en Go'
date = 2019-12-30 00:00:00 +0100
categories = ['archlinux', 'outils']
+++
## Yay

### Caractéristiques

Yay est basé sur la conception de yaourt , apacman et pacaur . Il est développé avec ces objectifs à l'esprit:

*    Fournir une interface pour pacman
*    Recherche / installation interactive de type Yaourt
*    Dépendances minimales
*    Minimiser les entrées utilisateur
*    Savoir quand les packages git doivent être mis à niveau 
*    Effectuer une résolution avancée des dépendances
*    Téléchargez les PKGBUILD depuis ABS ou AUR
*    Complétez l'onglet AUR
*    Interrogez l'utilisateur à l'avance pour toutes les entrées (avant de commencer les builds)
*    Termes de recherche étroits ( yay linux header linux recherchera d'abord linux puis resserrera sur l'en- header )
*    Trouvez les fournisseurs de packages correspondants pendant la recherche et autorisez la sélection
*    Supprimer les dépendances make à la fin du processus de construction
*    Exécuter sans sourcer PKGBUILD 

### Installation

Si vous migrez à partir d'un autre assistant AUR, vous pouvez simplement installer Yay avec cet assistant.

Alternativement, l'installation initiale de Yay peut être effectuée en clonant le PKGBUILD et en construisant avec makepkg:

    git clone https://aur.archlinux.org/yay.git
    cd yay
    makepkg -si

### Support

Étant donné que Yay n'est pas officiellement pris en charge par Arch Linux, le support ne doit pas être recherché sur les forums, les commentaires AUR ou d'autres canaux officiels.

Un package AUR cassé doit être signalé comme un commentaire sur la page AUR du package. Un paquet ne peut être considéré comme cassé que s'il ne parvient pas à se construire avec makepkg. Les rapports doivent être établis à l'aide de makepkg et inclure la sortie complète ainsi que toute autre information pertinente. Ne faites jamais de rapports en utilisant Yay ou tout autre outil externe.

### Questions fréquemment posées

* Yay n'affiche pas la sortie colorée. Comment je le répare?
    * Assurez-vous que vous disposez de l'option Color dans votre /etc/pacman.conf (voir problème [# 123](https://github.com/Jguer/yay/issues/123) ).
* Yay ne vous invite pas à ignorer les packages lors de la mise à niveau du système.
    * Le comportement par défaut a été modifié après la v8.918 . Pour restaurer le comportement de saut de package, utilisez `--combinedupgrade` (rendez-le permanent en ajoutant `--save` ).  
Remarque: le fait de sauter des packages laissera votre système dans un état [partiellement mis à niveau](https://wiki.archlinux.org/index.php/System_maintenance#Partial_upgrades_are_unsupported)
* Parfois, les différences sont imprimées sur le terminal, et d'autres fois, elles sont paginées via moins. Comment puis-je réparer ça?
    * Yay utilise git diff pour afficher les différences, ce qui par défaut indique moins de ne pas paginer si la sortie peut tenir dans une longueur de terminal. Ce comportement peut être ignoré en exportant vos propres indicateurs ( export LESS=SRX ).
* Yay ne me demande pas de modifier PKGBUILDS, et je n'aime pas le menu diff! Que puis-je faire?
    * `yay --editmenu --nodiffmenu --save`
* Comment puis-je dire à Yay d'agir uniquement sur les packages AUR ou uniquement sur les packages repo?
    * `yay -{OPERATION} --aur yay -{OPERATION} --repo`
* Un message Out Of Date AUR Packages s'affiche. Pourquoi Yay ne les met-il pas à jour?
    * Ce message ne signifie pas que des packages AUR mis à jour sont disponibles. Cela signifie que les packages ont été signalés comme obsolètes sur l'AUR, mais leurs responsables n'ont pas encore mis à jour les PKGBUILD (voir les [packages AUR obsolètes](https://wiki.archlinux.org/index.php/Arch_User_Repository#Foo_in_the_AUR_is_outdated.3B_what_should_I_do.3F) ).
* Yay n'installe pas les dépendances ajoutées à un PKGBUILD lors de l'installation.
    * Yay résout toutes les dépendances à l'avance. Vous êtes libre de modifier le PKGBUILD de quelque manière que ce soit, mais tous les problèmes que vous causez sont les vôtres et ne doivent pas être signalés à moins qu'ils ne puissent être reproduits avec le PKGBUILD d'origine.
Je sais que mon package -git contient des mises à jour mais vous ne proposez pas de le mettre à jour

Yay utilise un cache de hachage pour les packages de développement. Normalement, il est mis à jour à la fin de l'installation du package avec le message Found git repo . Si vous effectuez une transition entre les assistants aur et n'avez pas installé le paquet devel en utilisant yay à un moment donné, il est possible qu'il n'ait jamais été ajouté au cache. `yay -Y --gendb` corrigera la version actuelle de chaque paquet de développement et commencera à vérifier à partir de là.
* Je veux aider!
    * Consultez **CONTRIBUTING.md** pour plus d'informations.

### Exemples d'opérations personnalisées

Commande |	Description
----------|----------------
`yay <Search Term>` |	Présenter le menu de sélection d'installation du package.
`yay -Ps` |	Imprimer les statistiques du système.
`yay -Yc` |	Nettoyez les dépendances inutiles.
`yay -G <AUR Package>` |	Téléchargez PKGBUILD depuis ABS ou AUR.
`yay -Y --gendb` |	Générer une base de données de packages de développement utilisée pour la mise à jour devel.
`yay -Syu --devel --timeupdate` |	Effectuez la mise à niveau du système, mais vérifiez également les mises à jour des packages de développement et utilisez l'heure de modification PKGBUILD (pas le numéro de version) pour déterminer la mise à jour.

