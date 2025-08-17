+++
title = 'Contabo VPS debian 12 Web(caddy) + Messagerie(maddy) + Nebula'
date = 2025-03-13 00:00:00 +0100
categories = vps
+++
*__Caddy__ est un serveur web open source écrit en go avec mise en place de certificat SSL automatique.  
__Maddy Mail Server__ est un serveur de messagerie open source qui gère les domaines xoyize.xyz cinay.eu yanfi.net et ouestline.xyz*

[![Contabo](contabo-logo-a.png){:width="130" .left}](https://contabo.com/en/)  
**VPS** 1 NVMe  
4 vCPU Cores  
6 GB RAM  
100 GB NVMe  
1 Snapshot  
**Location**: Portsmouth ![](gb.png){:width="15" .normal}  
**IP**: 158.220.91.148  
**IPv6**: 2a02:c204:2231:478::1  
**OS**: Debian 12  

## Debian 12 (bookworm)

![](debian12-logo.png){:width="150" .normal}  

On se connecte en root sur le VPS

    ssh root@158.220.91.148

Le terminal

```
Linux vmi1628819.contaboserver.net 6.1.0-10-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.37-1 (2023-07-03) x86_64
  _____
 / ___/___  _  _ _____ _   ___  ___
| |   / _ \| \| |_   _/ \ | _ )/ _ \
| |__| (_) | .` | | |/ _ \| _ \ (_) |
 \____\___/|_|\_| |_/_/ \_|___/\___/

Welcome!

This server is hosted by Contabo. If you have any questions or need help,
please don't hesitate to contact us at support@contabo.com.
```

### Mise à jour + Réseau IPV6

Les commandes

```bash
apt update && apt upgrade -y  # Mise à jour
apt autoremove -y             # supprimer les paquets inutilisés
apt install rsync curl tmux jq figlet git tree -y   # Quelques outils
enable_ipv6                   # Activation ipv6
reboot                        # redémarrage
```

se reconnecter ssh  

    ssh root@158.220.91.148

Vérifier l'adressage : `ip a`

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:50:56:56:eb:8b brd ff:ff:ff:ff:ff:ff
    altname enp0s18
    altname ens18
    inet 158.220.91.148/20 brd 158.220.95.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 2a02:c204:2231:478::1/64 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::250:56ff:fe56:eb8b/64 scope link 
       valid_lft forever preferred_lft forever
```

### Date et heure + Synchro

Activer le fuseau Europe/Paris

    timedatectl set-timezone Europe/Paris

Horloge système synchronisée : `timedatectl`

```
               Local time: Sat 2024-11-30 08:03:51 CET
           Universal time: Sat 2024-11-30 07:03:51 UTC
                 RTC time: Sat 2024-11-30 07:03:51
                Time zone: Europe/Paris (CET, +0100)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

### Reconfigurer locales

Activer uniquement  **en_US.UTF-8** et **fr_FR.UTF-8**

    dpkg-reconfigure locales

Default en_US.UTF-8

```
Generating locales (this might take a while)...
  en_US.UTF-8... done
  fr_FR.UTF-8... done
Generation complete.
```

### Motd

Motd

    rm /etc/motd && nano /etc/motd

```bash
 __   __ ___  ___   ___        _     _               _  ___         
 \ \ / /| _ \/ __| |   \  ___ | |__ (_) __ _  _ _   / ||_  )        
  \ V / |  _/\__ \ | |) |/ -_)| '_ \| |/ _` || ' \  | | / /         
   \_/  |_|  |___/ |___/ \___||_.__/|_|\__,_||_||_| |_|/___|        
  ___            _                          _    _      _   _  _  __
 | _ \ ___  _ _ | |_  ___ _ __   ___  _  _ | |_ | |_   | | | || |/ /
 |  _// _ \| '_||  _|(_-<| '  \ / _ \| || ||  _|| ' \  | |_| || ' < 
 |_|  \___/|_|   \__|/__/|_|_|_|\___/ \_,_| \__||_||_|  \___/ |_|\_\
  _  ___  ___     ___  ___  __    ___  _     _  _ _   ___           
 / || __|( _ )   |_  )|_  )/  \  / _ \/ |   / || | | ( _ )          
 | ||__ \/ _ \ _  / /  / /| () |_\_, /| | _ | ||_  _|/ _ \          
 |_||___/\___/(_)/___|/___|\__/(_)/_/ |_|(_)|_|  |_| \___/          
```

### Hostname 

    hostnamectl set-hostname xoyize.xyz
    hostnamectl

```
 Static hostname: xoyize.xyz
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 3a11b621fd7febb8779d5928674ab56a
         Boot ID: 2a69bd7a60844a6db206b6b27183504d
  Virtualization: kvm
Operating System: Debian GNU/Linux 12 (bookworm)              
          Kernel: Linux 6.1.0-28-amd64
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.16.1-0-g3208b098f51a-prebuilt.qemu.org
```

Ajout ip au fichier hosts

    nano /etc/hosts

```
127.0.0.1       xoyize.xyz localhost 
```

Vérifier : `hostname -f` --> xoyize.xyz

### Reverse DNS

Activer le reverse DNS IPV4 et IPV6 pour le domaine de messagerie mx1.xoyize.xyz : Control panel → Reverse DNS management  
![](contabo-reverse-dnsa.png){: .normal}  
Désactiver VNC: Your services &rarr; Manage &rarr; VPS Control &rarr; Manage &rarr; Disable VNC et valider par un clic sur Disable   
`Un reboot de la machine est obligatoire pour la prise en compte`{: .prompt-info }

### Créer un utilisateur

Après s'être connecté en tant qu'utilisateur root, créer le premier utilisateur :  
`adduser xouser`

```bash
Ajout de l'utilisateur « xouser » ...
Ajout du nouveau groupe « xouser » (1000) ...
Ajout du nouvel utilisateur « xouser » (1000) avec le groupe « xouser » (1000) ...
Création du répertoire personnel « /home/xouser » ...
Copie des fichiers depuis « /etc/skel » ...
Nouveau mot de passe : 
Retapez le nouveau mot de passe : 
passwd : mot de passe mis à jour avec succès
Modifier les informations associées à un utilisateur pour xouser
Entrer la nouvelle valeur, ou appuyer sur ENTER pour la valeur par défaut
	NOM []: 
	Numéro de chambre []: 
	Téléphone professionnel []: 
	Téléphone personnel []: 
	Autre []: 
Cette information est-elle correcte ? [O/n]
Ajout du nouvel utilisateur « xouser » aux groupes supplémentaires « users » ...
Ajout de l'utilisateur « xouser » au groupe « users » ...
```

Ajout à sudoers

```bash
apt install sudo   # installer sudo
#
echo "xouser     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/xouser
```

Pour lire le journal système

    sudo usermod -a -G adm $USER

### OpenSSH, clé et script

![OpenSSH](ssh_logo1.png){:width="100" .left}  
**connexion avec clé**  
<u>sur l'ordinateur de bureau</u>
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec le serveur.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/xoyize-ed25519

Envoyer les clés publiques sur le serveur KVM   

    ssh-copy-id -i ~/.ssh/xoyize-ed25519.pub xouser@158.220.91.148

<u>sur le serveur KVM</u>
On se connecte  

    ssh xouser@158.220.91.148

Modifier la configuration serveur SSH dans le VPS

    sudo nano /etc/ssh/sshd_config 

Ajouter

```conf
Port = 55148
PasswordAuthentication no
PermitRootLogin no
ClientAliveInterval 60
```

Relancer le serveur

    sudo systemctl restart sshd

Test connexion

    ssh -p 55148 -i ~/.ssh/xoyize-ed25519 xouser@158.220.91.148

### Historique de la ligne de commande

Ajoutez la recherche d’historique de la ligne de commande au terminal Se connecter en utilisateur debian Tapez un début de commande précédent, puis utilisez shift + up (flèche haut) pour rechercher l’historique filtré avec le début de la commande.

```bash
# Global, tout utilisateur
echo '"\e[1;2A": history-search-backward' | sudo tee -a /etc/inputrc
echo '"\e[1;2B": history-search-forward' | sudo tee -a /etc/inputrc
```

### Parefeu

![Texte alternatif](ufw-logo-b.png){: width="60" .left}  
Installation Debian / Ubuntu

    sudo apt install ufw

Les règles

```bash
sudo ufw allow 55148/tcp  # port SSH
sudo ufw allow https      # port 443
# Ouvrir les ports 25,587 et 993 pour le serveur messagerie
sudo ufw allow 25
sudo ufw allow 587
sudo ufw allow 993
```

Activer le parefeu

    sudo ufw enable

Command may disrupt existing ssh connections. Proceed with operation (y|n)? y  
Firewall is active and enabled on system startup

Vérifier : `sudo ufw  status`

```
Status: active

To                         Action      From
--                         ------      ----
55139/tcp                  ALLOW       Anywhere                  
443                        ALLOW       Anywhere                  
25                         ALLOW       Anywhere                  
587                        ALLOW       Anywhere                  
993                        ALLOW       Anywhere                  
55139/tcp (v6)             ALLOW       Anywhere (v6)             
443 (v6)                   ALLOW       Anywhere (v6)             
25 (v6)                    ALLOW       Anywhere (v6)             
587 (v6)                   ALLOW       Anywhere (v6)             
993 (v6)                   ALLOW       Anywhere (v6)             
```

### DNS OVH xoyize.xyz

![](dns-logo-new.png){:width="50" .left}

Modifier les enregistrements DNS A et AAAA chez le fournisseur de domaine OVH  

**IP**: 158.220.91.148  
**IPv6**: 2a02:c204:2231:478::1  

Zone DNS domaine xoyize.xyz et MX mx1.xoyize.xyz

```
$TTL 3600
@	IN SOA dns106.ovh.net. tech.ovh.net. (2024113005 86400 3600 3600000 60)
        IN NS     ns106.ovh.net.
        IN NS     dns106.ovh.net.
        IN MX     10 mx1
        IN A     158.220.91.148
        IN AAAA     2a02:c204:2231:478::1
*       IN A     158.220.91.148
*       IN AAAA     2a02:c204:2231:478::1
mx1     IN A     158.220.91.148
mx1     IN AAAA     2a02:c204:2231:478::1
```

joker`(*)` pour tous les sous-domaines


## Caddy serveur

![](caddy-logo.png){:width="100" .left} *est un serveur web open source écrit en go proposant la mise en place de certificat SSL automatiquement*

* [Caddy serveur](/posts/Caddy_serveur/)
* [Caddy documentation](https://caddyserver.com/docs/)
* [Moving to the Caddy web server](https://www.ericlight.com/moving-to-the-caddy-web-server/)
* [Caddy markdown hugo git](https://olivier.dossmann.net/wiki/services/caddy/)

### Installer caddy

```bash
# Prérequis
sudo apt update -y
sudo apt install libcap2-bin libnss3-tools wget tar -y
# Versions : https://github.com/caddyserver/caddy/releases
wget https://github.com/caddyserver/caddy/releases/download/v2.9.0/caddy_2.9.0_linux_amd64.tar.gz
# Décompresser
tar xvf caddy_2.9.0_linux_amd64.tar.gz
# Déplacer le binaire 
sudo mv caddy /usr/local/bin/
# utilisateur restreint nommé caddy
sudo useradd --shell /bin/false --home-dir /etc/caddy --system caddy
# configuration dans le répertoire .config, et les certificats dans le répertoire .local + log
sudo mkdir -p /etc/caddy/.config /etc/caddy/.local /var/log/caddy
# Modifier la propriété des répertoires
sudo chown -R caddy: /etc/caddy /var/log/caddy
# Lorsque vous exécutez un logiciel en tant qu’utilisateur non root 
# Linux interdit à ces processus d’écouter des numéros de port inférieurs à 1024
# Contourner cette restriction et exécuter Caddy en toute sécurité en tant qu’utilisateur non root
sudo setcap cap_net_bind_service+ep /usr/local/bin/caddy
```

Afficher version : `caddy --version`  
**v2.9.0 h1:rteY8N18LsQn+2KVk6R10Vg/AlNsID1N/Ek9JLjm2yE=**

### Service caddy

Créer le service `caddy.service`

    sudo nano /etc/systemd/system/caddy.service


```
[Unit]
Description=Caddy web server
After=network-online.target

