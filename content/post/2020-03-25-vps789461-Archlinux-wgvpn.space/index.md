+++
title = 'vps789461 (wgvpn.space) Archlinux 64bits (INACTIF)'
date = 2020-03-25 00:00:00 +0100
categories = vps
+++
*OVH vps789461 (1 vCore/2GoRam/20GoSSD) Debian Buster*

![archlinux](archlinux-logo-001.png){:width="300"}

# Serveur VPS OVH 

![OVH](OVH-320px-Logo.png){:width="50"}  
Arch Linux (en version 64 bits)


PARAMETRES D'ACCES:
L'adresse IPv4 du VPS est : 51.77.151.245
L'adresse IPv6 du VPS est : 2001:41d0:0404:0200:0000:0000:0000:01cf

Le nom du VPS est : vps789461.ovh.net

Le compte administrateur suivant a été configuré sur le VPS :
Nom d'utilisateur : root
Mot de passe :      pLS9xuhn


    ssh root@51.77.151.245


### Réseau systemd-networkd

La configuration [systemd-networkd](https://wiki.archlinux.org/index.php/Systemd-networkd)

    cat /etc/systemd/network/eth0.network 

```
[Match]
Name=eth0
[Network]
DHCP=ipv4
```


Vérifier le réseau `ip a` 

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether fa:16:3e:4a:27:c9 brd ff:ff:ff:ff:ff:ff
    inet 51.77.151.245/32 scope global dynamic eth0
       valid_lft 84602sec preferred_lft 84602sec
    inet6 fe80::f816:3eff:fe4a:27c9/64 scope link 
       valid_lft forever preferred_lft forever
```

### domaine wgvpn.space

Zone dns OVH

```
$TTL 3600
@	IN SOA dns20.ovh.net. tech.ovh.net. (2020022809 86400 3600 3600000 300)
        IN NS     ns20.ovh.net.
        IN NS     dns20.ovh.net.
        IN A      51.77.151.245
        IN AAAA   2001:41d0:404:200::1cf
```

### utilisateur arch

Utilisateur **arch** existant, changer mot de passe  

    passwd arch 

Visudo pour les accès root via utilisateur **arch**  

```bash
echo "arch     ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
```

Changer le mot de passe root

    passwd root

Les outils à installer

    pacman -S base-devel 

localisation française, le fichier /etc/locale.conf doit contenir la bonne valeur pour LANG  

	nano /etc/locale.conf

Ajouter  

```
LANG=fr_FR.UTF-8
LC_COLLATE=C
```

Il faut supprimer le **#** au début de la ligne fr_FR.UTF-8 UTF-8 dans le fichier **/etc/locale.gen**  

	nano /etc/locale.gen

```
fr_FR.UTF-8 UTF-8
```

puis exécuter:  

	locale-gen

spécifier la locale pour la session courante  

	export LANG=fr_FR.UTF-8

fuseau horaire de Paris  

	ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime

### OpenSSH, clé et script

![OpenSSH](ssh_logo1.png){:width="100"}

**connexion avec clé**  

<u>sur l'ordinateur de bureau</u>  
Générer une paire de clé curve25519-sha256 (ECDH avec Curve25519 et SHA2) nommé **kvm-cinay** pour une liaison SSH avec le serveur KVM.  

    ssh-keygen -t ed25519 -o -a 100 -f ~/.ssh/kvm-vps789461

Envoyer la clé publique sur le serveur KVM   

    scp ~/.ssh/kvm-vps789461.pub arch@51.77.151.245:/home/arch/

<u>sur le serveur KVM</u>  
On se connecte  

    ssh arch@51.77.151.245

Copier le contenu de la clé publique dans /home/$USER/.ssh/authorized_keys  

    cd ~

Sur le KVM ,créer un dossier .ssh  

```bash
mkdir .ssh
cat $HOME/kvm-vps789461.pub >> $HOME/.ssh/authorized_keys
```

et donner les droits  

    chmod 600 $HOME/.ssh/authorized_keys

effacer le fichier de la clé  

    rm $HOME/kvm-vps789461.pub

Modifier la configuration serveur SSH  

    sudo nano /etc/ssh/sshd_config

Modifier

```conf
Port 55039
PermitRootLogin no
PasswordAuthentication no
```

Relancer openSSH  

    sudo systemctl restart sshd

Accès depuis le poste distant avec la clé privée  

    ssh -p 55039 -i ~/.ssh/kvm-vps789461 arch@51.77.151.245

### Installer utilitaires  

    sudo pacman -S rsync curl tmux jq figlet git

### Hostname

    sudo hostnamectl set-hostname wgvpn.space

### Scripts motd et ssh_rc_bash

Motd

    sudo nano /etc/motd

```
                 ____  ___  ___  _ _    __  _               
 __ __ _ __  ___|__  |( _ )/ _ \| | |  / / / |              
 \ V /| '_ \(_-<  / / / _ \\_, /|_  _|/ _ \| |              
  \_/ | .__//__/ /_/  \___/ /_/   |_| \___/|_|              
 __ __|_| __ _ __ __ _ __  _ _      ___ _ __  __ _  __  ___ 
 \ V  V // _` |\ V /| '_ \| ' \  _ (_-<| '_ \/ _` |/ _|/ -_)
  \_/\_/ \__, | \_/ | .__/|_||_|(_)/__/| .__/\__,_|\__|\___|
         |___/      |_|                |_|                  
Archlinux 64
```


Script **ssh_rc_bash**  
>**ATTENTION!!! Les scripts sur connexion peuvent poser des problèmes pour des appels externes autres que ssh**

    nano ssh_rc_bash

```
#!/bin/bash

get_infos() {
    seconds="$(< /proc/uptime)"
    seconds="${seconds/.*}"
    days="$((seconds / 60 / 60 / 24)) jour(s)"
    hours="$((seconds / 60 / 60 % 24)) heure(s)"
    mins="$((seconds / 60 % 60)) minute(s)"
    
    # Remove plural if < 2.
    ((${days/ *} == 1))  && days="${days/s}"
    ((${hours/ *} == 1)) && hours="${hours/s}"
    ((${mins/ *} == 1))  && mins="${mins/s}"
    
    # Hide empty fields.
    ((${days/ *} == 0))  && unset days
    ((${hours/ *} == 0)) && unset hours
    ((${mins/ *} == 0))  && unset mins
    
    uptime="${days:+$days, }${hours:+$hours, }${mins}"
    uptime="${uptime%', '}"
    uptime="${uptime:-${seconds} seconds}"

   if [[ -f "/sys/devices/virtual/dmi/id/board_vendor" ||
                    -f "/sys/devices/virtual/dmi/id/board_name" ]]; then
	model="$(< /sys/devices/virtual/dmi/id/board_vendor)"
	model+=" $(< /sys/devices/virtual/dmi/id/board_name)"
   fi

   if [[ -f "/sys/devices/virtual/dmi/id/bios_vendor" ||
                    -f "/sys/devices/virtual/dmi/id/bios_version" ]]; then
        bios="$(< /sys/devices/virtual/dmi/id/bios_vendor)"
        bios+=" $(< /sys/devices/virtual/dmi/id/bios_version)"
        bios+=" $(< /sys/devices/virtual/dmi/id/bios_date)"
   fi
}

#clear
PROCCOUNT=`ps -Afl | wc -l`  		# nombre de lignes
PROCCOUNT=`expr $PROCCOUNT - 5`		# on ote les non concernées
GROUPZ=`users`
ipinfo=$(curl -s ipinfo.io) 		# info localisation format json
#ipinfo=$(curl -s iplocality.com) 		# info localisation format json
publicip=$(echo $ipinfo | jq -r '.ip')  # extraction des données , installer préalablement "jq"
ville=$(echo $ipinfo | jq -r '.city')
pays=$(echo $ipinfo | jq -r '.country')
cpuname=`cat /proc/cpuinfo |grep 'model name' | cut -d: -f2 | sed -n 1p`
iplink=`ip link show |grep -m 1 "2:" | awk '{print $2}' | cut -d: -f1`

if [[ $GROUPZ == *irc* ]]; then
ENDSESSION=`cat /etc/security/limits.conf | grep "@irc" | grep maxlogins | awk {'print $4'}`
PRIVLAGED="IRC Account"
else
ENDSESSION="Unlimited"
PRIVLAGED="Regular User"
fi
get_infos
logo=$(figlet "`hostname --fqdn`")
#meteo=$(curl fr.wttr.in/$ville?0)
sdx=$(df -h |grep "/dev/sd") # les montages /dev/sd
distri=$(lsb_release -sd)
distri+=" $(uname -m)"

echo -e "
\e[1;31m$logo
\e[1;35m   \e[1;37mHostname \e[1;35m= \e[1;32m`hostname`
\e[1;35m \e[1;37mWired IpV4 \e[1;35m= \e[1;32m`ip addr show $iplink | grep 'inet\b' | awk '{print $2}' | cut -d/ -f1`
\e[1;35m \e[1;37mWired IpV6 \e[1;35m= \e[1;32m`ip addr show $iplink | grep -E 'inet6' |grep -E 'scope link' | awk '{print $2}' | cut -d/ -f1`
\e[1;35m     \e[1;37mKernel \e[1;35m= \e[1;32m`uname -r`
\e[1;35m    \e[1;37mDistrib \e[1;35m= \e[1;32m$distri
\e[1;35m     \e[1;37mUptime \e[1;35m= \e[1;32m`echo $uptime`
\e[1;35m       \e[1;37mBios \e[1;35m= \e[1;32m`echo $bios`
\e[1;35m      \e[1;37mBoard \e[1;35m= \e[1;32m`echo $model`
\e[1;35m        \e[1;37mCPU \e[1;35m= \e[1;32m`echo $cpuname`
\e[1;35m \e[1;37mMemory Use \e[1;35m= \e[1;32m`free -m | awk 'NR==2{printf "%s/%sMB (%.2f%%)\n", $3,$2,$3*100/$2 }'`
\e[1;35m   \e[1;37mUsername \e[1;35m= \e[1;32m`whoami`
\e[1;35m   \e[1;37mSessions \e[1;35m= \e[1;32m`who | grep $USER | wc -l`
\e[1;35m\e[1;37mPublic IpV4 \e[1;35m= \e[1;32m`echo $publicip`
\e[1;35m\e[1;37mPublic IpV6 \e[1;35m= \e[1;32m`ip addr show $iplink | grep -m 1 'inet6\b'  | awk '{print $2}' | cut -d/ -f1`
\e[1;35m\e[1;33m$sdx
\e[1;0m
"
```

    chmod +x ssh_rc_bash # rendre le bash exécutable
    ./ssh_rc_bash        # exécution

## Certificats letsencrypt - domaine wgvpn.space

![letsencrypt](letsencrypt-logo1.png){:width="100"}  
Installation gestionnaire des certificats Let's Encrypt

    cd ~
    sudo pacman -S socat # prérequis


```
git clone https://github.com/Neilpang/acme.sh.git
cd acme.sh
./acme.sh --install 
```

>Se déconnecter puis se reconnecter pour la prise en compte

Se connecter sur l'api OVH pour les paramètres (clé et secret)

    export OVH_AK="votre application key"
    export OVH_AS="votre application secret"

Premier lancement pour la génération des certificats

    acme.sh --dns dns_ovh --issue --keylength ec-384 -d wgvpn.space

Rechercher le lien qui suit le texte suivant **Please open this link to do authentication:**  
Se connecter sur le lien , s'authentifier puis sélectionner "unlimited" et valider.Le message suivant s'affiche 

    OVH authentication Success ! 

Lancer une seconde fois la génération des certificats et patienter quelques minutes...

    acme.sh --dns dns_ovh --issue  --keylength ec-384 -d wgvpn.space 

Les certificats sont disponibles

```
[Wed Mar 25 22:40:04 CET 2020] Your cert is in  /home/arch/.acme.sh/wgvpn.space_ecc/wgvpn.space.cer 
[Wed Mar 25 22:40:04 CET 2020] Your cert key is in  /home/arch/.acme.sh/wgvpn.space_ecc/wgvpn.space.key 
[Wed Mar 25 22:40:04 CET 2020] The intermediate CA cert is in  /home/arch/.acme.sh/wgvpn.space_ecc/ca.cer 
[Wed Mar 25 22:40:04 CET 2020] And the full chain certs is there:  /home/arch/.acme.sh/wgvpn.space_ecc/fullchain.cer 
```

Créer des liens avec **/etc/ssl/private/**

```
# domaine wgvpn.space
sudo ln -s /home/arch//.acme.sh/wgvpn.space_ecc/fullchain.cer /etc/ssl/private/wgvpn.space-fullchain.pem   # full chain certs
sudo ln -s /home/arch//.acme.sh/wgvpn.space_ecc/wgvpn.space.key /etc/ssl/private/wgvpn.space-key.pem     # cert key
sudo ln -s /home/arch//.acme.sh/wgvpn.space_ecc/wgvpn.space.cer /etc/ssl/private/wgvpn.space-chain.pem   # cert domain
sudo ln -s /home/arch//.acme.sh/wgvpn.space_ecc/ca.cer /etc/ssl/private/wgvpn.space-ca.pem                 # intermediate CA cert
```

## Nginx

Installer nginx-mainline

    sudo -s
    pacman -S nginx-mainline

la configuration

    nano /etc/nginx/nginx.conf

```
user http;
worker_processes auto;
worker_cpu_affinity auto;

events {
    multi_accept on;
    worker_connections 1024;
}

http {
    charset utf-8;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    server_tokens off;
    log_not_found off;
    types_hash_max_size 4096;
    client_max_body_size 16M;

    # MIME
    include mime.types;
    default_type application/octet-stream;

    # logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log warn;

    # load configs
    include /etc/nginx/conf.d/*.conf;
    #include /etc/nginx/sites-enabled/*;
}
```

Créer le dossier de conf

    mkdir -p /etc/nginx/conf.d/

Vérifier , lancer et activer nginx

    nginx -t
    systemctl start nginx
    systemctl enable nginx

### Configurer SSL,TLS  

* ssl (tls1.2 tls1.3) , Headers   
* Diffie-Hellman : `openssl dhparam -out /etc/ssl/private/dh2048.pem -outform PEM -2 2048`  

Regroupement dans un fichier  **/etc/nginx/ssl_dh_header**

```
    ##
    # SSL Settings
    ##
    ssl_certificate /etc/ssl/private/wgvpn.space-fullchain.pem;
    ssl_certificate_key /etc/ssl/private/wgvpn.space-key.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    ssl_dhparam /etc/ssl/private/dh2048.pem;

    # intermediate configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    add_header Strict-Transport-Security "max-age=31536000";

```

Le fichier nginx wgvpn.space

    sudo nano /etc/nginx/conf.d/wgvpn.space.conf

```
server {
    listen 443 ssl;
    listen [::]:443 ssl;
	server_name wgvpn.space;

	access_log /var/log/nginx/wgvpn.space.access.log;
	error_log /var/log/nginx/wgvpn.space.error.log;

    # SSL
    include ssl_dh_header;

    index index.html;
    
	location / {
		root /srv/archlinux/;
	}

}
```

Un fichier index.html

    echo "<html>Archlinux Repository wgvpn.space</html>" > /srv/archlinux/index.html

Activer le vhost

    nginx -t # vérifier
    systemctl reload nginx # recharger





## Miroir des paquets ArchLinux

Choisir le miroir ArchLinux le plus rapide  
Installer pacman-contrib et pacman-mirrorlist (rankmirrors) 

    sudo pacman -S pacman-contrib pacman-mirrorlist

Utiliser rankmirrors pour tester le miroir le plus proche de chez vous. La commande ci-dessous permet de tester les miroirs en Belgique, France, Hollande et Allemagne (les 5 plus rapide) 

    curl -s "https://www.archlinux.org/mirrorlist/?country=FR&country=NL&country=BE&country=GE&protocol=https&use_mirror_status=on" | sed -e 's/^#Server/Server/' -e '/^#/d' | rankmirrors -n 5 -

```
Server = https://mirrors.eric.ovh/arch/$repo/os/$arch
Server = https://mirror.cyberbits.eu/archlinux/$repo/os/$arch
Server = https://mirror.wormhole.eu/archlinux/$repo/os/$arch
Server = https://archlinux.mirror.liteserver.nl/$repo/os/$arch
Server = https://archlinux.mailtunnel.eu/$repo/os/$arch
```

Vérifier la compatibilité avec rsync sur le site <www.archlinux.org/mirrors>.  

Script de synchro **repoarchsynchro**

    nano repoarchsynchro

```
#!/bin/bash
#
########
#
# Copyright © 2014-2019 Florian Pritz <bluewind@xinu.at>
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, see <http://www.gnu.org/licenses/>.
#
########
#
# This is a simple mirroring script. To save bandwidth it first checks a
# timestamp via HTTP and only runs rsync when the timestamp differs from the
# local copy. As of 2016, a single rsync run without changes transfers roughly
# 6MiB of data which adds up to roughly 250GiB of traffic per month when rsync
# is run every minute. Performing a simple check via HTTP first can thus save a
# lot of traffic.

# Directory where the repo is stored locally. Example: /srv/repo
target="/srv/archlinux/repo"

# Directory where files are downloaded to before being moved in place.
# This should be on the same filesystem as $target, but not a subdirectory of $target.
# Example: /srv/tmp
tmp="/srv/archlinux/tmp"

# Lockfile path
lock="/srv/archlinux/syncrepo.lck"

# If you want to limit the bandwidth used by rsync set this.
# Use 0 to disable the limit.
# The default unit is KiB (see man rsync /--bwlimit for more)
bwlimit=0

# The source URL of the mirror you want to sync from.
# If you are a tier 1 mirror use rsync.archlinux.org, for example like this:
# rsync://rsync.archlinux.org/ftp_tier1
# Otherwise chose a tier 1 mirror from this list and use its rsync URL:
# https://www.archlinux.org/mirrors/
source_url='rsync://archlinux.mailtunnel.eu/archlinux/'

# An HTTP(S) URL pointing to the 'lastupdate' file on your chosen mirror.
# If you are a tier 1 mirror use: http://rsync.archlinux.org/lastupdate
# Otherwise use the HTTP(S) URL from your chosen mirror.
lastupdate_url='https://archlinux.mailtunnel.eu/lastupdate'


#### END CONFIG

[ ! -d "${target}" ] && mkdir -p "${target}"
[ ! -d "${tmp}" ] && mkdir -p "${tmp}"

exec 9>"${lock}"
flock -n 9 || exit

rsync_cmd() {
        local -a cmd=(rsync -rtlH --safe-links --delete-after ${VERBOSE} "--timeout=600" "--contimeout=60" -p --delay-updates --no-motd "--temp-dir=${tmp}")

        if stty &>/dev/null; then
                cmd+=(-h -v --progress)
        else
                cmd+=(--quiet)
        fi

        if ((bwlimit>0)); then
                cmd+=("--bwlimit=$bwlimit")
        fi

        "${cmd[@]}" "$@"
}

# if we are called without a tty (cronjob) only run when there are changes
if ! tty -s && [[ -f "$target/lastupdate" ]] && diff -b <(curl -Ls "$lastupdate_url") "$target/lastupdate" >/dev/null; then
        # keep lastsync file in sync for statistics generated by the Arch Linux website
        rsync_cmd "$source_url/lastsync" "$target/lastsync"
        exit 0
fi

rsync_cmd --exclude='*.links.tar.gz*' --exclude='/other' --exclude='/sources' --exclude='/iso' "${source_url}" "${target}"

#echo "Last sync was $(date -d @$(cat ${target}/lastsync))"
```

Le rendre exécutable

    chmod +x /home/arch/repoarchsynchro

### Ajouter une tâche planifiée 

Cron se trouve dans le dépôt core, il s'agit du paquet cronie. 

    sudo pacman -S cronie
    sudo systemctl start cronie && sudo systemctl enable cronie

Ajout de la tâche

    sudo -s
    export EDITOR=/usr/bin/nano
    crontab -e

```    
# Synchro Dépôts archlinux en mirroir
00 04 * * * /home/arch/repoarchsynchro.sh
```

Ajouter votre miroir privé dans votre /etc/pacman.d/mirrorlist
Sur votre machine :

Modifiez votre /etc/pacman.d/mirrorlist et ajouter l’url vers votre NAS comme premier serveur dans la liste   

```
Server = http://nas:9080/$repo/os/$arch
#Server = http://192.168.1.2:9080/$repo/os/$arch ou utilisez directement l'ip du NAS si l'url ci-dessus ne fonctionne pas
Server = ...
```