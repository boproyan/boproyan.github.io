+++
title = 'Archlinux conteneurs - Routage des adresses IPv6 publiques vers vos conteneurs lxc/lxd'
date = 2020-05-01 00:00:00 +0100
categories = ['virtuel']
+++
![lxc](Linux_Containers_logo.png){:width="100"}

## Routage des adresses IPv6 publiques vers vos conteneurs lxc/lxd

L'énorme quantité d'adresses IPv6 disponibles pour la plupart des VPS / serveurs racine hébergés commercialement avec un préfixe IPv6 public vous permet d'acheminer une adresse IPv6 publique vers chaque conteneur qui tourne sur votre serveur. Ce tutoriel vous montre comment le faire, même si vous n'avez aucune expérience préalable du routage,

### Créez votre conteneur LXC

Nous supposons que vous l'avez déjà fait - juste à titre de référence, voici comment vous pouvez créer un conteneur :

    lxc launch ubuntu:18.04 my-container

### Quelle adresse IP voulez-vous attribuer à votre conteneur ?

Vous devez d'abord déterminer quel préfixe est acheminé vers votre hôte. En général, vous pouvez le faire en vérifiant dans le panneau de contrôle de votre fournisseur. Vous cherchez quelque chose comme `2a01:4f9:c010:278::1/64`. Une autre option serait d'exécuter `ip a`

et recherchez une ligne inet6 dans la section de votre interface réseau primaire (cela ne fonctionne que si vous avez configuré votre serveur pour avoir une adresse IPv6).   

>Notez que les adresses qui commencent par `fe80::` et les adresses qui commencent par `fd`, entre autres, ne sont pas des adresses IPv6 publiques.

Vous pouvez alors définir une nouvelle adresse IPv6 pour votre conteneur. Le choix de la nouvelle adresse - pour autant qu'elle soit comprise dans le préfixe - est entièrement laissé à votre discrétion.

Souvent,` <préfixe>::1` est utilisé pour l'hôte lui-même, donc vous pourriez, par exemple, choisir <préfixe>::2. Notez que certains fournisseurs utilisent certaines adresses IP à d'autres fins. Consultez la documentation de votre fournisseur pour plus de détails.

Si vous ne voulez pas faciliter la recherche de l'IPv6 publique de votre conteneur, ne choisissez pas `<prefix>::1, <prefix>::2, <prefix>::3` etc. mais quelque chose de plus aléatoire comme `<prefix>:af15:99b1:0b05:1`, par exemple `2a01:4f9:c010:278:af15:99b1:0b05:0001`  
Assurez-vous que votre adresse IPv6 comporte 8 groupes de 4 chiffres hexadécimaux chacun !

Pour cet exemple, nous choisissons l'adresse IPv6 `2a01:4f9:c010:278::8` 

### Trouvez l'ULA de votre conteneur

Nous devons trouver l'ULA (adresse locale unique - similaire à une adresse IPv4 privée qui n'est pas acheminée sur Internet) du conteneur. Avec lxc, c'est assez facile :

```
uli@myserver:~$ lxc list
+--------------+---------+-----------------------+-----------------------------------------------+
|     NAME     |  STATE  |         IPV4          |                     IPV6                      |
+--------------+---------+-----------------------+-----------------------------------------------+
| my-container | RUNNING | 10.144.118.232 (eth0) | fd42:830b:36dc:3691:216:3eff:fed1:9058 (eth0) |
+--------------+---------+-----------------------+-----------------------------------------------+
```

Vous devez regarder dans la colonne IPv6 et copier l'adresse qui y est indiquée. Dans cet exemple, l'adresse est `fd42:830b:36dc:3691:216:3eff:fed1:9058`

### Configuration du routage IPv6

Nous pouvons maintenant dire à l'hôte Linux de router l'IPv6 public que vous avez choisi vers l'IPv6 privé du conteneur. C'est assez simple :

    sudo ip6tables -t nat -A PREROUTING -d <public IPv6> -j DNAT --to-destination <container private IPv6>

Dans notre exemple, il s'agirait de

    sudo ip6tables -t nat -A PREROUTING -d 2a01:4f9:c010:278::8 -j DNAT --to-destination fd42:830b:36dc:3691:216:3eff:fed1:9058

Tout d'abord, testez la commande en l'exécutant dans un shell. Si elle fonctionne (c'est-à-dire si elle n'affiche aucun message d'erreur), vous pouvez l'enregistrer de façon permanente, par exemple en l'ajoutant à **/etc/rc.local** (après #!/bin/bash, avant la `exit 0`). Les utilisateurs avancés devraient préférer l'ajouter à **/etc/network/interfaces**

### Connectez-vous à votre conteneur en utilisant SSH sur votre IPv6 public (facultatif)

Remarque : cette étape nécessite que votre ordinateur local dispose d'une connectivité IPv6 fonctionnelle. Si vous n'êtes pas sûr, consultez le site [ipv6-test.com](http://ipv6-test.com/)

D'abord, ouvrez un shell sur votre conteneur :

    lxc exec my-container bash

Après l'exécution de ce programme, vous devriez voir une invite du shell racine à l'intérieur de votre conteneur :

    root@my-container:~#

Les commandes suivantes doivent être entrées dans le shell du conteneur, et non dans l'hôte !

Maintenant, nous pouvons créer un utilisateur pour se connecter (dans cet exemple, nous créons l'utilisateur **uli**) :

```bash
root@my-container:~# adduser uli
Adding user `uli' ...
Adding new group `uli' (1001) ...
Adding new user `uli' (1001) with group `uli' ...
Creating home directory `/home/uli' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for uli
Enter the new value, or press ENTER for the default
        Full Name []: 
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [Y/n]
```

Il vous suffit de saisir un mot de passe (vous ne verrez rien à l'écran lorsque vous le saisirez) deux fois, pour toutes les autres lignes, il vous suffit d'appuyer sur la touche "Entrée".

L'image ubuntu:18.04 lxc utilisée dans cet exemple ne permet pas l'authentification par mot de passe SSH dans sa configuration par défaut. Pour remédier à cela, changez `PasswordAuthentication no` en `PasswordAuthentication yes` dans **/etc/ssh/sshd_config** et redémarrez le serveur SSH en lançant le `service sshd restart`. Assurez-vous de bien comprendre les implications en matière de sécurité avant de faire cela !

Maintenant, déconnectez-vous de votre shell de conteneur en appuyant sur `Ctrl+D`. Les commandes suivantes peuvent être entrées sur votre bureau ou sur tout autre serveur disposant d'une connectivité IPv6.

Maintenant, connectez-vous à votre serveur :

    ssh <username>@<public IPv6 address>

dans cet exemple :

    ssh uli@2a01:4f9:c010:278::8

Si vous avez tout configuré correctement, vous verrez l'invite du shell pour votre conteneur :

    uli@my-container:~$

Note : N'oubliez pas de configurer un pare-feu pour votre conteneur, par exemple ufw !  
L'IPv6 de votre conteneur est exposé à l'internet et en supposant que personne ne le devinera, ce n'est pas une bonne pratique de sécurité.