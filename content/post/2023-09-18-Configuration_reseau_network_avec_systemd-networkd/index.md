+++
title = 'Configuration réseau (network) à l'aide de systemd-networkd'
date = 2023-09-18 00:00:00 +0100
categories = ['network']
+++
*systemd-networkd est un logiciel de configuration*

- [systemd-networkd](#systemd-networkd)
    - [Fichiers de configuration](#fichiers-de-configuration)
    - [Configuration](#configuration)
        - [Configuration manuelle des adresses IP](#configuration-manuelle-des-adresses-ip)
        - [Modification de l'adresse IPv4 principale](#modification-de-ladresse-ipv4-principale)
        - [Configuration de l'adresse IPv4 principale via DHCP](#configuration-de-ladresse-ipv4-principale-via-dhcp)
        - [Configuration de l'adresse IPv6 principale via SLAAC](#configuration-de-ladresse-ipv6-principale-via-slaac)
        - [Configuration d'adresses IP supplémentaires](#configuration-dadresses-ip-supplémentaires)
        - [Changer les résolveurs DNS](#changer-les-résolveurs-dns)

## systemd-networkd

### Fichiers de configuration

Voici des détails concernant les fichiers de configuration réseau pour systemd-networkd, y compris des informations sur l'emplacement du fichier de configuration par défaut.

*    Extension de fichier: `.network`
*    Emplacement du fichier: `/etc/systemd/network/`
*    Convention de dénomination : `[priority]-[interface].network` , avec `[priority]` étant utilisé pour classer les fichiers (les fichiers sont traités de manière alphanumérique*) et `[interface]` offrant un moyen pratique pour un utilisateur d'associer un fichier à une interface particulière.
*    Fichier de configuration par défaut : `/etc/systemd/network/05-eth0.network` 

Lorsque systemd-networkd affiche les interfaces réseau, les fichiers de configuration sont traités de manière alphanumérique. En tant que tel, vous verrez que les fichiers sont généralement précédés d'un numéro à 2 chiffres pour faciliter leur classement. Le fichier de configuration par défaut est précédé de 05. Si nous voulions créer un fichier de configuration pour une interface différente, nous pourrions le faire précéder d'un numéro ci-dessous 05(à traiter avant) ou supérieur (à traiter après).
{: .prompt-info }

### Configuration 

Voici un exemple de fichier de configuration typique pour systemd-networkd. Il définit statiquement l'adresse IPv4 et permet à SLAAC de configurer l'adresse IPv6.

Fichier : /etc/systemd/network/05-eth0.network

```
    [Match]
    Name=eth0

    [Network]
    DHCP=no
    DNS=203.0.113.1 203.0.113.2 203.0.113.3
    Domains=ip.linodeusercontent.com
    IPv6PrivacyExtensions=false

    Gateway=192.0.2.1
    Address=192.0.2.123/24
```

*    Name :eth0, l'interface par défaut configurée pour l'Internet public sur la plupart des instances de calcul. Lors de l'utilisation d'un VLAN, l'interface Internet publique peut être configurée différemment.
*    DHCP :no, qui désactive DHCP et vous permet de définir statiquement l'adresse IPv4 principale dans les champs ultérieurs.
*    DNS : Une liste d'adresses IP mappées aux résolveurs DNS de Linode. Les adresses IP fournies dans cet exemple sont des espaces réservés et ne fonctionnent pas.
*    Domains :ip.linodeusercontent.com, qui est défini comme un « domaine de recherche ». Il s'agit d'un moyen rapide de convertir des noms d'hôte simples en noms de domaine complets, mais ce n'est pas souvent nécessaire.
*    IPv6PrivacyExtensions :false, qui désactive les extensions de confidentialité et aide à résoudre tout problème lié à la configuration automatique de votre adresse IPv6 SLAAC.
*    Gateway : configure statiquement l'adresse de la passerelle IPv4.
*    Address : configure statiquement l'adresse IPv4.

#### Configuration manuelle des adresses IP

Les paramètres

*        Adresse(s) IPv4 publique(s) et passerelle IPv4 associée
*        Adresse IPv4 privée (si une a été ajoutée)
*        Adresse IPv6 SLAAC et passerelle IPv6 associée
*        Plage acheminée IPv6 /64 ou /56 (si une a été ajoutée)
*        Résolveurs DNS 

fichier de configuration réseau

    sudo nano /etc/systemd/network/05-eth0.network

Une fois que vous avez modifié le fichier de configuration pour l'adapter à vos besoins, vous devez redémarrez le service :

    sudo systemctl restart systemd-networkd

#### Modification de l'adresse IPv4 principale

Pour modifier l'adresse IPv4 configurée sur le système, définissez les paramètres **Gateway** et **Address** pour qu'ils correspondent à la nouvelle adresse IP et à l'adresse IP de la passerelle correspondante.

Fichier : /etc/systemd/network/05-eth0.network

```
...
Gateway=192.0.2.1
Address=192.0.2.123/24
```

#### Configuration de l'adresse IPv4 principale via DHCP

DHCP peut être utilisé pour configurer automatiquement votre adresse IPv4 principale. L'adresse IPv4 principale est définie comme l'adresse IPv4 attribuée à votre système qui se trouve en première position lorsqu'elle est triée numériquement. Pour activer DHCP, définissez le paramètre DHCP sur oui et supprimez (ou commentez) les lignes qui définissent la passerelle et l'adresse de l'adresse IPv4 principale.

Fichier : /etc/systemd/network/05-eth0.network

```
...
[Network]
DHCP=yes
...
# Gateway=192.0.2.1
# Address=192.0.2.123/24
```

Important :Lorsque vous utilisez DHCP, l'adresse IPv4 configurée sur votre système peut changer si vous ajoutez ou supprimez des adresses IPv4 sur votre instance de calcul. Si cela se produit, tout outil ou système utilisant l'adresse IPv4 d'origine ne pourra plus se connecter.
{: .prompt-warning }

#### Configuration de l'adresse IPv6 principale via SLAAC

SLAAC est utilisé pour configurer automatiquement votre adresse IPv6 principale. Pour que cela fonctionne, votre système doit accepter les publicités du routeur. Vous devrez peut-être également désactiver les extensions de confidentialité IPv6. Dans systemd-networkd, cela signifie définir IPv6PrivacyExtensions sur false et IPv6AcceptRA sur true.

Fichier : /etc/systemd/network/05-eth0.network

```
...
[Network]
...
IPv6PrivacyExtensions=false
IPv6AcceptRA=true
```

Le paramètre **IPv6AcceptRA** n'est pas strictement requis tant que l'exécution de la variable du noyau **net.ipv6.conf.eth0.autoconf** est définie sur 1 (et non sur 0). Vous pouvez déterminer le paramètre en exécutant la commande suivante : `sysctl net.ipv6.conf.eth0.autoconf` 
{: .prompt-info }

Si vous souhaitez désactiver l'adressage IPv6 SLAAC et configurer à la place votre adresse IPv6 de manière statique (non recommandé), vous pouvez définir explicitement le paramètre I**Pv6AcceptRA** sur **false**, puis ajouter votre adresse IPv6 principale (en utilisant le préfixe /128).

Fichier : /etc/systemd/network/05-eth0.network

```
...
IPv6AcceptRA=false
Address=[ip-address]/128
```

#### Configuration d'adresses IP supplémentaires

Des adresses IP supplémentaires peuvent être configurées en ajoutant un autre paramètre d'adresse dans la section [Réseau] du fichier de configuration.

Fichier : /etc/systemd/network/05-eth0.network

    ...
    Adresse=[adresse IP]/[préfixe]

Dans l'exemple ci-dessus, effectuez les remplacements suivants :

*    [adresse IP] : L'adresse IP que vous souhaitez configurer de manière statique. Si vous configurez une adresse à partir d’une plage IPv6, vous pouvez choisir n’importe quelle adresse dans cette plage. Par exemple, dans la plage 2001:db8:e001:1b8c::/64, l'adresse 2001:db8:e001:1b8c::1 peut être utilisée.
*    [préfixe] : le préfixe est basé sur le type d'adresse IP que vous ajoutez :
*        Adresse IPv4 publique : /24
*        Adresse IPv4 privée : /17
*        Adresse IPv6 SLAAC : /128 (bien qu'il soit recommandé de la configurer automatiquement via SLAAC)
*        Adresse IPv6 parmi une plage : /64 ou /56 (selon la taille de la plage)

#### Changer les résolveurs DNS

Les résolveurs DNS sont les entités qui résolvent les noms de domaine en leur adresse IPv4 correspondante. Par défaut, l'instance de calcul doit utiliser les résolveurs DNS du centre de données dans lequel elle réside. Vous pouvez les modifier en définissant le paramètre DNS sur une liste délimitée par des espaces des adresses IP de vos résolveurs DNS préférés.

Fichier : /etc/systemd/network/05-eth0.network

```
...
DNS=203.0.113.1 203.0.113.2 203.0.113.3
```

Dans l'exemple ci-dessus, remplacez les adresses IP fournies par les adresses IP des résolveurs DNS que vous souhaitez utiliser. Les adresses IPv4 et IPv6 peuvent être utilisées ensemble.