+++
title = 'creer-une-application-web-avec-jupyter-ipywidgets-et-voila-7b03d5dd-ab10-47cb-a2bd-bd99fa9e2457'
date = 2019-10-05 00:00:00 +0100
categories = divers
+++
URL:     https://linuxfr.org/news/creer-une-application-web-avec-jupyter-ipywidgets-et-voila-7b03d5dd-ab10-47cb-a2bd-bd99fa9e2457
Title:   Créer une application web avec Jupyter, ipywidgets et voilà
Authors: aboulle
         ZeroHeure, Ysabeau et Arkem
Date:    2019-10-04T11:23:14+02:00
License: CC by-sa
Tags:    jupyter et ipywidgets
Score:   16


Vous connaissez sans doute [Jupyter](https://jupyter.org/), cet outil de développement tournant dans un navigateur qui est particulièrement en vogue chez les scientifiques et plus généralement dans les domaines liés au traitement des données. Aujourdʼhui je vais te parler d’une possibilité offerte par Jupyter qu’il ne me semble pas, sauf erreur de ma part, avoir vu évoquée ici, à savoir le développement dʼapplications web.

----

[Journal à l'origine de la dépêche](https://linuxfr.org/users/aboulle/journaux/creer-une-application-web-avec-jupyter-ipywidgets-et-voila)
[Site de Jupyter](https://jupyter.org/)
[Code des exemples de l'article](https://github.com/aboulle/Divers)

----

# À propos de Jupyter
À titre personnel, et peut-être comme beaucoup des plus anciens (disons 40 ans et plus), j’ai longtemps été très réticent à ce « machin à la mode » ne voyant pas bien ce qu’il pouvait apporter à mon flux de travail habituel basé sur un éditeur de texte et une console, et aussi ne lui trouvais-je que des inconvénients :



- lancer ```jupyter-notebook``` dans une console, basculer sur le navigateur, parcourir l’arborescence, juste pour pouvoir visualiser un fichier ipynb me semble très peu ergonomique. Nous sommes en 2019 et ce truc ne gère pas le double clic. Il y a des solutions pour contourner ce problème, par exemple [nteract](https://nteract.io/desktop) est une application de bureau basée sur electron qui permet de se passer du navigateur ;
- le fait que les cellules de code puissent être exécutées dans n’importe quel ordre peut amener à des confusions à la lecture des notebooks ;
- le format ipynb (qui est en fait du json contenant plus d’informations que le simple code) est nativement peu compatible avec git : par exemple le simple fait d’exécuter une cellule modifie la numérotation de celle-ci et git détecte une modification, là encore, [il y a des solutions pour contourner ça](https://nextjournal.com/schmudde/how-to-version-control-jupyter), mais tout de même ;
- tout cela a été [mieux présenté par d’autres que moi](https://www.youtube.com/watch?v=7jiPeIFXb6U) ([diapos](https://docs.google.com/presentation/d/1n2RlMdmv1p25Xy5thJUhkKGvjtV-dkAIsUXP-AL4ffI/edit#slide=id.g362da58057_0_1) de la présentation filmée).



Eh bien, j’avais tort.



En effet, Jupyter ne vise pas à remplacer notre bonne vieille console, et encore moins notre éditeur de texte favori, mais se place entre les deux. Je paraphraserai [la dépêche Python pour les sciences](https://linuxfr.org/news/python-pour-les-sciences-une-presentation) en disant que Jupyter est _une console sous stéroïdes_. En effet, Jupyter permet d’exécuter des blocs de code sans avoir à écrire / sauvegarder / exécuter un script en bonne et due forme et, à l’instar de la console [iPython](https://ipython.org/) dont [Jupyter dérive directement](https://linuxfr.org/news/sortie-de-ipython-jupyter-notebook-4-1), tous les objets sont sauvegardés dans un noyau pour pouvoir être réutilisés ailleurs dans le programme sans avoir à tout ré-exécuter. Ceci en fait un excellent outil d’expérimentation et de prototypage de programmes. 



Par ailleurs, le fait que les notebooks Jupyter contiennent non  seulement le code, mais aussi les graphiques et figures produits, et qu’il soit possible d’y adjoindre du texte enrichi (Markdown, HTML, LaTeX…) les rendent particulièrement intéressants pour l’enseignement et le partage des connaissances (et non pas le partage du code, car comme dit précédemment et [comme le font remarquer certains à juste titre](https://linuxfr.org/nodes/115554/comments/1763182),  le code est fortement obscurci par le format ipynb). 


Il est intéressant de noter que Jupyter est régulièrement évoqué au sein du mouvement _open science_, mouvement qui vise à faciliter la diffusion au sein de la communauté scientifique et auprès du grand public, non seulement des résultats et connaissances scientifiques, mais aussi des données brutes et des protocoles d’analyses et de traitements de ces données. Voir par exemple ces quelques liens: [[1]](https://reproducible-analysis-workshop.readthedocs.io/en/latest/index.html) [[2]](https://arxiv.org/ftp/arxiv/papers/1804/1804.05492.pdf) [[3]](https://www.linuxjournal.com/content/jupyter-future-open-science). 



Bien évidemment ces notebooks Jupyter sont exportables dans différents types de formats (HTML, PDF, etc.) et peuvent également être aisément mis en ligne. [Nbviewer](https://nbviewer.jupyter.org/) permet, par exemple, de partager des notebooks  simplement en passant une URL ou l’adresse d’un dépôt git. 



Pour modérer cet enthousiasme débordant il est, premièrement, bon de rappeler que toutes ces visualisations (y compris sur nbviewer) sont strictement statiques. Il n’est pas possible d’interagir avec celles-ci et donc de ré-exécuter tout ou partie du notebook. Deuxièmement, c’est bien joli de partager des notebooks, mais quid des lecteurs qui ne maîtrisent pas bien, voire pas du tout, le langage dans lequel lesdits notebooks ont été développés ? Ce sont ces points que je me propose d’aborder ici. 


_PS1 : toutes les bibliothèques présentées ci-dessous sont installables via_ ```pip``` _ou_ ```conda-forge```.
_PS2 : Les petits extraits de code donnés ci-dessous sont disponibles sur [mon github](https://github.com/aboulle/Divers)._



# Ajouter des composants graphiques avec ipywidgets



[Ipywidgets](https://github.com/jupyter-widgets/ipywidgets) désigne un ensemble de composants graphiques pour le langage python (slider, combo box, boutons, etc.) destinés à rendre les notebooks plus interactifs. En gros il s’agit d’une architecture permettant de lier un objet python (le widget), tournant dans le noyau, à sa représentation JavaScript/HTML/CSS tournant dans le navigateur. Par exemple, afficher un slider qui permet de modifier la variable d’entrée d’une fonction et d’en afficher le résultat s’écrit simplement :



```python
from ipywidgets import interact
import ipywidgets as widgets

def f(x):
    return x**2

interact(f, x=10.);
```







La fonction ```interact``` est, en fait, un raccourci vers un ensemble de widgets graphiques avec des choix faits par défaut selon le type d’objet (int, float, bool, list, etc.) passé à la fonction f. Il est possible d’avoir un contrôle beaucoup plus fin en paramétrant le widget à la main. Le code ci-dessous donne strictement le même résultat :



```python
# définit l’objet slider
mon_slider = widgets.FloatSlider(
    value=10,
    min=-10,
    max=30,
    step=0.1,
    description='x',
    disabled=False,
    continuous_update=True,
    orientation='horizontal',
    readout = True
)

# créé une zone de texte pour l’affichage du résultat
resultat = widgets.Output()

# définit l’action à effectuer lorsque le slider est modifié
def maj_resultat(change):
    with resultat:
        resultat.clear_output()
        print(f(change['new']))

# observe le slider
mon_slider.observe(maj_resultat, names='value')

# affiche les widgets
display(mon_slider,resultat)
```





C’est, évidemment, beaucoup plus lourd, mais il me semble que cet exemple illustre bien la richesse des potentialités offertes par ipywidgets. [La documentation de ipywidgets](https://ipywidgets.readthedocs.io/en/latest/index.html) est tout simplement excellente, et il est possible de maîtriser cette bibliothèque assez rapidement.



Ipywidgets n’est pas seulement une bibliothèque d’objets graphiques. Il s’agit véritablement d’un cadre de développement sur lequel les développeurs peuvent s’appuyer pour écrire leurs propres bibliothèques de widgets. En voici quelques-unes :



- [ipyvuetify](https://github.com/mariobuikhuizen/ipyvuetify), pour celles et ceux qui trouveraient les widgets de base d’ipywidget trop austères, cette bibliothèque apporte à Jupyter les widgets [vuetify](https://vuetifyjs.com) qui implémentent des composants graphiques obéissants aux spécifications de _material design_ ;
- [bqplot](https://github.com/bloomberg/bqplot), un « doit-avoir » absolu pour quiconque s’intéresse à la visualisation en 2D de données. Je donnerai un exemple d’implémentation ci-dessous, mais, en bref, dans bqplot chaque élément d’un graphique (axes, légende, données, graduations, etc.) est en fait un widget avec lequel il est possible d’interagir et de modifier les propriétés programmatiquement ;
- [ipyvolume](https://github.com/maartenbreddels/ipyvolume), même chose mais pour la visualisation de données en 3D, en s’appuyant sur WebGL ;
- [ipyleaflet](https://github.com/jupyter-widgets/ipyleaflet), affichage et manipulation de cartes et données géographiques ;
- [ipywebrtc](https://github.com/maartenbreddels/ipywebrtc), permet de diffuser et manipuler du contenu audio ou vidéo depuis à peu près n’importe quelle source (fichier, webcam, etc.).



Afin d’illustrer la compatibilité entre ipywidgets et d’autres bibliothèques (ici bqplot) le code ci-dessous permet d’effectuer les actions suivantes :



1. sélectionner une fonction à tracer via un menu déroulant ;
1. la figure est mise à jour en fonction du choix (ipywidgets -> bqplot) ;
1. il est possible de déplacer des points dans la figure ;
1. les coordonnées des points sont affichées dans un champ texte (bqplot -> ipywidgets).



```python
from ipywidgets import interact, fixed
import ipywidgets as widgets
from bqplot import pyplot as plt
import numpy as np
from numpy.random import rand

# génère des abscisses
x = np.arange(0,10,0.1)

# créé une figure y = f(x)
ma_figure = plt. figure(animation_duration = 300)
mon_tracé = plt.scatter(x, x**2, enable_move=True)
plt.xlabel('Axe des x')

# initialise une zone d’affichage de texte
resultat2 = widgets.Output()

# choix de la fonction à tracer -> créé automatiquement un menu déroulant
# modifie le tracé en fonction de la valeur du widget
# il est possible d’utiliser interact via un décorateur
# il est possible de fixer les variables ne devant pas faire l’objet d’un widget
@interact(fonction=['parabole', 'sinus', 'hasard'], x=fixed(x))
def choix_fonction(fonction, x):
    if fonction=='parabole':
        with mon_tracé.hold_sync():
            mon_tracé.x = x
            mon_tracé.y = x**2
            plt.ylabel('x au carré')
    if fonction=='sinus':
        with mon_tracé.hold_sync():
            mon_tracé.x = x
            mon_tracé.y = np.sin(x)
            plt.ylabel('sin(x)')
    if fonction=='hasard':
        with mon_tracé.hold_sync():
            mon_tracé.x = x
            mon_tracé.y = rand(len(x))
            plt.ylabel('Nombres aléatoires')

# fonction qui lit et affiche les coordonnées d’un point déplacé
def affiche(name, value):
    with resultat2:
        resultat2.clear_output()
        print('Le point n° %i a été déplacé en x = %f y = %f'
              %(value['index'], value['point']['x'],value['point']['y']))

# détecte le déplacement d’un point
mon_tracé.on_drag_end(affiche)     

# créé la gui
# il est possible de mixer des widgets créés via interact avec d’autre définis « à la main »
widgets.VBox([ma_figure,resultat2])  
```






Toutes ces bibliothèques sont relativement jeunes et il peut arriver que la documentation ne soit pas exhaustive (c’est, par exemple, le cas pour bqplot). Dans ce cas, il peut être très intéressant de cloner ou télécharger le dépôt Github et d’aller fouiller dans le répertoire _examples_. Dans le cas de bqplot, c’est une véritable mine d’or.



# Masquer le code avec Appmode ou Voilà
Maintenant que nous savons comment créer une petite interface graphique, pourquoi ne pas cacher tout ce code afin de ne pas effrayer les débutants ? [Appmode](https://github.com/oschuett/appmode) est une extension pour Jupyter qui permet très exactement de faire cela : l’extension ajoute un bouton `Appmode` à l’interface de Jupyter qui permet de créer une nouvelle instance du notebook, celui-ci est alors entièrement exécuté et seules les widgets sont affichées.



Si elle est très efficace, cette extension peut être problématique si le notebook est destiné à être hébergé sur un serveur Jupyter ouvert fournit pas votre entreprise / université / école… En effet le notebook reste entièrement accessible et rien n’interdit l’exécution de code arbitraire. C’est ce problème que solutionne [voilà](https://github.com/voila-dashboards/voila). Ce projet est très jeune puisqu’il n’a été annoncé que cet été dans [ce très instructif billet de blog](https://blog.jupyter.org/and-voil%C3%A0-f6a2c08a4a93) mais il s’avère déjà particulièrement efficace. En bref, lorsque l’URL du notebook est appelée, celui-ci s’exécute intégralement et les cellules de résultats (incluant les widgets) sont converties en une page HTML + JavaScript qui est ensuite présentée à l’utilisateur. En principe, il (ou elle) ne peut plus exécuter de code arbitraire. Pour celles et ceux à qui ça parle (dont je ne fais pas partie), ça repose, entre autres, sur [tornado](https://www.tornadoweb.org/).



La [galerie](https://voila-gallery.org/services/gallery/) de _voilà_ regorge d’exemples, comme [celui-ci où on peut jouer avec une fonction gaussienne](https://voila-gallery.org/user/voila-gallery-gaussian-density-nzh9rp3d/voila/render/index.ipynb?token=zu-axkwTRI2Fr-QOaNHPGw). En voyant cet exemple il est utile de rappeler que tout ceci n’est rien d’autre qu’un notebook Jupyter.



# Héberger l’application web
Dernière étape pour finaliser notre application web : la mise en ligne. Comme je l’évoquais plus haut, _nbviewer_ est exclu puisque celui-ci ne permet pas d’interagir avec les notebooks. Si vous avez la chance d’avoir votre propre serveur Jupyter distant (ou d’avoir des administrateurs et des administratrices compétents et sympas), c’est immédiat. Il vous suffit d’activer l’extension _voilà_ : `jupyter serverextension enable voila --sys-prefix` puis de préfixer l’URL du notebook avec « voila » :
`http://URL_DU_SERVEUR/NOM_DU_NOTEBOOK.ipynb` 
devient 
`http://URL_DU_SERVEUR/voila/NOM_DU_NOTEBOOK.ipynb`



Notez qu’il est possible de visualiser le rendu du notebook en local, entrez `voila notebook.ipynb` dans votre terminal et le rendu sera visible sur `localhost:8866`.



Si vous n’avez pas de serveur Jupyter ouvert sur le web, tout n’est pas perdu. [Mybinder](https://mybinder.org/) permet de venir se brancher sur un dépôt git, puis, à l’aide d’un fichier ```requirements.txt``` ou ```environment.yml``` listant les dépendances requises, Mybinder va construire une image Docker du dépôt et votre notebook sera servi via un [JupyterHub](https://jupyterhub.readthedocs.io/en/latest/). Le contenu du fichier ```environment.yml``` pour les exemples précédents est :



```yaml
channels:
  - conda-forge
dependencies:
  - numpy
  - ipywidgets
  - bqplot
  - voila
```





Finalement le notebook est accessible par un lien de la forme : [![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/aboulle/Divers/master?filepath=%2Fipywidgets_linuxfr.ipynb)
Deux remarques utiles à ce stade :



1. la création de l’image Docker peut prendre plusieurs minutes ;
2. concernant _voilà_, il existe apparemment un moyen d’atterrir directement sur le rendu HTML du notebook et non le notebook lui-même en préfixant le nom du notebook avec ```/voila/render/```, mais pour moi cela ne semble pas fonctionner avant la première création de l’image. Il faut donc cliquer sur le bouton 'voilà' dans le notebook pour cacher le code source.



[Heroku](https://www.heroku.com/) est une solution alternative à mybinder. La procédure de déploiement est cependant moins aisée (mais les tutoriels sont très bons), et [les dépendances considérées comme « obscures » par heroku](https://devcenter.heroku.com/articles/python-pip#scientific-python-users) ne sont pas gérées : par exemple SciPy n’est pas géré, ce qui est handicapant pour des applications techniques ou scientifiques.



# Mot de la fin
Pour conclure, et pour illustrer le fait qu’il est possible de créer des applications relativement élaborées, je partage ici un lien vers une application scientifique que j’ai récemment développée et qu’un collègue de notre département TIC a œuvré à [mettre en ligne](https://radmax.unilim.fr) (un grand merci à lui !).



En bref il s’agit, dans le graphique de droite, de faire coller la courbe calculée (en rouge) sur des mesures de [diffraction des rayons X](https://fr.wikipedia.org/wiki/Cristallographie_aux_rayons_X) expérimentales. Les graphes de gauche sont manipulables et le rendu des calculs est donné en temps réel dans le graphe de droite. Pour le contexte, l’objectif final est de déterminer les dommages que subissent des matériaux soumis à des radiations. Ces dommages sont quantifiés par l’évolution en profondeur du taux de déformation et de désordre atomique (graphes de gauche). Le calcul est paramétré par les différents widgets. C’est encore expérimental, la stabilité n’est donc pas garantie ; de plus, il est possible que l’URL change dans les jours à venir.



Si ça vous intéresse, [voici les sources](https://github.com/aboulle/RaDMaX-webapp).
