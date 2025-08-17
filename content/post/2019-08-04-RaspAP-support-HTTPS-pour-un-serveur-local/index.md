+++
title = 'RaspAP , support HTTPS pour un serveur local'
date = 2019-08-04 00:00:00 +0100
categories = ['raspberry', 'ssl']
+++
## Support HTTPS

*HTTPS a besoin de certificats TLS et, bien que le déploiement de sites Web publics soit en grande partie un problème résolu grâce au protocole ACME et à Let's Encrypt, les serveurs Web locaux utilisent encore principalement HTTP car <u>personne ne peut obtenir un certificat universellement valide pour localhost</u>.*

### Certificats de confiance locaux

La meilleure solution consiste à gérer votre propre autorité de certification, mais elle nécessite généralement une procédure de configuration manuelle complexe. [Mkcert](https://github.com/FiloSottile/mkcert) est une excellente solution pour les sites Web locaux . Il s'agit d'un outil de configuration zéro permettant de créer des certificats de confiance locaux avec le nom de votre choix. mkcert crée et installe automatiquement une autorité de certification locale dans le magasin racine du système et génère des certificats approuvés localement. Cela fonctionne aussi parfaitement bien avec RaspAP. Cela vous permet de générer un certificat de confiance pour un nom d’hôte (par exemple, raspap.local) ou une adresse IP car il ne fonctionne que pour vous.

![raspap.local](raspaplocal.png){:width="200"}

* **Particularité:**  
    * il ne génère pas de certificats auto-signés, <u>mais des certificats signés par votre propre autorité de certification privée</u>.  
    * Cet outil <u>ne configure pas automatiquement les serveurs ou les clients mobiles pour utiliser les certificats</u>, à vous de choisir (voir la procédure ci-dessous).

En savoir plus sur mkcert [ici](https://blog.filippo.io/mkcert-valid-https-certificates-for-localhost/) et [suivez le projet sur GitHub](https://github.com/FiloSottile/mkcert). 

### Comment procéder

Suivez les étapes ci-dessous pour générer et installer un certificat de confiance locale pour RaspAP.  
Le domaine local raspap.local est utilisé dans les exemples ci-dessous.  
Vous pouvez le remplacer par le raspberrypi.local par défaut raspberrypi.local ou par votre propre nom d'hôte.

1 - Installer le binaire pré-construit (voir <https://github.com/FiloSottile/mkcert/releases>) 

```
sudo wget https://github.com/FiloSottile/mkcert/releases/download/v1.3.0/mkcert-v1.3.0-linux-arm -O /usr/local/bin/mkcert
sudo chmod +x /usr/local/bin/mkcert
mkcert -install # opération très longue, patienter...
```

Vous devriez voir une sortie comme celle-ci:

```
Created a new local CA at "/home/pi/.local/share/mkcert" 💥
The local CA is now installed in the system trust store! ⚡️
```

2 - Générer un certificat pour yanhotspot.loc : 

    mkcert yanhotspot.loc "*.yanhotspot.loc" yanhotspot.loc 

... et une sortie comme celle-ci:

```
Using the local CA at "/home/pi/.local/share/mkcert" ✨

Created a new certificate valid for the following names 📜
 - "yanhotspot.loc"
 - "*.yanhotspot.loc"
 - "yanhotspot.loc"

Reminder: X.509 wildcards only go one level deep, so this won't match a.b.yanhotspot.loc ℹ️

The certificate is at "./yanhotspot.loc+2.pem" and the key at "./yanhotspot.loc+2-key.pem" ✅
```

3 - Combiner la clé privée et le certificat: 

    cd /home/pi
    cat yanhotspot.loc+2-key.pem yanhotspot.loc+2.pem > yanhotspot.loc.pem

4 - Créer un répertoire pour le fichier .pem combiné dans lighttpd: 

    sudo mkdir /etc/lighttpd/ssl

5 - Définir les autorisations et déplacer le fichier .pem : 

    chmod 400 /home/pi/yanhotspot.loc.pem
    sudo mv /home/pi/yanhotspot.loc.pem /etc/lighttpd/ssl

6 - Éditer la configuration de lighttpd: 

    sudo nano /etc/lighttpd/lighttpd.conf

... et ajoutez le bloc suivant pour activer SSL avec votre nouveau certificat:

```
server.modules += ("mod_openssl")
$SERVER["socket"] == ":443" {
  ssl.engine = "enable"
  ssl.pemfile = "/etc/lighttpd/ssl/yanhotspot.loc.pem"
  ssl.ca-file = "/home/pi/.local/share/mkcert/rootCA.pem"
  server.name = "yanhotspot.loc"
  server.document-root = "/var/www/html"
}
```

7 - Redémarrer le service lighttpd: 

    sudo systemctl restart lighttpd 

8 - Vérifier que lighttpd a redémarré sans erreur: 

    sudo systemctl status lighttpd 

Vous devriez voir une réponse comme celle-ci:

```
● lighttpd.service - Lighttpd Daemon
   Loaded: loaded (/lib/systemd/system/lighttpd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2019-08-04 20:15:16 CEST; 4s ago
  Process: 3297 ExecStartPre=/usr/sbin/lighttpd -tt -f /etc/lighttpd/lighttpd.conf (code=exited, status=0/SUCCESS)
 Main PID: 3302 (lighttpd)
   Memory: 25.5M
   CGroup: /system.slice/lighttpd.service
           ├─3302 /usr/sbin/lighttpd -D -f /etc/lighttpd/lighttpd.conf
           ├─3310 /usr/bin/php-cgi
           ├─3311 /usr/bin/php-cgi
           ├─3312 /usr/bin/php-cgi
           ├─3313 /usr/bin/php-cgi
           └─3314 /usr/bin/php-cgi

août 04 20:15:14 raspberrypi systemd[1]: Starting Lighttpd Daemon...
août 04 20:15:16 raspberrypi systemd[1]: Started Lighttpd Daemon.
```

9 - Maintenant, copier **rootCA.pem** sur votre racine Web lighttpd ( important: ne partagez PAS rootCA-key.pem ): 

    sudo cp /home/pi/.local/share/mkcert/rootCA.pem /var/www/html 

10 - Ouvrir un navigateur et entrer l'adresse suivante: <https://yanhotspot.loc/rootCA.pem> .  
Acceptez l'avertissement non sécurisé dans le navigateur et téléchargez le certificat racine sur votre client.  
Importer le certificat racine (firefox).   
Préférences &rarr; Vie privée et sécurité &rarr; Certificats : Afficher les certificats  
Onglet *Autorités* **Importer** , sélectionner **rootCA.pem**
![](firefox-importer-certificat-racine.png){:width="600"}  
Ouvrir le lien <https://yanhotspot.loc> sur le navigateur firefox  
![](raspap-web.png){:width="300"}  
*Le verrou vert nous indique que la connexion est sécurisée*

