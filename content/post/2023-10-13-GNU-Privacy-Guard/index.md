+++
title = 'GNU-Privacy-Guard  gpg'
date = 2023-10-13 00:00:00 +0100
categories = chiffrement
+++
*GnuPG (ou GPG, de l'anglais GNU Privacy Guard) est l'implémentation GNU du standard OpenPGP défini dans la RFC 48805, distribuée selon les termes de la licence publique générale GNU (<https://fr.wikipedia.org/wiki/GNU_Privacy_Guard>)*  

Ce logiciel permet la transmission de messages électroniques signés et chiffrés, garantissant ainsi leurs authenticité, intégrité et confidentialité. 


# GNU Privacy Guard 

![](Gnupg_logo.png)

GPG est l'acronyme de *GNU Privacy Guard*. Il permet le chiffrement et la signature de données.    
Vous trouverez dans le présent document la méthode de création de clés GPG en lignes de commande. Pour créer des clés avec une interface graphique reportez-vous à la page [[:Seahorse]].  
  
L'application **GnuPG** sert à chiffrer des données : vous pouvez vous en servir pour communiquer en toute sécurité (courriel, messagerie instantanée, etc.) et pour chiffrer vos fichiers (qui pourront d'ailleurs être également déchiffrés sous d'autres systèmes d'exploitation comme Windows).  
  
  
## Introduction 
  
  
Comment fonctionne le chiffrement ?  
  
Il serait dommage d'utiliser GPG sans connaître la différence entre une clef publique et une clef privée (*public key/private key*). Par conséquent, je vous invite à visiter la page [[Cryptographie asymétrique]](http://fr.wikipedia.org/wiki/Cryptographie_asym%C3%A9trique) sur Wikipédia.  
À noter qu'on peut également chiffrer en symétrique avec GPG (option « -c »).  
  
**Remarque** : Dans cette documentation le terme de *clef* utilisé seul peut désigner la clef publique ou la clef privée. Cet abus de langage couramment utilisé ne pose pas de problème quand le concept de clef publique-clef privée est bien compris.  
  
Ci-dessous une autre formulation des principaux concepts :  
  
### Clef publique et clef privée 
  
La **clef publique** sert à chiffrer un message qui pourra être déchiffré uniquement à l'aide de la **clef privée**. Pour que cela fonctionne, il faut fournir à vos correspondants votre clef publique. Ceux-ci chiffreront leur message avec cette clef, et vous seul pourrez le déchiffrer à l'aide de votre clef privée.  
  
La **clef privée** ne doit jamais être divulguée. Pour plus de sécurité, cette clé est chiffrée en symétrique et un mot de passe y est donc associé : ainsi même lorsque quelqu'un a accès à votre clef privée (en accédant à votre ordinateur par exemple), il ne peut utiliser votre clé privée pour déchiffrer vos données.  
  
Une clef publique et une clef privée fonctionnent en inverse. Ce que chiffre l'une, l'autre peut le déchiffrer, et vice versa. Chacune fait une partie du travail. Ainsi, vous pouvez **signer** vos messages à l'aide de votre clef privée en envoyant avec votre message une copie conforme de celui-ci, chiffrée avec votre clef privée. Votre clef publique pourra déchiffrer ce message, attestant ainsi de l'identité de son emetteur.  
  
Un *mot de passe* est associé à la clef privée : il vous sera demandé pour signer ou déchiffrer les fichiers, courriels ou messages. Le chiffrage ne nécessite pas de mot de passe.  
  
Évidemment, ne perdez pas ce mot de passe et ne le divulguez à personne.  
  
### Validité de la clef 
  
Lors de la création de la clef publique, une date d'expiration vous sera demandée : c'est la date à partir de laquelle vous et vos correspondants ne pourrez plus utiliser cette clef pour chiffrer des données.   
  
**Note :** Ceci ne vous empêchera pas de relire des données chiffrées avec cette clef publique. Il est aussi possible de modifier cette date d'expiration ultérieurement.     
Si vous entrez "0", la clé n'expirera jamais (cela diminue la sécurité).  
  
### Certificat de révocation 
  
Le Certificat de révocation sert à invalider une clef. Vos correspondants ne pourront alors plus utiliser cette clef. Il doit être utilisé lorsque quelqu'un est susceptible d'avoir obtenu votre clef privée ou votre mot de passe.  
  
  
### Identifiant 
  
L'identifiant d'une clef est appelé *uid* *(User IDentifiant)* dans GPG. Il est de la forme « Prénom NOM (commentaire) <adresse de courriel> » (ou « Prénom NOM <adresse de courriel> » si aucun commentaire n'a été spécifié).  
  
Une clef est composée de différents champs :   

  - le champ « pub » correspond à la partie publique.  
  - le champ « sub » correspond à une sous-clé.  
  - le champ « uid » correspond à une adresse email et un nom.  
  
La notation « <id> » utilisée dans cette documentation est une chaine de caractères propre à une clé du trousseau. Typiquement, cela peut être le *nom*, ou *un identifiant de la clé* (« pub » ou « uid »), ou encore *l'adresse de courriel*.  
  
  
### Empreinte 
  
Une empreinte (*fingerprint* en anglais) sert à identifier de manière unique une clef.  
L'idée étant que, plusieurs personnes pouvant générer une clef avec un même *uid*, il faut pouvoir déterminer si on a la bonne clef publique en comparant son empreinte.  
  
Si l'empreinte ne correspond pas vous devez supprimer la clef publique invalide de votre trousseau.  
  
**Note :** Les 8 derniers caractères du *fingerprint* correspondent à l'identifiant *pub* de la clé.  
  
### Signer une clef publique 
  
Signer une clef publique, c'est certifier que cette clef est bien celle de la personne indiquée par l'identifiant. C'est pour cela qu'il faut faire la vérification d'empreinte *avant* de signer une clef.  
  
Après avoir signé une clef publique vous pouvez la renvoyer sur un serveur. Il est parfois apprécié de ne pas publier directement la clé signée, mais de l'envoyer à son propriétaire afin du lui en laisser le choix.    
Votre signature apporte à tous votre garantie sur l'authenticité de cette clef.  
  
  
## Pré-requis 
  
  * Disposer des droits d'administration.  
  * Disposer d'une connexion à Internet configurée et activée.  
  
## Installation
 
Il est nécessaire d'installer le paquet **gnupg2**.  
Il existe également diverses interfaces graphiques pour GnuPG. Sous Ubuntu **Seahorse** est installée par défaut. Il existe aussi :  

  * **gpa**, tente de devenir l'interface graphique standard de GnuPG,  
  * **kgpg** ou **[[apt>Kleopatra]]**, développées pour l'environnement KDE, mais fonctionne également sous Gnome,  
  * [[Pyrite]](https://github.com/ryran/pyrite).  

Nous favoriserons ici l'usage de gpg en ligne de commande.  
  
**N.B. :** Lorsque vous utilisez GPG pour chiffrer vos messages, c'est transparent pour vous car c'est pris en charge par vos logiciels habituels.  
  
  
  
## Utilisation
  
Cette section illustre les usages les plus communs de GnuPG. Vous pouvez évidement obtenir d'autres détails en utilisant la commande :

```
gpg --help
```
  
  
  
### Création d'une clef 
  
Ouvrez une console et exécutez la commande suivante : 

```
gpg --gen-key
```
 qui vous renvoie:  


```
Sélectionnez le type de clef désiré :  
   (1) RSA et RSA (par défaut)  
   (2) DSA et Elgamal  
   (3) DSA (signature seule)  
   (4) RSA (signature seule)  
Quel est votre choix ?   

```
  
Choisissez *RSA et RSA (par défaut)* en tapant *1*.  
On vous proposera une clef de  2048 bits, cela vous assurera une bonne protection, appuyez sur *Entrer*.  
  
<note important>Dans le cas d'une clé RSA, **il est aujourd'hui (2015) vivement recommandé d'utiliser une clé d'une longueur minimale de 4096 bits !**  
  
Voir << [[Les clés primaires devraient être des clés DSA-2 ou RSA, de 4096 bits ou plus (de préférence RSA)]](https://help.riseup.net/fr/security/message-security/openpgp/best-practices#les-cl%C3%A9s-primaires-devraient-%C3%AAtre-des-cl%C3%A9s-dsa-2-ou-rsa-de-4096-bits-ou-plus-de-pr%C3%A9f%C3%A9rence-rs) >> ou encore [[|l'interview de Phil Zimmermann [de]]](http://www.heise.de/ix/artikel/Passivitaet-heisst-sich-abzufinden-1981718.htm) , l'inventeur de PGP, qui, en 2013, recommandait déjà une longueur minimale de 3072 bits pour les clés RSA.</note>  
  
Choisissez alors dans combien de temps votre clef expirera. Vous pouvez rentrer *30* comme nombre de jours pour faire vos premiers essais.  
  
Confirmez par *o*.  
  
Vous allez alors créer un *identifiant* pour votre clef :  
  - Il faut d'abord donner le *Nom réel*, c'est-à-dire vos *Prénom* et *NOM*.  
  - Remplissez ensuite votre *Adresse électronique* (même si vous utilisez cette clef uniquement pour chiffrer vos fichiers). Cette adresse doit correspondre avec celle que vous utilisez, sans artifice anti-spam.  
  - Le *Commentaire* est optionnel.  
Validez par « O ».  
Il faut maintenant fournir une *Phrase de passe* dans la fenêtre qui s'ouvre. Le terminal vous renvoie ceci : 

```
De nombreux octets aléatoires doivent être générés. Vous devriez faire  
autre chose (taper au clavier, déplacer la souris, utiliser les disques)  
pendant la génération de nombres premiers ; cela donne au générateur de  
nombres aléatoires une meilleure chance d'obtenir suffisamment d'entropie.
```
  
Patientez jusqu'à la fin de l'opération.  
  
### Création du Certificat de révocation 
  
Ouvrez un [[:terminal]], et exécutez la commande suivante : 

```
gpg --gen-revoke <votre_courriel>
```
  
le terminal vous renvoie: 

```
Faut-il créer un certificat de révocation pour cette clef ? (o/N)   

```
  
Faites « o ».  
Il ne vous reste plus qu'à choisir la cause de la révocation. Et à répondre aux dernières questions.  
  
### Gérer son trousseau de clefs 
  
Voici quelques commandes pour gérer son trousseau de clefs:  
  * Lister toutes les clefs: 

```
gpg --list-keys
```
  
  * Exporter une clef publique sur un serveur de clef : 

```
gpg --send-key <id> --keyserver <serveur>
```
 Vous pouvez spécifier un serveur de clef spécifique avec l'option « ''--keyserver <serveur>'' », c'est aussi bien pour la recherche, l'importation ou exportation de clef.  
  * Rechercher une clef publique sur un serveur de clef : 

```
gpg --search-keys <identifiant>
```
  
  * Ajouter une clef publique depuis un serveur de clef : 

```
gpg --recv-keys <identifiant>
```
  
  * Ajouter une clef publique contenue dans le presse papier (vous avez reçu la clef publique par courriel par exemple) : 

```
gpg --import
```
 Collez ensuite la clef dans le terminal. et taper Ctrl D  
  * Supprimer une clef publique : 

```
gpg --delete-keys <identifiant>
```
  
  
### Configurer les logiciels pour utiliser GnuPG 
  
  * Courriel :  
    * Thunderbird, *via* Enigmail   
    * Evolution : Evolution Gpg  
    * Kmail : KMail Openpgp  
    * Claws-mail avec le module [[gpg]](http://www.claws-mail.org/plugin.php?plugin=gpg)  
    
  * Jabber :  
      * Gajim  
  
### Récupérer ou partager un trousseau de clé déjà existant 
  
Cela est utile par exemple si votre trousseau de clé a été créé sous Windows. Pour cela il vous suffit de copier tous les fichiers présents dans le dossier du trousseau de clé *keyring* vers le dossier correspondant sous Ubuntu. Vous le trouverez dans le dossier caché **.gnupg**.  
  
Si vous êtes en dual boot avec Windows et que vous utilisez GnuPG dans les deux systèmes, il vaut mieux partager le dossier du trousseau de clefs. Pour cela on crée un lien symbolique à la place du répertoire **.gnupg** avec la commande : 

```
sudo ln -s <chemin du dossier du trousseau de clé/> /home/<utilisateur>/.gnupg
```
  
  
Exemple : 

```
sudo ln -s /media/windowsdata/Mes\ documents/GPGkeys/ /home/toto/.gnupg
```
  
  
## Utilisation Avancée 
  
### Ajouter ou retirer un uid d'une clef 
  
Ceci est généralement utilisé par les personnes disposant de plusieurs adresses de courriels.  
  


```
gpg --edit-key <identifiant>
```
  
  
Gpg vous rend la main en affichant ''gpg>'' après avoir affiché des détails sur la clef à éditer.  
  
Pour ajouter un *<uid>* :  

  - Tapez la commande ''adduid''  
  - Renseignez les infos demandées.  
  
Pour retirer un *<uid>* :  

  - Sélectionnez un *uid* par la commande ''uid <numéro_entre_parenthèses>''. Gpg affiche alors une étoile à côté de l'*uid* sélectionné.  
  - Retirez alors l'*uid* par la commande ''deluid''.  
  
Quittez avec ''quit'' en validant les modifications par ''o''.  
  
### Vérifier l'empreinte 
  


```
gpg --fingerprint <identifiant>
```
  
  
### Signer une clef publique 
  


```
gpg --sign-key <identifiant>
```
  
  
Validez si vous êtes sûr de l'authenticité de la clef. Vous pouvez alors renvoyer la clef sur un serveur de clef (cf. la section « Gérer son trousseau de clefs »).  
  
### Changer la date d'expiration d'une clef 
  


```
gpg --edit-key <identifiant> expire
```
  
  
Vous devez disposer de la clef secrète pour changer cette date.  
  
Vous aurez d'abord à l'écran les informations concernant la clef puis vous pourrez rentrer une durée pour laquelle la clef sera valide en suivant le format indiqué. Validez ensuite par *o*, puis entrez éventuellement votre phrase de passe. Lorsque la ligne ''gpg>'' apparaît, entrez *quit* et validez les changements.  
  
  
### Signer et Chiffrer des fichiers avec GnuPG 
  
  * Signer un fichier : 

```
gpg --clearsign <mon_fichier>
```
 Votre mot de passe vous est demandé pour vérifier que c'est bien vous qui signez le fichier. Un fichier *mon_fichier.asc* est créé : c'est la signature du fichier *mon_fichier*.  
 
  * Chiffrer un fichier : 

```
gpg --encrypt <mon_fichier>
```
 L'identifiant du destinataire est demandé : c'est la personne qui pourra déchiffrer le fichier (qui peut très bien être vous). Un fichier *mon_fichier.gpg* est créé, c'est la copie chiffrée de * mon_fichier*. Cependant le fichier "mon_fichier.gpg" reste illisible par un éditeur de texte, il donne une suite corrompue de caractères (car il est sous forme binaire). Pour avoir un fichier directement lisible (sous forme ASCII) que l'on peut envoyer à un destinataire sous forme de texte et non en pièce jointe il faut utiliser la syntaxe suivante: 

```
gpg --armor --output "mon_fichier_chiffré" --encrypt "mon_fichier"
```
  
  * Déchiffrer un fichier : 

```
gpg --output <mon_fichier> --decrypt <mon_fichier.gpg>
```
 Votre mot de passe vous est demandé pour déchiffrer *mon_fichier.gpg*. Le fichier *mon_fichier* est créé et lisible.  
  
<note tip>Il faut bien comprendre ici que c'est là où intervient le mécanisme de clef publique/ clef privés. Vous chiffrez un fichier avec la clef publique de son destinataire et seule sa clef privée pourra le déchiffrer. D'où l'intérêt de ne pas hésiter à diffuser sa clef publique. Ce principe est bien sûr identique pour le mail.    
Vous pouvez également chiffrer et signer un fichier en même temps, il suffit de mettre les options *--sign* et *--encrypt*.</note>  

### Comment révoquer une clé PGP

noter l’identifiant de la clé publique que l’on souhaite révoquer

    gpg --list-keys

On note l’identifiant (8 caractères hexadécimaux du type A1B2C3D4)  
Ensuite on génère un certificat de révocation qu’on stocke dans un fichier

    gpg --gen-revoke A1B2C3D4 > revoke.txt

importer ce certificat de révocation

    gpg --import revoke.txt

On peut alors uploader la sortie de cette dernière commande sur un serveur de clé type keyserver.ubuntu.com pour informer que votre clé ne doit plus être utilisée.

    gpg --keyserver keyserver.ubuntu.com --send-keys A1B2C3D4

## Liens externes 
  
  * [[Le site officiel de GnuPG [en]]](https://gnupg.org/)   
  * [[Utilisation de GnuPG]](http://www.francoz.net/doc/gpg/)  
  * [[Introduction à GnuPG]](http://matrix.samizdat.net/crypto/gpg_intro/index.html)  
  * [["Mini-Howto" plutôt complet !]](https://gnupg.org/howtos/fr/)  
  * [[Télécharger GnuPG HOWTO par cho7]](http://gpglinux.free.fr/)  
  * [[Chiffrer et signer ses mails ou ses fichiers avec GPG]](https://www.mistra.fr/tutoriel-linux-chiffrer-signer-mails-fichiers-gpg.html)  
  * [[Bonnes pratiques pour l'utilisation d'OpenPGP]](https://help.riseup.net/fr/security/message-security/openpgp/best-practices)  
  * [[Principe du cryptage PGP]](http://laurent.flaum.free.fr/pgpintrofr.htm)  
  

