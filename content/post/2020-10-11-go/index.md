+++
title = 'Installer Golang go sur Linux'
date = 2020-10-11 00:00:00 +0100
categories = go
+++
![go](go-logo.png){:width="150"}


*Go , également connu sous le nom de golang , est un langage de programmation open source moderne développé par Google. Go essaie de rendre le développement logiciel sûr, rapide et accessible pour vous aider à créer des logiciels fiables et efficaces.*  
Ce didacticiel vous guidera à travers le téléchargement et l'installation de Go à partir des sources, ainsi que la compilation et l'exécution d'un «Hello, World!» programme sur un serveur Debian 10.  
Vous aurez besoin d'accéder à un serveur Debian 10 et à un utilisateur non root avec des sudo privilèges.

### Installer Go depuis les sources

Dans cette étape, nous installerons Go sur votre serveur.    
Tout d'abord, assurez-vous que votre apt index de package est à jour à l'aide de la commande suivante:

    sudo apt update

Maintenant, installez **curl** afin que vous puissiez récupérer la dernière version de Go:

    sudo apt install curl

Ensuite, visitez la [page officielle de téléchargements Go](https://golang.org/dl/) et trouvez l'URL de l'archive tar de la version binaire actuelle. Assurez-vous de copier le lien pour la dernière version compatible avec une architecture 64 bits.

Depuis votre répertoire personnel, utilisez curl pour récupérer l'archive tar:

    curl -O https://golang.org/dl/go1.15.3.linux-amd64.tar.gz

Ou wget

    wget https://golang.org/dl/go1.15.3.linux-amd64.tar.gz

Bien que l'archive provienne d'une source authentique, il est recommandé de vérifier à la fois l'authenticité et l'intégrité des éléments téléchargés sur Internet. Cette méthode de vérification certifie que le fichier n'a été ni falsifié ni corrompu ou endommagé pendant le processus de téléchargement. La sha256sumcommande produit un hachage unique de 256 bits:

    sha256sum go1.15.3.linux-amd64.tar.gz

```
go1.15.3.linux-amd64.tar.gz
 010a88df924a81ec21b293b5da8f9b11c176d27c0ee3962dc1738d2352d3c02d  go1.15.3.linux-amd64.tar.gz
```

Comparez le hachage de votre sortie à la valeur de la somme de contrôle sur la page de téléchargement Go . S'ils correspondent, il est sûr de conclure que le téléchargement est légitime.

Une fois Go téléchargé et l'intégrité du fichier validée, procédons à l'installation.

### Installation de Go

Nous allons maintenant utiliser **tar** pour extraire l'archive tar. Les indicateurs suivants sont utilisés pour indiquer à tar comment extraire, afficher et utiliser l'archive tar téléchargée:

*    Le drapeau `x` lui indique que nous voulons extraire des fichiers d'une archive tar
*    Le drapeau `v` indique que nous voulons une sortie détaillée, y compris une liste des fichiers extraits
*    Le drapeau `f` indique tarque nous allons spécifier un nom de fichier sur lequel opérer

Maintenant, mettons les choses ensemble et exécutons la commande pour extraire le package:

    sudo tar -C /usr/local -xzf go1.15.3.linux-amd64.tar.gz

>Remarque: Bien qu'il s'agisse de **/usr/local/go** l'emplacement officiellement recommandé, certains utilisateurs peuvent préférer ou exiger des chemins différents.

À ce stade, l'utilisation de Go nécessiterait de spécifier le chemin d'accès complet à son emplacement d'installation dans la ligne de commande. Pour rendre l'interaction avec Go plus conviviale, nous allons définir quelques chemins.

### Définition des chemins d'accès

Dans cette étape, nous allons définir des chemins dans votre environnement.
Ajouter au fichier `~/.bashrc`  

    export PATH=$PATH:/usr/local/go/bin

>dans certain cas il faut utiliser le fichier ~/.profile

Ensuite, actualisez votre profil en exécutant:

    source ~/.bashrc

Version

    go version

```
go version go1.15.2 linux/amd64
```

Avec l'installation Go en place et les chemins d'environnement nécessaires définis, confirmons que notre configuration fonctionne en composant un programme court.


### Espace de travail

Ensuite, nous allons mettre en place un espace de travail pour le langage Go où nous allons sauver les constructions du langage Go.

Un espace de travail est une hiérarchie de répertoires avec trois répertoires à sa racine :

*    **src** contient les fichiers sources du langage Go,
*    **pkg** contient des paquets objets 
*    **bin** contient des commandes exécutables.

Créez la hiérarchie de répertoires ci-dessus pour l'espace de travail du langage Go en utilisant la commande :

    mkdir -p $HOME/go_projects/{src,pkg,bin}

Ici, `$HOME/go_projects` est le répertoire où Go va construire ses fichiers.

Ensuite, nous devons pointer Go vers le nouvel espace de travail.

Pour ce faire, éditez le fichier `~/.bashrc` :

    nano ~/.bashrc

Ajoutez les lignes suivantes au point Aller au répertoire du nouvel espace de travail.

```
export GOPATH="$HOME/go_projects"
export GOBIN="$GOPATH/bin"
```

Si le language Go est installé sur un autre emplacement que l'emplacement par défaut (/usr/local/), vous devez spécifier le chemin d'installation(GOROOT) dans le fichier `~/.bashrc`  
Par exemple, si vous avez installé go lang dans votre répertoire HOME, vous devrez ajouter les lignes suivantes dans le fichier de profil de l'utilisateur.

```
export GOROOT=$HOME/go
export PATH=$PATH:$GOROOT/bin
```

Veuillez noter que si vous avez installé Golang en utilisant des gestionnaires de paquets, le chemin d'installation sera soit `/usr/lib/go` soit `/usr/lib/golang`.  
Vous devez mettre à jour la valeur correcte du chemin dans GOROOT.

Une fois que vous avez spécifié les valeurs appropriées, exécutez la commande suivante pour mettre à jour les valeurs de l'environnement Go lang.

    source ~/.bashrc

Maintenant, exécutez les commandes suivantes pour vérifier si vous avez correctement installé et configuré le langage Go.

Vérifions la version installée :

    go version

Exemple de sortie :

    go version go1.15.3 linux/amd64

Pour consulter les informations sur l'environnement Go lang, exécutez :

    go env

```
GO111MODULE=""
GOARCH="amd64"
GOBIN="/home/yannick/go_projects/bin"
GOCACHE="/home/yannick/.cache/go-build"
GOENV="/home/yannick/.config/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOINSECURE=""
GOMODCACHE="/home/yannick/go_projects/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/home/yannick/go_projects"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GCCGO="gccgo"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build184421888=/tmp/go-build -gno-record-gcc-switches"
```

### Créer un programme simple

Nous savons maintenant comment installer et configurer le langage Go, créons un programme simple "hello world".

Créez un répertoire séparé pour stocker le code source de votre programme. La commande suivante permet de créer un répertoire pour stocker le code source du programme "hello world".

    mkdir -p $HOME/go_projects/src/hello

Créez un fichier appelé hello.go :

    nano $HOME/go_projects/src/hello/hello.go

Ajoutez les lignes suivantes :

```
package main

import "fmt"

func main() {
 fmt.Println("Hello, World")
}
```

Enregistrez et fermez le fichier. Ensuite, exécutez la commande suivante pour compiler le programme "hello world" :

    go install $HOME/go_projects/src/hello/hello.go

Enfin, lancez le programme "hello world" en utilisant la commande

    $GOBIN/hello

La sortie de l'échantillon serait :

    Hello, World

### Conclusion

En téléchargeant et en installant le dernier package Go et en définissant ses chemins, vous avez maintenant un système à utiliser pour le développement Go. Pour en savoir plus sur l'utilisation de Go, consultez notre série de développement [How To Code in Go ](https://www.digitalocean.com/community/tutorial_series/how-to-code-in-go). Vous pouvez également consulter la documentation officielle sur [ How to Write Go Code](https://golang.org/doc/code.html)

* [The Go Programming Language website](https://golang.org/)
* [How to write Go code](https://golang.org/doc/code.html)

Si vous ne souhaitez plus utiliser le langage Go, vous pouvez le désinstaller en supprimant simplement le répertoire Go, c'est-à-dire /usr/local/go. Vous pouvez également supprimer les répertoires de l'espace de travail.

## Go authentification

* [OAuth2 provider](https://docs.gitea.io/en-us/oauth2-provider/)
* [[Résolu] Drone + Gitea + Nginx (reverse proxy) : auth impossible sous drone](https://mondedie.fr/d/11239-resolu-drone-gitea-nginx-reverse-proxy-auth-impossible-sous-drone)
* [Basic Authentication in GoLang RESTful Web API](http://learningprogramming.net/golang/golang-restful-web-api/basic-authentication-in-golang-restful-web-api/)
* [GitHub OAuth 2 Tutorial](https://requests-oauthlib.readthedocs.io/en/latest/examples/github.html)
* [Go Oauth2 Tutorial](https://tutorialedge.net/golang/go-oauth2-tutorial/) - <https://github.com/TutorialEdge/go-oauth-tutorial>
* [Test Driven Learning - Go](https://ldez.github.io/blog/2015/12/04/test-driven-learning-go/)




