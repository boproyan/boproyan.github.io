+++
title = 'Fail2ban'
date = 2022-09-27 00:00:00 +0100
categories = outils
+++
## Fail2ban

![Fail2ban](fail2ban.png)

*Fail2ban lit des fichiers de log et bannit les adresses IP qui ont obtenu un trop grand nombre d'échecs lors de l'authentification. Il met à jour les règles du pare-feu pour rejeter cette adresse IP. Ces règles peuvent êtres défines par l'utilisateur.*

Les commandes sont exécutées en mode su sudo

Installation

	apt install fail2ban

**fail2ban.conf**

Rien à faire dans le fichier **/etc/fail2ban/fail2ban.conf**, vous pouvez laisser les options par défaut

**jail.conf**

Copier la configuration par défaut afin qu'elle ne soit pas supprimée en cas de mise à jour.  

	cp /etc/fail2ban/jail.conf  /etc/fail2ban/jail.local

Ce fichier est beaucoup plus intéressant, il contient tous les services à monitorer, et vous allez le découvrir, fail2ban ne se limite pas à SSH...  

	nano /etc/fail2ban/jail.local

>Tous les commentaires ont été supprimés pour allèger le fichier.  
Les commentaires sont visibles dans le fichier original **jail.conf**  

```
[INCLUDES]

before = paths-debian.conf

[DEFAULT]

ignoreip = 127.0.0.1/8
bantime  = 600
findtime  = 600
maxretry = 3
backend = auto
usedns = warn
logencoding = auto
filter = %(__name__)s

destemail = root@localhost
sender = root@localhost
mta = sendmail
protocol = tcp
chain = INPUT
port = 0:65535
fail2ban_agent = Fail2Ban/%(fail2ban_version)s

banaction = iptables-multiport
banaction_allports = iptables-allports
protocol="%(protocol)s", chain="%(chain)s"]
action_mw = %(banaction)s[name=%(__name__)s, bantime="%(bantime)s", port="%(port)s", protocol="%(protocol)s", chain="%(chain)s"]
            %(mta)s-whois[name=%(__name__)s, sender="%(sender)s", dest="%(destemail)s", protocol="%(protocol)s", chain="%(chain)s"]

action_mwl = %(banaction)s[name=%(__name__)s, bantime="%(bantime)s", port="%(port)s", protocol="%(protocol)s", chain="%(chain)s"]
             %(mta)s-whois-lines[name=%(__name__)s, sender="%(sender)s", dest="%(destemail)s", logpath=%(logpath)s, chain="%(chain)s"]

action_xarf = %(banaction)s[name=%(__name__)s, bantime="%(bantime)s", port="%(port)s", protocol="%(protocol)s", chain="%(chain)s"]
             xarf-login-attack[service=%(__name__)s, sender="%(sender)s", logpath=%(logpath)s, port="%(port)s"]

action_cf_mwl = cloudflare[cfuser="%(cfemail)s", cftoken="%(cfapikey)s"]
                %(mta)s-whois-lines[name=%(__name__)s, sender="%(sender)s", dest="%(destemail)s", logpath=%(logpath)s, chain="%(chain)s"]

action_blocklist_de  = blocklist_de[email="%(sender)s", service=%(filter)s, apikey="%(blocklist_de_apikey)s", agent="%(fail2ban_agent)s"]

action_badips = badips.py[category="%(__name__)s", banaction="%(banaction)s", agent="%(fail2ban_agent)s"]
action_badips_report = badips[category="%(__name__)s", agent="%(fail2ban_agent)s"]

action = %(action_)s
```

Pour envoyer un mail contenant le whois, placez la variable sur :

	action = %(action_mw)s

Pour envoyer un mail avec le whois ET les logs, placez la variable sur :

	action = %(action_mwl)s


Pour activer la surveillance d'un service, il suffit de placer la variable "enabled" à "true"  

Par défaut la protection du service SSH est activée, pas les autres.

Si votre ssh n'écoute pas sur le port 22, pensez à le changer... (port = N° de port).

Rappelez vous également que le loglevel de SSHD (**/etc/ssh/sshd_config**) doit absolument être positionné sur DEBUG (**LogLevel DEBUG**) sans quoi, Fail2ban ne bloquera rien concernant SSH.  

```
#
# JAILS
#

# SSH
[sshd]

enabled = true
port    = 55027
logpath = %(sshd_log)s
backend = %(sshd_backend)s

# Mail servers
[postfix]

enabled  = true
port     = smtp,ssmtp,submission
filter   = postfix
logpath  = /var/log/mail.log

[sasl]

enabled  = true
port     = smtp,ssmtp,submission,imap2,imap3,imaps,pop3,pop3s
filter   = postfix-sasl
# You might consider monitoring /var/log/mail.warn instead if you are
# running postfix since it would provide the same log lines at the
# "warn" level but overall at the smaller filesize.
logpath  = /var/log/mail.log

[dovecot]

enabled = true
port    = smtp,ssmtp,submission,imap2,imap3,imaps,pop3,pop3s
filter  = dovecot
logpath = /var/log/mail.log

# nginx
[nginx-http-auth]

enabled = true
port    = http,https
logpath = %(nginx_error_log)s
```

Redémarrer fail2ban pour implémenter les règles:

    systemctl restart fail2ban

Afficher les règles de pare-feu actuelles

    iptables -S

