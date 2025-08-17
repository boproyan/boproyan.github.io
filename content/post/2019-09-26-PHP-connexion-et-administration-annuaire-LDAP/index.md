+++
title = 'PHP connexion et administration annuaire LDAP'
date = 2019-09-26 00:00:00 +0100
categories = ['serveur']
+++
## PHP - Connexion à un annuaire LDAP

### Introduction à LDAP

PHP permet la connexion et l'envoi de requêtes sur un annuaire LDAP, c'est-à-dire un serveur permettant de stocker des informations de manière hiérarchique.

Un serveur LDAP est conçu pour être capable de gérer les opérations suivantes :

*    établir la connexion avec l'annuaire
*    rechercher des entrées
*    comparer des entrées
*    ajouter des entrées
*    modifier des entrées
*    supprimer des entrées
*    annuler ou abandonner une opération
*    fermer la connexion avec l'annuaire

Ainsi PHP fournit un ensemble de fonctions (pour peu que le module LDAP soit installé)
permettant de réaliser ces opérations. Ces fonctions seront explicitées au long de l'article.

Avant de commencer, il faut installer le module LDAP pour le langage PHP. Ce module ajoute un certain nombre de fonctions qui débutent par « ldap_ » et qui permettent d'interfacer le serveur. 

### Séquence d'interrogation avec LDAP

L'interrogation d'un serveur LDAP avec PHP se fait selon une séquence simple,
nécessitant un nombre peu élevé de fonctions spécialisées. La séquence basique est la suivante :

