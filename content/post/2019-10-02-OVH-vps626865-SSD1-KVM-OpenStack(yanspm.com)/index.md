+++
title = 'OVH VPS SSD 1 KVM OpenStack (1 vCore/2GoRam/20GoSSD) domaine yanspm.com'
date = 2019-10-02 00:00:00 +0100
categories = ['serveur']
+++
# Debian Stretch yanspm.com

## KVM OpenStack OVH (2 GB Mémoire, 1 CPU, 20 GB SSD, 100 Mbps)

![Debian 9](debian9a.png){:width="100"}

Package: 2 GB Mémoire, 1 CPU, 20 GB SSD, 100 Mbps  
Selected Location: Strasbourg  
Debian Stretch 64  
Livraison : vps626865 

* IPv4 du serveur : 51.77.192.45
* IPv6 du serveur : 2001:41d0:0404:0200:0000:0000:0000:0c11
* IPV6 gateway    : 2001:41d0:0404:0200:0000:0000:0000:0001

### VPS OVH , activation IPV6

Connexion sur "OVH8 VPS SSD 3 KVM 2 vCore(s) Ram 8Go SSD 40Go Debian Stretch"  

    ssh root@51.77.192.45

Sur le VPS OVH il faut désactiver l'initialisation réseau par le cloud  

```
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
```

Création du fichier **/etc/cloud/cloud.cfg.d/99-disable-network-config.cfg** en mode su

    echo "network: {config: disabled}" > /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg

Le fichier **/etc/network/interfaces** 

```
auto lo
iface lo inet loopback

auto ens3
iface ens3 inet dhcp
iface ens3 inet6 static
 address 2001:41d0:0404:0200:0000:0000:0000:0c11
 netmask 128
 post-up /sbin/ip -6 route add 2001:41d0:0404:0200:0000:0000:0000:0001 dev ens3
 post-up /sbin/ip -6 route add default via 2001:41d0:0404:0200:0000:0000:0000:0001 dev ens3
 pre-down /sbin/ip -6 route del default via 2001:41d0:0404:0200:0000:0000:0000:0001 dev ens3
 pre-down /sbin/ip -6 route del 2001:41d0:0404:0200:0000:0000:0000:0001 dev ens3
```

>Redémarrer la machine `systemctl reboot` pour la prise en compte des modifications du réseau

Vérifier le réseau `ip addr` 

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:27:b7:ff brd ff:ff:ff:ff:ff:ff
    inet 51.77.192.45/32 brd 51.77.192.45 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 2001:41d0:404:200::c11/128 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe27:b7ff/64 scope link 
       valid_lft forever preferred_lft forever
```

### Mise à jour Debian et installation des utilitaires

Màj  

    apt update && apt upgrade


Installer utilitaires  

    apt install rsync curl tmux jq figlet git mailutils dnsutils -y

### Configurer Locales (fr et en)

Locales : **fr_FR.UTF-8** et **en_US.UTF-8**  

    dpkg-reconfigure locales

```bash
Generating locales (this might take a while)...
  fr_FR.UTF-8... done
  en_US.UTF-8... done
Generation complete.
```

### Europe/Paris (TimeZone tzdata)

Europe/Paris  

    dpkg-reconfigure tzdata

```
Current default time zone: 'Europe/Paris'
Local time is now:      Sun Mar 18 19:49:55 CET 2018.
Universal Time is now:  Sun Mar 18 18:49:55 UTC 2018.
```

### Création utilisateur

Utilisateur **yanspm**  

    useradd -m -d /home/yanspm/ -s /bin/bash yanspm

Mot de passe **yanspm**  

    passwd yanspm

Visudo pour les accès root via utilisateur **yanspm**  

```bash
apt install sudo  # installé par défaut
echo "yanspm     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

Changer le mot de passe root

    passwd root

Déconnexion puis connexion ssh en mode utilisateur  

    ssh yanspm@51.77.192.45

### Hostname

Exécuter les instructions suivantes en mode su  
Configuration OVH à modifier **/etc/cloud/cloud.cfg** 

    preserve_hostname: true
    manage_etc_hosts: false

Modifier **hostname**  

    hostnamectl set-hostname --static yanspm.com

Modifier **/etc/hosts**  

    nano /etc/hosts

```
127.0.1.1 	yanspm.com vps626865
127.0.0.1       localhost
```

Vérifications

    hostnamectl

```
   Static hostname: yanspm.com
         Icon name: computer-vm
           Chassis: vm
        Machine ID: ae99d55cea1e4ccabe7432b2fa05d502
           Boot ID: 99269b96a32745bd868938d0f717bb89
    Virtualization: kvm
  Operating System: Debian GNU/Linux 9 (stretch)
            Kernel: Linux 4.9.0-8-amd64
      Architecture: x86-64
```

	hostname --fqdn
		yanspm.com

### OVH DNS fournisseur domaine

Domaine **yanspm.com** distribué par OVH

```
$TTL 86400
@	IN SOA dns101.ovh.net. tech.ovh.net. (2018121601 86400 3600 3600000 300)
    3600 IN NS     dns101.ovh.net.
    3600 IN NS     ns101.ovh.net.
    3600 IN A      51.77.192.45
    3600 IN AAAA   2001:41d0:404:200::c11
*   3600 IN CNAME  yanspm.com.
```


### OpenSSH, clé et script

![OpenSSH](ssh_logo1.png){:width="100"}

**connexion avec clé**  
<u>sur l'ordinateur de bureau</u>
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) nommé **kvm-cinay** pour une liaison SSH avec le serveur KVM.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/kvm-vps626865

Envoyer la clé publique sur le serveur KVM   

    scp ~/.ssh/kvm-vps626865.pub yanspm@51.77.192.45:/home/yanspm/

<u>sur le serveur KVM</u>
On se connecte  

    ssh yanspm@51.77.192.45

Copier le contenu de la clé publique dans /home/$USER/.ssh/authorized_keys  

    cd ~

Sur le KVM ,créer un dossier .ssh  

```bash
mkdir .ssh
cat $HOME/kvm-vps626865.pub >> $HOME/.ssh/authorized_keys
```

et donner les droits  

    chmod 600 $HOME/.ssh/authorized_keys

effacer le fichier de la clé  

    rm $HOME/kvm-vps626865.pub

Modifier la configuration serveur SSH  

    sudo nano /etc/ssh/sshd_config

Modifier

```conf
Port = 55031
PermitRootLogin no
PasswordAuthentication no
```


<u>session SSH ne se termine pas correctement lors d'un "reboot" à distance</u>  
Si vous tentez de **redémarrer/éteindre** une machine distance par **ssh**, vous pourriez constater que votre session ne se termine pas correctement, vous laissant avec un terminal inactif jusqu'à l'expiration d'un long délai d'inactivité. Il existe un bogue 751636 à ce sujet. Pour l'instant, la solution de contournement à ce problème est d'installer :  

    sudo apt-get install libpam-systemd

cela terminera la session ssh avant que le réseau ne tombe.  
Veuillez noter qu'il est nécessaire que PAM soit activé dans sshd.  

Relancer openSSH  

    sudo systemctl restart sshd

Accès depuis le poste distant avec la clé privée  

    ssh -p 55031 -i ~/.ssh/kvm-vps626865 yanspm@yanspm.com

**Exécution script sur connexion**  
Exécuter un fichier *utilisateur* nommé **$HOME/.ssh/rc** si *présent*  
Pour *tous les utilisateurs* exécuter un fichier nommé **/etc/ssh/sshrc** si *présent*  
Installer les utilitaires *curl jq figlet*  

Le batch  

    nano ~/.ssh/rc

