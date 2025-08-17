+++
title = 'Parefeu (firewall) UFW'
date = 2023-09-19 00:00:00 +0100
categories = ['parefeu']
+++
*UFW, ou pare - feu simple , est une interface pour gérer les règles de pare-feu dans Arch Linux, Debian ou Ubuntu. UFW est utilisé via la ligne de commande (bien qu'il dispose d'interfaces graphiques disponibles), et vise à rendre la configuration du pare-feu simple.*


## Configurer un pare-feu avec UFW

![ufw](ufw-logo-a.png){:width="100"}

### Mise à jour système

    sudo pacman -Syu	# Arch Linux    
    sudo apt update && sudo apt upgrade # Debian / Ubuntu

### Installer UFW

Debian démarrera automatiquement l'unité systemd d'UFW et lui permettra de démarrer au redémarrage, mais Arch ne le fera pas. Ce n'est pas la même chose que de dire à UFW d'activer les règles de pare-feu , car l'activation d'UFW avec systemd ou upstart indique uniquement au système init d'activer le démon UFW.

Par défaut, les jeux de règles d'UFW sont vides, de sorte qu'il n'applique aucune règle de pare-feu, même lorsque le démon est en cours d'exécution.   

**Arch Linux**

    sudo pacman -S ufw

Démarrez et activez l'unité systemd d'UFW:

    sudo systemctl start ufw
    sudo systemctl enable ufw

**Debian / Ubuntu**

    sudo apt-get install ufw

### Définir les règles par défaut

La plupart des systèmes n'ont besoin que d'un petit nombre de ports ouverts pour les connexions entrantes, et tous les ports restants fermés. Pour commencer avec une base simple de règles, la ufw defaultcommande peut être utilisée pour définir la réponse par défaut aux connexions entrantes et sortantes. Pour refuser toutes les connexions entrantes et autoriser toutes les connexions sortantes, exécutez:

    sudo ufw default allow outgoing
    sudo ufw default deny incoming

`ufw default` permet également l'utilisation du paramètre `reject`

>Mise en garde  
La configuration d'une règle de rejet ou de refus par défaut peut vous bloquer sauf si des règles d'autorisation explicites sont en place. Assurez-vous que vous avez configuré les règles d'autorisation pour SSH et d'autres services critiques conformément à la section ci-dessous avant d'appliquer les règles de refus ou de rejet par défaut.

### Ajout de règle

Les règles peuvent être ajoutées de deux manières: en indiquant le <u>numéro de port</u> ou en utilisant le <u>nom du service</u> .  

### Règles de base

#### refuser le trafic sur un certain port

De même, pour refuser le trafic sur un certain port (dans cet exemple, 111), vous n'auriez qu'à exécuter:

    sudo ufw deny 111

#### Autoriser les paquets basés sur TCP ou UDP

Pour affiner davantage vos règles, vous pouvez également autoriser les paquets basés sur TCP ou UDP. Les éléments suivants autoriseront les paquets TCP sur le port 80:

    sudo ufw allow 80/tcp
    sudo ufw allow http/tcp

Alors que cela autorisera les paquets UDP sur 1725:

    sudo ufw allow 1725/udp

#### Autoriser les connexions à partir d'une adresse IP

En plus d'autoriser ou de refuser basé uniquement sur le port, UFW vous permet également d'autoriser/bloquer par les adresses IP, les sous-réseaux et une combinaison adresse IP/sous-réseau/port.

Pour autoriser les connexions à partir d'une **adresse IP**:

    sudo ufw allow from 198.51.100.0

#### Autoriser les connexions à partir d'une adresse IP et Port

Pour autoriser une combinaison **adresse IP/port spécifique**:

    sudo ufw allow from 198.51.100.0 to any port 22 proto tcp

`proto tcp` peut être supprimé ou basculé en `proto udp` suivant vos besoins, et toutes les instances `allow` peuvent être modifiées en `deny` en fonction des besoins.

#### Autoriser les connexions à partir d'un sous-réseau spécifique

Pour autoriser les connexions à partir d'un **sous-réseau spécifique**:

    sudo ufw allow from 198.51.100.0/24

