+++
title = 'Dig ,recherche DNS en interrogeant les serveurs de noms'
date = 2020-03-07 00:00:00 +0100
categories = ['cli', 'outils']
+++
# Comment utiliser la commande Dig sous Linux

*Dig ( [Domain Information Groper](https://en.wikipedia.org/wiki/Dig_(command)) ) est un utilitaire de ligne de commande qui effectue une recherche DNS en interrogeant les serveurs de noms et en affichant le résultat. Dans ce tutoriel, vous trouverez toutes les utilisations de base de la commande que vous devez connaître dans le système d'exploitation Linux.*

Par défaut, dig envoie la requête [DNS](https://www.hostinger.com/tutorials/what-is-dns) aux serveurs de noms répertoriés dans le résolveur (/etc/resolv.conf) à moins qu'il ne lui soit demandé d'interroger un serveur de noms spécifique.

## Installer Dig sur Linux

Dig fait partie du package d'utilitaires DNS qui est souvent installé avec des serveurs de noms BIND. Vous pouvez également installer le package d'utilitaires qui contient dig séparément en accédant à votre VPS via SSH et en utilisant les commandes suivantes dans la ligne de commande:

    apt-get install dnsutils  # Debian et Ubuntu
    yum install bind-utils    # CentOS 7
    pacman -S bind-utils      # archlinux
    
Une fois installé, vérifiez la version, pour vous assurer que la configuration s'est terminée avec succès:

    dig -v

## Syntaxe Dig

Dans sa forme la plus simple, la syntaxe de l'utilitaire dig ressemblera à ceci:

    dig [serveur] [nom] [type]

[serveur] - l'adresse IP ou le nom d'hôte du serveur de noms à interroger.

Si l'argument serveur est le nom d'hôte, dig résoudra le nom d'hôte avant de poursuivre l'interrogation du serveur de noms.  
Il est facultatif et si vous ne fournissez pas d'argument serveur, alors dig utilise le serveur de noms répertorié dans **/etc/resolv.conf** 

[nom] - le nom de l'enregistrement de ressource à rechercher.

[type] - le type de requête demandé par `dig`. Par exemple, il peut s'agir d'un enregistrement **A**, d'un enregistrement **MX**, d'un enregistrement **SOA** ou de tout autre type. Par défaut, `dig` effectue une recherche pour un enregistrement **A** si aucun argument de type n'est spécifié.

## Comment utiliser la commande Dig

Permet d'entrer dans les utilisations de base de la commande 

### dig un nom de domaine

Pour effectuer une recherche DNS pour un nom de domaine, passez simplement le nom avec la commande dig:

    dig hostinger.com 

Par défaut, la commande `dig` affichera l'enregistrement A lorsqu'aucune autre option n'est spécifiée. La sortie contiendra également d'autres informations telles que la version dig installée, des détails techniques sur les réponses, des statistiques sur la requête, une section de questions ainsi que quelques autres.

### Réponses courtes

La commande `dig` ci-dessus comprend de nombreuses informations utiles dans différentes sections, mais il peut arriver que vous ne souhaitiez que le résultat de la requête. Vous pouvez le faire en utilisant l'option `+short`, qui affichera l'adresse IP (enregistrement A) du nom de domaine uniquement:

    dig hostinger.com +short

### Réponses détaillées

Parfois, vous souhaitez afficher la section des réponses en détail. Par conséquent, pour une information détaillée sur la section des réponses, vous pouvez arrêter d'afficher toute la section en utilisant l'option `+noall` et interroger la section des réponses uniquement en utilisant l'option `+answer` avec la commande `dig`.

    dig hostinger.com +noall +answer

Spécification des serveurs de noms

Par défaut, les commandes `dig` interrogeront les serveurs de noms répertoriés dans **/etc/resolv.conf** pour effectuer une recherche DNS pour vous. Vous pouvez modifier ce comportement par défaut en utilisant le symbole `@` suivi d'un nom d'hôte ou d'une adresse IP du serveur de noms.

La commande dig suivante envoie la requête DNS au serveur de noms de Google (8.8.8.8) en utilisant l'option `@ 8.8.8.8` 

    dig @8.8.8.8 hostinger.com

### Interroger tous les types d'enregistrement DNS

Pour interroger tous les types d'enregistrement DNS disponibles associés à un domaine, utilisez l'option `ANY` qui inclura tous les types d'enregistrement disponibles dans la sortie:

    dig hostinger.com ANY

### Rechercher le type d'enregistrement

Si vous souhaitez rechercher un enregistrement spécifique, ajoutez simplement le type à la fin de la commande.

Par exemple, pour rechercher uniquement la section d'échange de courrier - **MX** - réponse associée à un domaine, vous pouvez utiliser la commande dig suivante:

    dig hostinger.in MX 

De même, pour afficher les autres enregistrements associés à un domaine, spécifiez le type d'enregistrement à la fin de la commande `dig`:

```
dig hostinger.com txt (Query TXT record)
dig hostinger.com cname (Query CNAME record)
dig hostinger.com ns (Query NS record)
dig hostinger.com A (Query A record)
```

### Tracer le chemin DNS

Dig permet de tracer le chemin de recherche DNS en utilisant l'option `+trace` . L'option effectue des requêtes itératives pour résoudre la recherche de nom. Il interrogera les serveurs de noms à partir de la racine et parcourt ensuite l'arborescence de l'espace de noms à l'aide de requêtes itératives suivant les références en cours de route:

    dig hostinger.com +trace

### Recherche DNS inversée

La recherche DNS inversée vous permet de rechercher le domaine et le nom d'hôte associés à une adresse IP. Pour effectuer une recherche DNS inversée à l'aide de la commande dig, utilisez l'option `-x` suivie de l'adresse IP choisie. Dans l'exemple suivant, dig effectuera une recherche DNS inversée pour l'adresse IP associée à google.com:

    dig +answer -x 172.217.166.46

N'oubliez pas que si un enregistrement PTR n'est pas défini pour une adresse IP, il n'est pas possible d'effectuer une recherche DNS inversée car l'enregistrement PTR pointe vers le domaine ou le nom d'hôte.

### Requêtes par lots

Avec l'utilitaire dig, vous pouvez effectuer une recherche DNS pour une liste de domaines au lieu de la faire  individuellement. Vous devez fournir à dig une liste de noms de domaine , un par ligne, dans un fichier. Une fois le fichier prêt, spécifiez son nom avec l'option `-f`

    nano domain_name.txt

```
hostinger.com
google.com
ubuntu.com
```

    dig -f domain_name.txt +short

### Contrôle du comportement de dig

La sortie de la commande peut être personnalisée en permanence en définissant des options dans le fichier  **~/.digrc** qui s'exécuteront automatiquement avec la commande.

Supposons que vous souhaitiez afficher uniquement la section des réponses - spécifiez les options requises dans le fichier **~/.digrc**, afin de ne pas avoir à les saisir lors de l'exécution de la requête.

    echo "+noall +answer" > ~/.digrc

Effectuez maintenant une recherche de serveur DNS pour un domaine. La sortie confirme que dig s'exécute avec les options définies dans le fichier **~/.digrc**

## Conclusion

C'est tout ce dont vous avez besoin pour commencer à utiliser **dig** In Linux. Vous pouvez désormais effectuer des recherches **DNS** pour les domaines à l'aide de diverses options.  
Consultez la page de manuel en utilisant la commande man dig pour découvrir toutes les utilisations et options possibles. 
