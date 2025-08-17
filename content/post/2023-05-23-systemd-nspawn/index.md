+++
title = 'Archlinux  conteneur systemd-nspawn'
date = 2023-05-23 00:00:00 +0100
categories = ['systemd']
+++
*systemd-nspawn peut être utilisé pour exécuter une commande ou un système d'exploitation dans un espace de noms léger. Il est plus puissant que chroot car il virtualise entièrement la hiérarchie du système de fichiers, ainsi que l'arborescence des processus, les différents sous-systèmes IPC et le nom de l'hôte et du domaine.*


## Service systemd-nspawn

[systemd-nspawn](https://wiki.archlinux.org/title/Systemd-nspawn) ressemble à la commande chroot, mais c'est un chroot sur les stéroïdes.

systemd-nspawn limite l'accès à diverses interfaces du noyau dans le conteneur à la lecture seule, comme /sys, /proc/sys ou /sys/fs/selinux. Les interfaces réseau et l'horloge système ne peuvent pas être modifiées depuis le conteneur. Les nœuds de périphériques ne peuvent pas être créés. Le système hôte ne peut pas être redémarré et les modules du noyau ne peuvent pas être chargés à partir du conteneur.

*systemd-nspawn est un outil plus simple à configurer que LXC ou Libvirt.*

* [systemd-nspawn (wiki archlinux)](https://wiki.archlinux.org/title/Systemd-nspawn)
* [Using systemd-nspawn containers with publicly routable ips (IPv6 and IPv4) via bridged mode for high density testing whilst balancing tenant isolation](https://blog.karmacomputing.co.uk/using-systemd-nspawn-containers-with-publicly-routable-ips-ipv6-and-ipv4-via-bridged-mode-for-high-density-testing-whilst-balancing-tenant-isolation/)

### Exemple conteneur ArchLinux 

*Créer et démarrer un conteneur Arch Linux minimal*

Installez d'abord **arch-install-scripts** 

Ensuite, créez un répertoire pour contenir le conteneur. Dans cet exemple, nous utiliserons `~/MonConteneur` 

Ensuite, nous utilisons `pacstrap` pour installer un système Arch de base dans le conteneur.  
Au minimum, nous devons installer le paquet de base.

```shell
# pacstrap -K -c ~/MonConteneur base [paquets/groupes supplémentaires]
```

Astuce : Le paquet base ne dépend pas du paquet linux kernel et est prêt pour le conteneur.

Une fois l'installation terminée, chrootez dans le conteneur et définissez un mot de passe root :

```shell
# systemd-nspawn -D ~/MonConteneur
# passwd
# logout
```

Enfin, démarrez dans le conteneur :

```shell
# systemd-nspawn -b -D ~/MonConteneur
```

L'option `-b` permet de démarrer le conteneur (c'est-à-dire d'exécuter systemd en tant que PID=1), au lieu d'exécuter simplement un shell, et `-D` spécifie le répertoire qui devient le répertoire racine du conteneur.

Après le démarrage du conteneur, connectez-vous en tant que "root" avec votre mot de passe.

Note : Si la connexion échoue avec "Login incorrect", le problème vient probablement de la liste blanche du périphérique TTY securetty. Voir #Root login fails.
{: .prompt-info }

Le conteneur peut être mis hors tension en exécutant `poweroff` depuis le conteneur. Depuis l'hôte, les conteneurs peuvent être contrôlés par l'outil `machinectl`.

Remarque : pour mettre fin à la session à partir du conteneur, maintenez les touches `Ctrl ALTGR` enfoncées et appuyez rapidement sur `]` à trois reprises.
{: .prompt-info }

### Exemple conteneur Debian

*Créer un environnement Debian ou Ubuntu*

Installez **debootstrap**, ainsi que **debian-archive-keyring** ou **ubuntu-keyring**, ou les deux, en fonction de la distribution choisie.

    yay -S debootstrap debian-archive-keyring

Note : *systemd-nspawn* nécessite que le système d'exploitation dans le conteneur utilise *systemd init* (qu'il tourne en tant que PID 1) et que *systemd-nspawn* soit installé dans le conteneur. Ces exigences peuvent être satisfaites en s'assurant que le paquetage *systemd-container* est installé sur le système du conteneur. Le paquetage *systemd-container* dépend de dbus, qui n'est pas installé par défaut.  
**Assurez-vous que le paquetage *dbus* ou *dbus-broker* est installé sur le système de conteneurs.**
{: .prompt-info }

À partir de là, il est assez facile de configurer les environnements Debian ou Ubuntu :

```shell
# cd /var/lib/machines
# debootstrap --include=dbus-broker,systemd-container --components=main,universe codename container-name repository-url
```

Pour **Debian**, les noms de code valides sont soit les noms de roulement comme "stable" et "testing", soit les noms de version comme "bullseye" et "sid". 

Pour **Ubuntu**, le nom de code "xenial" ou "zesty" doit être utilisé. Une liste complète des noms de code se trouve dans /usr/share/debootstrap/scripts et la table officielle des noms de code et des numéros de version se trouve dans [DevelopmentCodeNames](https://wiki.ubuntu.com/DevelopmentCodeNames#Release_Naming_Scheme).  

Dans le cas d'une image Debian, le "repository-url" peut être https://deb.debian.org/debian/.  
Pour une image Ubuntu, le "repository-url" peut être http://archive.ubuntu.com/ubuntu/. Le "repository-url" ne doit pas contenir de barre oblique à la fin.

Le container bullseyes est créé

```
I: Base system installed successfully.
```

Tout comme Arch, Debian et Ubuntu ne vous laisseront pas vous connecter sans mot de passe. Pour définir le mot de passe root, exécutez systemd-nspawn sans l'option `-b` :

```shell
cd /var/lib/machines
systemd-nspawn -D ./nspbullseye
passwd
logout
```

### Options par défaut

Les conteneurs situés dans `/var/lib/machines/` peuvent être contrôlés par la commande `machinectl`, qui contrôle en interne les instances de l'unité de `service systemd-nspawn@`  
Les sous-répertoires de `/var/lib/machines/` correspondent aux noms des conteneurs, c'est-à-dire `/var/lib/machines/nom du conteneur/` 

Note : Si le conteneur ne peut pas être déplacé dans /var/lib/machines/ pour une raison quelconque, il peut être lié par un lien symbolique. Voir [machinectl  § FILES AND DIRECTORIES](https://man.archlinux.org/man/machinectl.1#FILES_AND_DIRECTORIES) pour plus de détails.
{: .prompt-info }


Il est important de comprendre que les conteneurs démarrés via `machinectl` ou `systemd-nspawn@.service` utilisent des options par défaut différentes de celles des conteneurs démarrés manuellement par la commande `systemd-nspawn`  
Les options supplémentaires utilisées par le service sont les suivantes :


*    `-b/--boot` - Les conteneurs gérés recherchent automatiquement un programme d'initialisation et l'invoquent en tant que PID 1.
*    `--network-veth` qui implique `--private-network` - Les conteneurs gérés obtiennent une interface réseau virtuelle et sont déconnectés du réseau hôte. 
*    `-U` - Les conteneurs gérés utilisent la fonctionnalité user_namespaces(7) par défaut si elle est supportée par le noyau. 
*    `--link-journal=try-guest`


Le comportement peut être modifié dans les fichiers de configuration par conteneur, voir le paragraohe **Configuration** pour plus de détails.

### machinectl

Note : L'outil `machinectl` nécessite que *systemd* et *dbus* soient installés dans le conteneur.
{: .prompt-info }

Les conteneurs peuvent être gérés par la sous-commande `machinectl nom-du-conteneur`  
Par exemple, pour démarrer un conteneur

    machinectl start nom-du-conteneur


Remarque : machinectl exige que le nom du conteneur soit composé uniquement de lettres ASCII, de chiffres et de traits d'union afin qu'il s'agisse d'un nom d'hôte valide. Par exemple, si le nom du conteneur contient un trait de soulignement, il ne sera tout simplement pas reconnu et l'exécution de machinectl start nom_du_conteneur entraînera l'erreur Nom_du_conteneur invalide.
{: .prompt-warning }

De même, il existe des sous-commandes telles que `poweroff`, `reboot`, `status` et `show`. Voir [machinectl  § Commandes machine](https://man.archlinux.org/man/machinectl.1#Machine_Commands) pour des explications détaillées.

**Astuce** : Les opérations de mise hors tension et de redémarrage peuvent être effectuées depuis le conteneur à l'aide des commandes `poweroff` et `reboot`.

D'autres commandes courantes sont :

*    `machinectl lis`t - affiche la liste des conteneurs en cours d'exécution
*    `machinectl login nom-du-conteneur` - ouvre une session de connexion interactive dans un conteneur
*    `machinectl shell [username@]container-name` - ouvre une session interactive dans un conteneur (cela invoque immédiatement un processus utilisateur sans passer par le processus de connexion dans le conteneur)
*     `machinectl enable container-name` et `machinectl disable container-name` - active ou désactive le lancement d'un conteneur au démarrage, voir [#Enable container to start at boot](https://wiki.archlinux.org/title/Systemd-nspawn#Enable_container_to_start_at_boot) pour plus de détails


**machinectl** possède également des sous-commandes pour gérer les images des conteneurs (ou des machines virtuelles) et les transferts d'images. Voir [machinectl § Image Commands](https://man.archlinux.org/man/machinectl.1#Image_Commands) et [machinectl § Image Transfer Commands](https://man.archlinux.org/man/machinectl.1#Image_Transfer_Commands) pour plus de détails. A partir du 2023Q1, les 3 premiers exemples de machinectl(1) § EXEMPLES démontrent les commandes de transfert d'images. [machinectl(1) § FILES AND DIRECTORIES](https://man.archlinux.org/man/machinectl.1#FILES_AND_DIRECTORIES) discute de l'endroit où trouver des images appropriées.


### Chaîne d'outils systemd

Une grande partie de la chaîne d'outils de base de systemd a été mise à jour pour fonctionner avec les conteneurs. Les outils qui le font fournissent généralement une option `-M`, `--machine=` qui prend un nom de conteneur comme argument.

Exemples :

Voir les journaux d'une machine particulière :

    journalctl -M nom-du-conteneur

Afficher le contenu du groupe de contrôle :

    systemd-cgls -M nom-du-conteneur

Voir l'heure de démarrage du conteneur :

    systemd-analyze -M nom-du-conteneur

Pour une vue d'ensemble de l'utilisation des ressources :

    systemd-cgtop

### Fichiers .nspawn

Les fichiers `.nspawn` peuvent être utilisés pour spécifier des paramètres par conteneur et non des paramètres globaux. Voir [systemd.nspawn(5)](https://man.archlinux.org/man/systemd.nspawn.5) pour plus de détails.

Remarque : les fichiers `.nspawn` peuvent être retirés de la base de données :

*    Les fichiers .nspawn peuvent être supprimés de manière inattendue de /etc/systemd/nspawn/ lorsque vous exécutez `machinectl remove`  [[5]](https://github.com/systemd/systemd/issues/15900)
*    L'interaction des options réseau spécifiées dans le fichier `.nspawn` et sur la ligne de commande ne fonctionne pas correctement lorsqu'il y a `--settings=override` (qui est spécifié dans le fichier `systemd-nspawn@.service`). Comme solution de contournement, vous devez inclure l'option `VirtualEthernet=on`, même si le service spécifie `--network-veth`.

### Lancement au démarrage

Lorsque vous utilisez fréquemment un conteneur, il se peut que vous souhaitiez le lancer au démarrage.

Assurez-vous d'abord que l'option `machines.target` est activée.

Les conteneurs détectables par `machinectl` peuvent être activés ou désactivés :

    machinectl enable nom-du-conteneur

Remarque :

*    Cela a pour effet d'activer l'unité systemd `systemd-nspawn@container-name.service`.
*    Comme mentionné dans [#Options systemd-nspawn par défaut](https://wiki.archlinux.org/title/Systemd-nspawn#Default_systemd-nspawn_options), les conteneurs démarrés par machinectl obtiennent une interface Ethernet virtuelle. Pour désactiver le réseau privé, voir [#Utiliser le réseau de l'hôte](https://wiki.archlinux.org/title/Systemd-nspawn#Use_host_networking).

### Contrôle des ressources

Vous pouvez profiter des groupes de contrôle pour implémenter des limites et une gestion des ressources de vos conteneurs avec `systemctl set-property`, voir [systemd.resource-control(5)](https://man.archlinux.org/man/systemd.resource-control.5).  
Par exemple, vous pouvez vouloir limiter la quantité de mémoire ou l'utilisation du processeur. Pour limiter la consommation de mémoire de votre conteneur à 2 GiB :

    sudo systemctl set-property systemd-nspawn@container-name.service MemoryMax=2G

Ou pour limiter l'utilisation du temps CPU à l'équivalent de 2 cœurs :

    sudo systemctl set-property systemd-nspawn@container-name.service CPUQuota=200%

Cela créera des fichiers permanents dans `/etc/systemd/system.control/systemd-nspawn@container-name.service.d/` 

D'après la documentation, `MemoryHigh` est la méthode préférée pour contrôler la consommation de mémoire, mais elle ne sera pas limitée de manière stricte comme c'est le cas avec `MemoryMax`. Vous pouvez utiliser les deux options en laissant `MemoryMax` comme dernière ligne de défense. Prenez également en considération le fait que vous ne limiterez pas le nombre de CPU que le conteneur peut voir, mais vous obtiendrez des résultats similaires en limitant le temps que le conteneur obtiendra au maximum, par rapport au temps total du CPU.

>Astuce : Si vous voulez que ces changements ne soient que temporaires, vous pouvez passer l'option `--runtime`  
Vous pouvez vérifier les résultats avec `systemd-cgtop`

### Mise en réseau

Les conteneurs *systemd-nspawn* peuvent utiliser le réseau hôte ou le réseau privé :

*    Dans le mode réseau hôte, le conteneur a un accès complet au réseau hôte. Cela signifie que le conteneur pourra accéder à tous les services réseau de l'hôte et que les paquets provenant du conteneur apparaîtront au réseau extérieur comme provenant de l'hôte (c'est-à-dire qu'ils partageront la même adresse IP).
*    En mode réseau privé, le conteneur est déconnecté du réseau de l'hôte. Toutes les interfaces réseau sont alors indisponibles pour le conteneur, à l'exception du périphérique de bouclage et des interfaces explicitement attribuées au conteneur.   
Il existe plusieurs façons de configurer des interfaces réseau pour le conteneur :
    *  Une interface existante peut être affectée au conteneur (par exemple, si vous avez plusieurs périphériques Ethernet).
    *   Une interface réseau virtuelle associée à une interface existante (c'est-à-dire une interface VLAN) peut être créée et attribuée au conteneur.
    *   Un lien Ethernet virtuel entre l'hôte et le conteneur peut être créé.  
Dans ce dernier cas, le réseau du conteneur est totalement isolé (du réseau extérieur et des autres conteneurs) et il appartient à l'administrateur de configurer le réseau entre l'hôte et les conteneurs. Cela implique généralement la création d'un pont réseau pour connecter plusieurs interfaces (physiques ou virtuelles) ou la mise en place d'une traduction d'adresses réseau entre plusieurs interfaces.

Le mode de<u> mise en réseau de l'hôte</u> convient aux conteneurs d'application qui n'exécutent aucun logiciel de mise en réseau susceptible de configurer l'interface attribuée au conteneur. Le mode réseau hôte est le mode par défaut lorsque vous exécutez systemd-nspawn à partir de l'interpréteur de commandes.

En revanche, le <u>mode réseau privé</u> convient aux conteneurs système qui doivent être isolés du système hôte. La création de liens Ethernet virtuels est un outil très flexible qui permet de créer des réseaux virtuels complexes. C'est le mode par défaut pour les conteneurs démarrés par machinectl ou systemd-nspawn@.service.

Les sous-sections suivantes décrivent des scénarios courants. Voir [systemd-nspawn(1) § Options de mise en réseau](https://man.archlinux.org/man/systemd-nspawn.1#Networking_Options) pour plus de détails sur les options disponibles de *systemd-nspawn*.

#### Utiliser la mise en réseau de l'hôte

Pour désactiver la mise en réseau privée et la création d'un lien Ethernet virtuel utilisé par les conteneurs démarrés avec machinectl, ajoutez un fichier .nspawn avec l'option suivante :

    /etc/systemd/nspawn/nom-du-conteneur.nspawn

```
[Network]
VirtualEthernet=no
```

Cette option remplacera l'option `-n/--network-veth` utilisée dans `systemd-nspawn@.service` et les conteneurs nouvellement démarrés utiliseront le mode réseau de l'hôte.

#### Utiliser un lien Ethernet virtuel

Si un conteneur est démarré avec l'option `-n/--network-veth`, *systemd-nspawn* créera un lien Ethernet virtuel entre l'hôte et le conteneur.  
Le côté hôte du lien sera disponible en tant qu'interface réseau nommée `ve-container-name`   
Le côté conteneur du lien sera nommé `host0`. Notez que cette option implique `--private-network` 

Remarque :

*    Si le nom du conteneur est trop long, le nom de l'interface sera raccourci (par exemple, `ve-long-conKQGh` au lieu de `ve-long-nom-du-conteneur`) afin de respecter la [limite de 15 caractères](https://stackoverflow.com/a/29398765). Le nom complet sera défini comme la propriété `altname` de l'interface (voir [ip-link(8)](https://man.archlinux.org/man/ip-link.8)) et pourra toujours être utilisé pour référencer l'interface.
*    Lors de l'examen des interfaces avec `ip link`, les noms d'interface seront affichés avec un suffixe, comme `ve-container-name@if2` et `host0@if9`. Le `@ifN` ne fait pas partie du nom de l'interface, mais `ip link` ajoute cette information pour indiquer à quel "emplacement" le câble Ethernet virtuel se connecte à l'autre extrémité.  
Par exemple, une interface Ethernet virtuelle hôte présentée comme `ve-foo@if2` est connectée au conteneur foo, et à l'intérieur du conteneur à la deuxième interface réseau - celle présentée avec l'index 2 lors de l'exécution d'ip link à l'intérieur du conteneur. De même, l'interface nommée `host0@if9` dans le conteneur est connectée à la 9ème interface réseau de l'hôte.


Lorsque vous démarrez le conteneur, une adresse IP doit être attribuée aux deux interfaces (sur l'hôte et dans le conteneur). Si vous utilisez [systemd-networkd](https://wiki.archlinux.org/title/Systemd-networkd) sur l'hôte ainsi que dans le conteneur, ceci est fait out-of-the-box :

*    le fichier `/usr/lib/systemd/network/80-container-ve.network` sur l'hôte correspond à l'interface `ve-container-name` et démarre un serveur DHCP, qui attribue des adresses IP à l'interface de l'hôte ainsi qu'au conteneur,
*    le fichier `/usr/lib/systemd/network/80-container-host0.network` du conteneur correspond à l'interface `host0` et démarre un client DHCP qui reçoit une adresse IP de l'hôte.

Si vous n'utilisez pas *systemd-networkd*, vous pouvez configurer des adresses IP statiques ou démarrer un serveur DHCP sur l'interface hôte et un client DHCP dans le conteneur. Voir [Configuration du réseau](https://wiki.archlinux.org/title/Network_configuration) pour plus de détails.


Pour permettre au conteneur d'accéder au réseau extérieur, vous pouvez configurer NAT comme décrit dans [Partage Internet#Activer NAT](https://wiki.archlinux.org/title/Internet_sharing#Enable_NAT). Si vous utilisez *systemd-networkd*, cela est fait (partiellement) automatiquement via l'option `IPMasquerade=both` dans /usr/lib/systemd/network/80-container-ve.network. Cependant, cette option n'émet qu'une règle [iptables](https://wiki.archlinux.org/title/Iptables) (ou [nftables](https://wiki.archlinux.org/title/Nftables)) telle que


    -t nat -A POSTROUTING -s 192.168.163.192/28 -j MASQUERADE


La table de filtrage doit être configurée manuellement comme indiqué dans [Partage Internet#Activer NAT](https://wiki.archlinux.org/title/Internet_sharing#Enable_NAT).  
Vous pouvez utiliser un joker pour faire correspondre toutes les interfaces commençant par ve- :


    iptables -A FORWARD -i ve-+ -o internet0 -j ACCEPT


Note : *systemd-networkd* et *systemd-nspawn* peuvent s'interfacer avec [iptables](https://wiki.archlinux.org/title/Iptables) (en utilisant la librairie [libiptc](https://tldp.org/HOWTO/Querying-libiptc-HOWTO/whatis.html)) ainsi qu'avec [nftables](https://wiki.archlinux.org/title/Nftables) [[7]](https://github.com/systemd/systemd/issues/13307)[[8]](https://github.com/systemd/systemd/blob/9ca34cf5a4a20d48f829b2a36824255aac29078c/NEWS#L295-L304). Dans les deux cas, la NAT IPv4 et IPv6 est prise en charge.
{: .prompt-info }

De plus, vous devez ouvrir le port UDP 67 sur les interfaces `ve-+` pour les connexions entrantes vers le serveur DHCP (géré par *systemd-networkd*) :

    iptables -A INPUT -i ve-+ -p udp -m udp --dport 67 -j ACCEPT

#### Utiliser un pont réseau

Si vous avez configuré un pont réseau sur le système hôte, vous pouvez créer un lien Ethernet virtuel pour le conteneur et ajouter son côté hôte au pont réseau.  
Cette opération s'effectue à l'aide de l'option `--network-bridge=nom du pont`  
Notez que l'option `--network-bridge` implique `--network-veth`, c'est-à-dire que le lien Ethernet virtuel est créé automatiquement.  
Cependant, le côté hôte du lien utilisera le préfixe `vb-` au lieu de `ve-`, de sorte que les options *systemd-networkd* de démarrage du serveur DHCP et de masquage d'IP ne seront pas appliquées.

La gestion du pont est laissée à l'administrateur. Par exemple, le pont peut connecter des interfaces virtuelles avec une interface physique, ou il peut connecter seulement des interfaces virtuelles de plusieurs conteneurs. Voir [systemd-networkd#Network bridge with DHCP](https://wiki.archlinux.org/title/Systemd-networkd#Network_bridge_with_DHCP) et [systemd-networkd#Network bridge with static IP addresses](https://wiki.archlinux.org/title/Systemd-networkd#Network_bridge_with_static_IP_addresses) pour des exemples de configurations utilisant *systemd-networkd*.

Il existe également une option `--network-zone=zone-name` qui est similaire à `--network-bridge` mais le pont réseau est géré automatiquement par *systemd-nspawn* et *systemd-networkd*  
L'interface de pont nommée `vz-zone-name` est automatiquement créée lorsque le premier conteneur configuré avec l'option `--network-zone=zone-name` est démarré, et est automatiquement supprimée lorsque le dernier conteneur configuré avec l'option `--network-zone=zone-name` se termine.  
Cette option permet donc de placer facilement plusieurs conteneurs apparentés sur un réseau virtuel commun.  
Notez que les interfaces `vz-*` sont gérées par *systemd-networkd* de la même manière que les interfaces `ve-*` en utilisant les options du fichier `/usr/lib/systemd/network/80-container-vz.network` 

#### Utiliser une interface "macvlan" ou "ipvlan

Au lieu de créer un lien Ethernet virtuel (dont le côté hôte peut ou non être ajouté à un pont), vous pouvez créer une interface virtuelle sur une interface physique existante (c'est-à-dire une interface [VLAN](https://wiki.archlinux.org/title/VLAN)) et l'ajouter au conteneur.  
L'interface virtuelle sera pontée avec l'interface hôte sous-jacente et le conteneur sera donc exposé au réseau extérieur, ce qui lui permet d'obtenir une adresse IP distincte via DHCP à partir du même réseau local que celui auquel l'hôte est connecté.

*systemd-nspawn* propose 2 options :

*    `--network-macvlan=interface` - l'interface virtuelle aura une adresse MAC différente de l'interface physique sous-jacente et sera nommée `mv-interface` 
*    `--network-ipvlan=interface` - l'interface virtuelle aura la même adresse MAC que l'interface physique sous-jacente et sera nommée `iv-interface` 

Les deux options impliquent `--private-network` 

#### Utiliser une interface existante

Si le système hôte possède plusieurs interfaces réseau physiques, vous pouvez utiliser l'option `--network-interface=interface` pour affecter une interface au conteneur (et la rendre indisponible pour l'hôte pendant le démarrage du conteneur).  
Notez que `--network-interface` implique `--private-network`

Note : Le passage d'interfaces réseau sans fil aux conteneurs systemd-nspawn n'est actuellement pas supporté [[9]](https://github.com/systemd/systemd/issues/7873)
{: .prompt-warning }

### Mappage des ports

Lorsque le réseau privé est activé, les ports individuels de l'hôte peuvent être mappés aux ports du conteneur en utilisant l'option `-p/--port` ou en utilisant le paramètre [Port](URL) dans un fichier *.nspawn*. Cela se fait en émettant des règles [iptables](https://wiki.archlinux.org/title/Iptables) dans la table nat, mais la chaîne `FORWARD` dans la table de filtrage doit être configurée manuellement comme indiqué dans [#Use a virtual Ethernet link (Utiliser un lien Ethernet virtuel)](https://wiki.archlinux.org/title/Systemd-nspawn#Use_a_virtual_Ethernet_link).

Par exemple, pour mapper un port TCP 8000 sur l'hôte au port TCP 80 dans le conteneur :

    /etc/systemd/nspawn/nom-du-conteneur.nspawn

```
[Network]
Port=tcp:8000:80
```

Remarque : systemd-nspawn exclut explicitement l'interface `loopback` lors du mappage des ports. Ainsi, dans l'exemple ci-dessus, localhost:8000 se connecte à l'hôte et non au conteneur. Seules les connexions vers d'autres interfaces sont soumises au mappage des ports. Voir [[10]](https://github.com/systemd/systemd/issues/6106) pour plus de détails.
{: .prompt-warning }

### Résolution nom domaine

La résolution des noms de domaine dans le conteneur peut être configurée de la même manière que sur le système hôte. En outre, systemd-nspawn fournit des options pour gérer le fichier /etc/resolv.conf à l'intérieur du conteneur :

*    `--resolv-conf` peut être utilisé en ligne de commande
*    `ResolvConf=` peut être utilisé dans les fichiers *.nspawn*

Ces options correspondantes ont de nombreuses valeurs possibles qui sont décrites dans [systemd-nspawn(1) § Options d'intégration](https://man.archlinux.org/man/systemd-nspawn.1#Integration_Options).  
La valeur par défaut est `auto`, ce qui signifie que :

*    Si `--private-network` est activé, le fichier `/etc/resolv.conf` est laissé tel quel dans le conteneur.
*    Sinon, si [systemd-resolved](https://wiki.archlinux.org/title/Systemd-resolved) est en cours d'exécution sur l'hôte, son fichier stub `resolv.conf` est copié ou monté dans le conteneur.
*    Sinon, le fichier` /etc/resolv.conf` est copié ou monté de l'hôte vers le conteneur.

Dans les deux derniers cas, le fichier est copié si la racine du conteneur est accessible en écriture, et monté par liaison s'il est en lecture seule.

Dans le second cas où [systemd-resolved](https://wiki.archlinux.org/title/Systemd-resolved) s'exécute sur l'hôte, *systemd-nspawn* s'attend à ce qu'il s'exécute également dans le conteneur, afin que ce dernier puisse utiliser le fichier de liens symboliques `/etc/resolv.conf` de l'hôte. Si ce n'est pas le cas, la valeur par défaut `auto` ne fonctionne plus et vous devez remplacer le lien symbolique en utilisant l'une des options `replace-*` 

