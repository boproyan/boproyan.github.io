+++
title = 'lettre-d-information-xmpp-01-octobre-2019-fosdem-2020-modernisation-de-xmpp-reseaux-de-pairs'
date = 2019-11-13 00:00:00 +0100
categories = application
+++
URL:     https://linuxfr.org/news/lettre-d-information-xmpp-01-octobre-2019-fosdem-2020-modernisation-de-xmpp-reseaux-de-pairs
Title:   Lettre d'information XMPP, 01 octobre 2019, FOSDEM 2020, modernisation de XMPP, réseaux de pairs
Authors: Nÿco
         Pierre Maziere, ZeroHeure, Ysabeau, tisaac et palm123
Date:    2019-11-08T16:17:49+01:00
License: CC by-sa
Tags:    
Score:   9


Bienvenu à la lettre d’information couvrant le mois de septembre.



Nouveau ce mois : nous avons explicité que cette lettre d’information peut être partagée et adaptée comme défini dans la licence CC by-sa 4.0, et nous avons ajouté les attributions car c’est le résultat d’un effort communautaire.



Soyez sympa, informez vos amis et collègues : faites suivre cette lettre d’information !



Soumettez  vos articles, tutoriels ou blog posts à propos de XMPP/Jabber [sur notre wiki](https://wiki.xmpp.org/web/News_and_Articles_for_the_next_XMPP_Newsletter#Current_newsletter).

----

[XMPP Newsletter, 01 Oct 2019, FOSDEM 2020, modernization of XMPP, peer networks](https://xmpp.org/2019/10/newsletter-01-october/)
[The XMPP Newsletter](https://xmpp.org/newsletter.html)
[Processus de contribution à la newsletter](https://wiki.xmpp.org/web/News_and_Articles_for_the_next_XMPP_Newsletter)

----

# Lettre d’information XMPP



## Articles



Ralph Meijer, président du conseil d’administration de la XSF, a écrit un article présentant les « [Messages XMPP attachés, bouclés, référencés](https://ralphm.net/blog/2019/09/09/fastening) » dont les spécifications sont actuellement en cours.



Dans l’éventualité d’une mise hors-service d’Internet, que celle-ci soit due à une catastrophe ou à un acte volontaire, les réseaux pair-à-pair sont très utiles. [Monal utilise Airdrop](http://monal.im/blog/xmppsignalbluetoothp2p-wifi-serverless-chat/) (WiFi et Bluetooth) en complément de XMPP et du protocole de chiffrement de bout-en-bout OMEMO.



![Monal](https://framapic.org/Qg1mv43w1673/akPmtepp0nfH.png)



JabberFr, la communauté francophone XMPP/Jabber, a de nouveau [traduit la lettre d’informations XMPP en français](https://news.jabberfr.org/2019/09/lettre-dactualite-xmpp-du-3-septembre-2019/), _merci beaucoup_.



Nous avons commencé un [guide très minimaliste](https://wiki.xmpp.org/web/Basic_communication_guide_for_XMPP_techies) pour aider à promouvoir un projet via les réseaux sociaux et d’autres moyens tels que des articles de blogs. Cela pourrait être utile aux personnes investies dans des projets XMPP et qui voudraient piocher des idées pour toucher d’autres communautés.



## Événements



Comme chaque année, le temps est venu : nous annonçons fièrement le sommet XMPP 24 et la participation de la XSF au [FOSDEM](https://fosdem.org/2020/). Le 24ᵉ sommet XMPP aura lieu lel jeudi 30 et vendredi 31 janvier, tandis que le FOSDEM se tiendra samedi 1ᵉʳ et dimanche 2 février. Préparez-vous pour ce rassemblement de la communauté ; c’est le bon moment pour réserver vos vols !



![XMPP Summit 24 & FOSDEM 2020](https://framapic.org/fTZlikXsyi0F/ZqlvJhNjWDFj.png)



## Nouvelles versions de logiciels



### Serveurs



* Réda Housni Alaoui relance le [serveur Vysper XMPP](https://github.com/apache/mina-vysper) (prononcé « whisper »), jusqu’alors en hibernation.
* [Xabber Server v.0.9 alpha est sorti](https://blog.xabber.com/xabber-server-v-0-9-alpha-is-released/), avec un mode d’installation rapide, une interface de gestion et des innovations intéressantes.
* [MongooseIM 3.4.1 est sorti](https://www.erlang-solutions.com/resources/download.html) avec une importante mise à jour de sécurité, réparant une vulnérabilité qui autorisait tout utilisateur identifié à planter le nœud avec une stanza malveillante sur certaines (néanmoins populaires) configurations. [Lisez le fil complet](https://twitter.com/MongooseIM/status/1176111308430815232) sur Twitter pour plus d’informations.
* La communauté _igniterealtime_ a beaucoup de nouvelles :
  * [Openfire 4.4.2](https://discourse.igniterealtime.org/t/openfire-4-4-2-release/86209) : « Cette version devrait mieux prendre en charge les connexions de serveur à serveur (s2s), corriger quelques problèmes XSS-style dans la console d’administration, et améliorer la stabilité des sessions client. »
  * Le greffon Fastpath Service a été livré en versions [4.4.4](https://discourse.igniterealtime.org/t/fastpath-service-plugin-4-4-4-released/86155) et [4.4.5](https://discourse.igniterealtime.org/t/fastpath-service-plugin-4-4-5-released/86198), apportant la prise en charge de gestion des queues de demandes de conversations, telle qu’une équipe de support pourrait en avoir l’usage.
  * [Greffon Search 1.7.3](https://discourse.igniterealtime.org/t/search-plugin-1-7-3-released/86207) : « Cette mise à jour ajoute la protection contre les attaques [[CSRF]] et [[XSS]]. »
  * [Greffon Monitoring Service 1.8.1](https://discourse.igniterealtime.org/t/monitoring-service-plugin-1-8-1-released/86208) : « Ce correctif ajoute la protection contre les attaques [[CSRF]] et [[XSS]] sur la page de configuration de l’archivage. »
* Jérôme Sautret, de ProcessOne, a annoncé [ejabberd 19.09](https://blog.process-one.net/ejabberd-19-09/) qui apporte une pile de gestion automatique de certificats améliorée.
* L’équipe Prosody vient de livrer une nouvelle mise à jour de leur branche stable, [Prosody 0.11.3](https://blog.prosody.im/prosody-0.11.3-released/) qui inclut parmi d’autres corrections, des améliorations de performance et de compatibilité.



### Clients et applications



* Profanity est sorti en [version 0.7.0](https://github.com/profanity-im/profanity/releases/tag/0.7.0) après cinq mois de travail, apportant le chiffrement de bout-en-bout OMEMO, suivi d’une version de correction [0.7.1](https://github.com/profanity-im/profanity/releases/tag/0.7.1).
* [Plusieurs vulnérabilités ont été trouvées dans Dino](https://gultsch.de/dino_multiple.html), veuillez mettre à jour dès que possible si vous êtes un utilisateur de Dino.
* Converse est sorti en versions [5.0.2](https://github.com/conversejs/converse.js/releases/tag/v5.0.2) et [5.0.3](https://github.com/conversejs/converse.js/releases/tag/v5.0.3) corrigeant, entre autres, des problèmes de sécurité. Les utilisateurs de Converse pourront trouver utile le nouveau [greffon d’Agayon pour vérifier les requêtes HTTP via XMPP](https://blog.agayon.be/converse_xep_0070.html). Pour les développeurs, il y a une [image Docker de Converse](https://github.com/conversejs/converse.js-docker) proposée par _odajay_.
* Les versions [2.5.8](https://github.com/siacs/Conversations/releases/tag/2.5.8), [2.5.9](https://github.com/siacs/Conversations/releases/tag/2.5.9), [2.5.10](https://github.com/siacs/Conversations/releases/tag/2.5.10) et [2.5.11](https://github.com/siacs/Conversations/releases/tag/2.5.11) de Conversations sont sorties.
* [Monal 4 pour iOS 13 et le mode sombre est sorti](https://monal.im/blog/monal-4-out/). La mise à jour Mac est attendue en octobre.



## Services



[Christopher Muclumbus](https://search.jabber.network/), une liste et un moteur de recherche des salles de conversations publiques XMPP, a été mis à jour avec une refonte visuelle, des avatars pour les salles de conversations, des liens vers des sites de conversations web anonymes et les journaux si disponibles, et des versions des logiciels sous forme de camembert. 
_NdM : le serveur s’appelle [désormais](https://github.com/horazont/muchopper/commit/983a63cd20b338c16658e63f5f92912ef6a4587b) search.jabber.network Crawle._



## Extensions et spécifications



Ce mois-ci, trois [[XEP]] ont été proposées, et deux mises à jour. Pas de XEP en Last Call (Dernier appel), New (nouvelle) ou Obsoleted (obsolète).



### Proposées



* Rattachement de message (Message Fastening)
  * Résumé : cette spécification définit un moyen de marquer les données utiles d’un message comme étant logiquement rattachées à un message précédent.
  * URL : https://xmpp.org/extensions/inbox/fasten.html
* Suites 2020 de conformité à XMPP (XMPP Compliance Suites 2020)
  * Résumé : ce document définit les niveaux de conformité au protocole XMPP.
  * URL : https://xmpp.org/extensions/inbox/cs-2020.html
* Jetons d’autorisation (Authorization Tokens)
  * Résumé : ce document définit une extension du protocole XMPP pour générer des jetons d’authentification pour les applications clientes et fournir des méthodes de gestion des connexions clientes.
  * URL : https://xmpp.org/extensions/inbox/auth-tokens.html



### Mises à jour



* La version 0.13.1 de la XEP-0280 (Message Carbons) a été publiée. 
  * Changements : ajout d’un exemple clair concernant les messages carbon problématiques (contrefaits) et sur la nécessité de les gérer (gl). 
  * URL : https://xmpp.org/extensions/xep-0280.html
* La version 1.16.0 de la XEP-0060 (Publish-Subscribe) a été publiée. 
  * Changements : Ajout d’une fonctionnalité pubsub#rsm disco#info pour éviter toute confusion (edhelas). 
  * URL : https://xmpp.org/extensions/xep-0060.html



## À bientôt en novembre !



Cette lettre d’information XMPP est un effort communautaire collaboratif. Merci à Nÿco, Daniel, Jwi et MDosch pour le regroupement des informations. Merci à Nÿco, Seve, Jwi et Matt pour la rédaction. Merci à Guus, Seve, Jonas pour les relectures.



Suivez et relayez les informations XMPP sur notre compte Twitter [@xmpp](https://twitter.com/xmpp).



## Licence



Cette lettre d’information est publiée sous licence CC by-sa : https://creativecommons.org/licenses/by-sa/4.0/
