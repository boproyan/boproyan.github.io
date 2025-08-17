+++
title = 'Résolveur DNS Unbound'
date = 2024-09-16 00:00:00 +0100
categories = ['dns']
+++
*Les serveurs DNS sont des machines discutant entre elles afin de se communiquer les correspondances entre nom de domaine et adresses IP.*

## Résolveur DNS Unbound

![DNS Unbound](unbound-250.png){:width="150" .left}  
*Les serveurs DNS sont des machines discutant entre elles afin de se communiquer les correspondances entre nom de domaine et adresses IP.*

* [Résolveur de nom de domaines Unbound](https://papy-tux.legtux.org/doc1150/index.php)
* [Comment configurer un résolveur DNS local avec Unbound sur Debian](https://fr.linux-console.net/?p=30723)
* [Serveur DNS récursif UNBOUND](https://www.akoonet.com/serveur-dns-recursif-unbound/)
* [Comprehensive Guide to Setting Up Unbound as a Local DNS Resolver](https://shape.host/resources/comprehensive-guide-setting-up-unbound-local-dns-resolver-ubuntu-2204)

Les quatre serveurs DNS qui chargent une page Web

1.    Le serveur DNS récursif (Resolving Name Server) : le serveur DNS récursif répond à une requête DNS et demande l’adresse à d’autres serveurs DNS, ou détient déjà un enregistrement de l’adresse IP du site.
*    Serveur racine du DNS (Root Name Server) : il s’agit du serveur de noms pour la zone racine. Il répond à des requêtes directes et peut renvoyer une liste de noms de serveurs faisant autorité pour le domaine de haut niveau correspondant.
*    Serveur DNS TLD : le serveur TLD (top-level domain : domaine de premier niveau) est l’un des serveurs DNS de haut niveau que l’on trouve sur Internet. Lorsque vous recherchez www.varonis.com, un serveur TLD répondra en premier pour le « .com », puis le DNS recherchera « varonis ».
*    Serveur de noms faisant autorité (Authoritative Name Server) : le serveur de noms faisant autorité constitue le terminus d’une requête DNS. Le serveur de noms faisant autorité contient l’enregistrement DNS répondant à la requête.

> Si vous utilisez un résolveur local (par exemple bind, dnsmasq, unbound, etc), ou tout autre logiciel générant un fichier **/etc/resolv.conf** (par exemple **resolvconf**), le service **systemd-resolved** ne doit pas être utilisé. Pour désactiver **systemd-resolved**, exécutez la commande suivante : `sudo systemctl disable systemd-resolved` Effacer puis recréer un fichier `/etc/resolv.conf` avec une directive dns, par exemple : `nameserver 1.1.1.1`
{: .prompt-warning }

### Prérequis

*À partir de la version 209, systemd contient un démon de configuration réseau nommé <u>systemd-networkd</u> qui peut être utilisé pour la configuration basique du réseau. De plus, depuis la version 213, la résolution de nom DNS peut être prise en charge par <u>systemd-resolved</u> au lieu d'un fichier /etc/resolv.conf statique. <u>Ces deux services sont activés par défaut</u>* 

Si vous utilisez un résolveur local (par exemple bind, dnsmasq, unbound, etc), ou tout autre logiciel générant un fichier **/etc/resolv.conf** (par exemple **resolvconf**), le service **systemd-resolved** ne doit pas être utilisé.  
Pour désactiver **systemd-resolved**, exécutez la commande suivante : `sudo systemctl disable systemd-resolved`  
Effacer puis recréer un fichier `/etc/resolv.conf` avec une directive dns, par exemple : `nameserver 1.1.1.1`
{: .prompt-warning }

### Installation

Passage en mode super utilisateur

    sudo -s 

Désinstaller bind si installé

    apt remove --purge bind* -y
    rm -r /var/cache/bind/

Installation des outils dns et du paquet Unbound :  

    apt install dnsutils unbound -y

### Configuration des adresses des serveurs de nom racine

Récupération de la liste des DNS racines

    curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache
    chown unbound:unbound /var/lib/unbound/root.hints

Indiquer l'adresse du fichier dans la configuration de unbound 

    nano /etc/unbound/unbound.conf.d/root-hints.conf

```
# Fichier des serveurs root à télécharger env tous les 6 mois :
# curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache
# 
server:
root-hints: "/var/lib/unbound/root.hints"
```

Vérifier le fichier

    unbound-checkconf /etc/unbound/unbound.conf.d/root-hints.conf 

*unbound-checkconf: no errors in /etc/unbound/unbound.conf.d/root-hints.conf*

### Configuration Unbound

Les fichiers de configuration par défaut sont situés sous `/etc/unbound/unbound.conf.d/`

```
remote-control.conf
root-hints.conf
root-auto-trust-anchor-file.conf
```

créer la configuration `cwwk-unbound.conf`

    /etc/unbound/unbound.conf.d/cwwk-unbound.conf

Configuration fonctionnelle

```
server:
    # ne rien enregistrer dans les journaux hormis les erreurs
    verbosity: 0

    # n'écouter que sur l'interface locale en IPv4
    # unbound nécessite d'être relancé si modifié
    interface: 192.168.0.205

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
    
    ###########################################################################
    # PERFORMANCE SETTINGS
    ###########################################################################
    # https://nlnetlabs.nl/documentation/unbound/howto-optimise/

    # Number of slabs in the infrastructure cache. Slabs reduce lock contention
    # by threads. Must be set to a power of 2.
    infra-cache-slabs: 4

    # Number of slabs in the key cache. Slabs reduce lock contention by
    # threads. Must be set to a power of 2. Setting (close) to the number
    # of cpus is a reasonable guess.
    key-cache-slabs: 4

    # Number  of  bytes  size  of  the  message  cache.
    # Unbound recommendation is to Use roughly twice as much rrset cache memory
    # as you use msg cache memory.
    msg-cache-size: 128525653

    # Number of slabs in the message cache. Slabs reduce lock contention by
    # threads. Must be set to a power of 2. Setting (close) to the number of
    # cpus is a reasonable guess.
    msg-cache-slabs: 4

    # The number of queries that every thread will service simultaneously. If
    # more queries arrive that need servicing, and no queries can be jostled
    # out (see jostle-timeout), then the queries are dropped.
    # This is best set at half the number of the outgoing-range.
    # This Unbound instance was compiled with libevent so it can efficiently
    # use more than 1024 file descriptors.
    num-queries-per-thread: 4096

    # The number of threads to create to serve clients.
    # This is set dynamically at run time to effectively use available CPUs
    # resources
    num-threads: 3

    # Number of ports to open. This number of file descriptors can be opened
    # per thread.
    # This Unbound instance was compiled with libevent so it can efficiently
    # use more than 1024 file descriptors.
    outgoing-range: 8192

    # Number of bytes size of the RRset cache.
    # Use roughly twice as much rrset cache memory as msg cache memory
    rrset-cache-size: 257051306

    # Number of slabs in the RRset cache. Slabs reduce lock contention by
    # threads. Must be set to a power of 2.
    rrset-cache-slabs: 4

    # Do no insert authority/additional sections into response messages when
    # those sections are not required. This reduces response size
    # significantly, and may avoid TCP fallback for some responses. This may
    # cause a slight speedup.
    minimal-responses: yes

    # # Fetch the DNSKEYs earlier in the validation process, when a DS record
    # is encountered. This lowers the latency of requests at the expense of
    # little more CPU usage.
    prefetch: yes

    # Fetch the DNSKEYs earlier in the validation process, when a DS record is
    # encountered. This lowers the latency of requests at the expense of little
    # more CPU usage.
    prefetch-key: yes

    # Have unbound attempt to serve old responses from cache with a TTL of 0 in
    # the response without waiting for the actual resolution to finish. The
    # actual resolution answer ends up in the cache later on.
    serve-expired: yes

    # Open dedicated listening sockets for incoming queries for each thread and
    # try to set the SO_REUSEPORT socket option on each socket. May distribute
    # incoming queries to threads more evenly.
    so-reuseport: yes
```

Vérifier la configuration

    unbound-checkconf /etc/unbound/unbound.conf.d/cwwk-unbound.conf

*unbound-checkconf: no errors in /etc/unbound/unbound.conf.d/cwwk-unbound.conf*

créer la configuration `local-unbound.conf`

    /etc/unbound/unbound.conf.d/local-unbound.conf

```
server:
    # private-address définit les sous-réseaux de réseau privé sur votre infrastructure.
    # Seuls les noms ‘private-domain’ et ‘local-data’  sont autorisés à avoir ces adresses privées.
	private-address: 192.168.0.0/16
	private-address: 192.168.10.0/16
	private-address: 192.168.70.0/16
	private-address: fe80::/10

    # access-control définit le contrôle d'accès pour les 
    # clients autorisés à effectuer des requêtes  récursives sur le serveur Unbound. 
    # Le paramètre « allow » permet les requêtes récursives, 
    # tandis que « allow_snoop » permet les requêtes récursives et non récursives.
    access-control: 192.168.0.0/16 allow
    access-control: 192.168.10.0/16 allow
    access-control: 192.168.70.0/16 allow

    # Pour configurer un domaine local pour votre réseau, 
    # vous pouvez définir une zone locale dans Unbound. 
    # Cela vous permet d'accéder facilement aux applications auto-organisées sur votre réseau local.
    #
    local-zone: "home.arpa." transparent
    #local-zone: "home.arpa." static
    # IPv4
    local-data: "cwwk.home.arpa.  86400 IN A 192.168.0.205"
    local-data: "pve.home.arpa.  86400 IN A 192.168.0.205"
    #local-data: "cockpit.home.arpa.  86400 IN A 192.168.0.217"
    local-data: "site.home.arpa.  86400 IN A 192.168.0.205"
    local-data: "alp01.home.arpa. 86400 IN A 192.168.10.210"
    local-data: "centreon.home.arpa.  86400 IN A 192.168.0.29"
    # IPv6
    local-data: "cwwk.home.arpa.  86400 IN AAAA 2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45"
    local-data: "pve.home.arpa.  86400 IN AAAA 2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45"
    # On ajoute les enregistrements PTR
    # PTR IPv4
    local-data-ptr: "192.168.0.205  86400 cwwk.home.arpa."
    local-data-ptr: "192.168.0.205  86400 pve.home.arpa."
    #local-data-ptr: "192.168.0.217  86400 cockpit.home.arpa."
    local-data-ptr: "192.168.10.210 86400 alp01.home.arpa."
    local-data-ptr: "192.168.10.29 86400 centreon.home.arpa."
    # PTR IPv6
    local-data-ptr: "2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45  86400 cwwk.home.arpa."
    local-data-ptr: "2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45  86400 pve.home.arpa."
    local-data-ptr: "2a01:e0a:9c8:2080:aab8:e0ff:fe04:ec45  86400 site.home.arpa."
```

Vérifier la configuration

    unbound-checkconf /etc/unbound/unbound.conf.d/local-unbound.conf

*unbound-checkconf: no errors in /etc/unbound/unbound.conf.d/local-unbound.conf*

**OPTION NON TESTEE**  
Configuration Unbound comme un résolveur DNS avec DNS-over-TLS (DoT)

créer la configuration `dot-unbound.conf`

    /etc/unbound/unbound.conf.d/dot-unbound.conf

```
server:
    # 
    # configure Unbound comme un résolveur DNS pour vos réseaux locaux.
    forward-zone:
    # envoie toutes les requêtes DNS.
    name: "."
    # permet DNS-over-TLS (DoT).
    forward-ssl-upstream: yes
    # spécifie les transitaires pour les requêtes DNS.
    forward-addr: 9.9.9.9@853#dns.quad9.net
    forward-addr: 149.112.112.112@853#dns.quad9.net
```

Dans cet exemple, nous utilisons des serveurs Quad9 DNS avec DoT activé.

Vérifier la configuration

    unbound-checkconf /etc/unbound/unbound.conf.d/dot-unbound.conf

*unbound-checkconf: no errors in /etc/unbound/unbound.conf.d/dot-unbound.conf*


Redémarrer Unbound 

    systemctl restart unbound

La machine utilise son propre serveur DNS (interface: 192.168.0.205), il faut modifier resolv.conf

    /etc/resolv.conf

```
nameserver 192.168.0.205
```

Faire `/etc/resolv.conf` immuable

Cette approche rendra immuable `/etc/resolv.conf` afin qu'il ne puisse pas être modifié, quels que soient les paquets installés ou ceux qui tentent de le modifier.

```
sudo rm -f /etc/resolv.conf
sudo nano /etc/resolv.conf
sudo chattr +i /etc/resolv.conf
```

Évidemment, vous devrez mettre le contenu approprié dans le fichier avant de définir le bit immuable. Chaque fois que vous souhaitez changer le fichier, vous devrez supprimer le bit, faire votre changement, puis restaurer le bit.

### Mise à jour automatique adresses des serveurs de nom racine

*Serveur racine du DNS (Root Name Server) : il s’agit du serveur de noms pour la zone racine. Il répond à des requêtes directes et peut renvoyer une liste de noms de serveurs faisant autorité pour le domaine de haut niveau correspondant.*

Fichier service `/etc/systemd/system/roothints.service`

```
[Unit]
Description=Update root hints for unbound
After=network.target
[Service]
ExecStart=/usr/bin/curl -o /var/lib/unbound/root.hints https://www.internic.net/domain/named.cache
```

Fichier timer `/etc/systemd/system/roothints.timer`

```
[Unit]
Description=Run root.hints semiannually
[Timer]
OnCalendar=semiannually
Persistent=true     
[Install]
WantedBy=timers.target
```

Tester le chargement du fichier :

```bash
mv /var/lib/unbound/root.hints /var/lib/unbound/root.hints.BU
systemctl start roothints.service
ls -ls /var/lib/unbound/root.hints*
```

Démarrer le timer roothints.timer

    sudo systemctl enable roothints.timer --now

*Created symlink /etc/systemd/system/timers.target.wants/roothints.timer → /etc/systemd/system/roothints.timer.*

### Tester le serveur DNS Unbound

Pour vous assurer que votre serveur DNS Unbound fonctionne correctement, vous pouvez effectuer quelques tests. Commencez par utiliser la commande `dig` pour interroger les noms de domaines externes/internet. Par exemple, exécutez la commande suivante pour interroger le nom de domaine **xoyize.xyz** :

    dig xoyize.xyz

Si la commande retourne des enregistrements DNS pour le domaine, cela signifie que votre serveur DNS Unbound résout avec succès les noms de domaine externes.

Ensuite, vous pouvez vérifier le domaine local et les sous-domaines que vous avez configurés plus tôt. Par exemple, exécutez la commande suivante pour interroger le sous-domaine **centreon.home.arpa** :

    dig centreon.home.arpa +short

Si la commande retourne l'adresse IP correcte associée au sous-domaine, cela signifie que votre serveur DNS Unbound résout avec succès les noms de domaine locaux.

Vous pouvez également vérifier les enregistrements DNS inversés (PTR) en interrogeant l'adresse IP associée à un sous-domaine. Par exemple, exécutez la commande suivante pour interroger l'adresse IP **192.168.0.29** :

    dig -x 192.168.0.29 +short

Si la commande retourne le nom de domaine approprié associé à l'adresse IP, cela signifie que votre serveur DNS Unbound résout avec succès les enregistrements DNS inversés.

### TcDump Tshark

* [TCPDump : capturer et analyser le trafic réseau sur Linux](https://www.malekal.com/tcpdump-capturer-des-paquets-reseaux-sur-linux/)
* [Maîtriser l'Outil d'Analyse Réseau TCPDump](https://blog.stephane-robert.info/docs/securiser/reseaux/analyse/tcpdump/)
* [DNS Over TLS With Unbound](https://www.jwillikers.com/dns-over-tls-with-unbound)

Pour tester DNS-over-TLS (DoT), vous pouvez utiliser la commande tcpdump pour surveiller le trafic. 

    sudo apt install tcpdump tshark

Une fois installé, exécutez la commande tcpdump suivante pour surveiller le trafic sur l'interface enp3s0 avec le port DoT 853:

```shell
sudo tcpdump -vv -x -X -s 1500 -i enp3s0 'port 853'
```

Déplacez-vous vers la machine cliente et effectuez des requêtes DNS en utilisant la commande `dig`

```shell
dig google.com
```

Si la sortie tcpdump affiche le trafic DNS sur le port 853, cela signifie que votre serveur DNS Unbound utilise avec succès DNS-over-TLS.

Analyse port 53

```shell
sudo tcpdump -v -i enp3s0 -s 65535 -w dns.pcap dst port 53
```

Analyse tshark

```shell
tshark -r dns.pcap
```

## Options

### Bloquer la publicité

* [Bloquer la publicité grâce au DNS](https://www.shaftinc.fr/blocage-pubs-unbound.html)
* [Blocage de pubs et traqueurs avec Unbound](https://framagit.org/Shaft/unbound-adblock)

Prérequis, jq et curl installés  
Script /usr/local/bin/unbound-adblock

<details>
<summary><b>Etendre Réduire</b></summary>
{% highlight bash %}  

#!/bin/bash

###############################################################################################################################
#
# Script construisant un fichier de configuration Unbound, à partir de listes connues
# en vue d'en faire un bloqueur de pubs et de mouchards
#
# Unbound doit répondre NXDOMAIN pour toute requête portant ces domaines
#
# Voir ici pour les détails : https://framagit.org/Shaft/unbound-adblock/
# et la documentation d'Unbound : https://nlnetlabs.nl/documentation/unbound/unbound.conf/
#
# Par Shaft, sous Licence Publique IV (https://www.shaftinc.fr/licence-publique-iv.html)
#
###############################################################################################################################

tmpErr=$(mktemp)
logFile="/var/log/adblock.log"

# On vérifie que le fichier de log existe et on le crée sinon
if [ ! -f $logFile ]; then
	touch $logFile && chown root:adm $logFile
fi

############### Fonctions de logs ###############
# Vu ici : http://www.cubicrace.com/2016/03/efficient-logging-mechnism-in-shell.html

function LOG(){
	local lvl=$1
	local msg="$2"
	echo -e "$(date +'%b %d %X %z') $lvl: $msg" >> $logFile
}

function SYSLOG(){
	local lvl=$1
	local msg="$2"
	# err : error ; info ; warning
	logger -p local0.$lvl -t ${0##*/} --id=$$ $msg
}

#################################################

# Suppresions des fichiers temporaires
function supp_temp(){
	for temp in ${!tmp@};do
		declare -n fic=$(printf $temp)
		rm $fic
	done
}

# On vérifie que les dépendances sont installées

for prog in jq curl; do
	command -v $prog > /dev/null
	if [ "$?" -ne 0 ];then
		LOG ERROR "$prog non installé. Arrêt du script"
		SYSLOG err "$prog non installé. Arrêt du script"
		rm $tmpErr
		exit 1
	fi
done

# On vérifie qu'un paramètre est bien passé au script
if [ -z "$1" ];then
	echo "usage : $0 <Fichier JSON>"
	LOG ERROR "Aucun fichier fourni en argument. Arrêt du script"
	SYSLOG err "Aucun fichier fourni en argument. Arrêt du script"
	rm $tmpErr
	exit 1
fi
json=$1

# Si le fichier json est incorrect, on arrête le script ici.
jq  -e '.' $json &> $tmpErr
if [ "$?" -ne 0 ];then
	LOG ERROR "Le fichier $json est incorrect :"
	LOG ERROR "$(cat $tmpErr)"
	LOG ERROR "Arrêt du script"
	SYSLOG err "Le fichier $json est incorrect, voir $logFile. Arrêt du script."
	rm $tmpErr
	exit 1
fi

tmpListe=$(mktemp)
tmpListeClean=$(mktemp)
tmpJSON=$(mktemp)

filePath="/var/lib/unbound"
confFile="/etc/unbound/unbound.conf.d/adblock.conf"
listFile="$filePath/liste-ad.txt"
diffFile="$filePath/modif-ad.diff"
sauvConf="$filePath/adblock.conf.bak"

NbEchecs=0


# On initialise les compteurs pour la boucle de DL des listes. Attention jq énumère à partir de 0
i=0
n=$(jq '. | length' $json)

# On teste si le JSON est vide
if [ -z "$n" ];then
	LOG ERROR "$json semble vide, merci de vérifier le fichier. Arrêt du script"
	LOG ERROR "Éventuellement, le télécharger en https://framagit.org/Shaft/unbound-adblock/-/raw/main/liste-adblock.json"
	SYSLOG err "$json semble vide, arrêt du script"
	supp_temp
	exit 1
fi

# On initialise le compteur de nouvelles listes
listCount=0

# Nombre de listes présentes dans le JSON lors du dernier DL
nbListes=$(grep -s "Téléchargement des [0-9]* listes de domaines" $logFile.1 $logFile | tail -1 | awk '{ print $8 }')

# Si le grep échoue ou ne renvoie tout simplement rien, on passe $n comme valeur
if [ "$?" -ne 0 ] || [ -z "$nbListes" ];then
	nbListes=$n
fi
# On copie le JSON dans le fichier temporaire
cp $json $tmpJSON

# Pour alléger un peu le script, on fait une fonction pour la création de la liste finale

make_liste(){
	tmpConf=$(mktemp)

	echo -e "# Liste générée le $(date)" > $tmpConf
	# Bien penser à indiquer la directive server au début du fichier de configuration ou Unbound pourrait ne pas démarrer
	echo -e "server:" >> $tmpConf

	while read domaine;do
			echo -e "local-zone: \"$domaine\" static" >> $tmpConf
	done < $tmpListeClean

	# Une fois le fichier crée, on vérifie l'ensemble de la configuration Unbound (avec notre fichier) et si tout est OK, on redémarre le service
	# S'il y a déjà un fichier confFile existant, on en fait un backup
	if [ -f $confFile ]; then
		cp -a $confFile $sauvConf
		LOG INFO "Sauvegarde du précédent fichier de configuration dans $sauvConf"
	fi

	LOG INFO "Copie de la nouvelle configuration dans $confFile"
	mv -f $tmpConf $confFile
	chmod 644 $confFile
	unbound-checkconf &> $tmpErr
	if [ "$?" -ne 0 ]; then
		LOG ERROR "La vérification de la configuration d'Unbound a échouée. Erreurs trouvées :"
		LOG ERROR "$(cat $tmpErr)"
		LOG ERROR "Arrêt du script."
		# Optionnel : si le script est lancé comme un service, on laisse un message dans le syslog pour avoir plus rapidement des infos
		SYSLOG err "La vérification de la configuration d'Unbound a échouée, voir $logFile. Arrêt du script."
		supp_temp
		if [ -f $sauvConf ]; then
			cp $sauvConf $confFile
			LOG INFO "Restauration du précédent fichier de configuration."
		else
			rm $confFile
			LOG INFO "Suppression du fichier de configuration."
		fi
		exit 1
	else
		LOG INFO "Redémarrage d'Unbound."
		unbound-control reload
		# systemctl restart unbound.service
		# On ne copie la nouvelle version de la liste que maintenant
		mv -f $tmpListeClean $listFile
		mv -f $tmpJSON $json
		chmod 644 $listFile
		LOG INFO "Blocage de $(wc -l $listFile | awk '{print $1}') domaines."
		SYSLOG info "Nouvelle version en place, $(wc -l $listFile | awk '{print $1}') domaines bloqués."
		rm $tmpErr
		exit 0
	fi
}

#################################################

# Si nbListes est supérieur à n alors une liste a été retirée du JSON
# On réinitialise les hash dans ce dernier
if [ $nbListes -gt $n ];then
	LOG INFO "Nombre de listes dans le JSON inférieur à celui présent lors de la dernière exécution."
	LOG INFO "Réinitialisation des hash des listes présentes."
	LOG INFO "Remise à zéro de $listFile et suppression des .list."
	SYSLOG info "Une liste a été retirée du JSON, réinitialisation des blocages"
	sed -i 's/\"Hash.*/\"Hash\"\: null/g' $json
	printf "" > $tmpListe
	rm -r $filePath/*.list
fi

LOG INFO "Téléchargement des $n listes de domaines"

# Boucle de téléchargement des listes, préparation et nettoyage avant de constituer la liste principale
while [ $i -le $(($n - 1)) ];do
	nom=$(jq --raw-output .[$i].Nom $json)
	url=$(jq --raw-output .[$i].URL $json)
	hash=$(jq --raw-output .[$i].Hash $json)
	tmpDL=$(mktemp)
	# Si $nom contient des espaces, on les remplace
	sauvList="$filePath/${nom//\ /-}.list"

	curl -sSf --connect-timeout 5 $url -o $tmpDL 2>$tmpErr

	if [ "$?" -ne 0 ];then
		LOG ERROR "Échec du téléchargement de liste $nom. L'erreur suivante a été détectée :"
		LOG ERROR "$(cat $tmpErr)"
		((NbEchecs++))

		# Si le téléchargement d'au moins la moitié des listes (arrondi inférieur) échoue et
		# qu'aucune nouvelle liste n'a été téléchargée, ça ne sert à rien de continuer
		if [ $NbEchecs -ge $(($n / 2)) -a $listCount -eq 0 ]; then
			LOG ERROR "Le téléchargement de $NbEchecs listes a échoué."
			LOG ERROR "Au moins la moitié des listes en erreur. Arrêt du script"
			#Optionnel : si le script est lancé comme un service, on laisse un message dans le syslog pour avoir plus rapidement des infos
			SYSLOG err "Le téléchargement de $NbEchecs listes sur $n a échoué, voir $logFile. Arrêt du script"
			supp_temp
			exit 1
		fi
	else
		# On regarde si le sha256 est différent
		if [ $(sha256sum $tmpDL  | awk '{print $1}') = $hash ]; then
			LOG INFO "Téléchargement de la liste $nom réussi. Pas de différences constatées. Liste ignorée."
		else
			case $hash in
				null)
					LOG INFO "Téléchargement de la liste $nom réussi. Hash nul, la liste est nouvelle."
					;;
				*)
					LOG INFO "Téléchargement de la liste $nom réussi. Différences constatées. Ajout à la nouvelle liste."
					;;
			esac
			((listCount++))
			# On écrit le nouveau hash dans le JSON
			cat $tmpJSON | jq --arg newHash "$(sha256sum $tmpDL | awk '{print $1}')" --argjson i "$i" '.[$i].Hash = $newHash' > $tmpJSON

			# On teste si la liste est un fichier hosts eg.
			# il contient des lignes type "127.0.0.1	méchant.example"
			# On adapte le traitement en fonction
			grep -E "^127\.0\.0\.1|^0\.0\.0\.0" $tmpDL &> /dev/null

			if [ "$?" -ne 0 ];then
				grep -v "^#" $tmpDL | awk '{print $1}' | tr -d "\r" | sed '/^\s*$/d' > $sauvList
			else
				# Il faut faire attention à enlever les lignes classiques d'un fichier hosts
				# telle que "127.0.0.1	localhost" qui peuvent rester.
				grep -E "^127\.0\.0\.1|^0\.0\.0\.0" $tmpDL | awk '{print $2}' | tr -d "\r" | sed '/^\s*$/d' | grep -Ev "^localhost|^localhost.localdomain|^local$|^127\.0\.0\.1$|^0\.0\.0\.0$" > $sauvList
			fi
			LOG INFO "Sauvegarde dans $sauvList"
		fi
	fi
	rm $tmpDL
	((i++))
done

# On loggue le nombre d'erreurs de téléchargements, s'il y en a
if [ $NbEchecs -gt 0 ];then
	LOG WARNING "Le téléchargement de $NbEchecs liste(s) sur $n a échoué."
	#Optionnel : si le script est lancé comme un service, on laisse un message dans le syslog pour avoir plus rapidement des infos
	SYSLOG warning "Le téléchargement de $NbEchecs liste(s) sur $n a échoué, voir $logFile"
fi

# Tri et suppression des doublons de la liste.
for file in $(ls $filePath/*.list);do
	cat $file >> $tmpListe
done
sort -bfu $tmpListe -o $tmpListeClean

rm $tmpListe

# On vérifie si la dernière version de la liste nettoyée est présente.
# Si c'est le cas, on vérifie si la nouvelle présente des différences et on reconstruit le fichier de conf en fonction

if [ -f $listFile ]; then
	if [ $listCount -eq 0 ]; then
		LOG INFO "Pas de différences constatées avec la liste du $(date -d @$(stat -c %Y $confFile)) déjà présente. Arrêt du script."
		SYSLOG info "Pas de nouvelle version, $(wc -l $listFile | awk '{print $1}') domaines bloqués."
		supp_temp
		exit 0
	else
		LOG INFO "Des différences ont été constatées avec la liste déjà présente, voir $diffFile"
		diff $tmpListeClean $listFile > $diffFile
		LOG INFO "Construction d'un nouveau fichier de configuration."
		make_liste
	fi
else
	LOG INFO "Pas de liste trouvée. Construction de celle-ci et du fichier de configuration."
	make_liste
fi
{% endhighlight %}
</details>

Droits en exécution

	chmod +x /usr/local/bin/unbound-adblock

Copier [liste-adblock.json](https://framagit.org/Shaft/unbound-adblock/-/blob/main/liste-adblock.json) dans un fichier `/var/lib/unbound/liste-adblock.json`

<details>
<summary><b>Etendre Réduire</b></summary>
{% highlight json %}  
[
  {
    "Nom": "MVPS",
    "URL": "https://winhelp2002.mvps.org/hosts.txt",
    "Hash": null
  },
  {
    "Nom": "AdAway",
    "URL": "https://adaway.org/hosts.txt",
    "Hash": null
  },
  {
    "Nom": "StevenBlack Unifié",
    "URL": "https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts",
    "Hash": null
  },
  {
    "Nom": "yoyo.org",
    "URL": "https://pgl.yoyo.org/adservers/serverlist.php?hostformat=nohtml&showintro=0&mimetype=plaintext",
    "Hash": null
  }
]

{% endhighlight %}
</details>

Copier [adblock](https://framagit.org/Shaft/unbound-adblock/-/blob/main/adblock) dans /etc/logrotate.d/ (Attention : ne pas supprimer l'option delaycompress au risque de casser la détection de retrait de liste du JSON).

    wget -O /etc/logrotate.d/adblock https://framagit.org/Shaft/unbound-adblock/-/blob/main/adblock

Lancer le script au démarrage de la machine avec un service systemd, ainsi une relance d'Unbound est moins pénalisante étant donné que redémarrer ce résolveur vide son cache.  
Créer [adblock.service](https://framagit.org/Shaft/unbound-adblock/-/blob/main/adblock.service) `    /etc/systemd/system/adblock.service`

```
[Unit]
Description=Unbound AdBlock List Making
After=unbound.service network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/unbound-adblock /var/lib/unbound/liste-adblock.json
RemainAfterExit=yes
User=root
PrivateTmp=true
ProtectHome=true

[Install]
WantedBy=multi-user.target
```

Activer et démarrer le service 

    systemctl enable --now adblock.service

vérifier que tout c'est bien passé via le fichier de log `/var/log/adblock.log`

```
sept. 16 08:00:19 +0000 INFO: Téléchargement des 4 listes de domaines
sept. 16 08:00:21 +0000 INFO: Téléchargement de la liste MVPS réussi. Hash nul, la liste est nouvelle.
sept. 16 08:00:21 +0000 INFO: Sauvegarde dans /var/lib/unbound/MVPS.list
sept. 16 08:00:21 +0000 INFO: Téléchargement de la liste AdAway réussi. Hash nul, la liste est nouvelle.
sept. 16 08:00:21 +0000 INFO: Sauvegarde dans /var/lib/unbound/AdAway.list
sept. 16 08:00:22 +0000 INFO: Téléchargement de la liste StevenBlack Unifié réussi. Hash nul, la liste est nouvelle.
sept. 16 08:00:22 +0000 INFO: Sauvegarde dans /var/lib/unbound/StevenBlack-Unifié.list
sept. 16 08:00:23 +0000 INFO: Téléchargement de la liste yoyo.org réussi. Hash nul, la liste est nouvelle.
sept. 16 08:00:23 +0000 INFO: Sauvegarde dans /var/lib/unbound/yoyo.org.list
sept. 16 08:00:23 +0000 INFO: Pas de liste trouvée. Construction de celle-ci et du fichier de configuration.
sept. 16 08:00:26 +0000 INFO: Copie de la nouvelle configuration dans /etc/unbound/unbound.conf.d/adblock.conf
sept. 16 08:00:26 +0000 INFO: Redémarrage d'Unbound.
sept. 16 08:00:27 +0000 INFO: Blocage de 166353 domaines.
```

Test avec un site qui est dans la liste

    dig ad2.adfarm1.adition.com

```
; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> ad2.adfarm1.adition.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 16929
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;ad2.adfarm1.adition.com.	IN	A

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Mon Sep 16 08:00:47 GMT 2024
;; MSG SIZE  rcvd: 52
```

`La requête DNS n'est pas servie et se voit répondre NXDOMAIN`{: .prompt-warning }

### Blocage des DMP

**Blocage des DMP (Data Management Platforms) avec Unbound**  

*Configuration nécessaire pour Unbound afin de bloquer certaines "Data Management Platforms"  (DMP) utilisées par de plus en plus de sites (liberation.fr, oui.scnf, lemonde.fr, fnac.com...) et qui échappent – pour l'instant aux bloqueurs de traqueurs traditionnels (uBlock Origin ou uMatrix par exemple)*

**Le cœur du problème**   
Dernièrement, une conjoncture d'éléments est venue semer le trouble chez nos aspirateurs :

* L'entrée en vigueur du RGPD en Europe
* La part de plus en plus grande d'internautes utilisant des bloqueurs de publicités
* La volonté des éditeurs de navigateurs web (Chrome et Firefox – et donc leurs dérivés) de bloquer par défaut les traqueurs ou les cookies tiers.

Un mouchard est dit tiers – ou 3rd-party quand on est disruptif – quand il n'est pas chargé depuis le domaine (ou un de ses sous domaines) que vous visitez. Par exemple quand on visite https://liberation.fr/, le script chargé depuis www.google-analytics.com est tiers. Si le même mouchard est chargé depuis un sous-domaine de liberation.fr, il est dit 1st-party (primaire a priori en bon français).
Le hic, pour nos amis start-uppers est que les mouchards/cookies tiers sont triviaux à bloquer. Ne pas charger les cookies tiers est d'ailleurs une option répandue dans les navigateurs depuis des années.

**La fourberie**  
La technique développée par les marketeux pour ne pas voir les juteuses données personnelles leur échapper est à la fois techniquement très simple et redoutable. Il suffit de transformer les crasses 3rd-party en 1st-party afin de les cacher sous le cyber-tapis pour reprendre l'expression de Reflets. Et pour se faire passer par le DNS et ses possiblités.

**Bloquer les indésirables avec Unbound**  
On entre dans la partie technique, pour faire court, on va décréter à Unbound que nous contrôlons les domaines indésirables (eulerian.net par exemple) afin de faire échouer la résolution DNS. La technique est [développée plus en détail sur un blog](https://www.shaftinc.fr/escalade-traque-eulerian.html). 

Pour mémoire :

* On crée un fichier de zone (**eulerian.net.zone**) où l'on met un enregistrement SOA.
* On l'associe à chaque domaine à bloquer dans la configuration d'Unbound (**adblock-war.conf**)

Cette technique sera valable, pour les versions d'Unbound supérieures à 1.7.0.

Créer un fichier `/var/lib/unbound/eulerian.net.zone`

```
$TTL 10800
eulerian.net.	IN SOA localhost. nobody.invalid. (
	1
	3600
	1200
	604800
	10800
	)
```

Dans le détail, on indique la durée de vie par défaut des enregistrements ($TTL 10800) puis le début de l'autorité pour la zone (avec le nom du serveur faisant autorité (localhost.), le mail de l'admin (nobody@invalid. – le @ est un caractère particulier dans le DNS, on le remplace donc par un point) et c'est tout, le reste du domaine sera vide. 

créer 2 copies de ce fichier nommées `eulerian.com.zone` et `eulerian.fr.zone` et changer le domaine.

fichier `/var/lib/unbound/eulerian.com.zone`

```
$TTL 10800
eulerian.com.	IN SOA localhost. nobody.invalid. (
	1
	3600
	1200
	604800
	10800
	)
```

fichier `/var/lib/unbound/eulerian.fr.zone`

```
$TTL 10800
eulerian.fr.	IN SOA localhost. nobody.invalid. (
	1
	3600
	1200
	604800
	10800
	)
```

Configuration d'Unbound

Dans un fichier .conf spécifique, typiquement  `/etc/unbound/unbound.conf.d/block-eurelian.conf`  sous Debian et dérivés, ajouter :

```
auth-zone:
			name: "eulerian.net."
			zonefile: "/var/lib/unbound/eulerian.net.zone"
		auth-zone:
			name: "eulerian.fr."
			zonefile: "/var/lib/unbound/eulerian.fr.zone"
		auth-zone:
			name: "eulerian.com."
			zonefile: "/var/lib/unbound/eulerian.com.zone"
```

On définit donc 3 zones sur lesquelles nous décrétons avoir l'autorité et pour chacune d'entre elles, on dit à Unbound d'utiliser les fichiers correspondants. On sauvegarde et on relance Unbound et normalement, le blocage est effectif :

Redémarrer unbound

    systemctl restart unbound

Vérifier

    dig v.oui.sncf

```
; <<>> DiG 9.18.28-1~deb12u2-Debian <<>> v.oui.sncf
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 42150
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;v.oui.sncf.			IN	A

;; ANSWER SECTION:
v.oui.sncf.		3600	IN	CNAME	voyages-sncf.eulerian.net.

;; Query time: 536 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Mon Sep 16 08:32:29 GMT 2024
;; MSG SIZE  rcvd: 78
```

`La requête n'est pas servie (NXDOMAIN)`{: .prompt-warning }

### Activer logs unbound

Modifier le fichier de configuration unbound

```
    logfile: /var/log/unbound.log
    verbosity: 1
    log-queries: yes
```

Créer le fichier log avec les droits

```
touch /var/log/unbound.log
chown unbound:unbound /var/log/unbound.log
```

Relancer le service unbound

    systemctl restart unbound

Visualiser les logs

    tail -f /var/log/unbound.log

## Alertes et Erreurs

En cas d'anomalie **subnetcache**

```
juin 17 16:23:23 cwwk systemd[1]: Starting unbound.service - Unbound DNS server...
juin 17 16:23:23 cwwk unbound[513377]: [513377:0] warning: subnetcache: serve-expired is set but not working for data originating from the subnet module cache.
juin 17 16:23:23 cwwk unbound[513377]: [513377:0] warning: subnetcache: prefetch is set but not working for data originating from the subnet module cache.
juin 17 16:23:23 cwwk unbound[513377]: [513377:0] info: start of service (unbound 1.17.1).
juin 17 16:23:23 cwwk systemd[1]: Started unbound.service - Unbound DNS server.
```

Modifier `/etc/unbound/unbound.conf.d/cwwk-unbound.conf`

```
    # # Fetch the DNSKEYs earlier in the validation process, when a DS record
    # is encountered. This lowers the latency of requests at the expense of
    # little more CPU usage.
    # prefetch: yes
    prefetch: no

    # Have unbound attempt to serve old responses from cache with a TTL of 0 in
    # the response without waiting for the actual resolution to finish. The
    # actual resolution answer ends up in the cache later on.
    # serve-expired: yes
    serve-expired: no
```

Relancer unbound et vérifier

```shell
sudo systemctl restart unbound
systemctl status unbound
```

Résultat

```
juin 17 16:33:44 cwwk systemd[1]: Starting unbound.service - Unbound DNS server...
juin 17 16:33:44 cwwk unbound[513517]: [513517:0] info: start of service (unbound 1.17.1).
juin 17 16:33:44 cwwk systemd[1]: Started unbound.service - Unbound DNS server.
```

## Annexe

### DNS Debian (resolvconf)

* Article original [Résolution des noms avec resolvconf sous Linux Debian](https://coagul.org/drupal/publication/r%C3%A9solution-noms-resolvconf-sous-linux-debian) du 27/08/2106

*En fonction du type de connexion utilisé, il est parfois nécessaire de faire appel à différents serveurs de noms (DNS). Par exemple, lors d’une connexion à son lieu de travail, il faut utiliser le serveur DNS de son réseau, mais lors d’une connexion à internet, il faut utiliser les serveurs DNS de son fournisseur d’accès. Dans ce cas, le paquet **"resolvconf"** sous Debian permet de résoudre ces problèmes.*

**Rappel sur l’utilité du fichier « /etc/resolv.conf »**  
Ce fichier permet d’indiquer le ou les domaines de recherche et les différents serveurs DNS à utiliser.

Par exemple, dans un réseau local, nous pourrions avoir un serveur DNS à l’adresse 192.168.0.1 chargé de gérer le domaine « mon-domaine.local ». En cas de défaillance du DNS local, nous pourrions faire appel aux serveurs DNS de notre fournisseur d’accès. Dans ce cas, le contenu du fichier « /etc/resolv.conf », pourrait ressembler à cela :

```
nameserver 192.168.0.1
nameserver 212.27.53.252
nameserver 212.27.52.252
search mon-domaine.local
```

La première ligne indique l’adresse du serveur DNS du réseau local. En cas de défaillance de ce serveur, les serveurs suivants seront utilisés (Serveurs du fournisseur d’accès à Internet).

La dernière ligne permet d’indiquer le nom du domaine géré par le serveur DNS local. Par exemple, si nous cherchons à contacter le serveur « **MonServeur**  », le système cherchera en fait à contacter l’adresse complète « **MonServeur.mon-domaine.local**  », car le nom du serveur indiqué ne comportait pas le domaine de recherche.

**Présentation et installation de resolvconf**  
Le programme **resolvconf** garde la trace des informations du système sur les serveurs de noms de domaine actuellement disponibles. Il ne faut pas le confondre avec le fichier de configuration **resolv.conf** qui porte malencontreusement presque le même nom. <u>Le programme resolvconf est optionnel sur les systèmes Debian.</u>

Le fichier de configuration **resolv.conf** contient des informations sur les serveurs de noms de domaine que le système doit utiliser. Néanmoins, quand plusieurs programmes doivent modifer dynamiquement le fichier de configuration resolv.conf, ils peuvent se chevaucher et le fichier peut ne plus être synchronisé. Le programme resolvconf s'occupe de ce problème. Il agit comme un intermédiaire entre les programmes qui fournissent des informations sur les serveurs de noms de domaine (par exemple les clients dhcp) et les programmes qui les utilisent (par exemple resolver).

Quand **resolvconf** est correctement installé, le fichier de configuration resolv.conf du répertoire **/etc/resolv.conf** est remplacé par un lien symbolique pointant vers le fichier /etc/resolvconf/run/resolv.conf et le résolveur utilise plutôt le fichier de configuration qui est généré dynamiquement par resolvconf à cet emplacement /etc/resolvconf/run/resolv.conf.

Le programme resolvconf est en général seulement nécessaire quand un système a plusieurs programmes qui ont besoin de modifier de façon dynamique les informations sur les serveurs de noms de domaine. Sur un système simple où les serveurs de noms de domaine ne changent pas souvent ou bien ne sont modifiés que par un programme, le fichier de configuration resolv.conf est suffisant.

Si le programme resolvconf est installé, vous n'aurez pas à modifier à la main le fichier de configuration resolv.conf car il sera changé de façon dynamique par les programmes. Si vous avez besoin de définir vous-même les serveurs de noms de domaine (comme avec une interface statique), ajoutez au fichier de configuration interfaces du répertoire /etc/network/interfaces une ligne comme celle-ci :

    dns-nameservers 127.0.0.1 80.67.169.12 80.67.169.40

Mettez la ligne indéntée dans un paragraphe iface, par exemple juste après la ligne gateway. Entrez les adresses IP des serveurs de noms de domaine dont vous avez besoin après dns-nameservers, toutes sur la même ligne, séparées par des espaces. N'oubliez pas le "s" à la fin de dns-nameservers.

Le programme resolvconf est un ajout plutôt récent à Debian et plusieurs anciens programmes ont besoin d'être mis à jour ou reconfigurés pour fonctionner correctement avec lui . Si vous rencontrez des problèmes, regardez */usr/share/doc/resolvconf/README* qui contient beaucoup d'informations sur la manière de faire fonctionner resolvconf avec d'autres programmes. 

	apt install resolvconf -y
	echo "nameserver 127.0.0.1" >> /etc/resolvconf/resolv.conf.d/head

>Une fois le paquet «  **resolvconf**  » installé, <u>il ne faut plus modifier le fichier</u> « **/etc/resolv.conf**  », car le contenu de celui-ci sera automatiquement géré et remplacé par «  **resolvconf**  ».

Le résultat de la commande  

	nslookup afnic.fr | grep Server

devrait ressembler à ceci:

	Server:     127.0.0.1


Vérifier la résolution de nom à partir du serveur :  

	dig @127.0.0.1 afnic.fr

```
; <<>> DiG 9.10.3-P4-Debian <<>> @127.0.0.1 afnic.fr
; (1 server found)
...
;; SERVER: 127.0.0.1#53(127.0.0.1)
...
```

La résolution fonctionne  

`Maintenant, vous disposez de votre propre résolveur DNS.`{: .prompt-tip }

