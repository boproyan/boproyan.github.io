+++
title = 'youtube-dl'
date = 2019-12-30 00:00:00 +0100
categories = application
+++
## youtube-dl

*[youtube-dl](https://github.com/ytdl-org/youtube-dl) sous licence [Unlicense](https://unlicense.org/) permet de télécharger les flux audio et vidéo de [nombreux sites](https://ytdl-org.github.io/youtube-dl/supportedsites.html)*

* [youtube-dl, récupérer les flux audio et vidéo de nombreux sites](https://www.blog-libre.org/2019/05/08/youtube-dl-recuperer-les-flux-audio-et-video-de-nombreux-sites/)
* [youtube-dl : comment récupérer légalement des flux audio et vidéo depuis un millier de sites](https://www.nextinpact.com/news/106174-youtube-dl-comment-recuperer-legalement-flux-audio-et-video-depuis-millier-sites.htm)

### Principales commandes et options

* **-F, --list-formats** : Lister tous les formats des flux audio et vidéo disponibles pour une URL
* **-r, --limit-rate** : Limiter le débit du téléchargement (500K ou 3.5M par exemple)
* **-a, --batch-file** : Fournir une liste d’URLs à télécharger via un fichier (une URL par ligne)
* **-i, --ignore-errors** : Continuer si une erreur se produit lors d’un téléchargement, surtout utile quand on fournit une liste d’URLs à télécharger (--batch-file)
* **-o, --output** : Nommer le fichier de sortie à partir de template, voir les exemples et la documentation
* **-x, --extract-audio** : Extraire le flux audio
* **--audio-format** : Spécifier le format audio de sortie (« best », « aac », « flac », « mp3 », « m4a », « opus », « vorbis », « wav », « best » par défaut)
* **--audio-quality** : Spécifier la qualité audio de sortie (valeur entre 0 la meilleure et 9 la pire)
* **-f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best'** : Télécharger le meilleur format mp4 disponible ou le meilleur format disponible si le mp4 n’est pas disponible, voir les [exemples](#output-template-examples) et la documentation

### Récupérer le son d’une vidéo

Une énorme quantité de clips et chansons sont présentes sur YouTube, vous avez envie d’avoir Je danse le Mia sous le coude ?

    youtube-dl --extract-audio --audio-format m4a --audio-quality 0 --output "~/Musique/%(title)s.%(ext)s" https://www.youtube.com/watch?v=wf4YT-vsq_4

Vous obtiendrez ~/Musique/IAM - Je Danse le Mia (Audio officiel).m4a.

### Récupérer une vidéo

Votre pêché mignon est Capitaine Marleau (comme moi) mais vous avez loupé celui de mardi soir (shit une rediff de la saison 1 !) ?

    youtube-dl -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best' --output "~/Téléchargements/%(title)s.%(ext)s" https://www.france.tv/france-3/capitaine-marleau/saison-1/304197-en-trompe-l-oeil.html

Vous obtiendrez ~/Téléchargements/Capitaine Marleau - En trompe-l'oeil.mp4.

### Récupérer une liste de vidéos

    youtube-dl -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best' --output "~/Téléchargements/%(title)s.%(ext)s" --ignore-errors --batch-file '~/Téléchargements/Liste_dl.txt'

Vous obtiendrez… un paquet de vidéos. Vous pouvez vous passer de l’option **--batch-file** en renseignant plusieurs URLs sur la ligne de commande.

    youtube-dl -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best' --output "~/Téléchargements/%(title)s.%(ext)s" --ignore-errors URL URL URL

### Créer un alias

*Sur Linux, les alias sont des raccourcis de commandes jugées trop longues par l’utilisateur.
En effet, le terminal est très pratique mais les commandes sont parfois lourdes et il devient facile de se tromper.
Un alias permet également de gagner du temps en créant une commande courte pour une séquence que l’on tape fréquemment.*

On ajoute l'alias qui permet de télécharger dans le dossier en cours une vidéo youtube au format **titre.mp4**  
Editer le fichier `$HOME/.bashrc` , et ajouter   

    alias youtube="youtube-dl -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best' --output '%(title)s.%(ext)s' --ignore-errors" 

Ouvrir un terminal et lancer la commande réduite `youtube` suivi du lien

    youtube https://www.youtube.com/watch?v=7KCVP-CWlTs

[Plus sur les commandes-Lien HS](/files/html/youtube-dl.htm)
