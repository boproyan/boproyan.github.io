+++
title = 'les-10-paliers-de-liberation-d-un-telephone-android'
date = 2019-10-28 00:00:00 +0100
categories = android
+++
URL:     https://linuxfr.org/news/les-10-paliers-de-liberation-d-un-telephone-android
Title:   Les 10 paliers de libération d’un téléphone Android
Authors: Denis Dordoigne
         Julien Jorge, BAud, patrick_g, gUI, Davy Defaud et ZeroHeure
Date:    2019-10-20T20:45:06+02:00
License: CC by-sa
Tags:    
Score:   29


Android est basé sur un logiciel libre et peut être utilisé avec des logiciels libres. On pourrait définir dix paliers de migration d’un équipement Android vers un équipement plus libre, les premiers sont faciles à passer, les derniers demandent un plus grand engagement, mais on peut s’arrêter au niveau que l’on souhaite après avoir avancé autant qu’on le pouvait.



Nous présenterons dans la suite de cette dépêche chacun de ces dix paliers, comment et pourquoi les atteindre. Le but étant que toute personne intéressée par le Libre sache ce qui est possible avant de décider de son objectif.

----


----

Palier 1 - Se doter d'un magasin de logiciels libres
=================



Le PlayStore n'est pas une application très pratique pour trouver et installer des logiciels libres. Heureusement, il existe une application plus adaptée pour répondre à ce besoin : F-Droid. Pour installer cette application, il suffit de se rendre sur https://www.f-droid.org depuis n'importe quel navigateur installé sur son équipement et de cliquer sur le lien de téléchargement pour obtenir un fichier apk (format de paquets d'Android). En exécutant l'apk, le système demande l'autorisation de procéder à l'installation depuis le navigateur : il est nécessaire de l'accepter (cette autorisation pourra être supprimée par la suite).



Une fois l'installation effectuée, on peut exécuter F-Droid en passant par les menus habituels. En lançant F-Droid la première fois, il est conseillé de personnaliser les dépôts (menu paramètres/dépôts). Une [liste des dépôts disponibles](https://f-droid.org/wiki/page/Known_Repositories) est présente sur le wiki dédié, mais à ce jour seuls [The Guardian project](https://guardianproject.info/) et [IzzyOnDroid](https://android.izzysoft.de/repo/info) semblent être maintenus.



Palier 2 - Installer ses premières applications
=================
Naviguer dans F-Droid permet de découvrir de nombreux logiciels intéressants. Lors de la première installation d'une application, il faut accorder l'autorisation de procéder à l'installation depuis F-Droid (ce ne sera demandé qu'une fois). Voici une liste toute subjective d'applications recommandées :