#### Bloquer une adresse IP

Pour bloquer toutes les connexions réseau qui proviennent d’une adresse IP spécifique, 15.15.15.51 par exemple, exécutez la commande suivante :

    sudo ufw deny from 15.15.15.51

Dans cet exemple, from 15.15.15.51 spécifie une adresse IP source de « 15.15.15.51 ». Si vous le souhaitez, vous pouvez spécifier ici un sous-réseau, tel que 15.15.15.0/24 à la place. L’adresse IP source peut être spécifiée dans toute règle de pare-feu, notamment une règle allow.

#### Bloquer les connexions à une interface réseau

Pour bloquer les connexions d’une adresse IP spécifique, par exemple 15.15.15.51 à une interface réseau spécifique, par exemple eth0, utilisez la commande suivante :

    sudo ufw deny in on eth0 from 15.15.15.51

C’est la même chose que dans l’exemple précédent, en ajoutant in on eth0. Vous pouvez spécifier l’interface réseau dans n’importe quelle règle de pare-feu. C’est un excellent moyen de limiter la règle à un réseau spécifique.

### Règle SSH

#### Autoriser SSH

pour autoriser les connexions entrantes et sortantes sur le port 22 pour SSH, vous pouvez exécuter:

    sudo ufw allow ssh

Vous pouvez également exécuter:

    sudo ufw allow 22

#### Autoriser un SSH entrant avec une adresse IP ou un sous-réseau spécifique

Pour autoriser les connexions SSH entrantes à partir d’une adresse IP ou d’un sous-réseau spécifique, vous devez spécifier la source. Par exemple, si vous souhaitez autoriser l’intégralité du sous-réseau 15.15.15.0/24, vous devez exécuter la commande suivante :

    sudo ufw allow from 15.15.15.0/24  to any port 22

### Règle rsync

Si vous utilisez un serveur cloud, vous voudrez probablement autoriser les connexions SSH entrantes (port 22) afin de pouvoir vous connecter à votre serveur et le gérer. Cette section vous montre de quelle manière configurer votre pare-feu avec plusieurs règles liées au SSH.

#### Autoriser un Rsync entrant avec une adresse IP ou un sous-réseau spécifique

Vous pouvez utiliser Rsync, qui fonctionne sur le port 873, pour transférer des fichiers d’un ordinateur à un autre.

Pour autoriser les connexions rsync entrantes à partir d’une adresse IP ou d’un sous-réseau spécifique, vous devez spécifier l’adresse IP source et le port de destination. Par exemple, si vous souhaitez autoriser que l’intégralité du sous-réseau 15.15.15.0/24 soit en mesure de rsync sur votre serveur, exécutez la commande suivante :

    sudo ufw allow from 15.15.15.0/24 to any port 873

### Règle MySQL

#### Autoriser MySQL avec une adresse IP ou un sous-réseau spécifique

Pour autoriser les connexions MySQL entrantes à partir d’une adresse IP ou d’un sous-réseau spécifique, vous devez spécifier la source. Par exemple, si vous souhaitez autoriser l’intégralité du sous-réseau 15.15.15.0/24, vous devez exécuter la commande suivante :

    sudo ufw allow from 15.15.15.0/24 to any port 3306

Autoriser MySQL sur l’interface réseau spécifique

Disons que vous disposez par exemple d’une interface réseau privée eth1, vous devez utiliser la commande suivante pour autoriser les connexions MySQL sur une interface réseau spécifique :

    sudo ufw allow in on eth1 to any port 3306

### Règle PostgreSQL

#### PostgreSQL avec une  IP ou un sous-réseau spécifique

Pour autoriser les connexions PostgreSQL entrantes à partir d’une adresse IP ou d’un sous-réseau spécifique, vous devez spécifier la source. Par exemple, si vous souhaitez autoriser l’intégralité du sous-réseau 15.15.15.0/24, vous devez exécuter la commande suivante :

    sudo ufw allow from 15.15.15.0/24 to any port 5432

La deuxième commande qui autorise le trafic sortant des connexions PostgreSQL established, n’est nécessaire que si la politique OUTPUT n’est pas défini sur ACCEPT.
Autoriser PostgreSQL sur une interface réseau spécifique

