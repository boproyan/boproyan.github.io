+++
title = '2019-10-18-xoyize.xyz-serveur-mail-complet-et-moderne'
date = 2019-10-21 00:00:00 +0100
categories = messagerie
+++
# xoyize.xyz - Serveur de Messagerie complet et moderne (MariaDB)

* [Article original rédigé par citizenz](https://www.citizenz.info/un-serveur-de-mail-complet-et-moderne)  


Système de base : Debian Buster  
Hébergeur : local  
Composants du serveur de mail :  

* Postfix (SMTP)Dovecot (IMAP)
* Rspamd (Antispam)
* Rainloop (Webmail)
* Dkim
* Filtres Sieve

### Passez en root et mise à jour distribution

    sudo -s
    apt-get update && apt-get upgrade

### Créer un groupe et un user vmail :

    groupadd -g 5000 vmail
    useradd -u 5000 -g vmail -s /usr/sbin/nologin -d /var/mail/vmail -m vmail

### Installation appli utiles + nginx, php, mysql (Mariadb)

Installation complémentaire (suite compilation Nginx + PHP7.3 + MariaDB)

    apt install php7.3-imap php7.3-xmlrpc php7.3-xsl php7.3-pspell php7.3-recode php-memcache memcached php-gettext php-pear mcrypt libruby

Installation complète

    apt install curl git unzip ntp ntpdate openssl php7.3-fpm php7.3 php7.3-common php7.3-gd php7.3-mysql php7.3-imap php7.3-cli php7.3-cgi php-pear mcrypt imagemagick libruby php7.3-curl php7.3-intl php7.3-pspell php7.3-recode php7.3-sqlite3 php7.3-tidy php7.3-xmlrpc php7.3-xsl memcached php-memcache php-imagick php-gettext php7.3-zip php7.3-mbstring

Outils (facultatif)

    apt install mc screen htop vim-nox nginx mariadb-server 

### MariaDB/Mysql

On sécurise Mysql en ajoutant un mot de passe root 

    mysql_secure_installation

* Enter current password for root (enter for none): ENTREE
* Set root password? [Y/n] Tapez Y
* New password: Entrez le mot de passe pour le user root
* Re-enter new password: mot de passe de nouveau...
* Remove anonymous users? [Y/n] Tapez Y
* Disallow root login remotely? [Y/n] Tapez Y
* Remove test database and access to it? [Y/n] Tapez Y
* Reload privilege tables now? [Y/n] Tapez Y

## PostfixAdmin

Télécharger Postfixadmin (la dernière version au 09/10/19 est la 3.2) 

    wget https://downloads.sourceforge.net/project/postfixadmin/postfixadmin/postfixadmin-3.2/postfixadmin-3.2.tar.gz
    tar xzf postfixadmin-3.2.tar.gz

Déplacer postfixadmin &rarr; /var/www/postfixadmin et créer le répertoire templates_c :

    mv postfixadmin-3.2/ /var/www/postfixadmin
    rm -f postfixadmin-3.2.tar.gz
    mkdir /var/www/postfixadmin/templates_c

On change les droits :

    chown -R www-data: /var/www/postfixadmin

Générer un mot de passe

    echo $(date +%s | sha256sum | base64 | head -c 12 ;) > /etc/mysql/pfa

Créer la base mysql pour postfixadmin

    mysql -u root -p$(cat /etc/mysql/mdp) -e "CREATE DATABASE postfixadmin; GRANT ALL ON postfixadmin.* TO 'postfixadmin'@'localhost' IDENTIFIED BY '$(cat /etc/mysql/pfa)'; FLUSH PRIVILEGES;"

Configuration  **/var/www/postfixadmin/config.local.php**

```php
<?php
$CONF['configured'] = true;

$CONF['database_type'] = 'mysqli';
$CONF['database_host'] = 'localhost';
$CONF['database_user'] = 'postfixadmin';
$CONF['database_password'] = 'NjM4ZTIyMWFj';
$CONF['database_name'] = 'postfixadmin';

$CONF['default_aliases'] = array (
  'abuse'      => 'abuse@xoyize.xyz',
  'hostmaster' => 'hostmaster@xoyize.xyz',
  'postmaster' => 'postmaster@xoyize.xyz',
  'webmaster'  => 'webmaster@xoyize.xyz'
);

$CONF['fetchmail'] = 'NO';
$CONF['show_footer_text'] = 'NO';

$CONF['quota'] = 'YES';
$CONF['domain_quota'] = 'YES';
$CONF['quota_multiplier'] = '1024000';
$CONF['used_quotas'] = 'YES';
$CONF['new_quota_table'] = 'YES';

$CONF['aliases'] = '0';
$CONF['mailboxes'] = '0';
$CONF['maxquota'] = '0';
$CONF['domain_quota_default'] = '0';
?>
```

Installer le schéma de la base en lançant le script suivant 

    sudo -u www-data php /var/www/postfixadmin/public/upgrade.php

Créer le superadmin Postfixadmin 

    sudo bash /var/www/postfixadmin/scripts/postfixadmin-cli admin add

```
Welcome to Postfixadmin-CLI v0.2
---------------------------------------------------------------
Admin:  
> postmaster@xoyize.xyz
Password:  
> z2rcXytUpBhhh3
Password (again):  
> z2rcXytUpBhhh3
Super admin:
(Super admins have access to all domains, can manage domains and admin accounts.) (y/n)
> y
Domain:  
> xoyize.xyz
Active: (y/n)
> y
The admin postmaster@xoyize.xyz has been added!
```

Installer  Letsencrypt  certbot  (Si non effectué par acme)

```
# apt install software-properties-common lsb-release
# add-apt-repository ppa:certbot/certbot
# apt update
# apt install python-certbot-nginx
```

Créer le fichier Nginx dans **/etc/nginx/conf.d/mail.xoyize.xyz.conf**

```
	##
	# Virtual Host pfa.xoyize.xyz
	##

	server {
	    listen 80;
	    listen [::]:80;
	
	    ## redirect http to https ##
	    server_name mail.xoyize.xyz;
	    return  301 https://$server_name$request_uri;
	}
	
	server {
	    listen 443 ssl http2;
	    listen [::]:443 ssl http2;
	    server_name mail.xoyize.xyz;

	    include ssl_dh_headers_ocsp;

	    root /var/www/postfixadmin/public;
	    index index.php;
	
	   location / {      
	     try_files $uri $uri/ /index.php;   
 	  }   
	        location ~ \.php$ {
	           fastcgi_split_path_info ^(.+\.php)(/.+)$;
		   if (!-f $document_root$fastcgi_script_name) {return 404;}
	           fastcgi_pass unix:/run/php/php7.3-fpm.sock;   # PHP7.3
	           fastcgi_index index.php;
	           include fastcgi_params;
	    	   #fastcgi_param SCRIPT_FILENAME $request_filename;
		   fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	        }
	

	    access_log /var/log/nginx/mail.xoyize.xyz-access.log;
	    error_log /var/log/nginx/mail.xoyize.xyz-error.log;
	}

```

On va créer le certificat Letsencrypt et laisser certbot configurer le fichier Nginx pour nous :

    certbot --nginx -d mail.xoyize.xyz

On renseigne premièrement l'adresse e-mail à laquelle les alerte de renouvellement de certificat seront envoyées.  
Puis on accepte les "Termes du service" : A  
Je réponds ensuite NON (n) pour partager mon adresse e-mail.  
Enfin, je choisis la réponse 2 (Redirect) afin que toutes les requêtes soient redirigées en HTTPS automatiquement.  

Ouvrez votre fichier /etc/nginx/conf.d/mail.xoyize.xyz.conf et ajoutez http2 comme ceci  

```
../..
    listen [::]:443 ssl http2; # managed by Certbot
    listen 443 ssl http2; # managed by Certbot
../..
```

et on redémarre Nginx : 

    service nginx restart

setup password : NWRi9DAzNzc1
postmaster@xoyize.xyz z2rcXytUpBhhh3
yanmaster@xoyize.xyz OWJ2hZjQ4zYz

```
/etc/php/php-fpm.d/postfixadmin.conf

[postfixadmin]
user = postfixadmin
group = postfixadmin
listen = /run/php/postfixadmin.sock
listen.owner = www-data
listen.group = www-data
pm = ondemand
pm.max_children = 4

    server {
      listen 8081;
      server_name postfixadmin;
      root /usr/share/webapps/postfixadmin/public/;
      index index.php;
      charset utf-8;
     
      access_log /var/log/nginx/postfixadmin-access.log;
      error_log /var/log/nginx/postfixadmin-error.log;
     
      location / {
        try_files $uri $uri/ index.php;
      }
     
      location ~* \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi_params;
        fastcgi_pass unix:/run/php/postfixadmin.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
      }
    }

```

## Postfix & Dovecot

Les paquets Dovecot Debian Buster sont pas à jour.

    debconf-set-selections <<< "postfix postfix/mailname string $(hostname -f)"
    debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
    apt install postfix postfix-mysql dovecot-imapd dovecot-lmtpd dovecot-pop3d dovecot-mysql

### Configuration de Postfix

    mkdir -p /etc/postfix/sql

```
cat > /etc/postfix/sql/mysql_virtual_domains_maps.cf << EOF
user = postfixadmin
password = NTUyMWFhZGFj
hosts = 127.0.0.1
dbname = postfixadmin
query = SELECT domain FROM domain WHERE domain='%s' AND active = '1'
EOF
 
cat > /etc/postfix/sql/mysql_virtual_alias_maps.cf << EOF
user = postfixadmin
password = NTUyMWFhZGFj
hosts = 127.0.0.1
dbname = postfixadmin
query = SELECT goto FROM alias WHERE address='%s' AND active = '1'
EOF
 
cat > /etc/postfix/sql/mysql_virtual_alias_domain_maps.cf << EOF
user = postfixadmin
password = NTUyMWFhZGFj
hosts = 127.0.0.1
dbname = postfixadmin
query = SELECT goto FROM alias,alias_domain WHERE alias_domain.alias_domain = '%d' and alias.address = CONCAT('%u', '@', alias_domain.target_domain) AND alias.active = 1 AND alias_domain.active='1'
EOF
 
cat > /etc/postfix/sql/mysql_virtual_alias_domain_catchall_maps.cf << EOF
user = postfixadmin
password = NTUyMWFhZGFj
hosts = 127.0.0.1
dbname = postfixadmin
query  = SELECT goto FROM alias,alias_domain WHERE alias_domain.alias_domain = '%d' and alias.address = CONCAT('@', alias_domain.target_domain) AND alias.active = 1 AND alias_domain.active='1'
EOF
 
cat > /etc/postfix/sql/mysql_virtual_mailbox_maps.cf << EOF
user = postfixadmin
password = NTUyMWFhZGFj
hosts = 127.0.0.1
dbname = postfixadmin
query = SELECT maildir FROM mailbox WHERE username='%s' AND active = '1'
EOF
 
cat > /etc/postfix/sql/mysql_virtual_alias_domain_mailbox_maps.cf << EOF
user = postfixadmin
password = NTUyMWFhZGFj
hosts = 127.0.0.1
dbname = postfixadmin
query = SELECT maildir FROM mailbox,alias_domain WHERE alias_domain.alias_domain = '%d' and mailbox.username = CONCAT('%u', '@', alias_domain.target_domain) AND mailbox.active = 1 AND alias_domain.active='1'
EOF

```

Puis on complète la configuration du fichier **/etc/postfix/main.cf** de Postfix 

```
postconf -e "virtual_mailbox_domains = mysql:/etc/postfix/sql/mysql_virtual_domains_maps.cf"
postconf -e "virtual_alias_maps = mysql:/etc/postfix/sql/mysql_virtual_alias_maps.cf, mysql:/etc/postfix/sql/mysql_virtual_alias_domain_maps.cf, mysql:/etc/postfix/sql/mysql_virtual_alias_domain_catchall_maps.cf"
postconf -e "virtual_mailbox_maps = mysql:/etc/postfix/sql/mysql_virtual_mailbox_maps.cf, mysql:/etc/postfix/sql/mysql_virtual_alias_domain_mailbox_maps.cf"

postconf -e "virtual_transport = lmtp:unix:private/dovecot-lmtp"
postconf -e 'smtp_tls_security_level = may'
postconf -e 'smtpd_tls_security_level = may'
postconf -e 'smtp_tls_note_starttls_offer = yes'
postconf -e 'smtpd_tls_loglevel = 1'
postconf -e 'smtpd_tls_received_header = yes'
 
postconf -e 'smtpd_tls_cert_file = /etc/letsencrypt/live/mail.exampple.com/fullchain.pem'
postconf -e 'smtpd_tls_key_file = /etc/letsencrypt/live/mail.xoyize.xyz/privkey.pem'
 
postconf -e 'smtpd_sasl_type = dovecot'
postconf -e 'smtpd_sasl_path = private/auth'
postconf -e 'smtpd_sasl_local_domain ='
postconf -e 'smtpd_sasl_security_options = noanonymous'
postconf -e 'broken_sasl_auth_clients = yes'
postconf -e 'smtpd_sasl_auth_enable = yes'
postconf -e 'smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination'
```

On configure le fichier **/etc/postfix/master.cf** de Postfix 

```
submission inet n       -       y       -       -       smtpd 
-o syslog_name=postfix/submission 
-o smtpd_tls_security_level=encrypt 
-o smtpd_sasl_auth_enable=yes
#  -o smtpd_reject_unlisted_recipient=no 
-o smtpd_client_restrictions=permit_sasl_authenticated,reject
#  -o smtpd_helo_restrictions=$mua_helo_restrictions
#  -o smtpd_sender_restrictions=$mua_sender_restrictions
#  -o smtpd_recipient_restrictions=
#  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject 
-o milter_macro_daemon_name=ORIGINATING
smtps     inet  n       -       y       -       -       smtpd 
-o syslog_name=postfix/smtps 
-o smtpd_tls_wrappermode=yes 
-o smtpd_sasl_auth_enable=yes
#  -o smtpd_reject_unlisted_recipient=no 
-o smtpd_client_restrictions=permit_sasl_authenticated,reject
#  -o smtpd_helo_restrictions=$mua_helo_restrictions
#  -o smtpd_sender_restrictions=$mua_sender_restrictions
#  -o smtpd_recipient_restrictions=
#  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject 
-o milter_macro_daemon_name=ORIGINATING
```

... et on redémarre Postfix : 

    service postfix restart

### Configuration Dovecot

On va configurer Dovecot :

    nano /etc/dovecot/dovecot-sql.conf.ext

```
driver = mysqlconnect = host=127.0.0.1 dbname=postfixadmin user=postfixadmin password=NjM4ZTIyMWFj
default_pass_scheme = MD5-CRYPT
iterate_query = SELECT username AS user FROM mailbox
user_query = SELECT CONCAT('/var/mail/vmail/',maildir) AS home, \  CONCAT('maildir:/var/mail/vmail/',maildir) AS mail, \  5000 AS uid, 5000 AS gid, CONCAT('*:bytes=',quota) AS quota_rule \  FROM mailbox WHERE username = '%u' AND active = 1
password_query = SELECT username AS user,password FROM mailbox \  WHERE username = '%u' AND active='1'
```

    nano /etc/dovecot/conf.d/10-mail.conf

```
...
mail_location = maildir:/var/mail/vmail/%d/%n
...
mail_uid = vmail
mail_gid = vmail
...
first_valid_uid = 5000
last_valid_uid = 5000
...
mail_privileged_group = mail
...
mail_plugins = quota
...
```

    nano /etc/dovecot/conf.d/10-auth.conf

```
...
disable_plaintext_auth = yes
...
auth_mechanisms = plain login
...
#!include auth-system.conf.ext
!include auth-sql.conf.ext
...
```

    nano /etc/dovecot/conf.d/10-master.conf

```
#default_process_limit = 100
#default_client_limit = 1000

# Default VSZ (virtual memory size) limit for service processes. This is mainly
# intended to catch and kill processes that leak memory before they eat up
# everything.
#default_vsz_limit = 256M

# Login user is internally used by login processes. This is the most untrusted
# user in Dovecot system. It shouldn't have access to anything at all.
#default_login_user = dovenull

# Internal user is used by unprivileged processes. It should be separate from
# login user, so that login processes can't disturb other processes.
#default_internal_user = dovecot

service imap-login {
  inet_listener imap {
    port = 0
  }
  inet_listener imaps {
    port = 993
    ssl = yes
  }

  # Number of connections to handle before starting a new process. Typically
  # the only useful values are 0 (unlimited) or 1. 1 is more secure, but 0
  # is faster. <doc/wiki/LoginProcess.txt>
  #service_count = 1

  # Number of processes to always keep waiting for more connections.
  #process_min_avail = 0

  # If you set service_count=0, you probably need to grow this.
  #vsz_limit = 64M
}

service pop3-login {
  inet_listener pop3 {
    port = 0
  }
  inet_listener pop3s {
    port = 995
    ssl = yes
  }
}

service lmtp {
 unix_listener /var/spool/postfix/private/dovecot-lmtp {
   mode = 0600
   user = postfix
   group = postfix
  }
  # Create inet listener only if you can't use the above UNIX socket
  #inet_listener lmtp {
    # Avoid making LMTP visible for the entire internet
    #address =
    #port = 
  #}
}

service imap {
  # Most of the memory goes to mmap()ing files. You may need to increase this
  # limit if you have huge mailboxes.
  #vsz_limit = 256M

  # Max. number of IMAP processes (connections)
  #process_limit = 1024
}

service pop3 {
  # Max. number of POP3 processes (connections)
  #process_limit = 1024
}

service auth {
  # auth_socket_path points to this userdb socket by default. It's typically
  # used by dovecot-lda, doveadm, possibly imap process, etc. Its default
  # permissions make it readable only by root, but you may need to relax these
  # permissions. Users that have access to this socket are able to get a list
  # of all usernames and get results of everyone's userdb lookups.
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }

  unix_listener auth-userdb {
    mode = 0600
    user = vmail
    group = vmail
  }

  # Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }

  # Auth process is run as this user.
  user = dovecot
}

service auth-worker {
  # Auth worker process is run as root by default, so that it can access
  # /etc/shadow. If this isn't necessary, the user should be changed to
  # $default_internal_user.
  user = vmail
}

service dict {
  # If dict proxy is used, mail processes should have access to its socket.
  # For example: mode=0660, group=vmail and global mail_access_groups=vmail
  unix_listener dict {
    mode = 0600
    user = vmail
    group = vmail
  }
}

```


    nano /etc/dovecot/conf.d/10-ssl.conf

```
...
ssl = yes
...
ssl_cert = </etc/ssl/private/xoyize.xyz-fullchain.pem
ssl_key = </etc/ssl/private/xoyize.xyz-key.pem
...
ssl_dh = </etc/ssl/private/dh2048.pem
...
ssl_min_protocol = TLSv1.2
...
ssl_cipher_list = EECDH+AES:EDH+AES+aRSA
...
ssl_prefer_server_ciphers = yes
...
```

    nano /etc/dovecot/conf.d/20-imap.conf

```
...
protocol imap { 
...
  mail_plugins = $mail_plugins imap_quota
  ...
}
...
```

    nano /etc/dovecot/conf.d/20-lmtp.conf

```
##
## LMTP specific settings
##

protocol imap {
  postmaster_address = postmaster@xoyize.xyz
  mail_plugins = $mail_plugins
}
```

    nano /etc/dovecot/conf.d/15-mailboxes.conf

```
##
## Mailbox definitions
##

# Each mailbox is specified in a separate mailbox section. The section name
# specifies the mailbox name. If it has spaces, you can put the name
# "in quotes". These sections can contain the following mailbox settings:
#
# auto:
#   Indicates whether the mailbox with this name is automatically created
#   implicitly when it is first accessed. The user can also be automatically
#   subscribed to the mailbox after creation. The following values are
#   defined for this setting:
# 
#     no        - Never created automatically.
#     create    - Automatically created, but no automatic subscription.
#     subscribe - Automatically created and subscribed.
#  
# special_use:
#   A space-separated list of SPECIAL-USE flags (RFC 6154) to use for the
#   mailbox. There are no validity checks, so you could specify anything
#   you want in here, but it's not a good idea to use flags other than the
#   standard ones specified in the RFC:
#
#     \All      - This (virtual) mailbox presents all messages in the
#                 user's message store. 
#     \Archive  - This mailbox is used to archive messages.
#     \Drafts   - This mailbox is used to hold draft messages.
#     \Flagged  - This (virtual) mailbox presents all messages in the
#                 user's message store marked with the IMAP \Flagged flag.
#     \Junk     - This mailbox is where messages deemed to be junk mail
#                 are held.
#     \Sent     - This mailbox is used to hold copies of messages that
#                 have been sent.
#     \Trash    - This mailbox is used to hold messages that have been
#                 deleted.
#
# comment:
#   Defines a default comment or note associated with the mailbox. This
#   value is accessible through the IMAP METADATA mailbox entries
#   "/shared/comment" and "/private/comment". Users with sufficient
#   privileges can override the default value for entries with a custom
#   value.

# NOTE: Assumes "namespace inbox" has been defined in 10-mail.conf.
namespace inbox {
  # These mailboxes are widely used and could perhaps be created automatically:
  mailbox Drafts {
    special_use = \Drafts
  }
  mailbox Spam {
   special_use = \Junk
   auto = subscribe
  }
  mailbox Junk {
    special_use = \Junk
  }
  mailbox Trash {
    special_use = \Trash
  }

  # For \Sent mailboxes there are two widely used names. We'll mark both of
  # them as \Sent. User typically deletes one of them if duplicates are created.
  mailbox Sent {
    special_use = \Sent
  }
  mailbox "Sent Messages" {
    special_use = \Sent
  }

  # If you have a virtual "All messages" mailbox:
  #mailbox virtual/All {
  #  special_use = \All
  #  comment = All my messages
  #}

  # If you have a virtual "Flagged" mailbox:
  #mailbox virtual/Flagged {
  #  special_use = \Flagged
  #  comment = All my flagged messages
  #}
}
```
 
    nano /etc/dovecot/conf.d/90-quota.conf

```
plugin {
 quota = dict:User quota::proxy::sqlquota
 quota_rule = *:storage=5GB
 quota_rule2 = Trash:storage=+100M
 quota_grace = 10%%
 quota_exceeded_message = Quota exceeded, please contact your system administrator.
}

plugin {
 quota_warning = storage=100%% quota-warning 100 %u
 quota_warning2 = storage=95%% quota-warning 95 %u
 quota_warning3 = storage=90%% quota-warning 90 %u
 quota_warning4 = storage=85%% quota-warning 85 %u
}

service quota-warning {
 executable = script /usr/local/bin/quota-warning.sh
 user = vmail

 unix_listener quota-warning {
  group = vmail
  mode = 0660
  user = vmail
 }
}

dict {
 sqlquota = mysql:/etc/dovecot/dovecot-dict-sql.conf.ext
}
```

    nano /etc/dovecot/dovecot-dict-sql.conf.ext

```
...
connect = host=127.0.0.1 dbname=postfixadmin user=postfixadmin password=NjM4ZTIyMWFj
...
map {
  pattern = priv/quota/storage
  table = quota2
  username_field = username
  value_field = bytes
}
map {
  pattern = priv/quota/messages
  table = quota2
  username_field = username
  value_field = messages
}
...
# map {
#   pattern = shared/expire/$user/$mailbox
#   table = expires
#   value_field = expire_stamp
#
#   fields {
#     username = $user
#     mailbox = $mailbox
#   }
# }
...
```

    nano /usr/local/bin/quota-warning.sh

```
#!/bin/sh
PERCENT=$1
USER=$2
cat << EOF | /usr/lib/dovecot/dovecot-lda -d $USER -o "plugin/quota=dict:User quota::noenforcing:proxy::sqlquota"
From: postmaster@xoyize.xyz
Subject: Quota warning
Your mailbox is $PERCENT% full. Don't forget to make a backup of old messages to remain able to receive mails.
EOF
```

    chmod +x /usr/local/bin/quota-warning.sh

et on redémarre Dovecot : 

    service dovecot restart

## RSPAMD

On commence par installer redis-server :

	apt install redis-server

Puis on installe Rspamd 

```
apt install software-properties-common lsb-release
wget -O- https://rspamd.com/apt-stable/gpg.key | apt-key add -
echo "deb http://rspamd.com/apt-stable/ $(lsb_release -cs) main" > /etc/apt/sources.list.d/rspamd.list
apt update
apt install rspamd
```

Configurer Rspamd

    nano /etc/rspamd/local.d/worker-normal.inc

```
bind_socket = "127.0.0.1:11333";
```

    nano /etc/rspamd/local.d/worker-proxy.inc

```
bind_socket = "127.0.0.1:11332";
milter = yes;
timeout = 120s;
upstream "local" {
  default = yes;
  self_scan = yes;
}
```

Il faut créer un mot de passe pour l'interface web de Rspamd :

    rspamadm pw --encrypt -p NTUyMWFhZGFj

Notez bien la clé qui va s'afficher. Elle ressemblera à ça : 

```
$2$b51na53x357bjrz5khmewri4o7um4s8i$mzegmbea3osb4yuzx93o4qzjonft8h87i9pxu9gwgyx9wkqqizab
```

    nano /etc/rspamd/local.d/worker-controller.inc

```
password = "$2$b51na53x357bjrz5khmewri4o7um4s8i$mzegmbea3osb4yuzx93o4qzjonft8h87i9pxu9gwgyx9wkqqizab"
```

Puis on con tinue la configuration :

    nano /etc/rspamd/local.d/classifier-bayes.conf

```
servers = "127.0.0.1";
backend = "redis";
```

    nano /etc/rspamd/local.d/milter_headers.conf

```
use = ["x-spamd-bar", "x-spam-level", "authentication-results"];
```

On rédémarre Rspamd :

   systemctl restart rspamd

Configurer Nginx pour l'interface web de Rspamd

    nano /etc/nginx/conf.d/spam.xoyize.xyz.conf

```
	##
	# Virtual Host spam.xoyize.xyz (Rspamd)
	##

	server {
	    listen 80;
	    listen [::]:80;
	
	    ## redirect http to https ##
	    server_name spam.xoyize.xyz;
	    return  301 https://$server_name$request_uri;
	}
	
	server {
	    listen 443 ssl http2;
	    listen [::]:443 ssl http2;
	    server_name spam.xoyize.xyz;
	
	    include ssl_dh_headers_ocsp;
	
	    location / {
              proxy_pass http://127.0.0.1:11334/;
              proxy_set_header Host $host;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	    }
	
	}
```

Configuration avec cerbot let's encrypt

```
server {
listen 80;  
listen [::]:80;  
server_name spam.xoyize.xyz;  
location / {  
   proxy_pass http://127.0.0.1:11334/;     
   proxy_set_header Host $host;     
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;     
}  
}
On va demander et installer un certificat Lets'encrypt :
# certbot --nginx -d spam.xoyize.xyz
```

On va terminer la configuration de Postfix on intégrant les paramètres pour Rspamd 

```
postconf -e "milter_protocol = 6"
postconf -e "milter_mail_macros = i {mail_addr} {client_addr} {client_name} {auth_authen}"
postconf -e "milter_default_action = accept"
postconf -e "smtpd_milters = inet:127.0.0.1:11332"
postconf -e "non_smtpd_milters = inet:127.0.0.1:11332"
```

... et on redémarre Postfix : 

    systemctl restart postfix

## Dovecot ,terminer la configuration

On termine la config de Dovecot en intégrant le module Sieve pour filtrer les mails

    apt install dovecot-sieve dovecot-managesieved

    nano /etc/dovecot/conf.d/20-lmtp.conf

```
protocol lmtp {
postmaster_address = postmaster@xoyize.xyz
mail_plugins = $mail_plugins sieve
}
```

    nano /etc/dovecot/conf.d/20-imap.conf

```
...
protocol imap {
...
mail_plugins = $mail_plugins imap_quota imap_sieve
...
}
...
```

    nano /etc/dovecot/conf.d/20-managesieve.conf

```
...
service managesieve-login {
inet_listener sieve {
port = 4190  
}
...
}
..
service managesieve {
process_limit = 1024
}
...
```

    nano /etc/dovecot/conf.d/90-sieve.conf

```
plugin {
...  
# sieve = file:~/sieve;active=~/.dovecot.sieve  
  sieve_plugins = sieve_imapsieve sieve_extprograms
  sieve_before = /var/mail/vmail/sieve/global/spam-global.sieve
  sieve = file:/var/mail/vmail/sieve/%d/%n/scripts;active=/var/mail/vmail/sieve/%d/%n/active-script.sieve
  imapsieve_mailbox1_name = Spam
  imapsieve_mailbox1_causes = COPY
  imapsieve_mailbox1_before = file:/var/mail/vmail/sieve/global/report-spam.sieve
  imapsieve_mailbox2_name = *
  imapsieve_mailbox2_from = Spam
  imapsieve_mailbox2_causes = COPY
  imapsieve_mailbox2_before = file:/var/mail/vmail/sieve/global/report-ham.sieve
  sieve_pipe_bin_dir = /usr/bin
  sieve_global_extensions = +vnd.dovecot.pipe
....  
}
```

Créer un répertoire qui va accueillir les scripts Sieve:

    mkdir -p /var/mail/vmail/sieve/global

    nano /var/mail/vmail/sieve/global/spam-global.sieve

```
require ["fileinto","mailbox"];

if anyof(
header :contains ["X-Spam-Flag"] "YES",  
header :contains ["X-Spam"] "Yes",  
header :contains ["Subject"] "*** SPAM ***"  
)  
{
fileinto :create "Spam";  
stop;  
}
```

    nano /var/mail/vmail/sieve/global/report-spam.sieve

```
require ["vnd.dovecot.pipe", "copy", "imapsieve"];
pipe :copy "rspamc" ["learn_spam"];
```

    nano /var/mail/vmail/sieve/global/report-ham.sieve

```
require ["vnd.dovecot.pipe", "copy", "imapsieve"];
pipe :copy "rspamc" ["learn_ham"];
```

... et on redémarre Dovecot : 

    systemctl restart dovecot

Il faut maintenant compiler les scripts Sieve et appliquer les bons droits :

```
sievec /var/mail/vmail/sieve/global/spam-global.sieve
sievec /var/mail/vmail/sieve/global/report-spam.sieve
sievec /var/mail/vmail/sieve/global/report-ham.sieve
chown -R vmail: /var/mail/vmail/sieve/
```

## DKIM 

Dkim   
DomainKeys Identified Mail (DKIM) est une méthode d'authentification par courrier électronique qui ajoute une signature cryptographique aux en-têtes des messages sortants.
Il permet au destinataire de vérifier qu'un courrier électronique prétendant provenir d'un domaine spécifique a bien été autorisé par le propriétaire de ce domaine.
Le but principal de cette opération est d'empêcher les messages électroniques falsifiés.

On peut avoir différentes clés DKIM pour tous les domaines et même plusieurs clés pour un seul domaine, mais pour simplifier cet article, nous allons utiliser une seule clé DKIM qui pourra ultérieurement être utilisée pour tous les nouveaux domaines.

Créer un répertoire afin d'accueillir les clés Dkim  

    mkdir /var/lib/rspamd/dkim/

Et on va créer la clé Dkim :

    rspamadm dkim_keygen -b 2048 -s mail -k /var/lib/rspamd/dkim/mail.key /var/lib/rspamd/dkim/mail.pub

```
mail._domainkey IN TXT ( "v=DKIM1; k=rsa; "
	"p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxWCZMxdrtElvmkKFm/HbArM2m08yx/amRnNQgyIO0ODS+GkmKOBvXWn2cbtqbC4tlNFZa2OP0CVa2I97hYYb2H+JHwwyET9p0hDZRkiK/ckiQc2CetA0Ugh0HfISzgfAw1FdSom57J0DOp4+dagNaWAknWYyFyGSpkXZrmeX3HT9AktecU/IeUaFf0XbEx9MCCaY2+fzUe0qfQWVz"
	"5OrO5jjKwZKBHPRPrsv+uQYqPZbthbOMPp2jSK1vgf/1nirAIlq3mB0LY0kX4lCp999ryvGHhAzvY4R1yzyD1OsHwe6jrhYOszmonKLUE+BB1zqmiywQtemKPevkWFmWLatMwIDAQAB"
) ; 
```

Dans cet exemple, on utilise "mail" comme sélecteur Dkim.  
On applique les bonnes permissions :  

    chown -R _rspamd: /var/lib/rspamd/dkim
    chmod 440 /var/lib/rspamd/dkim/*

On va enseuite "dire" à Rspamd de regarder au bon endroit pour les clés Dkim :

    nano /etc/rspamd/local.d/dkim_signing.conf

```
selector = "mail";
path = "/var/lib/rspamd/dkim/$selector.key";
allow_username_mismatch = true;
```

Rspamd prend également en charge la signature ARC (Authenticated Received Chain). Rspamd utilise le module dkim pour traiter les signatures ARC afin que nous puissions simplement copier la configuration précédente :

    cp /etc/rspamd/local.d/dkim_signing.conf /etc/rspamd/local.d/arc.conf

... et on redémarre Rspamd : 

    systemctl restart rspamd

## DNS 

Vous devez aussi configurer vos DNS.  
N'oubliez pas de configurer :  
- une entrée MX (serveur de mail) :  
Exemple : domain.com. MX 10 mail.xoyize.xyz.      
- une entrée SPF  
Exemple : domain.com. TXT "v=spf1 a mx ~all"     
- une entrée _dmarc  
Exemple : _dmarc.domain.com. TXT "v=DMARC1; p=none; adkim=r; aspf=r;"      
- une entrée Dkim  

Vous trouverez la clé publique Dkim ici :

   cat /var/lib/rspamd/dkim/mail.pub

```
mail._domainkey IN TXT ( "v=DKIM1; k=rsa; "
	"p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxWCZMxdrtElvmkKFm/HbArM2m08yx/amRnNQgyIO0ODS+GkmKOBvXWn2cbtqbC4tlNFZa2OP0CVa2I97hYYb2H+JHwwyET9p0hDZRkiK/ckiQc2CetA0Ugh0HfISzgfAw1FdSom57J0DOp4+dagNaWAknWYyFyGSpkXZrmeX3HT9AktecU/IeUaFf0XbEx9MCCaY2+fzUe0qfQWVz"
	"5OrO5jjKwZKBHPRPrsv+uQYqPZbthbOMPp2jSK1vgf/1nirAIlq3mB0LY0kX4lCp999ryvGHhAzvY4R1yzyD1OsHwe6jrhYOszmonKLUE+BB1zqmiywQtemKPevkWFmWLatMwIDAQAB"
) ; 
```

## RAINLOOP WEBMAIL

On télécharge Rainloop :

    cd /tmp
    wget http://www.rainloop.net/repository/webmail/rainloop-community-latest.zip
    mkdir /var/www/webmail
    unzip rainloop-community-latest.zip -d /var/www/webmail
    chown -R www-data:www-data /var/www/webmail

On crée le fichier Nginx pour Rainloop webmail :

    nano /etc/nginx/conf.d/webmail.xoyize.xyz.conf

```
server {
listen 80;  
listen [::]:80;  
root /var/www/webmail;  
index index.php index.html index.htm;   
server_name webmail.xoyize.xyz;  
client_max_body_size 100M;  
location / {  
   try_files $uri $uri/ /index.php?$query_string;  
}       
location ~ \.php$ {  
   include snippets/fastcgi-php.conf;       
   fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;       
   fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;       
   include fastcgi_params;       
}  
location ^~ /data {   
   deny all;       
}    
}
```

On crée et on installe un certificat Letsencrypt HTTPS :

    certbot --nginx -d webmail.xoyize.xyz

ET VOILA... ou presque. La configuration "technique" est terminée.  
Reste à configurer les noms de domaines et les boites mail dans Postfixadmin : https://mail.xoyize.xyz  
Je n’explique pas cette partie. Vous pourrez trouver plein d'exemples sur le net. 

Connection une fois tous les domaines et les boites mail configurées dans Postfixadmin, on va pouvoir se connecter sur l'interface d'admin de Rainloop à cette adresse : https://webmail.xoyize.xyz/?admin afin de configurer les domaines.
Une fois connecté à l'admin -- Onglet Domaine, Ajouter un domaine

A - IMAP + SMTP  
Partie IMAP  

Serveur : mail.xoyize.xyz  
Sécurité : SSL/TLS  
Port : 993  

Partie SMTP  
Serveur : mail.xoyize.xyz  
Sécurité : STARTTLS  
Port : 587  


Cliquez sur TEST, en bas à gauche de la fenêtre. Si TEST devient vert, la config est OK, si TEST devient rouge, il y a un truc qui cloche !

B - SIEVE  
Cliquez sur configuration sieve (beta)  
Cochez : Autoriser les scripts Sieve ET Autoriser les scripts personnels  

Partie SIEVE  
Serveur : mail.xoyize.xyz  
Sécurité : STARTTLS  
Port : 4190  

Partie SMTP  
Serveur : mail.xoyize.xyz  
Sécurité : STARTTLS  
Port : 587  
 
Cliquez enfin sur Modifier.  
 

Refaite cette même configuration pour chaque domaine que vous allez utiliser !

L'interface du webmail étant elle située ici : https://webmail.xoyize.xyz



[UPDATE] : pour activer l'auto learn (apprentissage) de Rspamd, créez le fichier suivant :

    nano /etc/rspamd/override.d/classifier-bayes.conf

```
autolearn = true;
Redémarrez Rspamd :
systemctl restart rspamd
```
