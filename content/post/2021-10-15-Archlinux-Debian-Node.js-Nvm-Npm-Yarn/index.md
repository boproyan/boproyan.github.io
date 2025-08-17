+++
title = 'Archlinux Debian , installation des paquets node npm nvm yarn'
date = 2021-10-15 00:00:00 +0100
categories = node
+++
![](Node_logo.png)  

# Archlinux - Node.js

[Node.js](http://nodejs.org/) est un environnement d'exécution JavaScript combiné avec des bibliothèques utiles. Il utilise le moteur [Google's V8 engine](https://v8.dev) pour exécuter du code en dehors du navigateur. Grâce à son modèle d'E/S non bloquant, piloté par les événements, il est adapté aux applications Web temps réel.

## NVM - Node Version Manager (Gérer les versions node.js) 

Il n'est pas rare d'avoir besoin ou envie de travailler dans différentes versions de **nodejs**. Une méthode privilégiée par les utilisateurs de node est l'utilisation de [NVM](https://github.com/creationix/nvm) (Node Version Manager). Le paquet **nvm** permet des installations alternatives faciles et peu coûteuses. 

### Installer nvm via le script

**Installer NVM en [utilisant le script d'installation et de mise à jour](https://github.com/nvm-sh/nvm#installation-and-update)**  

Pour installer ou mettre à jour nvm, vous devez exécuter le script d'installation. Pour ce faire, vous pouvez soit télécharger et exécuter le script manuellement, soit utiliser la commande cURL ou Wget suivante :

    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash
    wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash

L'exécution de l'une ou l'autre des commandes ci-dessus télécharge un script et le lance. Le script clone le dépôt **nvm** dans **~/.nvm**, et ajoute les lignes de source du snippet ci-dessous à votre profil (~/.bash_profile, ~/.zshrc, ~/.profile, ou ~/.bashrc).

    export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # Cela charge nvm

>Notes :  
- Si la variable d'environnement **$XDG_CONFIG_HOME** est présente, elle y placera les fichiers nvm.  
- Vous pouvez ajouter `--no-use` à la fin du script ci-dessus (`...nvm.sh --no-use`) pour reporter l'utilisation de nvm jusqu'à ce que vous l'utilisiez manuellement.  
- Vous pouvez personnaliser la source, le répertoire, le profil et la version d'installation en utilisant les variables **NVM_SOURCE**, **NVM_DIR, PROFILE** et **NODE_VERSION**. Ex : `curl ... | NVM_DIR="path/to/nvm"`. Assurez-vous que la variable **NVM_DIR** ne contient pas de barre oblique.

Exemple de résultat d'une installation via le script

```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13527  100 13527    0     0  21643      0 --:--:-- --:--:-- --:--:-- 21643
=> Downloading nvm from git to '/home/yannick/.nvm'
=> Clonage dans '/home/yannick/.nvm'...
remote: Enumerating objects: 288, done.
remote: Counting objects: 100% (288/288), done.
remote: Compressing objects: 100% (258/258), done.
remote: Total 288 (delta 35), reused 95 (delta 18), pack-reused 0
Réception d'objets: 100% (288/288), 146.70 Kio | 754.00 Kio/s, fait.
Résolution des deltas: 100% (35/35), fait.
=> Compressing and cleaning up git repository

=> Appending nvm source string to /home/yannick/.bashrc
=> Appending bash_completion source string to /home/yannick/.bashrc
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

>Note : Sous Linux, après avoir exécuté le script d'installation, si vous obtenez **nvm: command not found** ou si vous ne voyez aucun retour de votre terminal après avoir tapé `command -v nvm`, fermez simplement votre terminal actuel, ouvrez un nouveau terminal et essayez de vérifier à nouveau.

*NB. L'installateur peut utiliser git, curl, ou wget pour télécharger nvm et tout ce qui est disponible.*

### Mise à jour NVM

Pour une mise à jour manuelle avec git (nécessite git v1.7.10+) :

1.    changez dans le $NVM_DIR
2.    récupérez les dernières modifications
3.    vérifiez la dernière version
4.    activez la nouvelle version

```
cd "$NVM_DIR"
git fetch --tags origin
git checkout `git describe --abbrev=0 --tags --match "v[0-9]*" $(git rev-list --tags --max-count=1)`
```

### Utiliser NVM

L'utilisation est bien documentée sur le [GitHub du projet](https://github.com/nvm-sh/nvm)    

```
Node Version Manager (v0.35.2)

Note: <version> refers to any version-like string nvm understands. This includes:
  - full or partial version numbers, starting with an optional "v" (0.10, v0.1.2, v1)
  - default (built-in) aliases: node, stable, unstable, iojs, system
  - custom aliases you define with `nvm alias foo`

 Any options that produce colorized output should respect the `--no-colors` option.

Usage:
  nvm --help                                Show this message
  nvm --version                             Print out the installed version of nvm
  nvm install [-s] <version>                Download and install a <version>, [-s] from source. Uses .nvmrc if available
    --reinstall-packages-from=<version>     When installing, reinstall packages installed in <node|iojs|node version number>
    --lts                                   When installing, only select from LTS (long-term support) versions
    --lts=<LTS name>                        When installing, only select from versions for a specific LTS line
    --skip-default-packages                 When installing, skip the default-packages file if it exists
    --latest-npm                            After installing, attempt to upgrade to the latest working npm on the given node version
    --no-progress                           Disable the progress bar on any downloads
  nvm uninstall <version>                   Uninstall a version
  nvm uninstall --lts                       Uninstall using automatic LTS (long-term support) alias `lts/*`, if available.
  nvm uninstall --lts=<LTS name>            Uninstall using automatic alias for provided LTS line, if available.
  nvm use [--silent] <version>              Modify PATH to use <version>. Uses .nvmrc if available
    --lts                                   Uses automatic LTS (long-term support) alias `lts/*`, if available.
    --lts=<LTS name>                        Uses automatic alias for provided LTS line, if available.
  nvm exec [--silent] <version> [<command>] Run <command> on <version>. Uses .nvmrc if available
    --lts                                   Uses automatic LTS (long-term support) alias `lts/*`, if available.
    --lts=<LTS name>                        Uses automatic alias for provided LTS line, if available.
  nvm run [--silent] <version> [<args>]     Run `node` on <version> with <args> as arguments. Uses .nvmrc if available
    --lts                                   Uses automatic LTS (long-term support) alias `lts/*`, if available.
    --lts=<LTS name>                        Uses automatic alias for provided LTS line, if available.
  nvm current                               Display currently activated version of Node
  nvm ls [<version>]                        List installed versions, matching a given <version> if provided
    --no-colors                             Suppress colored output
    --no-alias                              Suppress `nvm alias` output
  nvm ls-remote [<version>]                 List remote versions available for install, matching a given <version> if provided
    --lts                                   When listing, only show LTS (long-term support) versions
    --lts=<LTS name>                        When listing, only show versions for a specific LTS line
    --no-colors                             Suppress colored output
  nvm version <version>                     Resolve the given description to a single local version
  nvm version-remote <version>              Resolve the given description to a single remote version
    --lts                                   When listing, only select from LTS (long-term support) versions
    --lts=<LTS name>                        When listing, only select from versions for a specific LTS line
  nvm deactivate                            Undo effects of `nvm` on current shell
  nvm alias [<pattern>]                     Show all aliases beginning with <pattern>
    --no-colors                             Suppress colored output
  nvm alias <name> <version>                Set an alias named <name> pointing to <version>
  nvm unalias <name>                        Deletes the alias named <name>
  nvm install-latest-npm                    Attempt to upgrade to the latest working `npm` on the current node version
  nvm reinstall-packages <version>          Reinstall global `npm` packages contained in <version> to current version
  nvm unload                                Unload `nvm` from shell
  nvm which [current | <version>]           Display path to installed node version. Uses .nvmrc if available
  nvm cache dir                             Display path to the cache directory for nvm
  nvm cache clear                           Empty cache directory for nvm

Example:
  nvm install 8.0.0                     Install a specific version number
  nvm use 8.0                           Use the latest available 8.0.x release
  nvm run 6.10.3 app.js                 Run app.js using node 6.10.3
  nvm exec 4.8.3 node app.js            Run `node app.js` with the PATH pointing to node 4.8.3
  nvm alias default 8.1.0               Set default node version on a shell
  nvm alias default node                Always default to the latest available node version on a shell

Note:
  to remove, delete, or uninstall nvm - just remove the `$NVM_DIR` folder (usually `~/.nvm`)
```

En mode utilisateur prompt $

Installer version [Node](https://nodejs.org/en/) LTS 

    nvm ls-remote --lts  # liste des versions LTS
    nvm install --lts # Installe la dernière version node LTS

```
Installing latest LTS version.
Downloading and installing node v12.19.0...
Downloading https://nodejs.org/dist/v12.19.0/node-v12.19.0-linux-x64.tar.xz...
######################################################################################################################### 100,0%
Computing checksum with sha256sum
Checksums matched!
Now using node v12.19.0 (npm v6.14.4)
```

Installer la dernière version 

    nvm ls-remote  # liste des versions 
    nvm install --latest-npm # Installe la dernière version node

```
No .nvmrc file found
Downloading and installing node v15.0.1...
Downloading https://nodejs.org/dist/v15.0.1/node-v15.0.1-linux-x64.tar.xz...
######################################################################################################################### 100,0%
Computing checksum with sha256sum
Checksums matched!
Now using node v15.0.1 (npm v6.14.4)
nvm_ensure_default_set: a version is required
Attempting to upgrade to the latest working version of npm...
* Installing latest `npm`; if this does not work on your node version, please report a bug!
/home/yannick/.nvm/versions/node/v15.0.1/bin/npm -> /home/yannick/.nvm/versions/node/v15.0.1/lib/node_modules/npm/bin/npm-cli.js
/home/yannick/.nvm/versions/node/v15.0.1/bin/npx -> /home/yannick/.nvm/versions/node/v15.0.1/lib/node_modules/npm/bin/npx-cli.js
+ npm@6.14.8
added 237 packages from 74 contributors, removed 51 packages and updated 197 packages in 9.365s
* npm upgraded to: v6.14.4
```

Liste des versions node installées

    nvm list

```
       v12.14.0
->     v12.19.0
        v15.0.1
         system
default -> 12.14 (-> v12.14.0)
node -> stable (-> v15.0.1) (default)
stable -> 15.0 (-> v15.0.1) (default)
iojs -> N/A (default)
unstable -> N/A (default)
lts/* -> lts/erbium (-> v12.19.0)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.22.1 (-> N/A)
lts/erbium -> v12.19.0
```

Choix de la version à utiliser

    nvm use 15.0

```
Now using node v15.0.1 (npm v6.14.4)
```

Liste des versions node disponibles

    nvm ls-remote

```
[...]
       v15.10.0
       v15.11.0
       v15.12.0
       v15.13.0
       v15.14.0
        v16.0.0
        v16.1.0
        v16.2.0
        v16.3.0
        v16.4.0
        v16.4.1
        v16.4.2
        v16.5.0
        v16.6.0
        v16.6.1
```

Installer et utiliser une version précise

    nvm install 16.6.1

Si vous installez plusieurs versions de node.js à l'aide de nvm (gestionnaire de versions de node), vous pouvez utiliser l'une des versions installées à l'aide de la commande suivante.

    nvm use 16.6.1

Vous devez définir une version de node par défaut en utilisant :

    nvm alias default 16.6.1

Ici 16.6.1 est le numéro de version de node et le changement persiste même après la fermeture du terminal.

**.nvmrc**

Vous pouvez créer un fichier `.nvmrc` contenant un numéro de version de node (ou toute autre chaîne que nvm comprend ; voir nvm --help pour plus de détails) dans le répertoire racine du projet (ou tout répertoire parent).
Ensuite, nvm use, nvm install, nvm exec, nvm run, et nvm which utiliseront la version spécifiée dans le fichier .nvmrc si aucune version n'est fournie sur la ligne de commande.

Par exemple, pour que nvm utilise par défaut la dernière version 5.9, la dernière version LTS, ou la dernière version du noeud pour le répertoire actuel :

    echo "5.9" > .nvmrc
    echo "lts/*" > .nvmrc # pour utiliser par défaut la dernière version LTS
    echo "node" > .nvmrc # pour utiliser par défaut la dernière version.

Ensuite, lorsque vous exécutez nvm :

    nvm use

```
Found '/home/yann/.nvmrc' with version <node>
Now using node v16.11.1 (npm v8.0.0)
```

nvm use et. al. vont parcourir la structure des répertoires vers le haut à partir du répertoire courant à la recherche du fichier `.nvmrc`  
En d'autres termes, l'exécution de nvm use et. al. dans n'importe quel sous-répertoire d'un répertoire contenant un `.nvmrc` entraînera l'utilisation de ce `.nvmrc`  
Le contenu d'un fichier `.nvmrc` doit être la `<version>` (telle que décrite par nvm --help) suivie d'une nouvelle ligne.  
Aucun espace de fin n'est autorisé, et le saut de ligne de fin est obligatoire.
{: .prompt-info }

**Intégration dans le shell**

Si vous préférez une solution plus légère, les recettes ci-dessous ont été contribuées par des utilisateurs de nvm.  
Appeler automatiquement `nvm use`

Mettez ce qui suit à la fin de votre $HOME/.bashrc :

```
cdnvm() {
    cd "$@";
    nvm_path=$(nvm_find_up .nvmrc | tr -d '\n')

    # If there are no .nvmrc file, use the default nvm version
    if [[ ! $nvm_path = *[^[:space:]]* ]]; then

        declare default_version;
        default_version=$(nvm version default);

        # If there is no default version, set it to `node`
        # This will use the latest version on your machine
        if [[ $default_version == "N/A" ]]; then
            nvm alias default node;
            default_version=$(nvm version default);
        fi

        # If the current version is not the default version, set it to use the default version
        if [[ $(nvm current) != "$default_version" ]]; then
            nvm use default;
        fi

        elif [[ -s $nvm_path/.nvmrc && -r $nvm_path/.nvmrc ]]; then
        declare nvm_version
        nvm_version=$(<"$nvm_path"/.nvmrc)

        declare locally_resolved_nvm_version
        # `nvm ls` will check all locally-available versions
        # If there are multiple matching versions, take the latest one
        # Remove the `->` and `*` characters and spaces
        # `locally_resolved_nvm_version` will be `N/A` if no local versions are found
        locally_resolved_nvm_version=$(nvm ls --no-colors "$nvm_version" | tail -1 | tr -d '\->*' | tr -d '[:space:]')

        # If it is not already installed, install it
        # `nvm install` will implicitly use the newly-installed version
        if [[ "$locally_resolved_nvm_version" == "N/A" ]]; then
            nvm install "$nvm_version";
        elif [[ $(nvm current) != "$locally_resolved_nvm_version" ]]; then
            nvm use "$nvm_version";
        fi
    fi
}
alias cd='cdnvm'
cd $PWD
```


## NPM - Node Packaged Modules (gestionnaire de paquets node.js) 

[npm](https://www.npmjs.org/) est le gestionnaire de paquets officiel pour **node.js**. Il peut être installé avec le paquet **npm**

### Gérer les paquets avec npm 

Tout paquet peut être installé en utilisant :

    $ npm install NomDuPaquet

Cette commande installe le paquet dans le répertoire courant sous **node_modules** et les exécutables sous **node_modules/.bin**

Pour une installation à l'échelle du système, le commutateur global `-g` peut être utilisé :

    # npm -g install packageName

Par défaut, cette commande installe le paquet sous **/usr/lib/node_modules/npm** et nécessite les privilèges "root" de l'administrateur.

#### Autoriser les installations pour l'ensemble des utilisateurs 

Pour permettre l'installation de paquets *globale* pour l'utilisateur courant, définissez le npm_config_prefix. Cette variable est utilisée par npm et [yarn](https://yarnpkg.com/en/).


    ~/.profile
    PATH="$HOME/.node_modules/bin:$PATH"
    export npm_config_prefix=~/.node_modules


Re-connexion ou `source` pour mettre à jour les changements.

Vous pouvez aussi spécifier le paramètre `--prefix` pour `npm install`.  
Cependant, ceci est ''non'' recommandé, puisque vous devrez l'ajouter à chaque fois que vous installerez un paquet global.

    $ npm -g install packageName --prefix ~/.node_modules

#### Mise à jour des paquets 

La mise à jour des paquets est aussi simple que 

    $ npm update packageName

Pour le cas de paquets installés globalement `-g`

    # npm update -g packageName


>Note : les paquets installés globalement nécessitent des privilèges d'administrateur à moins que `|prefix` ne soit défini sur un répertoire accessible en écriture par l'utilisateur}}

**Mise à jour de tous les paquets**

Cependant, parfois vous pouvez simplement mettre à jour tous les paquets, soit localement soit globalement. Si vous laissez le nom du paquet npm, vous tenterez de mettre à jour tous les paquets

    $ npm update

ou ajouter le drapeau `-g` pour mettre à jour les paquets installés globalement

    # npm update -g

#### Supprimer un paquets 

Pour supprimer un paquet installé avec le commutateur `-g`, il suffit de l'utiliser :

 # npm -g uninstall packageName

>Rappelez-vous que les paquets installés globalement nécessitent des privilèges d'administrateur

pour retirer un paquet local, déposez le commutateur et exécutez :

    $ npm uninstall packageName

#### Lister les paquets 

Pour afficher une vue arborescente des paquets globaux installés, utilisez :

    $ npm -g list

Cet arbre est souvent assez profond. Pour afficher uniquement les paquets de niveau supérieur, utilisez :

    $ npm list --depth=0

Pour afficher les paquets obsolètes qui peuvent avoir besoin d'être mis à jour :

    $ npm outdated

### Les paquets situés dans "Arch User Repository" (AUR) 

Certains paquets node.js peuvent être trouvés dans [Arch User Repository](https://wiki.archlinux.org/index.php/Arch_User_Repository) avec le nom **nodejs-packageName**

## Dépannage 

### node-gyp python errors 

Certains modules de nodes utilisent l'utilitaire **node-gyp** qui ne supporte pas Python 3, qui dans la plupart des cas sera l'exécutable Python par défaut pour l'ensemble du système. Pour résoudre de telles erreurs, assurez-vous que vous avez **python2** installé, puis définissez le Python npm par défaut comme tel :

    $ npm config set python /usr/bin/python2

En cas d'erreurs comme *gyp WARN EACCES l'utilisateur "root" n'a pas la permission d'accéder au ... dir*, l'option `--unsafe-perm` pourrait aider :

    # npm install --unsafe-perm -g node-inspector

### Cannot find module ... errors 

Depuis npm 5.x.x., le fichier package-lock.json est généré avec le fichier package.json. Des conflits peuvent survenir lorsque les deux fichiers font référence à des versions différentes du paquet. Une méthode temporaire pour résoudre ce problème a été :

    $ rm package-lock.json
    $ npm install

Cependant, des corrections ont été apportées après le npm 5.1.0 ou plus. Pour plus d'informations, voir [dépendances manquantes](https://github.com/npm/npm/pull/17508)

## Liens 

Pour plus d'informations sur Node.js et l'utilisation de son gestionnaire de paquets officiel NPM, vous pouvez consulter les ressources externes suivantes

* [Node.js documentation](https://nodejs.org/en/docs/)
* [NPM documentation]https://docs.npmjs.com/)
* Canal IRC #node.js sur irc.freenode.net

# Debian - Node.js

**Comment installer Nodejs & NPM sur Debian** , écrit par [Rahul](https://tecadmin.net/author/myadmin/) 

## Node.js

Node.js est une plate-forme construite sur le moteur JavaScript V8 de Chrome. Nodejs peut être utilisé pour créer facilement des applications réseau rapides et évolutives. La dernière version de node.js ppa est maintenue par son site officiel. Nous pouvons ajouter ce PPA aux systèmes Debian 10 (Buster) , Debian 9 (Étirer) Debian 8 (Jessie) et Debian 7 (Wheezy) . Utilisez ce tutoriel pour installer les dernières versions de Nodejs & NPM sur les systèmes Debian 10/9/8/7.

Pour installer une version spécifique de nodejs, consultez notre tutoriel [Installer la version spécifique de Nodejs avec NVM (english)](https://tecadmin.net/install-nodejs-with-nvm/) .

### Ajouter le PPA de Node.js

Vous devez ajouter Node.js PPA à votre système, fourni par le site officiel de Nodejs (<https://github.com/nodesource/distributions#debinstall>). Nous devons également installer le paquet software-properties-common s'il n'est pas déjà installé. Vous pouvez choisir d'installer la version la plus récente de Node.js ou la version LTS.

    sudo apt-get install curl software-properties-common

Pour la dernière version Node.js (v14.x au 11 octobre 2020)

```bash
# Using Debian, as root
sudo -s
curl -sL https://deb.nodesource.com/setup_14.x | bash -
```

Version Node.js LTS (v12.x au 11 octobre 2020

```bash
# Using Debian, as root
sudo -s
curl -sL https://deb.nodesource.com/setup_lts.x | bash -
```


### Installez Node.js sur Debian

Après avoir ajouté le fichier PPA requis, installons le paquet Nodejs. NPM sera également installé avec node.js. Cette commande installera également de nombreux autres packages dépendants sur votre système.

    sudo apt install nodejs

>NE PAS OUBLIER d'installer **Yarn** 

### Version Node.js

Une fois l'installation terminée, vérifiez et vérifiez la version installée de Node.js et NPM. Vous pouvez trouver plus de détails sur la version actuelle sur le site officiel de node.js.

    node -v

v12.4.0  # LTS
v14.13.1 # au 11/10/2020

Vérifiez également la version de NPM.

    npm -v

6.14.8

### Créer un serveur Web de démonstration (facultatif)

Ceci est une étape optionnelle. Si vous souhaitez tester votre installation de node.js. Créons un serveur Web avec le texte “Hello World!”. Créez un fichier http_server.js

    nano http_server.js

et ajouter le contenu suivant

```
var http = require('http');
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(3000, "127.0.0.1");
console.log('Server running at http://127.0.0.1:3000/');
```

Maintenant, démarrez le serveur Web en utilisant la commande.

    node http_server.js

Server running at http://127.0.0.1:3000/

Le serveur Web a été démarré sur le port 3000. Accédez maintenant à http://127.0.0.1:3000/ url dans le navigateur. 

## Yarn

### Comment installer Yarn sur Ubuntu, Debian et LinuxMint

Le "yarn" est un logiciel avancé de gestion de paquets pour les applications Node.js. C'est une alternative rapide, sécurisée et fiable que tout autre gestionnaire de paquets Nodejs. J'ai installé le dernier fichier Node.js sur les systèmes Ubuntu et Debian .

Ce didacticiel contient trois méthodes pour installer Yarn sur des systèmes Ubuntu, Debian et LinuxMint. Utilisez l'une des méthodes suivantes:

### Méthode 1 - Installer Yarn à l'aide de NPM

Un paquet de fils est disponible pour être installé avec NPM. Vous pouvez simplement utiliser la commande npm comme suit pour installer Yarn globalement. Pour installer le fil pour le projet actuel, supprimez simplement l'option -g de la commande.

    sudo npm install yarn -g

Vérifier la version installée:

    yarn -v

1.16.0

### Méthode 2 - Installer Yarn à l'aide d'un script

Yarn fournit également un [script shell](https://tecadmin.net/tutorial/bash-scripting/) pour l'installation. C'est le moyen le plus recommandé d'installer Yarn sur un système Linux. Ce script télécharge l'archive et les extraits de fils dans le répertoire .yarn de votre répertoire personnel. Définissez également la variable d’environnement PATH.

    curl -o- -L https://yarnpkg.com/install.sh | bash

Comme cette installation place tous les fichiers dans le répertoire personnel de l'utilisateur, elle est donc disponible uniquement pour l'utilisateur actuel.

### Méthode 3 - Installer le fil à l'aide d'Apt-get

L'équipe Yarn fournit également un [référentiel Apt](https://tecadmin.net/add-apt-repository-ubuntu/) pour installer le fil sur une machine Debian.  
Exécutez les commandes suivantes pour importer la clé gpg et configurer le référentiel yarn apt.

    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

Tapez maintenant les commandes suivantes pour installer le fil sur Ubuntu Debian et Linux.

    sudo apt-get update && sudo apt-get install yarn