Disons que vous disposez par exemple d’une interface réseau privée eth1, vous devez utiliser la commande suivante pour autoriser les connexions PostgreSQL sur une interface réseau spécifique :

    sudo ufw allow in on eth1 to any port 5432

La deuxième commande qui autorise le trafic sortant des connexions PostgreSQL established, n’est nécessaire que si la politique OUTPUT n’est pas défini sur ACCEPT.

### Règles HTTP HTTPS

Les serveurs Web, comme Apache et Nginx, écoutent généralement les requêtes des connexions HTTP et HTTPS sur le port 80 et 443, respectivement. Si votre politique pour le trafic entrant est configurée sur drop ou deny par défaut, il vous faudra créer des règles qui permettront à votre serveur de répondre à ces requêtes.

#### Autoriser tous les HTTP entrants

Pour autoriser toutes les connexions HTTP entrantes (port 80), exécutez la commande suivante :

    sudo ufw allow http

Vous pouvez également utiliser une autre syntaxe en spécifiant le numéro de port du service HTTP :

    sudo ufw allow 80

#### Autoriser tous les HTTP entrants

Exécutez la commande suivante pour autoriser toutes les connexions HTTPS entrantes (port 443) :

    sudo ufw allow https

Vous pouvez également utiliser une autre syntaxe en spécifiant le numéro de port du service HTTPS :

    sudo ufw allow 443

#### Autoriser tout le trafic HTTP et HTTPS entrant

Si vous souhaitez autoriser le trafic à la fois HTTP et HTTPS, vous pouvez créer une règle unique qui autorise les deux ports. Pour autoriser toutes les connexions HTTP et HTTPS entrantes (port 443), exécutez la commande suivante :

    sudo ufw allow proto tcp from any to any port 80,443

>Notez que vous devez indiquer le protocole en utilisant proto tcp lorsque vous spécifiez plusieurs ports.

### Règles Mail

Les serveurs de courrier électronique, comme Sendmail et Postfix, écoutent sur une variété de ports en fonction des protocoles utilisés pour livrer les courriels. Si vous utilisez un serveur de messagerie, vous devez déterminer les protocoles que vous utilisez et autoriser les types de trafic applicables. Nous allons également vous montrer de quelle manière créer une règle pour bloquer les courriels SMTP sortants.

#### Bloquer un courriel SMTP sortant

Si votre serveur ne doit pas envoyer de courriel sortant, il vous faudra bloquer ce type de trafic. Pour bloquer les courriels SMTP sortants, qui utilise le port 25, exécutez la commande suivante :

    sudo ufw deny out 25

Vous configurez votre pare-feu de sorte qu’il drop tout le trafic sortant sur le port 25. Si vous devez rejeter un autre service en fonction de son numéro de port, vous pouvez l’utiliser à la place du port 25, vous devez simplement le remplacer.

#### Autoriser tous les SMTP entrants

Pour permettre à votre serveur de répondre aux connexions SMTP, port 25, exécutez la commande suivante :

    sudo ufw allow 25

Remarque : il est courant que les serveurs SMTP utilisent le port 587 pour le courrier sortant.

#### Autoriser tous les IMAP entrants

Pour permettre à votre serveur de répondre aux connexions IMAP, port 143, exécutez la commande suivante :

    sudo ufw allow 143

#### Autoriser tous les IMAPS entrants

Pour permettre à votre serveur de répondre aux connexions IMAPS, port 993, exécutez la commande suivante :

    sudo ufw allow 993

#### Autoriser tous les POP3 entrants

Pour permettre à votre serveur de répondre aux connexions POP3, port 110, exécutez la commande suivante :

    sudo ufw allow 110

#### Autoriser tous les POP3S entrants

Pour permettre à votre serveur de répondre aux connexions POP3S, port 995, exécutez la commande suivante :

    sudo ufw allow 995

### Supprimer les règles

#### Lister les règles UFW

