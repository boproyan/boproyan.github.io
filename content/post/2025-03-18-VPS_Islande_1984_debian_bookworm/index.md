+++
title = 'VPS Islande 1984 debian 12'
date = 2025-03-18 00:00:00 +0100
categories = serveur wireguard vpn
+++
<div class="item">
  <div class="item__image">
    <img src="iceland.png" width="80"/> <img src="1984-logo.svg" width="150"/>
  </div>
  <div class="item__content">
    <div class="item__header">
      <h4><a href="https://1984.hosting/">VPS Islande 1984</a></h4>
    </div>
    <div class="item__description">
      <p>VPS vpsg1btm3e</p>
		<ul>
		  <li>debian 12</li>
		  <li>1 GB RAM</li>
		  <li>25GB NVMe SSD DISK</li>
		  <li>1TB TRANSFER(tx+rx)</li>
		  <li>1 x vCPU Core</li>
		  <li>1 x IPv4 Address 185.112.146.46</li>
		</ul>
    </div>
  </div>
</div>

![](vpsg1btm3e.png)

## Debian bookworm

![](debian12-logo.png){: .normal}  

### Connexion serveur

L'intialisation du serveur VPS s'est faite en fournissant une une clé publique SSH

Connexion ssh avec la clé

    ssh -i .ssh/iceyan-vps root@185.112.146.46

Mise à jour 

    apt update && apt -y upgrade

Installer sudo

    apt install sudo

**Versions noyau et debian**  

    uname -a && cat /etc/debian_version

Linux iceyan.xyz 6.1.0-9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.27-1 (2023-05-08) x86_64 GNU/Linux  
12.5


Adressage : `ip -a`

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:b9:70:92:2e brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet 185.112.146.46/24 brd 185.112.146.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:b9ff:fe70:922e/64 scope link 
       valid_lft forever preferred_lft forever
```

Modifier ou créer `/etc/hostname` : iceyan.xyz  
Ajouter `185.112.146.46	iceyan.xyz` au fichier `/etc/hosts`

### Utilisateur

Créer un utilisateur **iceyan** avec shell par défaut et son mot de passe

    useradd --create-home --user-group iceyan
    passwd iceyan

Shell par défaut

    usermod -s /bin/bash iceyan

Autoriser sudo pour utilisateur

    echo "iceyan     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers


### OpenSSH, clé et script

![OpenSSH](ssh_logo1.png){:width="70"}  

**connexion avec clé**  
<u>sur l'ordinateur de bureau</u>
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) pour une liaison SSH avec le serveur.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/iceyan-vps

Envoyer les clés publiques sur le serveur KVM   

    ssh-copy-id -i ~/.ssh/iceyan-vps.pub iceyan@185.112.146.46

<u>sur le serveur KVM</u>  
On se connecte  

    ssh iceyan@185.112.146.46

Modifier la configuration serveur SSH  

    sudo nano /etc/ssh/sshd_config

Modifier

```conf
Port = 55046
PermitRootLogin no
PasswordAuthentication no
```

Relancer le serveur

    sudo systemctl restart sshd

Test connexion

    ssh -p 55046 -i ~/.ssh/iceyan-vps iceyan@185.112.146.46

### Utilitaires et historique

Installer utilitaires  

    sudo apt install rsync curl tmux jq figlet git dnsutils tree -y


Motd

    sudo rm /etc/motd && sudo nano /etc/motd

```
  _                                                       
 (_) __  ___  _  _  __ _  _ _     __ __ _  _  ___         
 | |/ _|/ -_)| || |/ _` || ' \  _ \ \ /| || ||_ /         
 |_|\__|\___| \_, |\__,_||_||_|(_)/_\_\ \_, |/__|         
              |__/                      |__/              
  _  ___  ___     _  _  ___     _  _ _    __    _ _    __ 
 / |( _ )| __|   / |/ ||_  )   / || | |  / /   | | |  / / 
 | |/ _ \|__ \ _ | || | / /  _ | ||_  _|/ _ \ _|_  _|/ _ \
 |_|\___/|___/(_)|_||_|/___|(_)|_|  |_| \___/(_) |_| \___/
```

Script **ssh_rc_bash**  
>**ATTENTION!!! Les scripts sur connexion peuvent poser des problèmes pour des appels externes autres que ssh**

```shell
wget https://static.xoyize.xyz/files/ssh_rc_bash
chmod +x ssh_rc_bash # rendre le bash exécutable
./ssh_rc_bash        # exécution
```

![](iceyan.xyz.png)