- [AnySoftKeyboard](https://f-droid.org/fr/packages/com.anysoftkeyboard.languagepack.french) : ce clavier remplace celui fourni par défaut avec le système, mais donne aussi un accès beaucoup plus pratique aux caractères spéciaux, et surtout contient des flèches gauche/droite pour pouvoir arriver au bon endroit d'un texte sans avoir à viser précisément avec son doigt.
- [Etar](https://f-droid.org/fr/packages/ws.xsoh.etar/) est tout simplement le calendrier le mieux conçu pour Android, même la vue mensuelle est lisible !
- [Fennec F-Droid](https://f-droid.org/fr/packages/org.mozilla.fennec_fdroid/) est un navigateur libre dérivé de Firefox (pleinement compatible avec Firefox sync et les extensions Firefox).
- [Street Complete](https://f-droid.org/fr/packages/de.westnordost.streetcomplete/) qui a joliment grandi depuis [la dépêche y faisant référence](https://linuxfr.org/news/streetcomplete-jouez-a-completer-openstreetmap).
- Pour les personnes s'intéressant à la sécurité, on peut regarder les applications du [Guardian Project](https://guardianproject.info/)



On ne va pas tout de suite installer une grande quantité d'applications, il s'agit simplement de prendre l'outil en main : profitons de l'élan pris par ces réussites pour passer au palier suivant.



Palier 3 - Identifier les applications inutilisées
=================



Il y a forcément des applications qui ne servent à rien installées sur son équipement, dont celles pré-installées par son opérateur (TV d'Orange, SFR jeux, Mon compte Free, etc.) ou le constructeur (Samsung apps, Huawei AppGallery, etc.). Pour passer ce palier, il faut d'abord prendre un papier et un crayon et lister toutes les applications installées sur l'équipement, sous la forme de deux colonnes : l'une contenant le nom de l'application, l'autre un commentaire à venir. Cela va probablement être long, voire très long, mais prendre connaissance de tout le capharnaüm privateur de son téléphone est nécessaire pour pouvoir passer les paliers suivants…



Une fois cette liste établie, il devrait être facile d'y retrouver les applications pré-installées dont on n'a pas l'usage, celles qui ont été installées mais n'ont servi qu'une seule fois, ainsi que d'autres traces de l'histoire de l'équipement qu'il n'est pas très utile de conserver (il sera toujours possible de faire encadrer la liste pour se souvenir de ces applications courageusement abandonnées). Pour chacune des applications concernées, inscrire dans la seconde colonne « à oublier ».



![Exemple de liste contenant une entrée « à oublier »](http://denis.infini.fr/dlfp/liste_android.png)



Palier 4 - Supprimer ses premières applications privatrices
===========================================================
Se rendre la liste des applications de son téléphone depuis le menu de configuration et tenter dans l'ordre de :



1. cliquer sur l'application, et si le bouton « désinstaller » est présent, cliquer dessus
1. s'il n'y a pas de bouton « désinstaller » mais qu'il existe un bouton « désactiver », cliquer dessus
1. si après avoir cliqué sur « désactiver », un message indique que seules les mises à jour vont être supprimées, l'application restera présente dans les menus : pour certaines versions d'Android, il est possible de regrouper plusieurs applications dans un même dossier, faute de mieux pour ce palier-ci on se contentera de mettre toutes les applications dont on ne veut plus dans un dossier « poubelle » 



![Extrait d'un menu d'applications contenant un dossier poubelle](http://denis.infini.fr/dlfp/android_poubelle.png)



Palier 5 - Identifier les phéromones numériques
=================



Tel un chat venant se frotter contre une jambe pour y déposer ses phéromones, certains médias et acteurs du numérique veulent marquer leur territoire dans les équipements Android en y apposant leur logo, sans offrir un quelconque avantage à l'usager. Pour identifier les applications concernées, il faut passer son équipement en mode avion puis lancer chacune des applications non annotées de sa liste, afin de repérer celles qui refusent de se lancer ou ne fonctionnent plus. Pour chacune de ces applications, se poser la question suivante : « mais que peut bien m'offrir cette application que je ne trouverais pas sur le web ? ». Il y a plusieurs réponses possibles :


- **le contenu est plus lisible** (applications de médias par exemple) : il faudrait essayer d'accéder au site web depuis fennec F-Droid et vérifier si c'est vraiment le cas, parce que la plupart des sites web grand public sont lisibles en version mobile ; il est aussi possible de [passer en mode lecture](https://support.mozilla.org/fr/kb/activer-mode-lecture-firefox-android) pour vérifier si cette vue-ci n'est pas encore plus lisible que l'application, une autre possibilité est de tenter le passage [en version non-mobile](https://support.mozilla.org/fr/kb/passer-version-ordinateur-firefox-focus-android) pour voir le rendu



- **j'ai mon compte enregistré** (applications de vente en ligne par exemple) : on peut penser qu'il est dérangeant que toute personne utilisant le téléphone ait accès à des applications offrant un accès direct à ses comptes clients mais si on n'a pas peur de cette éventualité, pourquoi ne pas tout simplement enregistrer son mot de passe dans le navigateur et utiliser le site web directement ? En plus, si vraiment on n'aime pas taper ses mots de passe, on peut demander à Firefox de  [synchroniser les mots de passe sur tous ses équipements](https://support.mozilla.org/fr/kb/configurer-firefox-sync?redirectlocale=fr&redirectslug=comment-configurer-firefox-sync)



- **ça me donne accès à des fonctionnalités spécifiques** : c'est l'occasion de se demander si celles-ci sont utiles, si on apprécie vraiment de recevoir toutes ces notifications, s'il est nécessaire que tous nos contacts puissent tracer nos promenades, etc. (pour rappel, la réponse peut être « oui », passer ce palier ne nécessite pas de renoncer à ses usages)



Pour chacune des applications ne servant qu'à marquer le territoire, ajouter un simple favori dans son navigateur, procéder à sa suppression autant que possible, puis inscrire dans la seconde colonne de la liste « nettoyé ».



Palier 6 - Remplacer les applications privatrices
=================



Ce palier peut sembler évident à passer après avoir installé F-Droid, cependant il est recommandé de ne le passer qu'à ce stade pour deux raisons :



- il serait dommage de passer du temps à remplacer des applications pour se rendre compte que, finalement, on n'en a pas besoin
- le passage des paliers précédents permet d'acquérir une certaine maîtrise dans l'identification de ses besoins.



L'objectif de ce palier est de trouver un maximum d'applications libres pouvant satisfaire les besoins auxquels répondent les applications privatrices. On va donc encore passer notre liste en revue, et pour chaque logiciel qui n'a toujours pas d'annotation il va falloir identifier les besoins auxquels celui-ci répond, puis pour chacun de ces besoins chercher si, pour satisfaire le besoin, il existe :



 1. **une extension Firefox libre** : la plupart des extensions Firefox sont sous licence libre, et la plupart des applications n'utilisent que des fonctionnalités du web : sans surprise, beaucoup d'applications Android ont leur équivalent libre en extension Firefox
 1. **une application libre** : les catégories et le moteur de recherche de F-Droid peuvent aider, on peut aussi consulter les sites https://alternativeto.net/platform/android/?license=opensource et https://fsfe.org/campaigns/android/liberate.fr.html ; toutes les applications proposées par F-Droid sont libres
 1. **une extension Firefox non libre** : si on n'a pas trouvé d'application libre ou une extension Firefox libre, il est préférable d'utiliser une extension non libre à une application privative parce que le code source est ouvert (ce qui offre certains avantages du logiciel libre) et que cela peut permettre d'éviter une dépendance à l'écosystème privateur de Google (certaines applications interagissant avec Google sans que cela soit très transparent)

Bien entendu, on peut maintenant supprimer autant que possible les applications devenues inutiles, puis ajouter la mention « remplacé » dans la fameuse deuxième colonne de la liste qui doit sembler de moins en moins vide.



Palier 7 - Apprendre à exporter ses données
=================
Il ne s'agit pas ici de faire la morale aux personnes ne procédant pas à de fréquentes sauvegardes, mais pour passer les prochains paliers, il est nécessaire de savoir exporter et importer ses données. Il faut tout d'abord savoir qu'il n'y a pas de méthode universelle pour procéder aux sauvegardes, celles-ci dépendent totalement des logiciels et services utilisés. Quelques idées pour procéder à des sauvegardes :



- utiliser les fonctionnalités d'export des applications : la plupart des applications permettent d'enregistrer leurs données dans un fichier externe, si on souhaite continuer à utiliser ces applications, cela peut être suffisant ;
- utiliser une ou plusieurs applications dédiées à la sauvegarde ;
- exporter les données dans un [[format ouvert]], notamment ses contacts (format [[VCard]]), ses agendas (format [[iCalendar]]), ses podcasts et flux RSS (format [[OPML]]), etc. ; si les données sont synchronisées chez Google ou un autre fournisseur, il peut être plus facile de faire cet export [depuis le site web](https://takeout.google.com) du service que depuis l'application mobile.


Palier 8 - Déménager son _cloud_
===========================
Souvent lorsque l'on crée son calendrier ou ses contacts sur son équipement Android, ceux-ci se trouvent être synchronisés dans le _cloud_ de Google, du constructeur ou de l'opérateur. Le but de ce palier est de les synchroniser en lieu sûr. Franchir ce palier n'est pas difficile techniquement, mais cela nécessite une certaine intrépidité : après avoir passé avec succès tous les paliers précédents, il n'y a aucune raison d'avoir peur.


Aller sur https://www.chatons.org pour découvrir les hébergeurs promouvant et utilisant des logiciels libres. Certains proposent de synchroniser ses agendas, calendriers, tâches etc. dans leur infrastructure (avec [[Nextcloud]] ou [[Owncloud]]), parmi les plus gros on citera [la mère Zaclys](https://www.zaclys.com/), [Marsnet](http://www.marsnet.org/) et [Infini](https://www.infini.fr). Comparer les offres (notamment en termes de limites, le stockage quasi-illimité n'existe pas chez les petits hébergeurs indépendants), et choisir celle qui répond le plus à ses besoins. Note : c'est peut être aussi le moment de se demander s'il est bien nécessaire de tout synchroniser sur Internet.



Exporter une dernière fois, dans un format ouvert, les données actuellement synchronisées, puis les importer depuis l'interface Nextcloud du nouvel hébergeur. Supprimer les comptes devenus obsolètes dans le téléphone (menu « utilisateurs et comptes » dans les préférences - on peut aussi choisir pour chaque compte ce qui doit être synchronisé, si on veut conserver son compte Google pour les autres usages que les contacts et le calendrier). Depuis F-Droid, installer DAVx, l'exécuter afin de configurer son compte.


Il reste maintenant à installer les applications permettant de synchroniser ses différents besoins (bien entendu depuis F-Droid, ces applications existent également dans le PlayStore mais y sont payantes) : [les différentes applications Nextcloud](https://search.f-droid.org/?q=nextcloud&lang=fr), [Tasks](https://f-droid.org/fr/packages/org.dmfs.tasks/), etc.


Palier 9 - Sélectionner son équipement
=========================
Les plus beaux paliers sont déjà passés et à partir de maintenant on va procéder à des actions partiellement irréversibles : si on ne souhaite pas ou ne peut pas aller plus loin, quelle qu'en soit la raison, on peut déjà être satisfait du chemin parcouru, et pourquoi pas envisager d'aller plus loin lorsqu'on changera d'équipement.


L'objectif à partir d'ici va être de remplacer Android par un système plus libre : on prendra pour exemple [[LineageOS]] qui semble être le plus utilisé des dérivés de la version libre d'Android 



Mon équipement est-il compatible ?
----------------------------------
Pour savoir si son équipement est compatible avec LineageOS, il existe une page dans un wiki tentant de répertorier tous les équipements compatibles : https://wiki.lineageos.org/devices/ ; il faut faire attention, certains équipements verrouillent le système d'amorçage sans méthode officielle (ou amateur mais simple) pour le déverrouiller… Si on ne se sent pas une âme de hacker, et/ou qu'on craint de tout bloquer, il est prudent de ne pas tenter l'aventure. Afin d'avoir de vrais retours d'expérience d'installation sur un équipement précis, on peut faire une recherche dans le [Reddit de LineageOS](https://www.reddit.com/r/LineageOS).


Comment choisir un nouvel équipement ?
--------------------------------------



### Méthode simple
On peut trouver des équipements avec un système LineageOS pré-installé, surtout en occasion. On peut aussi choisir un équipement étant reconnu pour être particulièrement adapté à l'installation de LineageOS : https://www.androidpolice.com/2019/07/05/guide-best-phones-tablets-lineageos/



### Méthode structurée
Définir soigneusement ses besoins, classer une liste d'équipements selon leur réponse aux besoins, et choisir le premier de la liste qui sera compatible avec LineageOS



### Méthode intelligente
Prendre en compte dans les besoins la réparabilité de l'équipement en vérifiant celle-ci sur le site [iFixit](https://www.ifixit.com/). L'équipement restera utilisable bien après l'arrêt de sa prise en charge par le constructeur et l'opérateur, il est _a priori_ pertinent de s'assurer qu'on ne devra pas le jeter parce que la batterie a vieilli ! Il faut regarder tout ce qui est facilement remplaçable ou réparable dans l'équipement choisi. Il faut bien faire attention à vérifier la difficulté indiquée pour chaque tutoriel, on a déjà vu des équipements mourir suite à des maltraitances de personnes ayant surestimé leurs compétences en bricolage !



Palier 10 - Installer LineageOS
=============================
Il faut d'abord installer fastboot et adb sur son ordinateur (ces applications sont empaquetées dans la plupart des distributions). Ensuite il faudra changer le bootloader, re-partitionner, installer... mais tout ça se fait en suivant tout simplement en suivant le [guide d'installation](https://wiki.lineageos.org/install_guides.html) de son équipement.


![Logo de LineageOS](https://upload.wikimedia.org/wikipedia/commons/thumb/f/f3/LineageOS_Logo.svg/240px-LineageOS_Logo.svg.png)



On peut réinstaller toutes les applications libres découvertes en passant les paliers précédents. En cas de besoin de conserver des applications privatrices, il faut installer [Aurora store](https://f-droid.org/fr/packages/com.aurora.store/) ou [ApkTrack](https://f-droid.org/fr/packages/fr.kwiatkowski.ApkTrack/) pour pouvoir les récupérer depuis le PlayStore. Si certaines de ces applications refusent de fonctionner à cause de dépendances à des outils Google, il est possible d'installer tout ou partie de ceux-ci : https://wiki.lineageos.org/gapps.html


Bonus : partager, sensibiliser, accompagner
=========================================
On est maintenant possesseur d'un équipement aussi libre qu'on le pouvait, il faut partager cela avec son entourage :



-  proposer des logiciels libres : lorsqu'on est en contact avec une personne ayant un besoin, l'orienter vers une application F-Droid qui y répond ;
- sensibiliser les personnes pouvant être intéressées : expliquer qu'il n'est pas nécessaire de tout faire tout de suite, que l'on peut procéder palier par palier, et montrer sa liste !
- si une personne veut passer les mêmes paliers que les siens (ou même aller plus loin), proposer de l'accompagner ou d'être disponible en soutien si besoin ;
- en compagnie de militants du libre n'ayant même pas passé le premier palier, frimer !