L'une des tâches les plus courantes lors de la gestion d'un pare-feu consiste à répertorier les règles.  
Vous pouvez vérifier l'état de l'UFW et répertorier toutes les règles avec:

    sudo ufw status

Si UFW est désactivé, vous verrez quelque chose comme ceci:

    Status: inactive

Sinon, si UFW est actif, la sortie imprimera une liste de toutes les règles de pare-feu actives:

    Status: active To Action From -- ------ ---- 22/tcp 
    ALLOW Anywhere 22/tcp (v6) 
    ALLOW Anywhere (v6)

Pour obtenir des informations supplémentaires, utilisez le status verbose :

    sudo ufw status verbose

La sortie comprendra des informations sur la journalisation, les stratégies par défaut et les nouveaux profils:

    Status: active Logging: on (low) 
    Default: deny (incoming), allow (outgoing), disabled (routed) 
    New profiles: skip To Action From -- ------ ---- 22/tcp 
    ALLOW Anywhere 22/tcp (v6) 
    ALLOW Anywhere (v6)

Utilisez `status numbered` pour obtenir l'ordre et le numéro d'identification de toutes les règles actives.   
Ceci est utile lorsque vous souhaitez insérer une nouvelle règle numérotée ou supprimer une règle existante en fonction de son numéro.

    sudo ufw status numbered

    Status: active To Action From -- ------ ---- 22/tcp 
    ALLOW IN Anywhere 22/tcp (v6) 
    ALLOW IN Anywhere (v6)

Il existe deux façons de supprimer des règles UFW:

* Par numéro de règle 
* Par spécification

La suppression de règles UFW par le numéro de règle est plus facile car il vous suffit de rechercher et de taper le numéro de la règle que vous souhaitez supprimer, pas la règle complète.

#### Supprimer une règle UFW par numéro

Pour supprimer une règle UFW par son numéro, vous devez d'abord répertorier les règles et trouver le numéro de la règle que vous souhaitez supprimer:

    sudo ufw status numbered

La commande vous donnera une liste de toutes les règles de pare-feu et leurs numéros:

```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 55036/tcp                  ALLOW IN    Anywhere                  
[ 2] 80/tcp                     ALLOW IN    Anywhere                  
[ 3] 443/tcp                    ALLOW IN    Anywhere                  
[ 4] DNS                        ALLOW IN    Anywhere                  
[ 5] 51820/udp                  ALLOW IN    Anywhere                  
[ 6] 55036/tcp (v6)             ALLOW IN    Anywhere (v6)             
[ 7] 80/tcp (v6)                ALLOW IN    Anywhere (v6)             
[ 8] 443/tcp (v6)               ALLOW IN    Anywhere (v6)             
[ 9] DNS (v6)                   ALLOW IN    Anywhere (v6)             
[10] 51820/udp (v6)             ALLOW IN    Anywhere (v6)             
```

Une fois que vous connaissez le numéro de règle, utilisez la commande ufw delete suivie du numéro de la règle que vous souhaitez supprimer.


Par exemple, pour supprimer la règle avec le numéro 4, vous devez taper:

    sudo ufw delete 5

Vous serez invité à confirmer que vous souhaitez supprimer la règle:

    Deleting: allow 51820/udp Proceed with operation (y|n)? y

Tapez y, appuyez sur Enter et la règle sera supprimée:

    Rule deleted

Chaque fois que vous supprimez une règle, le numéro de règles change. Pour être sûr, répertoriez toujours les règles avant de supprimer une autre règle.
{: .prompt-warning }

#### Supprimer une règle UFW par son nom

La deuxième méthode pour supprimer une règle consiste à utiliser la commande `ufw delete` suivie de la règle.

Par exemple, si vous avez ajouté une règle qui ouvre le port 2222, à l'aide de la commande suivante:

    sudo ufw allow 2222

Vous pouvez supprimer la règle en tapant:

    sudo ufw delete allow 2222

#### Supprimer les règles et désactiver UFW

La réinitialisation d'UFW désactivera le pare-feu et supprimera toutes les règles actives.  
Ceci est utile lorsque vous souhaitez annuler toutes vos modifications et recommencer à zéro.

Pour réinitialiser UFW, tapez la commande suivante:

    sudo ufw reset

