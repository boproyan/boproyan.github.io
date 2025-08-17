+++
title = 'intel-14-nm-amd-7-nm-arm-7-nm-et-mon-serveur'
date = 2019-12-04 00:00:00 +0100
categories = divers
+++
URL:     https://linuxfr.org/users/oliver_h/journaux/intel-14-nm-amd-7-nm-arm-7-nm-et-mon-serveur
Title:   Intel = 14 nm, AMD = 7 nm, ARM = 7 nm… et mon serveur ?
Authors: Oliver
Date:    2019-12-03T23:49:17+01:00
License: CC by-sa
Tags:    intel, amd, amd64, arm et serveur
Score:   24


Je me lance dans un projet personnel qui nécessite un serveur pour lequel je viens de faire le tour des actualités. Je vous partage ici mes découvertes et mes réflexions. Bonne lecture.

^(^Je ^place ^ce ^document ^sous ^licence ^CC0.)

# Intel = 14 nm

Intel avait l’habitude d’adopter une nouvelle finesse de gravure tous les deux ans. Et avec chaque nouvelle finesse de gravure, les nouveaux processeurs étaient toujours plus véloces :

* 45 nm en novembre 2007 (Penryn)
* 32 nm en janvier 2010 (Westmere, 2 ans après)
* 22 nm en avril 2012 (Ivy Bridge, 2 ans après)
* 14 nm en septembre 2014 (Broadwell, 2 ans après)
* 10 nm avec deux tentatives pas très concluantes au niveau des performances en 2018 (Cannon Lake) et 2019 (Ice Lake)

