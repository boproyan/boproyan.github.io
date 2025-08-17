+++
title = 'Configuration nginx , ssl (tls1.2 tls1.3) , Diffie Hellman ,Headers et OCSP'
date = 2020-06-05 00:00:00 +0100
categories = ['ssl']
+++
## Configuration nginx

fichier **/etc/nginx/ssl_dh_header_ocsp**

* Les certificats *Let's Encrypt* du domaine dans **/etc/ssl/private/**
* Remplacer le domaine **xoyize.xyz** par le votre 
* Diffie-Hellman : `openssl dhparam -out /etc/ssl/private/dh2048.pem -outform PEM -2 2048`

```
    ##
    # SSL Settings
    ##
    ssl_certificate /etc/ssl/private/xoyize.xyz-fullchain.pem;
    ssl_certificate_key /etc/ssl/private/xoyize.xyz-key.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    ssl_dhparam /etc/ssl/private/dh2048.pem;

    # intermediate configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;


    add_header Content-Security-Policy "upgrade-insecure-requests; "; 
    add_header Content-Security-Policy "default-src 'self'; font-src *;img-src * data:; script-src *; style-src *;";
    add_header Feature-Policy "geolocation none;midi none;microphone none;camera none;fullscreen self;payment none;";

    # Add headers to serve security related headers
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    add_header X-Frame-Options "SAMEORIGIN"; 
    add_header Referrer-Policy "no-referrer" always;

    add_header Strict-Transport-Security 'max-age=31536000; includeSubDomains; preload';

    # OCSP settings
    ssl_stapling on;
    ssl_stapling_verify on;
    #ssl_trusted_certificate /etc/ssl/private/xoyize.xyz-fullchain.pem;
    ssl_trusted_certificate /etc/ssl/private/ocsp-certs.pem;
    resolver 127.0.0.1;

```

Le fichier est à inclure dans tous les fichiers de configuration des VHOST , exemple

    /etc/nginx/conf.d/sub.xoyize.xyz.conf 

```
server {
    listen 80;
    listen [::]:80;

    ## redirect http to https ##
    server_name sub.xoyize.xyz;
    return  301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name sub.xoyize.xyz;

    include ssl_dh_headers_ocsp;

    location / {
	proxy_pass http://192.168.0.45:4040;
    }

}
```

### Que sont les dossiers de la CAA ?

