+++
title = 'Guide et SSO avec le module Nginx 'auth_request''
date = 2019-10-08 00:00:00 +0100
categories = nginx
+++
# Le guide "auth_request_module" de nginx

[nginx's auth_request_module howto](https://www.0ink.net/2019/05/10/nginx_mod_authrequest.html)

Cet article tente de compléter les documentations nginx concernant le module auth_request et comment le configurer . À mon avis, cette documentation est un peu incomplète.

## Quel est le module auth_request de nginx

La documentation de ce module indique qu'elle implémente l'autorisation du client en fonction du résultat d'une sous-demande.

Cela signifie que lorsque vous envoyez une requête HTTP à une URL protégée, nginx effectue une sous-demande interne sur une URL d'autorisation définie. 

* Si le résultat de la sous-demande est HTTP 2xx, nginx envoie par proxy la requête HTTP d'origine au serveur principal. 
* Si le résultat de la sous-demande est HTTP 401 ou 403, l'accès au serveur principal est refusé.

En configurant nginx , vous pouvez rediriger ces 401 ou 403 vers une page de connexion où l'utilisateur est authentifié puis redirigé vers la destination d'origine.

L'ensemble du processus de sous-demande d'autorisation est ensuite répété, mais comme l'utilisateur est maintenant authentifié, la sous-demande renvoie HTTP 200 et la requête HTTP d'origine est envoyée par proxy au serveur principal.

## Configuration de nginx

Dans votre configuration nginx …

Ce bloc configure la zone du serveur Web qui sera protégée:

```
        location /hello {
          error_page 401 = @error401;	# Specific login page to use
          auth_request /auth;		# The sub-request to use
          auth_request_set $username $upstream_http_x_username;	# Make the sub request data available
          auth_request_set $sid $upstream_http_x_session;	# send what is needed

          proxy_pass http://sample.com:8080/hello;	# actual location of protected data
          proxy_set_header X-Forwarded-Host $host;	# Custom headers with authentication related data
          proxy_set_header X-Remote-User $username;
          proxy_set_header X-Remote-SID $sid;
        }
```

*    `error_page 401` définit la page de connexion personnalisée à utiliser (le cas échéant). En théorie, pour une API REST, il serait possible de s'authentifier à l'aide d'un en-tête de **token** fourni, ce qui rend la page de connexion inutile.
*    `auth_request /auth` définit que cet emplacement nécessite une authentification et définit l'emplacement de la sous-requête à utiliser.
*    `auth_request_set` peut être utilisé pour extraire des données des en-têtes de sous-demande et les rendre disponibles ultérieurement (par exemple, à un serveur de contenu principal utilisant des en-têtes personnalisés.
*    `proxy_pass` définit le serveur de contenu principal.
*    `proxy_set_header` définit un en-tête personnalisé utilisé pour transmettre des informations au serveur backend de contenu. 

Ce bloc configure le serveur de sous-requête d’authentification:

```
        location = /auth {
          proxy_pass http://auth-server.sample.com:8080/auth;	# authentication server
          proxy_pass_request_body off;				# no data is being transferred...
          proxy_set_header Content-Length '0';
          proxy_set_header Host $host;				# Custom headers with authentication related data
          proxy_set_header X-Origin-URI $request_uri;
          proxy_set_header X-Forwarded-Host $host;
        }
```

*    `proxy_pass` où la sous-requête doit être traitée.
*    `proxy_pass_request_body` off et `proxy_set_header Content-Length 0` permettent de supprimer le corps du contenu et d’envoyer uniquement les en-têtes au serveur d’authentification.
*    `proxy_set_header` détails supplémentaires envoyés à la sous-requête. Par exemple, l' `X-Origin-URI` 

Ceci implémente l'URL du pager de connexion

```
        # If the user is not logged in, redirect them to login URL
        location @error401 {
          return 302 https://$host/login/?url=https://$http_host$request_uri;
        }         
```

Dans cet exemple, la page de connexion se trouve sur le même proxy inverse, mais ce n'est pas nécessairement le cas.

La page de connexion actuelle:

```
        location /login/ {
          proxy_pass http://auth-server.sample.com:8080/login/;	# Where the login happens
          proxy_set_header X-My-Real-IP $remote_addr;		# Additional parameters to send to login page
          proxy_set_header X-My-Real-Port $remote_port;
          proxy_set_header X-My-Server-Port $server_port;
        }
```

*    `proxy_pass` pointe sur l'emplacement du script de connexion
*    `proxy_set_header` peut être utilisé pour transmettre des champs supplémentaires pouvant être requis par le script de connexion. Pour implémenter, par exemple, des règles d'accès basées sur `$remote_addr` 

Donc, dans cet exemple particulier, nous faisons référence à un serveur avec DEUX emplacements:

1.    http://auth-server.sample.com:8080/auth - L'URI de sous-requête qui n'est pas visible à l'extérieur mais qui gère la sous-requête.
2.    http://auth-server.sample.com:8080/login/ - L'URI de connexion qui gère la conversation de connexion. 

## Le serveur d'authentification

C’est là que la documentation de nginx manque un peu. Il n’ya pas d’exemple de serveur d’authentification à utiliser.

Dans mon exemple, nous avons un workflow d'authentification simple. Lorsqu'un utilisateur non authentifié rencontre le serveur, la sous-requête est appelée et recherche (et échoue) la présence d'un cookie de session.

L'utilisateur est ensuite redirigé vers la page de connexion, où la connexion réelle a lieu. En cas de succès, un cookie de session est défini et l'utilisateur est redirigé vers l'URL d'origine.

Ceci est implémenté en utilisant le script suivant:

```python
#/usr/bin/env python
from bottle import route, run, request, response, abort, redirect
import sys
import uuid

SIGNATURE = uuid.uuid4()
COOKIE = 'demo-cookie'

sessions = {}

def check_login(username,password):
  if username == password:
    return True
  return False

def is_active_session():
  sid = request.get_cookie(COOKIE, secret=SIGNATURE)
  if not sid:
    return None
  if sid in sessions:
    return sid
  else:
    return None    

@route('/hello')
def show_headers():
  hdrs = dict(request.headers)
  import pprint
  return "Got it\n<pre>\n"+pprint.pformat(hdrs,2)+"\n</pre>\n"

@route('/login/', method='GET')
def user_login():
  sid = is_active_session()
  if sid:
    return 'Logged in as %s' % sessions[sid]

  return '''
    <form method="post">
     Username: <input name="username" type="text" /><br/>
     Password: <input name="password" type="password" /><br/>
     <input value="Login" type="submit" />
    </form>'''

@route('/login/',method='POST')
def do_login():
  username = request.forms.get('username')
  password = request.forms.get('password')
  if 'url' in request.query:
    url = request.query.url
  else:
    url = None

  if check_login(username,password):
    sid = uuid.uuid4
    sessions[sid] = username
    response.set_cookie(COOKIE, sid, secret=SIGNATURE, path="/", httponly=True)
    if url:
      redirect(url)
    else:
      return "Welcome %s" % username
  else:
    return "Login Failed"

@route('/auth')
def auth():
  sid = is_active_session()
  if sid:
    response.set_header('X-Username', sessions[sid])
    response.set_header('X-Session', str(sid))
    return 'OK ' + str(sid)
  else:
    abort(401,"Unathenticated")

run(host='0.0.0.0', port=8080)
```

[snippets/nginx_mod_authrequest/auth1.py](https://github.com/alejandroliu/0ink.net/blob/master/snippets/nginx_mod_authrequest/auth1.py) -- [view raw ](https://github.com/alejandroliu/0ink.net/raw/master/snippets/nginx_mod_authrequest/auth1.py)

Ceci fait des utilisations du micro cadre de la [bottle](https://bottlepy.org/) 

Il implémente quatre itinéraires:

1.    GET /hello
    Ceci est juste une URL de démonstration utilisée pour les tests. Affiche uniquement les en-têtes de demande.
2.    GET /login/
    C'est le point d'entrée de la page de connexion.
3.    POST /login/
    Ceci est le gestionnaire de la page de connexion.
4.    GET /auth
    C'est le gestionnaire de sous-requête. 

Pour la démo, nous ne faisons pas vraiment de traitement de connexion. Il vous suffit de faire en sorte que le nom d'utilisateur soit identique au mot de passe pour vous connecter. Tout le reste est un échec de connexion.

Lorsque l'utilisateur se connecte avec succès, nous définissons un cookie. L'URL de connexion et la ressource protégée ( /hello URL) se /hello dans la même étendue de cookie, nous pouvons utiliser le cookie défini par la page de connexion comme jeton de vérification dans la sous-demande.

Notez que la page de connexion peut être aussi simple ou aussi complexe que nécessaire. Par exemple, il est possible d'implémenter un flux de travail SAML , OpenID Connect ou tout autre flux de travail Single-Sign-On disponible.

En variante, une authentification à deux facteurs pourrait être implémentée ici. Les possibilités sont infinies.

Une utilisation intéressante du module [auth_request](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html) serait de déléguer l' [authentification de base](https://en.wikipedia.org/wiki/Basic_access_authentication) à un autre serveur ou même d'implémenter des authentifications non prises en charge par nginx, comme par exemple un simple en-tête Token-Bearer ou une [authentification Digest](https://en.wikipedia.org/wiki/Digest_access_authentication).

## authentification de base

Cela peut sembler idiot puisque [nginx](https://nginx.org/en/) prend en charge l’[authentification de base](https://en.wikipedia.org/wiki/Basic_access_authentication) tout de suite. Le cas d'utilisation de cela est lorsque vous avez un cluster de serveurs frontaux nginx et que vous souhaitez qu'ils s'authentifient tous auprès d'un serveur d'identité central. De plus, étant donné que l'URI peut être transmis, un contrôle d'accès plus sophistiqué peut être mis en œuvre. Enfin, des valeurs supplémentaires peuvent être passées dans les en-têtes, telles que les noms de groupe, les jetons, etc.

### configuration nginx

Ressource protégée:

```
        location /hello {
          auth_request /auth;		# The sub-request to use
          auth_request_set $username $upstream_http_x_username;	# Make the sub request data available

          proxy_pass http://sample.com:8080/hello;	# actual location of protected data
          proxy_set_header X-Forwarded-Host $host;	# Custom headers with authentication related data
          proxy_set_header X-Remote-User $username;
        }
```

>REMARQUE: contrairement à l'exemple précédent, il n'est pas nécessaire de fournir une page @error401.

Configuration de sous-requête:

```
        location = /auth {
          proxy_pass http://auth-server.sample.com:8080/auth;	# authentication server
          proxy_pass_request_body off;				# no data is being transferred...
          proxy_set_header Content-Length '0';
          proxy_set_header Host $host;				# Custom headers with authentication related data
          proxy_set_header X-Origin-URI $request_uri;
          proxy_set_header X-Forwarded-Host $host;
        }
```

### serveur d'authentification

L'implémentation en python (encore une fois, en utilisant [bottle](https://bottlepy.org/) ):

```python
#/usr/bin/env python
from bottle import route, run, request, response, abort, redirect
import base64

def check_login(username,password):
  if username == password:
    return True
  return False

def authorize(basic_str):
  if not basic_str:
    return False
  auth = basic_str.split()
  if auth[0] != 'Basic':
    # We only support basic authentication
    return False
  auth = base64.b64decode(auth[1])
  auth = auth.split(':',2)
  if len(auth) != 2:
    # Unable to decode username password...
    return False
  
  if check_login(auth[0],auth[1]):
    response.set_header('X-Username', auth[0])
    return True

  return False  


@route('/hello')
def show_headers():
  hdrs = dict(request.headers)
  import pprint
  return "Got it\n<pre>\n"+pprint.pformat(hdrs,2)+"\n</pre>\n"

@route('/auth')
def auth():
  if authorize(request.headers.get('Authorization')):
    return 'Yes, go in'

  response.set_header('WWW-Authenticate', 'Basic realm="DEMO REALM"')
  response.status = 401
  return 'Unauthenticated'

run(host='0.0.0.0', port=8080)
```

[snippets/nginx_mod_authrequest/auth2.py](https://github.com/alejandroliu/0ink.net/blob/master/snippets/nginx_mod_authrequest/auth2.py) -- [view raw ](https://github.com/alejandroliu/0ink.net/raw/master/snippets/nginx_mod_authrequest/auth2.py)

Comme dans l'exemple précédent, nous ne procédons à aucune vérification d'utilisateur / mot de passe. Nous vérifions seulement si le nom d'utilisateur et le mot de passe correspondent.

Contrairement à l'exemple précédent, toute l'authentification est gérée par une seule route ( `/auth` ). Il renvoie 'WWW-Authenticate' pour demander à l'utilisateur un mot de passe. Et s'il voit un en-tête d' `Authorization` , il le validera.

## authentification digest

Cela implémente l'authentification [Digest](https://en.wikipedia.org/wiki/Digest_access_authentication) pour nginx à l'aide du module de demande d'authentification ([auth request module](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html)).  
La configuration de nginx est la même que dans l'[authentification de base](https://en.wikipedia.org/wiki/Basic_access_authentication) .

L'implémentation en python (en utilisant la structure du framework [bootle](https://bottlepy.org/) ):

```python
#/usr/bin/env python
from bottle import route, run, request, response, abort, redirect
import base64
import hashlib
import random
import shlex

REALM = 'Demo Realm'

def md5sum(x):
  return hashlib.md5(x).hexdigest()

def gen_noise(len = 16):
  letters = '0123456789abcdef'
  salt = ''
  for x in range(len):
    salt = salt + random.choice(letters)
  return salt

def authorize(auth):
  if not auth:
    return False
  auth = shlex.split(auth)
  if len(auth) < 2 or auth[0] != 'Digest':
    return False

  req = {}
  for x in auth:
    if x[-1:] == ',':
      x = x[:-1]
    x = x.split('=',2)
    if len(x) == 0:
      continue
    elif len(x) == 1:
      req[x[0].lower()] = True
    else:
      req[x[0].lower()] = x[1]

  # make sure that all fields are there...
  for x in ['username','realm','nonce','uri','response']:
    if not x in req:
      print('Missing "%s" in response' % x)

  method = request.headers.get('X-Origin-Method')
  if not method:
    method = 'GET'

  ha1 = md5sum('%s:%s:%s' % (req['username'],req['realm'],req['username']))
  ha2 = md5sum('%s:%s' % (method, req['uri']))
  resp = md5sum('%s:%s:%s' % ( ha1, req['nonce'], ha2))

  #print(req)
  #print("calc: %s\nrecv: %s\n" % (resp, req['response']))

  if resp == req['response']:  
    response.set_header('X-Username', req['username'])
    return True

  return False  


@route('/hello')
def show_headers():
  hdrs = dict(request.headers)
  import pprint
  return "Got it\n<pre>\n"+pprint.pformat(hdrs,2)+"\n</pre>\n"

@route('/auth')
def auth():
  if authorize(request.headers.get('Authorization')):
    return 'Yes, go in'

  nonce = gen_noise(32)
  opaque = gen_noise(32)
  
  #print("nonce=%s\nopaque=%s\n" % (nonce, opaque))
  response.set_header('WWW-Authenticate', 'Digest realm="%s", nonce="%s", opaque="%s"' % (REALM, nonce, opaque))
  response.status = 401
  return 'Unauthenticated'

run(host='0.0.0.0', port=8080)
```

[ snippets/nginx_mod_authrequest/auth3.py](https://github.com/alejandroliu/0ink.net/blob/master/snippets/nginx_mod_authrequest/auth3.py) -- [view raw](https://github.com/alejandroliu/0ink.net/raw/master/snippets/nginx_mod_authrequest/auth3.py)

# Exemple nginx module 'http_auth_request_module'

Assurez-vous que votre NGINX Open Source est compilé avec l'option de configuration module `--with-http_auth_request_module`

Dans l'emplacement qui nécessite une authentification de demande, spécifiez la directive auth_request dans laquelle spécifier un emplacement interne où une sous-demande d'autorisation sera transmise :

```
    location /private/ {
        auth_request /auth;
        #...
    }
```

Ici, pour chaque demande à /private, une sous-demande à l'emplacement interne /auth sera faite.  
Spécifiez un emplacement interne et la directive proxy_pass à l'intérieur de cet emplacement qui transmettra les sous-demandes d'authentification par proxy à un serveur ou service d'authentification :

```
    location = /auth {
        internal;
        proxy_pass http://auth-server;
        #...
    }
```

Comme le corps de la requête est rejeté pour les sous-requêtes d'authentification, vous devrez désactiver la directive proxy_pass_request_body et également définir l'en-tête Content-Length sur une chaîne nulle :

```
    location = /auth {
        internal;
        proxy_pass              http://auth-server;
        proxy_pass_request_body off;
        proxy_set_header        Content-Length "";
        #...
    }
```

Passez l'URI de la requête originale complète avec les arguments de la directive proxy_set_set_header :

```
    location = /auth {
        internal;
        proxy_pass              http://auth-server;
        proxy_pass_request_body off;
        proxy_set_header        Content-Length "";
        proxy_set_header        X-Original-URI $request_uri;
    }
```

En option, vous pouvez définir une valeur variable basée sur le résultat de la sous-requête avec la directive auth_request_set :

```
    location /private/ {
        auth_request        /auth;
        auth_request_set $auth_status $upstream_status;
    }
```

Exemple complet  
Cet exemple résume les étapes précédentes en une seule configuration :

```
http {
    #...
    server {
    #...
        location /private/ {
            auth_request     /auth;
            auth_request_set $auth_status $upstream_status;
        }

        location = /auth {
            internal;
            proxy_pass              http://auth-server;
            proxy_pass_request_body off;
            proxy_set_header        Content-Length "";
            proxy_set_header        X-Original-URI $request_uri;
        }
    }
}
```


## SSO avec le module Nginx auth_request

Récemment, nous avons eu le défi de connecter un site Web statique à notre infrastructure SSO (Single Sign-On) existante.

Les composants suivants sont impliqués

*    api.example.com : le point de terminaison de l'API SSO
*    login.example.com : interface utilisateur orientée utilisateur pour l'API SSO; Fournit les formulaires d'inscription et de connexion, etc.
*    staticpage.example.com : contenu statique du site Web devant être sécurisé / connecté au SSO. 

L'authentification sur l'API SSO est effectuée avec un jeton qui peut être fourni via l'entête `X-SHOPWARE-SSO-Token` ou via le cookie `shopware_sso_token` .

### challenge

Notre tâche était de nous assurer que toutes les demandes à *staticpage.example.com* sont autorisées par *api.example.com*   
Les demandes non authentifiées doivent être redirigées vers *login.example.com* 

### Nginx à la rescousse

Heureusement, [nginx](http://nginx.org/) est également capable de résoudre ce problème pour nous.

Tout ce dont nous avons besoin, c'est du module [auth_request](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html) 

>Le module `ngx_http_auth_request_module` implémente l'autorisation du client en fonction du résultat d'une sous-demande.  
-> Si la sous-demande renvoie un code de réponse 2xx, l'accès est autorisé.  
-> S'il renvoie 401 ou 403, l'accès est refusé avec le code d'erreur correspondant. 


### Configuration

Dans le vhost de *staticpage.example.com* nous devons ajouter la directive `auth_request` :

```
server {
    server_name staticpage.example.com;
    auth_request /auth;
    ...
}
```

Pour chaque demande adressée à http://staticpage.example.com/ , une sous-demande interne à http://staticpage.example.com/auth est effectuée.

Ajoutons ceci en tant qu'emplacement:

```
server {
    ...
    location = /auth {
        internal;
        proxy_pass https://api.example.com;
    }
}
```

La demande est maintenant transmise à notre noeud final SSO ( proxy_pass ). Veuillez noter que le chemin de l'emplacement est inclus dans cette demande. L'URL de la demande devient donc **https://api.example.com/auth** .

À ce stade, *api.example.com* est responsable de l'autorisation.  
Si la demande renvoie un code de réponse 2xx, la demande est autorisée. S'il renvoie 401 ou 403, l'accès est refusé.

Gérons la redirection au cas où l'API SSO renvoie le code HTTP 401. Avec la directive [error_page](http://nginx.org/en/docs/http/ngx_http_core_module.html#error_page) :

```
server {
    ...
    error_page 401 = @error401;
    location @error401 {
        return 302 https://login.example.com;
    }
}
```


Si la demande n'est pas autorisée, nous redirigerons l'utilisateur vers **https://login.example.com** aide du code d'état 302.  
Ici, l'utilisateur recevra un message d'erreur approprié et la possibilité de l'autoriser.

Nous devons maintenant transporter le jeton d'autorisation du client d'un système à un autre. Après avoir été autorisé à **login.example.com** , l'utilisateur reçoit un cookie contenant le jeton d'authentification. Le cookie est défini sur .example.com' afin que **staticpage.example.com** puisse également accéder au jeton. Tout ce que nous avons à faire maintenant, c’est de passer le jeton du cookie au serveur d’authentification.

Nous utilisons `$http_cookie ~* "shopware_sso_token=([^;]+)(?:;|$)"` pour faire correspondre le jeton du cookie de l'utilisateur, suivi d'un proxy_set_header pour le transmettre au backend.

```
location = /auth {
    ...

    if ($http_cookie ~* "shopware_sso_token=([^;]+)(?:;|$)") {
        set $token "$1";
    }
    proxy_set_header X-SHOPWARE-SSO-Token $token;
}
```

### Mission accomplie

Maintenant, **api.example.com** est capable de décider si la demande nécessite une authentification (jeton manquant ou expiré) et de répondre avec le code d'état 401.  
Pour les utilisateurs authentifiés mais non autorisés , il répond avec un code 403.  
Si l'utilisateur est authentifié et autorisé, il répond avec un code 200.

### appendice

```
server {
    server_name staticpage.example.com;

    root /var/www/staticpage.example.com/;

    error_page 401 = @error401;
    location @error401 {
        return 302 https://login.example.com;
    }

    auth_request /auth;

    location = /auth {
        internal;

        proxy_pass https://api.example.com;

        proxy_pass_request_body     off;

        proxy_set_header Content-Length "";
        proxy_set_header X-Original-URI $request_uri;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        if ($http_cookie ~* "shopware_sso_token=([^;]+)(?:;|$)") {
            set $token "$1";
        }
        proxy_set_header X-SHOPWARE-SSO-Token $token;
    }
}
```

## nginx-sso - SSO hors ligne simple pour nginx

nginx-sso est un système SSO (Single Sign-On) léger, hors ligne, qui fonctionne avec les cookies et ECDSA. Il peut facilement être utilisé avec vanilla nginx et n’importe quelle application dorsale.   L'implémentation de référence est écrite en golang et possède quelques fonctionnalités supplémentaires intéressantes telles que l'autorisation.

### SSO simple sur le Web


Remote-User Remote-Groups

Vous pouvez choisir entre Google, Facebook, LinkedIn, GitHub et plus de sites, couvrant parfaitement votre base d'utilisateurs. L'externalisation de l'authentification auprès de ces personnes est une meilleure idée que de (mal) gérer vous-même les informations d'identification des utilisateurs!

Bien que ces options soient définitivement la voie à suivre pour la plupart des applications, il existe des scénarios où elles ne fonctionnent pas:

*    Vous pourriez ne pas faire confiance à ces fournisseurs.
*    Vos applications peuvent vivre «hors ligne», par exemple dans un réseau d'entreprise.
*    Vous voudrez peut-être protéger les ressources statiques.
*    Vous pourriez être choqué par la complexité de systèmes tels que OpenID Connect. 

Une autre réalité de nos jours est que votre application risque de s’exécuter derrière un autre processus HTTP. De nombreuses personnes (moi-même inclus) utilisent aujourd'hui nginx pour mettre fin à TLS, aux demandes d'équilibrage de charge, etc. C'est pourquoi j'ai décidé de créer ma propre SSO légère, qui fonctionne avec nginx et des applications arbitraires.

### Fonctions  

Lors de la conception de nginx-sso, j'ai proposé une liste de fonctionnalités nécessaires et utiles:

*    Travailler hors connexion: ni l'utilisateur ni le service ne doivent parler à Internet.
*    Travail déconnecté: il n'est pas nécessaire que le service communique avec le fournisseur d'identité.
*    Sécurisé: la compromission d'un service ne doit affecter aucun autre service.
*    Fournir une authentification pour les applications dorsales.
*    Fournissez une autorisation pour accéder aux URI et aux applications / sites dorsaux.
*    Soyez simple à comprendre et à configurer.
*    Travailler avec nginx sans nécessiter de correction / construction manuelle de nginx ni de maintenance de modules en dehors de l’arbre. 

### Solutions SSO basées sur les cookies

En ce qui concerne l'authentification unique déconnectée / hors ligne, il est évident que l'utilisateur fournira ses propres informations d'identification au serveur d'applications, qui doit décider s'il est légitime ou non. Cela signifie qu’il faut vérifier l’intégrité et l’authenticité de la revendication des utilisateurs, ces deux tâches étant généralement accomplies à l’aide de MAC ou de signatures .

Pour moi, les MAC n'étaient pas une option, car un attaquant pourrait émettre ses propres tickets en compromettant un seul serveur d'applications. Cela laisse des signatures à clé publique, basées sur DSA ou ECC. Les signatures ECC sont le meilleur choix car elles sont plus efficaces et prennent moins de place dans un cookie.

nginx-sso utilise un cookie simple avec une signature supplémentaire ECSDA. La signature est faite sur la charge du cookie (nom d'utilisateur, groupes), ainsi que sur l'horodatage d'expiration et l'adresse IP de l'utilisateur.

### Authentification

L'authentification à l'aide de vanilla nginx est possible principalement grâce à un impressionnant nginx-plugin appelé [auth_request](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html) . Sur le site:

>Le module ngx_http_auth_request_module (1.5.4+) implémente l'autorisation du client en fonction du résultat d'une sous-demande. Si la sous-demande renvoie un code de réponse 2xx, l'accès est autorisé. S'il renvoie 401 ou 403, l'accès est refusé avec le code d'erreur correspondant. Tout autre code de réponse renvoyé par la sous-demande est considéré comme une erreur.


Avec ce module, chaque accès aux ressources configurées sur votre serveur nginx déclenchera une requête HTTP vers un serveur d’authentification.  
Cette demande contiendra les en-têtes de la demande originale que votre serveur d’authentification utilise pour accorder ou refuser l’accès.  
Le module auth_request n'est pas compilé par défaut ou dans toutes les distributions, mais il fait partie de la baseline nginx codebase et les distributions majeures ont des packages compilés avec ce module.

Ici, nous pouvons voir l'extrait de configuration nginx qui protège la ressource /secret avec une sous-requête à la ressource interne **/auth** qui envoie un proxy au serveur ssoauth qui s'exécute sur localhost.

```
  location = /auth {
    internal;
    proxy_pass http://127.0.0.1:8082;
    proxy_pass_request_body     off;
    proxy_set_header Content-Length "";
    proxy_set_header X-Original-URI $request_uri;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
  }

  location /secret {
    auth_request /auth;
  }
``` 

### Passer des variables à l'application

Bien que je sois capable d'effectuer l'authentification et l'autorisation dans le backend auth, cela n'a toujours pas aidé mon application backend à identifier l'utilisateur. Heureusement, j'ai découvert comment transmettre les variables renvoyées par le point final auth aux applications dorsales , voir l'exemple nginx.conf . Maintenant, le service backend peut simplement assumer la présence et l'exactitude de ces en-têtes HTTP et ne pas avoir à gérer le cookie sso .

```
location /secret {
  auth_request /auth;

  auth_request_set $user $upstream_http_remote_user;
  proxy_set_header Remote-User $user;
  auth_request_set $groups $upstream_http_remote_groups;
  proxy_set_header Remote-Groups $groups;
  auth_request_set $expiry $upstream_http_remote_expiry;
  proxy_set_header Remote-Expiry $expiry;

  [...]
  proxy_information_for_backend_application;
  [...]
}
```

### Autorisation

Comme un effet secondaire intéressant, étant donné que nous effectuons déjà une sous-demande pour chaque requête HTTP, nous pouvons également utiliser le point de terminaison **auth** pour effectuer une autorisation.   
Pour ce faire, j'ai mis en place une ACL très rudimentaire dans ssoauth. Il contient une liste de vhosts et de préfixes et pour chacun d’eux une liste d’utilisateurs et de groupes autorisés.  
De cette façon, même les ressources statiques peuvent être facilement protégées.

```
"acl": {
  "auth.domain.dev:8080": {
    "Users": ["jg123456"],
    "Groups": ["x:"],
    "UrlPrefixes": {
      "/secret/": {
        "Users": ["ba514378", "jb759123"],
        "Groups": ["y:engineering:cloud"]
      }
    }
  }
}
```

### Protéger les applications avec nginx-sso

Si vous écrivez une application personnalisée et souhaitez utiliser ce système (ou un système similaire), rien de plus simple.  
Vous pouvez simplement utiliser les entêtes `Remote-User`  `Remote-Groups`  pour effectuer une autorisation, par exemple en disant que *Tout le monde du groupe xyz* est un *administrateur*  
Vous pouvez également disposer de votre propre base de données d'utilisateurs et utiliser uniquement l'en `Remote-User` pour créer et rechercher ultérieurement le bon utilisateur.  
De cette façon, vous pouvez avoir des attributs (et des autorisations) supplémentaires pour chaque utilisateur.

Si vous utilisez un logiciel de stock, vous pourrez peut-être aussi utiliser ce schéma. La plupart des logiciels prennent en charge la connexion via l' Remote-User , même si le logiciel implémente ensuite sa propre base de données d'utilisateurs. Pour les logiciels à source fermée, vous pouvez parfois trouver des plug-ins qui activent cette fonctionnalité.

## Liens

* [Comment nous avons remplacé OpenResty par NGINX ?](https://jolicode.com/blog/comment-nous-avons-remplace-openresty-par-nginx)
* [Nginx Auth Request Module](https://github.com/perusio/nginx-auth-request-module)
* [SSO with Nginx auth_request module](https://developers.shopware.com/blog/2015/03/02/sso-with-nginx-authrequest-module/)
* [Décrit une configuration de base de l'utilisation de auth_request avec des cookies](https://developers.shopware.com/blog/2015/03/02/sso-with-nginx-authrequest-module/).
* [Discussion sur Hackernews traitant de auth_request et de différentes approches de l'authentification unique](https://news.ycombinator.com/item?id=7641148).
* [Le code sur GitHub](https://github.com/heipei/nginx-sso/)
* [Sécuriser les cookies de son application web](https://www.mon-code.net/article/103/Securiser-les-cookies-de-son-application-web)



