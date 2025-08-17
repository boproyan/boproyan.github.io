+++
title = 'Docker - Mise à jour automatique avec Watchtower'
date = 2025-06-05 13:00:00
categories = ['virtuel']
+++
*Watchtower est un container puissant et pratique pour automatiser la gestion des mises à jour sur Docker, l’ensemble de vos containers reste ainsi à jour tant pour les fonctionnalités que pour les mises à jour de sécurité.*

![image](docker-logo.png){: .normal}   

## Watchtower

* [WATCHTOWER (2019)](https://www.nas-forum.com/forum/topic/63740-tuto-mise-%C3%A0-jour-automatique-des-images-et-conteneurs-docker/)
* [Mettre à jour automatiquement ses containers Docker avec Watchtower](https://www.eliastiksofts.com/blog/2024/06/mettre-a-jour-automatiquement-ses-containers-docker-avec-watchtower)
* [Surveiller les mises à jour des conteneurs Docker avec Watchtower](https://belginux.com/surveiller-les-mises-a-jour-des-conteneurs-docker-avec-watchtower/)
* [Watchtower : mise à jour auto de vos containers Docker](https://www.antoineguilbert.fr/watchtower-mise-a-jour-auto-de-vos-containers-docker/)

### 1. Préambule

Le but de ce tutoriel est de mettre en place une application, `containrrr/watchtower` (Docker Hub, Github) qui va vérifier suivant un intervalle que vous définirez la présence d'une nouvelle version pour totalité ou partie de vos images Docker.  


Avantages :

- Plus besoin de vérifier via Docker Hub si une mise à jour est disponible
- Processus entièrement automatisé
- Ne consomme quasi aucune ressource

Inconvénients :

- Nécessite un accès à docker.sock, d'un point de vue sécurité ça peut être gênant pour certains
- Suivant l'intervalle de vérification que vous définissez, la quantité de données échangée peut être importante, à surveiller si vous avez un quota
- On ne peut l'installer par l'interface Docker de DSM (en fait si, mais c'est fastidieux).

Je pondérerai le volet concernant la sécurité, si vous avez bien respecté les conseils de sécurité développés dans les tutoriels de référence de ce forum vous ne devriez pas avoir de souci à vous faire.

### 2. Prérequis

- Savoir se connecter en SSH (root)
- Avoir Docker installé sur son NAS

C'est parti !

On commence par se connecter en SSH avec l'utilisateur root (voir le tutoriel  [TUTO] Accès SSH et ROOT via DSM 6).
On télécharge l'image :

docker pull containrrr/watchtower

Il ne reste plus qu'à créer le conteneur, mais il est avant tout intéressant de voir les fonctionnalités que l'application propose dans la documentation, et en particulier sur cette page.
C3es paramètres sont personnalisés par l'ajout de variables d'environnement à la création du conteneur.

### 3. Configuration

#### 3-A. Fréquence de mise à jour

- **INTERVAL** : Définit un intervalle de mise à jour en secondes.  
Ex : WATCHTOWER_POLL_INTERVAL=300  
- **SCHEDULING** : Le plus intéressant pour des particuliers je trouve, cron permet de définir une périodicité très personnalisable : tous les x jours, tous les mardi, 3 fois par jour le mercredi et le vendredi, à telle heure, x fois par mois, etc... la documentation de l'image renvoie vers ce site, mais je trouve ce générateur ou celui-ci très pratiques. L'image utilise un format CRON à 6 champs, contrairement aux liens ci-avant qui n'utilisent que 5 champs, en réalité, tous les champs sont décalés vers la droite pour proposer un réglage sur les secondes.  
Donc au lieu d'avoir :  
`minutes | heures | jour du mois | mois | jour de la semaine vous avez secondes | minutes | heures | jour du mois | mois | jour de la semaine`  
Ex : `WATCHTOWER_SCHEDULE=0 0 5 * * 6`  
Cela correspond à une exécution hebdomadaire, tous les samedi matin à 5:00  
Un conseil : Évitez d'utiliser 2:00 ou 3:00, à cause des changements d'heure 😉 

Ces deux méthodes sont exclusives, c'est donc l'une ou l'autre.

#### 3-B. Tri sélectif

Par défaut, watchtower surveille tous les conteneurs. Mais on ne souhaite pas spécialement mettre à jour tous ses conteneurs automatiquement, certains sont gérés par des scripts de mise à jour (Bitwarden par exemple), d'autres sont peu maintenus, et il est plus risqué de mettre à jour aveuglément ce type d'images que celles issues de collectifs reconnus (Linuxserver, Bitnami, etc...). Plusieurs méthodes de tri sont proposées :

- **LABEL ENABLE** : Permet d'utiliser les labels (des tags en quelque sorte) comme signes de distinction. Lorsqu'on crée un conteneur, on peut décider de lui ajouter un label, composé d'un intitulé et d'une valeur de type chaîne ou entier. Pour permettre à watchtower de mettre à jour un conteneur donné, il doit avoir parmi ses labels :  
`com.centurylinklabs.watchtower.enable=true`  
L' avantage énorme c'est que le nom du conteneur cible n'a pas d'importance, le fait qu'il n'existerait plus à un instant t n'a pas davantage d'incidence.
Pour faire en sorte que la sélection des conteneurs à surveiller se fasse par label, il faut ajouter la variable suivante à la création du conteneur watchtower :  
`WATCHTOWER_LABEL_ENABLE=true`

- **FULL EXCLUDE** (Ce n'est pas une variable d'environnement !!) : L'approche est exclusive au lieu d'être inclusive, elle se base sur le comportement par défaut de watchtower (surveillance de tous les conteneurs), mais il n'y a ici aucune variable d'environnement à préciser à la création du conteneur watchtower, en revanche les conteneurs que vous ne souhaiteriez pas surveiller doivent avoir le label suivant :  
`com.centurylinklabs.watchtower.enable=false`  

Notes :

- *Il n'est pas possible d'ajouter un label à un conteneur déjà existant*. Il faut le recréer et insérer le label dans le script de création du conteneur. Si vous savez les recréer facilement (peu de paramètres à entrer, vos volumes sont persistants et les fichiers de configuration sont maintenus à la suppression du conteneur, ou si vous utilisez docker-compose) alors je préconise la méthode des labels, dans le cas contraire préférez la méthode FULL EXCLUDE.
- La documentation est particulièrement claire à ce sujet.

#### 3-C. Gestion des images

- **CLEANUP** : Supprime la version précédente de l'image quand une plus récente a été téléchargée et qu'un nouveau conteneur a été créé. Cette fonctionnalité est désactivée par défaut.
Ex :` WATCHTOWER_CLEANUP=true`
- **ROLLING RESTART** : Par défaut, watchtower met à jour toutes les images avant de relancer les conteneurs, cette variable d'environnement permet de relancer les conteneurs au fur et à mesure, utilise si on souhaite réduire les périodes d'indisponibilités.
Ex : `WATCHTOWER_ROLLING_RESTART=true`
- **WAIT UNTIL TIMEOUT** : Par défaut, watchtower attend 10 secondes qu'un conteneur s'arrête une fois le signal d'extinction transmis. Certaines applications lourdes nécessitent plus de temps, cette variable permet d'ajuster ce délai.
Ex : `WATCHTOWER_TIMEOUT=30s`

Notes :

- On notera la présence du "s" pour "seconde".

- **LINKED CONTAINERS** : Certains conteneurs peuvent nécessiter que d'autres conteneurs soient opérationnels avant d'être créés, on prend souvent comme exemple le cas typique d'une application et d'une base de données. Afin d'éviter des erreurs, on souhaite que la base de données soit disponible avant l'application à la création, et inversement à la suppression. Ces relations existent par exemple lorsqu'on utilise l'argument --link en ligne de commande, ou avec l'instruction depends_on via docker-compose. On peut écraser ces réglages via le label :  
`com.centurylinklabs.watchtower.depends-on`  
tel que par exemple, si on applique ce label à un conteneur MariaDB :  
`com.centurylinklabs.watchtower.depends-on=wordpress,caldav`  
En cas de mise à jour disponible pour ce dernier, watchtower arrêtera au préalable les conteneurs wordpress et caldav.

#### 3-D. Autres paramètres

- **DEBUG** : Augmente le niveau de verbosité du stdout dans les logs du conteneur.
Ex :  `WATCHTOWER_DEBUG=true`
- **TRACE** : Un debug encore plus verbeux, attention les credentials éventuels sont en clair !
Ex :  `WATCHTOWER_TRACE=true`
- **TZ** : Permet de définir le fuseau horaire, assure le lien entre la variable CRON si vous l'utilisez et votre fuseau horaire. La liste des timezone est disponible à cette adresse.
Ex : `TZ=Europe/Brussels`

Il y a encore énormément de fonctionnalités que je n'aborderai pas ici, citons parmi les plus notables :

- les notifications => https://containrrr.dev/watchtower/notifications/
- La supervision de plusieurs instances Docker via un seul conteneur watchtower => https://containrrr.dev/watchtower/remote-hosts/ et https://containrrr.dev/watchtower/secure-connections/ (pour une connexion sécurisée (TLS), une lecture de https://www.nas-forum.com/forum/topic/66422-tuto-centralisation-des-instances-docker/ peut être instructive 😉)
- la possibilité d'exécuter des scripts avant et après la mise à jour (par l'utilisation de labels) => https://containrrr.dev/watchtower/lifecycle-hooks/
- la possibilité d'utiliser d'autres dépôts que Docker Hub, ouvrant la voie à des dépôts privés => https://containrrr.dev/watchtower/private-registries/

Au final ce tutoriel se contente de traduire et de mettre en valeur les variables importantes, la documentation (je crois que je me répète 😛) est très complète.

### 4. Création du conteneur

On a maintenant tous les outils en main, on va pouvoir créer notre conteneur. Pour se faire, plusieurs possibilités :

#### 4-A. En lignes de commande (SSH)

Voici un exemple de configuration :

```
docker create --name=watchtower \
--hostname=watchtower-nas
--restart=unless-stopped \
--label "com.centurylinklabs.watchtower.enable=true" \
-e WATCHTOWER_CLEANUP=true \
-e WATCHTOWER_DEBUG=true \
-e WATCHTOWER_LABEL_ENABLE=true\
-e WATCHTOWER_TIMEOUT=30s \
-e WATCHTOWER_SCHEDULE="0 0 5 * * 6"\
-e TZ=Europe/Brussels \
-v /var/run/docker.sock:/var/run/docker.sock \
containrrr/watchtower
```

Notes :

- On est obligé de monter en volume le socket Docker au vu des opérations que doit pouvoir effectuer watchtower sur les autres conteneurs.
- On peut tout à fait demander via le label de mise à jour que watchtower se mette lui-même à jour.

Si tout a bien fonctionné, vous devriez avoir comme retour une suite de chiffres et de lettres (correspondant à l'ID du conteneur), il ne reste plus qu'à le démarrer :

```
docker start watchtower
```

#### 4-B. Par Docker-compose (SSH)

Une autre personnalisation de l'image au format docker-compose : docker-compose.yml

```yaml
version: '2.1'
services:

   watchtower:
      image: containrrr/watchtower
      container_name: watchtower
      hostname: watchtower-nas
      network_mode: bridge
      environment:
         - WATCHTOWER_NOTIFICATIONS=email
         - WATCHTOWER_CLEANUP=true
         - WATCHTOWER_DEBUG=true
         - WATCHTOWER_LABEL_ENABLE=true
         - WATCHTOWER_TIMEOUT=30s
         - WATCHTOWER_SCHEDULE=0 0 5 * * 6
         - TZ=Europe/Brussels
      env_file:
         - /volume1/docker/watchtower/watchtower.env
      labels:
         - "com.centurylinklabs.watchtower.enable=true"
      volumes:
         - /var/run/docker.sock:/var/run/docker.sock
         - /volume1/docker/watchtower/config.json:/root/.docker/config.json
      restart: unless-stopped
```

Notes :

- J'ai cette fois-ci ajouté les notifications par mail, mais on remarque qu'on ne retrouve pas les autres variables qui sont liées au relais SMTP (https://containrrr.dev/watchtower/notifications/#email). En réalité, pour des raisons de confidentialié, j'ai regroupé les variables qui sont plus "sensibles" dans un fichier .env qui est invoqué par :

env_file:  
   `- /volume1/docker/watchtower/watchtower.env`

Fichier qui se présente sous la forme, et dont les permissions sont correctement réglées pour assurer le niveau de confidentialité désiré :

```
WATCHTOWER_NOTIFICATION_EMAIL_FROM=...
WATCHTOWER_NOTIFICATION_EMAIL_TO=...
WATCHTOWER_NOTIFICATION_EMAIL_SERVER=...
WATCHTOWER_NOTIFICATION_EMAIL_SERVER_USER=...
WATCHTOWER_NOTIFICATION_EMAIL_SERVER_PASSWORD=...
WATCHTOWER_NOTIFICATION_EMAIL_SERVER_PORT=...
WATCHTOWER_NOTIFICATION_EMAIL_SUBJECTTAG=...
```

J'ai monté le fichier config.json pour y ajouter des credentials pour des dépôts privés.

Pour créer le conteneur, on se place (cd) dans le dossier où se trouve le fichier docker-compose.yml et on tape :

```shell
docker-compose up -d
```

## 5. Configuration

Ci-après un exemple de logs suite à un déclenchement programmé de Watchtower :

```
2020-11-14 05:01:31 (info): Found new grafana/grafana:latest image (sha256:68f75fcab5a9372fb0844f5898c65a40b0ca34cff4176e269bade17ddb17ee75)
2020-11-14 05:01:48 (info): Found new telegraf:latest image (sha256:146c415345a26f48f8ef06c2f62ebde90c8ca4ca518db5d88137e60eeb0abeec)
2020-11-14 05:01:59 (info): Found new authelia/authelia:latest image (sha256:d05b2f15beecd4c164a861e0464ddfef1d574b1c67f8e343779daec26c7bbde9)
2020-11-14 05:03:17 (info): Found new linuxserver/mariadb:latest image (sha256:f48f265a3ed8b786973e85c2c681f2f79db1b64d38660c43900bfc48651c6f83)
2020-11-14 05:03:19 (info): Stopping /grafana (bfdbba3a284d51c1a5c5c807f8583857a0c6a03718a083e93e5bcf9671f7130f) with SIGTERM
2020-11-14 05:03:21 (info): Stopping /telegraf (56e1c8624a98344cba07ca6f745c0cd8a93d66b36ebc0af8992cac210c8e0d51) with SIGTERM
2020-11-14 05:03:22 (info): Stopping /authelia (6d4202898398cf0b46886d38c6591a15b83334e00fe20a0025fcd4437747f84c) with SIGTERM
2020-11-14 05:03:23 (info): Stopping /mariadb (763aeebe73c029c20de2a15d114c054ba70b26879fdaf82b3019944179f31573) with SIGTERM
2020-11-14 05:03:27 (info): Creating /mariadb
2020-11-14 05:03:31 (info): Creating /authelia
2020-11-14 05:03:34 (info): Creating /telegraf
2020-11-14 05:03:40 (info): Creating /grafana
2020-11-14 05:03:42 (info): Removing image sha256:ec8e3ebf50429170fa6387bb04e64e5b5f3d87f3221c42750b7fb5d922c5e978
2020-11-14 05:03:43 (info): Removing image sha256:48121fbe19c845fa401be9fdbf3cbeef50840b7537d8a0b9ca2eab081117beb6
2020-11-14 05:03:43 (info): Removing image sha256:7606d5ee069fcab20036c4bd521e1133ac8ca31e44f7d241f0d83a36315d6ca6
2020-11-14 05:03:43 (info): Removing image sha256:900b03b57e41a8bf7f43b8979872bb2d6642d9a4bcb738d053fb67e1d989e926
```

## Mise en place de Watchtower

Créer un nouveau fichier `docker-compose.yml` dans un nouveau dossier, avec le contenu suivant :

```yaml
version: "3"
services:
  watchtower:
    image: containrrr/watchtower:latest
    container_name: watchtower
    restart: unless-stopped
    environment:
      - WATCHTOWER_POLL_INTERVAL=300
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_INCLUDE_RESTARTING=true
      - WATCHTOWER_LOG_LEVEL=error
      - WATCHTOWER_HTTP_API_METRICS=false
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
```

Voici l'explication des différents éléments de configuration :

*    WATCHTOWER_POLL_INTERVAL : l'intervalle de vérification des mises à jour des containers, ici paramétrée à 5 minutes (300 secondes)
*    WATCHTOWER_CLEANUP : supprime automatiquement les anciennes images inutilisées suite à la mise à jour d'un container, ici activé
*    WATCHTOWER_INCLUDE_RESTARTING : redémarre les containers lors de leur mise à jour, ici activé
*    WATCHTOWER_LOG_LEVEL : le niveau du log d'erreurs, ici on log le niveau erreur au maximum
*    WATCHTOWER_HTTP_API_METRICS : active ou désactive l'API de metrics (qui permet à Prometheus de récupérer certains infos sur le fonctionnement de Watchtower par exemple)

Démarrez le container

```shell
docker-compose up -d
```

puis vérifiez que tout est OK avec les logs : 

```shell
docker container logs watchtower
```