[Service]
User=caddy
Group=caddy
Type=exec
WorkingDirectory=/var/caddy/www

ExecStart=/usr/local/bin/caddy run --config /etc/caddy/Caddyfile
ExecReload=/usr/local/bin/caddy reload --config /etc/caddy/Caddyfile
ExecStop=/usr/local/bin/caddy stop

LimitNOFILE=1048576
LimitNPROC=512

PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=strict
ReadWritePaths=/etc/caddy/.local /etc/caddy/.config /var/log

CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

Noter , répertoire par défaut : `/var/caddy/www`{: .prompt-tip }  

Activer caddy

    sudo systemctl enable caddy 

## Maddy Mail Server mx1.xoyize.xyz

![](mailserver-logo.png){:width="100" .left}

*Maddy Mail Server met en œuvre toutes les fonctionnalités requises pour faire fonctionner un serveur de courrier électronique. Il peut envoyer des messages via SMTP (fonctionne comme MTA), accepter des messages via SMTP (fonctionne comme MX) et stocker des messages tout en y donnant accès via IMAP. En outre, il met en œuvre des protocoles auxiliaires qui sont obligatoires pour assurer une sécurité raisonnable du courrier électronique (DKIM, SPF, DMARC, DANE, MTA-STS).  
Il remplace Postfix, Dovecot, OpenDKIM, OpenSPF, OpenDMARC et d'autres encore par un seul démon avec une configuration uniforme et un coût de maintenance minimal.*

### Installation

*Mise en place un serveur de messagerie utilisant maddy*

Prérequis :

* Serveur Debian 12
    * Vérifier que le fournisseur d'accès internet ne bloque pas le trafic SMTP port TCP 25
        * Depuis un poste externe : `sudo nmap -sS -p25 158.220.91.148`
        
```
Starting Nmap 7.95 ( https://nmap.org ) at 2024-11-30 09:27 CET
Nmap scan report for xoyize.xyz (158.220.91.148)
Host is up (0.0082s latency).

PORT   STATE    SERVICE
25/tcp filtered smtp

Nmap done: 1 IP address (1 host up) scanned in 0.43 seconds
```

    * Vérifier le DNS Reverse IPV4 xoyize.xyz (activation chez le fournisseur de VPS)

```
dig -x 158.220.91.148 +short            # --> xoyize.xyz.
dig -x 2a02:c204:2231:478::1 +short     # --> xoyize.xyz.
```

* Domaine OVH : xoyize.xyz
    * Protection DNS contre le transfert
    * DNSSEC activé

