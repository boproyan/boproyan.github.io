+++
title = 'Certificat Wildcard Let's Encrypt sur un domaine hébergé par OVH'
date = 2021-06-04 00:00:00 +0100
categories = ssl
+++
* Pré-requis : Un nom de domaine hébergé chez OVH - Un serveur dédié/VPS

La première chose à faire, c'est d'attribuer l'IP du serveur au nom de domaine. Pour cela, nous nous rendons sur l'espace client d'OVH. Une fois connectés, nous nous rendons dans l'onglet, Web => Domaines => votre-domaine.tld => Zone DNS.

Maintenant que votre IP est attribuée au nom de domaine, on va pouvoir rentrer dans le vif du sujet.

Le client ACME que j'utilise pour la gestion des certificats Let's Encrypt s'appelle "acme.sh". On le trouve facilement sur [GitHub](https://github.com/Neilpang/acme.sh).  
Acme.sh n'a pas besoin des droits root pour fonctionner, je me place donc dans mon home sur le serveur => /home/toto

Pour l'installer :

    wget -O -  https://get.acme.sh | sh 

Durant la courte installation, vous aurez certainement un message en rouge marqué :

```
It is recommended to install socat first.
We use socat for standalone server if you use standalone mode.
If you don't use standalone mode, just ignore this warning.
```

Acme.sh recommande l'installation du paquet "socat" pour gérer les certificats en mode standalone, mais comme nous n'allons pas utiliser ce mode, vous pouvez largement ignorer ce message.

Vous trouverez le dossier d'installation dans => /home/toto/.acme.sh  
Maintenant, il faut régénérer le fichier ".bashrc" de l'utilisateur pour pouvoir utiliser immédiatement les commandes. Pour cela, fait simplement un bash  
Pour que le <u>client se mette à jour automatiquement</u>, je vous conseille d'activer cette option :

    acme.sh --auto-upgrade

Quand vous allez taper cette dernière commande, la liste de toutes les commandes possible s'afficheront. Mais si vous voulez les revoir :

    acme.sh --help

Pour générer des certificats Wildcard et les renouveler automatiquement, nous devons passer par l'API d'OVH. Pour ce faire, nous allons créer une application dans l'API.  
On se rend sur l'[API d'OVH](https://eu.api.ovh.com/createApp/).  
Inscrivez vos informations de connexion à votre compte OVH ou le domaine y est enregistré, puis dans Application Name et Application Description mettez ce que vous voulez.Exemple:

    Application name : acme.sh
    Application description : Create certifcate for acme.sh

En cliquant sur "Create Keys" vous avez une nouvelle page qui s'ouvre avec vos clés.  
**Sauvegardé => Application Key et Application Secret**  

Maintenant, nous retournons sur le serveur, dans notre home et nous allons utiliser notre Application Key et Application Secret.

    export OVH_AK="votre application key"
    export OVH_AS="votre application secret"

Ensuite, on va <u>lancer une première fois</u> la génération du certificat :

    acme.sh --issue --keylength ec-384 -d domaine.tld -d '*.domaine.tld' --dns dns_ovh 

Vous devriez voir apparaitre dans votre shell, quelque chose de similaire à :

```
Registering account
Registered
ACCOUNT_THUMBPRINT='M7HOIDF6546B8s_FPp4tJYODH5468Ka5MII'
Creating domain key
The domain key is here: /home/toto/.acme.sh/domaine.tld_ecc/domaine.tld.key
Multi domain='DNS:domaine.tld,DNS:*.domaine.tld'
Getting domain auth token for each domain
Getting webroot for domain='domaine.tld'
Getting webroot for domain='*.domaine.tld'
Found domain api file: /home/toto/.acme.sh/dnsapi/dns_ovh.sh
Using OVH endpoint: ovh-eu
OVH consumer key is empty, Let's get one:
Please open this link to do authentication: https://eu.api.ovh.com/auth/?credentialToken=swtntIJFflOEmX7oOIDFOHFIH6464684688964FOUGHUIGGDFGFibhawiivUdgMZi4r
Here is a guide for you: https://github.com/Neilpang/acme.sh/wiki/How-to-use-OVH-domain-api
Please retry after the authentication is done.
Error add txt for domain:_acme-challenge.domaine.tld
Please add '--debug' or '--log' to check more details.
See: https://github.com/Neilpang/acme.sh/wiki/How-to-debug-acme.sh
```

À la ligne 13, vous avez un lien, recopiez le, puis ouvrez le dans votre navigateur. 

* Réinscrivez vos identifiants de connexion à OVH 
* à la ligne VALIDITY mettez UNLIMITED. 
* En validant, vous avez une nouvelle page qui s'ouvre sur GitHub avec marqué => OVH SUCCESS. C'est que tout est bon 

Vous pouvez maintenant, relancer votre commande pour générer définitivement votre certificat :

    acme.sh --issue --keylength ec-384 -d domaine.tld -d '*.domaine.tld' --dns dns_ovh 

Il faudra patienter environs 2 minutes pour que le client test si tout est OK. Après ce long moment d'attente, vous verrez apparaître dans le shell :

```
Your cert is in /home/toto/.acme.sh/domaine.tld_ecc/domaine.tld.cer
Your cert key is in /home/toto/.acme.sh/domaine.tld_ecc/domaine.tld.key
The intermediate CA cert is in /home/toto/.acme.sh/domaine.tld_ecc/ca.cer
And the full chain certs is there: /home/toto/.acme.sh/domaine.tld_ecc/fullchain.cer
J'utilise Nginx en reverse proxy avec Docker, donc je place mes certificats dans => /home/toto/docker/nginx/ssl avec une très belle commande :
acme.sh --install-cert -d domaine.tld --ecc \
--ca-file /home/toto/docker/nginx/ssl/chain.pem \
--cert-file /home/toto/docker/nginx/ssl/cert.pem \
--key-file /home/toto/docker/nginx/ssl/privkey.pem \
--fullchain-file /home/toto/docker/nginx/ssl/fullchain.pem \
--reloadcmd "cd /home/toto/docker && docker-compose restart nginx"
```

La dernière ligne est super, car elle permet par exemple, de redémarrer automatiquement mon conteneur nginx quand le client acme.sh renouvelle le certificat.  

Vous voilà avec un certificat Wildcard Let's Encrypt qui se renouvelle automatiquement. Vous pourrez le voir avec la commande "crontab -l", il y a bien une tâche CRON créée par acme.sh qui s'occupe de tout.
