+++
title = 'tout-ce-que-vous-avez-voulu-savoir-sur-linux-avant-de-vous-y-mettre'
date = 2019-11-25 00:00:00 +0100
categories = divers
+++
URL:     https://linuxfr.org/news/tout-ce-que-vous-avez-voulu-savoir-sur-linux-avant-de-vous-y-mettre
Title:   Tout ce que vous avez voulu savoir sur Linux avant de vous y mettre
Authors: Ysabeau
         Davy Defaud et ZeroHeure
Date:    2019-11-24T18:16:32+01:00
License: CC by-sa
Tags:    
Score:   4


Mes camarades de la modération m’ayant proposé de transformer le journal présentant mon dernier guide _Linux : une introduction_ en dépêche, le voici donc, avec quelques ajouts concernant les modèles de LibreOffice et l’art de faire du fichier EPUB, pour aussi répondre à certaines questions.

Concernant la licence de ce petit guide d’introduction à Linux, objet de cette dépêche, le ban et l’arrière‐ban du pinaillage en la matière ayant été convoqué, je suggère d’aller faire un tour sur les commentaires du journal d’origine.

----

[Journal à l’origine de la dépêche](https://linuxfr.org/users/ysabeau/journaux/tout-ce-que-vous-avez-voulu-savoir-sur-linux-avant-d-entrer-dedans)
[Linux : une introduction](https://numericoach.net/Linux-une-introduction)
[PaletteMaker](https://numericoach.net/PaletteMaker-une-macro-qui-genere-des-palettes)

----

Linux : une introduction, présentation du guide
========================

Vous, je ne sais pas, mais moi, avant d’aborder Linux, il y avait des tas de trucs qui me laissaient perplexe. Il faut dire que, quand on vient des mondes monolithiques macOS et Windows, la diversité du monde Linux peut dérouter. Par exemple, les concepts de distribution et d’environnements de bureau ne sont pas évidents à comprendre d’entrée de jeu. Quand on a un ordinateur qui tourne sous Windows ou, pire, un ordinateur de marque Apple (pire, parce que la marque impose un écosystème plutôt fermé), on a une interface graphique, épicétout, et il faut aller à la pêche aux logiciels un peu partout pour avoir un ordinateur pleinement fonctionnel, etc.



Bref, on peut se poser, et je m’en suis posée, tout un tas de questions avant et après avoir commencé à entrer dans l’univers de Linux. Et comme j’ai la faiblesse de penser que je suis loin d’être la seule dans le même cas, j’ai fini par rédiger ce (pas si) petit guide qui répond à tout ce genre de questions et à quelques autres. Et comme il est abondé d’un glossaire et d’une biblio‐sitographie, il devrait pouvoir être utile aux personnes qui veulent passer à Linux ou, tout simplement, pour répondre aux diverses questions dans les manifestations sur le Libre et les _install parties_.


Ça s’appelle « [Linux : une introduction](https://numericoach.net/Linux-une-introduction) » et on peut le télécharger au format PDF et EPUB.

Les grands chapitres :
    
- Linux, un système, un noyau ?
- Une distribution, qu’est‑ce que c’est ?
- Un environnement de bureau, pourquoi en faut‑il un ?
- Comment est rangé un ordinateur sous GNU/Linux ?
- Linux c’est pour les geeks, non ?



Des modèles de LibreOffice et une proposition
=============================================



On m’a fait remarquer que les modèles de LibreOffice sont à la fois peu attrayants et tristounets. Ce qui ne serait pas gênant s’ils n’étaient aussi techniquement mal fichus la plupart du temps. Notez que ceux de la concurrence privatrice ne sont pas mieux faits sur ce plan.



Parmi les quelques modèles que j’ai déjà émis à destination du vaste monde, on trouvera donc ce [modèle de document long](https://numericoach.net/Modele-de-document-long-version-2018) avec des pages droite et gauche et des onglets qui reprennent automatiquement le titre du chapitre. Les explications figurent dans le document. Pour compléter on trouvera également sur ce site un dépliant sur [Writer et les styles](https://numericoach.net/Depliant-Writer-les-styles), ou encore un rapide [guide œcuménique sur la question](https://numericoach.net/Utiliser-les-styles-un-guide).


Mais, comme cette année j’ai prévu de faire un calendrier de l’Avent sur mes trois sites et que sur celui‑ci, l’idée est de proposer un modèle (ou un dessin) par jour, ce que je vous propose c’est de me dire ce que vous voudriez voir :
    
- type de document (texte long, brochure, présentation, feuille de calcul, que sais‑je) ;
- ambiance ou thème général ;
- gamme de couleurs.

Sachant qu’avec la macro [PaletteMaker](https://numericoach.net/PaletteMaker-une-macro-qui-genere-des-palettes), vous pourrez facilement intégrer dans votre profil de LibreOffice la palette de couleurs livrée avec le modèle, voire faire votre propre palette.

Pour chaque modèle, il y aura de rapides explications dans la partie commentaires des propriétés des documents.



Réflexion sur la production de fichiers EPUB
============================================


En théorie, quand je fais des guides ce de genre, j’en produis aussi une version EPUB, ce qui est le cas de celui‑ci. En pratique, l’EPUB étant plus exigeant que le PDF et prenant plus de temps à générer à cause des vérifications et des corrections, j’y rechigne un peu.

Avantages du format EPUB sur le PDF : il embarque le XML et est donc plus accessible des dispositifs d’assistance, et, évidemment, il est mieux lu sur une liseuse.

Pour transformer un document du type de ce guide en EPUB, il faut, au préalable, le préparer : supprimer les en‑têtes et pieds de page, paramétrer les tailles de polices en pourcentage et diminuer les retraits de paragraphes de liste. Éventuellement, revoir la position des images et faire une image de couverture.

**LibreOffice** exporte correctement les images et leur habillage, mais vire les listes à puces transformées en paragraphes et, quoique la fonctionnalité soit prévue, n’accepte pas d’image de couverture externe (un bogue sans doute). En outre, j’ai découvert avec consternation que, même quand on met en forme correctement un texte, c’est‑à‑dire **uniquement** avec des styles, LibreOffice fait de la cochonnerie par‐derrière et je ne comprends pas pourquoi.

**Calibre** est fâché avec la position des images, instaure un saut de paragraphe, à ce qu’il me semble, à chaque titre (niveau un, logique, mais aussi niveau deux), mais sait bien ajouter les balises `ul` et `li`.

Petite précision, le rendu calamiteux concernant les images dont je fais état est sur ma liseuse datant de 2011, sur ma tablette cela rend bien. Et, définitivement, la liseuse ne lit que les EPUB 2.

Pour terminer
=============



Un grand merci pour les remarques formulées, j’en ai tenu compte pour amender le document conçu, au départ, comme support à une conférence sur le sujet.



L’idée était d’avoir un document unique utilisable hors d’Internet, c’est pourquoi la formule du wiki ne me convient pas, et qui soit aussi une sorte de « vitrine », non seulement de mon savoir-faire, mais surtout de ce que l’on peut faire avec un logiciel comme LibreOffice.
