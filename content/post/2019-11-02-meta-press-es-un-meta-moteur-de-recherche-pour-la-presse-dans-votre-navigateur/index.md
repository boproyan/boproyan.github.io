+++
title = 'meta-press-es-un-meta-moteur-de-recherche-pour-la-presse-dans-votre-navigateur'
date = 2019-11-02 00:00:00 +0100
categories = divers
+++
URL:     https://linuxfr.org/news/meta-press-es-un-meta-moteur-de-recherche-pour-la-presse-dans-votre-navigateur
Title:   Meta‑Press.es : un méta‑moteur de recherche pour la presse dans votre navigateur
Authors: Siltaär
         ZeroHeure, Davy Defaud, Pierre Jarillon et palm123
Date:    2019-10-31T16:31:21+01:00
License: CC by-sa
Tags:    presse, revue_de_presse, news, google, degooglisons, javascript et webextensions
Score:   21


Mozilla vient de valider l’ajout de l’extension Meta‑Press.es à son catalogue. C’est l’aboutissement de plusieurs années d’efforts et c’est une étape importante pour ce projet de méta‑moteur de recherche, conçu d’abord pour les journalistes et les **revues de presse des associations**.
![logo de Meta‑Press](https://www.meta-press.es/theme/img/logo-metapress-256px-sq.png)


Meta‑Press.es est techniquement simple, il permet d’interroger suffisamment de journaux pour découvrir plusieurs millions de résultats en quelques secondes, tout en rapatriant les dix derniers de chaque journal dans le navigateur de l’utilisateur.


De là, les résultats peuvent être triés, explorés, filtrés, sélectionnés et exportés. Une sélection de résultats peut être réimportée plus tard dans le navigateur ou bien dans le navigateur d’un autre utilisateur. Elle peut encore servir à alimenter le flux RSS de la revue de presse d’une association.

----

[Site de Meta‑Press.es](https://www.meta-press.es)
[Les sources sur Framagit](https://framagit.org/Siltaar/meta-press-ext)
[L’extension dans le magasin de Mozilla Firefox](https://addons.mozilla.org/en-US/firefox/addon/meta-press-es/)
[Annonce de Meta‑Press.es sur Grimoire-Command](https://www.grimoire-command.es/2019/meta-presses_v10.html#_version_fr)
[Symposium DECODE le 5 novembre à Turin](https://decodeproject.eu/events/workshops-agenda-5th-november)

----

[![Copie d’écran de l’extension Meta‑Press](https://addons.cdn.mozilla.net/user-media/previews/full/227/227493.png?modified=1572341463)](https://addons.cdn.mozilla.net/user-media/previews/full/227/227493.png?modified=1572341463)


La validation par Mozilla est une date de lancement. Le projet sera présenté au dernier [symposium DECODE, le 5 novembre prochain à Turin](https://decodeproject.eu/events/workshops-agenda-5th-november).

Cette publication arrive à point nommé dans le bras de fer qui oppose Google et les éditeurs de presse en ligne concernant une rémunération des extraits d’articles publiés par le moteur de recherche (en application de la directive européenne sur le droit d’auteur adoptée le 26 mars 2019), pour montrer qu’il est possible de faire autrement. 


En effet, Meta‑Press.es ne publie rien, c’est chaque utilisateur qui interroge chaque journal depuis son propre ordinateur. De plus, Meta‑Press.es ne tente pas de voler la rémunération publicitaire des autres en mettant sa pub à côté des contenus importés.


# Feuille de route
Pour l’instant il y a cinquante sources disponibles (majoritairement en langue anglaise, mais de quarante pays différents), mais déjà des contributeurs en ont ajouté de nouvelles. La moitié du travail est fait, le méta‑moteur est fonctionnel et facilement installable. La suite sera un minutieux travail d’inventaire… Il suffit d’écrire **vingt lignes de JavaScript pour ajouter une nouvelle source**, et elles peuvent être testées depuis les préférences de l’extension, c’est donc à la portée de bon nombre de développeurs.


Il reste à [internationaliser l’extension, coder la vérification routinière des liens, travailler le graphisme](https://www.meta-press.es/fr/journal/2019/release_of_meta-press.es_v1.0.html), etc. Toute aide est la bienvenue.


## Motivation
_N. D. M. : ce paragraphe a été rédigé en modération à partir de [la présentation initiale du projet](https://www.meta-press.es/fr/journal/2017/motivations.html)._
    
J’ai fait partie, dès 2008, des citoyens indépendants engagés dans le même sens que [La Quadrature du Net](https://www.laquadrature.net/)… Après quelques traductions, je me suis volontairement attelé à une tâche besogneuse : la revue de presse. Pendant plus de cinq ans, j’ai consacré la majorité de mon temps libre à tenir la revue de presse de _La Quadrature_ à jour. La tâche était colossale. 
J’ai longuement cherché une alternative aux alertes Google News, sans rien trouver qui soit libre, ou même accessible, et qui, surtout, rivalise avec le pouvoir d’indexation d’un des plus gros moteurs de recherche !
J’ai commencé un premier prototype fin 2013. Benjamin Bayart m’avait dit, droit dans les yeux, que se passer des moteurs de recherche centralisés est un enjeu majeur du logiciel libre. Mais c’était trop lent et le navigateur ne permettait pas de faire partir les requêtes depuis le poste de l’utilisateur. La nouvelle norme (ECMAScript 5, 6 et 7) et la vitesse d’exécution actuelle ont changé la donne : au troisième trimestre 2017, le prototype était capable de découvrir des millions de résultats en moins de dix secondes. Un tour du monde de la presse, en un clic.
