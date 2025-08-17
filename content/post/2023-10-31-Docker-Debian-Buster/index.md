+++
title = 'Docker + Docker Compose sur Debian, installation et utilisation'
date = 2023-10-31 00:00:00 +0100
categories = debian
+++
![image](docker-logo.png){:width="300px"}   


## I - Docker

[How to Install and Use Docker on Debian 12](https://www.howtoforge.com/how-to-install-docker-engine-on-debian-12/)


*Docker a pour objectif de faciliter le déploiement d'applications, d'avoir plusieurs versions d'une même application sur un son serveur (phase de développement, tests), mais aussi d'automatiser le packaging d'applications. Avec Docker, on s'oriente vers de l'intégration et du déploiement en continu grâce au système de container.*

*De plus, Docker permet de garder son système de base propre, tout en installant de nouvelles fonctionnalités au sein de containers. En quelque sorte, on part d'une base qui est le système d'exploitation et on ajoute différentes briques conteneurisées qui sont les applications.*


### Conditions préalables

Pour suivre ce tutoriel, vous aurez besoin des éléments suivants :

* Un serveur debian 10 et 11 configuré en suivant le guide de configuration initiale du serveur debian 10 et 11, y compris un utilisateur sudo non root et un pare-feu.
* Un compte sur Docker Hub si vous souhaitez créer vos propres images et les pousser vers Docker Hub, comme indiqué aux étapes 7 et 8.

### 1 - Installation du docker

Le paquet d'installation de Docker disponible dans le référentiel Debian officiel peut ne pas être la dernière version. Pour nous assurer d'obtenir la dernière version, nous installerons Docker à partir du dépôt officiel de Docker. Pour ce faire, nous ajouterons une nouvelle source de paquet, ajouterons la clé GPG de Docker pour nous assurer que les téléchargements sont valides, puis installerons le paquet.

Tout d'abord, mettez à jour votre liste existante de paquets :

    sudo apt update

Ensuite, installez quelques paquets prérequis qui permettent à apt d'utiliser des paquets sur HTTPS :

    sudo apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common

Ajoutez ensuite la clé GPG du référentiel officiel du Docker à votre système :


```bash
sudo curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```


Ensuite, mettez à jour la base de données des paquets avec les paquets Docker du repo nouvellement ajouté :

    sudo apt update

Installez Docker :

    sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Version `sudo docker version`

```
Client: Docker Engine - Community
 Version:           24.0.7
 API version:       1.43
 Go version:        go1.20.10
 Git commit:        afdd53b
 Built:             Thu Oct 26 09:08:02 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          24.0.7
  API version:      1.43 (minimum version 1.12)
  Go version:       go1.20.10
  Git commit:       311b9ff
  Built:            Thu Oct 26 09:08:02 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.24
  GitCommit:        61f9fd88f79f081d64d6fa3bb1a0dc71ec870523
 runc:
  Version:          1.1.9
  GitCommit:        v1.1.9-0-gccaecfc
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

Vérifier que les services sont actifs , réponse "enabled"

    sudo systemctl is-enabled docker
    sudo systemctl is-enabled containerd

Les status 

    sudo systemctl status docker
    sudo systemctl status containerd

Pour afficher que les services sont actifs et en cours d'exécution :

```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Tue 2023-10-31 09:01:14 CET; 7min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 1465140 (dockerd)
      Tasks: 10
     Memory: 43.6M
        CPU: 479ms
     CGroup: /system.slice/docker.service
             └─1465140 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; preset: enabled)
     Active: active (running) since Tue 2023-10-31 09:01:09 CET; 12min ago
       Docs: https://containerd.io
   Main PID: 1465057 (containerd)
      Tasks: 9
     Memory: 23.5M
        CPU: 2.744s
     CGroup: /system.slice/containerd.service
             └─1465057 /usr/bin/containerd
[...]
oct. 31 09:01:09 rnmkcy.eu systemd[1]: Started containerd.service - containerd container runtime.

