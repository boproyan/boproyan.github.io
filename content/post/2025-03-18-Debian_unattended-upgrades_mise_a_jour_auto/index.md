+++
title = 'Notification de mises à jour sur Debian'
date = 2025-03-18 00:00:00 +0100
categories = ['debian']
+++
*Un élément central de la sécurité des serveurs Linux consiste à installer les mises à jour de sécurité à temps. C'est l'une des tâches essentielles des administrateurs Linux pour mettre le système à jour. De plus, ça garde votre système plus stable et sécurisé.*

![](update-debian-logo.png){:width="200"}  

* [The easy way to send a notification email when apt updates are available on a Debian or Ubuntu based system, if you don’t have an email server installed](https://twosortoftechguys.wordpress.com/2024/07/12/the-easy-way-to-send-a-notification-email-when-apt-updates-are-available-on-a-debian-or-ubuntu-based-system-if-you-dont-have-an-email-server-installed/)
* [Configuration des mises à jour automatiques sous Debian](https://std.rocks/fr/gnulinux_debian_auto_update.html)

## Notification si des mises à jour disponibles

*Le moyen le plus simple d'envoyer un e-mail de notification lorsque des mises à jour apt sont disponibles sur un système basé sur Debian ou Ubuntu.*

### Script de notification

Vous devez maintenant créer un script qui vérifiera si des mises à jour sont disponibles et vous enverra un e-mail si nécessaire. Ce script peut donc être placé dans votre compte utilisateur habituel et exécuté en tant qu'utilisateur normal.

Prérequis:

* serveur de notification ntfy

Le script `debian_update_notifier.sh`

```shell
#!/bin/bash

# Capture the list of upgraded packages
upgraded_packages=$(apt-get -s upgrade | grep '^Inst' | awk '{print $2}' | tr '[:upper:]' '[:lower:]' | sort | uniq)

# Check if there are upgraded packages
if [[ -n "$upgraded_packages" ]]; then
    # Construct the email from, subject and body formatted for ssmtp
    body="Des mises à jour sont disponibles"
    body+=$'\n'
    body+="Liste des paquets concernés"
    body+=$'\n\n'
    body+="$upgraded_packages"
    body+=$'\n\n'

    # Send the email
	# Message
	curl \
	-H "X-Email: ntfy@cinay.eu" \
	-H "Title: 💻 Debian 12 $HOSTNAME" \
	-H "Authorization: Bearer tk_xxxxxxxxxxxxxxxxxxxxxxxx" \
	-H prio:low \
	-d "$body" \
	https://noti.rnmkcy.eu/yan_infos
    
fi
```

Ne pas oublier le bon token tk_...

Rendre exécutable le sctipt

```shell
chmod +x debian_update_notifier.sh
```

### Tâche pour vérifier les mises à jour apt.

#### Systemd timer

Créer la tâche à exécuter une fois par jour par systemd timer

Fichier service `/etc/systemd/system/aptupdate.service`

```
[Unit] 
Description=Mises à jour Debian

[Service] 
Type=oneshot 
ExecStart=/home/userone/debian_update_notifier.sh
```

Fichier service `/etc/systemd/system/aptupdate.timer`

```
[Unit] 
Description=Exécute chaque jour debian_update_notifier.sh  

[Timer] 
OnUnitActiveSec=24h 
Unit=aptupdate.service 

[Install] 
WantedBy=timers.target 
```

Activation et démarrage

```shell
sudo systemctl daemon-reload
sudo systemctl enable aptupdate.timer --now
```

#### Cron

Création d'une tâche cron pour vérifier les mises à jour une fois par jour, à l'heure de votre choix. 

ouvrir votre éditeur crontab 

```shell
crontab -e
```

Et puis collez cette entrée crontab en fin de fichier

```
38 18 * * * $HOME/debian_update_notifier.sh
```

Remplacez 17 13 par l'heure que vous avez choisie, en spécifiant d'abord les minutes puis les heures et $HOME par /home/user, user votre nom utilisateur

### Confirmer si apt vérifie périodiquement les mises à jour.

Dans la plupart des cas, les systèmes Ubuntu sont configurés pour vérifier régulièrement les mises à jour, mais ce n'est pas toujours le cas sur les systèmes Debian. Dans ce cas, vous ne recevrez une notification que si vous avez récemment effectué un `sudo apt update` . Procédez comme suit :

```shell
ls /etc/apt/apt.conf.d
```

Si vous voyez un fichier nommé « 10periodic », faites simplement

```shell
cat /etc/apt/apt.conf.d/10periodic
```

Et assurez-vous qu'il contient cette ligne et que le numéro à la fin de la ligne est « 1 » :

APT::Periodic::Update-Package-Lists "1";

Si c'est le cas, vous avez terminé. Si vous ne voyez pas ce fichier, ni aucun fichier dont le nom contient « périodique », procédez comme suit :

```shell
sudo nano /etc/apt/apt.conf.d/10periodic
```

Ajouter les ligne suivantes

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "0";
APT::Periodic::AutocleanInterval "0";
```

Un redémarrage du système peut être nécessaire pour que cette action soit prise en compte.

Votre système devrait désormais vérifier les mises à jour d'APT une fois par jour et vous envoyer un e-mail si nécessaire. 

## unattended-upgrades

### Paquet unattended-upgrades

Installer le paquet de mises à niveau sans surveillance sur Debian

Tout d'abord, si le paquet unattended-upgrades n'est pas déjà installé sur votre système, vous pouvez l'installer à l'aide des commandes ci-dessous dans la console :

```shell
sudo apt update
sudo apt install unattended-upgrades
```

### Edition fichier de configuration

Le fichier de configuration pour les mises à niveau sans surveillance se trouve dans le répertoire `/etc/apt/apt.conf.d/`  
Son nom est `50unattended-upgrades`. 

```shell
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Le fichier complet **50unattended-upgrades**

```
Unattended-Upgrade::Origins-Pattern {
        // Les mises à jour recommandées sont des modifications de logiciels
        // mais pas des mises à jour qui affecteront la sécurité de votre système. 
        // Elles tendent à corriger des bogues et des problèmes gênants.
        "origin=Debian,codename=${distro_codename}-updates";
        // Les mises à jour "proposed" sont des mises à jour qui attendent d'être placées dans la file d'attente des mises à jour recommandées après avoir été testées. Il se peut qu'elles ne soient jamais recommandées ou qu'elles soient remplacées par une mise à jour plus récente.
        // Elles sont déconseillées dans un environnement de production
        //"origin=Debian,codename=${distro_codename}-proposed-updates";
        "origin=Debian,codename=${distro_codename},label=Debian";
        "origin=Debian,codename=${distro_codename},label=Debian-Security";
        "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
};

// Envoyer un email à cette adresse pour les problèmes ou les mises à jour de paquets
// Si vide ou non défini, aucun email n'est envoyé, assurez-vous que vous
// d'avoir une configuration de messagerie fonctionnelle sur votre système. Un paquet qui fournit
// 'mailx' doit être installé. Par exemple, « user@example.com »
Unattended-Upgrade::Mail "rnmkcy@cinay.eu";

// Suppression automatique des paquets inutilisés après la mise à jour
// (équivalent à apt-get autoremove)
Unattended-Upgrade::Remove-Unused-Dependencies "true";

// Permettre au système de redémarrer automatiquement si nécessaire
//Unattended-Upgrade::Automatic-Reboot "false";

// Si le redémarrage automatique est activé et nécessaire, le redémarrage s'effectue à l'heure
// spécifiée au lieu de redémarrer immédiatement.
// Paramètre par défaut : "now"
//Unattended-Upgrade::Automatic-Reboot-Time "02:00";
```
{: file="/etc/apt/apt.conf.d/50unattended-upgrades" }

> `Adapater votre adresse de messagerie`
{: .prompt-info }

### Activer les mises à jour automatiques sur Debian

<u>Pour activer les mises à niveau sans surveillance</u>, vous devrez configurer le fichier `/etc/apt/apt.conf.d/20auto-upgrades`  
Exécutez la commande `sudo dpkg-reconfigure --priority=low unattended-upgrades` dans la console 

```shell
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

La fenêtre suivante apparaîtra après avoir exécuté la commande ci-dessus, vous demandant automatiquement si vous souhaitez télécharger et installer automatiquement les mises à jour stables. Utilisez la touche de tabulation pour sélectionner l'option Oui et appuyez sur Entrée.  
![](update-debian-a.png)

Le fichier `/etc/apt/apt.conf.d/20auto-upgrades` sera mis à jour avec le contenu suivant

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```
{: file="/etc/apt/apt.conf.d/20auto-upgrades" }

> Autre possibilité, créer ou éditer le fichier `/etc/apt/apt.conf.d/20auto-upgrades` pour activer **unattended-upgrades**
{: .prompt-info }

```
// Exécuter "apt-get update" automatiquement tous les n-jours (0=désactivé)
APT::Periodic::Update-Package-Lists "1";

// Exécuter le script "unattended-upgrade"
// tous les n-jours (0=désactivé)
// Nécessite le paquet "unattended-upgrades", écrira
// log dans /var/log/unattended-upgrades
APT::Periodic::Unattended-Upgrade "1";
```
{: file="/etc/apt/apt.conf.d/20auto-upgrades" }

Pour voir si le service mises à niveau sans surveillance est activé et en cours d'exécution, vous pouvez exécuter la commande ci-dessous

```shell
sudo systemctl status unattended-upgrades.service
```

![](update-debian-b.png)  
le service **unattended-upgrades** est activé pour installer automatiquement les mises à jour.

Si le service n'est pas autorisé et démarré, vous pouvez le faire manuellement

```shell
sudo systemctl enable unattended-upgrades
sudo systemctl start unattended-upgrades
```

> Toute modification du fichier **/etc/apt/apt.conf.d/20auto-upgrades** doit être suivi d'un redémarrage du service:  `sudo systemctl restart unattended-upgrades`
{: .prompt-warning }

Pour <u>désactiver les mises à jour autiomatiques</u> sur Debian, exécuter `sudo dpkg-reconfigure --priority=low unattended-upgrades`, sélectionner l'option **Non** et appuyez sur Entrée

### Tester la configuration

Lancer `unattended-upgrade` en mode debug pour vérifier que la configuration est OK 

```shell
sudo unattended-upgrade -d --dry-run
```

### Modifier la planification du téléchargement et des installations

Par défaut, le téléchargement des mises à jour s'exécute deux fois par jour, à 6h et à 18h, avec des installations prévues pour 6h. Nous pouvons modifier ces paramètres en éditant deux fichiers systemd.

#### Modification du planificateur de téléchargement

Éditer le fichier `/etc/systemd/system/timers.target.wants/apt-daily.timer` et remplacer la valeur existante par celle souhaitée 

```
[Unit]
Description=Daily apt download activities

[Timer]
OnCalendar=*-*-* 6,18:00
RandomizedDelaySec=12h
Persistent=true

[Install]
WantedBy=timers.target
```
{: file="/etc/systemd/system/timers.target.wants/apt-daily.timer" }

#### Modification du planificateur d'installation

Éditer le fichier `/etc/systemd/system/timers.target.wants/apt-daily-upgrade.timer` et remplacer la valeur existante par celle souhaitée  

```
[Unit]
Description=Daily apt upgrade and clean activities
After=apt-daily.timer

[Timer]
OnCalendar=*-*-* 6:00
RandomizedDelaySec=60m
Persistent=true

[Install]
WantedBy=timers.target
```
{: file="/etc/systemd/system/timers.target.wants/apt-daily.timer" }

#### Prise en compte des modifications

Exécuter ces commandes pour prendre en compte les modifications :

```shell
sudo systemctl daemon-reload
sudo systemctl restart apt-daily-upgrade.timer
sudo systemctl restart apt-daily.timer
```

### Configurer les alertes par mail

*Il peut être utile de recevoir des rapports par email pour s'assurer que les mises à jour sont correctement appliquées et pour savoir quand un serveur a été redémarré afin d'appliquer les dernières mises à jour. Pour cela, nous devons configurer au moins un client SMTP. Dans cet article, je montrerai comment configurer msmtp.*

Installer le paquet **msmtp** et définir des permissions restrictives sur le fichier /etc/msmtprc :

```shell
sudo apt update
sudo apt install msmtp
touch $HOME/.msmtprc
chmod 600 $HOME/.msmtprc
```

Éditer le fichier `$HOME/.msmtprc` et l'adapter à son serveur de mails 

```
account default
host mx1.xoyize.xyz
port 587
from yack@cinay.eu
user yack@cinay.eu
password mot_passe_yack
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

Test

```
echo "Test d’envoi de message" | mail -s "Depuis serveur $HOSTNAME" rnmkcy@cinay.eu
```

**Options**  
Éditer le fichier /etc/apt/apt.conf.d/50unattended-upgrades :

```
// Envoyer un courriel à cette adresse pour signaler des problèmes ou des paquets qui ont été mis à jour
Unattended-Upgrade::Mail "rnmkcy@cinay.eu";
[…]
// Remplacer la valeur "on-change" par "always" et lancer unattended-upgrade -d pour tester que le rapport par courriel fonctionne
Unattended-Upgrade::MailReport "on-change";
// Optionnel : Paramétrer le champ From dans le courriel qui sera envoyé
Unattended-Upgrade::Sender "rnmkcy@cinay.eu";
```

