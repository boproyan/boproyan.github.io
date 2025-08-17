+++
title = 'cwwk KVM - Service de notification ntfy (Alpine Linux  vm-ntfy)'
date = 2025-03-13 00:00:00
categories = ['virtuel']
+++
*[Ntfy](https://ntfy.sh/), qui se prononce “notify”, est un service de notification ultra léger, permettant d'envoyer des messages vers un smartphone ou un ordinateur via de simples scripts, sans besoin de compte et totalement gratuitement !*

![](alpine-linux-logo.png){:width="300"} ![](ntfy-logo.png){:width="80"}

## Alpine Linux

*Création machine virtuelle Alpine  de type KVM avec 2 Go de RAM, 1 cœur de processeur et 5 Go de disque dur.*

* [Création machines virtuelles KVM avec la commande virsh](/posts/Installer_KVM_Kernel_Virtual_Machine_sur_un_serveur/#création-machines-virtuelles-kvm-avec-la-commande-virsh)
* [Accéder aux machines virtuelles KVM via le client VNC](/posts/Installer_KVM_Kernel_Virtual_Machine_sur_un_serveur/#accéder-aux-machines-virtuelles-kvm-via-le-client-vnc)
* [Wiki Alpine Linux](https://wiki.alpinelinux.org/wiki/)

### Créer vm-ntfy sur un serveur

[Les dernières images Alpine Linux](https://alpinelinux.org/downloads/)  

Création d'une image virtuelle **vm-ntfy** sous le serveur cwwk rnmkcy.eu  
On se connecte sur le serveur cwwk en SSH, puis on exécute la commande suivante pour créer  une machine virtuelle Alpine avec 2 Go de RAM, 1 cœur de processeur et 5 Go de disque dur

```shell
sudo virt-install \
--osinfo alpinelinux3.17 \
--name vm-ntfy \
--memory 1024 \
--vcpus 1 \
--cpu host \
--hvm \
--disk path=/srv/kvm/libvirt/images/vm-ntfy.qcow2,format=qcow2,size=5 \
--cdrom /home/yick/FreeUSB2To/iso/alpine-standard-3.21.0-x86_64.iso \
--network bridge=br0 \
--graphics vnc  
```

Note: La dernière version Alpine Linux est alpinelinux3.20  au 10/07/2024 mais KVM ne connait que alpinelinux3.17 (`sudo virt-install --osinfo list |grep alpine`)

Après exécution dans un terminal de la commande ci dessus, on arrive sur l'écran suivant  
![](alpine-linux01.png)  

### Configurer vm-ntfy

Le serveur cwwk n'a pas d'affichage, il faut créer un tunnel ssh depuis un poste client

    ssh -L 5900:127.0.0.1:5900 yick@192.168.0.205 -p 55205 -i /home/yann/.ssh/yick-ed25519

Puis lancer de ce même poste un client VNC  
![](alpine-linux02.png){:width="300"}  
la console s'affiche   
![](alpine-linux03.png)  

Une fois l'image ISO lancée, on arrive à un invite de connexion.   
Indiquez `root` comme nom d'utilisateur, aucun mot de passe ne vous sera demandé à cette étape.   

Le système est utilisable, mais on veut l'installer, ce qui passe par la commande suivante (clavier qwerty)

```
setup-alpine # saisir setup)qlpine
```

Une suite de questions :  
![](alpine-linux04.png)  
mot de passe root (toorrtyuiop)  
utilisateur ntfy  (ntfy49)  
![](alpine-linux05.png)  
APK mirror (f) ,patienter ...  
![](alpine-linux06.png)  

Relever l'adresse ip allouée : `ip a` --> 192.168.10.200  
Puis redémarrer : `reboot`  
La fenêtre vnc se ferme  


### Explications sur la procédure

*Normalement, vous n'avez rien à faire, les paramètres par défaut doivent convenir. Mais si vous le désirez, vous pouvez les modifier pour utiliser une interface particulière, une IP fixe, un serveur proxy, etc.  
Une soixantaine de serveurs mirroir vous seront proposés pour télécharger les paquets. Choisissez un numéro dans la liste ou demandez au système de les tester et de sélectionner le plus rapide. Vous pouvez aussi modifier le fichier des sources. Il vous faudra ensuite choisir votre serveur SSH : OpenSSH, Dropbear ou aucun.* 

On termine par la méthode d'installation. Il en existe quatre : 

*    none : le système et ses données sont placés en RAM et seront perdus après le redémarrage
*    sys : le système et ses données sont placés sur un HDD/SSD
*    data : le système est placé en RAM, les données sur un HDD/SSD
*    lvm : utilisation de Logical Volume Manager, les deux choix précédents seront proposés (lvmsys, lvmdata)

Si vous stockez le système en mémoire, il faudra trouver un moyen de sauvegarder la configuration. Vous pourrez le faire uniquement depuis un lecteur de disquettes (!) ou une clé USB. Une fois le système installé, vous pourrez l'utiliser directement s'il est placé en mémoire ou redémarrer si vous avez opté pour un stockage classique.

Il n'est pas conseillé d'utiliser directement le compte root pour les actions du quotidien.  
Si utilisateur non créé dans la procédure d'installation, le créer avec son propre espace dans /home/ 

    adduser aluser

Vous pouvez utiliser l'utilisateur pour vous connecter via SSH (impossible avec le compte root)  

### Connexion vm-ntfy via SSH

Sur un poste linux du réseau

    ssh ntfy@192.168.10.200

Une fois connecté ,vous pouvez accéder au "root" de manière classique avec la commande :

    su -

Mise à jour

```shell
apk update
apk upgrade 
# Vous pouvez fusionner les deux lignes avec 
apk -U upgrade
```

Editeur nano (Vous pouvez aussi opter pour vi qui est nativement présent sur le système)

    apk add nano

### Réseau - IP statique

[How to configure static IP address on Alpine Linux](https://www.cyberciti.biz/faq/how-to-configure-static-ip-address-on-alpine-linux/)

Le fichier de configuration `/etc/network/interfaces`

    /etc/network/interfaces

```shell
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
        address 192.168.100.50/24
        gateway 192.168.100.1
```

Fichier de résolution dns

    /etc/resolv.conf

```
nameserver 1.1.1.1
nameserver 9.9.9.9
```

Les modifications apportées à /etc/network/interfaces peuvent être activées en exécutant

```shell
service networking restart 
```

Ou en redémarrant la machine

    reboot

### OpenSSH avec clés

*Connexion ssh sur un autre port avec un jeu de clés*

Générer une paire de clé sur l'ordinateur de bureau PC1  
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec la machine virtuelle vm-ntfy

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/vm-ntfy
    chmod 600 ~/.ssh/vm-ntfy

Copier la clé publique `cat ~/.ssh/vm-ntfy.pub` dans le presse-papier

	ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOOHNS4Lm43VFOIhT/XBFvF0LdoMxmH7SjJK7QAT1h1u yann@PC1

On se connecte sur la machine virtuelle alpine linux "vm-ntfy" en proxy avec le serveur cwwk (192.168.0.205)

```bash
ssh -o ProxyCommand="ssh -W %h:%p -p 55205 -i /home/yann/.ssh/yick-ed25519 yick@192.168.0.205" ntfy@192.168.100.50 -p 55069 -i /home/yann/.ssh/vm-ntfy
```

Créer le répertoire et ouvrir nouveau fichier

    mkdir -p $HOME/.ssh/
    nano $HOME/.ssh/authorized_keys

Coller le contenu du presse-papier , sauver le fichier et sortir

Modifier les droits

    chmod 600 $HOME/.ssh/authorized_keys

Passer en mode su

    su -

Modifier la configuration serveur SSH

    nano /etc/ssh/sshd_config

Modifier

```
Port = 55069
PasswordAuthentication no
```

Relancer le serveur

    service sshd restart

Test connexion

```bash
ssh -o ProxyCommand="ssh -W %h:%p -p 55205 -i /home/yann/.ssh/yick-ed25519 yick@192.168.0.205" ntfy@192.168.100.50 -p 55069 -i /home/yann/.ssh/vm-ntfy
```

### sudo

Passer en root

    su -

Editer la configuration des dépôts

    nano /etc/apk/repositories

Trouvez maintenant la ligne qui se termine dans /community Ensuite, retirez le # au début de la ligne. Le fichier résultant devrait ressembler à ceci

```
#/media/cdrom/apks
http://mirrors.ircam.fr/pub/alpine/v3.20/main
http://mirrors.ircam.fr/pub/alpine/v3.20/community
```

Mise à jour des dépôts

    apk update

Installer sudo

    apk add sudo

Pas de mot de passe sudo pour l'utilisateur alouest

    echo "ntfy     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

### motd

Message acceuil connexion ssh /etc/motd

```
    _    _        _              _     _                     
   /_\  | | _ __ (_) _ _   ___  | |   (_) _ _  _  _ __ __    
  / _ \ | || '_ \| || ' \ / -_) | |__ | || ' \| || |\ \ /    
 /_/ \_\|_|| .__/|_||_||_|\___| |____||_||_||_|\_,_|/_\_\    
           |_|                                               
  _  _       _    _   __  _            _    _                
 | \| | ___ | |_ (_) / _|(_) __  __ _ | |_ (_) ___  _ _   ___
 | .` |/ _ \|  _|| ||  _|| |/ _|/ _` ||  _|| |/ _ \| ' \ (_-<
 |_|\_|\___/ \__||_||_|  |_|\__|\__,_| \__||_|\___/|_||_|/__/
  _  ___  ___     _   __  ___     _   __    __     ___   __  
 / |/ _ \|_  )   / | / / ( _ )   / | /  \  /  \   | __| /  \ 
 | |\_, / / /  _ | |/ _ \/ _ \ _ | || () || () |_ |__ \| () |
 |_| /_/ /___|(_)|_|\___/\___/(_)|_| \__/  \__/(_)|___/ \__/ 
```

### Mises à jour automatique

Installer curl

    sudo apk add curl

Pour chaque nouveau serveur Alpine Linux, créer un script shell nommé apk-autoupgrade dans le dossier /etc/periodic/daily/ avecles permissions suivantes : 700

```shell
echo -e "#!/bin/sh\napk upgrade --update | sed \"s/^/[\`date\`] /\" >> /var/log/apk-autoupgrade.log" > /etc/periodic/daily/apk-autoupgrade && \
	chmod 700 /etc/periodic/daily/apk-autoupgrade
```

Si les tâches cron ne sont pas activées

```shell
rc-service crond start
rc-update add crond
```

Le script exécute la commande apk upgrade --update une fois par jour, apk par défaut ne demande jamais l’intervention de l’utilisateur. 

Additif pour notification ntfy avec notification email

```
echo "#
curl \
-H "X-Email: ntfy@cinay.eu" \
-H "Title: 💻 Alpine Linux serveur vm-ntfy : Fin exécution script 'apk-autoupgrade'" \
-H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
-H prio:low \
-d "Serveur de notification vm-ntfy 192.168.100.50 
✔️ Fin exécution script /etc/periodic/daily/apk-autoupgrade" \
https://noti.rnmkcy.eu/yan_infos
" >> /etc/periodic/daily/apk-autoupgrade
```

## Relayer les e-mails vers un compte (MSMTP, MailX, Sendmail)

*Si vous exécutez un programme alpine et que vous avez besoin d'un moyen pour que votre programme vous alerte via un compte de messagerie standard*  

Passer en mode su (`sudo -s`)  

### Modification hostname

Utiliser la commande echo ou printf pour écraser le fichier

```shell
echo "noti" > /etc/hostname
```

Activez immédiatement le changement en exécutant la commande suivante. En d'autres termes, utilisez le fichier /etc/hostname comme nom d'hôte

```shell
hostname -F /etc/hostname
```

Assurez-vous de mettre à jour le fichier `/etc/hosts` avec une configuration IP static

```
127.0.0.1	noti localhost localhost.localdomain localhost
::1		localhost localhost.localdomain
```

Vérification: `hostname -f`  
**noti**

### msmtp

Installation

```shell
apk add msmtp
```


Configuration

Créer un fichier de configuration global `/etc/msmtprc`

```
# Définir les valeurs par défaut pour les comptes 
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        /var/log/msmtp.log

# Gmail
account        cinay
host           mx1.xoyize.xyz
port           587
from           yack@cinay.eu
user           yack@cinay.eu
password       <your password>

# Set a default account
account default : cinay
aliases        /etc/aliases

```

### alias Sendmail

Par défaut alpine vient avec busebox sendmail, msmtp peut agir comme une alternative sendmail, y compris la syntaxe et l'option, là je crée un script local.d pour écraser le lien busebox vers msmtp.

Contenu de `/etc/local.d/msmtp-sendmail.start`

```shell
#!/bin/sh
ln -sf /usr/bin/msmtp /usr/bin/sendmail
ln -sf /usr/bin/msmtp /usr/sbin/sendmail
```

Rendre exécutable

```shell
chmod +x /etc/local.d/msmtp-sendmail.start
```

et l'exécuter la première fois

    /etc/local.d/msmtp-sendmail.start

### Mailx et alias

Installez mailx pour le programme qui utilise le courrier

```shell
apk ajouter mailx
```

Créer un fichier `/etc/aliases` 

Contenu de `/etc/aliases`

```
root: ntfy@cinay.eu
default: ntfy@cinay.eu
```

### Test envoi message 

    echo "Test envoi via msmtp" | mail -s "Alpine Linux ntfy" vpn@cinay.eu

## Ntfy

*Alpine Linux, processus d'installation de ntfy, un outil qui envoie des notifications de bureau quand une commande à long terme finit d'exécuter*

### Installer ntfy

Outils

    sudo apk add curl net-tools

installer ntfy 

    sudo apk add ntfy

Activer et lancer le service ntfy

```bash
# syntaxe commande
sudo rc-update add ntfy
# OU
sudo rc-update add ntfy default
```

Renvoie le résultat suivant

`* service ntfy added to runlevel default`

Démarrer le service ntfy

```bash
sudo rc-service ntfy start
# OU
sudo service ntfy start
```

Renvoie le résultat suivant

```
 * Caching service dependencies ...                                                    [ ok ]
 * /var/log/ntfy.log: creating file
 * /var/log/ntfy.log: correcting owner
 * /var/lib/ntfy: creating directory
 * /var/lib/ntfy: correcting owner
 * Starting ntfy ...                                                                   [ ok ]
```

Port en écoute : 8080


### Créer une instance ntfy privée

Voir doc [Instance privée](/posts/Ntfy/#instance-privée)

#### Configurer server.yml

Le fichier de configuration `/etc/ntfy/server.yml`

```
listen-http: ":8080"

auth-file: /var/lib/ntfy/user.db
auth-default-access: "deny-all"
behind-proxy: true
```

* **Listen-http** - Le port sur lequel le serveur Web HTTP écoute (par défaut = 80). Vous pouvez également ajouter une adresse IP à laquelle vous connecter (listen-http : "127.0.0.1:8080")
* **auth-file** - Emplacement du user.dbfichier d’authentification.
* **auth-default-access** - Définir ceci sur “deny-all” forcera tous les sujets à être privés par défaut.
* **behind-proxy** - Si ntfy est déployé derrière un proxy comme Caddy, nginx, etc., alors cela doit être défini sur true.

Redémarrer le serveur ntfy

    sudo service ntfy restart 

#### Créer utilisateur 

Passer en mode su : `sudo -s`

**Créer utilisateur dans NTFY** ([Users and roles](https://docs.ntfy.sh/config/#users-and-roles))  
Puisque nous avons déjà défini l' attribut `auth-file` et `auth-default-access`  
Ajout de notre premier utilisateur **administrateur** "notif".

    ntfy user add --role=admin notif

Second **utilisateur** "notifmob"

    ntfy user add notifmob

Une fois l'utilisateur créé, nous pouvons l'abonner à notre premier sujet de notification.

    ntfy access notifmob yan_infos rw

```
granted read-write access to topic yan_infos

user notifmob (role: user, tier: none)
- read-write access to topic yan_infos
```

#### Jetons (Tokens)

*Le jeton peut être utilisé à la place du nom d'utilisateur et du mot de passe lors de l'envoi de messages. ([Access tokens](https://docs.ntfy.sh/config/#access-tokens))*

    ntfy token add notif

Une fois le jeton d'accès créé, vous pouvez l'utiliser pour vous authentifier auprès du serveur ntfy, par exemple lorsque vous publiez ou vous abonnez à des sujets.

**ntfy curl + token**

```shell
curl \
    -H "Title: Information!" \
    -H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
    -H prio:low \
    -H tags:info \
    -d "Notification utilisant un jeton d'accès" \
    https://noti.rnmkcy.eu/yan_infos
```

**ntfy CLI + token**

```
ntfy publish \
  --token tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2 \
  ntfy.example.com/mysecrets \
  "Look ma, with auth"
```

### Notifications par courriel 

Pour permettre la transmission de messages par e-mail, vous pouvez configurer un serveur SMTP pour les messages sortants. Une fois configuré, vous pouvez définir l'en-tête X-Email pour envoyer des messages par e-mail (par exemple curl -d "hi here" -H "X-Email: phil@example.com" ntfy.sh/mytopic).

À ce jour, seuls les serveurs SMTP avec PLAIN auth et STARTLS sont pris en charge. Pour activer l'envoi d'e-mails, vous devez définir les paramètres suivants :

* **base-url** est l'URL racine du serveur ntfy; cela est nécessaire pour le pied de page de messagerie
* **smtp-sender-addr** est le nom d'hôte:port du serveur SMTP
* **smtp-sender-user** et **smtp-sender-pass** sont le nom d'utilisateur et le mot de passe de l'utilisateur SMTP
* **smtp-sender-from** est l'adresse électronique de l'expéditeur

Ajouter ce qui suit au fichier de configuration `/etc/ntfy/serveur.yml` pour le courrier sortant

```
base-url: "https://noti.rnmkcy.eu"
smtp-sender-addr: "mx1.xoyize.xyz:587"
smtp-sender-user: "yack@cinay.eu"
smtp-sender-pass: "Abd13Kf+sfAk2Dzifjafldk"
smtp-sender-from: "yack@cinay.eu"
```

Redémarrer le service ntfy

    sudo service ntfy restart

Tester

```shell
curl \
    -H "X-Email: ntfy@cinay.eu" \
    -H "Title: Information!" \
    -H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
    -H prio:low \
    -H tags:info \
    -d "Notification utilisant un jeton d'accès" \
    https://noti.rnmkcy.eu/yan_infos
```


Message reçu  
![](ntfy-email.png){: .normal}