```bash
#!/bin/bash

#clear
PROCCOUNT=`ps -Afl | wc -l`  		# nombre de lignes
PROCCOUNT=`expr $PROCCOUNT - 5`		# on ote les non concernées
GROUPZ=`users`
ipinfo=$(curl -s ipinfo.io) 		# info localisation format json
publicip=$(echo $ipinfo | jq -r '.ip')  # extraction des données , installer préalablement "jq"
ville=$(echo $ipinfo | jq -r '.city')
pays=$(echo $ipinfo | jq -r '.country')
cpuname=`cat /proc/cpuinfo |grep 'model name' | cut -d: -f2 | sed -n 1p`
iplink=`ip link show |grep -m 1 "2:" | awk '{print $2}' | cut -d: -f1`

echo "\033[0m\033[1;31m"  
figlet "`hostname --fqdn`"
echo "\033[0m
\033[1;35m    \033[1;37mHostname \033[1;35m= \033[1;32m`hostname`
\033[1;35m  \033[1;37mWired IpV4 \033[1;35m= \033[1;32m`ip addr show $iplink | grep 'inet\b' | awk '{print $2}' | cut -d/ -f1`
\033[1;35m  \033[1;37mWired IpV6 \033[1;35m= \033[1;32m`ip addr show $iplink | grep -E 'inet6' |grep -E 'scope link' | awk '{print $2}' | cut -d/ -f1`
\033[1;35m      \033[1;37mKernel \033[1;35m= \033[1;32m`uname -r`
\033[1;35m      \033[1;37mDebian \033[1;35m= \033[1;32m`cat /etc/debian_version`
\033[1;35m      \033[1;37mUptime \033[1;35m= \033[1;32m`uptime | sed 's/.*up ([^,]*), .*/1/' | sed -e 's/^[ \t]*//'`
\033[1;35m         \033[1;37mCPU \033[1;35m= \033[1;32m`echo $cpuname`
\033[1;35m  \033[1;37mMemory Use \033[1;35m= \033[1;32m`free -m | awk 'NR==2{printf "%s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }'`
\033[1;35m    \033[1;37mUsername \033[1;35m= \033[1;32m`whoami`
\033[1;35m    \033[1;37mSessions \033[1;35m= \033[1;32m`who | grep $USER | wc -l`
\033[1;35m \033[1;37mPublic IpV4 \033[1;35m= \033[1;32m`echo $publicip`
\033[1;35m \033[1;37mPublic IpV6 \033[1;35m= \033[1;32m`ip addr show $iplink | grep -m 1 'inet6\b'  | awk '{print $2}' | cut -d/ -f1`
\033[0m"
df -h /
#curl fr.wttr.in/$ville?0
```

Effacer motd  

    sudo rm /etc/motd

Déconnexion puis connexion  

### systemd/journal

Ajout de l'utilisateur courant au groupe systemd-journal  

    sudo gpasswd -a $USER systemd-journal

Accès utilisateur aux fichiers log     

    sudo gpasswd -a $USER adm

Après déconnexion puis reconnexion , l'utilisateur a accès au journal:  

    journalctl

Pour avoir les lignes NON TRONQUEES  

    export SYSTEMD_LESS=FRXMK journalctl

Pour un mode permanent ,modifier **~/.bashrc**

    echo "export SYSTEMD_LESS=FRXMK journalctl" >> ~/.bashrc

### Nginx PHP7 MariaDb

![lemp](/images/nginx-php7-mariadb1.png "lemp"){:width="100"}