Utilisation image préconstruite (Linux, amd64)  
Disponible sur [GitHub](https://github.com/foxcpp/maddy/releases) ou [maddy.email/builds](https://maddy.email/builds/).

Archive au format zst, installer l'outil si besoin (est installé par défaut sur debian 12): 

    sudo apt install zstd

Téléchargement et décompression

```bash
# Téléchargement
wget https://github.com/foxcpp/maddy/releases/download/v0.7.1/maddy-0.7.1-x86_64-linux-musl.tar.zst
# décompression
tar -I zstd -xvf maddy-0.7.1-x86_64-linux-musl.tar.zst
```

Contenu archive

```
./maddy-0.7.1-x86_64-linux-musl/
./maddy-0.7.1-x86_64-linux-musl/maddy
./maddy-0.7.1-x86_64-linux-musl/systemd/
./maddy-0.7.1-x86_64-linux-musl/systemd/maddy.service
./maddy-0.7.1-x86_64-linux-musl/systemd/maddy@.service
./maddy-0.7.1-x86_64-linux-musl/maddy.conf
```

Copier l'exécutable maddy que vous pouvez copier dans /usr/local/bin

    sudo cp maddy-0.7.1-x86_64-linux-musl/maddy /usr/local/bin/

### Systemd service + utilisateur maddy

*distribution basée sur systemd*

On a utilisé une archive préconstruite, copiez manuellement `systemd/*.service` dans `/etc/systemd/system`

    sudo cp maddy-0.7.1-x86_64-linux-musl/systemd/*.service /etc/systemd/system/

Vous devez recharger la configuration du gestionnaire de service pour rendre le service disponible 

    sudo systemctl daemon-reload

En outre, vous devez créer l'utilisateur et le groupe maddy (maddy ne s'exécute jamais en tant que root)

    sudo useradd -mrU -s /sbin/nologin -d /var/lib/maddy -c "maddy mail server" maddy

Créer le répertoire et copier le fichier de configuration

    sudo mkdir -p /etc/maddy
    sudo cp maddy-0.7.1-x86_64-linux-musl/maddy.conf /etc/maddy/


### Hôte mx1.xoyize.xyz + messagerie xoyize.xyz

* Si vous installez un très petit serveur de messagerie, vous pouvez utiliser xoyize.xyz dans les deux champs. 
* Cependant, pour faciliter une future migration de service, <u>il est recommandé d'utiliser une entrée DNS séparée</u> à cet effet. Il s'agit généralement de mx1.xoyize.xyz, mx2, etc. Vous pouvez bien sûr utiliser un autre sous-domaine, par exemple : smtp1.xoyize.xyz. Un serveur de basculement de courrier électronique sera possible si vous transférez mx2.xoyize.xyz vers un autre serveur (à condition que vous le configuriez pour gérer votre domaine).

Ouvrir `/etc/maddy/maddy.conf` 

```
$(hostname) = mx1.xoyize.xyz
$(primary_domain) = xoyize.xyz
$(local_domains) = $(primary_domain)
```

Si vous souhaitez gérer plusieurs domaines, vous devez toujours en désigner un comme "primaire".  
Ajoutez tous les autres domaines à la ligne `local_domains`  

    $(local_domains) = $(primary_domain) example.com other.example.com

### Certificat TLS mx1.xoyize.xyz

![](tls-logo.png){:width="80" .left} *Une chose qui ne peut pas être configurée automatiquement, ce sont les certificats TLS (/etc/maddy/maddy.conf)*.    

![](maddy-certificat01.png)

Créer le dossier de  stockage des certificats

```bash
sudo mkdir -p /etc/maddy/certs/mx1.xoyize.xyz
sudo chown $USER:root /etc/maddy/certs/mx1.xoyize.xyz # pour écriture dess certificats par acme
```

**ACME.sh**  

Création des certificats ([Serveur , installer et renouveler les certificats SSL Let's encrypt via Acme](/posts/Acme-Certficats-Serveurs/))

```
acme.sh --dns dns_ovh --server letsencrypt --issue --keylength ec-384 -d 'mx1.xoyize.xyz'
```

Si vous utilisez **acme.sh** pour gérer vos certificats, vous pouvez simplement exécuter

```
acme.sh --ecc --install-cert -d 'mx1.xoyize.xyz' \
  --key-file       /etc/maddy/certs/mx1.xoyize.xyz/privkey.pem  \
  --fullchain-file /etc/maddy/certs/mx1.xoyize.xyz/fullchain.pem
```

`Problème de lecture des certificats par le daemon maddy, malgré l'utilisation des ACL !!! + problème de renouvellement des certificats car aucun shell ne peut être exécuté !!!`{: .prompt-warning }

Vous devez vous assurer que maddy peut les lire lorsqu'il tourne en tant qu'utilisateur non privilégié (maddy ne tourne jamais en tant que root, même au démarrage), une façon de le faire est d'utiliser les ACLs 

```bash
sudo apt install acl # si non installé
sudo setfacl -R -m g:maddy:rwx /etc/maddy/certs/mx1.xoyize.xyz
```

maddy recharge les certificats TLS à partir du disque une fois par minute, de manière à ce qu'il remarque le renouvellement. Il est possible de forcer le rechargement via `systemctl reload maddy` 

Tâche pour le renouvellemnt des certificats `mx1.xoyize.xyz`

```
45 01 * * * "$HOME/.acme.sh"/acme.sh --cron --home "$HOME/.acme.sh" --renew-hook "$HOME/.acme.sh/acme.sh --ecc --install-cert -d 'mx1.xoyize.xyz' --key-file /etc/maddy/certs/mx1.xoyize.xyz/privkey.pem --fullchain-file /etc/maddy/certs/mx1.xoyize.xyz/fullchain.pem"
```

### Maddy - Lancement et activation 

Installer les outils net

    sudo apt-get install net-tools

On vérifie si un service est en écoute sur le port 25

```bash
# Ecoute sur le port 25 ?
sudo ss -ltnp | grep -w ':25'

# si un service écoute sur le port 25
# supprimer bind qui écoute sur le port 25 et redémarrer
sudo apt purge bind* -y
sudo systemctl reboot
```

lancer et activer maddy

    sudo systemctl enable maddy --now 

maddy initie le répertoire `/var/lib/maddy/` et génère la clé DKIM

    sudo systemctl status maddy

Les informations

```
● maddy.service - maddy mail server
     Loaded: loaded (/etc/systemd/system/maddy.service; enabled; preset: enabled)
    Drop-In: /run/systemd/system/service.d
             └─zzz-lxc-service.conf
     Active: active (running) since Wed 2024-11-20 15:26:48 CET; 1min 48s ago
       Docs: man:maddy(1)
             man:maddy.conf(5)
             https://maddy.email
   Main PID: 221 (maddy)
     Status: "Listening for incoming connections..."
      Tasks: 6 (limit: 525)
     Memory: 8.6M
        CPU: 334ms
     CGroup: /system.slice/maddy.service
             └─221 /usr/local/bin/maddy run

nov. 20 15:26:48 xoyize.xyz maddy[221]: table.file: ignoring non-existent file: /etc/maddy/aliases
nov. 20 15:26:48 xoyize.xyz maddy[221]: smtp: listening on tcp://0.0.0.0:25
nov. 20 15:26:48 xoyize.xyz maddy[221]: modify.dkim: generating a new rsa2048 keypair...
nov. 20 15:26:48 xoyize.xyz maddy[221]: modify.dkim: generated a new rsa2048 keypair, private key is in dkim_keys/xoyize.xyz_default.key, TXT record with public key is in dkim_keys/xoyize.xyz_default.dns,
nov. 20 15:26:48 xoyize.xyz maddy[221]: put its contents into TXT record for default._domainkey.xoyize.xyz to make signing and verification work
nov. 20 15:26:48 xoyize.xyz maddy[221]: submission: listening on tls://0.0.0.0:465
nov. 20 15:26:48 xoyize.xyz maddy[221]: submission: listening on tcp://0.0.0.0:587
nov. 20 15:26:48 xoyize.xyz maddy[221]: imap: listening on tls://0.0.0.0:993
nov. 20 15:26:48 xoyize.xyz maddy[221]: imap: listening on tcp://0.0.0.0:143
nov. 20 15:26:48 xoyize.xyz systemd[1]: Started maddy.service - maddy mail server.
```

### DKIM, DMARC, MTA-STS et TLS(DANE)

#### DKIM

*L’enregistrement DKIM est un enregistrement TXT modifié qui ajoute des signatures cryptographiques à vos messages électroniques. Vous ajoutez un enregistrement DKIM à votre système de nom de domaine (DNS), et il contient la cryptographie à clé publique utilisée par le serveur de messagerie récepteur pour authentifier un message.* 

>**DKIM**, norme technique Domain Keys Identified Mail (DKIM) est une forme de processus d'authentification par courriel. Il permet à une organisation d'ajouter une signature numérique à ses messages électroniques, de sorte que les destinataires ont un moyen de valider l'email en jumelant sa clé cryptographique publique aux enregistrements DNS. Le processus de signature DKIM comporte trois étapes principales, à commencer par l'expéditeur qui identifie les champs à inclure dans leur signature d'enregistrement DKIM. La plateforme email de l'expéditeur crée alors un hachage des champs de texte, qui inclut la signature DKIM. Dans la dernière étape, la passerelle électronique validera la signature DKIM en jumelant la clé publique à la clé privée.
{: .prompt-tip }

Et la dernière, la clé DKIM, est un peu délicate. maddy a généré une clé pour vous lors du premier démarrage.  
Vous pouvez la trouver dans `/var/lib/maddy/dkim_keys/xoyize.xyz_default.dns`{: .prompt-info }  
Vous devez la mettre dans un enregistrement TXT pour le domaine default._domainkey.xoyize.xyz

```
default._domainkey IN TXT "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMII...hQIDAQAB"
```

#### DMARC

>**DMARC** est une norme technique qui aide à protéger contre l'hameçonnage, le spoofing et le spam. C'est un protocole d'authentification par courriel qui utilise SPF et DKIM pour déterminer l'authenticité du message par courriel. Avec une politique DMARC en place, vous pouvez vérifier si le SPF et le DKIM s'alignent sur le domaine d'adresse indiqué sur le "Header" et spécifier ce que le serveur récepteur doit faire en cas d'échec du SPF et du DKIM.
{: .prompt-tip }

Définir sa politique de sécurité DMARC !

La politique de sécurité DMARC consiste à dire aux serveurs distants (qui sont capables d’interpréter ces politiques) quoi faire en cas de non-conformité d’un email. À ce jour, il n’existe que TROIS valeurs possibles qui s’appliqueront au domaine où DMARC sera implémenté (p=) ou à tous ses sous-domaines (sp=) :

*    La policy none (ou p=none) : Aucun traitement de l’email non-conforme ne sera fait, l’email pourra être délivré en boite de réception ou en spam.
*    La policy quarantine (ou p=quarantine) : L’email non-conforme sera délivré en courrier indésirable.
*    La policy reject (p=reject) : L’email non-conforme ne sera pas délivré et un bounce sera envoyé à l’expéditeur.

Toute autre déclaration de policy amènera chez les FAI / Webmails des comportements différents :

*    Gmail : Transforme la valeur existante par none (dmarc=pass p=NONE sp=NONE) ;
*    Microsoft : Produit comme résultat une erreur permanente (dmarc=permerror) ;
*    Yahoo : Produit un résultat inconnu (dmarc=unknown) ;
*    La Poste : Ignore l’enregistrement (dmarc=none reason= »No policy found »).

Ces politiques ne sont pas enregistrées sous forme de phrases lisibles par l'homme, mais plutôt sous forme de commandes lisibles par la machine afin que les services de messagerie puissent les interpréter automatiquement. Cette politique DMARC ressemblerait en fait à ceci :

politique de sécurité valide pour un enregistrement DMARC  
`v=DMARC1; p=quarantine; rua=mailto:postmaster@xoyize.xyz;`

**Politique DMARC xoyize.xyz**  
![](dmarc_xoyize.xyz.png)

```
_dmarc        IN TXT     "v=DMARC1; p=quarantine; rua=mailto:postmaster@xoyize.xyz"
```

#### MTA-STS 

>**MTA-STS**, Google vise à rendre Gmail plus sécurisé avec l'agent de transfert de courrier/Strict Transport Security (MTA-STS). Le mécanisme charge le serveur SMTP de s'assurer que l'autre serveur SMTP doit être chiffré et que le nom de domaine figurant sur le certificat doit correspondre au domaine. Lorsque MTA-STS a été activé pour votre domaine, vous demandez que les serveurs de messagerie externes n'envoient des messages à votre domaine que lorsque la connexion SMTP est à la fois cryptée avec TLS 1.2 ou supérieur et authentifiée avec un certificat public valide. MTA-STS protège contre les attaques Man-in-the-Middle (MITM) et les attaques de dégradation et résout les problèmes de sécurité SMTP tels que les certificats TLS expirés.
{: .prompt-tip }

**Prérequis, caddy ou nginx installé**

*Le MTA-STS, comme son nom l'indique, est un protocole qui permet le transport crypté de messages entre deux serveurs de messagerie SMTP. MTA-STS spécifie aux serveurs d'envoi que les courriers électroniques ne doivent être envoyés que par une connexion cryptée TLS, et ne doivent pas être délivrés du tout dans le cas où une connexion sécurisée n'est pas établie via la commande STARTTLS. En renforçant la sécurité des courriers électroniques en transit, MTA-STS contribue à atténuer les attaques de type "Man-In-The-Middle" (MITM) telles que les attaques de déclassement SMTP et les attaques de spoofing DNS.*  
![](mta_sts.png){:width="350" .left}

* [Qu'est-ce que MTA-STS ?](https://powerdmarc.com/fr/mta-sts-implementation-guide/)
* [Enable MTA-STS in 5 Minutes with NGINX](https://www.naut.ca/blog/2020/04/07/mta-sts-in-5-minutes/)

![](mta_sts_doc.png)

Créer le dossier 

    sudo mkdir -p /var/caddy/www/.well-known

**Enregistrement MTA-STS DNS domaine xoyize.xyz**  
![](mta_sts_doc2.png)

```
; Mark domain as MTA-STS compatible 
_mta-sts.xoyize.xyz.   TXT    "v=STSv1; id=8E5bpv87mdL4Jd3AL65Czp4DHbF7W6ky;"
; and request reports about failures to be sent to postmaster@xoyize.xyz
_smtp._tls.xoyize.xyz. TXT    "v=TLSRPTv1;rua=mailto:postmaster@xoyize.xyz"
```

**Stratégie pour serveur de messagerie xoyize.xyz**  

* **version**: La version du protocole du fichier. Au moment de la rédaction de ce document, il doit s'agir de STSv1.
* **mode**: Il s'agit du mode de la politique. Les valeurs disponibles sont **testing**, **enforce**, ou **none**.
    * **testing**: Les expéditeurs enverront vos rapports (TLSRPT) indiquant les échecs d'application de la politique. Cela nécessite que TLSRPT soit également implémenté pour fonctionner. Les échecs de connexion TLS ne seront pas bloqués, tout en étant capable de recevoir des rapports.
    * **enforce**: Les serveurs de messagerie expéditeurs qui prennent en charge MTA STS ne délivreront pas de courrier à votre domaine lorsque l'authentification du certificat échoue, ou ne peut pas négocier TLS. Des rapports sur ces échecs sont également envoyés.
    * **none**: Les expéditeurs traiteront le domaine comme s'il n'avait pas de politique active. Cela désactive effectivement le MTA STS.
* **mx**: Les enregistrements MX pour le domaine. Ils doivent correspondre aux enregistrements MX publiés dans le DNS de votre domaine. Vous pouvez spécifier le FQDN ou un hôte joker (mx : mail.example.org ou mx : *.example.org). Assurez-vous que chaque enregistrement MX est ajouté sur sa propre ligne dans le fichier de stratégie.
* **max_age**: La durée de vie maximale de la politique exprimée en secondes. Cela représente la durée pendant laquelle un expéditeur mettrait en cache la politique du domaine. Il est recommandé d'utiliser une valeur équivalente à plusieurs semaines ou plus, mais ne dépassant pas 31557600 (environ 1 an).

Pour le domaine **xoyize.xyz** 

**A - caddy (UTILISE)**

[Using Caddy to enable MTA-STS](https://www.ericlight.com/using-caddy-to-enable-mta-sts/)

Quoi qu'il en soit, une fois que j'ai utilisé la directive respond de Caddy, il était facile de l'utiliser pour servir mon fichier de politique MTA-STS très pratique !

Il suffit d'ajouter ce nouveau bloc Server à votre fichier Caddy `/etc/caddy/Caddyfile` :

```
mta-sts.xoyize.xyz {

respond "version: STSv1
mode: testing
mx: mx1.xoyize.xyz
max_age: 86401"

header Content-Type "text/plain; charset=utf-8"

}
```

Ce simple bloc serveur vous permettra d'obtenir un certificat HTTPS brillant, de rediriger tous les visiteurs HTTP vers HTTPS, et de fournir aux demandeurs votre fichier de politique MTA. (Évidemment, remplacez mon nom de domaine et mes entrées MX par les vôtres).

Les observateurs attentifs remarqueront que le fichier de politique MTA-STS devrait être hébergé à l'adresse https://mta-sts.xoyize.xyz/.well-known/mta-sts.txt... mais c'est là toute l'élégance de la directive **Respond** de Caddy : où que vous alliez sur mta-sts.xoyize.xyz, vous obtiendrez exactement la même réponse !

<https://mta-sts.xoyize.xyz/.well-known/mta-sts.txt>  
![](mta-sts.xoyize.xyz.png)  

[MTA-STS Checker - Check MTA-STS Record](https://dmarcly.com/tools/mta-tls-checker)

**B - nginx**  
Fichier `/var/caddy/www/.well-known/mta-sts.txt`

```
version : STSv1
mode : testing
max_age : 604800
mx : mx1.xoyize.xyz
```

Mise en forme pour affichage navigateur

```
version: STSv1\r\nmode: testing\r\nmx: xoyize.xyz\r\nmax_age: 604800\r\n
```

fichier configuration `/etc/nginx/conf.d/mta-sts.conf`

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name mta-sts.xoyize.xyz;
    root /var/caddy/www;

    include /etc/nginx/conf.d/security.conf.inc;
      location = /.well-known/mta-sts.txt {
        default_type text/plain;
        return 200 "version: STSv1\r\nmode: testing\r\nmx: mx1.xoyize.xyz\r\nmax_age: 604800\r\n";
      }
}
```

Redémarrez NGINX (`sudo systemctl reload nginx`) et pointez le sous-domaine mta-sts vers le serveur NGINX via un enregistrement A ou !AAAA

![](mta_sts_doc1.png)  
<https://mta-sts.xoyize.xyz/.well-known/mta-sts.txt>  

#### TLSA (DANE)

*Le résumé du RFC 6698 « The DNS-Based Authentication of Named Entities (DANE) Transport Layer Security (TLS) Protocol : TLSA", dans lequel DANE est proposé, décrit assez bien ce qu'est DANE : « Les communications cryptées sur l'internet utilisent souvent le protocole TLS (Transport Layer Security), qui dépend de tiers pour certifier les clés utilisées. Ce document améliore cette situation en permettant aux administrateurs de noms de domaine de spécifier les clés utilisées dans les serveurs TLS de ce domaine. Cela nécessite des améliorations correspondantes dans le logiciel client TLS, mais aucun changement dans le logiciel serveur TLS ».*

>**TLS** représente la sécurité de la couche de transport. Le protocole cryptographique fournit une sécurité de bout en bout des données et est généralement implémenté pour chiffrer les protocoles Application Layer comme SMTP, HTTP, FTP et IMAP. Ce mécanisme est mis en œuvre en collaboration avec le MTA-STS et le DANE. Le protocole permet à un domaine de signaler des problèmes de livraison d'emails lorsque l'email manque de chiffrement TLS. Grâce au support MTA-STS, il garantit que les courriels envoyés au domaine obtiennent le chiffrement TLS et sont livrés en toute sécurité.
{: .prompt-tip }

>**DANE**, DNS-based Authentification of Named Entities (DANE) for SMTP fournit une méthode sécurisée pour le transport de courrier en permettant au propriétaire du domaine de certifier les clés utilisées par ses clients ou serveurs et de générer un certificat. Le mécanisme est censé être publié dans le domaine MX. La vérification DANE peut encore être supportée par un serveur de messagerie de domaine différent en demandant à l'administrateur et en mettant en place des enregistrements TLSA. Les extensions de sécurité du système de noms de domaine (DNSSEC) sont une exigence pour DANE. Pour que le modèle de sécurité fonctionne, l'enregistrement DNS doit être signé avec DNSSEC.
{: .prompt-tip }

[How to use DANE/TLSA](https://weberblog.net/how-to-use-danetlsa/)

Il est également recommandé de définir un enregistrement TLSA (DANE). Utilisez <https://www.huque.com/bin/gen_tlsa> pour en générer un.  
Copier/coller le contenu du certificat `/etc/maddy/certs/mx1.xoyize.xyz/fullchain.pem`, port sur 25, le protocole de transport sur "tcp" et le nom de domaine sur le nom d'hôte MX.  
![](xoyize.xyz-tlsa.png)  

Et cliquer sur generate  
![](xoyize.xyz-tlsa1.png)  

```
_25._tcp.mx1.xoyize.xyz. IN TLSA 3 1 1 (93f7192e5a9ebfd5d79eeaec632b03e6aef6502c0608bd14ec7f7895a6f327b0)
```

Ajouter une entrée TLSA à l'enregistrement DNS xoyize.xyz OVH  
![](xoyize.xyz-tlsa2.png)  

Test: `dig _25._tcp.mx1.xoyize.xyz tlsa +dnssec +multi`  

![](xoyize.xyz-tlsa3.png){:width="600"}  


### OVH DNS xoyize.xyz complet

DNS xoyize.xyz

```
$TTL 3600
@	IN SOA dns106.ovh.net. tech.ovh.net. (2025040200 86400 3600 3600000 60)
        IN NS     ns106.ovh.net.
        IN NS     dns106.ovh.net.
        IN MX     10 mx1
        IN A     158.220.91.148
        IN AAAA     2a02:c204:2231:478::1
    600 IN TXT     "v=spf1 a mx -all"
        IN CAA     128 issue "letsencrypt.org"
*        IN A     158.220.91.148
*        IN AAAA     2a02:c204:2231:478::1
_25._tcp.mx1        IN TLSA     3 1 1 93f7...5a6f327b0
_dmarc        IN TXT     "v=DMARC1;p=quarantine;"
_mta-sts        IN TXT     "v=STSv1; id=8E5b...6ky;"
_smtp._tls        IN TXT     "v=TLSRPTv1;rua=mailto:postmaster@xoyize.xyz"
default._domainkey        IN TXT     ( "v=DKIM1;k=rsa;p=MIIBIjANBgkqhkiG9w0BAQEFA...dLrP3ghQIDAQAB;" )
mx1        IN A     158.220.91.148
mx1        IN AAAA     2a02:c204:2231:478::1
```

### Comptes utilisateurs 

*Gestion base sqlite3 ou LLDAP*  

`Comptes utilisateurs gérer par base sqlite3`{: .prompt-info }

#### Gestion sqlite3 (ACTIVE)

![](SQLite3-logo-a.png){:width="70" .left}

*Contrairement à des logiciels comme postfix et dovecot, le __serveur de messagerie maddy__ utilise par défaut des <u>"utilisateurs virtuels"</u>, ce qui signifie qu'il ne se préoccupe pas des utilisateurs du système et n'en a pas connaissance.*

Les boîtes aux lettres IMAP ("comptes") et les identifiants d'authentification sont séparés.

Pour enregistrer les informations d'identification d'un utilisateur, utilisez la commande `maddy creds create`  

```bash
sudo -u maddy maddy creds create xoyi@xoyize.xyz # on vous demande de saisir le mot de passe du compte
```

Notez que le nom d'utilisateur est une adresse électronique. Ceci est nécessaire car le nom d'utilisateur est utilisé pour autoriser l'accès IMAP et SMTP (à moins que vous ne configuriez des mappings personnalisés, non décrits ici).

Après avoir enregistré les informations d'identification de l'utilisateur, vous devez également créer un compte de stockage local 

```bash
sudo -u maddy maddy imap-acct create xoyi@xoyize.xyz
```

Voilà, c'est fait. Vous avez maintenant votre première adresse électronique.  
Lorsque vous vous authentifiez avec votre client de messagerie, n'oubliez pas que le nom d'utilisateur est "xoyi@xoyize.xyz", et pas seulement "xoyi".

L'exécution de `maddy creds --help` et `maddy imap-acct --help` peut s'avérer utile pour connaître les autres commandes.  

Créer l'utilisateur postmaster sans shell ni home

    sudo useradd xoyi

Notez que les comptes IMAP et les identifiants sont gérés séparément, mais que les noms d'utilisateur doivent correspondre par défaut pour que les choses fonctionnent.

Ajout utilisateur local linux au serveur de messagerie et le stockage local

```bash
sudo -u maddy maddy creds create xouser@xoyize.xyz # on vous demande de saisir le mot de passe du compte
sudo -u maddy maddy imap-acct create xouser@xoyize.xyz
```

Liste des utilisateurs : `maddy creds list` et des stockages : `maddy imap-acct list`

```
sudo -u maddy maddy creds list
xouser@xoyize.xyz
xoyi@xoyize.xyz

sudo -u maddy maddy imap-acct list
xoyi@xoyize.xyz
xouser@xoyize.xyz
```

**OPTIONNEL**  
Le compte de soumission SMTP et le compte IMAP sont tous deux xoyi@xoyize.xyz, et non xoyi  

Dans le table.chain par défaut, seul le courrier envoyé au compte existant sera accepté.    
Si vous souhaitez que leno@xoyize.xyz reçoive tous les envois à une adresse inexistante, vous pouvez ajouter la ligne suivante au bloc `local_rewrites` de `table.chain` au fichier de configuration `/etc/maddy/maddy.conf` 

```
    optional_step regexp "(.+)@(.+)" "xoyi@$2"
```

`table.chain` après modification  
![](/images/maddy-table.chain
.png)   

La première règle redirigera le courrier électronique envoyé à foo+git@xoyize.xyz vers foo@xoyize.xyz  
La deuxième règle s'assure que le courrier destiné à postmaster (sans le domaine) sera envoyé à xoyi@xoyize.xyz  
La troisième règle trouvera la règle de redirection dans le fichier /etc/maddy/aliases  
Et la dernière règle FACULTATIVE transférera tous les courriers dont le destinataire est inconnu à xoyi@xoyize.xyz.

#### Alias de messagerie

Les alias de messagerie sont stockés dans le fichier `/etc/maddy/aliases` sous le format suivant  

```
# adresse mail alias: adresse mail destinataire
achats@xoyize.xyz: xoyi@xoyize.xyz
hideme@xoyize.xyz: xoyi@xoyize.xyz
vpn@xoyize.xyz: xoyi@xoyize.xyz
postmaster@xoyize.xyz: xoyi@xoyize.xyz
abuse@xoyize.xyz: xoyi@xoyize.xyz
```

Redémarrer le service

    sudo systemctl restart maddy

#### Gestion LLDAP (INACTIVE)

`Maddy créera automatiquement un compte imap si un nouvel utilisateur se connecte via LDAP.`{: .prompt-tip }

maddy prend en charge l'authentification via LDAP en utilisant la liaison DN. Les mots de passe sont vérifiés par le serveur LDAP.  
Maddy a besoin de connaître le DN à utiliser pour la liaison. Il peut être obtenu soit par recherche de répertoire ou template  

Notez que les backends de stockage utilisent traditionnellement des adresses e-mail, si vous utilisez des identifiants non-email comme nom d'utilisateur, vous devez les mapper sur les e-mails de livraison en utilisant auth_map (voir la page de documentation pour le backend de stockage utilisé).  
auth.ldap peut également être utilisé comme module de table. De cette façon, vous pouvez vérifier si le compte existe. Il fonctionne seulement si le modèle DN n'est pas utilisé.

[Exemple de configuration Maddy avec LLDAP](https://github.com/lldap/lldap/blob/main/example_configs/maddy.md)  
Exemple de configuration de maddy avec LLDAP fonctionnant dans docker.
Vous pouvez remplacer `local_authdb` par un autre nom si vous souhaitez utiliser plusieurs backends d'authentification.  
Si vous ne voulez utiliser qu'un seul backend de stockage, assurez-vous de désactiver `auth.pass_table local_authdb` dans votre configuration s'il est encore actif.

```
#auth.pass_table local_authdb {
#    table sql_table {
#        driver sqlite3
#        dsn credentials.db
#        table_name passwords
#    }
#}
auth.ldap local_authdb {
    urls ldap://127.0.0.1:3890

    bind plain "cn=admin,ou=people,dc=rnmkcy,dc=eu" "admin_password"
    base_dn "dc=rnmkcy,dc=eu"
    filter "(&(|(uid={username})(mail={username}))(objectClass=person))"

    starttls off
    debug off
    connect_timeout 1m
}
```

Après les modifications, on redémarre le serveur de messagerie

    sudo systemctl restart maddy

## Messagerie ouestline.xyz

*Ajout domaine ouestline.xyz pour la messagerie (maddy) et le web (caddy)*

### DNS ouestline.xyz

Enregistrement DNS MX pointe vers **mx1.xoyize.xyz**  

```
$TTL 3600
@	IN SOA dns111.ovh.net. tech.ovh.net. (2024120221 86400 3600 3600000 300)
        IN NS     ns111.ovh.net.
        IN NS     dns111.ovh.net.
        IN MX     10 mx1.xoyize.xyz
        IN A     158.220.91.148
        IN AAAA     2a02:c204:2231:478::1
        IN CAA     0 issue "letsencrypt.org"
*        IN A     158.220.91.148
*        IN AAAA     2a02:c204:2231:478::1
```

### Modifier le paramétrage maddy

Modifier le fichier de configuration `/etc/maddy/maddy.conf`

Ajouter tous les domaines que vous souhaitez que Maddy gère __`$(local_domains)`__, comme ceci :

```
$(primary_domain) = xoyize.xyz
$(local_domains) = $(primary_domain) ouestline.xyz
```

Notez que vous devez choisir un domaine comme « principal » à utiliser dans les messages générés automatiquement.
{: .prompt-info }

Vous pouvez créer des comptes en utilisant les deux domaines dans le nom, envoyer et recevoir des messages, etc...   
`N'oubliez pas de configurer les enregistrements SPF, DMARC et MTA-STS correspondants`{: .prompt-warning }.

Notez également que vous n'avez pas vraiment besoin d'un certificat TLS distinct pour chaque domaine géré. Vous pouvez avoir un nom d'hôte, par exemple **mx1.xoyize.xyz**, défini comme enregistrement MX pour plusieurs domaines.

### DKIM, DMARC et MTA-STS domaine ouestline.xyz

DKIM : `/var/lib/maddy/dkim_keys/ouestline.xyz_default.dns`  
_mta-sts id: Chaîne aléatoire de 32 caractères alphanumériques  

Les enregistrements DNS

```
_dmarc        IN TXT     "v=DMARC1;p=quarantine;ruf=mailto:postmaster@ouestline.xyz;"
# Générer un code alphanumérique de longueur 32 pour id :
# tr -cd '[:alnum:]' < /dev/urandom | fold -w32 | head -n1
_mta-sts        IN TXT     "v=STSv1; id=RzW4KbrYuyd6mg4wAkbe9sH2YS2I78wY;"
_smtp._tls        IN TXT     "v=TLSRPTv1;rua=mailto:postmaster@ouestline.xyz"
v=DKIM1; k=rsa; p=MIIBIjANBg...asCxTzdzHxgEQIDAQAB
```

Fichier de politique MTA-STS, il suffit d’ajouter ce nouveau bloc Server à votre fichier Caddy :

```
mta-sts.ouestline.xyz {
        respond "version: STSv1
mode: testing
mx: mx1.xoyize.xyz
max_age: 86401"

        header Content-Type "text/plain; charset=utf-8"
}
```

[MTA-STS Checker - Check MTA-STS Record](https://dmarcly.com/tools/mta-tls-checker)

### OVH DNS ouestline.xyz complet

![](dns-logo-new.png){:height="50"}

Configuration finale DNS ouestline.xyz

```
$TTL 3600
@	IN SOA dns111.ovh.net. tech.ovh.net. (2024120228 86400 3600 3600000 300)
            IN NS     ns111.ovh.net.
            IN NS     dns111.ovh.net.
            IN A     158.220.91.148
            IN AAAA     2a02:c204:2231:478::1
        600 IN TXT     "v=spf1 a mx ~all"
            IN CAA     0 issue "letsencrypt.org"
*           IN A     158.220.91.148
*           IN AAAA     2a02:c204:2231:478::1
_dmarc      IN TXT     "v=DMARC1;p=quarantine;rua=mailto:postmaster@ouestline.xyz;"
_mta-sts    IN TXT     "v=STSv1; id=RzW4KbrYuyd6mg4wAkbe9sH2YS2I78wY;"
_smtp._tls  IN TXT     "v=TLSRPTv1;rua=mailto:postmaster@ouestline.xyz"
default._domainkey        IN TXT     ( "v=DKIM1;k=rsa;p=MIIBIjANBg...asCxTzdzHxgEQIDAQAB;" )
```

### Utilisateurs ouestline.xyz

L'exécution de `maddy creds --help` et `maddy imap-acct --help` peut s'avérer utile pour connaître les autres commandes.  

Créer l'utilisateur postmaster sans shell ni home

    sudo useradd ouest

Notez que les comptes IMAP et les identifiants sont gérés séparément, mais que les noms d'utilisateur doivent correspondre par défaut pour que les choses fonctionnent.

Ajout utilisateur local linux au serveur de messagerie et le stockage local

```bash
sudo -u maddy maddy creds create ouest@ouestline.xyz # on vous demande de saisir le mot de passe du compte
sudo -u maddy maddy imap-acct create ouest@ouestline.xyz
```

Liste des utilisateurs : `maddy creds list` et des stockages : `maddy imap-acct list`

```
sudo -u maddy maddy creds list |grep ouest
ouest@ouestline.xyz

sudo -u maddy maddy imap-acct list |grep ouest
ouest@ouestline.xyz
```

### Alias ouestline.xyz

Ajouter au fichier alias `/etc/maddy/aliases`

```
postmaster@ouestline.xyz: ouest@ouestline.xyz
```


Redémarrer le service

    sudo systemctl restart maddy

### Modification msmtp pour intégrer le domaine ouestline.xyz

*__msmtp__ est un client SMTP très simple et facile à configurer pour l'envoi de courriels. Son mode de fonctionnement par défaut consiste à transférer les courriels au serveur SMTP que vous aurez indiqué dans sa configuration*

Modifier le fichier de configuration pour y ajouter le domaine ouestline.xyz

Créer la configuration

```shell
cat << 'EOF' >> ~/.msmtprc
#
account ouestline
host mx1.xoyize.xyz
port 587
from ouest@ouestline.xyz
user ouest@ouestline.xyz
password xxxxxxxxxxxxxxxxxxxx
EOF
```

Le mot de passe postmaster est ajouté au fichier `~/.msmtprc`


### Msmtp envoi message depuis ouestline.xyz en une ligne de commande 

```bash
echo -e "Subject: Test messagerie postmaster ouestline.xyz\r\nMIME-Version: 1.0\nContent-Type: text/; charset=utf-8\r\n\r\n \
/><head>Serveur maddy ouestline.xyz </head><body> \
<h2>Messagerie</h2><p>Test msmtp en mode ligne de commande </p></body>" |msmtp --from=ouest@ouestline.xyz -t yanfi@yanfi.net
```

Message reçu sur yanfi.net  
![](ouestline.xyz-msmtp.png)


## Messagerie cinay.eu

*Ajout domaine cinay.eu pour la messagerie (maddy) et le web (caddy)*

### DNS cinay.eu

Enregistrement DNS MX pointe vers **mx1.xoyize.xyz**  

```
$TTL 3600
@	IN SOA dns111.ovh.net. tech.ovh.net. (2024120221 86400 3600 3600000 300)
        IN NS     ns111.ovh.net.
        IN NS     dns111.ovh.net.
        IN MX     10 mx1.xoyize.xyz
        IN A     158.220.91.148
        IN AAAA     2a02:c204:2231:478::1
        IN CAA     0 issue "letsencrypt.org"
*        IN A     158.220.91.148
*        IN AAAA     2a02:c204:2231:478::1
```

### Modifier le paramétrage maddy

Modifier le fichier de configuration `/etc/maddy/maddy.conf`

Ajouter tous les domaines que vous souhaitez que Maddy gère __`$(local_domains)`__, comme ceci :

```
$(primary_domain) = xoyize.xyz
$(local_domains) = $(primary_domain) cinay.eu
```

Notez que vous devez choisir un domaine comme « principal » à utiliser dans les messages générés automatiquement.
{: .prompt-info }

Vous pouvez créer des comptes en utilisant les deux domaines dans le nom, envoyer et recevoir des messages, etc...   
`N'oubliez pas de configurer les enregistrements SPF, DMARC et MTA-STS correspondants`{: .prompt-warning }.

Notez également que vous n'avez pas vraiment besoin d'un certificat TLS distinct pour chaque domaine géré. Vous pouvez avoir un nom d'hôte, par exemple **mx1.xoyize.xyz**, défini comme enregistrement MX pour plusieurs domaines.

### DKIM, DMARC et MTA-STS domaine cinay.eu

DKIM : `/var/lib/maddy/dkim_keys/cinay.eu_default.dns`  
_mta-sts id: Chaîne aléatoire de 32 caractères alphanumériques  

Commandes pour générer une chaine aléatoire

```bash
# Alphanumérique Longueur 32 et nombre 1
cat /dev/urandom | tr -dc '[:alnum:]' | fold -w ${1:-32} | head -n 1
```

Les enregistrements DNS

```
_dmarc        IN TXT     "v=DMARC1;p=quarantine;ruf=mailto:postmaster@cinay.eu;"
# Générer un code alphanumérique de longueur 32 pour id :
# tr -cd '[:alnum:]' < /dev/urandom | fold -w32 | head -n1
_mta-sts        IN TXT     "v=STSv1; id=1iIaeWlbRGJcsxHNQh9uNQRHmZhaThtP;"
_smtp._tls        IN TXT     "v=TLSRPTv1;rua=mailto:postmaster@cinay.eu"
v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w..QwIDAQAB
```

Fichier de politique MTA-STS, il suffit d’ajouter ce nouveau bloc Server à votre fichier Caddy :

```
mta-sts.cinay.eu {
        respond "version: STSv1
mode: testing
mx: mx1.xoyize.xyz
max_age: 86401"

        header Content-Type "text/plain; charset=utf-8"
}
```

[MTA-STS Checker - Check MTA-STS Record](https://dmarcly.com/tools/mta-tls-checker)

### OVH DNS cinay.eu complet

![](dns-logo-new.png){:height="50"}

Configuration finale DNS cinay.eu

```
$TTL 3600
@	IN SOA dns111.ovh.net. tech.ovh.net. (2024120228 86400 3600 3600000 300)
            IN NS     ns111.ovh.net.
            IN NS     dns111.ovh.net.
            IN A     158.220.91.148
            IN AAAA     2a02:c204:2231:478::1
        600 IN TXT     "v=spf1 a mx ~all"
            IN CAA     0 issue "letsencrypt.org"
*           IN A     158.220.91.148
*           IN AAAA     2a02:c204:2231:478::1
_dmarc      IN TXT     "v=DMARC1;p=quarantine;rua=mailto:postmaster@cinay.eu;"
_mta-sts    IN TXT     "v=STSv1; id=1iIaeWlbRGJcsxHNQh9uNQRHmZhaThtP;"
_smtp._tls  IN TXT     "v=TLSRPTv1;rua=mailto:postmaster@cinay.eu"
default._domainkey        IN TXT     ( "v=DKIM1; k=rsa; p=MIIBIjANBgkqhki...r+Hp+RFNOAQBxLIQwIDAQAB" )
```

### Utilisateurs cinay.eu

L'exécution de `maddy creds --help` et `maddy imap-acct --help` peut s'avérer utile pour connaître les autres commandes.  

Créer l'utilisateur yack sans shell ni home

    sudo useradd yack

Notez que les comptes IMAP et les identifiants sont gérés séparément, mais que les noms d'utilisateur doivent correspondre par défaut pour que les choses fonctionnent.

Ajout utilisateur local linux au serveur de messagerie et le stockage local

```bash
sudo -u maddy maddy creds create yack@cinay.eu # on vous demande de saisir le mot de passe du compte
sudo -u maddy maddy imap-acct create yack@cinay.eu
```

Liste des utilisateurs : `maddy creds list` et des stockages : `maddy imap-acct list`

```
sudo -u maddy maddy creds list |grep yack
yack@cinay.eu

sudo -u maddy maddy imap-acct list |grep yack
yack@cinay.eu
```

### Alias cinay.eu

Ajouter au fichier alias `/etc/maddy/aliases`

```
postmaster@cinay.eu: yack@cinay.eu
```

Redémarrer le service

    sudo systemctl restart maddy

### Modification msmtp pour intégrer le domaine cinay.eu

*__msmtp__ est un client SMTP très simple et facile à configurer pour l'envoi de courriels. Son mode de fonctionnement par défaut consiste à transférer les courriels au serveur SMTP que vous aurez indiqué dans sa configuration*

Modifier le fichier de configuration pour y ajouter le domaine cinay.eu

Créer la configuration

```shell
cat << 'EOF' >> ~/.msmtprc
#
account cinay
host mx1.xoyize.xyz
port 587
from yack@cinay.eu
user yack@cinay.eu
password xxxxxxxxxxxxxxxxxxxx
EOF
```

Le mot de passe postmaster est ajouté au fichier `~/.msmtprc`


### Msmtp envoi message depuis cinay.eu en une ligne de commande 

```bash
echo -e "Subject: Test messagerie postmaster cinay.eu\r\nMIME-Version: 1.0\nContent-Type: text/; charset=utf-8\r\n\r\n \
/><head>Serveur maddy cinay.eu </head><body> \
<h2>Messagerie</h2><p>Test msmtp en mode ligne de commande </p></body>" |msmtp --from=yack@cinay.eu -t yanfi@yanfi.net
```

## Messagerie yanfi.net

*Ajout domaine yanfi.net pour la messagerie (maddy) et le web (caddy)*

### DNS yanfi.net

Enregistrement DNS MX pointe vers **mx1.xoyize.xyz**  

```
$TTL 3600
@	IN SOA dns111.ovh.net. tech.ovh.net. (2024120221 86400 3600 3600000 300)
        IN NS     ns111.ovh.net.
        IN NS     dns111.ovh.net.
        IN MX     10 mx1.xoyize.xyz
        IN A     158.220.91.148
        IN AAAA     2a02:c204:2231:478::1
        IN CAA     0 issue "letsencrypt.org"
*        IN A     158.220.91.148
*        IN AAAA     2a02:c204:2231:478::1
```

### Modifier le paramétrage maddy

Modifier le fichier de configuration `/etc/maddy/maddy.conf`

Ajouter tous les domaines que vous souhaitez que Maddy gère __`$(local_domains)`__, comme ceci :

```
$(primary_domain) = xoyize.xyz
$(local_domains) = $(primary_domain) yanfi.net
```

Notez que vous devez choisir un domaine comme « principal » à utiliser dans les messages générés automatiquement.
{: .prompt-info }

Redémarrer maddy

    sudo systemctl restart maddy

Vous pouvez créer des comptes en utilisant les deux domaines dans le nom, envoyer et recevoir des messages, etc...   
`N'oubliez pas de configurer les enregistrements SPF, DMARC et MTA-STS correspondants`{: .prompt-warning }.

Notez également que vous n'avez pas vraiment besoin d'un certificat TLS distinct pour chaque domaine géré. Vous pouvez avoir un nom d'hôte, par exemple **mx1.xoyize.xyz**, défini comme enregistrement MX pour plusieurs domaines.

### DKIM, DMARC et MTA-STS domaine yanfi.net

DKIM : `/var/lib/maddy/dkim_keys/yanfi.net_default.dns`  
_mta-sts id: Chaîne aléatoire de 32 caractères alphanumériques  

Commandes pour générer une chaine aléatoire

```bash
# Alphanumérique Longueur 32 et nombre 1
cat /dev/urandom | tr -dc '[:alnum:]' | fold -w ${1:-32} | head -n 1
```

Les enregistrements DNS

```
_dmarc        IN TXT     "v=DMARC1;p=quarantine;ruf=mailto:postmaster@yanfi.net;"
# Générer un code alphanumérique de longueur 32 pour id :
# tr -cd '[:alnum:]' < /dev/urandom | fold -w32 | head -n1
_mta-sts        IN TXT     "v=STSv1; id=quoQ800pwMEOA7FjaVltD1w1Mxyvd3EA;"
_smtp._tls        IN TXT     "v=TLSRPTv1;rua=mailto:postmaster@yanfi.net"
v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCg...rIfPJp1USphE9u0QIDAQAB
```

Fichier de politique MTA-STS, il suffit d’ajouter ce nouveau bloc Server à votre fichier Caddy :

```
mta-sts.yanfi.net {
        respond "version: STSv1
mode: testing
mx: mx1.xoyize.xyz
max_age: 86401"

        header Content-Type "text/plain; charset=utf-8"
}
```

[MTA-STS Checker - Check MTA-STS Record](https://dmarcly.com/tools/mta-tls-checker)

### OVH DNS yanfi.net complet

![](dns-logo-new.png){:height="50"}

Configuration finale DNS yanfi.net

```
$TTL 3600
@	IN SOA dns111.ovh.net. tech.ovh.net. (2024120228 86400 3600 3600000 300)
            IN NS     ns111.ovh.net.
            IN NS     dns111.ovh.net.
            IN A     158.220.91.148
            IN AAAA     2a02:c204:2231:478::1
        600 IN TXT     "v=spf1 a mx ~all"
            IN CAA     0 issue "letsencrypt.org"
*           IN A     158.220.91.148
*           IN AAAA     2a02:c204:2231:478::1
_dmarc      IN TXT     "v=DMARC1;p=quarantine;rua=mailto:postmaster@yanfi.net;"
_mta-sts    IN TXT     "v=STSv1; id=quoQ800pwMEOA7FjaVltD1w1Mxyvd3EA;"
_smtp._tls  IN TXT     "v=TLSRPTv1;rua=mailto:postmaster@yanfi.net"
default._domainkey        IN TXT     ( "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA5VSj5dWgL4JMitusWh3xVZck7phN6ys+sz4TKOv0tXRwi6zHFwC6VIR5TqgdurhgQrlxVixFS0GoPm3fQyMRawN0x9wOWiQuPg5v7WcyhlY27FjlpLvo1YuXG4+M+rhTN3VDwNv/CCrys7LbGtXwFql4CKTTxIndieyWEbWlghkCGhSLCdwxKO47HTDFjb8XTdTMEJnNeNLi0fWn1+KzexvayCd+5QGGVwQfs+OULsKlYHa6k3mSNSmkbm/bL1sbHrL5Ug0aVpLQULq7GD0GLZ3gN0/W5eej9Hlr8KvtXD44tHIKC6tXEiAKVNM/WleRZd8Vi8rIfPJp1USphE9u0QIDAQAB" )
```

### Utilisateurs yanfi.net

L'exécution de `maddy creds --help` et `maddy imap-acct --help` peut s'avérer utile pour connaître les autres commandes.  

Créer les utilisateurs claudine et yanfi sans shell ni home

    sudo useradd claudine
    sudo useradd yanfi

Notez que les comptes IMAP et les identifiants sont gérés séparément, mais que les noms d'utilisateur doivent correspondre par défaut pour que les choses fonctionnent.

Ajout utilisateur local linux au serveur de messagerie et le stockage local

```bash
sudo -u maddy maddy creds create yanfi@yanfi.net # on vous demande de saisir le mot de passe du compte
sudo -u maddy maddy imap-acct create yanfi@yanfi.net

sudo -u maddy maddy creds create claudine@yanfi.net # on vous demande de saisir le mot de passe du compte
sudo -u maddy maddy imap-acct create claudine@yanfi.net
```

Liste des utilisateurs : `maddy creds list` et des stockages : `maddy imap-acct list`

```
sudo -u maddy maddy creds list 
sudo -u maddy maddy imap-acct list 
```

### Alias yanfi.net

Ajouter au fichier alias `/etc/maddy/aliases`

```
postmaster@yanfi.net: yanfi@yanfi.net
```

Redémarrer le service

    sudo systemctl restart maddy

### Modification msmtp pour intégrer le domaine yanfi.net

*__msmtp__ est un client SMTP très simple et facile à configurer pour l'envoi de courriels. Son mode de fonctionnement par défaut consiste à transférer les courriels au serveur SMTP que vous aurez indiqué dans sa configuration*

Modifier le fichier de configuration pour y ajouter le domaine yanfi.net

Créer la configuration

```shell
cat << 'EOF' >> ~/.msmtprc
#
account yanfi
host mx1.xoyize.xyz
port 587
from yanfi@yanfi.net
user yanfi@yanfi.net
password xxxxxxxxxxxxxxxxxxxx
EOF
```

Le mot de passe postmaster est ajouté au fichier `~/.msmtprc`

### Msmtp envoi message depuis yanfi.net en une ligne de commande 

```bash
echo -e "Subject: Test messagerie postmaster yanfi.net\r\nMIME-Version: 1.0\nContent-Type: text/; charset=utf-8\r\n\r\n \
/><head>Serveur maddy yanfi.net </head><body> \
<h2>Messagerie</h2><p>Test msmtp en mode ligne de commande </p></body>" |msmtp --from=yanfi@yanfi.net -t ouest@ouestline.net
```

## Messagerie rnmkcy.eu

*Ajout domaine rnmkcy.eu pour la messagerie (maddy) et le web (caddy)*

### DNS rnmkcy.eu

Enregistrement DNS MX pointe vers **mx1.xoyize.xyz**  

```
$TTL 3600
@	IN SOA dns110.ovh.net. tech.ovh.net. (2025043000 86400 3600 3600000 300)
        IN NS     dns110.ovh.net.
        IN NS     ns110.ovh.net.
        IN MX     10 mx1.xoyize.xyz.
        IN A     82.64.18.243
        IN AAAA     2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45
        IN CAA     128 issue "letsencrypt.org"
*        IN A     82.64.18.243
*        IN AAAA     2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45
```

### Modifier le paramétrage maddy

Modifier le fichier de configuration `/etc/maddy/maddy.conf`

Ajouter tous les domaines que vous souhaitez que Maddy gère __`$(local_domains)`__, comme ceci :

```
$(primary_domain) = xoyize.xyz
$(local_domains) = $(primary_domain) rnmkcy.eu
```

Notez que vous devez choisir un domaine comme « principal » à utiliser dans les messages générés automatiquement.
{: .prompt-info }

Redémarrer maddy

    sudo systemctl restart maddy

Vous pouvez créer des comptes en utilisant les deux domaines dans le nom, envoyer et recevoir des messages, etc...   
`N'oubliez pas de configurer les enregistrements SPF, DMARC et MTA-STS correspondants`{: .prompt-warning }.

Notez également que vous n'avez pas vraiment besoin d'un certificat TLS distinct pour chaque domaine géré. Vous pouvez avoir un nom d'hôte, par exemple **mx1.xoyize.xyz**, défini comme enregistrement MX pour plusieurs domaines.

### DKIM, DMARC et MTA-STS domaine rnmkcy.eu

DKIM : `/var/lib/maddy/dkim_keys/rnmkcy.eu_default.dns`  
_mta-sts id: Chaîne aléatoire de 32 caractères alphanumériques  

Commandes pour générer une chaine aléatoire

```bash
# Alphanumérique Longueur 32 et nombre 1
cat /dev/urandom | tr -dc '[:alnum:]' | fold -w ${1:-32} | head -n 1
```

Les enregistrements DNS

```
_dmarc        IN TXT     "v=DMARC1;p=quarantine;"
# Générer un code alphanumérique de longueur 32 pour id :
# tr -cd '[:alnum:]' < /dev/urandom | fold -w32 | head -n1
_mta-sts        IN TXT     "v=STSv1; id=x3Tms0KSWIXGtpsz722YmEN7NoY0gmwc"
_smtp._tls         IN TXT     "v=TLSRPTv1;rua=mailto:postmaster@rnmkcy.eu"
default._domainkey IN TXT     ( "v=DKIM1;k=rsa;p=MIIBIjANBg...lzQIDAQAB;" )
```

Fichier de politique MTA-STS, serveur nginx :

Créer le dossier `sudo mkdir -p /sharenfs/rnmkcy/racine/.well-known`

Fichier `/sharenfs/rnmkcy/racine/.well-known/mta-sts.txt`

```
version : STSv1
mode : testing
max_age : 604800
mx : mx1.xoyize.xyz
```

Mise en forme pour affichage navigateur

```
version: STSv1\r\nmode: testing\r\nmx: mx1.xoyize.xyz\r\nmax_age: 604800\r\n
```

fichier configuration `/etc/nginx/conf.d/mta-sts.rnmkcy.eu.conf`

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name mta-sts.rnmkcy.eu;
    root /sharenfs/rnmkcy/racine;

    include /etc/nginx/conf.d/security.conf.inc;
      location = /.well-known/mta-sts.txt {
        default_type text/plain;
        return 200 "version: STSv1\r\nmode: testing\r\nmx: mx1.xoyize.xyz\r\nmax_age: 604800\r\n";
      }
}
```

Redémarrez NGINX (`sudo systemctl reload nginx`) et pointez le sous-domaine mta-sts vers le serveur NGINX via un enregistrement A ou !AAAA
<https://mta-sts.rnmkcy.eu/.well-known/mta-sts.txt>  

[MTA-STS Checker - Check MTA-STS Record](https://dmarcly.com/tools/mta-tls-checker)

### OVH DNS rnmkcy.eu complet

![](dns-logo-new.png){:height="50"}

Configuration finale DNS rnmkcy.eu

```
$TTL 3600
@	IN SOA dns110.ovh.net. tech.ovh.net. (2025043005 86400 3600 3600000 300)
        IN NS     dns110.ovh.net.
        IN NS     ns110.ovh.net.
        IN MX     10 mx1.xoyize.xyz.
        IN A     82.64.18.243
        IN AAAA     2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45
        IN CAA     128 issue "letsencrypt.org"
*        IN A     82.64.18.243
*        IN AAAA     2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45
*    600 IN TXT     "v=spf1 a mx -all"
_dmarc        IN TXT     "v=DMARC1;p=quarantine;"
_mta-sts        IN TXT     "v=STSv1; id=x3Tms0KSWIXGtpsz722YmEN7NoY0gmwc"
_smtp._tls        IN TXT     "v=TLSRPTv1;rua=mailto:postmaster@rnmkcy.eu"
default._domainkey        IN TXT     ( "v=DKIM1;k=rsa;p=MIIBIjANBg...lzQIDAQAB;" )
```

### Utilisateurs rnmkcy.eu

L'exécution de `maddy creds --help` et `maddy imap-acct --help` peut s'avérer utile pour connaître les autres commandes.  

Créer l'utilisateur sans shell ni home

    sudo useradd cwwk

Notez que les comptes IMAP et les identifiants sont gérés séparément, mais que les noms d'utilisateur doivent correspondre par défaut pour que les choses fonctionnent.

Ajout utilisateur local linux au serveur de messagerie et le stockage local

```bash
sudo -u maddy maddy creds create cwwk@rnmkcy.eu # on vous demande de saisir le mot de passe du compte
sudo -u maddy maddy imap-acct create cwwk@rnmkcy.eu
```

Liste des utilisateurs : `maddy creds list` et des stockages : `maddy imap-acct list`

```
sudo -u maddy maddy creds list 
sudo -u maddy maddy imap-acct list 
```

### Alias rnmkcy.eu

Ajouter au fichier alias `/etc/maddy/aliases`

```
postmaster@rnmkcy.eu: cwwk@rnmkcy.eu
```

Redémarrer le service

    sudo systemctl restart maddy

### Modification msmtp pour intégrer le domaine rnmkcy.eu

*__msmtp__ est un client SMTP très simple et facile à configurer pour l'envoi de courriels. Son mode de fonctionnement par défaut consiste à transférer les courriels au serveur SMTP que vous aurez indiqué dans sa configuration*

Modifier le fichier de configuration pour y ajouter le domaine rnmkcy.eu

Créer la configuration

```shell
cat << 'EOF' >> ~/.msmtprc
#
account cwwk
host mx1.xoyize.xyz
port 587
from cwwk@rnmkcy.eu
user cwwk@rnmkcy.eu
password xxxxxxxxxxxxxxxxxxxx
EOF
```

Le mot de passe postmaster est ajouté au fichier `~/.msmtprc`

### Msmtp envoi message depuis rnmkcy.eu en une ligne de commande 

Pour utiliser mail

```shell
sudo ln -fs /usr/bin/msmtp /usr/sbin/sendmail
sudo apt install bsd-mailx
echo "set mta=/usr/bin/msmtp" |sudo tee /etc/mail.rc
```

Test envoi message

```bash
# mail [-dEIinv] [-a header] [-b bcc-addr] [-c cc-addr] [-r from-addr] [-s subject] [--] to-addr ...
echo "Test messagerie cwwk rnmkcy.eu" | mail -r cwwk@rnmkcy.eu -s "Depuis serveur $HOSTNAME" yack@cinay.eu
```

## Test et vérification serveur mail

[17 outils SMTP pour diagnostiquer et tester la sécurité du courrier électronique](https://geekflare.com/fr/smtp-testing-tools/)

### Validateur MTA-STS

[Validateur MTA-STS gratuit](https://esmtp.email/tools/mta-sts/)  
![](mta_sts_xoyize.xyz.png)  

### CheckTLS

[CheckTLS](https://www.checktls.com) vous permet de vérifier la sécurité de votre messagerie et le fonctionnement de vos courriels. EmailSentry Outlook vous aide à vous assurer que tous les courriels que vous envoyez sont privés, légaux et sûrs.  
![](xoyize.net08c1.png){:width="400"}   
L’outil est approuvé par HIPAA, CCPA, PCI, GDPR et d’autres réglementations de conformité pour la sécurité des emails. CheckTLS vous aide également à résoudre les problèmes détectés dans vos courriels après le test.  
![](xoyize.xyz-checktls01.png){:width="400"}  
![](xoyize.xyz-checktls02.png){:width="400"}   
![](xoyize.xyz-checktls03.png){:width="400"}   
![](xoyize.xyz-checktls04.png){:width="400"}   

![](xoyize.xyz-checktls.png)   

### SSL-Tools

[SSL-Tools](https://ssl-tools.net/mailservers) est un outil assez simple qui permet d’entrer la partie « domaine » d’une adresse électronique et de tester sa sécurité.  
![](xoyize.xyz-ssltools.png)   

[ImmuniWeb](https://www.immuniweb.com/ssl/) offre un outil efficace pour tester la sécurité SSL.  
![](xoyize.xyz-immuniweb.png)

### MTA-STS Lookup

[MTA-STS Lookup - Check domains for Inbound Transport Layer Security (TLS) Enforcement - MxToolbox](https://mxtoolbox.com/mta-sts.aspx)
![](xoyize.xyz-mta-sts-a.png)   

### MxToolbox

[MxToolbox](https://mxtoolbox.com/diagnostic.aspx)
![](xoyize.xyz-mx-lookup.png)

### Mx lookup

[Mx lookup - MxToolBox](https://mxtoolbox.com/SuperTool.aspx)  
![](xoyize.xyz-mxtoolbox.png)   

### Vérifier DKIM

[Vérifier DKIM](https://dmarcian.com/fr/dkim-inspector/)  
![](xoyize.xyz-dkim-check.png)   

### Vérifier DMARC

[Vérifier DMARC](https://dmarcian.com/dmarc-inspector/)  
![](xoyize.xyz-dmarc-check.png)   

## Mail-tester

*mail-tester.com est un service en ligne gratuit qui vous permet de tester vos e-mails pour les problèmes de configuration Spam, Contenu Malformé et Mail Server*

Allez au testeur de courrier <https://www.mail-tester.com/>.  
![](mail-tester-xoyaz01.png){: .normal}

Résultats  
![](mail-tester-xoyize01.png){: .normal}  
Note 10/10 avec 2 remarques sans conséquences:

1. Il n'y a pas de version HTML de votre message.  
*Vérifiez si votre message est bien formaté ou non*
2. Votre message ne contient pas d'en-tête List-Unsubscribe  
*L'en-tête List-Unsubscribe est nécessaire si vous envoyez des emails en masse, il permet à l'utilisateur de facilement de désinscrire de vos listes.*

## Serveur web caddy

![](caddy-logo.png){:width="100" .left}

### xoyize.xyz

Créer dossier www (`WorkingDirectory=/var/caddy/www/xoyize.xyz`)

```bash
sudo mkdir -p /var/caddy/www/xoyize.xyz
echo "<h1>Serveur caddy xoyize.xyz</h1><h2>Dossier /var/caddy/www/xoyize.xyz/</h2>" | sudo tee -a /var/caddy/www/xoyize.xyz/index.html
```

Modifier fichier de configuration ,authentification basic

    sudo nano /etc/caddy/Caddyfile

```
# OVH Domaine xoyize.xyz
xoyize.xyz {
    root xoyize.xyz
    # Authentification basique
    basicauth * {
     xouser $2a$14$eUX5bO/DcXKakANhDeK8R.B7cd3rhWCUqBFZZvJpOst3tBlj4Q0ge
    }
    encode gzip
    file_server
}

```

on recharge

    sudo systemctl daemon-reload

On relance le service caddy

    sudo systemctl restart caddy

### ouestline.xyz

Créer dossier www (`WorkingDirectory=/var/caddy/www/xoyize.xyz`), y ajouter un fichier `index/` 

```bash
sudo mkdir -p /var/caddy/www/ouestline.xyz
sudo touch /var/caddy/www/ouestline.xyz/index/
echo "<h1>Serveur caddy ouestline.xyz</h1><h2>Dossier /var/caddy/www/ouestline.xyz/</h2>" | sudo tee -a /var/caddy/www/ouestline.xyz/index/
```

**Modifier le fichier de configuration** __`/etc/caddy/Caddyfile`__  
Ajouter

```
ouestline.xyz {
    root ouestline.xyz
    encode gzip
    file_server
}
```

On relance le service caddy

    sudo systemctl restart caddy

On vérifie sur le lien <https://ouestline.xyz>  
![](caddy-ouestline.xyz.png)

### Dossier "static"

Créer le dossier et modifier les droits

```shell
sudo mkdir -p /var/caddy/www/Divers/
sudo chown xouser /var/caddy/www/Divers/
```

Créer une première synchronisation via rsync de "static" depuis le poste linux appelant

```shell
rsync -avz --progress --stats -e "ssh -p 55148 -i /home/yann/.ssh/xoyize-ed25519" /home/yann/media/yannstatic/static  xouser@158.220.91.148:/var/caddy/www/Divers/
```

**Modifier le fichier de configuration** __`/etc/caddy/Caddyfile`__  afin qu'il ne serve que des fichiers statiques, utilisez la directive `file_server`, puis définissez le répertoire racine `/var/caddy/www/Divers/static`. L'astérisque (*) qui suit immédiatement la racine indique à Caddy qu'il doit répondre à toutes les requêtes - sans lui, Caddy ne fonctionnerait pas.

```
# Servir des fichiers statiques
static.xoyize.xyz {
    root * /var/caddy/www/Divers/static
    file_server
}

static.cinay.eu {
    root * /var/caddy/www/Divers/static
    file_server
}

static.yanfi.net {
    root * /var/caddy/www/Divers/static
    file_server
}

static.ouestline.xyz {
    root * /var/caddy/www/Divers/static
    file_server
}
```

On relance le service caddy

    sudo systemctl restart caddy

On vérifie sur les liens **https://static.domaine.tld**

## PHP8 MariaDB

### Php

![](php8-logo.png){: width="80" .left}  
*Installer la version PHP8.3 sur debian 12*

#### Ajout dépôt sury

Pour installer la version de 8 de php, ajouter le dépôt sury.

```bash
sudo apt install -y lsb-release apt-transport-https ca-certificates wget
sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" |sudo tee /etc/apt/sources.list.d/php.list
```

Mise à jour des dépôts :

sudo apt update && sudo apt upgrade -y

#### Installer PHP8.3 + Modules nécessaires à Nextcloud

Installer php8.3 et modules

```bash
sudo apt -y install \
   php8.3 \
   php8.3-fpm \
   php8.3-sqlite3 \
   php8.3-cli \
   php8.3-gd \
   php8.3-imap \
   php8.3-mysql \
   php8.3-soap \
   php8.3-apcu \
   php8.3-common \
   php8.3-gmp  \
   php8.3-intl \
   php8.3-opcache \
   php8.3-xml \
   php8.3-curl \
   php8.3-igbinary \
   php8.3-readline  \
   php8.3-zip \
   php8.3-bcmath \
   php8.3-imagick \
   php8.3-mbstring \
   php8.3-redis \
   php8.3-bz2 \
   php8.3-smbclient \
   imagemagick libmagickcore-6.q16-6-extra 
```

Vérifiez votre installation PHP en vérifiant sa version : `php --version`

```
PHP 8.3.14 (cli) (built: Nov 25 2024 18:23:27) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.3.14, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.14, Copyright (c), by Zend Technologies
```

Remplacer l’utilisateur et le groupe par défaut par caddy dans le fichier de configuration PHP-FPM.

    sudo nano /etc/php/8.3/fpm/pool.d/www.conf

Dans le fichier, recherchez les lignes ci-dessous et changez leur valeur en caddy :

```
user = caddy
group = caddy
listen.owner = caddy
listen.group = caddy
```

Redémarrer PHP-FPM pour appliquer les modifications :

    sudo systemctl restart php8.3-fpm

### Installer MariaDB

![](mariadb-logo-a.png){: width="50" .left}  

```bash
sudo apt install mariadb-server -y
sudo mysql_secure_installation # Y à tout et nouveau mot de passe n
```

## Sauvegardes

### BorgBackup + Notification ntfy

![](borg-logo-a.png){: width="100" .left}    
Dépôt: `ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/xoyize.xyz`  
Clé privée SSH borg: `/root/.ssh/id_borg_ed25519`  
Passphrase: `/root/.borg/xoyize.xyz.passphrase`  
Fichier d'exclusion: `/root/.borg/xoyize.xyz.exclusions`  
Exécution automatique: `02h10`  

Suivre la procédure suivante en modifiant dépôt,clé privée, passphrase et fichier d'exclusion:   [BorgBackup VPS](/posts/BorgBackup_entre_serveurs/#borgbackup-vps)

Le script modifié `/root/.borg/borg-backup.sh` pour envoi message via ntfy

```bash
export BORG_PASSPHRASE="$(cat /root/.borg/xoyize.xyz.passphrase)"
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/xoyize.xyz
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
borg create -v --progress --stats --exclude-from /root/.borg/xoyize.xyz.exclusions ${BORG_REPOSITORY}::${BACKUP_DATE} /
borg prune -v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 $BORG_REPOSITORY
# Message
curl \
-H "X-Email: ntfy@cinay.eu" \
-H "Title: 💻 $HOSTNAME : Fin Borgbackup `date +%d/%m/%Y-%Hh%M`" \
-H "Authorization: Bearer tk_jkrtwpoaqvgz2ic4nalzcfd447l5fzr" \
-H prio:low \
-d "Serveur contabo Debian 12
`hostname -I`
 ✔️ Fin de la sauvegarde Borgbackup `date +%d/%m/%Y-%Hh%M`" \
https://noti.rnmkcy.eu/yan_infos
```

`Modifier la valeur Bearer tk_...`{: .prompt-warning }

## Fail2ban

### Installer Fail2ban sur votre serveur

Pour commencer, installez Fail2ban à partir du dépôt par défaut de votre distribution Linux.

installer et configurer Fail2ban sur Debian. Exécutez la commande suivante pour mettre à jour le dépôt et installer Fail2ban :

```shell
sudo apt update
sudo apt install fail2ban -y
```

## Protégez votre serveur mail avec Fail2ban

*Protéger un serveur de messagerie est crucial dans le paysage numérique actuel, où les attaques par force brute et les abus liés au spam sont omniprésents. Fail2ban est un outil open-source robuste qui aide à protéger votre serveur contre les activités malveillantes en bloquant dynamiquement les adresses IP suspectes. Ce guide fournit une explication complète, étape par étape, sur la configuration de Fail2ban pour la protection d’un serveur de messagerie, garantissant une sécurité accrue des emails.*

* [How to integrate Caddy with fail2ban ](https://muetsch.io/how-to-integrate-caddy-with-fail2ban.html)
* [foxcpp/maddy](https://github.com/foxcpp/maddy/tree/master/dist/fail2ban)

### Configurer les paramètres par défaut de Fail2ban

*Les fichiers de configuration de Fail2ban se trouvent dans le répertoire `/etc/fail2ban`. Évitez de modifier directement le fichier jail.conf par défaut ; créez plutôt un fichier de remplacement local.*

Le fichier jail.local remplacera la configuration par défaut et restera intact lors des mises à jour.

```shell
sudo nano /etc/fail2ban/jail.local
```

Déclarer le backend comme étant systemd

```
[DEFAULT]
backend = systemd
```

### Protéger les services de messagerie avec Fail2ban

### maddy-dictonary-attack

Configurer le jail maddy-subm

Créer le fichier `/etc/fail2ban/jail.d/maddy-dictonary-attack.conf`

```
[maddy-dictonary-attack]
port     = 993,587,25
filter   = maddy-dictonary-attack
bantime  = 72h
maxretry = 3
findtime = 6h
backend  = systemd
```

Créer le fichier `/etc/fail2ban/filter.d/maddy-dictonary-attack.conf`

```
[INCLUDES]
before = common.conf

[Definition]
failregex    = smtp\: MAIL FROM error repeated a lot\, possible dictonary attack\t\{\"count\"\:\d+,\"msg_id\":\".+\",\"src_ip\"\:\"<HOST>:\d+\"\}$
               smtp\: too many RCPT errors\, possible dictonary attack\t\{\"msg_id\":\".+\","src_ip":"<HOST>:\d+\"\}
journalmatch = _SYSTEMD_UNIT=maddy.service + _COMM=maddy
```


### maddy-auth.conf

Créer le fichier `/etc/fail2ban/jail.d/maddy-auth.conf`

```
[maddy-auth]
port     = 993,587,25
filter   = maddy-auth
bantime  = 96h
backend  = systemd
```

Créer le fichier `/etc/fail2ban/filter.d/maddy-auth.conf`

```
[INCLUDES]
before = common.conf

[Definition]
failregex    = authentication failed\t\{\"reason\":\".*\",\"src_ip\"\:\"<HOST>:\d+\"\,\"username\"\:\".*\"\}$
journalmatch = _SYSTEMD_UNIT=maddy.service + _COMM=maddy
```

## Maintenance

### Ne plus recevoir de message de dmarc-support@google.com

Pour ne plus rece voir des messages de dmarc-support@google.com, il faut modifier les DNS du serveur de messagerie  
Remplacer l'enregistrement dmarc

```
_dmarc        IN TXT     "v=DMARC1;p=quarantine;rua=mailto:postmaster@cinay.eu;"
```

par

```
_dmarc        IN TXT     "v=DMARC1;p=quarantine;"
```

### Debian - Mises à jour auto

* [Mises à jour automatiques sur Debian](/posts/Debian_unattended-upgrades_mise_a_jour_auto/)

### Snapshots

* **Snapshots** - capture l'état du VPS concerné et le stocke pour un retour en arrière ultérieur. Veuillez noter que les instantanés sont une fonctionnalité disponible uniquement pour certains SDV, qu'ils ne remplacent pas une sauvegarde correcte de vos données (c.-à-d. non programmée, uniquement des retours en arrière complets, pas de récupération d'un seul fichier) et qu'ils seront automatiquement supprimés après 30 jours (voir la vue détaillée).
* **Snapshots Rollback** - applique un snapshot au Serveur Virtuel concerné. Accessible dans la vue détaillée des snapshots. Veuillez noter que toutes les données sur le VPS seront remplacées par les données de l'instantané, il n'y a aucune garantie de réussite et dans le cas d'instantanés multiples, le retour en arrière d'un instantané supprimera automatiquement tous les instantanés plus récents du VPS concerné.

Création snapshot le 23 avril 2025  
![](xoyize-snapshot.png)

## Nebula + Serveur NFS

### Nebula

* [Nebula est un outil pour interconnecter de manière transparente des ordinateurs](/posts/Nebula/)
    * [Contabo serveur xoyize](/posts/Nebula/#contabo-serveur-xoyize)

IP Privée: 10.19.55.5

### Serveur NFS

* [NFSv4 (Network File System), partages réseau linux](/posts/NFSv4/)
* [NFS : partage réseau sécurisé et rapide](https://blog.stephane-robert.info/docs/services/stockage/nfs/)
* [How to auto-mount an NFS share using systemd](https://www.geraldonit.com/auto-mount-nfs-share-using-systemd/)

Exécutez la commande ufw suivante pour permettre à votre sous-réseau de réseau local d’accéder au serveur NFS. La commande suivante permettra au réseau 10.19.55.0/24 d’accéder au serveur NFS.

```shell
sudo ufw allow from 10.19.55.0/24 to any port nfs
sudo ufw reload
```

La règle ajoutée

```
2049                       ALLOW       10.19.55.0/24             
```

Créer le dossier

```shell
sudo mkdir -p /sharenfs/xoyize
sudo chown $USER:$USER -R /sharenfs/xoyize
```


Ouvrir le fichier `/etc/exports` à l’aide de la commande d’éditeur nano suivante.

    sudo nano /etc/exports

Ajoutez la ligne suivante au fichier.

```bash
/sharenfs/xoyize    10.19.55.0/255.255.255.0(rw,no_root_squash,no_subtree_check)
```

Redémarrer serveur NFS

```bash
sudo systemctl restart nfs-server 
sudo systemctl status nfs-server
```

Enfin, exécutez la commande suivante pour vérifier le répertoire et le système de fichiers exportés sur le serveur NFS à l’aide de la commande suivante.

```bash
sudo showmount -e 10.19.55.5 
```

Réponse

```
Export list for 10.19.55.5:
/sharenfs/xoyize 10.19.55.0/255.255.255.0
```

>Pour autoriser les accès au serveu NFS sur le réseau 10.19.55.0/24, il faut ouvrir les ports entrants du firewall nebula 
{: .prompt-tip }

Dans le fichier `/etc/nebula/config.yaml`, dans la zone `firewall:` , `inbound:` , remplacer `proto: icmp` par `proto: any` et relancer le service nebula; `suso systemctl restart nebula`