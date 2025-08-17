+++
title = 'https-tor-vpn-de-quoi-est-ce-que-ca-protege-exactement'
date = 2019-11-06 00:00:00 +0100
categories = vpn
+++
URL:     https://linuxfr.org/news/https-tor-vpn-de-quoi-est-ce-que-ca-protege-exactement
Title:   HTTPS, Tor, VPN : de quoi est‐ce que ça protège exactement ?
Authors: Collectif
         Ysabeau, antistress, mathrack, tisaac, Davy Defaud, ZeroHeure, voxdemonix, Arcaik, Xavier Claude, Nils Ratusznik, palm123, Yves Bourguignon, Nicolas Boulay, dzecniv, sebas, gle, Florent Zara, Trollgouin, Maderios, aurel, Bruno, Benoît Sibaud, Bisaloo, koon, Coles, Olivier HUMBERT, Pierre Tramal, Poupy-Le-Vicieux, jean_r et cracky
Date:    2018-03-02T17:03:21+01:00
License: CC by-sa
Tags:    https, tls, tor, vpn, vie_privée, vie-privée et privacy
Score:   6


La dépêche « [Protéger sa vie privée sur le Web, exemple avec Firefox](/news/proteger-sa-vie-privee-sur-le-web-exemple-avec-firefox) » (février 2018) a ouvert [des questions](/news/proteger-sa-vie-privee-sur-le-web-exemple-avec-firefox#comment-1731111) sur la protection par HTTPS, Tor et VPN. Ces techniques protègent, mais contre quoi et dans quelles limites ? Cet article essaie de l’expliquer plus en détails. N’ayez pas l’illusion d’être totalement protégés en utilisant l’une d’elles : soit, elle ne permet pas vraiment ce que vous croyez, soit il faut l’utiliser d’une certaine façon ou la compléter d’autres précautions pour avoir le résultat attendu.

En effet, pour bien dissimuler votre navigation il faut tenir compte des limites techniques. C’est comparable au chiffrement de vos courriels (même de bout en bout) qui peut être insuffisant pour dissimuler entièrement votre message à un espion. Par exemple, si vous chiffrez votre courriel, mais que celui-ci a pour objet « j’ai des morpions » ou que vous l’adressez à `sos-morpions@sante.gouv.fr`, le contenu et les pièces jointes de votre message ont beau être chiffrés, un espion peut quand même en avoir une relativement bonne idée, car les [métadonnées](http://libre-ouvert.toile-libre.org/?article180/rassurez-vous-il-ne-s-agit-que-de-metadonnees-vraiment-mais-ne-s-agit-il-pas-que-de-votre-vie-privee-en-fait) et l’objet ne sont pas chiffrés.

![I feel like a secret detective, par binkle_28, sous CC BY-SA 2.0](https://live.staticflickr.com/2145/2333243258_a2d1976efd_b.jpg) _I feel like a secret detective, par binkle_28, sous CC BY-SA 2.0_

----


----

#HTTPS


HTTPS n’a pas été conçu pour dissimuler l’identité (techniquement, le _hostname_, ou nom d’hôte) du site accédé.

## Explications techniques
### Fonctionnement et limites du procédé
HTTPS consiste à chiffrer les échanges HTTP avec [TLS](https://fr.wikipedia.org/wiki/Transport_Layer_Security) (originellement, du SSL). Toute la requête HTTPS est donc chiffrée, le chemin mais aussi le nom d’hôte (contenu dans l’en‑tête `Host`). Cependant, le nom d’hôte (l’adresse du site Internet) peut fuiter d’autres manières. Par exemple, parce que la requête DNS du nom est faite juste avant la requête HTTP. Mais ce qui le donne le plus souvent est l’extension [SNI](https://fr.wikipedia.org/wiki/Server_Name_Indication), qui donne justement le nom en clair à l’initiation de la connexion TLS.
    
SNI n’a pas comme objectif principal de détruire la vie privée mais simplifie le fonctionnement quand la communication est chiffrée. Il permet à un serveur Web qui possède plusieurs certificats de présenter celui que le client attend. Il permet aussi à un répartiteur de charge d’envoyer la requête sur le bon serveur sans devoir déchiffrer le TLS (et peut donc gérer plus de connexions). Une nouvelle version, Encrypted SNI, permet justement de cacher ce nom de domaine. Elle est en cours de déploiement par les navigateurs et les serveurs Web.
    
Le HTTPS ne cache pas non plus l’adresse IP de destination.

### Attaque de l’homme du milieu
Il est relativement aisé de procéder à une attaque [[MITM]] et ainsi d’accéder au contenu de la communication en clair : votre employeur qui a installé le certificat de son autorité de certification interne sur votre poste (vous chiffrez alors sans le savoir avec la clé de votre employeur…), votre FAI ou une agence gouvernementale qui a accès aux certificats d’une autorité supposée de confiance, etc. Le système de « sécurité » de votre employeur peut aussi être obsolète, on a pu voir des boîtes noires communiquer en SSL et non plus en TLS vers Internet, ce qui peut entraîner des fuites de données en plus de celle extraite du serveur mandataire (proxy) menteur.

## Exemples
Mettons de côté l’attaque par déchiffrement décrite ci‑dessus, et supposons qu’un espion qui ne peut pas déchiffrer ma communication souhaite en savoir plus sur mes activités en ligne, voici des exemples de ce qu’il pourra observer.
    
Lorsque je navigue en HTTPS sur Wikipédia, cet espion qui intercepterait ma requête saurait que j’accède à l’encyclopédie et pourrait en déduire que je la consulte. L’espion ne pourrait cependant pas pouvoir savoir quelle page de Wikipédia je consulte. Par conséquent, il ne pourrait pas savoir sur quel sujet je me renseigne étant donné que Wikipédia est un site généraliste.
    
En revanche, dans le cas où je me connecte, toujours en HTTPS, à un site spécialisé, le sujet qui m’occupe pourrait être déduit de la thématique du site car l’identité du site est elle‑même connue. Par exemple, le sujet qui m’intéresse pourrait être inféré à partir de l’identité du site si je consulte `www.j-ai-des-morpions.org`.

## Conseil technique
L’extension multi‐navigateur [HTTPS Everywhere](https://www.eff.org/https-everywhere/), développée (sous licence libre GPL version 2+) collaborativement par l’_Electronic Frontier Foundation_ ([page de don](https://supporters.eff.org/donate)) et _The Tor Project_ ([page de don](https://donate.torproject.org/pdr)), permet de recourir, à chaque fois que cela est possible, à HTTPS.
    
Cela évite :
    
- d’une part, que le contenu transféré puisse être lu par ceux qui les intercepteraient (exemple : vos identifiants et mot de passe de connexion) ;
- d’autre part, que ce contenu puisse être modifié par ces mêmes personnes.
    
Mais, si vous souhaitez dissimuler l’identité du site accédé, cette technique doit être couplée à l’une de ces deux techniques : Tor ou VPN et qu’un serveur tiers puisse prendre facilement la place du serveur que vous voulez visiter (dans le cas d’un DNS menteur, par exemple).

# Tor


Tor, est un réseau décentralisé qui permet de faire passer sa connexion par un circuit composé de trois relais (serveurs) différents. Ainsi le premier sait uniquement d’où vient le trafic, l’intermédiaire ne sait rien, et le dernier sait uniquement où va le trafic. Par conséquent, nul ne peut connaître à la fois l’expéditeur et le destinataire.
    
Tor n’est pas limité au Web uniquement, mais permet l’utilisation de tout protocole réseau basé sur [TCP/IP](https://fr.wikipedia.org/wiki/Transmission_Control_Protocol).

## Explications techniques
Le recours à Tor peut recouvrir deux hypothèses :
    
1. l’accès à un site Web « classique » (c’est-à-dire extérieur à Tor) en passant par le réseau Tor pour bénéficier d’un anonymat renforcé ; 
2. l’accès à une version du site proposée au sein même de Tor (un tel site est doté d’une adresse en _.onion_) pour bénéficier d’un anonymat encore accru et/ou contourner la censure de sites.

## Exemple
Sans quitter Tor, vous pouvez vous rendre sur Facebook à l’adresse [https://facebookcorewwwi.onion](https://en.wikipedia.org/wiki/Facebookcorewwwi.onion). C’est l’adresse d’un serveur qui appartient à Facebook et qui n’est accessible que depuis le réseau Tor. L’on sait que Facebook peut servir à mobiliser, à informer et à s’informer dans les pays répressifs : ce serveur peut permettre de contourner une éventuelle censure (plus _[ici](https://www.lemonde.fr/pixels/article/2014/11/07/facebook-s-invite-sur-le-reseau-anonyme-tor_4518801_4408996.html)_).

##Conseil technique
Il est facile de mal utiliser Tor. Par exemple, si les requêtes de DNS passent par le réseau « normal », on peut en déduire ce que vous tentez de joindre.
    
C’est pourquoi il est très fortement conseillé d’utiliser le [Tor Browser](https://www.torproject.org/download/download-easy.html.en), un navigateur libre, multi‐système et développé par le [_Tor Project_](https://torproject.org/). Il est basé sur [Firefox ESR](https://donate.mozilla.org/fr/) et développé grâce à la collaboration (notamment financière) de Mozilla. Il inclut par défaut des extensions comme HTTPS Everywhere (évoquée ci‑avant) uBlock Origin ou NoScripts, visant à protéger la vie privée et à supprimer les traces que vous pourriez laisser sur le Web.
    
Il faut aussi rappeler que les nœuds de sorties peuvent voir le trafic à destination des sites que vous visitez. Celui-ci peut intercepter votre trafic s’il est en clair (par exemple, si vous n’utilisez pas HTTPS).

## Dons
Si vous souhaitez soutenir Tor, en France, vous [pouvez faire un don](https://nos-oignons.net/Donnez/index.fr.html) à l’association [Nos oignons](https://nos-oignons.net/), qui maintient un certain nombre de nœuds de sorties. Vous pouvez aussi donner directement au [Tor Project](https://donate.torproject.org/pdr).

#VPN (Virtual Private Network)


Un VPN fait « sortir » votre connexion Internet (pas seulement le Web) à une autre adresse par un tunnel chiffré. Il est donc impossible d’intercepter des communications en clair à la sortie de votre machine.

##Explications techniques
Votre machine établit un tunnel chiffré avec un serveur VPN et route tout le trafic par le tunnel, sauf le trafic chiffré lui-même (eh oui, sinon ça ne marcherait pas).
    
Le serveur VPN se charge alors d’acheminer votre trafic à bon port via sa connexion Internet. Tout le trafic sur votre interface réseau se résumera à une connexion [UDP](https://fr.wikipedia.org/wiki/User_Datagram_Protocol) ou [TCP](https://fr.wikipedia.org/wiki/Transmission_Control_Protocol), au choix, à un seul serveur (celui du service VPN).
    
Dans le cas d’[OpenVPN](https://fr.wikipedia.org/wiki/OpenVPN), c’est le serveur qui décide des routes à configurer sur le client.
    
Même si l’on en parle souvent pour changer d’adresse IP, un réseau privé virtuel (_Virtual Private Network_) peut fournir un réseau local virtuel. Donc, vous pouvez par exemple installer un serveur chez vous, mettre un VPN dessus et, une fois connecté, accéder à votre réseau comme si vous étiez à la maison.
    
Attention au [DNS Leak](https://www.dnsleaktest.com/results.html) : si vous contactez les serveurs DNS de votre FAI, ça revient à leur donner les sites que vous visitez… autant rester en HTTPS !

##Exemples
Si vous achetez un abonnement chez un fournisseur de VPN par crainte de voir vos données capturées par les méchants FAI, le VPN permettra effectivement de cacher votre trafic de votre FAI. Sur Internet, votre adresse IP publique aura changé. Cependant, c’est maintenant l’opérateur du service VPN qui peut regarder votre trafic !
    
Il est donc crucial de bien [se renseigner sur la politique de protection des données](https://le-routeur-wifi.com/pourquoi-pas-utiliser-vpn-gratuits/) du fournisseur VPN que vous choisirez. Certains fournisseurs de service VPN (souvent gratuits) procèdent en effet au même type de récolte/revente des données que les FAI : passer par leur service revient donc à ne rien utiliser du tout ! 
    
Si l’idée vous prenait d’utiliser un réseau Wi‑Fi public qui n’est pas chiffré, comme on en trouve dans certains bars, cafés, restaurants, gares et aéroports, alors les communications en clair (sans crypto) peuvent être interceptées et analysées. La meilleure solution, c’est, soit d’éviter ces connexions, soit d’utiliser des protocoles de communications chiffrés avec vérification de la clé (par exemple, HTTPS signé ou SSH). Dans la plupart des cas, les conditions d’utilisation de ces points d’accès devraient vous refroidir. Mais si vous devez absolument utiliser des connexions non chiffrées, alors un VPN est le bienvenu pour protéger vos communications.

##Conseil technique
* utiliser ce [script pour installer OpenVPN](https://github.com/Angristan/OpenVPN-install) ou [PiVPN](http://www.pivpn.io/) ;
* dans le futur, [WireGuard](https://en.wikipedia.org/wiki/WireGuard) sera peut-être une solution intéressante ; pour le moment, il n’en existe encore qu’une version expérimentale du code.

Utilisation conjointe de Tor et de HTTPS
========================================
Pour voir l’effet de l’utilisation conjointe de Tor et de HTTPS, consultez  [ce site](https://www.eff.org/pages/tor-and-https). Ci‑dessous, la traduction des instructions que vous trouverez en anglais sur le site.
    
Les données potentiellement visibles comprennent : le site que vous visitez (_site.com_), votre nom d’utilisateur et mot de passe (_user/pw_), les données que vous transmettez (_data_), votre adresse IP (_location_) et si vous utilisez ou non Tor (_tor_) :
    
- cliquez sur le bouton « Tor » pour voir les données visibles par les espions lorsque vous utilisez Tor ; le bouton devient vert pour indiquer que Tor est allumé ;
- cliquez sur le bouton « HTTPS » pour voir les données visibles par les espions lorsque vous utilisez HTTPS ; le bouton devient vert pour indiquer que HTTPS est activé ;
- lorsque les deux boutons sont verts, vous voyez les données visibles par les espions lorsque vous utilisez les deux outils ;
- lorsque les deux boutons sont gris, vous voyez les données visibles par les espions lorsque vous n’utilisez aucun des deux outils.

# Annexe : I2P, l’Invisible Internet Project
[I2P](https://geti2p.net) est une autre solution pour surfer de manière anonyme. Quand l’objectif de Tor est d’abord d’accéder à l’Internet « normal » de manière anonyme et ensuite de pouvoir se connecter à des sites accessibles seulement au réseau Tor (_Tor hidden services_), I2P permet, quant à lui, en premier lieu, d’accéder aux sites `.i2p` (ainsi que d’en publier) et, dans une moindre mesure, (il existe moins de portes de sorties) d’accéder à tout l’Internet. Quelques applications sont également accessibles : forum, tchat, partage de documents, téléchargement de torrents ou courriel interne au réseau.
    
Vu de loin, I2P fonctionne sur les mêmes principes que Tor, par couches de chiffrement successives. Basé sur un fonctionnement pair à pair, I2P est plus décentralisé. Ainsi, sa recherche initiale de pairs ne repose pas sur une liste connue à l’avance, et est adéquate pour un téléchargement de torrents.
    
Tor est beaucoup plus utilisé, la question se pose de savoir si I2P pourrait passer à l’échelle. Tor a fait l’objet de plus de recherche et son chef de projet est connu. À l’inverse, les développeurs d’I2P sont uniquement connus par leurs pseudonymes et ils considèrent ne pas avoir eu assez de revues de code par la communauté scientifique pour valider le système et passer officiellement à la fameuse version 1.
    
C’est en évaluant les différences et le niveau de sécurité désiré que l’on opte pour l’une ou l’autre solution.
    
Le client officiel est en Java et il existe le client [i2pd](https://github.com/PurpleI2P/i2pd), écrit en C++ pour une meilleure consommation des ressources.

# Annexe : Internet ne s’arrête pas au navigateur !
Si vous utilisez Debian et le gestionnaire de paquets [APT](https://fr.wikipedia.org/wiki/Advanced_Packaging_Tool), vous utilisez peut‐être le protocole HTTP pour contacter un dépôt et ainsi récupérer les mises à jour et les paquets à installer. Cependant, cette utilisation est potentiellement risquée ([attaque par rejeu](https://fr.wikipedia.org/wiki/Attaque_par_rejeu), _cf._ [1](https://unix.stackexchange.com/questions/317698/yum-install-http-is-this-safe) et [2](https://isis.poly.edu/~jcappos/papers/cappos_mirror_ccs_08.pdf) ou [attaque de l’homme du milieu](https://fr.wikipedia.org/wiki/Attaque_de_l%27homme_du_milieu), _cf._ le récent [CVE-2019-3462](https://lists.debian.org/debian-security-announce/2019/msg00010.html)). Le paquet [apt-transport-https](https://packages.debian.org/search?keywords=apt-transport-https) permet de chiffrer la partie HTTP de ce trafic Internet. Si vos miroirs habituels sont compatibles, ouvrez votre fichier `/etc/apt/sources.list` et remplacez `http` par `https` :
    
    deb https://foo distro main

Les paquets [apt-transport-tor](https://packages.debian.org/search?keywords=apt-transport-tor) et [onionbalance](https://packages.debian.org/search?keywords=onionbalance) vous permettent de faire passer les mises à jour et les paquets par le réseau Tor. Ainsi, pour _Stretch_, votre fichier `/etc/apt/sources.list` devrait simplement contenir :
    
    deb  tor+http://vwakviie2ienjx6t.onion/debian          stretch            main
    deb  tor+http://vwakviie2ienjx6t.onion/debian          stretch-updates    main
    deb  tor+http://sgvtcaew4bxjd7ln.onion/debian-security stretch/updates    main

N. D. A. : par sécurité, vous devriez vérifier les adresses en vous appuyant sur d’autres sources ! [https://onion.debian.org/](https://onion.debian.org/)


Vous pouvez également utiliser l’extension expérimentale [TorBirdy](https://trac.torproject.org/projects/tor/wiki/torbirdy) pour le client de messagerie Thunderbird. Ainsi, la connexion entre votre machine et votre serveur de messagerie transitera par le réseau Tor. Votre serveur de courriel n’aura donc pas connaissance de votre localisation.
    
Si le service Tor est actif, vous pouvez utiliser [torsocks](https://gitweb.torproject.org/torsocks.git/refs/) (vérifiez dans les dépôts de votre distribution) pour forcer un logiciel à faire passer son trafic par Tor, par exemple :
    
    torsocks clawsmail [options de clawsmail]
