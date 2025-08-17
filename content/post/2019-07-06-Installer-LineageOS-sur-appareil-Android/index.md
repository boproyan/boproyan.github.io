+++
title = 'Installer LineageOS sur son appareil Android'
date = 2019-07-06 00:00:00 +0100
categories = android
+++
Lorsque j’ai installé [LineageOS sur mon smartphone](https://linuxfr.org/news/mon-nouveau-smartphone-android-degooglise), j’ai suivi des tutoriels sans réellement comprendre ce que je faisais. Je voulais donc écrire cette dépêche pour éclairer celles et ceux qui voudraient tenter l’aventure et avoir plus de contrôle sur leur téléphone Android. Ce n’est donc pas un tutoriel mais plus un guide qui explique le fonctionnement et l’écosystème d’Android et de LineageOS.


![copies d'écran LineageOS](https://lineageos.org/assets/img/security.png)

----

[LineageOS](https://www.lineageos.org/)  
[F-Droid](https://f-droid.org/)  
[MicroG](https://microg.org/)  
[CHATONS (cloud et synchronisation)](https://chatons.org/fr)  
[Dégoogliser votre téléphone Android](https://wiki.zaclys.com/index.php/D%C3%A9googliser_votre_t%C3%A9l%C3%A9phone_Android)  
[Mon téléphone libre (site de l'auteur)](http://www.montelephonelibre.fr/)  

----

# Avant l'installation



## Qu'est-ce que LineageOS ?



LineageOS est un système pour téléphone et tablettes basé sur Android Open Source Project (AOSP) — Android tel que fournit par Google. Il peut être installé à la place du système Android qui est préinstallé sur les téléphones. Contrairement à ce système, maintenu (ou pas) par le fabriquant de l'appareil, il s'agit d'un projet communautaire. 
C'est un dérivé de Cyanogenmod créé en décembre 2016, quand la boite qui était derrière ce dernier, Cyanogen Inc., a annoncé l'arrêt du projet et de son infrastructure.


## Système alternatif : quelle différences entre un téléphone et un ordinateur classique ?


Lorsqu'on a déjà installé un ordinateur sous Linux ou avec un autre système, on se demande forcément à un moment ou à un autre : mais pourquoi est-ce si compliqué, qu'y a t-il de différent sur un téléphone android ?

Sur les ordinateurs de bureau ou portables classiques (PC), l'architecture est standardisée et le BIOS fournit la liste du matériel présent. Ainsi, un installateur de système d'exploitation pourra connaître et trouver le matériel sans problème particulier.


Pour les téléphones, c'est plus compliqué : l'architecture et le matériel sont souvent spécifiques pour chaque téléphone et il n'y a pas de moyen de détection. Le noyau des systèmes des téléphones doivent être compilés avec un arbre de périphérique (**device tree** : liste du matériel présent et à quel endroit il est).

Pour compliquer le tout, les constructeurs des cartes intégrées utilisées dans les téléphones adaptent en profondeur le noyau Linux et le rendent spécifique à leur matériel. Ce qui fait que chaque carte est fournie avec un noyau Linux spécifique et un ensemble de bibliothèques propriétaires pour faire fonctionner des périphériques plus ou moins essentiels (puce graphique, appareil photo, modem, puce wifi et bluetooth, etc.).

**Pour toutes ces raisons, il n'y a pas d'installateur universel, il y a autant d'images que de téléphones.**


## Android et la vie privée : qu'est ce qui pose problème ?

Voici quelques liens qui peuvent donner envie de mieux protéger sa vie privée :


- Infographie des [données envoyées à Google](https://e.foundation/wp-content/uploads/INFOGRAPHIE-V5.gif), en anglais.
- Achats, déplacements, enregistrements de voix... [J'ai fouillé dans les données que Google conserve sur moi depuis treize ans (et rien ne lui échappe)](https://mobile.francetvinfo.fr/internet/google/achats-deplacements-enregistrements-de-voix-j-ai-fouille-dans-les-donnees-que-google-conserve-sur-moi-depuis-treize-ans-et-rien-ne-lui-echappe_3254425.html).
- Documentaire [Nothing to hide](https://vimeo.com/193515863) - VO sous-titré en français, on a parlé de la suite [sur LinuxFr.org](https://linuxfr.org/news/documentaire-disparaitre-suite-de-nothing-to-hide-en-crowdfunding-derniers-jours).
- [Ce que collecte Google](https://framablog.org/2019/01/12/les-donnees-que-recolte-google-document-complet/), traduction par Framasoft de l'étude [Google Data Collection](https://digitalcontentnext.org/wp-content/uploads/2018/08/DCN-Google-Data-Collection-Paper.pdf) de l’équipe du professeur Douglas C. Schmidt, spécialiste des systèmes logiciels, chercheur et enseignant à l’Université Vanderbilt.
- Bilan de sécurité 2018 Android analysé par [Next-inpact](https://www.nextinpact.com/news/107754-android-bilan-securite-2018-positif-mais-imparfait.htm).

## Pourquoi installer LineageOS ?


### Avantages :
- protéger sa vie privée ;
- avoir plus de contrôle sur son téléphone ;
- avoir un téléphone mis à jour avec une version récente d'Android ;
- gagner en vitesse d'exécution et en espace libre (pas de surcouche constructeur).


### Inconvénients :
- perte de la garantie (bien que cette clause soit abusive, je vous laisse en discuter dans les commentaires) ;
- difficulté et risque de l'installation (possibilité de bloquer son téléphone en cas de mauvaise manipulation, cette page devrait vous aider à mieux comprendre ce que vous faites).


## Android est-il libre ?
Oui et non, le système de base Android est libre (**AOSP** : Android Open Source Project) mais les téléphones sont livrés avec des applications et bibliothèques propriétaires.


Voici généralement ce que l'on trouve dans un téléphone livré par un constructeur :


![Rom Stock](http://www.pixngraph.com/android-linuxfr/aosp-stock.png)


**Google Apps** : Applications fournies par Google (Play store, Gmail, Gmaps, etc.). Il est possible d'installer les [Google Apps sur LineageOS](https://en.wikipedia.org/wiki/List_of_Google_apps_for_Android).


Et voici ce que vous aurez après avoir installé LineageOS :


![Rom LineageOS](http://www.pixngraph.com/android-linuxfr/aosp-lineageos.png)


- Les pilotes matériels restent non libres ;
- l'installation d'applications non libres reste possible sur LineageOS (oui, vous pourrez continuer à utiliser Facebook sur LineageOS si vous le désirez !) ;
- il est également possible d'installer les Google Apps sur LineageOS, mais vous vous exposerez alors à nouveau votre vie privée. Je reparlerai plus tard de l'alternative microG.

## Qu'est ce qu'une ROM ?



On rencontre souvent le terme « ROM » (Read-Only Memory) pour parler de LineageOS. C'est un peu un abus de langage ici car on devrait plutôt parler de distribution. 
Une ROM est installée sur un téléphone en « flashant » une image, c'est-à-dire en copiant son contenu dans la partition système du téléphone, qui est en lecture seule dans les conditions habituelles d'utilisation. 
Par *ROM stock*, on désigne le système qui est préinstallé sur le téléphone. Toutes les autres ROMs sont qualifiées de *ROM custom*. On parle aussi de temps en temps de *firmware* (micrologiciel), bien qu'une distribution Android n'est pas si micro : elles avoisinent le giga-octet.

### Alternatives :



LineageOS est la ROMs alternative la plus répandue, pouvant s'installer sur le plus grand nombre de téléphones, mais il y en a d'autres :



- le projet [Replicant](https://replicant.us/) a pour but de construire une version d'Android entièrement libre (y compris les pilotes) ;
- [/e/](https://e.foundation/) se base sur LineageOS pour proposer une autre interface et des services qui respectent la vie privée et intègre microG;
- ROMs basées sur LineageOS, telles que [AICP](https://www.aicp-rom.com/), [Bliss](https://blissroms.com/) et un tas d'autres ; si vous arrivez à installer LineageOS, vous ne devriez pas rencontrer de problèmes pour tester d'autres ROMs basées dessus ;
- [AOSP](https://source.android.com/) elle-même, pour les téléphones pris en charge par Google ;
- des ROMs basées sur AOSP (et pas sur LineageOS), comme [AOSPExtended](https://www.aospextended.com/).

## Est-ce que je peux installer LineageOS sur mon téléphone ?



Tous les téléphones ne sont pas compatibles avec LineageOS, [voici la liste des téléphones officiellement compatibles](https://download.lineageos.org/). Pour connaître le nom précis de votre appareil, allez sur **Configuration** - **Nom de l'appareil**.


Votre téléphone n'est pas dans la liste ? Tout n'est pas perdu, des passionnés développent des ROM non officielles de belle qualité. Vous pouvez rechercher s'il existe une ROM pour votre téléphone sur les [forums XDA](https://forum.xda-developers.com).


Exemple :



- j'ai un LG G5, le nom de l'appareil est H850 ;
- il n'est pas dans la liste officielle LineageOS ;
- il existe une [ROM non officielle LineageOS 15.1](https://forum.xda-developers.com/lg-g5/development/rom-unofficial-lineageos-15-1-g5-t3806842) ;
- il existe une [ROM Beta non officielle LineageOS 16](https://forum.xda-developers.com/lg-g5/development/rom-unofficial-lineageos-16-0-g5-t3917647).


Cela peut paraître étrange de télécharger des fichiers d'installation à partir d'un forum, mais c'est comme cela dans le monde Android / LineageOS !

### Les versions LineageOS / Android


- CyanogenMod 12 (basé sur Android 5 - Lollipop)
- CyanogenMod 13 (basé sur Android 6 - Marshmallow)
- LineageOS 14 (basé sur Android 7 - Nougat)
- LineageOS 15 (basé sur Android 8 - Oréo)
- LineageOS 16 (basé sur Android 9 - Pixel)
- LineageOS 17 (basé sur Android 10 - Q)


# L'installation
Le but de cette partie est de comprendre ce que l'on fait lorsqu'on installe LineageOS, car bien souvent les tutoriels se limitent à une suite d'instructions à exécuter sans réelles explications.
Je ne vais donc pas détailler ici l'installation de chaque ROM, les étapes diffèrent selon chaque téléphone. Donc, après la lecture de cet article, je vous renvoie donc au tutoriel d'installation de la ROM pour votre téléphone.


## Le système de fichiers



Voici à quoi ressemble le système de fichiers sur un téléphone Android (il existe d'autres partitions, mais vous n'avez pas besoins de les connaitre pour comprendre l'installation d'une ROM).

![filesystem](http://www.pixngraph.com/android-linuxfr/filesystem1-description.png)


- Les parties _bootloader_ et _fastboot_ ne font pas vraiment partie du système de fichiers.
- Les partitions `/boot` et `/system` sont en lecture seule lorsque Android est démarré, c'est pourquoi il est impossible de supprimer les surcouches constructeurs et les Gapps d'un téléphone livré par les constructeurs.
- La partition `/data` contient les données et les applications, seule cette partition est accessible en écriture lorsque le téléphone est démarré.
- Il est possible d'accéder à la partition `/recovery` pour avoir accès en écriture aux autres partitions.


## Que sont les logiciels adb (Android Debug Bridge) et fastboot ?



Il est parfois nécessaire d'installer ces logiciels sur votre PC, ils permettent d'envoyer des commandes à votre téléphone et d'accéder à des partitions protégées en écriture.


![filesystem](http://www.pixngraph.com/android-linuxfr/filesystem1-adb+fastboot.png)


Pour l'installation sous Linux, il suffit généralement d'installer un paquet pour votre distribution.



Exemple pour debian/ubuntu :


```bash
apt-get install adb fastboot
```



Pour pouvoir lancer des commandes adb, il faut que votre téléphone les accepte et soit en mode développeur. Pour cela, vous devez aller dans :



- **Paramètres** - **À propos du téléphone** ;
- puis vous devez taper 7 fois sur la ligne **Numéro de build** (oui, oui, ce n'est pas une blague :)) ;
- retournez dans **Paramètres** - **{} Options pour les développeurs** ; 
- activez l'option **Débogage Android** (cette option est dans **Paramètres** ; - **Système** - **{} Options pour les développeurs** à partir d'Android 9 - Pie).


Voici quelques exemples de ce que vous pouvez faire avec ces commandes :



- **adb devices** : liste les téléphones reliés par USB à votre PC et prêts à recevoir des commandes ;
- **adb reboot recovery** : permet de redémarrer sur la partition `/recovery` ;
- **adb reboot bootloader** : permet de redémarrer en mode `fastboot` ;
- **adb install -l nomapplication.apk** : permet d'installer le _package_ ;
- **fastboot devices** : permet de lister les téléphones reliés par USB à votre PC et prêts à recevoir des commandes _fastboot_.


## Pourquoi déverrouiller le bootloader ?


Il faut déverrouiller le _bootloader_ pour pouvoir démarrer sur la partition `/recovery`. La manipulation dépendra entièrement du constructeur et du téléphone que vous possédez. Cela peut se faire très simplement ou alors passer par une procédure complexe (envoi de mail, code, etc.).


![filesystem](http://www.pixngraph.com/android-linuxfr/filesystem1-bootloader.png)


## Qu'est ce que TWRP ?



[TWRP](https://twrp.me/) est un programme qui s'installe dans la partition `/recovery`. Une fois installé, il vous permet, grâce à une interface graphique, de formater, de sauvegarder ou d'installer des images dans les partitions `/boot` et `/system`.


Bien souvent, depuis la page d'installation de la ROM LineageOS, des liens vers le logiciel TWRP à flasher spécifiques à votre téléphone sont disponibles.

**Remarque** : le projet TWRP propose des versions « live » permettant d'amorcer TWRP sans l'installer et d'y réaliser les opérations courantes (installation, injection d'une application système ou d'un patch).

![filesystem](http://www.pixngraph.com/android-linuxfr/filesystem1-twrp.png)


Pour démarrer sur TWRP, il faut utiliser une combinaison de touches qui dépend de votre téléphone (souvent bouton de démarrage + volume haut).


![TWRP](https://upload.wikimedia.org/wikipedia/commons/thumb/f/fe/TWRP_2.7.0.0.png/300px-TWRP_2.7.0.0.png)



TWRP est un logiciel libre, il possède une application permettant sa propre mise à jour simplifiée et aussi de se tenir informé des dernières versions du système.

## Installation de LineageOS
Et voilà, vous pouvez maintenant installer LineageOS : téléchargez la ROM, copiez-la sur une carte SD, rebootez sur `/recovery` en TWRP, [sauvegardez votre ROM actuelle](https://www.stechguide.com/take-twrp-backup-directly-pc-via-adb/), formatez les partitions, et enfin, installez votre nouvelle ROM !

![filesystem](http://www.pixngraph.com/android-linuxfr/filesystem1-lineageos.png)

## Quest-ce que le mode root ?
Après avoir démarré LineageOS, la partition `/system` n'est pas accessible en écriture. Rooter son téléphone permet de la rendre accessible.

![filesystem](http://www.pixngraph.com/android-linuxfr/filesystem1-root.png)


### Avantages du root :
- Pouvoir sauvegarder l'ensemble de son appareil ;
- faire fonctionner des logiciels qui ont besoin d'accéder à la partition `/system` (MicroG ou logiciels de backup par exemple).


### Inconvénients du root :
- certaines applications ne fonctionnent plus sur un appareil rooté ;
- problèmes de sécurité ?


### Installation de Magisk
Magisk est un logiciel qui permet de faire un pseudo root et de cacher à certaines applications que le téléphone est rooté.
[Magisk](https://magiskmanager.com) est un logiciel libre, une application permet de le configurer et d'installer les dernière versions.


## Qu'est ce que MicroG ?


Si vous voulez dégoogliser votre téléphone, vous ne voudrez certainement pas installer les Google Apps (GApps). Cependant, certaines applications dépendent des services Google. 
On pourrait penser que les [Open Gapps](https://opengapps.org/) font l'affaire, mais ce n'est pas le cas : elles ne font que fournir des logiciels propriétaires de Google. 
On pourrait également penser que l'on peut tout simplement s'en passer. C'est effectivement la meilleure solution, mais, parfois, certaines applications dépendent des Google Apps et ne fonctionnent pas sans.

[MicroG](https://microg.org/) fournit une alternative libre à certaines API utilisées par ces applications et peut-être installé à la place des GApps. Il faut malgré tout rester conscient que le bon fonctionnement de microG dépend du bon vouloir de Google et de l'évolution de ses API.


![filesystem](http://www.pixngraph.com/android-linuxfr/filesystem1-microg.png)


**Remarque** : En plus des ces API, MicroG fournit un moyen de se géolocaliser à l'aide du Wifi et des antennes téléphoniques, ce qui n'est pas fourni de base avec LineageOS (mais qui est une fonctionnalité des GApps). 
Cette fonctionnalité rend MicroG intéressant même pour les gens qui souhaitent totalement éviter Google car cette géolocalisation vient en complément et accélère parfois la localisation GPS. 
L'utilisateur devra choisir et activer des fournisseurs de géolocalisation dépendant ou non d'une connexion au réseau (Apple, Mozilla, bases de données locales) dans les paramètres de MicroG.

### ROM LineageOS for microG


Tous les téléphones pris officiellement en charge par LineageOS sont pris en charge par le projet [LineageOS for microG](https://lineage.microg.org/). Ces images fournissent LineageOS + microG + F-Droid de base. Cette solution simplifie grandement l'installation et doit être privilégiée.

### Étapes d'installation de microG


Si votre ROM n'est pas officiellement supportée, microG peut aussi être installé sur un système LineageOS existant, mais ce n'est pas simple.

Voici un résumé des étapes à réaliser pour l'installation de microG sans utiliser LineageOS for microG :

- rooter l'appareil ;
- ajouter un [dépôt à F-Droid](https://microg.org/download.html) ;
- installer les applications microG (GmsCore, UnifiedNlp, FakeStore, etc) ;


**Remarque** : Une application permet de savoir quels services fonctionnent et de configurer microG.

## Mettre à jour sa ROM
Si vous utilisez une ROM LineageOS (ou MicroG) officielle, vous devriez être notifié lorsqu'une mise à jour est disponible. L'installation (OTA : Over The Air) se fait donc très simplement.


Par contre, si vous êtes sur une version non officielle, je n'ai pas trouvé de méthode recommandée pour cette tâche. Je vous soumets donc la méthode que j'utilise et qui a fonctionné pour moi jusqu'à aujourd'hui. N'hésitez pas à en soumettre d'autres ou à la critiquer.

- Téléchargez la nouvelle ROM et placez là sur la carte SD ;
- Rebootez en mode recovery sous TWRP ;
- Installez la nouvelle ROM ;
- Redémarrez l'appareil. Le premier démarrage est en général long.

# Après l'installation


LineageOS est vraiment un système agréable à utiliser et complet, mais voici quelques suggestions de ce que vous pourriez faire après avoir installé LineageOS pour l'enrichir et le personnaliser.

## Les magasins d'applications (F-Droid - Aurora)
Que serait Android sans ses applications ? Et, pour installer des applications, il faut un magasin d'applications (Store). Si vous n'avez pas installé les Google Apps, vous n'avez plus Google Play, mais heureusement, des solutions alternatives existent.


### F-droid - Magasin d'applications libres
[F-droid](https://f-droid.org/fr/) est un magasin d'applications libres pour Android. Pour l'installer, vous devez utiliser un navigateur, télécharger l'APK et l'installer.
![f-droid](https://f-droid.org/repo/org.fdroid.fdroid/en-US/phoneScreenshots/screenshot-dark-home.png)

### Aurora - Utiliser Google Play sans Google Play
[Aurora](https://f-droid.org/fr/packages/com.aurora.store/) (disponible sur f-droid) permet d'installer des applications provenant de Google Play sans utiliser de compte Google. Vous pouvez donc installer des applications propriétaires de façon anonyme grâce à ce store.


- Cela ne fonctionne pas toujours du premier coup, il faut parfois patienter avant de pouvoir installer une application (renouvellement de Token) ;
- Aurora vous demande un compte Google au démarrage, mais vous pouvez ignorer cette étape pour rester anonyme


## Quelques suggestions d'applications libres


### Navigation web : Firefox et DuckDuckGo
LineageOS est livré avec le navigateur Android. Mais, depuis F-droid, vous pouvez facilement installer le navigateur [DuckDuckGo](https://f-droid.org/fr/packages/com.duckduckgo.mobile.android/), qui propose également un widget de recherche ou alors Firefox (baptisé ici [Fennec F-Droid](https://f-droid.org/fr/packages/org.mozilla.fennec_fdroid/)). 
Les plus puristes seront peut-être intéressés par [Icecat Mobile](https://f-droid.org/en/packages/org.gnu.icecat), un dérivé des versions Firefox prises en charge à long terme fourni avec les extensions LibreJS (qui bloque les scripts non-libres) et Tor.


### Cartographie et navigation : OsmAnd et Maps.Me


Il existe beaucoup d'alternatives de qualité basées sur OpenStreetMap pour remplacer Google Maps. 
La plus connue est [OsmAnd](https://f-droid.org/fr/packages/net.osmand.plus/), qui est un vrai couteau suisse de la navigation mobile et permet une navigation, des recherches et des calculs d'itinéraires (vélo, piéton, voiture, transports en commun) hors-lignes.
![OsmAnd](https://upload.wikimedia.org/wikipedia/commons/thumb/8/80/Osmand_street_routing.png/640px-Osmand_street_routing.png)
Une application plus rapide et plus légère (utilisant des cartes non-vectorielles), [Maps.me](https://f-droid.org/en/packages/com.github.axet.maps), pourra également intéresser certaines personnes.


### Photographies : OpenCamera - FreeDCam - Camera Roll
LineageOS est livrée avec une application pour prendre des photos. Mais celle-ci fera peut-être pâle figure par rapport à celle livrée par le constructeur et qu'il est maintenant impossible d'installer.


[OpenCamera](https://f-droid.org/fr/packages/net.sourceforge.opencamera/) sera sans doute une alternative plus complète.
Essayez également [FreeDCam](https://f-droid.org/fr/packages/troop.com.freedcam/) qui propose encore plus de fonctionnalités (pas sûr qu'elle fonctionne pour autant de téléphones qu'OpenCamera) mais qui est également plus complexe.
![freedcam](http://www.pixngraph.com/android-linuxfr/freedcam.png)

Enfin, pour organiser vos photos, testez [Camera Roll](https://f-droid.org/en/packages/us.koller.cameraroll/) !


### Gestion des fichiers : Amaze
[Amaze](https://f-droid.org/fr/packages/com.amaze.filemanager/) est un gestionnaire de fichiers complet (accès à des serveurs SMB, FTP et SFTP).


![Amaze](https://fscl01.fonpit.de/userfiles/6898953/image/comment-degoogliser-android-amaze-images-00-w782.jpg)


### Gestion des mails : K9mail
[K9mail](https://f-droid.org/en/packages/com.fsck.k9/) est une application mail éprouvée. Elle peut remplacer E-mail Android mais également Gmail.


### Calendrier et contacts : DAVx⁵, Nextcloud, Etar et OpenTasks


Un [tutoriel](https://dutailly.net/synchroniser-son-agenda-entre-thunderbird-sous-linux-et-android) très complet a été rédigé par [Ysabeau](https://linuxfr.org/news/synchronisation-thunderbird-android) pour synchroniser calendrier et contact avec [Davx^5](https://f-droid.org/fr/packages/at.bitfire.davdroid/) (anciennement DavDroid).


- Une application pour la gestion des **contacts** est livrée avec LineageOS.
- Pour le **calendrier**, on pourra utiliser Agenda, livré avec LineageOS ou [Etar](https://f-droid.org/en/packages/ws.xsoh.etar/)
- [OpenTasks](https://play.google.com/store/apps/details?id=org.dmfs.tasks&hl=fr&showAllReviews=true) - non disponible sur F-Droid pourra gérer et synchroniser vos **TODO List**

### Sauvegardes : OandBackup, SMS Backup + et Nextcloud 


L'application [OandBackup](https://f-droid.org/en/packages/dk.jens.backup/) permet une sauvegarde exhaustive, par application, du téléphone. Même si son interface est un peu austère, vous pouvez tout sauvegarder avec, à condition d'avoir un téléphone rooté.
![Oandbackup](https://framalibre.org/sites/default/files/OandbackupScreen1.jpg)

L'application [SMS Backup+](https://f-droid.org/fr/packages/com.zegoggles.smssync/) pourra être utilisée en complément d'OandBackup pour sauvegarder ses textos dans une boite mail. Elle ne nécessite pas un téléphone rooté.



Côté photos et documents, [Nextcloud](https://f-droid.org/en/packages/com.nextcloud.client) ou [OwnCloud](https://f-droid.org/fr/packages/com.owncloud.android/), qui permettent d'accéder à un espace en ligne (cloud) et d'y téléverser les photos qui sont prises dès que le téléphone est connecté (c'est bien sûr optionnel).

Enfin, il est toujours possible de sauvegarder entièrement le téléphone, même s'il n'est pas rooté, [à l'aide de TWRP](https://www.stechguide.com/take-twrp-backup-directly-pc-via-adb/).

### Et puis, en vrac :
- [VLC](https://f-droid.org/fr/packages/org.videolan.vlc/) : regarder des vidéos et écouter de la musique ;
- [Red Moon](https://f-droid.org/fr/packages/com.jmstudios.redmoon/) ou [Night light](https://f-droid.org/en/packages/com.corphish.nightlight.generic/) : filtre les lumières bleues pour le bien de vos yeux ;
- [KDE Connect](https://f-droid.org/en/packages/org.kde.kdeconnect_tp) : permet de recevoir et répondre à ses textos avec son ordinateur, de partager le presse papier, transférer des fichiers, partager les notifications dans un sens comme dans l'autre, piloter son ordinateur à partir du téléphone (souris, clavier, lecteur de musique), ouvrir un lien sur le téléphone depuis l'ordinateur et plus encore : une application très pratique et assez complète qui fonctionne aussi sur d'autres environnement de bureau que Plasma ;
- [NewPipe](https://f-droid.org/en/packages/org.schabi.newpipe/) : une belle alternative à l'application Youtube, qui permet de chercher, visionner les vidéos, les écouter en arrière plan et les télécharger pour les regarder ou les écouter plus tard. Tout ça sans les publicités. Indisponible sur le Play Store ;
- [AfWall+](https://f-droid.org/en/packages/dev.ukanth.ufirewall) : un par-feu permettant un contrôle fin par application et type de connexion (pratique notamment pour limiter l'utilisation des données mobiles sur un forfait restreint ! Nécessite un téléphone rooté) ;
- [ClipStack](https://f-droid.org/fr/packages ;/com.catchingnow.tinyclipboardmanager/) : un bon gestionnaire de presse-papier.
- [ForceDoze](https://f-droid.org/en/packages/com.suyashsrijan.forcedoze) : pour forcer votre téléphone à dormir quand son écran est éteint. À utiliser avec précautions ;
- [Drowser](https://f-droid.org/en/packages/com.jarsilio.android.drowser) : pour tuer les applications voulues lorsque l'écran du téléphone s'éteint (idem !) ;
- [Barcode Scanner](https://f-droid.org/en/packages/com.google.zxing.client.android) : pour scanner les codes barres et partager des données avec des QR codes ;
- [OpenFoodFacts](https://f-droid.org/en/packages/openfoodfacts.github.scrachx.openfood) et [OpenBeautyFacts](https://f-droid.org/es/packages/openfoodfacts.github.scrachx.openbeauty) : pour scanner et évaluer des produits du commerce avec la base de données [Open Food Facts](https://world.openfoodfacts.org/) ; 
- [Sky Map](https://f-droid.org/en/packages/com.google.android.stardroid) : une application anciennement éditée par Google et maintenant gérée par la communauté pour observer le ciel ;
- [SatStat](https://f-droid.org/en/packages/com.vonglasow.michael.satstat) : pour consulter la boussole du téléphone, afficher les données GPS et de divers capteurs du téléphone (accélération, rotation, champs magnétique, orientation, lumière, proximité, température, pression) ;
- [Dessin](https://f-droid.org/wiki/page/com.simplemobiletools.draw) : une application de dessin rudimentaire. Pratique pour noter les scores pendant une partie de jeu de société ;
- [Audio Recorder](https://f-droid.org/en/packages/com.github.axet.audiorecorder/) : enregistrer du son avec le microphone du téléphone ;
- [Signal](https://play.google.com/store/apps/details?id=org.thoughtcrime.securesms&hl=fr) - non disponible sur F-Droid (le développeur principal s'y oppose, est hostile aux dérivés se connectant au serveur officiel et l'application inclut une bibliothèque Google non-libre) : permet d'échanger des messages et des médias chiffrer avec ses contacts. Voir aussi : [Riot.im](https://f-droid.org/en/packages/im.vector.alpha) (Matrix), [Rocket Chat](https://f-droid.org/en/packages/chat.rocket.android), [Mattermost](https://f-droid.org/en/packages/com.mattermost.mattermost), [Delta Chat](https://f-droid.org/en/packages/com.b44t.messenger) (messagerie instantanée par courriels) et [Telegram](https://f-droid.org/en/packages/org.telegram.messenger).


# Conclusion et projets



J'ai beaucoup appris en rédigeant cette dépêche. Merci également à toutes les personnes qui y ont participé.

La liste des applications présentées dans cette dépêche est loin d'être complète, je compte sur vous pour partager vos trouvailles dans les commentaires.


J'ai pour projet de faire quelques diapos à partir de cette dépêche dans le but d'organiser des « Expositions » sur Android et la vie privée. Beaucoup de personnes y sont aujourd'hui sensibles (même si beaucoup ne le sont pas également ;)).

Pour finir, j'aimerais également étudier la possibilité d'importer des téléphones reconditionnés, de les installer avec LineageOS et de les vendre en France avec une garantie. Cela m'a été demandé par des proches, et j'ai créé le site web [http://www.montelephonelibre.fr](http://www.montelephonelibre.fr/) (soyez indulgents, le site a été créé pour cette dépêche, il est tout neuf :)). N'hésitez pas à m'envoyer des messages et à vous inscrire à la newsletter pour avoir des nouvelles !
