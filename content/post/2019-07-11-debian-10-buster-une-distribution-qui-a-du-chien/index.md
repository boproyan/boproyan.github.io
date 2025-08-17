+++
title = 'Debian 10 Buster , une distribution qui a du chien'
date = 2019-07-11 00:00:00 +0100
categories = ['debian']
+++
# Debian 10 Buster

* URL:     <https://linuxfr.org/news/debian-10-buster-une-distribution-qui-a-du-chien>  
* Authors: Collectif thomasv, antistress, j, M5oul, mzf, BAud, Davy Defaud, Arcaik, Xavier Claude, jihele, bolikahult, Arkem, jona, Raoul Volfoni, Apichat, Étienne BERSAC et cyberjunkie  
* Date:    2019-06-11T23:05:49+02:00  
* License: CC by-sa   


Debian GNU/Linux est une distribution communautaire entièrement construite avec des logiciels libres. Sa version 10, nom de code _Buster_ (en référence au [chien](https://pixar.fandom.com/wiki/Buster) d’Andy dans _[Toy Story 2](https://fr.wikipedia.org/wiki/Toy_Story_2))_, a été publiée le 6 juillet 2019.


![Debian 10](https://framapic.org/3VPBjhgYxkgc/Wb7DEAzZOIdC.png)


_Buster_ est disponible officiellement sur dix architectures différentes : AMD64, ARM64, ARMel, ARMhf, i386, MIPS, MIPS64el, MIPSel, PowerPC64el et s390x ([les mêmes](https://lists.debian.org/debian-devel-announce/2019/04/msg00003.html) que pour _[Stretch](https://linuxfr.org/news/debian-9-stretch-deploie-ses-tentacules)_, la précédente version).


Cette nouvelle version de Debian GNU/Linux contient plus de 51 000 paquets, dont 15 000 nouveaux. Par ailleurs, 6 000 paquets ont été supprimés depuis _Stretch_.


Parmi les nouveautés, la sécurité est à l’honneur avec la prise en charge de [SecureBoot](https://fr.wikipedia.org/wiki/UEFI) pour les architectures les plus répandues, l’activation d’[[AppArmor]] sur les nouvelles installations, le choix de [[Wayland]] comme serveur d’affichage par défaut pour [[GNOME]], ou encore les avancées concernant le chantier des compilations reproductibles.

----

[Nouveautés de Debian 10](https://www.debian.org/releases/buster/amd64/release-notes/ch-whats-new.fr.html)
[Notes de publication pour Debian 10 (« Buster »)](https://www.debian.org/releases/buster/amd64/release-notes/index.fr.html)
[Debian Buster Artwork](https://wiki.debian.org/DebianArt/Themes/futurePrototype)
[[PDF] Carte de référence Debian](https://www.debian.org/doc/manuals/refcard/refcard)
[Annonce de la publication](https://www.debian.org/News/2019/20190706)
[Annonce Debian Edu](https://www.debian.org/News/2019/20190707)

----

Présentation
============
Le projet [[Debian]] a été initié par [Ian Murdock](https://fr.wikipedia.org/wiki/Ian_Murdock) en 1993. C’est l’un des premiers systèmes d’exploitations à utiliser un noyau Linux.
    
Le projet est développé internationalement par des bénévoles grâce à Internet. Le [Debian Project Leader](https://www.debian.org/devel/leader) élu chaque année  (actuellement [Sam Hartman](https://linuxfr.org/news/sam-hartman-a-ete-elu-dpl-2019-debian-project-leader), qui succède à Chris Lamb) guide cette communauté en s’appuyant sur le [Contrat social Debian](https://www.debian.org/social_contract) et [Les principes du logiciel libre selon Debian](https://www.debian.org/social_contract#guidelines).
    
Réputée pour sa stabilité, Debian sert de base à de nombreuses autres distributions. Le site [DistroWatch](https://fr.wikipedia.org/wiki/DistroWatch) en dénombre [132 actives](https://distrowatch.com/search.php?ostype=All&category=All&origin=All&basedon=Debian&notbasedon=None&desktop=All&architecture=All&status=Active) dont les populaires [Ubuntu](https://ubuntu.com/), [Linux Mint](https://linuxmint.com/), [Tails](https://tails.boum.org/index.fr.html) ou [Raspbian](https://www.raspberrypi.org/downloads/raspbian/).
Pour assurer cette stabilité, le cycle de développement d’une nouvelle version est en général de plusieurs années. La sortie de _Buster_ aura ainsi nécessité plus de deux ans de préparation.
    
Debian est une distribution généraliste. Elle fournit des outils autant pour la bureautique (LibreOffice, Thunderbird, GNOME, KDE,  Xfce…), que pour le développement (GCC, Emacs, Vim, JDK, etc.), les serveurs Web (Apache, nginx), messagerie (Postfix…), virtualisation (KVM), conteneurisation (LXC, Docker…).
    
La liste des [utilisateurs officiellement déclarés](https://www.debian.org/users/) contient des entreprises comme [Blackblaze](https://www.debian.org/users/com/backblaze), des organisations à but non lucratif comme [TuxFamily](https://www.debian.org/users/org/tuxfamily), des institutions éducatives ou encore des organisations gouvernementales comme l’[INSEE](https://www.debian.org/users/gov/insee).


Cycle de développement
======================
À partir de la publication de _Stretch_ en juin 2017, _Buster_ est entrée dans sa phase de développement durant laquelle les paquets ajoutés dans le dépôt _unstable_ migraient automatiquement dans _testing_ au bout de quelques jours.
    
Après environ un an et demi de développement, les vannes se sont fermées et _Buster_ est progressivement entrée dans sa [phase de gel](https://release.debian.org/buster/freeze_policy.html), composée de trois étapes.


Transition freeze
-----------------
Le gel de transition a débuté le 12 janvier 2019, interdisant les transitions de grande ampleur (comme les bibliothèques dont dépendent beaucoup de logiciels) et les migrations de paquets introduisant de nouvelles régressions.


Soft freeze
-----------
Le gel léger a débuté le 12 février 2019, pendant lequel le délai de migration était fixé à au moins dix jours et interdisant l’entrée (ou le retour) dans _testing_ des paquets absents de _testing_.


Full freeze
-----------
Le gel complet a débuté le 12 mars 2019, qui a restreint les migrations aux corrections de bogues critiques pour la publication et aux bogues marqués importants dans les paquets optionnels.
    
![Debian 10](https://framapic.org/Hn2wMrTlvPa9/kMeJw7RIfZnO.image)


Nouveautés
===========
## Paquets mis à jour
Côté plomberie :
    
- Linux 4.9 → [4.19](https://www.developpez.com/actu/230249/Le-noyau-Linux-4-19-est-disponible-tour-d-horizon-des-nouveautes-qui-accompagnent-cette-version-LTS/) ;
- GCC 6.3 → [8.3](https://linuxfr.org/news/sortie-de-gcc-8-1) ;
- glibc 2.24-11 → 2.28-10 ;
- Python 3.5.3 → 3.7.3.


Quelques environnements graphiques :
    
- GNOME 3.22 → [3.30.2](https://linuxfr.org/news/parution-de-gnome-3-30) (qui inclut les changements [des](https://linuxfr.org/news/gnome-fete-ses-20-ans) [versions](https://linuxfr.org/news/des-nouvelles-de-gnome-a-l-occasion-de-la-3-26) [intermédiaires](https://linuxfr.org/news/gnome-3-28)) ;
- KDE Plasma 5.8 → [5.14](https://kde.org/announcements/plasma-5.14.0.php) ;
- LXDE 9 → 10 ;
- LXQt 0.11 → [0.14](https://lxqt.org/release/2019/01/25/lxqt-0140/) ;
- MATE 1.16 → [1.20](https://linuxfr.org/news/sortie-de-mate-1-20) ;
- [Xfce 4.12](https://linuxfr.org/news/xfce-4-12-est-la) (déjà dans _Stretch_) ;
- Cinnamon 3.2 → [3.8](https://blog.linuxmint.com/?p=3557).


Quelques applications :
    
- GIMP 2.8 → [2.10.8](https://linuxfr.org/news/gimp-2-10-8-wilber-kid) (qui inclut les nouveautés [des](https://linuxfr.org/news/gimp-2-10-6-rien-ne-nous-arrete) [versions](https://linuxfr.org/news/gimp-2-10-4-on-garde-le-rythme) [précédentes](https://linuxfr.org/news/sortie-de-gimp-2-10-2) de la série [2.10](https://linuxfr.org/news/gimp-2-10-roule-au-gegl)) ;
- Firefox ESR 52 puis 60 → [60](https://linuxfr.org/news/firefox-60-et-60-esr) (Firefox [ESR](https://developer.mozilla.org/fr/docs/Mozilla/Firefox/Firefox_ESR) bénéficie d’[une exception](https://lwn.net/Articles/676963/) pour recevoir les mises à jour de versions majeures dans la version stable de Debian) ;
- LibreOffice 5.2 → [6.1](https://wiki.documentfoundation.org/ReleaseNotes/6.1/fr) (qui inclut les nouveautés [des](https://linuxfr.org/news/sortie-de-libreoffice-5-3) [versions](https://linuxfr.org/news/libreoffice-5-4-5) [intermédiaires](https://linuxfr.org/news/sortie-de-libreoffice-6-0)) ;
- GnuPG 2.1 → 2.2 ;
- PostgreSQL 9.6 → 11.3 ;
- Flatpak 0.8 → [1.2](https://github.com/flatpak/flatpak/releases/tag/1.2.0).


## Nouveaux paquets intéressants
- `rustc`, le compilateur du langage [Rust](https://fr.wikipedia.org/wiki/Rust_(langage)), nécessaire notamment pour compiler les versions récentes du navigateur Firefox, est disponible en version 1.34 (en fait [plus de 500 paquets dans Debian dépendent déjà de Rust](https://people.debian.org/~mafm/posts/2019/20190617_debian-gnulinux-riscv64-port-in-mid-2019/)) ;
- `matrix-synapse`, le serveur Matrix de référence, en version 0.99. La version 1, sortie récemment, [arrivera bientôt dans les rétroportages](https://matrix-team.pages.debian.net/blogue/2019/06/26/june-2019-matrix-on-debian-update/) ;
- [taptempo](https://linuxfr.org/users/mzf/journaux/un-tap-tempo-en-ligne-de-commande), [qu’on ne présente plus](https://linuxfr.org/tags/taptempo/public), en version 1.4.4 ;
- [[DokuWiki]], absent de _Stretch_ (voir le [bogue n^(o) 854592](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=854592)), est de retour dans _Buster_ ;
- [`exa`](https://the.exa.website/), un remplaçant de `ls` ;
- [vlc-bittorrent](https://github.com/johang/vlc-bittorrent) fait [son entrée](http://people.skolelinux.org/pere/blog/Non_blocking_bittorrent_plugin_for_vlc.html) dans Debian, il permet de lire les [fichiers et liens magnets Torrent](http://people.skolelinux.org/pere/blog/Web_browser_integration_of_VLC_with_Bittorrent_support.html) avec VLC et depuis le navigateur… et [ça marche](http://people.skolelinux.org/pere/blog/Non_blocking_bittorrent_plugin_for_vlc.html) !
- [hollywood](https://packages.debian.org/buster/hollywood), qui vous permettra de passer pour un hacker, un vrai ;
- [[LilyPond]], un programme de composition de partitions musicales, de retour après avoir été absent de _Stretch_ ; il est intéressant de constater que le [bogue n^(o) 746005](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=746005), qui est à l’origine du retrait de LilyPond de _Stretch_, n’est toujours pas résolu ;
- [Movim](https://linuxfr.org/news/movim-0-14-scotty), en version 0.14 ;
- [[poezio]], en version 0.12.1 ;
- [spectre-meltdown-checker](https://github.com/speed47/spectre-meltdown-checker).


## Paquets supprimés notables
- `manpages-fr`, qui n’était plus maintenu depuis 2014, a été supprimé de _Buster_ suite à l’ouverture du [bogue n^(o) 871564](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=871564) ; la  traduction des pages de manuel a depuis repris, de même que leur empaquetage ; hélas, le nouveau paquet n’a pas pu être prêt avant le gel ;
- [Amarok](https://fr.wikipedia.org/wiki/Amarok_(logiciel)), suite à la transition de Qt4 vers Qt5 dans Debian ([Amarok est resté en Qt4](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=784448)) ;
- `debian-doc-fr`, qui n’était plus mis à jour depuis… [douze ans !](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=897281)
- [[Redmine]], suite à la transition vers Ruby on Rails 5 dans Debian ; contrairement au cas d’Amarok, une version de Redmine fonctionnant avec RoR5 existe mais n’a [pas été empaquetée à temps](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=921049).

iptables est remplacé par nftables
----------------------------------
Le cadriciel [iptables est remplacé par nftables](https://www.debian.org/releases/buster/amd64/release-notes/ch-whats-new.fr.html#nftables). Debian _Buster_ contiendra néanmoins [des outils pour pouvoir continuer à utiliser iptables](https://wiki.debian.org/nftables), mais recommande fortement de migrer vers nftables.


AppArmor activé par défaut
--------------------------
Sur les nouvelles installations, [[AppArmor]] sera dorénavant activé par défaut. AppArmor fournira par défaut des profils pour certains programmes (Apache, GnuPG, Bash…), tandis que d’autres programmes seront livrés avec leur propre profil AppArmor. Enfin, le paquet `apparmor-profiles-extra` fournira des profils supplémentaires pour les paquets n’embarquant pas leur propre profil de confinement AppArmor.
    
> Si vous faites la mise à niveau vers _Buster_, il suffit de vérifier que le paquet `apparmor` est bien installé pour profiter de ses fonctionnalités.


SecureBoot avec UEFI
--------------------
Absente lors de la sortie de _Stretch_, la prise en charge de SecureBoot avec [[UEFI]] est enfin d’actualité avec _Buster_, pour les architectures AMD64, i386 et ARM64.
    
_Cf._ <https://debamax.com/blog/2019/04/19/an-overview-of-secure-boot-in-debian>.
    
> Pour profiter de SecureBoot à l’occasion d’une mise à niveau vers _Buster_, il faut installer les paquets `shim-signed`, `grub-efi-amd64-signed` ou `grub-efi-ia32-signed` et activer l’UEFI.

## /usr fusionné pour les nouvelles installations
Sur les nouvelles installations, l’arborescence du système de fichiers est modifiée comme indiqué ci‐dessous :
    
`/bin` → `/usr/bin` ;
`/sbin` → `/usr/sbin` ;
`/lib` → `/usr/lib`.
    
Les anciens répertoires deviennent des liens symboliques pointant vers les nouveaux.
    
> En cas de mise à niveau vers _Buster_, le système de fichiers n’est pas modifié, mais vous pouvez installer le paquet `usrmerge` pour lancer manuellement la conversion. 


Application du nouveau schéma de nommage pour les périphériques réseau
----------------------------------------------------------------------
Depuis _Stretch_, les nouvelles installations utilisent un nouveau schéma de nommage pour les périphériques réseau. Ainsi, les interfaces ne se nomment plus `eth0` ou `wlan0` mais ont plutôt des noms ressemblant à `enp1s1` ou `wlp3s0`.
    
À partir de _Buster_, ce changement s’appliquera aussi aux installations existantes. [Une section des notes de publication](https://www.debian.org/releases/buster/amd64/release-notes/ch-information.fr.html#migrate-interface-names) détaille comment anticiper ce changement ou, au contraire, conserver l’ancien schéma de nommage.


Adoption de Wayland dans la session par défaut
----------------------------------------------
Avec la mise à jour de GNOME vers la version 3.30 est venu un changement important : le passage de X.Org à Wayland comme serveur d’affichage par défaut pour cet environnement graphique. Si beaucoup de problèmes ont été [corrigés au fil du temps](http://libre-ouvert.toile-libre.org/?article224/test-de-gnome-wayland-sur-debian-pas-encore-tout-a-fait-ca), tout n’est pas encore parfait, par exemple [du point de vue de l’accessibilité](https://lists.debian.org/debian-accessibility/2019/02/msg00004.html). Ainsi, certains ont récemment fait connaître [leurs inquiétudes](https://jmtd.net/log/buster_wayland/) concernant l’ampleur de cette transition et le peu de discussion qui l’a accompagnée.
    
Un [rapport de bogue](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=927667) a été écrit pour lancer cette discussion et il a été convenu qu’il restait trop peu de temps avant la publication de _Buster_ pour revenir en arrière, et qu’on n’aurait pas assez de temps pour éprouver correctement un retour à X.Org. Par comparaison, [Fedora](https://fr.wikipedia.org/wiki/Fedora_(GNU/Linux)) propose Wayland par défaut depuis la version 25 (sortie en novembre 2016, basée sur GNOME 3.22), mais aussi [[SUSE Linux Enterprise Desktop]] 15 (juin 2018, GNOME 3.26) et [[RHEL]] 8 (mai 2019, GNOME 3.28).
    
Toutes les applications ne tournent pas encore nativement sous Wayland (Firefox, Thunderbird et GIMP, par exemple, sont en cours de conversion), mais grâce au composant XWayland qui fournit une couche intermédiaire vous n’y verrez que du feu.
    
Petite exception, toutefois, pour le gestionnaire de paquets graphique Synaptic, par exemple, dont le modèle de permission n’est pas compatible avec les règles de sécurité de Wayland : il ne se lancera que dans un mode que l’on pourrait qualifier de « lecture seule » :
![Avertissement au lancement de Synaptic](https://pix.toile-libre.org/upload/original/1562626625.png)
À la place, il vous faudra utiliser un outil graphique comme la Logithèque GNOME, ou les outils habituels en ligne de commande.


Attention au coup de la panne
-----------------------------
Les notes de publication avertissent au sujet d’un [problème potentiel lors du démarrage](https://www.debian.org/releases/buster/amd64/release-notes/ch-information.fr.html#entropy-starvation) des systèmes sous _Buster_. En effet, faute d’avoir suffisamment d’entropie, un système pourrait mettre jusqu’à plusieurs heures à démarrer. Un contournement par défaut est appliqué pour les systèmes AMD64 récents, et les notes de publication proposent d’autres options pour les autres architectures et les machines virtuelles.
    
Un [article passionnant](https://daniel-lange.com/archives/152-hello-buster.html) décrit ce problème en détail et énumère un grand nombre de solutions ainsi que leurs avantages et leurs inconvénients. Le [wiki de Debian](https://wiki.debian.org/BoottimeEntropyStarvation) résume également ces informations.


Quel navigateur Web privilégier ?
---------------------------------
Debian ne conseille pas véritablement _un_ navigateur, mais prévient que tous les navigateurs proposés dans Debian ne sont pas égaux en termes de suivi des failles de sécurité. De ce point de vue, il vaut mieux [privilégier](https://www.debian.org/releases/buster/amd64/release-notes/ch-information.fr.html#browser-security) un navigateur s’appuyant sur le paquet [webkit2gtk](https://webkitgtk.org/) (c.-à-d. [Luakit](https://packages.debian.org/buster/luakit), [Midori](https://packages.debian.org/buster/midori), [surf](https://packages.debian.org/buster/surf) ou [GNOME Web](https://packages.debian.org/buster/epiphany-browser)), Firefox ou encore Chromium.


Avancement des compilations reproductibles
==========================================
Le projet [Reproducible Builds](https://reproducible-builds.org/) a pour objectif de rendre la compilation des paquets déterministe afin de toujours obtenir le même binaire à partir des mêmes sources (et du même environnement de compilation). Cela permettra de vérifier que les paquets distribués sont bien ceux qu’ils prétendent être.
    
Même si _Buster_ serait en théorie reproductible à 93 %, [elle ne le sera en pratique qu’à 54 %](https://lists.debian.org/debian-devel/2019/03/msg00017.html). La différence s’explique par des paquets construits avant décembre 2016 qui n’ont jamais été reconstruits depuis, pour 24 % du total, ainsi que 12 % de paquets ayant reçu une [modification binaire pour une architecture spécifique](https://wiki.debian.org/binNMU) et affectés par le [bogue n^(o) 894441](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=894441).
    
![Évolution de reproductibilité depuis 2014](https://tests.reproducible-builds.org/debian/unstable/amd64/stats_pkg_state.png)


Un point sur les Debian Pure Blends
===================================
Les _Debian Pure Blends_ sont des variantes de Debian adaptées à certains groupes d’utilisateurs. Il ne s’agit pas de divergences (_forks_), mais bien d’ensembles de paquets composés uniquement à partir de paquets Debian. Cela signifie que vous pouvez installer ces variantes soit en utilisant les médias (images d’installation ou autonomes) fournis par chaque projet, soit en installant le méta‐paquet dédié sur une installation Debian classique.
    
Parmi les [nombreuses variantes](https://www.debian.org/blends/), on trouve notamment :
    
- [Debian Edu](https://blends.debian.org/edu/), spécialisée dans le domaine de l’éducation et dont la version 10 est sortie [un jour après _Buster_](https://www.debian.org/News/2019/20190707) ;
- [Debian Astro](https://blends.debian.org/astro/), pour les astronomes professionnels ou amateurs ;
- [Debian Accessibility](https://blends.debian.org/accessibility), pour les personnes en situation de handicap ;
- [Freedombox](https://wiki.debian.org/FreedomBox), pour les amateurs d’auto‐hébergement, dont la version 19.1 est sortie avec _Buster_.


Les titres auxquels vous avez échappé pour cette dépêche 
========================================================
![Teckel](https://framapic.org/jJ8Gdt8zRJ3A/L7uZVVoyBavH.jpg)
    
- « Le chien entre en scène » ;
- « Une distribution qui ne manque pas de mordant » ;
- « Le meilleur ami du libriste » ;
- « La version qui décOUAF ! » ;
- « La distribution qui tonne » ;
- « _Buster_, nom d’un OS » ;
- « _Buster_ lâche son os » ;
- « _Buster_, une distrib’ au poil ».
    
N’hésitez pas à proposer vos idées dans les commentaires !


Revue de presse (non exhaustive)
================================
- [[_Programmez !_] Sortie de Debian Linux 10 « Buster »](https://www.programmez.com/actualites/sortie-de-debian-linux-10-buster-29179)
- [[_informaticien.be_] Debian 10 Buster : la nouvelle version stable disponible](https://www.informaticien.be/index.ks?page=news_item&id=27780)
- [[_Next INpact_] Debian 10 disponible : Linux 4.19, Secure Boot, AppArmor et Wayland par défaut](https://www.nextinpact.com/brief/debian-10-disponible---linux-4-19--secure-boot--apparmor-et-wayland-par-defaut-9323.htm)
- [[_Developpez.com_] Buster, la version 10 de Debian Linux est disponible avec le support du Secure Boot](https://linux.developpez.com/actu/268842/Buster-la-version-10-de-Debian-Linux-est-disponible-avec-le-support-du-Secure-Boot-et-plus-de-paquets-que-Stretch/)
- [[_ZDnet_] _Debian 10 ‘Buster’ Linux arrives_](https://www.zdnet.com/article/debian-10-buster-linux-arrives/)


Et après ?
==========
Certains projets se dessinent déjà pour l’après _Buster_ :
    
- le [projet « 100 papercuts »](https://bits.debian.org/2019/06/100-papercuts-kickoff.html) sera lancé pendant [DebConf19](https://debconf19.debconf.org/) et visera à corriger tous les petits problèmes qui gâchent l’expérience utilisateur ;
- la [fin de la prise en charge de Python 2](https://www.debian.org/releases/buster/amd64/release-notes/ch-information.en.html#deprecated-components) ;
- la [fin des téléversements binaires](https://lists.debian.org/debian-devel-announce/2019/07/msg00002.html) ; dorénavant, seuls les téléversements de paquets source seront autorisés.
    
La version 11 de Debian s’appellera _[Bullseye](https://lists.debian.org/debian-devel-announce/2016/07/msg00002.html)_ (le [cheval](https://pixar.fandom.com/wiki/Bullseye) de [Woody](https://linuxfr.org/news/sortie-de-la-debian-gnulinux-30-woody)) et sortira… quand elle sera prête !
    
Bon, c’est pas tout ça, mais je fais quoi maintenant moi aujourd’hui : j’installe la nouvelle version de Debian ou je vais voir au cinéma le [nouvel épisode de _Toy Story_](https://fr.wikipedia.org/wiki/Toy_Story_4) ?
