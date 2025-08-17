+++
title = 'Introduction à crypttab avec des exemples'
date = 2023-08-11 00:00:00 +0100
categories = chiffrement
+++
*Dans un système d'exploitation basé sur Linux, le fichier crypttab ( /etc/crypttab) est utilisé pour stocker des informations statiques sur les périphériques de bloc chiffrés qui sont destinés à être configurés et déverrouillés au démarrage.*

- [crypttab](#crypttab)
    - [Configuration logicielle requise et conventions utilisées](#configuration-logicielle-requise-et-conventions-utilisées)
    - [Comment les données sont organisées dans le fichier crypttab](#comment-les-données-sont-organisées-dans-le-fichier-crypttab)
        - [colonne 1 : le nom du mappeur de périphérique](#colonne-1-le-nom-du-mappeur-de-périphérique)
        - [colonne 2 : le périphérique de bloc chiffré](#colonne-2-le-périphérique-de-bloc-chiffré)
        - [colonne 3 : chemin absolu vers la clé de chiffrement](#colonne-3-chemin-absolu-vers-la-clé-de-chiffrement)
        - [colonne 4 : les options de chiffrement](#colonne-4-les-options-de-chiffrement)

## crypttab

*Traduction article original [Introduction to crypttab with examples](https://linuxconfig.org/introduction-to-crypttab-with-examples)  28 December 2021 by Egidio Docile* 


Dans ce didacticiel, nous apprenons comment le fichier crypttab est structuré et comment organiser les données qu'il contient.

*À quoi sert le fichier crypttab*  
*Comment les données sont organisées dans le fichier crypttab*

### Configuration logicielle requise et conventions utilisées

Catégorie |	Exigences, conventions ou version du logiciel utilisée
 ---------| ---------------------------
Système |	Indépendant de la distribution
Logiciel |	Aucun logiciel spécifique nécessaire
Autre |	Aucun
Conventions | 	# - nécessite que les commandes linux données soient exécutées avec les privilèges root soit directement en tant qu'utilisateur root, soit en utilisant la commande sudo  
 | $ - nécessite que les commandes linux données soient exécutées en tant qu'utilisateur normal non privilégié

### Comment les données sont organisées dans le fichier crypttab

Le fichier /etc/crypttab sur les distributions Linux est utilisé pour stocker des informations statiques sur les périphériques de bloc chiffrés qui doivent être déverrouillés et définis lors du démarrage du système.   Chaque ligne du fichier est dédiée à un périphérique bloc et les données qu'il contient sont organisées en colonnes. Il y a quatre colonnes, dans l'ordre :

1.    Le nom du mappeur de périphérique qui doit être utilisé pour le volume
-    La référence de périphérique de bloc chiffré
-    La clé de cryptage qui devrait éventuellement être utilisée pour déverrouiller l'appareil
-    Une liste d'options séparées par des virgules pour le périphérique

Parmi les champs énumérés ci-dessus, seuls **les deux premiers sont obligatoires**.

#### colonne 1 : le nom du mappeur de périphérique

Dans chaque ligne du fichier /etc/crypttab, la première colonne, obligatoire, est utilisée pour stocker le nom du mappeur de périphérique à utiliser pour un périphérique de bloc chiffré.  
Qu'est-ce que c'est exactement ?

Sous Linux, le principal moyen de configurer un périphérique de bloc chiffré consiste à utiliser l'utilitaire **cryptsetup**. Avec lui, nous pouvons utiliser deux méthodes de chiffrement : **plain** et **LUKS**  
La première méthode est plus simple et ne nécessite aucune métadonnée à stocker sur l'appareil  
La seconde est plus riche en fonctionnalités : l'appareil est chiffré à l'aide d'une clé principale et peut être déverrouillé à l'aide de plusieurs mots de passe. Les mots de passe eux-mêmes sont hachés avec un sel qui est stocké sur un en-tête créé (par défaut) sur l'appareil chiffré (il peut également être stocké séparément). Si l'en-tête est endommagé, toutes les données sont perdues.

Lorsque nous déverrouillons un périphérique à l'aide de l'utilitaire **cryptsetup**, nous devons spécifier le nom du mappeur de périphérique à utiliser pour le volume déverrouillé. Le mappeur de périphériques est le système utilisé par Linux pour mapper les périphériques de bloc sur des périphériques virtuels de niveau supérieur. Il est utilisé, par exemple, pour les volumes logiques **LVM** et les groupes de volumes, pour les périphériques **RAID** , et également pour stocker des périphériques de bloc chiffrés, comme dans ce cas. Les volumes de mappeur de périphériques sont représentés dans le répertoire **/dev/mapper** et peuvent être répertoriés simplement en utilisant la commande **ls** comme dans l'exemple ci-dessous :

```
ls /dev/mapper
root_lv
home_lv
[...]
```

Dans la sortie de la commande ci-dessus, nous pouvons voir deux fichiers représentant des volumes logiques.

Supposons que nous voulions déverrouiller un périphérique de bloc chiffré **LUKS** avec **cryptsetup**.  
Dans la situation la plus basique, nous utiliserions la syntaxe suivante :

    $ sudo cryptsetup luksOpen /path/to/encrypted/block/device dm-volume-name

Le **volume name** est exactement ce que nous devons fournir dans la première colonne de chaque ligne du fichier **crypttab**.

#### colonne 2 : le périphérique de bloc chiffré

La deuxième colonne du fichier **crypttab** est utilisée pour référencer le périphérique de bloc chiffré. Une référence peut être faite par **path** , par exemple : /dev/sda1, mais comme le chemin d'un périphérique bloc n'est pas garanti de rester le même à chaque démarrage, la meilleure façon de le référencer est d'utiliser son **UUID** ou Universally Unique identifier .  
Nous pouvons le faire en utilisant la même notation que nous utiliserions dans **/etc/fstab**:

    UUID=2ae2767d-3ec6-4d37-9639-e16f013f1e60

#### colonne 3 : chemin absolu vers la clé de chiffrement

Lorsque vous utilisez LUKS comme méthode de chiffrement de l'appareil, nous pouvons configurer un fichier à utiliser comme clé de l'appareil. Nous avons vu comment procéder dans un précédent tutoriel . Si nous voulons que la clé soit utilisée pour déverrouiller l'appareil au démarrage (notez que cela pourrait représenter un problème de sécurité), nous devons spécifier son chemin absolu dans le troisième champ du fichier crypttab.    Si nous ne voulons pas utiliser un fichier de clé pour ouvrir le périphérique bloc, nous pouvons simplement écrire `"none"` ou `"-"` dans ce champ.

*Que se passe-t-il si le fichier de clé de chiffrement se trouve sur un autre appareil, par exemple une clé USB ?*  
Dans ce cas, nous pouvons ajouter un signe `":"` (deux-points) après le chemin du fichier de clé spécifié, suivi d'un identifiant pour le système de fichiers sur lequel se trouve la clé.  
Encore une fois, la méthode recommandée pour référencer le système de fichiers est par son UUID.  
Par exemple, pour spécifier que le fichier clé est dans le répertoire **/keyfiles** sur le système de fichiers qui a l'UUID **17513654-34ed-4c84-9808-3aedfc22a20e**, nous écrirons :

    /keyfiles:UUID=17513654-34ed-4c84-9808-3aedfc22a20e

Pour que cela fonctionne, bien sûr, le système doit être capable de lire le système de fichiers dans lequel le fichier clé est stocké. Si, pour une raison quelconque, nous utilisons un fichier clé pour déverrouiller le système de fichiers racine (c'est une mauvaise pratique et rend le cryptage inutile, car si quelqu'un obtient l'appareil sur lequel la clé est stockée, il a un accès complet aux données), nous aurions également besoin de régénérer le système initramfs , afin qu'il inclue le fichier crypttab modifié.

<u>Si le fichier de clés spécifié est introuvable</u>, l'utilisateur est invité à entrer manuellement un mot de passe pour déverrouiller le périphérique de bloc chiffré comme solution de secours.

#### colonne 4 : les options de chiffrement

Nous pouvons utiliser la quatrième colonne de chaque ligne crypttab pour spécifier les options de chiffrement qui doivent être utilisées pour déverrouiller le périphérique de bloc chiffré.  
Nous pouvons, par exemple, spécifier le type de chiffrement , le chiffrement , le hachage et la taille  
Ceci est généralement nécessaire lorsque le périphérique de bloc a été chiffré en utilisant **plain dm-crypt** au lieu de LUKS. Étant donné qu'avec ce système, il n'y a pas d'en-tête où les métadonnées de chiffrement sont stockées, les paramètres de chiffrement doivent être fournis à chaque ouverture de l'appareil.

Par exemple, pour ouvrir et utiliser **/dev/sda1** en tant que périphérique de cryptage **plain-dm** à partir de la ligne de commande, et le mapper en tant que **sda1_crypt**, nous écrirons :

```
sudo cryptsetup open \
  --type plain \
  --cipher=aes-xts-plain64 \
  --hash=sha512 \
  --size=512
  /dev/sda1
  sda1_crypt
```

Pour spécifier les mêmes options et valeurs de manière statique dans le fichier **crypttab**, dans la <u>quatrième colonne</u> de la ligne dédiée, nous écrirons :

    plain,cipher=aes-xts-plain64,hash=sha512,size=512

Si nous utilisons **LUKS** , ces informations sont stockées dans l'en-tête des métadonnées, il n'est donc pas nécessaire de les signaler de cette façon.  
Tout ce que nous avons à faire est de nous assurer que le mode luks est utilisé. Nous pouvons le faire en remplaçant "plain" par "luks".

Les autres options pouvant être utilisées dans cette colonne sont :

Option |	fonction
--------|--------
discard | 	Nécessaire pour autoriser les demandes de rejet (TRIM) via le périphérique de bloc chiffré (cela a des implications en matière de sécurité)
header | 	Nécessaire pour spécifier l'emplacement de l'en-tête LUKS s'il est séparé du périphérique de bloc chiffré
noauto |	Si cette option est utilisée, l'appareil n'est pas automatiquement déverrouillé au démarrage
nofail | 	Marque le déverrouillage du périphérique bloc comme non essentiel. Le processus de démarrage n'est pas arrêté si le déverrouillage échoue
readonly |	Définir le périphérique de bloc chiffré en mode lecture seule
tries= |	Prend le nombre de tentatives auxquelles l'utilisateur est invité à fournir le bon mot de passe. La valeur par défaut est 0, ce qui signifie aucune limite.
headless= |	Prend un booléen comme valeur. Si **true**, l'utilisateur n'est jamais invité à entrer un mot de passe de manière interactive

>*Ce n'est pas la liste complète des options qui peuvent être utilisées dans le fichier crypttab. Pour les connaître tous, vous pouvez consulter le manuel de crypttab.*

Nous avons appris quel est le rôle du fichier **/etc/crypttab** dans un système Linux qui est utilisé pour stocker des données statiques sur les périphériques de bloc chiffrés qui doivent être déverrouillés au démarrage. Nous avons également appris comment les informations sont organisées dans le fichier et avons vu certaines des options qui peuvent être spécifiées dans la quatrième colonne de chaque ligne.
{: .prompt-info }