### Modifier les fichiers de configuration d'UFW

Bien que des règles simples puissent être ajoutées via la ligne de commande, il peut y avoir un moment où des règles plus avancées ou spécifiques doivent être ajoutées ou supprimées. Avant d'exécuter les règles entrées via le terminal, UFW exécutera un fichier `before.rules`, qui permet le bouclage, le ping et DHCP. Pour ajouter pour modifier ces règles, modifiez le fichier **/etc/ufw/before.rules**. Un fichier `before6.rules` se trouve également dans le même répertoire pour IPv6.

Un `after.rule` et un `after6.rule` existent également pour ajouter toutes les règles qui devraient être ajoutées après qu'UFW exécute vos règles ajoutées en ligne de commande.

Un fichier de configuration supplémentaire se trouve à l'adresse **/etc/default/ufw**. À partir de là, IPv6 peut être désactivé ou activé, des règles par défaut peuvent être définies et UFW peut être défini pour gérer les chaînes de pare-feu intégrées.

### Statut UFW

Vous pouvez vérifier l'état de UFW à tout moment avec la commande: `sudo ufw status`  
Cela affichera une liste de toutes les règles, et si UFW est actif ou non:

```
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443                        ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443 (v6)                   ALLOW       Anywhere (v6)
```

### Activer le pare-feu

Avec les règles que vous avez choisies en place, vous allez obtenir **Status: inactive**  
Pour activer UFW et appliquer vos règles de pare-feu:

    sudo ufw enable

De même, pour désactiver les règles d'UFW:

    sudo ufw disable

>Remarque : Cela laisse toujours le service UFW en cours d'exécution et activé lors des redémarrages.

### Enregistrement (logs)

Vous pouvez activer la journalisation 

    sudo ufw logging on

Pour désactiver la journalisation 

    sudo ufw logging off

Niveaux journaux peuvent être réglés en cours d' exécution `sudo ufw logging low|medium|high`, soit en sélectionnant low, mediumou highdans la liste. Le réglage par défaut est `low`.

Une entrée de journal normale ressemblera à ce qui suit et sera située dans **/var/logs/ufw**:

    Sep 16 15:08:14 <hostname> kernel: [UFW BLOCK] IN=eth0 OUT= MAC=00:00:00:00:00:00:00:00:00:00:00:00:00:00 SRC=123.45.67.89 DST=987.65.43.21 LEN=40 TOS=0x00 PREC=0x00 TTL=249 ID=8475 PROTO=TCP SPT=48247 DPT=22 WINDOW=1024 RES=0x00 SYN URGP=0

Les valeurs initiales répertorient la date, l'heure et le nom d'hôte de votre serveur.Les autres valeurs importantes incluent:

*    **UFW BLOCK**: Cet emplacement est l'endroit où se trouvera la description de l'événement enregistré. Dans ce cas, il a bloqué une connexion.
*    **IN**: si cela contient une valeur, l'événement était alors entrant
*    **OUT**: Si cela contient une valeur, l'événement était sortant
*    **MAC**: une combinaison des adresses MAC de destination et de source
*    **SRC**: l'IP de la source du paquet
*    **DST**: l'IP de la destination du paquet
*    **LEN**: longueur du paquet
*    **TTL**: le paquet TTL, ou le temps de vivre . Combien de temps il rebondira entre les routeurs jusqu'à son expiration, si aucune destination n'est trouvée.
*    **PROTO**: Le protocole du paquet
*    **SPT**: le port source du package
*    **DPT**: port de destination du package
*    **WINDOW**: La taille du paquet que l'expéditeur peut recevoir
*    **SYN URGP**: indiqué si une prise de contact à trois voies est requise. 0signifie que ce n'est pas le cas.


Décommentez la ligne `#& stop` du fichier `/etc/rsyslog.d/20-ufw.conf` pour arrêter la journalisation de tout ce qui correspond à la dernière règle, ce qui arrêtera la journalisation des messages UFW générés par le noyau dans le fichier contenant normalement les messages kern.* (par exemple, /var/log/kern.log).
{: .prompt-info }