[Prise en charge historique, modification bashrc et inputrc](/posts/Configurer_et_utiliser_historique_bash/#prise-en-charge-historique-modification-bashrc-et-inputrc)  
POUR tous les utilisateurs ,y compris root et avec touche SHIFT

```bash
echo "
# appel alphabétique commandes
shopt -s histappend
PROMPT_COMMAND='history -a'" | sudo tee -a $HOME/.bashrc

echo "
# appel alphabétique commandes
shopt -s histappend
PROMPT_COMMAND='history -a'" | sudo tee -a /root/.bashrc

echo '
# AVEC la touche SHIFT
"\e[1;2A": history-search-backward
"\e[1;2B": history-search-forward
' | sudo tee -a /etc/inputrc
```

Redémarrer le terminal la prise en compte

### Hostname

Domaine iceyan.xyz

    hostnamectl 

```
 Static hostname: iceyan.xyz
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 75971165a9bd46a6814eeb1061620d63
         Boot ID: 6620df06ff2d4ccd851f310805a20ed2
  Virtualization: kvm
Operating System: Debian GNU/Linux 12 (bookworm)   
          Kernel: Linux 6.1.0-9-amd64
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: 1.14.0-2
```

### Parefeu

![ufw](ufw-logo-a.png){:width="50" .normal}  
*UFW, ou pare - feu simple , est une interface pour gérer les règles de pare-feu dans Arch Linux, Debian ou Ubuntu. UFW est utilisé via la ligne de commande (bien qu'il dispose d'interfaces graphiques disponibles), et vise à rendre la configuration du pare-feu facile (ou simple).*

Installer ufw

    sudo apt install ufw

Les règles par défaut

    sudo ufw default allow outgoing
    sudo ufw default deny incoming

Ajout des règles spécifiques

```shell
sudo ufw allow 55046/tcp  # Port ssh
sudo ufw allow 80/tcp     # http
sudo ufw allow 443/tcp    # https
sudo ufw allow 4443/tcp   # searXNG
sudo ufw allow dns        # DNS port 53
sudo ufw allow 853        # DNS over TLS
```

Désactiver IPV6

    sudo nano /etc/default/ufw

Changez la ligne qui dit : `IPV6=yes` en `IPV6=no` puis redémarrez le service ufw. Exécuter `sudo ufw reload` si l'instance ufw est déjà activée.

Activation parefeu

    sudo ufw enable

Valider la commande par touche y au clavier

```
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

Status du parefeu : `sudo ufw status`

```
Status: active

To                         Action      From
--                         ------      ----
55046/tcp                  ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
DNS                        ALLOW       Anywhere                  
853                        ALLOW       Anywhere                  
4443/tcp                   ALLOW       Anywhere                  
```

### OVH DNS iceyan.xyz

![](ovh-logo-a.png){:width="150" .normal} ![](dns-logo.png){:width="50" .normal} 

OVH: Domaine **iceyan.xyz** , DNSSEC Actif

```
$TTL 3600
@	IN SOA dns11.ovh.net. tech.ovh.net. (2023021400 86400 3600 3600000 60)
         IN NS     dns12.ovh.net.
         IN NS     ns12.ovh.net.
         IN A      185.112.146.46
*        IN A      185.112.146.46
```

### Postfix messagerie

![](postfix-logo.png){:width="100" .normal}  
*On va configurer Postfix afin qu’il puisse être utilisé pour envoyer des notifications par e-mail uniquement par les applications locales installées sur le même serveur que Postfix*

Installation outils

```shell
sudo apt install mailutils postfix
```

Installation postfix, répondre aux questions :  
Internet site &rarr; OK  
iceyan.xyz &rarr; OK

Configurer Postfix pour qu’il traite les demandes d’envoi d’e-mails uniquement à partir du serveur sur lequel il s’exécute, c’est-à-dire à partir de localhost.   
Postfix doit être configuré pour n’écouter que sur l’ interface loopback , l’interface réseau virtuelle que le serveur utilise pour communiquer en interne.

Modifier les lignes suivantes dans le fichier de configutration `/etc/postfix/main.cf`

```
mydestination = $myhostname, localhost.$mydomain, $mydomain
inet_interfaces = loopback-only
```

Redémarrez Postfix.

    sudo systemctl restart postfix 

Test envoi message 

    echo "Test envoi via postfix smtp" | mail -s "serveur debian iceyan.xyz" vpn@cinay.eu

## Dns DoT DoH

*DNS over TLS (DoT) et DNS over HTTPS (DoH) sont deux protocoles standardisés
par l'IETF dans les [RFC 7858](https://www.rfc-editor.org/rfc/rfc7858) et
[RFC 8484](https://www.rfc-editor.org/rfc/rfc8484) respectivement. Ces deux
protocoles ont pour but de sécuriser les requêtes DNS entre un client et le
résolveur (encapsulation du protocole DNS
dans une session TLS pour DoT ou HTTPS pour DoH.)*

**DNS**  
![](DoH_illustration_1.png){: width="350" .normal}  
Les requêtes DNS sont envoyées en texte brut, ce qui signifie que tout le monde peut les lire. Le DNS sur HTTPS et le DNS sur TLS chiffrent les requêtes et les réponses du DNS pour que la navigation des utilisateurs reste sécurisée et privée. Cependant, les deux approches ont leurs avantages et leurs inconvénients.

**DNS sur HTTPS et le DNS sur TLS**  
![](DoH_illustration_2.png){: width="350" .normal}  
Le DNS sur TLS et le DNS sur HTTPS</u> sont deux normes développées pour le chiffrement du trafic DNS en texte brut afin d'empêcher les parties malveillantes, les annonceurs, les FAI et autres de pouvoir interpréter les données. Pour poursuivre l'analogie, ces normes visent à enfermer les cartes postales envoyées par courrier dans une enveloppe, pour que quiconque puisse envoyer une carte postale sans craindre les indiscrétions d'un tiers.

#### DoH vs DoT

![](doh-dot-logo.png){: width="250" .left}Outre DNS sur HTTPS, il existe un autre protocole qui vise également à chiffrer les requêtes DNS. C'est ce qu'on appelle DNS sur TLS (DoT).  
Pour les personnes vivant dans des pays où la censure d'Internet est sévère, il est plus avantageux d'utiliser DoH.

* Le DoT fonctionne sur le port TCP 853 , qui peut être facilement bloqué par un pare-feu national.
* DoH fonctionne sur le port TCP 443 , qui est le port standard pour les sites Web HTTPS, ce qui rend DoH très difficile à bloquer, car si le port TCP 443 est bloqué, alors presque tous les sites Web HTTPS seront également bloqués.
* DoH permet aux applications Web d'accéder aux informations DNS via les API de navigateur existants, de sorte qu'aucun résolveur de stub n'est nécessaire.

### Résolveur (Unbound)

![](unbound-250.png){:width="150" .normal}  

Pour l'installation complète **Unbound** qui ajoute la mise à jour des serveurs "racine", le blocage des publicités et des DMP (Data Management Platforms), voir lien [Résolveur DNS Unbound](/posts/unbound-resolveur-DNS/)
{: .prompt-info } 

Commençons par installer et configurer le résolveur DNS. Il existe plusieurs
logiciels pour faire de la résolution comme [BIND 9](https://www.isc.org/bind),
[Knot Resolver](https://www.knot-resolver.cz) ou encore
[Unbound](https://nlnetlabs.nl/projects/unbound/about/). Nous avons choisi
d'utiliser Unbound et cette partie documente comment installer et configuer ce
résolveur.

En règle général Unbound est disponible dans les dépôts des distributions, l'installer depuis le gestionnaire de paquet de votre machine.

```bash
sudo apt install unbound
```

Maintenant que Unbound est installé, il ne reste plus qu'à le **configurer** avant
de démarrer le service.

Il s'agit du résolveur DNS, celui ci n'est accessible que depuis la machine
locale via le port 53.  

Créer le fichier de configuration `/etc/unbound/unbound.conf.d/unbound-iceyan.conf` avec le contenu suivant :

```
server:
    # ne rien enregistrer dans les journaux hormis les erreurs
    verbosity: 0

    # n'écouter que sur l'interface locale en IPv4
    # unbound nécessite d'être relancé si modifié
    interface: 127.0.0.1

    port: 53

    # refuser tout le monde sauf les connexions locales (pas forcément
    # nécessaire vu que le serveur n'écoute que sur la boucle locale en IPv4)
    access-control: 0.0.0.0/0 refuse
    access-control: 127.0.0.1/32 allow

    # par défaut, unbound ne log pas les requêtes ni les réponses
    # on peut le rappeler au cas où
    log-queries: no
    log-replies: no

    # imposer la QNAME minimisation (RFC 7816)
    # Pour mieux protéger la vie privée
    qname-minimisation: yes
    # même si le serveur faisant autorité ne le veut pas
    #   après discussion, il est possible que cette option ne soit
    #   pas recommandée dans le cadre d'un résolveur ouvert
    qname-minimisation-strict: yes
```

Vérifier la validité du fichier de configuration avec la commande
suivante :

```bash
sudo unbound-checkconf /etc/unbound/unbound.conf.d/unbound-iceyan.conf
```

*unbound-checkconf: no errors in /etc/unbound/unbound.conf.d/unbound-iceyan.conf*

Toutes les règles disponibles sont détaillées dans le manuel `man 5 unbound.conf` ou [dans le manuel en ligne](https://nlnetlabs.nl/documentation/unbound/unbound.conf/).

Démarrer le résolveur.

```bash
sudo systemctl start unbound.service
```

S'assurer que tout fonctionne bien à l'aide de la commande
`dig` disponible dans le paquet `bind9-dnsutils` ou `dnsutils`. Pour cela il
suffit de spécifier l'adresse de notre résolveur, ici `127.0.0.1` ou `::1` et
d'effectuer une requête DNS. Ici on demande à Unbound de récupérer
l'enregistrement `AAAA` associé au nom de domaine `afnic.fr`.

```bash
dig @127.0.0.1 A afnic.fr
```

Résultat commande 

```
; <<>> DiG 9.18.24-1-Debian <<>> @127.0.0.1 A afnic.fr
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33331
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;afnic.fr.			IN	A

;; ANSWER SECTION:
afnic.fr.		600	IN	A	51.178.83.21

;; Query time: 52 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Fri Feb 16 07:44:25 GMT 2024
;; MSG SIZE  rcvd: 53
```

Une réponse est bien renvoyée. Le résolveur fonctionne.Vérifier que tout est opérationnel en IPv4, et en utilisant UDP et TCP.

```
$ dig +notcp @127.0.0.1 A afnic.fr  # connexion UDP en IPv4 au résolveur
$ dig +tcp @127.0.0.1 A afnic.fr    # connexion TCP en IPv4 au résolveur
```

À ce stade, un résolveur Unbound est configuré en local et écoute sur le port
`53`. Il peut donc être utilisé pour résoudre toutes les requêtes en provenance
de la machine.
{: .prompt-info }

### Frontal DNS (dnsdist)

![](dnsdist-logo.png){: .normal}  
*dnsdist est un répartiteur de charge pour serveurs DNS avec la particularité de gérer DoH et DoT. Le but est donc de l'installer sur la même machine qu'Unbound et de le mettre devant : Unbound n'écoutera que localement et dnsdist, lui, sera ouvert au public*

* [POWERDNS - dnsdist documentation](https://dnsdist.org/index_TOC/)
* [Documentation technique de mon résolveur DoH (bortzmeyer)](https://www.bortzmeyer.org/doh-mon-resolveur/)
* [Une zone locale home.arpa signée avec DNSSEC](https://www.shaftinc.fr/home-arpa-dnssec.html)

#### Prérequis

    sudo apt install gnupg git

Cette section s'attarde sur l'installation et la configuration d'un frontal
DoT/DoH qui transmettra les requêtes DNS à un résolveur local écoutant sur le
port 53 de l'interface local `127.0.0.1` (IPv4).

Ici `dnsdist` se trouve être une très bonne solution pour remplir ce rôle.

#### Installation version courante

depuis juin 2023 pour debian bookworm

    sudo apt install dnsdist

#### Installer la dernière version dnsdist

Erreur depuis juin 2023 pour debian bookworm 

W: An error occurred during the signature verification. The repository is not updated and the previous index files will be used. GPG error: http://repo.powerdns.com/debian bookworm-dnsdist-16 InRelease: The following signatures were invalid: EXPKEYSIG 1B0C6205FD380FBB PowerDNS Release Signing Key <powerdns.support@powerdns.com>
W: Failed to fetch http://repo.powerdns.com/debian/dists/bookworm-dnsdist-16/InRelease  The following signatures were invalid: EXPKEYSIG 1B0C6205FD380FBB PowerDNS Release Signing Key <powerdns.support@powerdns.com>
W: Some index files failed to download. They have been ignored, or old ones used instead.
{: .prompt-danger }


Si vous désiriez utiliser la dernière version de dnsdist,
[PowerDNS](https://www.powerdns.com) qui développpe le logiciel propose et
maintient [ses propres dépôts pour Debian, Raspbian, Ubuntu et
CentOS](https://repo.powerdns.com/).

Ainsi, voici comment installer la dernière version de dnsdist, actuellement
`1.6.0`, tel que défini sur [le site de PowerDNS](https://repo.powerdns.com/)  
Exécuter ce qui suit 

```
# récupération du nom de la distribution et de son ID
# ATTENTION: le fichier /etc/os-release peut être différent selon la distribution
export D_NAME=$(. /etc/os-release; echo ${VERSION_CODENAME})
export D_ID=$(. /etc/os-release; echo ${ID})

# ajout du dépôt aux sources
# Debian / Ubuntu
echo "deb [signed-by=/etc/apt/keyrings/dnsdist-master-pub.asc arch=amd64] http://repo.powerdns.com/debian bullseye-dnsdist-master main" | sudo tee /etc/apt/sources.list.d/pdns.list

# gestion de l'étiquette et de la priorité du nouveau dépôt
sudo tee /etc/apt/preferences.d/dnsdist-master << EOF
Package: dnsdist*
Pin: origin repo.powerdns.com
Pin-Priority: 600
EOF

# Exécuter
sudo install -d /etc/apt/keyrings; curl https://repo.powerdns.com/CBC8B383-pub.asc | sudo tee /etc/apt/keyrings/dnsdist-master-pub.asc &&
sudo apt-get update &&
sudo apt-get install dnsdist
```

#### Configurer dnsdist

**Créer le fichier de configuration IPV4** `/etc/dnsdist/dnsdist.conf` pour une utilisation avec les certificats qui seront générés par **acme** dans le dossier `/etc/dnsdist` 

```
-- le résolveur DoT/DoH est public, on accepte tout le monde en IPv4
setACL({'0.0.0.0/0'})

-- serveur DNS où transférer les requêtes entrantes
newServer({address='127.0.0.1:53', name='Unbound (local)'})

-- configuration de DoT
-- modifier l'IPv4 en conséquence
addTLSLocal('185.112.146.46:853', '/etc/dnsdist/dns-server.crt', '/etc/dnsdist/dns-server.key', {minTLSVersion='tls1.2'})

-- configuration de DoH
-- modifier l'IPv4 en conséquence
addDOHLocal('185.112.146.46:443', '/etc/dnsdist/dns-server.crt', '/etc/dnsdist/dns-server.key', '/', {minTLSVersion='tls1.2'})

-- limitation du trafic à 100 requêtes par seconde
-- plus d'info en <https://dnsdist.org/advanced/qpslimits/>
addAction(MaxQPSIPRule(100), DropAction())

-- disable security status polling via DNS
setSecurityPollSuffix("")
```

L'ensemble des options de configuration est détaillé [sur le site de dnsdist](https://dnsdist.org/index/).

L'option `setSecurityPollSuffix("")` a été ajoutée pour contourner l'erreur suivante :  
`Error while retrieving the security update for version dnsdist-1.6.1: Unable to get a valid Security Status update`{: .prompt-danger }  

Explication : Le paquet Debian pour dnsdist désactive l'interrogation de sécurité en lui attribuant une chaîne vide dans le fichier de configuration par défaut. Nous devons faire de même pour le fichier dnsdist.conf afin que l'interrogation de sécurité soit désactivée et que le journal systemd de dnsdist reste propre.
{: .prompt-info }

Vérifier que le fichier de configuration est valide avec la
commande :

```
sudo dnsdist --check-config
# Configuration '/etc/dnsdist/dnsdist.conf' OK!
```

dnsdist est donc configuré pour écouter sur l'interface publique de la machine
sur les ports 443 (DoH) et 853 (DoT). Toutes les requêtes sont ensuite relayées
à un serveur écoutant le port 53 sur l'interface locale.

Pour l'instant les fichiers `dot-server.crt`, `dot-server.key`,
`doh-server.crt` et `doh-server.key` n'existent pas. Il est nécessaire de les
générer. Cela peut se faire avec `openssl` dans le cas de certificats
auto-signés. Il est aussi possible de passer par une autorité de certification,
ceci est le but de la prochaine partie.

>Remarque: attention aux droits des fichiers du certificat et de la clé, l'utilisateur ou le groupe `dnsdist` (parfois `_dnsdist`, vérifier le contenu du fichier `/etc/group` avec la commande

        grep dnsdist /etc/group

`_dnsdist:x:118:`

Avec le protocole de création des certificats **Acme**

 Les droits en écriture à l'utilisateur sur le dossier `/etc/dnsdist`

    sudo chown $USER -R /etc/dnsdist

### Gestion des certificats

![LetsEncrypt.png](LetsEncrypt.png){:width="150".normal}   
Afin de pouvoir proposer une connexion sécurisée au résolveur, il est
nécessaire de posséder au moins un certificat par service. On peut générer ses propres certificats auto-signés. Cependant pour
augmenter la confiance dans le service proposé, il est intéressant de s'appuyer
sur un tiers de confiance, l'[autorité de
certification](https://fr.wikipedia.org/wiki/Autorit%C3%A9_de_certification).
Nous allons utiliser [Let's Encrypt](https://letsencrypt.org/) comme autorité
de certification.

Installation client acme.sh

```
cd ~
sudo apt install socat -y # prérequis
git clone https://github.com/acmesh-official/acme.sh.git
cd acme.sh
./acme.sh --install 
```

Fermer le terminal, puis le réouvrir et vérifier

    acme.sh -h

Création des certificats

    export OVH_AK="xxxxxxxxxxxxxx"
    export OVH_AS="yyyyyyyyyyyyyyyyyyyyyyyy"

Génération des certificats dans le dossier `/etc/dndist` avec une chaîne préférentielle pour android

    acme.sh --dns dns_ovh --server letsencrypt --issue --keylength ec-384 -d 'doh.iceyan.xyz' -d 'dot.iceyan.xyz' -d 'searx.iceyan.xyz' -d 'wg.iceyan.xyz' --preferred-chain 'ISRG Root X1' --key-file /etc/dnsdist/dns-server.key --fullchain-file /etc/dnsdist/dns-server.crt

```
[Fri Feb 16 07:58:49 AM GMT 2024] Your cert is in: /home/iceyan/.acme.sh/doh.iceyan.xyz_ecc/doh.iceyan.xyz.cer
[Fri Feb 16 07:58:49 AM GMT 2024] Your cert key is in: /home/iceyan/.acme.sh/doh.iceyan.xyz_ecc/doh.iceyan.xyz.key
[Fri Feb 16 07:58:49 AM GMT 2024] The intermediate CA cert is in: /home/iceyan/.acme.sh/doh.iceyan.xyz_ecc/ca.cer
[Fri Feb 16 07:58:49 AM GMT 2024] And the full chain certs is there: /home/iceyan/.acme.sh/doh.iceyan.xyz_ecc/fullchain.cer
[Fri Feb 16 07:58:49 AM GMT 2024] Installing key to: /etc/dnsdist/dns-server.key
[Fri Feb 16 07:58:49 AM GMT 2024] Installing full chain to: /etc/dnsdist/dns-server.crt
```

Relancer manuellement le service dnsdist

    sudo systemctl restart dnsdist

Status

    systemctl status dnsdist

![](iceyan.xyz-dnsdist.png)

**Configurer la notification par email de Let's Encrypt lorsqu'un certificat est ignoré, renouvelé ou erroné**  

On utilise la méthode SMTP pour obtenir une notification par email  

pour un serveur smtp local postfix

```
cat >> $HOME/.bashrc << EOF
export SMTP_FROM="iceyan@iceyan.xyz"
export SMTP_TO="vpn@cinay.eu"
export SMTP_HOST="127.0.0.1" # même serveur sans authentification
export SMTP_SECURE="none"
export SMTP_BIN="/usr/bin/python3"
export SMTP_TIMEOUT="30"
EOF
source $HOME/.bashrc
```

Enfin, exécutez la commande suivante pour activer la notification smtp pour votre Let's Encrypt lorsqu'un certificat est ignoré, renouvelé ou erroné. 

    acme.sh --set-notify --notify-hook smtp

Vous obtiendrez des informations d'erreur ou de succès à l'écran comme suit :

![](iceyan.xyz-smtp.png)

Et voici l'email de test :

![](iceyan.xyz-smtp-a.png)

Ajouter les commandes `--set-notify --notify-hook smtp` dans le script de renouvellement des certificats

### Renouvellement certificats

![LetsEncrypt.png](LetsEncrypt.png){:width="150".normal}   

`Lors du renouvellement "--reloadcmd 'sudo systemctl restart dnsdist'" n'est pas exécuté`{: .prompt-danger }


Pour palier au problème création d'un script `~/certificat_expire.sh` qui exécute toutes les commandes , y compris celles du `--reloadcmd`, lors du renouvellement

```shell
#!/usr/bin/sh

# Domaine
_domain="iceyan.xyz"
# Test expiration certificats
PEM="/etc/dnsdist/dns-server.crt"
# 2 jours en secondes 
DAYS="172800" 
# OpenSSL
_openssl="/usr/bin/openssl"
# Date expiration dans 2 jours ?
$_openssl x509 -enddate -noout -in "$PEM"  -checkend "$DAYS" | grep -q 'Certificate will expire'

if [ $? -eq 0 -o ! -z "$1" ] 
then
  # certificat expire dans 2 jours , on renouvelle
  echo "Certificat expire dans 2 jours, on force le renouvellement"
  "$HOME/.acme.sh"/acme.sh --force --cron --home "$HOME/.acme.sh" --renew-hook "$HOME/.acme.sh/acme.sh --ecc --install-cert -d 'doh.iceyan.xyz' -d 'dot.iceyan.xyz' -d 'searx.iceyan.xyz' -d 'wg.iceyan.xyz' --preferred-chain 'ISRG Root X1' --key-file /etc/dnsdist/dns-server.key --fullchain-file /etc/dnsdist/dns-server.crt --set-notify --notify-hook smtp"
  #	Les autres commandes à exécuter
  echo "Recharge service dnsdist"
  sudo systemctl restart dnsdist
else
  echo "Certificat $PEM valide"
  $_openssl x509 -enddate -noout -in "$PEM" | cut -d '=' -f 2
fi
```

Rendre le script exécutable

    chmod +x $HOME/certificat_expire.sh

Modifier la tâche cron

	crontab -e

```
4 12 * * * "$HOME/certificat_expire.sh" &>/dev/null
```

### Test résolveur DNS depuis un client (homer)

Pour s'assurer que le résolveur fonctionne de manière nominal, on va utiliser Homer.

Homer est un outil développé par l'Afnic, qui permet de tester et débugger un
résolveur DoT/DoH en ligne de commande.

Homer est un logiciel libre et le code est disponible [sur la forge logicielle
Framagit](https://framagit.org/bortzmeyer/homer).

L'outil nécessite python3, et certains modules associés :

```
sudo apt-get install python3 python3-pycurl python3-dnspython python3-openssl python3-netaddr  # debian
yay -S python-pycurl python-dnspython python-pyopenssl python-netaddr # archlinux
```

Il ne reste plus qu'à récupérer Homer

    git clone https://framagit.org/bortzmeyer/homer
    cd homer

et à le lancer 

    ./remoh.py https://doh.iceyan.xyz afnic.fr


```
id 0
opcode QUERY
rcode NOERROR
flags QR RD RA AD
edns 0
payload 1232
option ECS 0.0.0.0/0 scope/0
;QUESTION
afnic.fr. IN AAAA
;ANSWER
afnic.fr. 600 IN AAAA 2001:41d0:404:200::2df6
;AUTHORITY
;ADDITIONAL

Total elapsed time: 0.60 seconds
```

    ./remoh.py --dot dot.iceyan.xyz framagit.org

```
id 17695
opcode QUERY
rcode NOERROR
flags QR RD RA AD
edns 0
payload 1232
option ECS 0.0.0.0/0 scope/0
;QUESTION
framagit.org. IN AAAA
;ANSWER
framagit.org. 3600 IN AAAA 2a01:4f8:231:4c99::75
;AUTHORITY
;ADDITIONAL

Total elapsed time: 0.73 seconds
```

Homer peut aussi être utilisé pour tester la bonne configuration du résolveur :

```
$ ./remoh.py --check --dot dot.iceyan.xyz framasoft.org
OK

$ ./remoh.py --check https://doh.iceyan.xyz chatons.org
OK
```

Dans le cas où votre résolveur est mal configuré, Homer s'arrête est affiche
l'erreur rencontrée. Par exemple pour un certificat non configuré :

```
$ ./remoh.py --dot 198.51.100.19 framasoft.org
198.51.100.19: Certificate error: "198.51.100.19 is not in the certificate
Could not connect to "198.51.100.19"
```

On peut demander à Homer de ne pas vérifier le certificat
avec l'option `-k | --insecure` :

```
$ ./remoh.py --insecure --dot 198.51.100.19 framasoft.org
id 35430
opcode QUERY
rcode NOERROR
flags QR RD RA
edns 0
payload 4096
option ECS ::/0 scope/0
;QUESTION
framasoft.org. IN AAAA
;ANSWER
framasoft.org. 3600 IN AAAA 2a01:4f8:141:3421::212
;AUTHORITY
;ADDITIONAL

Total elapsed time: 0.04 seconds (41.83 ms/request)
```

## Vpn Wireguard

![wireguard](wireguard_icon.png){:width="50" .normal}  
*WireGuard est un serveur VPN à code source ouvert, gratuit, moderne et rapide, doté d'un chiffrement de pointe. Il est plus rapide et plus simple que l'IPSec et l'OpenVPN*  

* Les procédures pour une installation complète : [Serveur vpn wireguard](/posts/Ubound_Wireguard_Wg-Web/)

## BorgBackup

*Sauvegarde serveur sur une boîte de stockage*

* [BorgBackup VPS](/posts/BorgBackup_entre_serveurs/#borgbackup-vps)

Le fichier /root/.borg/borg-backup.sh

```
export BORG_RSH='ssh -i /root/.ssh/id_borg_ed25519'
export BORG_PASSPHRASE=$(cat /root/.borg/iceyan_xyz.passphrase)
BACKUP_DATE=`date +%Y-%m-%d-%Hh%M`
BORG_REPOSITORY=ssh://u326239@u326239.your-storagebox.de:23/./backup/borg/iceyan.xyz
borg create -v --progress --stats --exclude-from /root/.borg/exclusions-borg.txt ${BORG_REPOSITORY}::${BACKUP_DATE} /
borg prune -v --list --stats --keep-daily=7 --keep-weekly=4 --keep-monthly=6 $BORG_REPOSITORY
```

Ajouter ce qui suit à `/root/.borg/borg-backup.sh` pour envoi message via ntfy + email

```
# Message
curl \
-H "X-Email: ntfy@cinay.eu" \
-H "Title: VPS Islande iceyan.xyz" \
-H "Authorization: Bearer tk_jkrtwpoaqvgz2ic4nalzcfd447l5fzr" \
-H prio:low \
-H tags:information_source \
-d "Fin sauvegarde borgbackup `date +%d/%m/%Y-%Hh%M`" \
https://noti.rnmkcy.eu/yan_infos
```

`Modifier la valeur Bearer tk_...`

## Métamoteur (INACTIF)

### SearXNG

![SearXNG](searxng-logo.png){:width="150" .normal}  

*[SearXNG](https://docs.searxng.org/) est un métamoteur qui recherche ses informations à travers plusieurs moteurs de recherche généralistes*

Suivre en totalité la procédure d'installation : [SearXNG (métamoteur de recherche libre)](/posts/SearXNG-Metamoteur-Recherche-Libre/)

### nginx-light

**serveur web/proxy nginx (version de base)**  
Nginx (« engine X ») est un serveur web et mandataire inverse de haute performance créé par Igor Sysoev. Il peut être utilisé en tant que serveur web autonome ou comme mandataire pour réduire la charge des serveurs dorsaux HTTP ou de courrier électronique.

* Ce métapaquet fournit une version très légère de nginx avec seulement l'ensemble minimal de fonctionnalités et de modules.
* Modules HTTP standards : Core, Access, Auth Basic, Auto Index, Empty GIF, FastCGI, Map, Proxy, Rewrite, SCGI, uWSGI.
* Modules HTTP additionnels : Auth Request, Charset, Gzip, Gzip Precompression, Headers, HTTP/2, Index, Log, Real IP, Slice, SSI, SSL, Stub Status, Thread Pool, WebDAV, Upstream.
* Modules tiers : Echo. 

Installer nginx-light

    sudo apt install nginx-light

Supprimer le vhost par défaut

    sudo rm /etc/nginx/sites-enabled/default

### searx.iceyan.xyz

Le proxy pour le métamoteur searxng

    sudo nano /etc/nginx/sites-enabled/searxng

```
server {
    listen 4443 ssl http2;
    server_name searx.iceyan.xyz;
    ssl_certificate /etc/dnsdist/dns-server.crt;
    ssl_certificate_key /etc/dnsdist/dns-server.key;


    location / {
        include uwsgi_params;
        uwsgi_pass unix:////usr/local/searxng/run/socket;
    }
}
```

Recharger nginx

    sudo systemctl reload nginx

Accès au moteur de recherche : <https://searx.iceyan.xyz:4443/>  
![](/images/searx.iceyan.xyz.png)

Lien paramètrage métamoteur dans un navigateur : https://searx.iceyan.xyz:4443/search?q=%s

### Instructions pour désactiver 

On arrête et désactive l’application uwsgi

    sudo systemctl stop uwsgi.service
    sudo systemctl disable uwsgi.service

On arrête et désactive nginx

    sudo systemctl stop nginx
    sudo systemctl disable nginx

Efface la règle parefeu "4443/tcp"

```bash
sudo ufw status numbered  # exemple effacer règles 12 et 6
sudo ufw delete 12        # y
sudo ufw delete 6         # y
```

Après effacement

```bash
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 55046/tcp                  ALLOW IN    Anywhere                  
[ 2] 80/tcp                     ALLOW IN    Anywhere                  
[ 3] 443/tcp                    ALLOW IN    Anywhere                  
[ 4] DNS                        ALLOW IN    Anywhere                  
[ 5] 853                        ALLOW IN    Anywhere                  
```

## Maintenance

### Mises à jour automatiques sur debian

[Mises à jour automatiques sur Debian](/posts/Debian_unattended-upgrades_mise_a_jour_auto/)