```

L'installation de Docker vous donne maintenant non seulement le service Docker (démon) mais aussi l'utilitaire de ligne de commande du docker, ou le client Docker. Nous verrons comment utiliser la commande docker plus loin dans ce tutoriel.

Version docker

    docker -v

```
Docker version 24.0.7, build afdd53b
```

### 2 - Exécution de la commande Docker sans sudo (optionnel)

Par défaut, la commande du docker ne peut être exécutée que par l'utilisateur root ou par un utilisateur du groupe de dockers, qui est automatiquement créé pendant le processus d'installation du docker. Si vous essayez d'exécuter la commande du docker sans la préfixer avec sudo ou sans être dans le groupe docker, vous obtiendrez un résultat comme ceci :

```
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```

Si vous voulez éviter de taper sudo à chaque fois que vous exécutez la commande docker, ajoutez votre nom d'utilisateur au groupe docker :

    sudo usermod -aG docker ${USER}

Pour appliquer la nouvelle appartenance au groupe, déconnectez-vous du serveur et reconnectez ou tapez ce qui suit :

    su - ${USER}

Vous serez invité à entrer le mot de passe de votre utilisateur pour continuer.

Confirmez que votre utilisateur est maintenant ajouté au groupe de dockers en tapant :

    id -nG

```
admbust adm cdrom floppy audio dip video plugdev systemd-journal netdev vboxusers docker www-default partage
```

Si vous avez besoin d'ajouter un utilisateur au groupe de dockers sous lequel vous n'êtes pas connecté, déclarez ce nom d'utilisateur explicitement en utilisant :

    sudo usermod -aG docker username

Le reste de cet article suppose que vous exécutez la commande docker en tant qu'utilisateur dans le groupe docker. Si vous choisissez de ne pas le faire, préparez les commandes avec sudo.

Examinons maintenant la commande docker.

Étape 3 - Utilisation de la commande Docker

L'utilisation du docker consiste à lui passer une chaîne d'options et de commandes suivie d'arguments. La syntaxe prend cette forme :

    docker [option] [command] [arguments]

Pour afficher toutes les sous-commandes disponibles, tapez :

    docker

À partir du docker 18, la liste complète des sous-commandes disponibles comprend :

Option | Comment EN | Comment FR
--- | --- | ---
attach | Attach local standard input, output, and error streams to a running container | Attacher les flux d'entrée, de sortie et d'erreurs standard locaux à un conteneur en cours d'exécution
build |  Build an image from a Dockerfile | Construire une image à partir d'un Dockerfile
commit | Create a new image from a container's changes | Créer une nouvelle image à partir des modifications d'un conteneur
cp |     Copy files/folders between a container and the local filesystem | Copier des fichiers/dossiers entre un conteneur et le système de fichiers local
create | Create a new container | Créer un nouveau conteneur
diff |   Inspect changes to files or directories on a container's filesystem | Inspecter les modifications apportées aux fichiers ou répertoires du système de fichiers d'un conteneur
events | Get real time events from the server | Obtenir les événements en temps réel à partir du serveur
exec |   Run a command in a running container | Exécuter une commande dans un conteneur en cours d'exécution
export | Export a container's filesystem as a tar archive | Exporter le système de fichiers d'un conteneur sous forme d'archive tar
history  |   Show the history of an image | Afficher l'historique d'une image
images | List images | Liste des images
import | Import the contents from a tarball to create a filesystem image | Importer le contenu d'une archive pour créer une image de système de fichiers
info |   Display system-wide information | Afficher les informations sur l'ensemble du système
inspect  |   Return low-level information on Docker objects | Retourner les informations de bas niveau sur les objets Docker
kill |   Kill one or more running containers | Tuer un ou plusieurs conteneurs en cours d'exécution
load |   Load an image from a tar archive or STDIN | Charger une image depuis une archive tar ou STDIN
login |  Log in to a Docker registry | Se connecter à un registre Docker
logout | Log out from a Docker registry | Déconnexion d'un registre Docker
logs |   Fetch the logs of a container | Récupérer les "logs" d'un conteneur
pause |  Pause all processes within one or more containers | Pause Tous les processus dans un ou plusieurs conteneurs
port |   List port mappings or a specific mapping for the container | Liste des mappages de port ou un mappage spécifique pour le conteneur
ps |     List containers | Liste des conteneurs
pull |   Pull an image or a repository from a registry | Sortir une image ou un référentiel d'un registre
push |   Push an image or a repository to a registry | Pousse une image ou un référentiel vers un registre
rename | Rename a container | Renommer un conteneur
restart  |   Restart one or more containers | Redémarrer un ou plusieurs conteneurs
rm |    Remove one or more containers | Retirer un ou plusieurs récipients
rmi | Remove one or more images | Supprimer une ou plusieurs images
run |    Run a command in a new container | Exécuter une commande dans un nouveau conteneur
save |   Save one or more images to a tar archive (streamed to STDOUT by default) | Enregistrer une ou plusieurs images dans une archive tar (diffusée en continu vers STDOUT par défaut)
search | Search the Docker Hub for images | Rechercher des images dans le Docker Hub
start |  Start one or more stopped containers | Démarrer un ou plusieurs conteneurs arrêtés
stats |  Display a live stream of container(s) resource usage statistics | Afficher un flux en direct des statistiques d'utilisation des ressources du ou des conteneurs
stop |   Stop one or more running containers | Arrêter un ou plusieurs conteneurs en mouvement
tag |    Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE | Créer une balise TARGET_IMAGE qui se réfère à SOURCE_IMAGE
top |    Display the running processes of a container | Afficher les processus en cours d'un conteneur
unpause  |   Unpause all processes within one or more containers | Unpause tous les processus dans un ou plusieurs conteneurs
update | Update configuration of one or more containers | Mettre à jour la configuration d'un ou plusieurs conteneurs
version  |   Show the Docker version information | Afficher les informations de version du Docker
wait |   Block until one or more containers stop, then print their exit codes | attendre qu'un ou plusieurs conteneurs s'arrêtent, puis imprimer leur code de sortie

Pour afficher les options disponibles pour une commande spécifique, tapez :

    docker docker-subcommand --help

Pour afficher des informations sur Docker à l'échelle du système, utilisez :

    docker info 

Examinons quelques-unes de ces commandes. Nous allons commencer par travailler avec des images.

### 4 - Travailler avec les images du docker

Les conteneurs Docker sont construits à partir d'images Docker. Par défaut, Docker extrait ces images de Docker Hub, un registre de Docker géré par Docker, la société derrière le projet Docker. N'importe qui peut héberger ses images Docker sur Docker Hub, donc la plupart des applications et distributions Linux dont vous aurez besoin auront des images hébergées là.

Pour vérifier si vous pouvez accéder et télécharger des images depuis Docker Hub, tapez :

    docker run hello-world

La sortie indiquera que le Docker fonctionne correctement :

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9db2ca6ccae0: Pull complete
Digest: sha256:4b8ff392a12ed9ea17784bd3c9a8b1fa3299cac44aca35a85c90c5e3c7afacdc
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

Docker n'a d'abord pas pu trouver l'image hello-world localement, alors il a téléchargé l'image depuis Docker Hub, qui est le référentiel par défaut. Une fois l'image téléchargée, Docker crée un conteneur à partir de l'image et l'application à l'intérieur du conteneur est exécutée, affichant le message.

Vous pouvez rechercher des images disponibles sur Docker Hub en utilisant la commande docker avec la sous-commande de recherche. Par exemple, pour rechercher l'image Ubuntu, tapez :

    docker search ubuntu

Le script parcourra le Docker Hub et retournera une liste de toutes les images dont le nom correspond à la chaîne de recherche. Dans ce cas, la sortie sera similaire à celle-ci :

```
NAME                                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
ubuntu                                                 Ubuntu is a Debian-based Linux operating sys…   8320                [OK]
dorowu/ubuntu-desktop-lxde-vnc                         Ubuntu with openssh-server and NoVNC            214                                     [OK]
rastasheep/ubuntu-sshd                                 Dockerized SSH service, built on top of offi…   170                                     [OK]
consol/ubuntu-xfce-vnc                                 Ubuntu container with "headless" VNC session…   128                                     [OK]
ansible/ubuntu14.04-ansible                            Ubuntu 14.04 LTS with ansible                   95                                      [OK]
ubuntu-upstart                                         Upstart is an event-based replacement for th…   88                  [OK]
neurodebian                                            NeuroDebian provides neuroscience research s…   53                  [OK]
1and1internet/ubuntu-16-nginx-php-phpmyadmin-mysql-5   ubuntu-16-nginx-php-phpmyadmin-mysql-5          43                                      [OK]
ubuntu-debootstrap                                     debootstrap --variant=minbase --components=m…   39                  [OK]
nuagebec/ubuntu                                        Simple always updated Ubuntu docker images w…   23                                      [OK]
tutum/ubuntu                                           Simple Ubuntu docker images with SSH access     18
i386/ubuntu                                            Ubuntu is a Debian-based Linux operating sys…   13
1and1internet/ubuntu-16-apache-php-7.0                 ubuntu-16-apache-php-7.0                        12                                      [OK]
ppc64le/ubuntu                                         Ubuntu is a Debian-based Linux operating sys…   12
eclipse/ubuntu_jdk8                                    Ubuntu, JDK8, Maven 3, git, curl, nmap, mc, …   6                                       [OK]
darksheer/ubuntu                                       Base Ubuntu Image -- Updated hourly             4                                       [OK]
codenvy/ubuntu_jdk8                                    Ubuntu, JDK8, Maven 3, git, curl, nmap, mc, …   4                                       [OK]
1and1internet/ubuntu-16-nginx-php-5.6-wordpress-4      ubuntu-16-nginx-php-5.6-wordpress-4             3                                       [OK]
pivotaldata/ubuntu                                     A quick freshening-up of the base Ubuntu doc…   2
1and1internet/ubuntu-16-sshd                           ubuntu-16-sshd                                  1                                       [OK]
ossobv/ubuntu                                          Custom ubuntu image from scratch (based on o…   0
smartentry/ubuntu                                      ubuntu with smartentry                          0                                       [OK]
1and1internet/ubuntu-16-healthcheck                    ubuntu-16-healthcheck                           0                                       [OK]
pivotaldata/ubuntu-gpdb-dev                            Ubuntu images for GPDB development              0
paasmule/bosh-tools-ubuntu                             Ubuntu based bosh-cli                           0                                       [OK]
...
```

Dans la colonne OFFICIAL, OK indique une image construite et soutenue par l'entreprise derrière le projet. Une fois que vous avez identifié l'image que vous souhaitez utiliser, vous pouvez la télécharger sur votre ordinateur en utilisant la sous-commande pull.

Exécutez la commande suivante pour télécharger l'image ubuntu officielle sur votre ordinateur :

    docker pull ubuntu

```
Using default tag: latest
latest: Pulling from library/ubuntu
6b98dfc16071: Pull complete
4001a1209541: Pull complete
6319fc68c576: Pull complete
b24603670dc3: Pull complete
97f170c87c6f: Pull complete
Digest: sha256:5f4bdc3467537cbbe563e80db2c3ec95d548a9145d64453b06939c4592d67b6d
Status: Downloaded newer image for ubuntu:latest
```

Après le téléchargement d'une image, vous pouvez ensuite exécuter un conteneur à l'aide de l'image téléchargée avec la sous-commande Exécuter. Comme vous l'avez vu avec l'exemple hello-world, si une image n'a pas été téléchargée lorsque le docker est exécuté avec la sous-commande run, le client Docker téléchargera d'abord l'image, puis exécutera un conteneur en l'utilisant.

Pour voir les images qui ont été téléchargées sur votre ordinateur, tapez :

    docker images

La sortie doit ressembler à ce qui suit :

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              16508e5c265d        13 days ago         84.1MB
hello-world         latest              2cb0d9787c4d        7 weeks ago         1.85kB
```

Comme vous le verrez plus loin dans ce tutoriel, les images que vous utilisez pour exécuter des conteneurs peuvent être modifiées et utilisées pour générer de nouvelles images, qui peuvent ensuite être téléchargées (poussées est le terme technique) vers Docker Hub ou d'autres registres Docker.

Examinons plus en détail la façon d'utiliser les conteneurs.

### 5 - Utilisation d'un conteneur de docker

Le conteneur hello-world que vous avez utilisé à l'étape précédente est un exemple de conteneur qui fonctionne et sort après avoir émis un message de test. Les conteneurs peuvent être beaucoup plus utiles que cela, et ils peuvent être interactifs. Après tout, elles sont similaires aux machines virtuelles, mais plus respectueuses des ressources.

Par exemple, exécutons un conteneur en utilisant la dernière image d'Ubuntu. La combinaison des commutateurs -i et -t vous donne un accès interactif à l'interpréteur de commandes dans le conteneur :

    docker run -it ubuntu

Votre invite de commande devrait changer pour refléter le fait que vous travaillez maintenant à l'intérieur du conteneur et devrait prendre cette forme :

```
root@d9b100f2f636:/#
```

Notez l'ID du conteneur dans l'invite de commande. Dans cet exemple, il s'agit de d9b100f2f2f636. Vous aurez besoin de cet ID de conteneur plus tard pour identifier le conteneur lorsque vous voudrez l'enlever.

Vous pouvez maintenant exécuter n'importe quelle commande à l'intérieur du conteneur. Par exemple, mettons à jour la base de données des paquets à l'intérieur du conteneur. Vous n'avez pas besoin de préfixer une commande avec sudo, car vous opérez dans le conteneur en tant qu'utilisateur root :

    apt update

Ensuite, installez n'importe quelle application. Installons Node.js :

    apt install nodejs

Ceci installe Node.js dans le conteneur à partir du dépôt officiel d'Ubuntu. Une fois l'installation terminée, vérifiez que Node.js est installé :

    node -v

Vous verrez le numéro de version affiché dans votre terminal :

    v8.10.0

Toute modification apportée à l'intérieur du conteneur ne s'applique qu'à ce conteneur.  
Pour quitter le conteneur, tapez exit à l'invite.  
Regardons maintenant la gestion des conteneurs sur notre système.

### 6 - Gestion des conteneurs de dockers

Après avoir utilisé Docker pendant un certain temps, vous aurez de nombreux conteneurs actifs (en cours d'exécution) et inactifs sur votre ordinateur. Pour visualiser les actifs, utilisez :

    docker ps

Vous verrez une sortie similaire à la suivante :

    CONTAINER ID        IMAGE               COMMAND             CREATED   

Dans ce tutoriel, vous avez démarré deux conteneurs ; l'un de l'image hello-world et l'autre de l'image ubuntu. Les deux conteneurs ne fonctionnent plus, mais ils existent toujours sur votre système.

Pour afficher tous les conteneurs - actifs et inactifs, lancez le docker ps avec l'interrupteur -a :

    docker ps -a

Vous verrez une sortie similaire à celle-ci :

```
d9b100f2f636        ubuntu              "/bin/bash"         About an hour ago   Exited (0) 8 minutes ago                           sharp_volhard
01c950718166        hello-world         "/hello"            About an hour ago   Exited (0) About an hour ago                       festive_williams
```

Pour afficher le dernier conteneur que vous avez créé, passez-lui l'option -l :

    docker ps -l

```
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
    d9b100f2f636        ubuntu              "/bin/bash"         About an hour ago   Exited (0) 10 minutes ago                       sharp_volhard
```

Pour démarrer un conteneur à l'arrêt, utilisez la fonction de démarrage du docker, suivie de l'ID du conteneur ou du nom du conteneur. Démarrons le conteneur basé sur Ubuntu avec l'ID d9b100f2f636 :

    docker start d9b100f2f636

Le conteneur démarre et vous pouvez utiliser docker ps pour voir son état :

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
d9b100f2f636        ubuntu              "/bin/bash"         About an hour ago   Up 8 seconds                            sharp_volhard
```

Pour arrêter un conteneur en cours de fonctionnement, utilisez la fonction docker stop, suivie de l'ID ou du nom du conteneur. Cette fois, nous utiliserons le nom que Docker a attribué au conteneur, qui est sharp_volhard :

    docker stop sharp_volhard

Une fois que vous avez décidé que vous n'avez plus besoin d'un conteneur, supprimez-le à l'aide de la commande docker rm, en utilisant à nouveau l'ID ou le nom du conteneur. Utilisez la commande docker ps -a pour trouver l'ID ou le nom du conteneur associé à l'image hello-world et le supprimer.

    docker rm festive_williams

Vous pouvez démarrer un nouveau conteneur et lui donner un nom en utilisant le commutateur --name. Vous pouvez également utiliser le commutateur --rm pour créer un conteneur qui se retire tout seul lorsqu'il est arrêté. Reportez-vous à la commande d'aide de l'exécution du docker pour plus d'informations sur ces options et d'autres.

Les conteneurs peuvent être transformés en images que vous pouvez utiliser pour construire de nouveaux conteneurs. Voyons comment cela fonctionne.

### 7 - Transformation d'un conteneur en une image docker

Lorsque vous démarrez une image Docker, vous pouvez créer, modifier et supprimer des fichiers comme vous le pouvez avec une machine virtuelle. Les modifications que vous apportez ne s'appliqueront qu'à ce contenant. Vous pouvez le démarrer et l'arrêter, mais une fois que vous l'aurez détruit avec la commande rm du docker, les changements seront perdus pour de bon.

Cette section vous montre comment enregistrer l'état d'un conteneur en tant que nouvelle image du Docker.

Après avoir installé Node.js dans le conteneur Ubuntu, vous avez maintenant un conteneur qui fonctionne à partir d'une image, mais le conteneur est différent de l'image que vous avez utilisée pour le créer. Mais vous voudrez peut-être réutiliser ce conteneur Node.js comme base pour de nouvelles images plus tard.

Ensuite, validez les modifications sur une nouvelle instance d'image du Docker à l'aide de la commande suivante.

    docker commit -m "What you did to the image" -a "Author Name" container_id repository/new_image_name

Le commutateur -m est pour le message de livraison qui vous aide, vous et les autres, à savoir quels changements vous avez faits, tandis que -a est utilisé pour spécifier l'auteur. Le container_id est celui que vous avez noté plus tôt dans le tutoriel lorsque vous avez démarré la session interactive du Docker. A moins que vous n'ayez créé des dépôts supplémentaires sur Docker Hub, le dépôt (repository) est généralement votre nom d'utilisateur Docker Hub.

Par exemple, pour l'utilisateur sammy, avec l'ID de conteneur d9b100f2f636, la commande serait :

    docker commit -m "added Node.js" -a "sammy" d9b100f2f636 sammy/ubuntu-nodejs

Lorsque vous validez une image, la nouvelle image est enregistrée localement sur votre ordinateur. Plus loin dans ce tutoriel, vous apprendrez comment pousser une image vers un registre Docker comme Docker Hub pour que d'autres puissent y accéder.

Le fait de lister à nouveau les images du Docker affichera la nouvelle image, ainsi que l'ancienne dont elle est dérivée :

    docker images

Vous verrez une sortie comme celle-ci :

```
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
sammy/ubuntu-nodejs   latest              7c1f35226ca6        7 seconds ago       179MB
ubuntu                   latest              113a43faa138        4 weeks ago         81.2MB
hello-world              latest              e38bc07ac18e        2 months ago        1.85kB
```

Dans cet exemple, ubuntu-nodejs est la nouvelle image, qui a été dérivée de l'image ubuntu existante de Docker Hub. La différence de taille reflète les changements qui ont été apportés. Et dans cet exemple, le changement a été l'installation de NodeJS. Ainsi, la prochaine fois que vous aurez besoin d'exécuter un conteneur en utilisant Ubuntu avec NodeJS préinstallé, vous pourrez simplement utiliser la nouvelle image.

Vous pouvez également construire des images à partir d'un fichier docker (Dockerfile), ce qui vous permet d'automatiser l'installation du logiciel dans une nouvelle image. Cependant, cela n'entre pas dans le cadre de ce tutoriel.

Partageons maintenant la nouvelle image avec d'autres personnes pour qu'elles puissent créer des conteneurs à partir de celle-ci.

### 8 - Pousser les Docker Images vers un Docker Repository

La prochaine étape logique après la création d'une nouvelle image à partir d'une image existante est de la partager avec quelques-uns de vos amis, le monde entier sur Docker Hub, ou tout autre registre Docker auquel vous avez accès. Pour pousser une image vers Docker Hub ou tout autre registre Docker, vous devez y avoir un compte.

Cette section vous montre comment pousser une image du Docker vers le Hub du Docker. Pour savoir comment créer votre propre registre de dockers privé, consultez la section Comment créer un registre de dockers privé sur Ubuntu 14.04.

Pour pousser votre image, connectez-vous d'abord à Docker Hub.

    docker login -u docker-registry-username

Vous serez invité à vous authentifier à l'aide de votre mot de passe Docker Hub. Si vous avez spécifié le mot de passe correct, l'authentification devrait réussir. 

>Remarque : Si votre nom d'utilisateur du registre Docker est différent du nom d'utilisateur local que vous avez utilisé pour créer l'image, vous devrez marquer votre image avec votre nom d'utilisateur du registre. Pour l'exemple donné à la dernière étape, vous devez taper :  
`docker tag sammy/ubuntu-nodejs docker-registry-username/ubuntu-nodejs`  

Ensuite, vous pouvez pousser votre propre image en utilisant :

    docker push docker-registry-username/docker-image-name

Pour pousser l'image ubuntu-nodejs vers le dépôt sammy, la commande serait :

    docker push sammy/ubuntu-nodejs

Le processus peut prendre un certain temps pour se terminer lorsqu'il télécharge les images, mais une fois terminé, le résultat ressemblera à ceci :

```
The push refers to a repository [docker.io/sammy/ubuntu-nodejs]
e3fbbfb44187: Pushed
5f70bf18a086: Pushed
a3b5c80a4eba: Pushed
7f18b442972b: Pushed
3ce512daaf78: Pushed
7aae4540b42d: Pushed

...
```

Après avoir poussé une image vers un registre, elle devrait être listée dans le tableau de bord de votre compte, comme cela apparaît dans l'image ci-dessous.

![](ec2vX3Z.png){:width="600"}

Nouvelle liste d'images de Docker sur Docker Hub

Si une tentative de poussée entraîne une erreur de ce type, il est probable que vous ne vous soyez pas connecté :

```
The push refers to a repository [docker.io/sammy/ubuntu-nodejs]
e3fbbfb44187: Preparing
5f70bf18a086: Preparing
a3b5c80a4eba: Preparing
7f18b442972b: Preparing
3ce512daaf78: Preparing
7aae4540b42d: Waiting
unauthorized: authentication required
```

Connectez-vous avec le login du docker docker login et répétez la tentative de poussée. Vérifiez ensuite qu'il existe sur votre page de dépôt Docker Hub.

Vous pouvez maintenant utiliser docker pull sammy/ubuntu-nodejs pour tirer l'image vers une nouvelle machine et l'utiliser pour exécuter un nouveau conteneur.

### 9 - [Comment supprimer des images, des conteneurs et des volumes Docker](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes-fr)

### 10 - [Docker Exec: How to Enter Into a Docker Container's Shell?](https://kodekloud.com/blog/docker-exec/)

### Conclusion

Dans ce tutoriel, vous avez installé Docker, travaillé avec des images et des conteneurs, et déplacé une image modifiée vers Docker Hub. Maintenant que vous connaissez les bases, explorez les [autres tutoriels de Docker](https://www.digitalocean.com/community/tags/docker?type=tutorials) dans la communauté DigitalOcean.

## II - Docker Compose

### Introduction

*Docker est un excellent outil pour automatiser le déploiement d'applications Linux à l'intérieur de conteneurs logiciels, mais pour tirer pleinement parti de son potentiel, chaque composant d'une application doit fonctionner dans son propre conteneur individuel. Pour des applications complexes avec beaucoup de composants, orchestrer tous les conteneurs pour démarrer, communiquer et s'arrêter ensemble peut rapidement devenir compliqué.  
La communauté Docker a mis au point une solution populaire appelée Fig, qui vous permet d'utiliser un seul fichier YAML pour orchestrer tous vos conteneurs et configurations Docker. Cela est devenu si populaire que l'équipe Docker a décidé de créer **Docker Compose** à partir de la source Fig, qui est maintenant obsolète. Docker Compose permet aux utilisateurs d'orchestrer plus facilement les processus des conteneurs Docker, y compris le démarrage, l'arrêt et la mise en place de liens et de volumes intra-container.*

Dans ce tutoriel, nous allons vous montrer comment installer la dernière version de Docker Compose pour vous aider à gérer les applications multi-containers sur un serveur debian 10 et 11.

### Conditions préalables

Pour suivre cet article, vous aurez besoin de :

* Un serveur debian 10 et 11 et un utilisateur non root avec des privilèges sudo. Cette configuration initiale du serveur avec le tutoriel debian 10 et 11 explique comment configurer cela.
* Docker installé avec les instructions des étapes 1 et 2 de Comment installer et utiliser le docker sur debian 10 et 11

>Remarque : Bien que les conditions préalables donnent des instructions pour installer Docker sur debian 10 et 11, les commandes de docker dans cet article devraient fonctionner sur d'autres systèmes d'exploitation tant que Docker est installé.

### 1 - Installation de Docker Compose

Bien que nous puissions installer Docker Compose à partir des dépôts Debian officiels, il y a plusieurs versions mineures derrière la dernière version, donc nous allons l'installer à partir du dépôt GitHub de Docker. La commande ci-dessous est légèrement différente de celle que vous trouverez sur la page Communiqués. En utilisant l'option -o pour spécifier d'abord le fichier de sortie plutôt que de rediriger la sortie, cette syntaxe évite de se heurter à une erreur de refus de permission causée par l'utilisation de sudo.

Nous allons vérifier la version actuelle et, si nécessaire, la mettre à jour dans la commande ci-dessous :

    sudo curl -L https://github.com/docker/compose/releases/download/1.25.3/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

Ensuite, nous allons définir les permissions :

    sudo chmod +x /usr/local/bin/docker-compose

Ensuite, nous vérifierons que l'installation a réussi en vérifiant la version :

    docker-compose --version


Ceci imprimera la version que nous avons installée :

```
docker-compose version 1.25.3, build d4d1b42b
```

Maintenant que Docker Compose est installé, nous sommes prêts à lancer un exemple "Hello World".

### 2 - Utilisation d'un conteneur avec Docker Compose

Le registre public des dockers, Docker Hub, comprend une image Hello World pour démonstration et test. Il illustre la configuration minimale requise pour exécuter un conteneur en utilisant Docker Compose : un fichier YAML qui appelle une seule image. Nous allons créer cette configuration minimale pour faire fonctionner notre conteneur hello-world.

Tout d'abord, nous allons créer un répertoire pour le fichier YAML et y accéder :

    mkdir hello-world
    cd hello-world

Ensuite, nous allons créer le fichier YAML :

    nano docker-compose.yml

Placez le contenu suivant dans le fichier, enregistrez le fichier et quittez l'éditeur de texte :
docker-compose.yml

```
my-test:
 image: hello-world
```

La première ligne du fichier YAML est utilisée comme partie du nom du conteneur. La deuxième ligne spécifie l'image à utiliser pour créer le conteneur. Lorsque nous lançons la commande **docker-composer up**, elle cherchera une image locale par le nom que nous avons spécifié, **hello-world**. Une fois cela en place, nous sauvegarderons et quitterons le fichier.

Nous pouvons regarder manuellement les images sur notre système avec la commande docker images :

    docker images

Lorsqu'il n'y a pas d'images locales du tout, seuls les titres des colonnes s'affichent :

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

Maintenant, alors que nous sommes encore dans le répertoire **~/hello-world**, nous allons exécuter la commande suivante :

    docker-composer up

La première fois que nous exécutons la commande, s'il n'y a pas d'image locale nommée hello-world, Docker Compose la sortira du dépôt public de Docker Hub :

```
Pulling my-test (hello-world:)...
latest: Pulling from library/hello-world
9db2ca6ccae0: Pull complete
Digest: sha256:4b8ff392a12ed9ea17784bd3c9a8b1fa3299cac44aca35a85c90c5e3c7afacdc
Status: Downloaded newer image for hello-world:latest
. . .
```

Après avoir tiré l'image, **docker-compose** crée un conteneur, attache et exécute le programme hello, qui à son tour confirme que l'installation semble fonctionner :

```
. . .
Creating helloworld_my-test_1...
Attaching to helloworld_my-test_1
my-test_1 |
my-test_1 | Hello from Docker.
my-test_1 | This message shows that your installation appears to be working correctly.
my-test_1 |
. . .
```

Ensuite, il imprime une explication de ce qu'il a fait :

```
 To generate this message, Docker took the following steps:
my-test_1  |  1. The Docker client contacted the Docker daemon.
my-test_1  |  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
my-test_1  |     (amd64)
my-test_1  |  3. The Docker daemon created a new container from that image which runs the
my-test_1  |     executable that produces the output you are currently reading.
my-test_1  |  4. The Docker daemon streamed that output to the Docker client, which sent it
my-test_1  |     to your terminal.
```

Les conteneurs des dockers fonctionnent tant que la commande est active, donc une fois que hello a fini de fonctionner, le conteneur s'est arrêté. Par conséquent, lorsque nous regardons les processus actifs, les en-têtes de colonnes apparaîtront, mais le conteneur hello-world ne sera pas listé parce qu'il ne fonctionne pas :

    docker ps

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
```

Nous pouvons voir les informations sur le conteneur, dont nous aurons besoin à l'étape suivante, en utilisant le drapeau -a. Ceci montre tous les conteneurs, pas seulement ceux qui sont actifs :

    docker ps -a

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
06069fd5ca23        hello-world         "/hello"            35 minutes ago      Exited (0) 35 minutes ago                       hello-world_my-test_1
```

Ceci affiche les informations dont nous aurons besoin pour retirer le conteneur lorsque nous en aurons fini avec lui.

### 3 - Retrait de l'image (facultatif)

Pour éviter d'utiliser inutilement de l'espace disque, nous allons supprimer l'image locale. Pour ce faire, nous devrons supprimer tous les conteneurs qui font référence à l'image à l'aide de la commande **docker rm**, suivie soit de l'ID CONTAINER, soit du NOM. Ci-dessous, nous utilisons l'ID CONTAINER du **docker ps -a** commande que nous venons d'exécuter. Assurez-vous de remplacer l'ID de votre conteneur :

    docker rm 06069fd5ca23

Une fois que tous les conteneurs qui font référence à l'image ont été supprimés, nous pouvons supprimer l'image :

    docker rmi hello-world

### Conclusion

Nous avons maintenant installé **Docker Compose**, testé notre installation en exécutant un exemple Hello World, et supprimé l'image de test et le conteneur.

Bien que l'exemple de Hello World ait confirmé notre installation, la configuration simple ne montre pas l'un des principaux avantages de Docker Compose - être capable de faire monter et descendre un groupe de conteneurs Docker tous en même temps. Pour voir la puissance de Docker Compose en action, vous pouvez consulter cet exemple pratique, [Comment configurer un environnement de test d'intégration continue avec Docker et Docker Compose sur Ubuntu 16.04 (en)](https://www.digitalocean.com/community/tutorials/how-to-configure-a-continuous-integration-testing-environment-with-docker-and-docker-compose-on-ubuntu-16-04).


## Conteneur Docker systemd service

### Introduction

Il existe de nombreuses façons d'orchestrer la gestion, l'initialisation, le déploiement des conteneurs de docker, etc. Même Docker apporte son propre outil et son mode vanille. Il existe également de nombreux autres outils tiers d'orchestration de conteneurs tels que Kubernetes, Rancher, Apache Mesos, etc.

Le démon Docker offre des moyens simples pour démarrer, arrêter, gérer et interroger le statut des conteneurs déployés. Dans cet article, nous allons voir comment utiliser une approche docker + systemd uniquement pour déployer des conteneurs en tant que services Linux sans avoir besoin d'outils tiers ou de descripteurs de déploiement complexes.

Dans ce tutoriel, nous montrerons comment déployer Portainer en tant que service systemd de Linux.

### Conteneur existant

La méthode la plus simple pour déployer un conteneur en tant que service consiste à créer un conteneur de docker avec un nom donné et ensuite de mapper chacune des opérations de docker (démarrage et arrêt) aux commandes de service du système.

    docker run -d --name portainer --privileged -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v /opt/docker/data/portainer:/data portainer/portainer

Une fois que nous avons créé ce conteneur, nous pouvons le démarrer, l'arrêter et le redémarrer en utilisant les commandes habituelles du docker en indiquant le nom du conteneur (docker stop portainer, docker start portainer, docker restart portainer).

Nous pouvons créer un nouveau fichier d'unité systemd avec la description du service en créant un nouveau fichier dans /etc/systemd/system/. Pour les besoins de cet exemple, nous allons créer un nouveau fichier portainer.service avec le contenu suivant

	
```
[Unit]
Description=Portainer container
After=docker.service
Wants=network-online.target docker.socket
Requires=docker.socket
 
[Service]
Restart=always
ExecStart=/usr/bin/docker start -a portainer
ExecStop=/usr/bin/docker stop -t 10 portainer
 
[Install]
WantedBy=multi-user.target
```

Le fichier d'unité crée un nouveau service et associe les commandes de démarrage et d'arrêt du docker aux séquences de démarrage et d'arrêt du service.

Le fichier unit décrit comme des dépendances la cible réseau en ligne et la prise docker, si la prise docker ne démarre pas ce service ne le fera pas non plus. Il ajoute également une dépendance à docker.service, de sorte que ce service ne fonctionnera pas tant que docker.service n'aura pas démarré.

Nous pouvons maintenant démarrer/arrêter le service en émettant la commande correspondante :

	
    systemctl start portainer
    systemctl stop portainer

Nous pouvons également installer le service pour qu'il fonctionne au démarrage en courant :
	
    systemctl enable portainer.service

### Créer un conteneur au démarrage

Nous pouvons améliorer le fichier d'unité précédent pour créer le conteneur s'il n'existe pas, cela nous permettra de sauter la première étape de création manuelle du conteneur (docker run...) et peut être utilisé pour déployer les conteneurs en créant simplement le fichier d'unité de descripteur de service.

```
[Unit]
Description=Portainer container
After=docker.service
Wants=network-online.target docker.socket
Requires=docker.socket
 
[Service]
Restart=always
ExecStartPre=/bin/bash -c "/usr/bin/docker container inspect portainer 2> /dev/null || /usr/bin/docker run -d --name portainer --privileged -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v /opt/docker/data/portainer:/data portainer/portainer"
ExecStart=/usr/bin/docker start -a portainer
ExecStop=/usr/bin/docker stop -t 10 portainer
 
[Install]
WantedBy=multi-user.target
```

Nous avons simplement ajouté une ligne supplémentaire avec une instruction **ExecStartPre**. Les fichiers d'unité Systemd présentent certaines limitations lors de l'exécution de commandes dans les définitions **ExecStart**, **ExtecStop**, **ExecStartPre**..... Afin de surmonter ces limitations, nous avons encapsulé la commande dans une commande **"/bin/bash -c"** qui nous permet un peu plus de flexibilité.

La première commande (docker container inspect) vérifiera si un conteneur de ce nom existe déjà dans la machine hôte du docker. Si cette commande provoque une erreur (c'est-à-dire si le conteneur n'existe pas), l'instruction suivante sera exécutée. Cette commande est la même que celle que nous avons utilisée précédemment pour créer le conteneur manuellement.

Il est vraiment important d'exécuter la commande d'exécution du Docker avec le drapeau **-d (détaché)**, sinon la commande ne reviendra pas et l'étape suivante (ExecStart) ne pourra pas être exécutée, de sorte que le systemd ne sera jamais actif.

En fonction de l'existence antérieure du conteneur, la séquence de démarrage du service se comportera d'une manière ou d'une autre :

* Un conteneur nommé portainer existe déjà :
    * **ExecStartPre** reviendra sans erreur sur la première commande (docker container inspect...) donc n'exécutera pas la seconde (docker run...).
    * **ExecStart** s'exécutera normalement et le processus s'attachera au conteneur.
* Un conteneur nommé portainer n'existe pas :
    * **ExecStartPre** reviendra avec une erreur sur la première commande (docker container inspect...), la deuxième commande créera et démarrera le conteneur en mode détaché en renvoyant le contrôle à systemctl.
    * **ExecStart** s'exécutera (docker start) ce qui n'aura aucun effet mais s'attachera au conteneur en cours d'exécution.

### Conclusion

Dans ce tutoriel, nous avons montré une façon simple de déployer des conteneurs dockers en tant que services systemd Linux. Il s'agit d'un "chea

## Désinstaller

Pour une désinstallation complète  

    sudo apt purge -y docker-ce docker-buildx-plugin docker-compose docker-compose-plugin python3-docker python3-dockerpty docker.io