* [CAA record checking now mandatory for Certificate Authorities](https://dnsspy.io/blog/caa-record-checking-now-mandatory-certificate-authorities/)
* [CAA Record Generator](https://sslmate.com/caa/)

CAA signifie "Certificate Authority Authorization" et est un type spécial d'enregistrement DNS qui indique quelles autorités de certification peuvent émettre des certificats pour votre domaine et sous-domaines. Il a également une méthode d'envoi d'alertes chaque fois qu'une autorité de certification reçoit une demande pour un nouveau certificat qui est interdit selon vos dossiers CAA.

DNS OVH &rarr; Domaines (sélectionner le domaine) &rarr; **Zone DNS** &rarr; **Ajouter une entrée** de type CAA

        3600 IN CAA    0 issue "letsencrypt.org"

Vérification : [CAA record check](https://dnsspy.io/labs/caa-validator)

### SSL cipher dh

Modifier le fichier **/etc/nginx/conf.d/default.conf** ou le fichier de configuration du VHOST

```
    # certificats ssl let's encrypt
    ssl_certificate /etc/ssl/private/xinyiczen.xyz.crt.pem;
    ssl_certificate_key /etc/ssl/private/xinyiczen.xyz.key.pem;

    ssl_session_timeout 5m;
    ssl_session_cache shared:SSL:50m;
    ssl_prefer_server_ciphers on;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ALL:!aNULL:!eNULL:!LOW:!EXP:!RC4:!3DES:+HIGH:+MEDIUM;

    # Clé Diffie-Hellman (DH)
    ssl_dhparam /etc/ssl/private/dh4096.pem;
```


### OCSP Stapling (Online Certificate Status Protocol )

Mise en place de l’OCSP Stapling (Online Certificate Status Protocol ) sur NGINX  
On va télécharger les certificats que Let’s Encrypt utilise pour signer les certificats qu’il délivre.   
Let’s Encrypt utilise uniquement Let’s Encrypt Authority X3 pour signer les certificats et garde Let’s Encrypt Authority X4 en cas de problèmes. X1 et X2 ne sont plus utilisés.  
Intégration de X3 et X4 dans un fichier par concaténation  :

```
cd /etc/ssl/private
rm -f ocsp-certs.pem
wget -O- https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem https://letsencrypt.org/certs/lets-encrypt-x4-cross-signed.pem | tee -a ocsp-certs.pem > /dev/null
```
 

Modification du fichier de configuration principal **/etc/nginx/nginx.conf**:

    nano /etc/nginx/nginx.conf

 Et ajoutez ceci dans la balise **http {**  :

```nginx
    ##OCSP settings
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/ssl/private/ocsp-certs.pem;
    # Facultatif (utiise par défaut les DNS contenus dans resolv.conf)
    #resolver 8.8.4.4 8.8.8.8 valid=300s;
    #resolver_timeout 5s;
```
 

### headers

Tout ce qui suit peut être à l'intérieur d'un fichier **.conf** situé soit sous **/etc/nginx/conf.d** ou dans le cas d'une installation **yunohost** , sous **/etc/nginx/conf.d/domaine.tld.d/**  
**domaine.tld** est à remplacé par le nom du domaine déclaré à l'installation de yunohost


L’en-tête **CSP** permet d’autoriser seulement les domaines déclarés à exécuter du script JavaScript, une feuille de style css, etc... (ici domaine xinyiczen.xyz et uniquement en https)

[Content Security Policy](https://wiki.mozilla.org/Security/Guidelines/Web_Security#Content_Security_Policy)

```
    ## CSP nginx
    # add_header Content-Security-Policy "default-src 'self' *.xinyiczen.xyz; img-src 'self' data: https: *.xinyiczen.xyz"; # TROP BLOQUANT
    add_header Content-Security-Policy "default-src https: data: 'unsafe-inline' 'unsafe-eval'";
```

[X-Frame-Options ](https://wiki.mozilla.org/Security/Guidelines/Web_Security#X-Frame-Options)

```
    # autoriser uniquement les Iframes provenant du même domaine
    add_header X-Frame-Options SAMEORIGIN;
```

[X-XSS-Protection](https://wiki.mozilla.org/Security/Guidelines/Web_Security#X-XSS-Protection)

```
    # activer la protection contre les attaques Xss et MIME
    add_header X-XSS-Protection "1; mode=block";
```

[X-Content-Type-Options](https://wiki.mozilla.org/Security/Guidelines/Web_Security#X-Content-Type-Options)

```
    add_header X-Content-Type-Options nosniff;
```

###  Vérification de la mise en place des headers  

Pour la prise en compte des ajouts dans la configuration du block serveur, il faut relancer le service nginx :

    systemctl restart nginx

Avec l’aide de la commande **curl**, on va vérifier les différentes en-têtes (**headers**) :

    curl -I https://xinyiczen.xyz

Résultat : 

```
HTTP/1.1 200 OK
Server: nginx/1.10.1
Date: Thu, 24 Nov 2016 09:14:27 GMT
Content-Type: text/html
Content-Length: 868
Last-Modified: Wed, 23 Nov 2016 19:43:09 GMT
Connection: keep-alive
ETag: "5835f14d-364"
Strict-Transport-Security: max-age=31536000;includeSubDomains
Content-Security-Policy: default-src 'self' *.xinyiczen.xyz; img-src 'self' data: https: *.xinyiczen.xyz
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Accept-Ranges: bytes
```


### Liens

 * <https://angristan.fr/configurer-https-nginx/>  
 * <https://www.noobunbox.net/serveur/securite/securiser-https-nginx>