```
-P INPUT DROP
-P FORWARD DROP
-P OUTPUT ACCEPT
-N f2b-dovecot
-N f2b-nginx-http-auth
-N f2b-postfix
-N f2b-sasl
-N f2b-sshd
-A INPUT -p tcp -m multiport --dports 25,465,587 -j f2b-postfix
-A INPUT -p tcp -m multiport --dports 80,443 -j f2b-nginx-http-auth
-A INPUT -p tcp -m multiport --dports 55027 -j f2b-sshd
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -p tcp -m tcp --dport 55027 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 53 -j ACCEPT
-A INPUT -p udp -m udp --dport 53 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 993 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 587 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 25 -j ACCEPT
-A OUTPUT -o lo -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 55027 -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 53 -j ACCEPT
-A OUTPUT -p udp -m udp --dport 53 -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 443 -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 993 -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 587 -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 25 -j ACCEPT
-A f2b-dovecot -j RETURN
-A f2b-nginx-http-auth -j RETURN
-A f2b-postfix -j RETURN
-A f2b-sasl -j RETURN
-A f2b-sshd -j RETURN
```


## UFW avec Fail2ban

Comme nous le savons, ufw est un très bon choix en raison de sa facilité d'utilisation, pour un environnement réseau peu complexe, je l'utilise généralement au lieu d'écrire directement des règles iptables. 

Mais fail2ban n'a pas de configuration jail pour ufw par défaut

**Comment utiliser ufw avec fail2ban ?**

Avant tout, installons fail2ban sur le dispositif, nous pouvons simplement utiliser le gestionnaire de paquets pour le faire, voici ce que je fais sous Debian 

    sudo apt install fail2ban

Nous devons savoir que fail2ban fonctionne avec deux fichiers de configuration habituellement, les fichiers de configuration jail et filter :

*    configuration jail : définit chaque politique jail.
*    configuration du filtre : définit le fonctionnement de chaque filtre.

Dans la plupart des cas, nous pouvons trouver le fichier de configuration jail par défaut dans `/etc/fail2ban/jail.conf`, nous devrions d'abord sauvegarder ce fichier, ou simplement créer une autre configuration jail sous `/etc/fail2ban/jail.local` et désactiver certaines règles en double de l'ancienne. Voici ma configuration jail `/etc/fail2ban/jail.local` maintenant :

```
[ufw]
enabled=true
filter=ufw.aggressive
action=iptables-allports
logpath=/var/log/ufw.log
maxretry=1
bantime=-1

[sshd]
enabled=true
filter=sshd
mode=normal
port=22
protocol=tcp
logpath=/var/log/auth.log
maxretry=3
bantime=-1
ignoreip = 127.0.0.0/8 ::1
```

Vous pouvez définir votre propre politique de jail comme vous le souhaitez. Et maintenant, nous avons besoin d'une configuration de filtre pour définir comment ufw jail fonctionne exactement, créons un fichier de configuration de filtre `/etc/fail2ban/filter.d/ufw.aggressive.conf` et ajoutons ceci :

```
[Definition]
failregex = [UFW BLOCK].+SRC=<HOST> DST
ignoreregex =
```

Cela signifie qu'il faut récupérer l'IP source qui est rejetée par l'ufw, afin qu'elle puisse fonctionner avec la politique que nous avons définie ci-dessus. Enfin, démarrons fail2ban :

```bash
systemctl enable fail2ban
systemctl start fail2ban
systemctl status fail2ban
```

Pour vérifier l'état de la prison :

```bash
fail2ban-client status ufw
fail2ban-client status sshd
```

Débannir une IP d'un jail spécifique :

    fail2ban-client set ufw unbanip 192.0.2.2

Pour plus d'informations, vous pouvez simplement lire la [documentation de fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page).

Après avoir configuré le service fail2ban comme ci-dessus, toute personne essayant de scanner les ports du serveur se verra refuser l'accès à tous les ports du serveur à partir du moment où il commence à scanner et est bloqué par ufw pour la première fois, il ne va pas trouver d'autres ports ouverts et les ports encore accessibles pour d'autres personnes. Mais vous devez faire attention en même temps, si vous avez bloqué les requêtes ICMP dans l'ufw, fail2ban bloquera toute personne qui essaiera d'envoyer un ping à votre serveur.
{: .prompt-warning }

Pendant un certain temps d'utilisation, je me suis bloqué une fois, et j'ai dû utiliser un autre réseau pour me connecter au serveur. Je dois dire que c'est mon erreur, mais voici une autre chose, parfois j'utilise mon serveur comme un proxy, un jour je ne peux pas me connecter à Telegram, et j'ai découvert que c'est parce que fail2ban a interdit une adresse IP Telegram, je n'ai aucune idée comment cette chose est arrivée. Donc, ces histoires nous ont dit, utilisez cet outil avec précaution, et n'oubliez pas d'ajouter les adresses IP auxquelles votre service doit se connecter comme IP à ignorer.  
{: .prompt-warning }

L'environnement du réseau public est très complexe et dangereux, et il existe d'innombrables dispositifs malveillants qui tentent d'analyser les adresses IP sur Internet en permanence. Cela signifie que l'utilisation de fail2ban sur le réseau public nécessite une lecture fréquente des journaux et une écriture fréquente des règles de pare-feu. Même si un seul exemple ne suffit pas, je dois mentionner que fail2ban a cassé mon service ufw une fois sur mon Raspberry Pi. Si vous utilisez fail2ban dans un environnement réseau complexe et sur un appareil bas de gamme, vous devrez peut-être faire plus attention à l'état du service.
{: .prompt-warning }

