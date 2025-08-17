+++
title = 'Envoi de message en ligne de commande en utilisant msmtp'
date = 2024-02-28 00:00:00 +0100
categories = ['messagerie']
+++
*Il peut être utile de recevoir des rapports par email pour s'assurer que les mises à jour sont correctement appliquées et pour savoir quand un serveur a été redémarré afin d'appliquer les dernières mises à jour. Pour cela, nous devons configurer au moins un client SMTP. Dans cet article, je montrerai comment configurer msmtp.*

## Msmtp

Les principaux points forts de 'msmtp' sont les suivants :

*    L'envoi d'emails via MUA, typiquement Emacs ou Mutt. Assurez-vous simplement d'indiquer au MUA sur votre machine de ne pas appeler /usr/sbin/sendmail, mais d'appeler le msmtp. 
*    Redirection des courriels vers un SMTP (le serveur facilite l'envoi)
*    Profils - vous pouvez configurer 'msmtp' avec différents SMTP et configurations, ce qui le rend idéal pour les clients mobiles. 
*    Pipelining de commandes
*    Prise en charge du proxy SOCKS et des IDN (noms de domaine internationalisés)

### Installation

Installer le paquet **msmtp** et définir des permissions restrictives sur le fichier /etc/msmtprc :

```shell
sudo apt update
sudo apt install msmtp
touch $HOME/.msmtprc
chmod 600 $HOME/.msmtprc
```

### Configuration

Éditer le fichier `$HOME/.msmtprc` et l'adapter à son serveur de mails 

```
account default
host mx1.exemple.net
port 587
from user@domain.xyz  # client du serveur mx1.exemple.net
user user@domain.xyz  # Identique précédent sinon problème
password mot_passe_user
auto_from off
add_missing_from_header on
auth on
logfile ~/.msmtp.log
tls on
tls_starttls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
```
{: file="$HOME/.msmtprc" }

Définir msmtp comme le programme par défaut pour sendmail :

    sudo ln -fs /usr/bin/msmtp /usr/sbin/sendmail

Pour pouvoir utiliser la commande mail, nous devons installer **mailx**

    sudo apt install bsd-mailx

Configurer l'agent de transport du courrier pour qu'il utilise msmtp

    sudo nano /etc/mail.rc

ajouter ce qui suit en fin de fichier

    set mta=/usr/bin/msmtp

### Shell

Envoi en ligne de commande

```shell
echo "Test d’envoi de message" | mail -s "Depuis serveur $HOSTNAME" desti@destinataire.xyz
```

### PHP

```php
<?php
$lien = 'https://site.mondomain.tld/phplogin/activate.php?email=' . "test@domain.tld" . '&code=' . '4567';
$message = '<p>Please click the following link to activate your account: <a href="' . $lien . '">' . $lien . '</a></p>';

$from    = 'postmaster@mondomain.tld';
$noreply = 'noreply@mondomain.tld';
$subject = 'Account Activation Required';
$headers = 'From: ' . $from . "\r\n" . 'Reply-To: ' . $noreply . "\r\n" . 'X-Mailer: PHP/' . phpversion() . "\r\n" . 'MIME-Version: 1.0' . "\r\n" . 'Content-Type: text/html; charset=UTF-8' . "\r\n";

// Chemin vers fichier texte
$file ="/tmp/message.txt";
// Ouverture en mode écriture
$fileopen=(fopen("$file",'w'));
// Ecriture dans le fichier texte
fwrite($fileopen,"To: yack@cinay.eu"."\n");
//fwrite($fileopen,"From: postmaster@mondomain.tld"."\n");
fwrite($fileopen,$headers);
fwrite($fileopen,"Subject: ".$subject."\n"."\n"."\n");
fwrite($fileopen,$message."\n");
// On ferme le fichier proprement
fclose($fileopen);
// Exécuter bash
exec('/usr/bin/msmtp -t < /tmp/message.txt');
?>

```