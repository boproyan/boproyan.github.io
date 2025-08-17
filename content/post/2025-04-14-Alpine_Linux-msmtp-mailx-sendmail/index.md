+++
title = 'Alpine Linux - Relayer les e-mails vers un compte (msmtp, MailX, Sendmail)'
date = 2025-04-14 08:00:00
categories = ['messagerie']
+++
*Si vous exécutez un programme alpine et que vous avez besoin d'un moyen pour que votre programme vous alerte via un compte de messagerie standard*  

Passer en mode su (`sudo -s`)  

### Modification hostname (Facultatif)

*On veut donner le nom __alpine__ à la machine*

Utiliser la commande echo pour écraser le fichier

```shell
echo "alpine" > /etc/hostname
```

Activez immédiatement le changement en exécutant la commande suivante. En d'autres termes, utilisez le fichier /etc/hostname comme nom d'hôte

```shell
hostname -F /etc/hostname
```

Assurez-vous de mettre à jour le fichier `/etc/hosts` avec une configuration IP static, remplacer `localhost.my.domain` par `alpine`

```
127.0.0.1	alpine localhost localhost.localdomain localhost
::1		localhost localhost.localdomain
```

Vérification: `hostname -f`  
**alpine**

### msmtp

*msmtp est un client SMTP très simple et facile à configurer pour l'envoi de courriels. Son mode de fonctionnement par défaut consiste à transférer les courriels au serveur SMTP que vous aurez indiqué dans sa configuration.*

Installation

```shell
apk add msmtp
```


Configuration

Créer un fichier de configuration global `/etc/msmtprc`

```
# Définir les valeurs par défaut pour les comptes 
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        /var/log/msmtp.log

# Gmail
account        cinay
host           mx1.serveurmail.net
port           587
from           utilisateur@domain.tld
user           utilisateur@domain.tld
password       <Mot de passe utilisateur>

# Set a default account
account default : cinay
aliases        /etc/aliases
```

### alias Sendmail

Par défaut alpine vient avec busebox sendmail, msmtp peut agir comme une alternative sendmail, y compris la syntaxe et l'option, création script local.d pour écraser le lien busebox vers msmtp.

Contenu de `/etc/local.d/msmtp-sendmail.start`

```shell
#!/bin/sh
ln -sf /usr/bin/msmtp /usr/bin/sendmail
ln -sf /usr/bin/msmtp /usr/sbin/sendmail
```

Rendre exécutable

```shell
chmod +x /etc/local.d/msmtp-sendmail.start
```

et l'exécuter la première fois

    /etc/local.d/msmtp-sendmail.start

### Mailx et alias

Installez mailx pour le programme qui utilise le courrier

```shell
apk ajouter mailx
```

Créer un fichier `/etc/aliases` 

Contenu de `/etc/aliases`

```
root: origine@domain.tld
default: origine@domain.tld
```

### Test envoi message 

    echo "Test envoi via msmtp" | mail -s "Alpine Linux ntfy" destinataire@exemple.xyz