Ou là, Intel a du mal à adopter pleinement le 10 nm… Espérons que le troisième rafraîchissement *(optimization)* de la micro-architecture en 10 nm (10 nm++ [Tiger Lake](https://en.wikipedia.org/wiki/Tiger_Lake_(microarchitecture)) prévu pour 2020) sera plus véloce que le quatrième rafraîchissement de celle en 14 nm (14 nm+++ [Comet Lake](https://en.wikipedia.org/wiki/Comet_Lake)).

Par rapport à la régularité du cycle biennal, Intel stagne 6 années sur la gravure 14 nm, et peut-être plus longtemps si on en croit des rumeurs tablant sur [2022 pour voir enfin arriver des processeurs Intel 10 nm dans les ordinateurs fixes](https://www.tweaktown.com/news/68127/intel-cancels-10 nm-desktop-14 nm-hold-until-2022/index.html).

D’ailleurs, ces difficultés de passer à la gravure 10 nm ont plutôt incité Intel à temporiser la pleine production en 10 nm. Par conséquent, Intel a plutôt intérêt à s’orienter vers des prévisions de vente pessimistes. Mais la conjoncture économique est au renouvellement des ordinateurs. Et fait rare, [Intel s’excuse pour ses retards de livraison](https://www.zdnet.fr/actualites/intel-s-excuse-pour-ses-retards-de-livraison-et-pointe-une-demande-excessive-de-cpu-39894299.htm). Face à la demande, Intel favorise les deux segments qui rapportent le plus : les Xeon haut de gamme (gros serveurs) et les i9 ([Gamer](https://fr.wikipedia.org/wiki/Gamer)).

Attention, ne vous faites pas avoir, la dernière et 10ᵉ génération des processeurs Intel est en double gravure 14 nm et 10 nm : la publicité de Intel met en avant la finesse de gravure 10 nm, mais leurs tests de vélocité utilisent des processeurs gravés en 14 nm.

Ce qui me choque le plus est la compétitivité de mon vieil ordinateur portable acheté en 2012, chez [pcubuntoo](https://pcubuntoo.fr/) avec un processeur i7-3610QM (22 nm Ivy Bridge) et 8 Go DDR3, à 800 € sans disque dur. Ce bon vieux portable (7 ans) fonctionne toujours aussi bien et me semble dans la même gamme de prix que les portables équipés du i7-1060G7 (toujours le même nombre de cœurs et la même quantité de mémoire).

Néanmoins, au niveau consommation électrique et fréquence de fonctionnement, le dernier i7-1060G7 doit battre mon vieux i7-3610QM (2012). J’ai bien l’impression que c’est seulement sur ces deux aspects que les processeurs de Intel se sont améliorés : consommation et fréquence. Rappelons que Intel a raté le marché des téléphones, et que depuis une dizaine d’années, Intel essaye de produire des processeurs capables de rivaliser avec ceux utilisés dans les téléphones. Une des priorités de Intel est la réduction de la consommation électrique.

Une autre amélioration des processeurs Intel est le paquet de nouvelles instructions. Mais bon, la plupart des logiciels sont compilés avec la compatibilité des premiers processeurs [AMD64](https://help.ubuntu.com/18.04/installation-guide/amd64/ch02s01.html) ([Pentium II](https://fedoraproject.org/wiki/Architectures/x86#Supported_Hardware)) de la fin des années 90 (il y a 20 ans).

Notons que Intel a changé sa [stratégie *tic-tac*](https://fr.wikipedia.org/wiki/Intel#Stratégies_tic-tac_et_processus-architecture-optimisation). Avec la nouvelle stratégie *processus-architecture-optimisation*, Intel profite de la réduction de la taille de gravure pour réduire la consommation électrique. Puis, Intel apporte des rafraîchissements *(optimizations)* pour améliorer la vélocité.

Bref, tout ça pour dire, que j’ai du mal à justifier le renouvellement de mon vieil ordinateur, d’autant plus que la mode est à la réutilisation, l’économie circulaire. À la rigueur, on peut installer un nouveau SSD plus volumineux en espérant pouvoir stocker toutes les vidéos que nous avons tendance à capturer avec nos ordiphones ! Mais on touche un autre sujet qui nécessite un article à part entière : comment bien stocker et protéger son patrimoine numérique avec les logiciels libres d’aujourd’hui.

Pour ceux qui sont dans mon cas, on trouve des SSD SATA à des prix accessibles : 60 € pour 500 Go, 110 € pour 1 To et 230 € pour 2 To.


# AMD = 7 nm

Alors que Intel essaye de passer à 10 nm, AMD est déjà loin avec la gravure en 7 nm. J’invite les experts qui pensent que le 10 nm de Intel correspond au 7 nm de AMD à nous donner des explications dans les commentaires (j’ai la flegme de vérifier la taille des transistors dans la documentation de chacun des fondeurs).

Je n’aime pas les situations de quasi-monopôle (Intel, Microsoft, Google, YouTube…) et j’essaye de ne pas cautionner ces monopôles et choisissant des alternatives. Mais bon, reconnaissons que Intel contribue au noyau Linux et à la pile graphique, ce qui garantie une certaine compatibilité avec leurs processeurs. AMD y contribue moins, favorise davantage le segment Windows… allez donnons lui une chance dans ce journal.

Pour avancer rapidement, AMD fait fondre ses processeurs par des sous-traitants. Et d’ici que Intel livrent enfin des processeurs 10 nm pour les ordinateurs fixes, AMD en sera peut-être à l’étape 5 nm, car son sous-traitant [TSMC est déjà prêt pour la production en 5 nm](https://en.wikipedia.org/wiki/5_nanometer#5_nm_process_nodes).

De plus, AMD n’essaye pas non plus de rivaliser avec la faible consommation électrique des processeurs de nos téléphones portables, une contrainte de moins que Intel. Le vrai débouché des processeurs x86 sont les serveurs, et ceux qui achètent les serveurs ne sont plus les grandes entreprises, mais les gestionnaires des centres de données *(datacenters)* : Amazon, Google, Microsoft… Historiquement HP et Dell étaient les plus gros acheteurs de processeurs pour serveur, mais les *cloud providers* ont leurs équipes dédiées pour la fabrication de leur matériel.

Les deux segments prioritaires de AMD ont été les *cloud providers* et les *gamers*. Donc, en plus de proposer des processeurs en 7 nm, AMD propose aussi davantage de cœurs. Sa micro-architecture Zen (Zen+, Zen 2, Zen 3) est utilisée pour ses processeurs EPYC et Ryzen. Ces deux gammes de processeurs, peuvent embarquer jusqu’à 64 cœurs chacun !

Par conséquent, les processeurs AMD connaissent de plus en plus de succès. Le fait de passer par un fondeur multi-clients, AMD peut plus facilement d’adapter la production à la demande. Récemment, des [japonais faisaient la queue avant l’ouverture des boutiques pour acquérir le dernier processeur AMD Ryzen 9](https://wccftech.com/amd-ryzen-9-3950x-entire-inventory-outsold-japan-worldwide/).

Sur le lien précédent, deux images sont intéressantes :

## Le nombre de processeurs vendus par AMD et Intel sur 1 an

Nous pouvons constater une envolée des ventes de AMD depuis cet été, alors que pour Intel ça stagne.

![Le nombre de processeurs vendus par AMD et Intel sur 1 an](https://cdn.wccftech.com/wp-content/uploads/2019/12/AMD-Mindfactory-Market-Share-November-2019-mMr8FET-1480x699.png)


## Le chiffre d’affaires

Nous pouvons remarquer que le chiffre d’affaires de AMD est moindre pour le même volume de processeurs vendus. Certainement car AMD vend ses processeurs moins chers que Intel.

![Le chiffre d’affaires de processeurs vendus par AMD et Intel sur 1 an](https://cdn.wccftech.com/wp-content/uploads/2019/12/AMD-Mindfactory-Market-Share-November-2019-G1UqpYZ-1480x699.png)

## Exemples de serveurs AMD


### 800 € = Ryzen 7 2700 (8 cœurs) + HDD 2 To

Configuration obtenue sur le site LDLC en optant pour le Ryzen 7 2700, un processeur d’ancienne génération (Zen+, 2018), qui se trouve moins cher que les Ryzen 5 de nouvelle génération (Zen 2, 2019). Le Ryzen 7 2700 permet d’obtenir un petit ratio **55 €/cœur** (coût par cœur sans compter RAM, SSD et HDD).

|Prix | Description
|------:|-------------
| 210 € | AMD Ryzen 7 2700 (8 cœurs 3.2 GHz)
| 210 € | 32 Go DDR4 2933 MHz
| 80 € | SSD 500 Go M.2 PCI x4
| 70 € | HDD 2 To
| 100 € | Alimentation 80PLUS Platinium 550 W
| 20 € | Ventirad d’entrée de gamme (pas besoin de lumière)
| 80 € | Carte mère d’entrée de gamme avec 4 *slots* mémoire
| 30 € | Boîtier mini tour d’entrée de gamme

### 2500 $ = Ryzem 9 3900X (12 cœurs) + HDD 20 To

Chez [System76](https://system76.com/desktops/thelio-r1/configure) :

* Ryzem 9 3900X (12 cœurs 3.8 GHz)
* 32 Go DDR4
* SSD 500 Go M.2 PCI express
* HDD 20 To

J’estime le ratio à **100 $/cœur** (sans prendre en compte les prix supposés des RAM, SSD et HDD).

# Serveurs ARM

Depuis 20 ans, des constructeurs tentent de concurrencer les serveurs basés sur les processeurs x86 (AMD et Intel). Nous avions les PowerPC, mais quand Apple a abandonné les PowerPC pour les Duo-cœurs d’Intel, l’industrie s’est aussi orientée vers les processeurs Intel. La solution ARM devenaient la seule alternative crédible face au monopôle des x86, grâce à un prix de vente et une consommation électrique très faibles.

Mais il ne suffit pas de réduire le coût d’achat et le coût d’utilisation (factures électriques), il faut surtout avoir un haut ratio `vitesse d’exécution / coût de possession`. Et sur ce point, la fabrication en masse de composant pour l’écosystème x86 permettent aux constructeurs de serveurs x86 de proposer des solutions suffisamment performantes pour ne pas se faire trop distancer par les serveurs ARM qui n’atteignaient pas la taille critique.

Un autre élément en faveur des serveurs x86 qui est bien plus important : la rétro-compatibilité avec les anciens logiciels. Et oui, on migre pas d’un claquement de doigt les logiciels (dont le système d’exploitation) de l’architecture x86 vers ARM. C’est un argument similaire qui laisse [Linus Torvalds penser à l’échec des serveurs ARM](https://www.extremetech.com/computing/286311-linus-torvalds-claims-arm-wont-win-in-the-server-space) : Ce n’est pas facile de développer/déboguer sur sa machine de dév. x86 et déployer sur une machine ARM.

Par contre, quand nous commençons un tout nouveau projet, on peut dès le début tout mettre en place pour faciliter le déploiement sur une architecture ARM.

C’est ce que fait Amazon depuis un an avec ses processeurs ARM, nommés Graviton. Et la [seconde génération Graviton est en cours de genèse](https://siliconangle.com/2019/11/28/report-aws-developing-new-graviton-chip-32-cœurs-20-speed/) qui embarquent 32 cœurs. Amazon a la taille critique pour produire suffisamment de processeurs Graviton afin d’obtenir des processeurs ARM deux fois moins coûteux que les équivalents chez AMD et Intel. Par contre, les meilleurs processeurs Intel et AMD restent plus véloces que les Graviton (pour un même nombre de cœurs).

Et c’est possible de s’en procurrer. \o/

System76 propose le [Starling Pro ARM](https://system76.com/servers/stap1b/configure).
Une configuration avec 96 cœurs et 128 Go RAM coûte dans les 6000 $ pour un ratio **65 $/cœur**.

[Gigabyte propose aussi des serveurs ARM](https://www.gigabyte.com/fr/ARM-Server/Marvell-ThunderX), dénommés *“blockchain infrastructure”*.


# Serveurs RISC-V

Le top du top, ce serait, quand même, d’utiliser des processeurs RISC-V sous licence *open hardware*. Peut-être dans 10 ans…


# Projet personnel nécessitant un *gros* serveur

Dans mon cas, je commence un projet personnel et j’ai besoin de développer, tester et laisser tourner mon logiciel plusieurs jours d’affilée, voir plusieurs mois, avec haute disponibilité et mise à jour à chaud de nouvelles versions.

J’ai cinq possibilités pour mon serveur :

1. Louer une machine virtuelle dans les nuages ;
2. Louer une machine dédiée dans un centre de données *(datacenter)* ;
3. Acheter un serveur d’occasion ;
4. Acheter son propre serveur neuf ;
5. Utiliser son ordinateur personnel.

## La location

La première possibilité *(cloud)* est intéressante pour une courte durée (de quelques heures à quelques jours).
La seconde est celle d’une machine dédiée, c’est-à-dire d’une machine physique. Cette possibilité est plus avantageuse si la machine est utilisée plusieurs mois.

Exemple de prix pour une machine avec SSD 2 To :

1. 1112 €/mois pour AWS m5a.8xlarge (16 cœurs 2,5 GHz + RAM 128 Go)
2. &nbsp; 235 €/mois pour Online Core-5-S (20 cœurs 2,2 GHz + RAM 128 Go)

Pour 6 mois d’utilisation, nous avons les ratios suivant :

1. 417 €/cœur
2. &nbsp; 70 €/cœur

Personnellement, je ne suis pas (encore) à l’aise à déboguer sur un serveur distant, je préfère avoir la machine physiquement à côté de moi. Apparemment, [Facebook contribue à Visual Studio Code](https://developers.facebook.com/blog/post/2019/11/19/facebook-microsoft-partnering-remote-development/?_fb_noscript=1) afin de permettre de travailler à distance. Si vous le faite déjà, je veux bien avoir vos conseils dans les commentaires, merci.

## Serveur d’occasion

Les petites annonces, comme sur leboncoin par exemple, proposent des serveurs d’occasion, souvent vieux de plus de 5 ans, mais à des tarifs qui peuvent être intéressants. On peut même y trouver des serveurs pour [baie](https://fr.wikipedia.org/wiki/Baie_(centre_de_données)) *(rack)*.

Je ne suis pas familier avec les serveurs au format 1U, 2U et 4U. Mais si vous avez l’habitude d’en installer, merci de partager vos conseils dans les commentaires, ils seront les bienvenus. :-)

## Serveur neuf

Acheter du neuf est intéressant si on a vraiment besoin des technologies récentes.
Il se peut aussi les produits proposés en occasion ne correspondent pas à nos besoins, où à des prix moins intéressant que le neuf à performance égale.

Prix | Processeur (cœurs) | Stockage | Ratio | RAM | SSD + HDD
-----:|-------------------------|----------:|-----------:|----------:|-------------:|
800 € |Ryzen 7 2700 (8) | 2 To HDD| 55 €/cœur | 4 Go/cœur | 312 Go/cœur
2500 $|Ryzem 9 3900X (12) | 20 To HDD| 100 $/cœur | 3 Go/cœur | 1700 Go/cœur
6240 $|ThunderX ARMv8 2 GHz (96)| 500 Go SSD| 65 $/cœur | 1 Go/cœur | 5 Go/cœur


## Ordinateur portable

La plupart des portables avec un gros processeur sont des portables pour *gamer* avec aussi de grandes capacités graphiques. Trouver un ordinateur portable de dév. sans Windows, sans carte graphique et sans processeur Intel, est un vrai challenge. Comment faites-vous ?

Par contre, si votre employeur achète des portables pour y installer GNU/Linux, vous pourriez le convaincre d’acheter ces portables sans Windows, notamment via les boutiques suivantes :

* http://shop.ekimia.fr/
* https://keynux.com/
* https://pcubuntoo.fr/
* https://www.ldlc-pro.com/ordinateurs/pc-portable/c5376/+fv403-937,5085,6718,15416.html
* https://pilot.search.dell.com/laptops/9964/laptops

Consulter aussi la liste des bons constructeurs et vendeurs d’ordinateurs :

* https://bons-constructeurs-ordinateurs.info/
* https://bons-vendeurs-ordinateurs.info/

# Conseils et suggestions

Merci de m’éclairer ma lanterne avec des astuces. :-D 

