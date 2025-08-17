+++
title = 'Générer des certificats SSL auto-signés avec mkcert'
date = 2024-02-21 00:00:00 +0100
categories = ['ssl']
+++
![Texte alternatif](certificat-removebg.png){:height="100"} ![](ssl-logoA.png){:height="100"}

**mkcert** *est un outil facile d’utilisation qui va se charger de tout. Il génère notre autorité de certification locale, qui servira à signer le(s) certificat(s). Il suffira de déployer sa clé sur toutes les machines clientes pour que nous n’ayons aucune erreur du type “self signed…”*

### Installation de mkcert

MKCert est disponible dans les dépôts officiels de la plupart des distributions Linux

```
sudo pacman -S mkcert                # archlinux
sudo apt install mkcert              # debian
sudo apk add mkcert                  # alpine linux
```

### Générer autorité de certification CA

Pour que les certificats soient considérés comme valides, il faut que l’autorité de certification soit ajoutée au système de confiance (trust-store) de notre hôte.

    mkcert -install

```
Created a new local CA 💥
The local CA is now installed in the system trust store! ⚡️

# si déjà effectué
The local CA is already installed in the system trust store! 👍
```

Il est possible de copier notre CA pour l’ajouter sur d’autres machines (pour que les certificats soient considérés comme valides par ces machines).

    mkcert -CAROOT 

Renvoie

    /home/bookvm/.local/share/mkcert

Affichons le contenu :

```shell
ls -l /home/bookvm/.local/share/mkcert
total 8
-r-------- 1 bookvm bookvm 2484 Nov 18 19:25 rootCA-key.pem
-rw-r--r-- 1 bookvm bookvm 1643 Nov 18 19:25 rootCA.pem
```

Pour vérifier l'autorité de certification AC

    openssl x509 -noout -text -in /home/bookvm/.local/share/mkcert/rootCA.pem 

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            5b:97:fc:a2:92:63:65:59:48:10:90:e8:64:24:49:cb
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: O = mkcert development CA, OU = bookvm@vm-debian12, CN = mkcert bookvm@vm-debian12
        Validity
            Not Before: Nov 18 19:25:03 2023 GMT
            Not After : Nov 18 19:25:03 2033 GMT
        Subject: O = mkcert development CA, OU = bookvm@vm-debian12, CN = mkcert bookvm@vm-debian12
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (3072 bit)
```

La clé publique de cette autorité de certification doit être ajoutée sur les machines (pour que celle-ci accepte les certificats signés par cette autorité).
{: .prompt-info }

pour une machine distante 

    scp $(mkcert -CAROOT)/rootCA.pem vm-debian.fr:/usr/local/share/ca-certificates/mkcert.crt
    ssh vm-debian.fr sudo update-ca-certificates

```
Updating certificates in /etc/ssl/private...
rehash: warning: skipping ca-certificates.crt,it does not contain exactly one certificate or CRL
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

### Génération d’un certificat

Sur la machine locale (ayant généré l’autorité de certification)

    mkcert -cert-file bookvm.loc.crt -key-file bookvm.loc.key bookvm.loc fmy.bookvm.loc gpx.bookvm.loc iptv.bookvm.loc rss.bookvm.loc static.bookvm.loc test.bookvm.loc traduction.bookvm.loc

Résultat

```
Created a new certificate valid for the following names 📜
 - "bookvm.loc"
 - "fmy.bookvm.loc"
 - "gpx.bookvm.loc"
 - "iptv.bookvm.loc"
 - "rss.bookvm.loc"
 - "static.bookvm.loc"
 - "test.bookvm.loc"
 - "traduction.bookvm.loc"

The certificate is at "bookvm.loc.crt" and the key at "bookvm.loc.key" ✅

It will expire on 21 May 2026 🗓
```

Le certificat se retrouve dans le dossier où vous avez exécuté la commande mkcert. On se retrouve avec deux fichiers :

```shell
ls -l bookvm*
-rw-r--r-- 1 bookvm bookvm 1700 Feb 21 17:07 bookvm.loc.crt
-rw------- 1 bookvm bookvm 1704 Feb 21 17:07 bookvm.loc.key
```

### Test de la clé

On va simplement installer nginx, sur notre machine et le configurer avec notre clé (remplacer les noms des fichiers du certificat avec les vôtres)

```shell
sudo su
apt install -y nginx
cp bookvm.loc.crt /etc/ssl/private
cp bookvm.loc.key /etc/ssl/private

cat <<EOF >>/etc/nginx/conf.d/bookvm.loc.conf
server {
  listen  80;
  server_name bookvm.loc;
  return 301 https://$server_name$request_uri;
  root /var/www/html;
}
server {
  listen *:443 ssl http2;
  server_name bookvm.loc;
  ssl_certificate /etc/ssl/private/bookvm.loc.crt;
  ssl_certificate_key /etc/ssl/private/bookvm.loc.key;
  root /var/www/html;
}
EOF

echo "127.0.0.1 bookvm.loc.key bookvm.loc fmy.bookvm.loc gpx.bookvm.loc iptv.bookvm.loc rss.bookvm.loc static.bookvm.loc test.bookvm.loc traduction.bookvm.loc" >> /etc/hosts
echo Test >> /var/www/html/test.html
```

On teste :

    curl -I https://bookvm.loc/index.html

```shell
HTTP/2 200 
server: nginx/1.24.0
date: Wed, 21 Feb 2024 17:39:39 GMT 00:00:00 +0100
last_modified_at: 2024-02-21
content-type: text/html
content-length: 7086
last-modified: Mon, 04 Dec 2023 15:16:35 GMT
etag: "656ded53-1bae"
accept-ranges: bytes
```

Pas d’erreur, le chiffrement est bien en place !

### Diffuser sa chaîne d'authentification

*Installation de la clé de l’autorité de certification sur toutes les machines*

#### Sur les machines de type Archlinux

```shell
sudo cp ~/.local/share/mkcert/rootCA.pem /etc/ca-certificates/trust-source/anchors/rootCA.crt
# Faites de même avec tous les fichiers /etc/ssl/private/*.pem ajoutés manuellement et renommez-les en *.crt
sudo  trust extract-compat
```

#### Sur les machines de type Debian 

```shell
sudo cp ~/.local/share/mkcert/rootCA.pem /usr/local/share/ca-certificates/rootCA.crt
sudo apt install -y ca-certificates
sudo update-ca-certificates
Updating certificates in /etc/ssl/private...
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

#### Sur les machines de type Redhat

```shell
sudo cp ~/.local/share/mkcert/rootCA.pem /etc/pki/ca-trust/source/anchors/rootCA.crt
sudo dnf install -y ca-certificates
sudo update-ca-trust
```

On teste :

```shell
curl https://bookvm.loc
Test
```