* [PHP 7.3 for Nginx](https://websiteforstudents.com/install-php-7-3-php-7-3-fpm-with-apache2-nginx-on-ubuntu-16-04-18-04-18-10/)
* [Debian 9.x + PHP 7.3.x](http://jc.etiemble.free.fr/abc/index.php/trucs-astuces/lamp/deb9php73)

Script nommé **debian9-compilation-nginx-tls1.3-php7.3-MariaDB.sh.txt** pour compiler et installer nginx, openssl + tlsv1.3, php7.3 et MariaDB sur debian stretch [Téléchargement script]({{ site.url }}/files/debian9-compilation-nginx-tls1.3-php7.3-MariaDB.sh.txt) 

Définition des chemins et fichiers de configuration nginx  
**/etc/nginx/conf.d/yanspm.com.conf** configuration de base du domaine  
Création dossier **/etc/nginx/conf.d/yanspm.com.d/** pour les fichiers de configuration supplémentaires    

	sudo mkdir -p /etc/nginx/conf.d/yanspm.com.d

Déplacer et renommer le fichier de configuration par défaut  

	sudo mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/yanspm.com.conf

Modifier le fichier   

	sudo nano /etc/nginx/conf.d/yanspm.com.conf

```
server {
    listen 80;
    listen [::]:80;
    root /var/www/ ;
        location ~ \.php$ {
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           fastcgi_pass unix:/run/php/php7.3-fpm.sock;   # PHP7.3
           fastcgi_index index.php;
           include fastcgi_params;
	   fastcgi_param SCRIPT_FILENAME $request_filename;
        }
}
```

Vérifier

    sudo nginx -t

Recharger  

    sudo systemctl reload nginx

Activer le service

    sudo systemctl enable nginx

**MariaDb**

Installer MariaDb :

    sudo apt install mariadb-server -y

Initialiser le mot de passe root ( ) + sécurisation

    sudo mysql_secure_installation

```
Enter current password for root (enter for none):
Set root password? [Y/n] y
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

**MariaDB mot de passe pour root**  
Même après avoir exécuté *mysql_secure_installation*, l’utilisateur **root** n’a toujours pas besoin de mot de passe.  
Voici la solution 

```
sudo -s
mysql -u root
MariaDB [(none)]> use mysql;
MariaDB [mysql]> update user set plugin='' where User='root';
MariaDB [mysql]> flush privileges;
```

>ATTENTION!!! L'utilisation d'un mot de passe "root" entraîne une erreur de rotation des logs sur mysql 

Message d'erreur  

```
/etc/cron.daily/logrotate:
mysqladmin: connect to server at 'localhost' failed
error: 'Access denied for user 'root'@'localhost' (using password: NO)'
error: error running shared postrotate script for '/var/log/mysql/mysql.log /var/log/mysql/mysql-slow.log /var/log/mysql/mariadb-slow.log /var/log/mysql/error.log '
run-parts: /etc/cron.daily/logrotate exited with return code 1
```

Il faut un mot de passe pour se connecter en "root" à la base mysql qui n'est pas fourni dans la commande du fichier de configuration logrotate **/etc/logrotate.d/mysql-server**, modifier le fichier  
Remplacer les lignes 

```
            mysqladmin --defaults-file=/etc/mysql/debian.cnf --local flush-error-log \
              flush-engine-log flush-general-log flush-slow-log
```

Par les lignes suivantes

```
            mysqladmin --defaults-file=/etc/mysql/debian.cnf --local flush-error-log \
              flush-engine-log flush-general-log flush-slow-log \
              -uroot -p$(cat /etc/mysql/mdp ) 
```

Lancer manuellement "logrotate" pour vérification

	/usr/sbin/logrotate /etc/logrotate.conf


**MariaDb installer la version 10.3 (janvier 2019)**

Ajouter la clé et le dépôt

```
sudo apt-get install software-properties-common dirmngr
sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xF1656F24C74CD1D8
sudo add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://ftp.igh.cnrs.fr/pub/mariadb/repo/10.3/debian stretch main'
```

Installer mariadb 10.3

```
sudo apt-get update
sudo apt-get install mariadb-server
```


### Unbound (résolveur DNS)

![DNS Unbound](unbound-250.png){:width="100"}

*Les serveurs DNS sont des machines discutant entre elles afin de se communiquer les correspondances entre nom de domaine et adresses IP. Voir le lien [DNS Unbound](/posts/unbound-resolveur-DNS/)*

Adresse mail dans la variable `REPORT_EMAIL="admin@desti.tld"` du fichier **/etc/unbound/dnsunbound-update-root-dns.sh**


### Quad9 (résolveur DNS sécurisé)

![quad9](quad9-logo.png){:width="100"}  
A utiliser avec unbound  
Voir le lien [Quad9 résolveur DNS public accessible de manière sécurisée (DNS sur TLS)](/posts/quad9-dns-resolver-secure/)


### Certificats SSL letsencrypt (acme)

![SSL Letsencrypt](letsencrypt-logo1.png){:width="100"}  
[installer et renouveler les certificats SSL Let's encrypt ](https://yann.cinay.eu/2017/08/31/Acme-Certficats-Serveurs/)



Installer acme

    cd ~
    sudo -s  # en mode super utilisateur
    apt install netcat -y  # prérequis
    git clone https://github.com/Neilpang/acme.sh.git
    cd acme.sh
    ./acme.sh --install # --nocron
    cd ..
    rm -rf acme.sh/

Ajouter les variables pour l'accès api OVH

```
export OVH_AK="Lj45UFRTDESDnnh5"
export OVH_AS="J582RTDESDnnh5rhDE89ZFSxng58aKL"
```

Génération des certificats (le wildcard est autorisé)

	/root/.acme.sh/acme.sh --dns dns_ovh --issue --keylength ec-384 -d 'yanspm.com' -d '*.yanspm.com'

Il faut s'identifier chez OVH , un lien est fourni dans le résultat `Please open this link to do authentication: https://eu.api.ovh.com/auth/?credentialToken=kdfyjvuioiogg265bhbgdfbghh`  
Sélectionner `validity : unlimited` et **login** , vous obtiendrez le message suivant `OVH authentication Success !` .  
Il faut relancer la commande et patienter quelques minutes...

	/root/.acme.sh/acme.sh --dns dns_ovh --issue --keylength ec-384 -d 'yanspm.com' -d '*.yanspm.com'

Certificats  

```
...
[jeudi 23 mai 2019, 10:40:08 (UTC+0200)] Your cert is in  /root/.acme.sh/yanspm.com_ecc/yanspm.com.cer 
[jeudi 23 mai 2019, 10:40:08 (UTC+0200)] Your cert key is in  /root/.acme.sh/yanspm.com_ecc/yanspm.com.key 
[jeudi 23 mai 2019, 10:40:09 (UTC+0200)] The intermediate CA cert is in  /root/.acme.sh/yanspm.com_ecc/ca.cer 
[jeudi 23 mai 2019, 10:40:09 (UTC+0200)] And the full chain certs is there:  /root/.acme.sh/yanspm.com_ecc/fullchain.cer 
```

Création des liens sur **/etc/ssl/private** pour nginx  

```
ln -s /root/.acme.sh/yanspm.com_ecc/fullchain.cer /etc/ssl/private/yanspm.com-fullchain.pem     # full chain certs
ln -s /root/.acme.sh/yanspm.com_ecc/yanspm.com.key /etc/ssl/private/yanspm.com-key.pem           # cert key
ln -s /root/.acme.sh/yanspm.com_ecc/yanspm.com.cer /etc/ssl/private/yanspm.com-chain.pem          # cert
ln -s /root/.acme.sh/yanspm.com_ecc/ca.cer /etc/ssl/private/yanspm.com-ca.pem                   # intermediate CA cert
```

Vérification de la mise à jour automatique  

	crontab -e

		9 0 * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null

>NOTE: Sous-domaines **nc.yanspm.com -> Nextcloud , imap smtp et mail**

### Nginx + SSL + Diffie-Hellmann + Entêtes

**ssl**  
Il faut préalablement demander des certificats (ca+key) SSL pour le domaine  auprès d'une autorité de certification (let's encrypt ou autre)  
Le fichier de configuration  

	sudo nano /etc/nginx/ssl_params

```
    ssl_certificate /etc/ssl/private/yanspm.com-fullchain.pem;
    ssl_certificate_key /etc/ssl/private/yanspm.com-key.pem;
    ssl_session_timeout 5m;
    ssl_session_cache shared:SSL:50m;
    ssl_prefer_server_ciphers on;

    # Ciphers with modern compatibility
    # New protocol TLSv1.3
    ssl_protocols TLSv1.3 TLSv1.2;
    ssl_ciphers 'TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-256-GCM-SHA384:TLS13-AES-128-GCM-SHA256:EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

```

**Diffie-Hellmann**  
Générer une clé Diffie-Hellmann  
*L’algorithme Diffie-Hellman est un algorithme d’échange de clés, utilisé notamment lors de l’ouverture d’une connexion à un site sécurisé via le protocole SSL/TLS.*

Générer une clé Diffie Hellman (patienter quelques minutes)

    sudo openssl dhparam -out /etc/ssl/private/dh2048.pem -outform PEM -2 2048

Ajouter la ligne suivante au fichier **ssl_params**

    ssl_dhparam /etc/ssl/private/dh2048.pem;

**Entêtes**  
Le fichier de configuration  

	sudo nano /etc/nginx/header_params

```
    # Follows the Web Security Directives from the Mozilla Dev Lab and the Mozilla Obervatory + Partners
    # https://wiki.mozilla.org/Security/Guidelines/Web_Security
    # https://observatory.mozilla.org/ 
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"; 
    add_header Content-Security-Policy "upgrade-insecure-requests";
    add_header Content-Security-Policy-Report-Only "default-src https: data: 'unsafe-inline' 'unsafe-eval'";
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    add_header X-Frame-Options "SAMEORIGIN";
```


On change le dossier racine  

    sudo mkdir /var/www/default-www
    sudo mv /var/www/index/ /var/www/default-www/

Configuration de base avec SSL et sécurité + letsencrypt (renouvellement)  

	sudo nano /etc/nginx/conf.d/yanspm.com.conf

```
server {
    listen 80;
    listen [::]:80;

    ## redirect http to https ##
    server_name yanspm.com;
    return  301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name yanspm.com;

    ####    Locations
    # On cache les fichiers statiques
    location ~* \./|css|js|png|jpg|jpeg|gif|ico|svg|eot|woff|ttf)$ { expires max; }
    # On interdit les dotfiles
    location ~ /\. { deny all; }

    include ssl_params;
    include header_params;

    root /var/www/default-www ;
    index index.php index/ index.htm;
        location ~ \.php$ {
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           fastcgi_pass unix:/run/php/php7.3-fpm.sock;   # PHP7.3
           fastcgi_index index.php;
           include fastcgi_params;
	   fastcgi_param SCRIPT_FILENAME $request_filename;
        }

    # if folder yanspm.com.d , uncomment the following directive
    #include conf.d/yanspm.com.d/*.conf;

    access_log /var/log/nginx/yanspm.com-access.log;
    error_log /var/log/nginx/yanspm.com-error.log;
}

```

Vérifier  

	sudo nginx -t

Relancer  

	sudo systemctl reload nginx

Test redirection http/https avec curl depuis un poste distant  

	curl -I yanspm.com

```
HTTP/1.1 301 Moved Permanently
Server: nginx/1.15.5
Date: Sat, 22 Dec 2018 16:13:31 GMT
Content-Type: text/
Content-Length: 169
Connection: keep-alive
Location: https://yanspm.com/
```

Tester le lien <https://yanspm.com>  

Vérifier les entêtes depuis un autre poste 

    curl -I https://yanspm.com

```
HTTP/2 200 
server: nginx/1.15.5
content-type: text/
content-length: 867
last-modified: Wed, 19 Dec 2018 16:41:58 GMT
etag: "5c1a74d6-363"
expires: Thu, 31 Dec 2037 23:55:55 GMT
cache-control: max-age=315360000
strict-transport-security: max-age=63072000; includeSubDomains; preload
content-security-policy: upgrade-insecure-requests
content-security-policy-report-only: default-src https: data: 'unsafe-inline' 'unsafe-eval'
x-content-type-options: nosniff
x-xss-protection: 1; mode=block
x-download-options: noopen
x-permitted-cross-domain-policies: none
x-frame-options: SAMEORIGIN
accept-ranges: bytes
```

### parefeu (iptables V4 V6)

![parefeu Iptables](iptables1.png){:width="100"}  
[installer un parefeu serveur](https://yann.cinay.eu/2018/10/08/Pare-feu-iptables-IPv4-IPv6-versions-bureau-et-serveur/)

## Applications

### Rainloop webmail

![Rainloop](rainloop.png){:width="50"}  

* [Rainloop](http://rainloop.net/)  
* [Installation And Configuration Of  RainLoop Webmail Client With Nginx](https://www.howtoforge.com/installation-and-configuratio-of-rainloop-webmail-client-with-nginx-on-ubuntu-14.04)
* [Installing the RainLoop Email Client on Ajenti V ](https://www.digitalocean.com/community/tutorials/installing-the-rainloop-email-client-on-ajenti-v)

On va utiliser le domaine **webmail.yanspm.com**  

**MariaDB**  
Créer dans Mariadb une base de données *rainloopdb*, utilisateur *rainloopuser* et un mot de passe situé sous */etc/mysql/rainlooppassword*   
Pour info le mot de passe root de la base mysql est situé sous */etc/mysql/mdp*  

Génération du mot de passe de la base *rainloopdb*

    sudo -s
    echo $(date +%s | sha256sum | base64 | head -c 20 ;) > /etc/mysql/rainlooppassword

Création de la base mysql *rainloopdb*

    sudo -s

```
mysql -u root -p$(cat /etc/mysql/mdp) <<EOF
create database rainloopdb;
GRANT ALL PRIVILEGES ON rainloopdb.* TO 'rainloopuser'@'localhost' IDENTIFIED BY 'rainlooppassword';
flush privileges;
exit;
EOF
```

>REMPLACER *rainlooppassword* par le contenu du fichier */etc/mysql/rainlooppassword*

**Dépendances PHP7.3**

    sudo apt install php7.3-fpm  php7.3-mysql php7.3-cli php7.3-curl php7.3-sqlite

Ce serveur utilise PHP7.3 , l'extension **mcrypt** n'a pas été reconduite , il faut installer la dernière version connue php7.1-mcrypt

    sudo apt install php7.1-mcrypt

```
Creating config file /etc/php/7.1/mods-available/calendar.ini with new version
Creating config file /etc/php/7.1/mods-available/ctype.ini with new version
Creating config file /etc/php/7.1/mods-available/exif.ini with new version
Creating config file /etc/php/7.1/mods-available/fileinfo.ini with new version
Creating config file /etc/php/7.1/mods-available/ftp.ini with new version
Creating config file /etc/php/7.1/mods-available/gettext.ini with new version
Creating config file /etc/php/7.1/mods-available/iconv.ini with new version
Creating config file /etc/php/7.1/mods-available/pdo.ini with new version
Creating config file /etc/php/7.1/mods-available/phar.ini with new version
Creating config file /etc/php/7.1/mods-available/posix.ini with new version
Creating config file /etc/php/7.1/mods-available/shmop.ini with new version
Creating config file /etc/php/7.1/mods-available/sockets.ini with new version
Creating config file /etc/php/7.1/mods-available/sysvmsg.ini with new version
Creating config file /etc/php/7.1/mods-available/sysvsem.ini with new version
Creating config file /etc/php/7.1/mods-available/sysvshm.ini with new version
Creating config file /etc/php/7.1/mods-available/tokenizer.ini with new version
Paramétrage de libmcrypt4 (2.5.8-3.3) ...
Paramétrage de php7.1-mcrypt (7.1.25-1+0~20181207224650.11+stretch~1.gbpf65b84) ...

Creating config file /etc/php/7.1/mods-available/mcrypt.ini with new version
```

**Installer rainloop**  

Créer le dossier du site web (en mode su)

    mkdir -p /var/www/rainloop/{public/,logs}
    cd /var/www/rainloop/public//
    curl -s http://repository.rainloop.net/installer.php | php

```
#!/usr/bin/env php


       [RainLoop Webmail Installer]


 * Connecting to repository ...
 * Downloading package ...
 * Complete downloading!
 * Installing package ...
 * Complete installing!

 * [Success] Installation is finished!
```

Création du fichier de configuration nginx pour rainloop

    nano /etc/nginx/conf.d/rainloop.conf

```
server {
    listen 80;
    listen [::]:80;

    ## redirect http to https ##
    server_name webmail.yanspm.com;
    return  301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name webmail.yanspm.com;

    root /var/www/rainloop/public/;
    index index.php;

    include ssl_params;
    include header_params;
    # Diffie-Hellmann
    # Uncomment the following directive after DH generation
    # > openssl dhparam -out /etc/ssl/private/dh4096.pem -outform PEM -2 4096
    # ssl_dhparam /etc/ssl/private/dh4096.pem;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

        location ~ \.php$ {
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           fastcgi_pass unix:/run/php/php7.3-fpm.sock;   # PHP7.3
           fastcgi_index index.php;
           include fastcgi_params;
	   fastcgi_param SCRIPT_FILENAME $request_filename;
        }

    location ~ /\.ht {
        deny all;
    }

    location ^~ /data {
      deny all;
    }

    # if folder yanspm.com.d , uncomment the following directive
    #include conf.d/yanspm.com.d/*.conf;

    access_log /var/www/rainloop/logs/access.log;
    error_log /var/www/rainloop/logs/error.log;
}
```

Vérification et rechargement nginx

    nginx -t
    systemctl reload nginx

Les permissions du dossier web rainloop

    chown -R www-data: /var/www/rainloop/public//

**Configurer rainloop (mode administration)**  
Accèder au panneau d'administration avec la configuration par défaut <https://webmail.yanspm.com/?admin> utilisateur *admin* et mot de passe *12345*

![](rainloop-a.png)  
Cliquer sur change et modifier le mot de passe admin, se reconnecter   
![](rainloop-admin.png){:width="300"}  

**Général**  
![](rainloop-b.png){:width="400"}  

**Domaines**  
![](rainloop-c.png){:width="400"}  

**Contacts**  
![](rainloop-d.png){:width="400"}  

**Sécurité ,authentification en deux étapes**  
![](rainloop-e.png){:width="400"}  
Se déconnecter du mode administration, puis se connecter en utilisateur sur l'un des domaines <https://webmail.yanspm.com>   
Aller dans les paramètres et sécurité  
![](rainloop2fa-1.png){:width="300"}  
Cliquer sur le lien ,"Configurer l'authentification en deux étapes"  

![](rainloop2fa-01.png){:width="300"}  
Cliquer sur **Activer**  

![](rainloop2fa-02.png){:width="400"}  
scanner le code QR avec l'application "And OTP" android, faire le test du code et ensuite activer (copier la clé secrète pour une utilisation sur un autre périphérique) puis cliquer sur **Fait** pour terminer l'opération.

>**L'opération "authentification en deux étapes" doit être effectuée pour chaque domaine ayant un utilisateur**

### Shaarli

![image](shaarli_logo.png){:width="50px"}  
*Voulez-vous partager les liens que vous découvrez ? Shaarli est un gestionnaire de signets minimaliste et un service de partage de liens que vous pouvez installer sur votre propre serveur. Il est conçu pour être personnel (monoposte), rapide et pratique.*

Utlisation domaine shaarli.yanspm.com

**Installation shaarli**  
Dans la plupart des cas, vous devriez télécharger la dernière version de Shaarli depuis la [page des versions](https://github.com/shaarli/Shaarli/releases). Téléchargez notre archive shaarli-full pour inclure les dépendances.

```
wget https://github.com/shaarli/Shaarli/releases/download/v0.10.2/shaarli-v0.10.2-full.zip
unzip shaarli-v0.10.2-full.zip
sudo mv Shaarli /var/www/
sudo mkdir -p /var/log/shaarli # stockage des logs
```

Création du fichier de configuration nginx pour shaarli

    sudo -s
    nano /etc/nginx/conf.d/shaarli.conf

```
server {
    listen 80;
    listen [::]:80;

    ## redirect http to https ##
    server_name shaarli.yanspm.com;
    return  301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name shaarli.yanspm.com;

    root /var/www/Shaarli;
    index index.php;

    include ssl_params;
    include header_params;
    # Diffie-Hellmann
    # Uncomment the following directive after DH generation
    # > openssl dhparam -out /etc/ssl/private/dh4096.pem -outform PEM -2 4096
    # ssl_dhparam /etc/ssl/private/dh4096.pem;

        location ~ \.php$ {
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           fastcgi_pass unix:/run/php/php7.3-fpm.sock;   # PHP7.3
           fastcgi_index index.php;
           include fastcgi_params;
	   fastcgi_param SCRIPT_FILENAME $request_filename;
        }

    # if folder yanspm.com.d , uncomment the following directive
    #include conf.d/yanspm.com.d/*.conf;

    access_log /var/log/shaarli/shaarli_access.log;
    error_log /var/log/shaarli/shaarli_error.log;
}
```

Vérification et rechargement nginx

    nginx -t
    systemctl reload nginx

Les permissions du dossier web shaarli

    chown -R www-data: /var/www/Shaarli/

Ouvrir le lien <https://shaarli.yanspm.com> pour poursuivre l'installation

![](shaarli01.png){:width="500"}   
Après l'installation, se connecter en utilisateur/mot de passe   
Paramétrer *Français*, **Tools** &rarr; **Configure your Shaarli**  

### NetData

![](netdata-logomark.png){:width="50"}  
[NetData](/posts/Netdata-Performance-Monitoring-Tool/)

Après installation de base  
Modifier la configuration par défaut **/etc/nginx/conf.d/yanspm.com.conf** et ajouter avant le } de fin

```
    location /stub_status {
        stub_status;
        # Security: Only allow access from the IP below.
        allow 127.0.0.1;
        # Deny anyone else
        deny all;
    }
```

Vérifier la configuration de nginx et redémarrer le service

    nginx -t
    systemctl restart nginx

Le module ‘stub_status’de Nginx a été activé, vérifier avec la commande curl

    curl https://yanspm.com/stub_status

```
Active connections: 1 
server accepts handled requests
 3 3 2 
Reading: 0 Writing: 1 Waiting: 1 
```

Pour surveiller Nginx à l’aide de Netdata, nous avons besoin du module’stub_status’de Nginx qui est activé.  
Créer le fichier python.d/nginx.conf

    nano /etc/netdata/python.d/nginx.conf

```
localhost:
  name : 'local'
  url  : 'https://yanspm.com/stub_status'
```

Redémarrez maintenant Netdata en utilisant systemctl.

    systemctl restart netdata

La configuration de Netdata pour la surveillance de Nginx est terminée.  

Créer un proxy nginx pour accès à netdata **/etc/nginx/conf.d/netdata.conf**

```
upstream backend {
    # the netdata server
    server 127.0.0.1:19999;
    keepalive 64;
}

server {
    listen 80;
    listen [::]:80;

    ## redirect http to https ##
    server_name netdata.yanspm.com;
    return  301 https://$server_name$request_uri;

}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name netdata.yanspm.com;

    include ssl_params;
    include header_params;
    # Diffie-Hellmann
    # Uncomment the following directive after DH generation
    # > openssl dhparam -out /etc/ssl/private/dh4096.pem -outform PEM -2 4096
    # ssl_dhparam /etc/ssl/private/dh4096.pem;

#    include conf.d/netdata.yanspm.com.d/*.conf;
    location / {
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_pass_request_headers on;
        proxy_set_header Connection "keep-alive";
        proxy_store off;
    }

    access_log /var/log/nginx/netdata-access.log;
    error_log /var/log/nginx/netdata-error.log;
}
```

Vérification et rechargement nginx

    nginx -t
    systemctl reload nginx

Accès <https://netdata.yanspm.com>

![](netdata01.png){:width="600"}


### Gitea

![gitea](gitea-logo.png)

*Gitea est une interface web permettant de visualiser les branches, les commits, les tags, etc. de ses projets git. Il a une utilisation proche de celle de GitHub, tout en restant très léger.*

* [Systemd,lancer une application au démarrage](/posts/Systemd-service-pour-lancer-application/)

**Installation**  
On installe le binaire gitea dans le "home" utilisateur  

    mkdir $HOME/.gitea
    cd ~/.gitea   
    # Télécharger le binaire Gitea
    wget https://dl.gitea.io/gitea/1.6.2/gitea-1.6.2-linux-amd64 -O gitea
    chmod +x gitea

Lancer l'application au démarrage  
**Créer le service systemd "gitea.service"** 

    sudo nano  /etc/systemd/system/gitea.service

```
[Unit]
Description=Gitea gestion web des projets git

[Service]
Type=simple
ExecStart=/home/yanspm/.gitea/gitea web
User=yanspm   
Group=yanspm

[Install]
WantedBy=multi-user.target
```

Lancement du service

    sudo systemctl daemon-reload
    sudo systemctl start gitea.service

Vérifier status

    sudo systemctl status gitea.service

```
● gitea.service - Affiche l'heure sur un YoctoDisplay
   Loaded: loaded (/etc/systemd/system/gitea.service; disabled; vendor preset: enabled)
   Active: active (running) since Fri 2019-01-04 14:21:28 CET; 1min 8s ago
 Main PID: 32138 (gitea)
    Tasks: 7 (limit: 4915)
   CGroup: /system.slice/gitea.service
           └─32138 /home/yanspm/.gitea/gitea web

janv. 04 14:21:28 yanspm.com gitea[32138]: 2019/01/04 14:21:28 [T] Log path: /home/yanspm/.gitea/
log
janv. 04 14:21:28 yanspm.com gitea[32138]: 2019/01/04 14:21:28 [I] Gitea v1.6.2 built with: binda
ta, sqlite
janv. 04 14:21:28 yanspm.com gitea[32138]: 2019/01/04 14:21:28 [I] Log Mode: Console(Info)
janv. 04 14:21:28 yanspm.com gitea[32138]: 2019/01/04 14:21:28 [I] XORM Log Mode: Console(Info)
janv. 04 14:21:28 yanspm.com gitea[32138]: 2019/01/04 14:21:28 [I] Cache Service Enabled
janv. 04 14:21:28 yanspm.com gitea[32138]: 2019/01/04 14:21:28 [I] Session Service Enabled
janv. 04 14:21:28 yanspm.com gitea[32138]: 2019/01/04 14:21:28 [I] SQLite3 Supported
janv. 04 14:21:28 yanspm.com gitea[32138]: 2019/01/04 14:21:28 [I] Run Mode: Development
janv. 04 14:21:28 yanspm.com gitea[32138]: 2019/01/04 14:21:28 Serving [::]:3000 with pid 32138
janv. 04 14:21:28 yanspm.com gitea[32138]: 2019/01/04 14:21:28 [I] Listen: http://0.0.0.0:3000
```

Activer le service

    sudo systemctl enable gitea.service

**Créer proxy nginx pour l'accès à gitea**

    sudo nano /etc/nginx/conf.d/gitea.conf

```
server {
    listen 80;
    listen [::]:80;

    ## redirect http to https ##
    server_name gitea.yanspm.com;
    return  301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name gitea.yanspm.com;

    include ssl_params;
    include header_params;

    location / { 
	proxy_pass http://127.0.0.1:3000; 
    } 

    access_log /var/log/nginx/gitea-access.log;
    error_log /var/log/nginx/gitea-error.log;
}

```

Vérifier  

	sudo nginx -t

Relancer  

	sudo systemctl reload nginx

Accès au site par le lien <https://gitea.yanspm.com>

**MariaDB/MySQL base gitea**  
Version MariadDB/MySQL  

    mysql --version
    mysql  Ver 15.1 Distrib 10.3.11-MariaDB, for debian-linux-gnu (x86_64) using readline 5.2

>ATTENTION MariaDB version > 10.1 sinon erreur...

Créer dans Mariadb une base de données *gitea*, utilisateur *gitea* et un mot de passe situé sous */etc/mysql/gitea*   
Pour info le mot de passe root de la base mysql est situé sous */etc/mysql/mdp*  

Génération du mot de passe de la base *gitea*

    sudo -s
    echo $(date +%s | sha256sum | base64 | head -c 12 ;) > /etc/mysql/gitea

Création de la base mysql *gitea*

    sudo -s

```
mysql -u root -p$(cat /etc/mysql/mdp)
create database gitea;
GRANT ALL PRIVILEGES ON gitea.* TO 'gitea'@'localhost' IDENTIFIED BY 'giteapassword';
flush privileges;
exit;
```

>REMPLACER *giteapassword* par le contenu du fichier */etc/mysql/gitea*


**Utiliser "gitea"**  
La première connexion au site <https://gitea.yanspm.com>  , cliquer sur **s'inscrire**  
**Configuration initiale** ,saisir le mot de passe de la base mysql  
**Configuration générale** ,modifier le titre du site, désactiver le port du serveur SSH  
Cliquer sur **"Installer Gitea"**

Se connecter de nouveau, créer un compte puis créer un dépôt , par exemple "wikistatic"  

**Poste contenant les sources des dépôts**"  
Du poste qui contient les sources de notre application "wikistatic"  
Renseigner le fichier **~/.git-credential** , https://yannick:mot-de-passe@gitea.yanspm.com
Renseigner le fichier **~/.gitconfigl** 

```
[user]
	name = Gitea Yanspm
	email = yannick@cinay.xyz
[credential]
	helper = store
[core]
	editor = nano

[http]
	postBuffer = 524288000
```

Première initialisation du dépôt après sa création sous gitea.yanspm.com

```
cd ~/media/gitea/wikistatic  # se rendre dans le dossier contenant les sources
git init                     # initialisation
git add .                    # on prend le conteu du dossier
git commit -m "Initialisation"
git remote add origin https://gitea.yanspm.com/yannick/wikistatic.git
git push -u origin master
```

### msmtp , envoyer des messages

Avec l'utilitaite ssmtp , on utilise un serveur smtp existant pour envoyer des messages  
Voir le [lien](/posts/Debian-ssmtp-msmtp-Mise-A-jour-automatique-avec-Cron-APT/) paragraphe "Méthode 3 : msmtp"  
Les opérations suivantes se font en mode su  

    apt --purge remove exim4*   # suppression exim
    apt install mailutils msmtp msmtp-mta # installation msmtp et utilitaire

Le fichier de configuration

    nano ~/.msmtprc

Le contenu

```
account default
tls on
host messagerie.xyz   # smtp.messagerie.xyz
port 587
from user@messagerie.xyz
auth on
user user@messagerie.xyz  # user
password <mot de passe>
tls_starttls on
tls_certcheck on
tls_trust_file /etc/ssl/certs/ca-certificates.crt

logfile ~/.msmtp.log
```

#### Vérifications et tests

On peut tout simplement vérifier que tout est correctement configuré en s’envoyant un email avec cette commande : 

    printf "Subject:Test expéditeur\nCeci est un test" | msmtp destinataire@desti.tld

Pour que la commande **mail** fonctionne, vous devrez mettre ce qui suit dans */etc/mail.rc*  
Veuillez noter que pour avoir l'utilitaire mail, vous devrez installer le paquet **mailutils**.  
Fichier : **/etc/mail.rc**

    set sendmail="/usr/bin/msmtp -t"

Vous pouvez tester que le courrier fonctionne avec la commande suivante :

    echo "Ceci est un corps de message" | mail -s "Hello there from mail command line" desti@destinataire.tld

**Crontab** offre une façon  d'envoyer des courriels. Au début du fichier crontab, vous pouvez l'éditer avec `sudo crontab -e` et déclarer la variable **MAILTO** :

```
MAILTO="xand@xand.es"
* * * * * /tmp/aaa.sh
```

### Sauvegarde via Borg

![](borg-logo.png)

On utilise BorgBackup  
Le script **/root/.borg/borg-backup** est lancé chaque jour à 3h00 ,compte rendu dans le fichier **/var/log/borg-backup.log**  

### Localisation IP

<u>Adresse IPv4</u>  
Elle est constituée d’une série de 4 nombres compris entre 0 et 255 séparés par des points “.“.  
Elle est limitée par le nombre d’appareils connectés en simultanée (4 294 967 296) sur internet.  
Exemple d’adresse IPv4 : 88.232.28.174

<u>Adresse IPv6</u>  
Elle est constitué de 8 blocs hexadécimaux de 8 octets séparés par des deux points “:“.  
L’intérêt de cette norme est qu’elle permet d’avoir beaucoup plus d’adresses IP disponibles (340 282 366 920 938 463 463 374 607 431 768 211 456 combinaisons possibles). Cette augmentation du nombre d’adresses est devenue nécessaire avec le développement d’internet et le nombre de périphériques connectés (ordinateurs, smartphones, tablettes, objets connectés).  
Exemple d’adresse IPv6 : 3bec:c504:3127: 5800:10c3:30bf:cb2f:1eee

*Pour plus d’informations sur les adresses IP, consultez l’article complet dédié au sujet sur [Wikipedia : https://fr.wikipedia.org/wiki/Adresse_IP](https://fr.wikipedia.org/wiki/Adresse_IP)*

Se connecter sur le serveur, passer en mode super utilisateur et se rendre dans la racine  

    cd /var/www/default-www

Cloner le dépôt dans la "racine" du site web **/var/www/default-www/**   

    git clone https://gitea.yanspm.com/yannick/localisation-IP-V4-V6.git

Le lien <https://yanspm.com/localisation-IP-V4-V6/localisation-serveur-vpn-ipv4-ipv6/>


### Calibre-web

* [Calibre-web](/posts/calibre-web/)

## Système

### Liste des tâches (crontab)

Tâches exécutées automatiquement  

    sudo crontab -e

```
# Mise à jour des certificats
9 0 * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
# Mise à jour automatique des serveurs DNS de la racine
50 02 * * * /etc/unbound/dnsunbound-update-root-dns.sh > /dev/null
# Générer une liste des domaines à bloquer
40 02 * * * /etc/unbound/unbound.conf.d/liste-domaines-bloques.sh > /dev/null
# Sauvegarde sur distant avec BorgBackup
00 03 * * * /root/.borg/borg-backup > /dev/null
```


## Annuaire LDAP

* [OpenLDAP installation et configuration annuaire](/posts/OpenLDAP-installation-configuration-annuaire/) 

### Installation , configuration et tests openLDAP

**Installation et configuration**

    DEBIAN_FRONTEND=noninteractive apt-get -yq install slapd ldap-utils
    dpkg-reconfigure slapd

```
Voulez-vous omettre la configuration d’OpenLDAP ? Non
Nom de domaine : ldap.yanspm.com
Nom d’entité (« organization ») : ldap.yanspm.com
Mot de passe de l’administrateur : *******
Module de base de données à utiliser : MDB
Faut-il supprimer la base de données lors de la purge du paquet ? Non
Faut-il déplacer l’ancienne base de données ? Oui
```

**Tests**

    systemctl enable slapd  # activation
    systemctl start slapd   # exécution

Ecoute

    netstat -laputn | grep slapd

```
tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      28128/slapd         
tcp6       0      0 :::389                  :::*                    LISTEN      28128/slapd         
```

Avec quels arguments s'exécute le service.   

    cat /var/run/slapd/slapd.args

```
/usr/sbin/slapd -h ldap:/// ldapi:/// -g openldap -u openldap -F /etc/ldap/slapd.d 
```

**-h** &rarr; les modalités d'accès à slapd : ldap et ldapi (ldap via TCP 389 et ldapi via socket unix)  
**-g** et **-u** &rarr; identité groupe et utilisateur sur lequel "tourne" le processus slapd  
**-F** &rarr; emplacement du ficher de configuration de slapd

Afficher les données de l'annuaire

    ldapsearch -x -H ldap://ldap.yanspm.com -b 'dc=ldap,dc=yanspm,dc=com'

```
# extended LDIF
#
# LDAPv3
# base <dc=ldap,dc=yanspm,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# ldap.yanspm.com
dn: dc=ldap,dc=yanspm,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
o: ldap.yanspm.com
dc: ldap

# admin, ldap.yanspm.com
dn: cn=admin,dc=ldap,dc=yanspm,dc=com
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
```

### Peuplement de l'annuaire

Peupler l'annuaire en créant les deux **OU** puis un compte système avec son groupe primaire.  
Créer un fichier au format LDIF (LDAP Data Interchange Format)

```
cat > ou.ldif << EOF
dn: ou=posixaccounts,dc=ldap,dc=yanspm,dc=com
objectclass: OrganizationalUnit

dn: ou=posixgroups,dc=ldap,dc=yanspm,dc=com
objectclass: OrganizationalUnit

EOF
```

Ajouter à l'annuaire (mot de passe admin)

    ldapadd  -x -H ldap://ldap.yanspm.com -D 'cn=admin,dc=ldap,dc=yanspm,dc=com' -f ou.ldif -W

Ajouter le compte (utilisateur mot de passe)  avec le groupe primaire  
Pour créer l'empreinte du mot de passe ,utiliser `slappasswd`  


```
cat > userandupg.ldif << EOF
dn: uid=yannick, ou=posixaccounts,dc=ldap,dc=yanspm,dc=com
uid: yannick
sn: yannick
homeDirectory: /home/yannick
cn: yannick
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
loginShell: /bin/bash
uidNumber: 6001
gidNumber: 6001
gecos: yan
mail: yannick@cinay.xyz
userPassword: {SSHA}NGdnkYoutO+bSUVDV1X+H6WvBBi4Cv17

dn: cn=yannick,ou=posixgroups,dc=ldap,dc=yanspm,dc=com
objectClass: posixGroup
objectClass: top
gidNumber: 6001
cn: yannick

EOF
```

Ajout des 2 objets

    ldapadd  -x -H ldap://ldap.yanspm.com -D 'cn=admin,dc=ldap,dc=yanspm,dc=com' -f userandupg.ldif -W

Supprimer les .ldif

	rm *.ldif

### Sécurisation SSL

SSL, couche additionnelle à TCP qui chiffre les échanges.  
On va activer ldaps et modifier les paramètres de lancement du service et générer un certificat.  

**Création des certificats**  
Création du répertoire ssl et génération certificat autosigné (validité 10ans)

    mkdir -p /etc/ldap/ssl && cd /etc/ldap/ssl
    openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 3650 -nodes

Le champ **Common Name** est le plus important

    Common Name (e.g. server FQDN or YOUR name) []:ldap.yanspm.com

Changer les permissions sur les 2 fichiers (utilisateur *openldap*) , clé privée accessible par root uniquement

    chown openldap:openldap /etc/ldap/ssl/key.pem
    chown openldap:openldap /etc/ldap/ssl/cert.pem
    chmod 400 /etc/ldap/ssl/key.pem

**Configuration de slapd**  
Ajout de 2 directives pour trouver le couple clé/certificat  

```
cat > cert.ldif << EOF
dn: cn=config
changetype: modify
replace: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ldap/ssl/cert.pem
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ldap/ssl/cert.pem
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ldap/ssl/key.pem

EOF
```

Pour appliquer les modifications ,nous allons passer par le mode ldapi car il n'y a pas de compte administrateur associé à la racine cn=config

    ldapmodify -Y EXTERNAL -H ldapi:/// -f cert.ldif  # pas de mot de passe

```
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "cn=config"
```

modifier le fichier **/etc/default/slapd** pour lui indiquer qu'il doit gérer  la méthode d'accès ldaps

    nano /etc/default/slapd

```
SLAPD_SERVICES="ldap:/// ldapi:/// ldaps:///"
```

Relancer slapd et vérifier

    systemctl restart slapd && netstat -laputn | grep slapd

```
tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      28651/slapd         
tcp        0      0 0.0.0.0:636             0.0.0.0:*               LISTEN      28651/slapd         
tcp6       0      0 :::389                  :::*                    LISTEN      28651/slapd         
tcp6       0      0 :::636                  :::*                    LISTEN      28651/slapd    
```

Le service slapd est en écoute SSL sur le port 636

### OpenLDAP PHP sur serveur

Installer le module

    sudo apt install php7.3-ldap

Fichiers de test **/var/www/default-www/test.php** 

    sudo nano /var/www/default-www/test.php

```
<?php
$baseDN = "dc=ldap,dc=yanspm,dc=com";
$ldapServer = "ldap.yanspm.com";
$ldapServerPort = 389;
$dn = 'cn=admin,dc=ldap,dc=yanspm,dc=com';

echo "Connexion au serveur <br />";
$conn=ldap_connect($ldapServer);

// on teste : le serveur LDAP est-il trouvé ?
if ($conn) {
 echo "Le résultat de connexion est ".$conn ."<br />";
} else {
 die("connexion impossible au serveur LDAP");
}

// 2ème étape : on effectue une liaison au serveur, ici de type "anonyme"
// pour une recherche permise par un accès en lecture seule

// On dit qu'on utilise LDAP V3, sinon la V2 par défaut est utilisé
// et le bind ne passe pas. 
if (ldap_set_option($conn, LDAP_OPT_PROTOCOL_VERSION, 3)) {
    echo "Utilisation de LDAPv3 <br />";
 } else {
    echo "Impossible d'utiliser LDAP V3<br />";
    exit;
 }

// Instruction de liaison. 
// Connexion anonyme
print ("Connexion anonyme...<br />");
$bindServerLDAP=ldap_bind($conn);

echo "Liaison au serveur : ". ldap_error($conn)."<br />";
// en cas de succès de la liaison, renvoie Vrai
if ($bindServerLDAP)
  echo "Le résultat de connexion est $bindServerLDAP <br />";
else
  die("Liaison impossible au serveur ldap ...");

/* 3ème étape : on effectue une recherche anonyme, avec le dn de base,
 * par exemple, sur tous les noms commençant par B
 */
echo "Recherche suivant le filtre (sn=y*) <br />";
$query = "sn=y*";
$result=ldap_search($conn, $baseDN, $query);
echo "Le résultat de la recherche est $result <br />";

echo "Le nombre d'entrées retournées est ".ldap_count_entries($conn,$result)."<p />";
echo "Lecture de ces entrées ....<p />";
$info = ldap_get_entries($conn, $result);
echo "Données pour ".$info["count"]." entrées:<p />";

for ($i=0; $i < $info["count"]; $i++) {
        echo "dn est : ". $info[$i]["cn"] ."<br />";
        echo "premiere entree cn : ". $info[$i]["cn"][0] ."<br />";
        echo "premier email : ". $info[$i]["mail"][0] ."<p />";
}

// Test utilisateur mot de passe
if($bindServerLDAP = @ldap_bind($conn, "uid=yannick,ou=posixaccounts,dc=ldap,dc=yanspm,dc=com", "Jkhgtrdef45")) {
	  echo "OK Test utilisateur mot de passe <br />";
	} else {
	  echo "ERROR Test utilisateur mot de passe <br />";
	}

echo "Fermeture de la connexion";
ldap_close($conn);
?>
```

Attribut fichier

    sudo chown $USER:www-data /var/www/default-www/test.php
    chmod +x /var/www/default-www/test.php

Test sur le lien https://yanspm.com/test.php

```
Connexion au serveur
Le résultat de connexion est Resource id #2
Utilisation de LDAPv3
Connexion anonyme...
Liaison au serveur : Success
Le résultat de connexion est 1
Recherche suivant le filtre (sn=y*)
Le résultat de la recherche est Resource id #3
Le nombre d'entrées retournées est 1

Lecture de ces entrées ....

Données pour 1 entrées:

dn est : Array
premiere entree cn : yannick
premier email : yannick@cinay.xyz

OK Test utilisateur mot de passe
Fermeture de la connexion 
```

Une fois la vérification effectuée , supprimer le fichier **test.php**

    sudo rm /var/www/default-www/test.php

## Authentification JWT + LDAP

* Gestion des pages web sur yanspm.com avec  autorisation (utilisateur et mot de passe) + jeton (token jwt)  
* Toutes les pages du site sont en php et ont OBLIGATOIREMENT un début de page avec ce qui suit :

```php
<?php
session_start(); // On démarre la session AVANT toute chose

include_once "veriftoken.php";
// si token non valide , retour page login
 if (!veriftoken()) {
 	// accés non autorisé
  	header('Location: login.php');
  } 
?>
```

* En cas d'expiration du token ou mauvaise identification , retour sur la page **login.php**  

### priv.yanspm.com

Création dossier private

    sudo mkdir /var/www/private
    sudo chown $USER:$USER /var/www/private

Créer un fichier index.php

    echo "<?php echo 'Dossier /var/www/private'; ?>" > /var/www/private/index.php

Créer un fichier de configuration nginx domaine priv.yanspm.com

    sudo nano /etc/nginx/conf.d/priv.yanspm.com.conf

```
server {
    listen 80;
    listen [::]:80;

    ## redirect http to https ##
    server_name priv.yanspm.com;
    return  301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name priv.yanspm.com;

    include ssl_params;
    include header_params;
#    include dh_param;

    root /var/www/private/ ;
    index index.php index/ index.htm;
        location ~ \.php$ {
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           fastcgi_pass unix:/run/php/php7.3-fpm.sock;   # PHP7.3
           fastcgi_index index.php;
           include fastcgi_params;
	   fastcgi_param SCRIPT_FILENAME $request_filename;
        }

    location /stub_status {
        stub_status;
        # Security: Only allow access from the IP below.
        allow 127.0.0.1;
        # Deny anyone else
        deny all;
    }


    access_log /var/log/nginx/priv.yanspm.com-access.log;
    error_log /var/log/nginx/priv.yanspm.com-error.log;
}
```

Vérifier et relancer nginx

    sudo nginx -t
    sudo systemctl restart nginx

Accès lien https://priv.yanspm.com

### Autorisation ldap + jwt (les fichiers php)

Liste des fichiers

    /var/www/private/

```
authclient.php
easter_spring_eggs_flowers_pastel_1920x1080.jpg
generkey.php
index.php      
jwtclass.php
login.php
portal.php
test.php
veriftoken.php
yannick-white-black.png
```

* [Using the NGINX Auth Request Module](https://redbyte.eu/en/blog/using-the-nginx-auth-request-module/)

### Créer un serveur d'autorisation local - autorise.srv  

Ajouter au fichier **/etc/hosts**  &rarr; `127.0.1.1 autorise.srv`  
Le fichier de configuration nginx  

    sudo nano /etc/nginx/conf.d/autorise.srv.conf

```
server {
    listen 8888;
    listen [::]:8888;

    server_name autorise.srv;

    root /var/www/private/ ;
    
    index authclient.php;
        location ~ \.php$ {
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           fastcgi_pass unix:/run/php/php7.3-fpm.sock;   # PHP7.3
           fastcgi_index index.php;
           include fastcgi_params;
	   fastcgi_param SCRIPT_FILENAME $request_filename;
        }
    access_log /var/log/nginx/autorise.srv-access.log;
    error_log /var/log/nginx/autorise.srv-error.log;
}
```

Vérifier

    sudo nginx -t
    sudo systemctl restart nginx


## Application Single Sign-On pour PHP (compatible Ajax)

* https://github.com/jasny/sso.git

Jasny\SSO est une solution relativement simple et directe pour une implémentation SSO (Single Sign on). Avec SSO, la connexion à un site Web unique vous authentifiera pour tous les sites affiliés.

### Comment cela fonctionne

Lors de l'utilisation de SSO, quand peut distinguer 3 parties :

*    Client - C'est le navigateur du visiteur
*    Courtier - Le site Web qui est visité
*    Serveur - L'endroit où se trouvent les informations et les identifiants de l'utilisateur

Le courtier a une identité et un secret. Ceux-ci sont connus à la fois du courtier et du serveur.

Lorsque le client visite le courtier, il crée un jeton aléatoire, qui est stocké dans un cookie. Le courtier enverra ensuite le client au serveur, en lui transmettant l'identifiant et le jeton du courtier. Le serveur crée un hachage à l'aide du broker id, du broker secret et du token. Ce hachage est utilisé pour créer un lien vers la session de l'utilisateur. Lorsque le lien est créé, le serveur redirige le client vers le courtier.

Le courtier peut créer le même hachage de lien en utilisant le token (à partir du cookie), l'identifiant du courtier et le secret du courtier. Lors de l'exécution des requêtes, il passe qui a comme identifiant de session.

Le serveur remarquera que l'identifiant de session est un lien et utilisera la session liée. Ainsi, le courtier et le client utilisent la même session. Lorsqu'un autre courtier s'inscrit, il utilisera également la même session.

Pour une explication plus détaillée, veuillez lire [cet article](https://github.com/jasny/sso/wiki).

### Installer composer et librairies

Installer php composer  
Vous pouvez télécharger le script composer à partir du site Web getcomposer.org en exécutant la commande suivante. Il créera un fichier composer.phar dans le répertoire en cours.

    curl -sS https://getcomposer.org/installer | php

Copiez le fichier composer.phar dans le répertoire bin pour le rendre disponible n’importe où dans le système. Définissez également l’autorisation d’exécution sur le fichier. J’ai changé le nom de fichier de composer.phar en compositeur pour une utilisation facile.

    sudo mv composer.phar /usr/local/bin/composer
    chmod +x /usr/local/bin/composer


Installer cette librairie via composer

    composer require jasny/sso





