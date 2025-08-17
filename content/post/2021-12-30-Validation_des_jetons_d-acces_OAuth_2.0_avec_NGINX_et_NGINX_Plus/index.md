+++
title = 'Validation des jetons d'accès OAuth 2.0 avec NGINX et NGINX Plus'
date = 2021-12-30 00:00:00 +0100
categories = ['nginx', 'authentification']
+++
- [NGINX OAuth 2.0](#nginx-oauth-20)
  - [Introspection de jeton](#introspection-de-jeton)
  - [Module NGINX auth_request](#module-nginx-auth_request)
    - [Pour valider les jetons d'accès](#pour-valider-les-jetons-daccès)
    - [auth_request + JavaScript](#auth_request--javascript)
    - [Optimisation](#optimisation)
      - [1 Mise en cache par NGINX](#1-mise-en-cache-par-nginx)
      - [2 Mise en cache distribuée avec NGINX Plus](#2-mise-en-cache-distribuée-avec-nginx-plus)
      - [3 Extraction des attributs de la réponse d'introspection](#3-extraction-des-attributs-de-la-réponse-dintrospection)
  - [Configuration de production](#configuration-de-production)
    - [NGINX njs](#nginx-njs)
      - [frontend.conf](#frontendconf)
      - [oauth2.js](#oauth2js)
    - [OAuth 2.0 Token Introspection with NGINX Plus](#oauth-20-token-introspection-with-nginx-plus)
  - [Annexe](#annexe)
    - [Fichiers de configuration](#fichiers-de-configuration)
      - [auth_request.conf](#auth_requestconf)
      - [auth_request_attributes.conf](#auth_request_attributesconf)
      - [auth_request_cache.conf](#auth_request_cacheconf)
      - [auth_request_keyval.conf](#auth_request_keyvalconf)
      - [auth_request_njs.conf](#auth_request_njsconf)
      - [oauth2.js](#oauth2js)
      - [oauth2_attributes.js](#oauth2_attributesjs)
      - [oauth2_keyval.js](#oauth2_keyvaljs)
    - [Liens](#liens)

# NGINX OAuth 2.0

![Nginx](Nginx-logo-large.png)

Il existe de nombreuses options pour authentifier les appels d'API, des certificats client X.509 à l'authentification HTTP de base. Ces dernières années, cependant, une norme de facto a émergé sous la forme de jetons d'accès OAuth 2.0. Il s'agit d'informations d'authentification transmises par le client au serveur d'API, et généralement transportées sous forme d'en-tête HTTP.

OAuth 2.0 est toutefois un dédale de normes interconnectées. Les processus d'émission, de présentation et de validation d'un flux d'authentification OAuth 2.0 reposent souvent sur plusieurs normes connexes. À l'heure où nous écrivons ces lignes, il existe huit normes OAuth 2.0, et les jetons d'accès en sont un exemple, car la spécification de base OAuth 2.0 (RFC 6749) ne spécifie pas de format pour les jetons d'accès. Dans le monde réel, il existe deux formats couramment utilisés :

*    JSON Web Token (JWT) tel que défini par la RFC 7519.
*    Jetons opaques qui ne sont guère plus qu'un identifiant unique pour un client authentifié.

Après l'authentification, un client présente son jeton d'accès avec chaque requête HTTP pour accéder aux ressources protégées. La validation du jeton d'accès est nécessaire pour s'assurer qu'il a bien été émis par un fournisseur d'identité (IdP) de confiance et qu'il n'a pas expiré. Comme les IdP signent de manière cryptographique les JWT qu'ils émettent, les JWT peuvent être validés "hors ligne" sans dépendance d'exécution vis-à-vis de l'IdP. Généralement, un JWT comprend également une date d'expiration qui peut également être vérifiée. Le module NGINX Plus auth_jwt effectue la validation JWT hors ligne.

Les jetons opaques, quant à eux, doivent être validés en les renvoyant à l'IdP qui les a émis. Cela présente toutefois l'avantage que ces jetons peuvent être révoqués par l'IdP, par exemple dans le cadre d'une opération de déconnexion globale, sans que les sessions précédemment connectées restent actives. La déconnexion globale peut également rendre nécessaire la validation des JWT auprès de l'IdP.

Dans ce blog, nous décrivons comment NGINX et NGINX Plus peuvent agir en tant que Relying Party OAuth 2.0, en envoyant des jetons d'accès à l'IdP pour validation et en n'envoyant par proxy que les demandes qui passent le processus de validation. Nous discutons des différents avantages de l'utilisation de NGINX et NGINX Plus pour cette tâche, et comment l'expérience utilisateur peut être améliorée en mettant en cache les réponses de validation pendant une courte période. Pour NGINX Plus, nous montrons également comment le cache peut être distribué sur un cluster d'instances NGINX Plus, en mettant à jour le magasin de valeurs clés avec le module JavaScript, tel qu'introduit dans NGINX Plus R18.

Sauf indication contraire, les informations contenues dans ce blog s'appliquent à la fois à NGINX Open Source et à NGINX Plus. Les références à NGINX Plus s'appliquent uniquement à ce produit.


## Introspection de jeton

La méthode standard pour valider les jetons d'accès avec un IdP est appelée introspection de jeton. La RFC 7662, OAuth 2.0 Token Introspection, est maintenant une norme largement soutenue qui décrit une interface JSON/REST qu'une partie utilisatrice utilise pour présenter un jeton à l'IdP, et décrit la structure de la réponse. Elle est prise en charge par un grand nombre des principaux fournisseurs d'IdP et de clouds.

Quel que soit le format de jeton utilisé, l'exécution de la validation au niveau de chaque service ou application dorsale entraîne une duplication importante du code et un traitement inutile. Il faut tenir compte de diverses conditions d'erreur et de cas limites, et le faire dans chaque service dorsal est une recette pour une mise en œuvre incohérente et, par conséquent, une expérience utilisateur imprévisible. Considérez comment chaque service dorsal pourrait traiter les conditions d'erreur suivantes :

*    Jeton d'accès manquant
*    Jeton d'accès extrêmement grand
*    Caractères non valides ou inattendus dans le jeton d'accès
*    Présentation de plusieurs jetons d'accès
*    Décalage d'horloge entre les services dorsaux

![](OAuth-2.0-access-tokens_backends-validate.png)

## Module NGINX auth_request

### Pour valider les jetons d'accès

Pour éviter la duplication du code et les problèmes qui en découlent, nous pouvons utiliser NGINX pour valider les jetons d'accès au nom des services dorsaux. Cela présente un certain nombre d'avantages :

*    Les requêtes atteignent les services backend uniquement lorsque le client a présenté un jeton valide.
*    Les services backend existants peuvent être protégés par des jetons d'accès, sans qu'il soit nécessaire de modifier le code.
*    Seule l'instance NGINX (et non chaque application) doit être enregistrée auprès de l'IdP.
*    Le comportement est cohérent pour toutes les conditions d'erreur, y compris les jetons manquants ou invalides.

![](OAuth-2.0-access-tokens_NGINX-validates.png)

Avec NGINX agissant en tant que reverse proxy pour une ou plusieurs applications, nous pouvons utiliser le module auth_request pour déclencher un appel API à un IdP avant de transmettre une requête au backend. Comme nous le verrons dans un instant, la solution suivante présente un défaut fondamental, mais elle introduit le fonctionnement de base du module auth_request, que nous développerons dans les sections suivantes.

![](OAuth-2-nginx-001.png)

La directive [auth_request](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request) (ligne 5) spécifie l'emplacement pour le traitement des appels d'API. Le proxy vers le backend (ligne 6) ne se produit que si la réponse à auth_request est réussie. L'emplacement auth_request est défini à la ligne 9. Il est marqué comme [internal](https://nginx.org/en/docs/http/ngx_http_core_module.html#internal) pour empêcher les clients externes d'y accéder directement.

Les lignes 11 à 14 définissent divers attributs de la demande afin qu'elle soit conforme au format de la demande d'introspection de jeton. Notez que le jeton d'accès envoyé dans la demande d'introspection est un composant du corps défini à la ligne 14. Ici, token=$http_apikey indique que le client doit fournir le jeton d'accès dans l'en-tête de requête apikey. Bien sûr, le jeton d'accès peut être fourni dans n'importe quel attribut de la requête, auquel cas nous utilisons une variable NGINX différente.

### auth_request + JavaScript 

Comme nous l'avons mentionné, utiliser le module auth_request de cette manière n'est pas une solution complète. Le module auth_request utilise les codes d'état HTTP pour déterminer le succès (2xx = bon, 4xx = mauvais). Cependant, les réponses d'introspection de jeton OAuth 2.0 codent le succès ou l'échec dans un objet JSON, et renvoient le code d'état HTTP 200 (OK) dans les deux cas.


![](OAuth-2-nginx-002.png)  
Format JSON de la réponse d'introspection de jeton pour un jeton valide

Nous avons besoin d'un analyseur JSON pour convertir la réponse d'introspection de l'IdP en code d'état HTTP approprié afin que le module **auth_request** puisse interpréter correctement cette réponse.

Heureusement, l'analyseur JSON est une tâche triviale pour le module JavaScript de NGINX (njs). Ainsi, au lieu de définir un bloc [location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) pour effectuer la demande d'introspection de jeton, nous indiquons au module auth_request d'appeler une fonction JavaScript.

Editor - Le cas d'utilisation suivant n'est qu'un des nombreux cas d'utilisation du module JavaScript de NGINX. Pour la liste de tous les cas d'utilisation, voir  [Use Cases for the NGINX JavaScript Module](https://www.nginx.com/blog/harnessing-power-convenience-of-javascript-for-each-request-with-nginx-javascript-module/#Use-Cases-for-the-NGINX-JavaScript-Module)

Remarque : cette solution nécessite que le module JavaScript soit chargé en tant que module dynamique avec la directive load_module dans nginx.conf. Pour obtenir des instructions, consultez [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/nginscript/)
{: .prompt-info }

![](OAuth-2-nginx-003.png)  

La directive [js_content](https://nginx.org/en/docs/http/ngx_http_js_module.html#js_content) de la ligne 13 spécifie une fonction JavaScript, introspectAccessToken, comme gestionnaire de auth_request. La fonction de gestion est définie dans **oauth2.js** :

![](OAuth-2-nginx-004.png)  

Remarquez que la fonction introspectAccessToken effectue une sous-requête HTTP (ligne 2) vers un autre emplacement (**/oauth2_send_request**) qui est défini dans l'extrait de configuration ci-dessous. Le code JavaScript analyse ensuite la réponse (ligne 5) et renvoie le code d'état approprié au module **auth_request** en fonction de la valeur du champ **active**. Les jetons valides (active) renvoient HTTP **204 (No Content)** (mais le succès) et les jetons invalides renvoient HTTP **403 (Forbidden)**. Les conditions d'erreur renvoient HTTP **401 (Unauthorized)** afin que les erreurs puissent être distinguées des jetons invalides.

Remarque : ce code est fourni à titre de preuve de concept uniquement, et n'est pas de qualité production. Une solution complète avec une gestion et une journalisation complètes des erreurs est fournie ci-dessous.
{: .prompt-info }

L'emplacement de la cible de la sous-demande défini à la ligne 2 ressemble beaucoup à notre configuration originale de **auth_request**.  
![](OAuth-2-nginx-005.png)  

Toute la configuration nécessaire pour construire la demande d'introspection de jeton est contenue dans l'emplacement **/_oauth2_send_request**. L'authentification (ligne 19), le jeton d'accès lui-même (ligne 21) et l'URL du point de terminaison de l'introspection du jeton (ligne 22) sont généralement les seuls éléments de configuration nécessaires. L'authentification est nécessaire pour que l'IdP accepte les demandes d'introspection de jeton provenant de cette instance NGINX. La spécification OAuth 2.0 Token Introspection impose l'authentification, mais ne précise pas la méthode. Dans cet exemple, nous utilisons un jeton de porteur dans l'en-tête **Authorization**.

Avec cette configuration en place, lorsque NGINX reçoit une requête, il la transmet au module JavaScript, qui effectue une requête d'introspection de jeton auprès de l'IdP. La réponse de l'IdP est inspectée, et l'authentification est considérée comme réussie lorsque le champ **active** est **true**. Cette solution est un moyen compact et efficace d'effectuer une introspection de jeton OAuth 2.0 avec NGINX, et peut facilement être adaptée à d'autres API d'authentification.

Mais nous n'avons pas tout à fait terminé. Le plus grand défi de l'introspection de jeton en général est qu'elle ajoute de la latence à chaque requête HTTP. Cela peut devenir un problème important lorsque l'IdP en question est une solution hébergée ou un fournisseur de cloud. NGINX et NGINX Plus peuvent offrir des optimisations à cet inconvénient en mettant en cache les réponses d'introspection.

### Optimisation

#### 1 Mise en cache par NGINX

L'introspection des jetons OAuth 2.0 est fournie par l'IdP à un point de terminaison JSON/REST, et la réponse standard est donc un corps JSON avec un statut HTTP 200. Lorsque cette réponse est associée à la clé d'accès, elle devient hautement cachable.  

![](OAuth-2-nginx-006.png)  
Réponse d'introspection complète pour un jeton d'accès valide

NGINX peut être configuré pour mettre en cache une copie de la réponse d'introspection pour chaque jeton d'accès afin que la prochaine fois que le même jeton d'accès est présenté, NGINX serve la réponse d'introspection mise en cache au lieu d'effectuer un appel API à l'IdP. Cela améliore considérablement la latence globale pour les requêtes suivantes. Nous pouvons contrôler la durée d'utilisation des réponses mises en cache, afin d'atténuer le risque d'accepter un jeton d'accès expiré ou récemment révoqué. Par exemple, si un client d'API effectue généralement une rafale de plusieurs appels d'API sur une courte période, une validité de cache de 10 secondes peut être suffisante pour fournir une amélioration mesurable de l'expérience utilisateur.

La mise en cache est activée en spécifiant son stockage - un répertoire sur le disque pour le cache (réponses d'introspection) et une zone de mémoire partagée pour les clés (jetons d'accès).  
![](OAuth-2-nginx-007.png)  

La directive [proxy_cache_path](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path) alloue le stockage nécessaire : **/var/cache/nginx/oauth** pour les réponses d'introspection et une zone de mémoire appelée **token_responses** pour les clés. Elle est configurée dans le contexte **http** et apparaît donc en dehors des blocs **server** et **location**. La mise en cache elle-même est ensuite activée dans le bloc d'emplacement où les réponses d'introspection des jetons sont traitées :  

![](OAuth-2-nginx-008.png)  

La mise en cache est activée pour cet emplacement avec la directive [proxy_cache](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache) (ligne 26). Par défaut, NGINX met en cache l'URI mais dans notre cas, nous voulons mettre en cache la réponse en fonction du jeton d'accès présenté dans l'en-tête de requête **apikey** (ligne 27).

A la ligne 28, nous utilisons la directive [proxy_cache_lock](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_lock) pour indiquer à NGINX que si des demandes concurrentes arrivent avec la même clé de cache, il doit attendre que la première demande ait rempli le cache avant de répondre aux autres. La directive [proxy_cache_valid](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_valid) (ligne 29) indique à NGINX combien de temps il doit mettre en cache la réponse d'introspection. Sans cette directive, NGINX détermine le temps de mise en cache à partir des en-têtes cache-control envoyés par l'IdP ; cependant, ceux-ci ne sont pas toujours fiables, c'est pourquoi nous indiquons également à NGINX d'[ignorer les en-têtes](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ignore_headers) qui pourraient affecter la mise en cache des réponses (ligne 30).

Avec la mise en cache maintenant activée, un client présentant un jeton d'accès ne subit que le coût de latence de la demande d'introspection du jeton une fois toutes les 10 secondes.

#### 2 Mise en cache distribuée avec NGINX Plus 

La combinaison de la mise en cache du contenu avec l'introspection des jetons est un moyen très efficace d'améliorer les performances globales des applications avec un impact négligeable sur la sécurité. Cependant, si NGINX est déployé de manière distribuée - par exemple, sur plusieurs centres de données, plates-formes de cloud computing ou un cluster actif-actif - les réponses à l'introspection de jetons mises en cache ne sont disponibles que pour l'instance NGINX qui a effectué la demande d'introspection.

Avec NGINX Plus, nous pouvons utiliser le [module keyval](https://nginx.org/en/docs/http/ngx_http_keyval_module.html) - un magasin clé-valeur en mémoire - pour mettre en cache les réponses d'introspection de jetons. De plus, nous pouvons également synchroniser ces réponses sur un cluster d'instances NGINX Plus en utilisant le [module zone_sync](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html). Cela signifie que quelle que soit l'instance NGINX Plus qui a effectué la demande d'introspection de jeton, la réponse est disponible sur toutes les instances NGINX Plus du cluster.

Remarque : la configuration du module **zone_sync** pour le partage de l'état d'exécution sort du cadre de ce blog. Pour plus d'informations sur le partage de l'état dans un cluster NGINX Plus, consultez le [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/).
{: .prompt-info }

Dans [NGINX Plus R18](https://www.nginx.com/blog/nginx-plus-r18-released/#key-value-variables) et plus, le key-value store peut être mis à jour en modifiant la variable qui est déclarée dans la directive **keyval**. Comme le module JavaScript a accès à toutes les variables NGINX, cela permet de remplir les réponses d'introspection dans le key-value store pendant le traitement de la réponse.

Comme le cache du système de fichiers NGINX, le key-value store est activé en spécifiant son stockage, dans ce cas une zone de mémoire qui stocke la clé (jeton d'accès) et la valeur (réponse d'introspection).  
![](OAuth-2-nginx-009.png)  

Notez qu'avec le paramètre timeout de la directive [keyval_zone](https://nginx.org/en/docs/http/ngx_http_keyval_module.html#keyval_zone), nous spécifions la même période de validité de 10 secondes pour les réponses mises en cache qu'à la ligne 29 de **auth_request_cache.conf**, de sorte que chaque membre du cluster NGINX Plus supprime indépendamment la réponse lorsqu'elle expire. La ligne 2 spécifie la paire clé-valeur pour chaque entrée : la clé étant le jeton d'accès fourni dans l'en-tête de requête **apikey**, et la valeur étant la réponse d'introspection telle qu'évaluée par la variable **$token_data**.

Maintenant, pour chaque demande qui inclut un en-tête de demande **apikey**, la variable **$token_data** est remplie avec la réponse d'introspection précédente du jeton, le cas échéant. Nous mettons donc à jour le code JavaScript pour vérifier si nous avons déjà une réponse d'introspection de jeton.

![](OAuth-2-nginx-010.png)  

La ligne 2 vérifie s'il existe déjà une entrée dans le key-value store pour ce jeton d'accès. Étant donné qu'il existe deux façons d'obtenir une réponse d'introspection (à partir du magasin de valeurs clés ou d'une réponse d'introspection), nous déplaçons la logique de validation dans la fonction distincte suivante, **tokenResult** :

![](OAuth-2-nginx-011.png)  

Désormais, chaque réponse d'introspection de jeton est enregistrée dans le magasin de valeurs clés et synchronisée avec tous les autres membres du cluster NGINX Plus. L'exemple suivant montre une simple requête HTTP avec un jeton d'accès valide, suivie d'une requête à l'API NGINX Plus pour afficher le contenu du magasin de valeurs clés.

![](OAuth-2-nginx-012.png)  

Notez que le magasin clé-valeur utilise lui-même le format JSON, de sorte que la réponse d'introspection de jeton a automatiquement un échappement appliqué aux guillemets.

#### 3 Extraction des attributs de la réponse d'introspection 

Une capacité utile de l'introspection de jeton OAuth 2.0 est que la réponse peut contenir des informations sur le jeton en plus de son statut actif. Ces informations comprennent la date d'expiration du jeton et les attributs de l'utilisateur associé : nom d'utilisateur, adresse électronique, etc...

![](OAuth-2-nginx-013.png)  
Réponse d'introspection de jeton avec attributs de jeton

Ces informations supplémentaires peuvent être très utiles. Elles peuvent être enregistrées, utilisées pour mettre en œuvre des politiques de contrôle d'accès à grain fin, ou fournies aux applications dorsales. Nous pouvons exporter chacun de ces attributs vers le module **auth_request** en les envoyant comme en-têtes de réponse supplémentaires avec une réponse réussie (HTTP **204**).

![](OAuth-2-nginx-014.png)  

Nous itérons sur chaque attribut de la réponse d'introspection (ligne 23) et le renvoyons au module [auth_request](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html) en tant qu'en-tête de réponse. Chaque nom d'en-tête est préfixé par Token- pour éviter les conflits avec les en-têtes de réponse standard (ligne 26). Ces en-têtes de réponse peuvent maintenant être convertis en variables NGINX et utilisés dans le cadre d'une configuration régulière.

![](OAuth-2-nginx-015.png)  

Dans cet exemple, nous convertissons l'attribut **username** en une nouvelle variable, **$username** (ligne 11). La directive [auth_request_set](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request_set) nous permet d'exporter le contexte de la réponse d'introspection du jeton dans le contexte de la requête actuelle. L'en-tête de réponse pour chaque attribut (ajouté par le code JavaScript) est disponible sous la forme **$sent_http_token_attribute**. La ligne 12 inclut ensuite la valeur de **$username** en tant qu'en-tête de requête qui est envoyé par procuration au backend. Nous pouvons répéter cette configuration pour n'importe lequel des attributs renvoyés dans la réponse d'introspection de jeton.

![](OAuth-2.0-access-tokens_attributes-exported.png)  

## Configuration de production

Le code et les exemples de configuration ci-dessus sont fonctionnels et conviennent aux tests de validation de concept ou à la personnalisation pour un cas d'utilisation spécifique. Pour une utilisation en production, nous recommandons fortement une gestion des erreurs, une journalisation et une configuration flexible supplémentaires. Vous pouvez trouver une implémentation plus robuste et plus détaillée pour NGINX et NGINX Plus sur notre dépôt GitHub :

### NGINX njs

**[OAuth 2.0 Token Introspection with NGINX (disk caching)](https://github.com/nginxinc/NGINX-Demos/tree/master/oauth2-token-introspection-oss)**    


Cette configuration permet à NGINX de valider un jeton d'authentification par rapport à un serveur d'autorisation en utilisant OAuth 2.0 Token Introspection (RFC 7662). Cette solution utilise le [module auth_request](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html) et le [module JavaScript de NGINX](http://nginx.org/en/docs/njs/index.html) pour exiger l'authentification et effectuer la demande d'introspection de jeton.

Par défaut, le jeton d'authentification du client est attendu comme un jeton de porteur fourni dans l'en-tête `Authorization`. S'il est fourni ailleurs dans la requête HTTP, la variable `$access_token` doit être configurée pour indiquer où obtenir le jeton.

Les demandes d'introspection de jeton sont authentifiées. Par défaut, les variables `$oauth_client_id` et `$oauth_client_secret` sont utilisées pour effectuer une authentification HTTP de base avec le serveur d'autorisation. Si seule la variable $oauth_client_secret est spécifiée, cette valeur est utilisée pour effectuer une authentification avec un jeton de porteur dans l'en-tête `Authorization`.

Les réponses du serveur d'autorisation OAuth 2.0 sont mises en cache pour minimiser la latence de chaque requête.

Si la réponse d'introspection contient des données sur les membres, chaque membre peut être accédé en tant que variable NGINX en utilisant `auth_request_set $new_variable $sent_http_token_membername;` ;. Ces variables peuvent ensuite être enregistrées, utilisées pour le contrôle d'accès conditionnel et transmises par proxy en amont pour fournir des métadonnées d'identité à l'application dorsale.

Le résultat de l'introspection du jeton est enregistré dans le journal des erreurs. Changez le niveau de sévérité de `error_log` pour affecter la verbosité.

#### frontend.conf

```nginx
js_include conf.d/oauth2.js;

map $http_authorization $access_token {
    "~*^bearer (.*)$" $1;
    default $http_authorization;
}

# This is where token introspection responses will be stored if proxy_cache is enabled
proxy_cache_path /var/cache/nginx/tokens levels=1 keys_zone=token_responses:1m max_size=10m;

server {
    listen 80; # Use TLS in production

    # OAuth 2.0 Token Introspection configuration
    resolver 8.8.8.8;                  # For DNS lookup of OAuth server
    subrequest_output_buffer_size 16k; # To fit a complete response from OAuth server
    #error_log /var/log/nginx/error.log debug; # Enable to see introspection details

    #set $access_token $http_apikey; # Where to find the token. Remove when using Authorization header
    set $oauth_token_endpoint     "https://idp.example.com/oauth/token/introspect";
    set $oauth_token_hint         "access_token"; # E.g. access_token, refresh_token
    set $oauth_client_id          "my-client-id"; # Will use HTTP Basic authentication unless empty
    set $oauth_client_secret      "my-client-secret"; # If id is empty this will be used as a bearer token

    location / {
        auth_request /_oauth2_token_introspection;

        # Any member of the token introspection response is available as $sent_http_token_member
        #auth_request_set $username $sent_http_token_username;
        #proxy_set_header X-Username $username;

        proxy_pass http://my_backend;
    }

    location = /_oauth2_token_introspection {
        # This location implements an auth_request server that uses the JavaScript
        # module to perform the token introspection request.
        internal;
        js_content introspectAccessToken;
    }

    location = /_oauth2_send_introspection_request {
        # This location is called by introspectAccessToken(). We use the proxy_
        # directives to construct an OAuth 2.0 token introspection request, as per:
        #  https://tools.ietf.org/html/rfc7662#section-2
        internal;
        gunzip on; # Decompress if necessary

        proxy_method      POST;
        proxy_set_header  Authorization $arg_authorization;
        proxy_set_header  Content-Type "application/x-www-form-urlencoded";
        proxy_set_body    "token=$arg_token&token_hint=$oauth_token_hint";
        proxy_pass        $oauth_token_endpoint;

        proxy_cache           token_responses; # Enable caching of token introspection responses
        proxy_cache_key       $access_token;   # Cache the response for each unique access token
        proxy_cache_lock      on;              # Don't allow simultaneous requests for same token
        proxy_cache_valid     200 10s;         # How long to use cached introspection responses
        proxy_cache_use_stale error timeout;   # Use old responses if we cannot reach the server
        proxy_ignore_headers  Cache-Control Expires Set-Cookie; # Cache even when receiving these
    }
}
```

#### oauth2.js

```js
/*
 * This function is called by the NGINX auth_request directive to perform OAuth 2.0
 * Token Introspection. It uses a subrequest to construct a Token Introspection request
 * to the configured authorization server ($oauth_token_endpoint).
 *
 * Responses are aligned with the valid responses for auth_request:
 * 204: token is active
 * 403: token is not active
 * 401: error condition (details written to error log at error level)
 * 
 * Metadata contained within the token introspection JSON response is converted to response
 * headers. These in turn are available to the auth_request location with the auth_request_set
 * directive. Each member of the response is available to nginx as $sent_http_oauth_<member name>
 *
 * Copyright (C) 2019 Nginx, Inc.
 */
function introspectAccessToken(r) {
    // Prepare Authorization header for the introspection request
    var authHeader = "";
    if (r.variables.oauth_client_id.length) {
        var basicAuthPlaintext = r.variables.oauth_client_id + ":" + r.variables.oauth_client_secret;
        authHeader = "Basic " + basicAuthPlaintext.toBytes().toString('base64');    
    } else {
        authHeader = "Bearer " + r.variables.oauth_client_secret;
    }

    // Make the OAuth 2.0 Token Introspection request
    r.log("OAuth sending introspection request with token: " + r.variables.access_token)
    r.subrequest("/_oauth2_send_introspection_request", "token=" + r.variables.access_token + "&authorization=" + authHeader,
        function(reply) {
            if (reply.status != 200) {
                r.error("OAuth unexpected response from authorization server (HTTP " + reply.status + "). " + reply.body);
                r.return(401);
            }

            // We have a response from authorization server, validate it has expected JSON schema
            try {
                r.log("OAuth token introspection response: " + reply.responseBody)
                var response = JSON.parse(reply.responseBody);
                // TODO: check for errors in the JSON response first
                // We have a valid introspection response
                // Check for validation success
                if (response.active == true) {
                    r.warn("OAuth token introspection found ACTIVE token");
                    // Iterate over all members of the response and return them as response headers
                    for (var p in response) {
                        if (!response.hasOwnProperty(p)) continue;
                        r.log("OAuth token value " + p + ": " + response[p]);
                        r.headersOut['token-' + p] = response[p];
                    }
                    r.status = 204;
                    r.sendHeader();
                    r.finish();
                } else {
                    r.warn("OAuth token introspection found inactive token");
                    r.return(403);
                }
            } catch (e) {
                r.error("OAuth token introspection response is not JSON: " + reply.body);
                r.return(401);
            }
        }
    );
    r.return(401);
}
```

### OAuth 2.0 Token Introspection with NGINX Plus

**[OAuth 2.0 Token Introspection with NGINX Plus (key‑value caching)](https://github.com/nginxinc/NGINX-Demos/tree/master/oauth2-token-introspection-plus)**


## Annexe

### Fichiers de configuration


#### auth_request.conf 

```nginx
server {
    listen 80;

    location / {
        auth_request /_oauth2_token_introspection;                              
        proxy_pass http://my_backend;
    }

    location = /_oauth2_token_introspection {
        internal;
        proxy_method      POST;
        proxy_set_header  Authorization "bearer SecretForOAuthServer";
        proxy_set_header  Content-Type "application/x-www-form-urlencoded";
        proxy_set_body    "token=$http_apikey&token_hint=access_token";
        proxy_pass        https://idp.example.com/oauth/token;
    }
}

```

#### auth_request_attributes.conf 

```nginx
keyval_zone zone=access_tokens:4m timeout=100s sync;
keyval $http_apikey $token_data zone=access_tokens;

js_include oauth2.js; # Location of JavaScript code

server {
    listen 80;

    location / {
        auth_request /_oauth2_token_introspection;   
        auth_request_set $username $sent_http_token_username;
        proxy_set_header X-Username $username;
        proxy_pass http://my_backend;
    }

    location = /_oauth2_token_introspection {
        internal;
        js_content introspectAccessToken;                                       
    }

    location /_oauth2_send_request {
        internal;
        proxy_method      POST;
        proxy_set_header  Authorization "Bearer SecretForOAuthServer";
        proxy_set_header  Content-Type "application/x-www-form-urlencoded";
        proxy_set_body    "token=$http_apikey&token_hint=access_token";
        proxy_pass        https://idp.example.com/oauth/token/introspect;
    }
}

```

#### auth_request_cache.conf 

```nginx
proxy_cache_path /var/cache/nginx/oauth keys_zone=token_responses:1m max_size=2m;

js_include oauth2.js; # Location of JavaScript code

server {
    listen 80;

    location / {
        auth_request /_oauth2_token_introspection;
        proxy_pass http://my_backend;
    }

    location = /_oauth2_token_introspection {
        internal;
        js_content introspectAccessToken;                                       
    }

    location /_oauth2_send_request {
        internal;
        proxy_method      POST;
        proxy_set_header  Authorization "Bearer SecretForOAuthServer";
        proxy_set_header  Content-Type "application/x-www-form-urlencoded";
        proxy_set_body    "token=$http_apikey&token_hint=access_token";
        proxy_pass        https://idp.example.com/oauth/token/introspect;

        proxy_cache           token_responses; # Enable caching
        proxy_cache_key       $http_apikey;    # Cache for each access token
        proxy_cache_lock      on;              # Duplicate tokens must wait
        proxy_cache_valid     200 10s;         # How long to use each response
        proxy_ignore_headers  Cache-Control Expires Set-Cookie;
    }
}
```


#### auth_request_keyval.conf 

```nginx
keyval_zone zone=access_tokens:4m timeout=10s sync;
keyval $http_apikey $token_data zone=access_tokens;

js_include oauth2.js; # Location of JavaScript code

server {
    listen 80;

    location / {
        auth_request /_oauth2_token_introspection;
        proxy_pass http://my_backend;
    }

    location = /_oauth2_token_introspection {
        internal;
        js_content introspectAccessToken;                                       
    }

    location /_oauth2_send_request {
        internal;
        proxy_method      POST;
        proxy_set_header  Authorization "Bearer SecretForOAuthServer";
        proxy_set_header  Content-Type "application/x-www-form-urlencoded";
        proxy_set_body    "token=$http_apikey&token_hint=access_token";
        proxy_pass        https://idp.example.com/oauth/token/introspect;
    }
}
```

#### auth_request_njs.conf 

```nginx
js_include oauth2.js; # Location of JavaScript code

server {
    listen 80;

    location / {
        auth_request /_oauth2_token_introspection;
        proxy_pass http://my_backend;
    }

    location = /_oauth2_token_introspection {
        internal;
        js_content introspectAccessToken;                                       
    }

    location /_oauth2_send_request {
        internal;
        proxy_method      POST;
        proxy_set_header  Authorization "Bearer SecretForOAuthServer";
        proxy_set_header  Content-Type "application/x-www-form-urlencoded";
        proxy_set_body    "token=$http_apikey&token_hint=access_token";
        proxy_pass        https://idp.example.com/oauth/token/introspect;
    }
}

```

#### oauth2.js 

```js
function introspectAccessToken(r) {
    r.subrequest("/_oauth2_send_request",
        function(reply) {
            if (reply.status == 200) {
                var response = JSON.parse(reply.responseBody);
                if (response.active == true) {
                    r.return(204); // Token is valid, return success code
                } else {
                    r.return(403); // Token is invalid, return forbidden code
                }
            } else {
                r.return(401); // Unexpected response, return 'auth required'
            }
        }
    );
}
```

#### oauth2_attributes.js 

```js
function introspectAccessToken(r) {
    if (r.variables.token_data) {
        tokenResult(r); // Existing response in key-value store so do validation
    } else {
        r.subrequest("/_oauth2_send_request",
            function(reply) {
                if (reply.status == 200) {
                    r.variables.token_data = reply.responseBody; // Create entry
                    tokenResult(r); // Do validation of response
                } else {
                    r.return(401); // Unexpected response, return auth-required
                }
            }
        );
    }
}

function tokenResult(r) {
    var response = JSON.parse(r.variables.token_data);

    if (response.active) {
        // Convert all members of the response into response headers
        for (var p in response) {
            if (!response.hasOwnProperty(p)) continue;
            r.log("OAuth2 Token-" + p + ": " + response[p]);
            r.headersOut['Token-' + p] = response[p];
        }
        r.status = 204;
        r.sendHeader();
        r.finish();
    } else {
        r.return(401);
    }
}
```

#### oauth2_keyval.js 

```js
function introspectAccessToken(r) {
    if (r.variables.token_data) {
        tokenResult(r); // Existing response in key-value store so do validation
    } else {
        r.subrequest("/_oauth2_send_request",
            function(reply) {
                if (reply.status == 200) {
                    r.variables.token_data = reply.responseBody; // Create entry
                    tokenResult(r); // Do validation of response
                } else {
                    r.return(401); // Unexpected response, return auth-required
                }
            }
        );
    }
}

function tokenResult(r) {
    var response = JSON.parse(r.variables.token_data);

    if (response.active) {
        r.return(204);
    } else {
        r.return(401);
    }
}
```

### Liens

* [Validating OAuth 2.0 Access Tokens with NGINX and NGINX Plus](https://www.nginx.com/blog/validating-oauth-2-0-access-tokens-nginx/)
    * [nginx-gists/auth_request.conf](https://gist.github.com/nginx-gists/340731282df4b32f2266b9ddf4bd5de5)

