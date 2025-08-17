+++
title = 'Utiliser GPG pour chiffrer-déchiffrer un mot de passe'
date = 2020-03-29 00:00:00 +0100
categories = ['chiffrement']
+++
Pour résoudre le problème des mots de passes stockés en clair, on va installer GPG (Gnu Private Guard) et modifier notre configuration pour cacher les données sensibles.

>**gnugpg** est installé par défaut sur la plupart des distributions linux 

### Générer un jeu clé privée/publique gpg RSA 4096 

il faut se créer une paire clé publique/secrète pour chiffrer

	gpg --full-generate-key

```
pub   rsa4096 2020-03-28 [SC]
      FD5948679B2DE3F56D409F57FCC68962900D4EF9
uid                      xoyize-xyz <xoyize-xyz@cinay.xyz>
sub   rsa4096 2020-03-28 [E]
```

On a notre paire de clés avec identifiant **900D4EF9** (les 8 derniers caractères)  et une adresse de messagerie ** xoyize-xyz@cinay.xyz**,on peut chiffrer et déchiffrer des documents   

>NOTE : on peut utiliser l'identifiant ou l'adresse de messagerie

Créer un fichier de message chiffré avec la méthode suivante (aucune trace dans l'historique)

	gpg --encrypt -o .msmtp-xoyize-xyz.gpg -r xoyize-xyz@cinay.xyz - 

Saisir le mot de passe que l'on veut chiffrer puis tapez <Entrée> et enfin <Ctrl d> au clavier

Vérifier la présence du fichier

	ls .msmtp-xoyize-xyz.gpg

```
.msmtp-xoyize-xyz.gpg
```

Pour déchiffer
Il faut saisir la phrase secrète de xoyize une fois

	gpg --no-tty -q -d .msmtp-xoyize-xyz.gpg

```
               ┌─────────────────────────────────────────────────────────────────────────────────┐
               │ Veuillez entrer la phrase secrète pour déverrouiller la clef secrète OpenPGP :  │
               │ « xoyize-xyz <xoyize-xyz@cinay.xyz> »                                           │
               │ clef RSA de 4096 bits, identifiant 5A79BAD287BFD6FF,                            │
               │ créée le 2020-03-28 (identifiant de clef principale FCC68962900D4EF9).          │
               │                                                                                 │
               │                                                                                 │
               │ Phrase secrète : *****************************_________________________________ │
               │                                                                                 │
               │            <OK>                                               <Annuler>         │
               └─────────────────────────────────────────────────────────────────────────────────┘
```	

**Le contenu déchiffré s'affiche**

Si on relance une seconde fois

	gpg --no-tty -q -d .msmtp-cinay-xyz.gpg

**Le contenu déchiffré s'affiche**


### Configuration

GnuPG requiert une quantité assez minime de configuration. Par contre, par défaut, il va vous demander votre phrase de passe dès qu'il essaie d'accéder à vos clés : en pratique, dès que vous envoyez un mail signé, que vous recevez un mail chiffré, ou qu'il tente de déchiffrer un fichier - donc, tout le temps.

Pour que ça ne soit pas trop pénible, on utilise **gpg-agent**, qui nécessite un peu plus de configuration. 

Commencez par ajouter la ligne contenant `use-agent` dans le fichier **~/.gnupg/gpg.conf** (si ce fichier n'existe pas, il se créera).

    echo "use-agent" >> ~/.gnupg/gpg.conf

Puis, création du fichier **~/.gnupg/gpg-agent.conf** 

```
cat > ~/.gnupg/gpg-agent.conf << FINFICHIER
pinentry-program /usr/bin/pinentry-curses
default-cache-ttl 43200
max-cache-ttl 43200
FINFICHIER
```

en remplaçant 43200 par le temps, en secondes, pendant lequel vous voulez que gpg-agent se souvienne de votre phrase de passe (donc la fréquence à laquelle il va vous la redemander). Ici, toutes les 12 heures. 

Redémarrer gpg-agent

    gpgconf --reload gpg-agent

Pour vérifiez que ça fonctionne, lancez la commande suivante dans un terminal :

    eval "$(gpg-agent --daemon)"

```
gpg-agent: a gpg-agent is already running - not starting a new one
```

et lancez une commande qui nécessite votre phrase de passe (par exemple, chiffrez un fichier quelconque); vous allez pouvoir constater que c'est gpg-agent qui s'occupe de vous la demander et que si vous faites ça plusieurs fois de suite, il l'aura gardé en mémoire entre deux fois consécutives.

## Liens

### [Chiffrement et signature avec PGP](https://desfontain.es/blog/client-mail-4-gpg.html)