*    Etablissement de la connexion avec le serveur LDAP
*    Liaison et authentification sur le serveur (appelé bind en anglais)
*    Recherche d'une entrée (ou bien une autre opération)
*    Exploitation des résultats (uniquement dans le cas d'une recherche)
*    Fermeture de la connexion


### Connexion au serveur LDAP

Avant de pouvoir interroger le serveur LDAP, il est essentiel d'initier la connexion.
Pour cela l'interpréteur PHP a besoin de connaître quelques renseignements
relatifs à l'annuaire :

*    L'adresse du serveur
*    Le port sur lequel le serveur fonctionne
*    La racine de l'annuaire
*    Le login de l'utilisateur (généralement root) ainsi que son mot de passe


```php
<?
// Fichier de configuration pour l'interface PHP
// de notre annuaire LDAP
$server = "localhost";

$port = "389";

$racine = "o=commentcamarche, c=fr";

$rootdn = "cn=ldap_admin, o=commentcamarche, c=fr";

$rootpw = "secret";

?>
```

On définit donc cinq variables pour caractériser le serveur LDAP :
le nom du serveur, le port (389 par défaut), la racine supérieure de l'arborescence, la chaîne de connexion pour l'administrateur ainsi que son mot de passe.


La première fonction à utiliser est la fonction `ldap_connect()` permettant d'établir
une liaison avec le serveur. Sa syntaxe est la suivante :

    int ldap_connect ([string hostname [, int port]])



Cette fonction admet en paramètre le nom du serveur (éventuellement le port. Par défaut le port est 389). En cas d'échec cette fonction retourne 0 sinon elle retourne un entier permettant d'identifier le serveur et nécessaire dans les fonctions suivantes.


Voici un exemple de connexion au serveur :

```php
<?
echo "Connexion...<br>";

$ds=ldap_connect($server);

?>
```

Toutefois cette opération n'est pas suffisante pour pouvoir exécuter des opérations sur le serveur LDAP. En effet, il est nécessaire d'initier la liaison (en anglais **to bind**) avec le serveur LDAP à l'aide de la fonction `ldap_bind()` dont la syntaxe est la suivante :

    int ldap_bind (int identifiant [, string bind_rdn [, string bind_password]])

Cette fonction attend en paramètre l'identifiant du serveur retourné ainsi qu'éventuellement le Nom distingué relatif de l'utilisateur (*RDN - Relative Distinguished Name*) et son mot de passe. Si l'utilisateur et le mot de passe ne sont pas précisés, la connexion se fait de manière anonyme.

La déconnexion du serveur LDAP se fait tout naturellement par la fonction `ldap_close()` avec la syntaxe suivante :

    int ldap_close (int identifiant)

Cette fonction est similaire à la fonction `ldap_unbind()`.

Voici un exemple complet de connexion et de déconnexion à un serveur LDAP :

```php
<?
// Fichier de configuration pour l'interface PHP
// de notre annuaire LDAP
$server = "localhost";

$port = "389";

$racine = "o=commentcamarche, c=fr";

$rootdn = "cn=ldap_admin, o=commentcamarche, c=fr";

$rootpw = "secret";

echo "Connexion...<br>";

$ds=ldap_connect($server);

if ($ds==1)
{
// on s'authentifie en tant que super-utilisateur, ici, ldap_admin
$r=ldap_bind($ds,$rootdn,$rootpw);

// Ici les opérations à effectuer
echo "Déconnexion...<br>";

ldap_close($ds);

}
else {
echo "Impossible de se connecter au serveur LDAP";

}
?>
```

### Exécution des opérations


#### Ajouter une entrée avec ldap_add()

La fonction `ldap_add()` permet d'ajouter des entrées à un annuaire LDAP
auquel on s'est préalablement connecté (et lié). Voici sa syntaxe :

    int ldap_add (int identifiant, string dn, array entry)

La fonction `ldap_add()` admet en paramètre l'identifiant du serveur LDAP retourné par la fonction
`ldap_connect()` ainsi que le nom distingué de l'entrée (c'est-à-dire son emplacement dans l'annuaire) et un tableau contenant l'ensemble des valeurs des attributs de l'entrée.


Lorsqu'un attribut est multivalué (c'est-à-dire lorsqu'un attribut est lui-même composé de plusieurs valeurs), celles-ci sont indexées dans une case du tableau (les indices du tableau commencent à 0). Dans l'exemple ci-dessous par exemple, l'attribut "mail" possède plusieurs valeurs :

```php
<?php
$ds=ldap_connect($server); // On suppose que le serveur LDAP est sur cet hote
if ($ds) {
$r=ldap_bind($ds,$rootdn,$rootpw);

// preparation des données
$entry["cn"]="Pillou";

$entry["sn"]="Jean-Francois";

$entry["mail"][0]="webmaster@commentcamarche.net";

$entry["mail"][1]="Jeff@commentcamarche.net";

$entry["objectclass"]="person";

// Ajout des données dans l'annuaire
$r=ldap_add($ds, "cn=Jean-Francois Pillou, o=commentcamarche, c=fr", $entry);

ldap_close($ds);

} else {
echo "Connexion au serveur LDAP impossible";

}

?>
```

#### Comparer une entrée avec ldap_compare()

La fonction `ldap_compare()` permet de comparer la valeur d'un attribut d'une entrée de l'annuaire LDAP, auquel on s'est préalablement connecté (et lié), à une valeur passée en paramètre. Voici la syntaxe de la fonction ldap_compare() :

    int ldap_compare (int identifiant, string dn, string attribut, string valeur)

La fonction `ldap_compare()` admet en paramètre l'identifiant du serveur LDAP retourné par la fonction `ldap_connect()`, le nom distingué de l'entrée (c'est-à-dire son emplacement dans l'annuaire) ainsi que le nom de l'attribut de l'entrée et la valeur à laquelle on veut comparer sa valeur dans l'annuaire.

En cas d'erreur, la fonction `ldap_compare()` renvoie la valeur -1, en cas de réussite elle renvoie TRUE, enfin dans le cas contraire elle renvoie FALSE.

L'exemple suivant (largement inspiré de celui de www.php.net) montre comment comparer un mot de passe avec celui stocké dans l'annuaire pour un utilisateur donné :

```php
<?php
$ds=ldap_connect($server);

if ($ds) {
$r=ldap_bind($ds,$rootdn,$rootpw);

// preparation des données
$dn="cn=Pillou Jean-Francois, o=commentcamarche,c=fr";

$valeur="MonMot2Passe";

$attribut="password";

// Comparaison du mot de passe à celui dans l'annuaire
$resultat=ldap_compare($ds, $dn, $attribut, $valeur);

if ($resultat == -1) {
echo "Erreur:".ldap_error($ds);

}
elseif ($resultat == TRUE) {
echo "Le mot de passe est correct";

}
else ($resultat == FALSE) {
echo "Le mot de passe est erronné...";

}
ldap_close($ds);

} else {
echo "Connexion au serveur LDAP impossible";

}

?>
```

>Notez l'utilisation de la fonction `ldap_error()` sur laquelle nous reviendrons ultérieurement pour afficher les détails de l'erreur !

### Supprimer une entrée avec ldap_delete()

La fonction `ldap_delete()` permet de supprimer des entrées d'un annuaire LDAP. Voici sa syntaxe :

    int ldap_delete (int identifiant, string dn)

La fonction ldap_delete() admet uniquement deux paramètres :

* l'identifiant du serveur LDAP retourné par la fonction `ldap_connect()`
* le nom distingué de l'entrée à supprimer.

Une fois de plus, cette fonction renvoie *TRUE* en cas de réussite, `FALSE` en cas d'échec.

L'exemple suivant illustre la suppression d'un élément de l'annuaire :

```php
<?php
$ds=ldap_connect($server); // On suppose que le serveur LDAP est sur cet hote
if ($ds) {
$r=ldap_bind($ds,$rootdn,$rootpw);

// preparation des données
$dn="Pillou Jean-Francois,o=commentcamarche,c=fr";

// Supression de l'entrée de l'annuaire
$r=ldap_delete($ds, $dn);

ldap_close($ds);

} else {
echo "Connexion au serveur LDAP impossible";

}

?>
```

### Modifier une entrée avec ldap_modify()

La fonction `ldap_modify()` permet de modifier une entrée de l'annuaire LDAP. Sa syntaxe est la même que celle de la fonction ldap_add() :

    int ldap_modify (int identifiant, string dn, array entry)

La fonction `ldap_modify()` admet en paramètre l'identifiant du serveur LDAP retourné par la fonction `ldap_connect()` ainsi que le nom distingué de l'entrée (c'est-à-dire son emplacement dans l'annuaire) et un tableau contenant l'ensemble des valeurs des attributs de l'entrée à modifier.

Lorsqu'un attribut est multivalué (c'est-à-dire lorsqu'un attribut est lui-même composé de plusieurs valeurs), celles-ci sont indexées dans une case du tableau (les indices du tableau commencent à 0). Dans l'exemple ci-dessous par exemple, l'attribut
"mail" possède plusieurs valeurs :

```php
<?php
$ds=ldap_connect($server); // On suppose que le serveur LDAP est sur cet hote
if ($ds) {
$r=ldap_bind($ds,$rootdn,$rootpw);

// preparation des données
$entry["cn"]="Pillou";

$entry["sn"]="Jean-Francois";

$entry["mail"][0]="webmaster@commentcamarche.net";

$entry["mail"][1]="Jeff@commentcamarche.net";

$entry["objectclass"]="person";

// Ajout des données dans l'annuaire
$r=ldap_modify($ds, "cn=Jean-Francois Pillou, o=commentcamarche, c=fr", $entry);

ldap_close($ds);

} else {
echo "Connexion au serveur LDAP impossible";

}

?>
```

### Rechercher une entrée avec ldap_search()

La recherche d'entrée dans l'annuaire est sans aucun doute la fonction la plus utile
parmi les fonctions LDAP de PHP¨car un annuaire est prévu pour être plus souvent
sollicité en lecture (recherche) qu'en écriture (ajout/suppression/modification).

La fonction `ldap_search()` permet de rechercher une ou plusieurs entrées de l'annuaire LDAP à l'aide du DN de base, c'est-à-dire le niveau de l'annuaire à partir duquel la recherche est effectuée, ainsi qu'un filtre représentant le type de recherche que l'on désire effectuer. Sa syntaxe est la suivante :

```
int ldap_search (int identifiant, string base_dn,
string filter [, array attributs [, int attrsonly [,
int sizelimit [, int timelimit [, int deref]]]]])
```

La fonction `ldap_search()` admet en paramètre l'identifiant du serveur LDAP retourné par la fonction `ldap_connect()` ainsi que le nom distingué du dossier de base (c'est-à-dire celui à partir duquel la recherche doit s'effectuer) et le filtre de la recherche. La fonction ldap_search() est par défaut configurée avec l'option de récursivité ***LDAP_SCOPE_SUBTREE*** ce qui signifie que la recherche se fait dans toutes les branches filles du dossier de base.


Le paramètre ***attributs*** permet de restreindre les attributs et les valeurs retournées, c'est-à-dire qu'il s'agit d'un tableau contenant le nom des attributs (chaînes de caractères) des attributs que l'on désire utiliser. Par défaut l'intégralité des attributs des entrées est renvoyée par le serveur, ce qui peut donner un nombre de données très important.

Le paramètre ***attrsonly*** permet de demander à l'annuaire de retourner uniquement les types d'attributs et non leurs valeurs lorsqu'il vaut 1. Par défaut (ou lorsque ce paramètre vaut 0) les types des attributs ainsi que leurs valeurs sont retournés par le serveur.

Le sixième paramètre ***sizelimit*** comme son nom l'indique permet de limiter le nombre maximum de résultat retourné par l'annuaire afin de réduire le volume des données retournées. Il faut noter que si le serveur est configuré pour retourner moins de résultats, une valeur supérieure de l'attribut ne permettra pas de dépasser la valeur inscrite dans la configuration du serveur. La valeur ***0*** indique qu'aucune limite autre que celle imposée par le serveur n'est définie.

Le septième paramètre ***timelimit*** permet de limiter le temps maximal de la recherche pris par le serveur. Il faut noter que si le serveur est configuré pour retourner moins de résultats, une valeur supérieure de l'attribut ne permettra pas de dépasser la valeur inscrite dans la configuration du serveur. La valeur 0 indique qu'aucune limite autre que celle imposée par le serveur n'est définie.

Enfin le huitième paramètre ***deref*** permet d'indiquer selon sa valeur la façon de procéder avec les alias lors de la recherche. Les valeurs possibles de ce paramètre sont les suivantes :

*    ***LDAP_DEREF_NEVER*** : les alias ne sont jamais déréférencés. Il s'agit de la valeur par défaut.
*    ***LDAP_DEREF_SEARCHING*** : les alias sont déréférencés uniquement pendant la recherche et non pendant leur localisation.
*    ***LDAP_DEREF_FINDING*** : les alias sont déréférencés uniquement pendant leur localisation et non lors de la recherche.
*    ***LDAP_DEREF_ALWAYS*** : les alias sont toujours déréférencés.


L'exemple suivant permet de connaître le nombre de résultats retournés pour une recherche d'une personne dont le nom ou le prenom commence par la chaîne $person passée en paramètre :

```php
<?php
$ds=ldap_connect($server); // On suppose que le serveur LDAP est sur cet hote
if ($ds) {
$r=ldap_bind($ds,$rootdn,$rootpw);

$dn = "o=commentcamarche, c=fr";

$filtre="(|(sn=$person*)(cn=$person*))";

$restriction = array( "cn", "sn", "mail");

$sr=ldap_search($ds, $dn, $filtre, $restriction);

$info = ldap_get_entries($ds, $sr);

print $info["count"]." enregistrements trouves

";

ldap_close($ds);

} else {
echo "Connexion au serveur LDAP impossible";

}

?>
```

Toutefois, une fois l'opération de recherche effectuée, il s'agit d'exploiter les résultats obtenus. Ainsi, la majeure partie des fonctions LDAP ont pour but le traitement des résultats de la recherche.

Dans l'exemple ci-dessus, la fonction `ldap_get_entries()` permet de récupérer des informations sur les entrées retournées par la fonction `ldap_search()`.

### Traitement des résultats

De nombreuses fonctions LDAP permettent d'exploiter les résultats renvoyés
par la fonction ldap_search(). Ces fonctions ont un nom commençant généralement par

`ldap_get_` suivi du nom de l'élément à récupérer :

*    `ldap_get_dn()` permet de récupérer le DN de l'entrée
*    `ldap_get_entries()` permet de récupérer l'ensemble des entrées
*    `ldap_get_option()` permet de récupérer la valeur d'une option
*    `ldap_get_values()` permet de récupérer toutes les valeurs d'une entrée
*    `ldap_get_values_len()` permet de récupérer les valeurs binaires d'une entrée
*    `ldap_count_entries()` permet de récupérer le nombre d'entrées retournées par la fonction de recherche


### Récupérer le nombre d'entrées retournées

La fonction `ldap_count_entries()` permet de connaître le nombre d'entrées retournées par la fonction `ldap_search()`. Sa syntaxe est la suivante :

    int ldap_count_entries (int link_identifier, int result_identifier)

La fonction `ldap_count_entries()` admet en paramètre l'identifiant du serveur LDAP retourné par la fonction `ldap_connect()` ainsi que l'identifiant du résultat retourné par la fonction `ldap_search()` et retourne un entier représentant le nombre d'entrées stockées dans le résultat de la recherche.

Voici un exemple d'utilisation :

```php
<?php
$ds=ldap_connect($server); // On suppose que le serveur LDAP est sur cet hote
if ($ds) {
$r=ldap_bind($ds,$rootdn,$rootpw);

$dn = "o=commentcamarche, c=fr";

$filtre="(|(sn=$person*)(cn=$person*))";

$restriction = array( "cn", "sn", "mail");

$sr=ldap_search($ds, $dn, $filtre, $restriction);

$nombre = ldap_count_entries($ds, $sr);

print $nombre." enregistrements trouves

";

ldap_close($ds);

} else {
echo "Connexion au serveur LDAP impossible";

}

?>
```

### Récupérer les entrées retournées

La fonction `ldap_get_entries()` permet de récupérer l'ensemble des entrées retournées par la fonction `ldap_search()` ainsi que de lire les attributs associés et leur(s) valeur(s).

Sa syntaxe est la suivante :

    array ldap_get_entries (int link_identifier, int result_identifier)

La fonction `ldap_get_entries()` admet en paramètre l'identifiant du serveur LDAP retourné par la fonction `ldap_connect()` ainsi que l'identifiant du résultat retourné par la fonction `ldap_search()` et retourne un tableau multidimensionnel contenant le résultat de la recherche, c'est-à-dire le DN et le nom de l'entrée ainsi que la ou les valeurs de chacune d'entre-elles.

La structure du tableau retourné (on supposera qu'il se trouve dans la variable $entrees) est la suivante :

*    `$entrees["count"]` : nombre d'entrées dans le résultat
*    `$entrees[0]` : détail de la première entrée
*    `$entrees[i]["dn"]` : DN de la ième entrée
*    `$entrees[i]["count"]` : nombre d'attributs de la ième entrée
*    `$entrees[i][j]` : valeur du jème attribut de la ième entrée
*    `$entrees[i]["attribut"]` : valeur de l'attribut nommé "attribut" de la ième entrée (pour un attribut multivalué)
*    `$entrees[i]["attribut"]["count"]` : Nombre de valeurs du jème attribut de la ième entrée (pour un attribut multivalué)
*    `$entrees[i]["attribut"][j]` : Jème valeur de l'attribut nommé "attribut" de la ième entrée (pour un attribut multivalué)

>Les noms des attributs dans le tableau associatif sont en minuscules, ainsi l'attribut givenName devra être écrit givenname (par exemple `$entrees[i]["givenname"]`)

Voici un exemple d'utilisation :

```php
<?php
$ds=ldap_connect($server); // On suppose que le serveur LDAP est sur cet hote
if ($ds) {
$r=ldap_bind($ds,$rootdn,$rootpw);

$dn = "o=commentcamarche, c=fr";

$filtre="(|(sn=$person*)(cn=$person*))";

$restriction = array( "cn", "sn", "mail");

$sr = ldap_search($ds, $dn, $filter);

echo "Le nombre d'entrées retourné est de ".ldap_count_entries($ds,$sr)."<br>";

echo "Récupération des entrées ...";

$info = ldap_get_entries($ds, $sr);

echo "Affichage des données des ".$info["count"]. " entrées trouvées :";

for ($i=0; $i<$info["count"]; $i++)
{
echo "<p align="justify">";

echo "Le dn (Distinguished Name) est: ". $info[$i]["dn"] ."<br>";

echo "Nom (sn) : ". $info[$i]["sn"][0] . "<br>";

echo "Prénom (cn) : ". $info[$i]["cn"][0] . "<br>";

for($j=0;$j<$info[$i]["mail"]["count"];$j++) {
echo "Email numéro $j: ". $info[$i][ "mail"][$j] ."<br>";

}
}
echo "<p> ... Fermeture de la connexion";

ldap_close($ds);

} else {
echo "Connexion au serveur LDAP impossible";

}

?>
```

>Notez que pour récupérer uniquement le nombre de résultats total, il est préférable d'utiliser la fonction `ldap_count_entries()` que de passer par la fonction `ldap_get_entries()` puis `$entrees["count"]`



## PHP - Administration d'un annuaire LDAP

### Introduction à LDAP

PHP permet la connexion et l'envoi de requêtes sur un annuaire LDAP, c'est-à-dire un serveur permettant de stocker des informations de manière hiérarchique.

Dans le cadre de cet article, on se propose d'écrire un <u>interface d'administration</u>
pour un annuaire. On va prévoir les actions de base, c'est à dire :  
**ajouter, modifier et supprimer des éléments de notre annuaire LDAP**.  
Naturellement chacun à une action précise. Il y aura donc une page principale qui affichera les personnes contenues dans l'annuaire et qui proposera les différentes actions associées aux objets.

Dans chaque module, on va avoir besoin d'un certain nombre de variables qui caractérisent le serveur LDAP. On va donc créer un fichier de configuration externe qui sera chargé au démarrage de chaque module. De cette façon, lorsque vous voulez installer votre application sur une autre plate-forme, vous ne devez modifier que ce fichier de configuration.

De plus, on va prévoir la possibilité de modifier le modèle d'affichage,
c'est à dire l'apparence de votre interface. On utilisera donc deux fichiers :

*    header.php : qui définira le haut de page de notre interface
*    footer.php : qui définira le bas de page de notre interface

### Sécurisation de l'interface

On supposera que l'ensemble des pages PHP créées dans ce tutorial se trouvent
sous le répertoires **ldap_admin/**

Nous allons donc protéger le répertoire **ldap_admin/** qui contiendra les pages d'administration de notre annuaire. Chaque utilisateur qui voudra accéder à ces pages devra s'authentifier. Nous utilisons la méthode des fichiers **.htaccess**

On va ainsi définir que seul l'utilisateur *ldap_admin* peut accéder à l'interface d'administration du forum. On va dans un premier temps créer le fichier des utilisateurs.  

Placez-vous dans le répertoire ldap_admin et exécuter la commande suivante pour créer ce fichier (pour les utilisateurs de Linux) :

    htpasswd -c ldap_admin.passwd ldap_admin

>Remarque : le nom de ce fichier n'a aucune importance. L'option -c correspond à la création du fichier.

On va alors avoir à rentrer un mot de passe pour l'utilisateur: ldap_admin autorisé à accéder au répertoire protégé.

    New password:

On confirme.

    Re-type new password:

Une fois que le fichier des utilisateurs est généré, on va créer le fichier **.htaccess** qui sera stocké dans le répertoire à protéger, ici **ldap_admin/**

On protège aussi le fichier de configuration :

```
<Files config_LDAP.inc.php>

Order Deny,Allow
Deny From All
</Files>

AuthUserFile /home/httpd/html/services/ldap_admin/ldap_admin.passwd
AuthName "Acces Restreint"
AuthType Basic
<Limit GET POST>

require valid-user
</Limit>
```

La sécurisation de notre interface est maintenant terminée.

### Fichier de configuration

Commençons par commenter le fichier de configuration nommé **config_LDAP.inc.php** :

```php
<?
// Fichier de configuration pour l'interface PHP pour administrer
// notre annuaire LDAP
$server = "localhost";

$port = "389";

$racine = "o=commentcamarche, c=fr";

$rootdn = "cn=ldap_admin, o=commentcamarche, c=fr";

$rootpw = "secret";

?>
```

On définit donc cinq variables pour caractériser le serveur LDAP :

1. le nom du serveur
2. le port (389 par défaut)
3. la racine supérieure de l'arborescence 
4. la chaîne de connexion pour l'administrateur 
5. ainsi que son mot de passe.

Le fichier de configuration sera automatiquement appelé par le fichier **header.php**
qui définit le haut de notre interface.

### L'interface PHP/LDAP

Détaillons maintenant le fichier **admin.php** qui est la page principale de notre interface.

Cette page liste les personnes saisies dans l'annuaire et propose soit de les modifier soit de les supprimer. Elle propose aussi un lien en bas de page pour ajouter une nouvelle
personne.

Du point du vue du code source, elle ne comporte aucune grosse complexité.

*    En début de programme, le fichier **header.php**, qui décrit le haut de page de l'interface, est chargé. Ce même fichier charge automatiquement le fichier de configuration **config_LDAP.inc.php**.
*    Le fichier **footer.php**, utilisé pour le bas de page est chargé à la fin.


La connexion au serveur LDAP est réalisée grâce à la fonction `ldap_connect`.

Ensuite, la fonction `ldap_search` permet de rechercher tous les objets de type *person* et de les afficher avec une boucle de type « for » sous la forme d'un tableau en ajoutant deux liens qui correspondent respectivement à la modification et à la suppression.

Pour la modification, la page **modifie.php** est appelée. On lui passe en paramètre la valeur de *cn* contenue dans l'annuaire, qui correspond au nom concaténé au prénom. 

La même chose est réalisée pour le lien concernant la suppression sauf que la page **supprime.php** est appelée.

Le paramètre *cn* est encodé avec la fonction `urlencode()` de façon à transformer cette chaîne de caractère en une chaîne compatible avec le format des URL. Ainsi les espaces seront par exemple remplacés par des « + ».

    admin.php

```php
<?
// affichage du haut de la page contenu dans le fichier header.php
require("header.php");

echo "Les personnes suivantes sont inscrites dans l'annuaire :<p>";

// connexion au serveur LDAP : ds est égal à 1 si la connexion est OK
$ds=ldap_connect($server);

if ($ds==1)
{
// on recherche les objet de type person à partir de la racine
// de notre serveur LDAP, ici : o=commentcamarche, c=fr
$sr=ldap_search($ds, $racine, "objectclass=person");

$info = ldap_get_entries($ds, $sr);

echo "<table border=1>";

echo "<tr>
<th>Nom et prénom</th>
<th>Adresse e-Mail </th>
<th>Téléphone</th>
</tr>";

// on affiche sous forme d'un tableau les personnes enregistrées
// dans l'annuaire avec un lien pour modifier et un lien pour supprimer
for ($i=0;$i<$info["count"];$i++)
{
$mynom = $info[$i]["cn"][0];

$myemail = $info[$i]["mail"][0];

$mytel = $info[$i]["telephonenumber"][0];

echo" <tr><th>$mynom</th><th>

<A HREF=mailto:$myemail>$myemail</a></th><th>$mytel</th>";

$mynom=urlencode($mynom);

echo" <th><a href=\"modifie.php?cn=$mynom\">

Modifier</a></th>";

echo" <th><a href=\"supprime.php?cn=$mynom\">

Supprimer</a></th></tr>";

}
echo"</table>";

echo "<center>< br><a href=\"ajoute.php\">Ajouter une
nouvelle personne dans l'annuaire</a></center>";

}
// on ferme la connexion au serveur LDAP
ldap_close($ds);

// on affiche le bas de page défini dans le fichier footer.php
require("footer.php");

?>
```

### Ajout d'une entrée à LDAP

Continuons avec la page **ajoute.php** qui est utilisée pour ajouter de nouvelles personnes dans notre annuaire : Sur la copie d'écran, vous pouvez constater sur le haut et bas de page est exactement le même que sur notre page principale (dû à l'utilisation des fichiers header.php et footer.php). La page est constituée d'un formulaire avec des champs obligatoires. Vous retrouvez les boutons standards pour valider ou annuler votre nouvelle saisie. Pour essayer de garder toute la fonction « ajouter » dans la même page
(formulaire et enregistrement), on introduit une variable go qui détermine si la page est appellée pour la première fois ou si elle est rappellée après la validation du formulaire.

Cette variable est en fait un champ caché du formulaire (`<INPUT type="hidden" name="go" value="1">`).

Si la variable go est égale à 1 et que les champs *nom, prénom et mail* ne sont pas vides, on peut alors enregistrer la nouvelle personne dans l'annuaire.

Pour ce faire, on se connecte au serveur et on s'authentifie avec le super-utilisateur,
ici ldap_admin (fonction `ldap_bind()`).

Ensuite on doit préparer nos informations avant de les inscrire dans l'annuaire : on construit le champ cn en concaténant le nom et le prénom. On précise bien que c'est un objet de type *person* que l'on veut ajouter et on lance l'enregistrement avec la fonction `ldap_add()`.

Sinon, on affiche le formulaire de saisie avec un message d'erreur si l'on a validé le formulaire sans avoir rempli les champs obligatoires.

    ajoute.php

```php
<?
// on affiche le haut de la page contenu dans le fichier header.php
require("header.php");

if (($go==1) and ($nom!="") and ($prenom!="") and ($mail!=""))
{
$ds=ldap_connect($server);

if ($ds==1)
{
// on s'authentifie en tant que super-utilisateur, ici, ldap_admin
$r=ldap_bind($ds,$rootdn,$rootpw);

// préparation des données
$info["cn"]=$nom." ".$prenom;

$info["mail"]=$mail;

$info["telephonenumber"]=$tel;

$info["objectclass"]="person";

// ajout dans l'annuaire
$r=ldap_add($ds,"cn=$nom $prenom,$racine",$info);

// fermeture de la connexion
ldap_close($ds);

$go==0;

$nom=="";

$prenom="";

$mail="";

$tel="";

echo "L'enregistrement a réussi !!!\n";

echo "<P><A HREF=\"ajoute.php\">Ajouter
une nouvelle personne</A>\n";

echo "<P><A HREF=\"admin.php\">Retourner
à la page d'administration</A>\n";

}
} else {
if ($go==1)
{
$mes="ERREUR ! Vous devez obligatoirement remplir les champs en gras";

echo "<FONT COLOR=FF0000>$mes</FONT>\n";

}
echo "<FORM ACTION=\"ajoute.php\" METHOD=POST>\n";

echo "<TABLE BORDER=0>\n";

echo quot;<TR><TD> <B>Nom</B></TD>\n";

echo "<TD><INPUT TYPE=\"text\" NAME=\"nom\"
value=\"$nom\" SIZE=30 maxlength=80><BR></TD></TR>\n";

echo "<TR><TD> <B>Prénom</B></TD>\n";

echo "<TD><INPUT TYPE=\"text\" NAME=\"prenom\"
value=\"$prenom\" SIZE=30 maxlength=80><BR></TD></TR>\n";

echo "<TR><TD> <B>E-Mail</B></TD>\n";

echo "<TD><INPUT TYPE=\"text\" NAME=\"mail\"
value=\"$mail\" SIZE=40 maxlength=80><BR></TD></TR>\n";

echo "<TR><TD> Téléphone</TD>\n";

echo "<TD><INPUT TYPE=\"text\" NAME=\"tel\"
value=\"$tel\" SIZE=40 maxlength=255><BR></TD></TR>\n";

echo "</TABLE>\n";

echo "<INPUT type=\"hidden\" name=\"go\" value=\"1\"><BR><BR>\n";

echo "<INPUT type=\"submit\" value=\"Valider\">\n";

echo "<INPUT type=\"reset\" value=\"Annuler\">\n";

echo "</FORM>\n";

echo "<BR>Les champs en <B>gras</B> sont obligatoires.\n";

}
// on affiche le bas de la page contenu dans le fichier footer.php
require("footer.php");

?>
```

### Modification d'une entrée de l'annuaire LDAP

Regardons maintenant le source (plus long mais pas plus complexe !) de la page **modifie.php**.
Le nom et le prénom étant concaténé dans le champ cn de l'annuaire, <u>il sera impossible de faire une modification sur le nom ou sur le prénom</u>.

Pour modifier une personne, il faut d'abord la supprimer (avec `ldap_delete()`) puis la réenregistrer avec les nouvelles valeurs.

Le champ *mail* est bien sûr toujours <u>obligatoire</u>. On utilise le même principe pour centraliser sur la même page le formulaire de modification et le l'enregistrement des modifications (variable *go*).

    modifie.php

```php
<?
// on affiche le haut de la page contenu dans le fichier header.php
require("header.php");

$cn=urldecode($cn);

if (($go==1) and ($mail!=""))
{
// connexion au serveur
$ds=ldap_connect($server);

if ($ds==1)
{
// on s'authentifie en tant que super-utilisateur, ici, ldap_admin
$r=ldap_bind($ds,$rootdn,$rootpw);

// Suppression de l'ancien enregistrement
$r=ldap_delete($ds,"cn=$cn,$racine");

// Préparation des données
$info["cn"]=$cn;

$info["mail"]=$mail;

$info["telephonenumber"]=$tel;

$info["objectclass"]="person";

// Ajout dans l'annuaire
$r=ldap_add($ds,"cn=$cn,$racine",$info);

// fermeture de la connexion à l'annuaire LDAP
ldap_close($ds);

$go==0;

$nom=="";

$prenom="";

$mail="";

$tel="";

echo "La modification a réussi !!!\n";

echo "<P><A HREF=\"admin.php\">Retourner
à la page d'administration</A>\n";

}
} else {
if ($go==1)
{
$mes="ERREUR ! Vous devez obligatoirement remplir le champ mail";

echo "<FONT COLOR=FF0000>$mes</FONT>\n";

}
// connexion au serveur
$ds=ldap_connect($server);

if ($ds)
{
$recherche="cn=$cn";

$champs = array("cn", "telephonenumber", "mail");

// recherche les informations de la personne que l'on veut modifier
$sr=ldap_search($ds, $racine, $recherche, $champs);

$num= ldap_get_entries($ds,$sr);

if ($num["count"]>0)
{
$mynom = $num[0]["cn"][0];

$myemail = $num[0]["mail"][0];

$mytel = $num[0]["telephonenumber"][0];

echo "<FORM ACTION=\"modifie.php\" METHOD=POST>\n";

echo "<TABLE BORDER=0>\n";

echo "<TR><TD> <B>Modification de l'utilisateur : $cn</B></TD>\n";

echo "<TR><TD> <B>E-Mail</B></TD>\n";

echo "<TD><INPUT TYPE=\"text\" NAME=\"mail\" value=\"$myemail\"
SIZE=40 maxlength=80><BR></TD></TR>\n";

echo "<TR><TD> Téléphone</TD>\n";

echo "<TD><INPUT TYPE=\"text\" NAME=\"tel\" value=\"$mytel\"
SIZE=40 maxlength=255><BR></TD></TR>\n";

echo "</TABLE>\n";

echo "<INPUT type=\"hidden\" name=\"cn\" value=\"$cn\"><BR><BR>\n";

echo "<INPUT type=\"hidden\" name=\"go\" value=\"1\"><BR><BR>\n";

echo "<INPUT type=\"submit\" value=\"Modifier\">\n";

echo "<INPUT type=\"reset\" value=\"Annuler\">\n";

echo "</FORM>\n";

echo "<BR>Le champ <B>mail</B> est obligatoire.\n";

} else {
echo "Erreur ! La recherche n'a pas aboutie";

}
} else {
echo "Erreur ! Problème à la connexion avec le serveur LDAP";

}
}
require("footer.php");

?>
```

### Suppression d'une entrée de l'annuaire LDAP

On termine enfin avec la page **supprime.php** qui, avant de supprimer, va demander
une confirmation.

Si on confirme, on rappelle la page **supprime.php** avec le paramètre *go=1*

Ici, on est obligé de préciser *go=1* dans l'URL car seules les variables contenues dans les formulaires sont automatiquement passées d'une page à l'autre.

La suppression ne peut s'effectuer qu'avec le super-utilisateur `ldap_admin` (fonction `ldap_bind()`).

    supprime.php

```php
<?
// on affiche le haut de la page contenu dans le fichier header.php
require("header.php");

$cn=urldecode($cn);

if ($go==0) {
echo "Etes-vous sur de vouloir supprimer l'utilisateur $cn<br>\n";

$cn=urlencode($cn);

echo "<A HREF=\"supprime.php?go=1&cn=$cn\">oui</A><BR>\n";

echo "<A HREF=\"admin.php\">non</A><BR>\n";

}
else {
$cn=urldecode($cn);

// connexion au serveur LDAP
$ds=ldap_connect($server);

if ($ds==1) {
// on s'authentifie en tant que super-utilisateur, ici, ldap_admin
$r=ldap_bind($ds,$rootdn,$rootpw);

// Suppression de l'ancien enregistrement
$r=ldap_delete($ds,"cn=$cn,$racine");

echo "La suppression a réussi !!!\n";

echo "<P><A HREF=\"admin.php\">Retourner
à la page d'administration</A>\n";

}
}

// on affiche le bas de la page contenu dans le fichier footer.php
require("footer.php");

?>
```

### Modification d'une entrée de l'annuaire LDAP

Vous voilà prêt à administrer votre annuaire... ou à construire votre propre interface d'administration. Pour un souci de simplicité, seules les fonctionnalités de base ont été implémentées.

Une page pour rechercher une personne pourrait aussi être écrite.

La présentation de l'interface est sommaire mais assez pratique : toutefois, pour la page **admin.php**, il serait envisageable d'ajouter un alphabet qui lorsque l'on clique sur une lettre, déclenche l'affichage des personnes dont le nom commence par la lettre sélectionnée.

De même, vous pouvez modifier aussi la structure de l'annuaire.
Par exemple, vous pouvez ajouter des propriétés à votre objet *person* : un champ *photo* qui pourrait contenir le chemin d'accès complet à une photo d'identité stockée sur votre serveur...

Les sources sont commentés de façon à vous aider à concevoir de nouveaux programmes en PHP. 