+++
title = 'Auto-Hébergement avec HomeBox'
date = 2019-12-23T22:25:42+01:00 00:00:00 +0100
categories = divers
+++
Nouvelle version de HomeBox, pour Noël.

Après plusieurs mois de travail, de quelques développeurs, notamment Frédéric et moi même, une nouvelle version de la solution d'auto-hébergement a été publiée. Pour l'instant, pas de numéro de version, mais plutôt un branche "master" sur github.

https://github.com/progmaticltd/homebox

Je rappelle que c'est pour l'instant une version qui se déploie avec Ansible, elle ne s'adresse actuellement pas aux néophytes qui voudraient faire de l'autohébergement en quelques clics, mais plutôt aux personnes soucieuses de sécurité.

## Installation du système

- Génération d'une image ISO Debian personnalisée avec chiffrement du disque (LUKS) et installation entièrement automatique.
- Déverrouillez le système au démarrage en entrant la phrase secrète via SSH ou avec un YubiKey.
- Installez les paquets uniquement depuis Debian (Stretch).
- Génération automatique de certificats SSL avec letsencrypt.
- Mises à jour de sécurité automatiques (facultatif).
- Authentification centralisée avec une base de données d'utilisateurs LDAP, certificat SSL, politiques de mot de passe, intégration PAM.
- AppArmor activé par défaut, profils pour tous les démons.
- Sauvegarde automatique des données de déploiement pour rejouer l'installation avec les mêmes données (Parfait pour une réinstallation après un désastre).
- Peut être utilisé à domicile, sur un serveur dédié ou virtuel hébergé en ligne.
- Prise en charge flexible des adresses IP: IPv4, IPv6, IPv4 + IPv4, IPv4 + IPv6.
- Serveur DNS intégré, avec prise en charge CAA, DNSSEC et SSHFP (SSH « fingerprint »).
- Sites HTTPS certifiés « grade A », HSTS implémenté par défaut.
- Serveur DNS intégré avec prise en charge des enregistrements DNSSEC et SSHFP (empreinte SSH)

## Fonctionnalités emails

- Configuration et installation de Postfix, avec recherches LDAP, alias de messagerie internationalisés, conformité SSL.
- Génération des clés DKIM et enregistrements des champs DNS SPF et DMARC.
- Copie automatique des e-mails envoyés dans le dossier envoyé (ala GMail).
- Création automatique du compte postmaster et des adresses e-mail spéciales à l'aide des spécifications RFC 2142.
- Configuration de Dovecot, IMAPS, POP3S, Quotas, ManageSieve, apprentissage simple du spam et des faux positifs en déplaçant les e-mails dans et hors du dossier Junk.
- Filtres Sieves et réponse automatique pendant les vacances.
- Dossiers virtuels pour la recherche de serveur: messages non lus, vue des conversations, tous les messages, marqués et messages étiquetés comme "importants", etc.
- Adresses e-mail avec délimiteur de destinataire inclus, par ex. john.doe+lists@dbcooper.com.
- Création d'utilisateur maître en option, par ex. pour les familles avec enfants ou les communautés modérées.
- Recherche plein texte côté serveur dans les e-mails, les documents et fichiers joints et  archives compressées, avec de meilleurs résultats que GMail.
- Rapport d'accès hebdomadaire, mensuel et annuel détaillé par pays, FAI, adresses IP, etc.
- Webmail RoundCube en option avec gestion des filtres sieve, formulaire de changement de mot de passe, création automatique d'identité, accès au compte principal, etc.
- Messagerie Web SOGo en option avec gestion des filtres Sieve, formulaire de changement de mot de passe, gestion du calendrier et du carnet d'adresses, interface graphique pour importer les autres e-mails du compte.
- Importation automatique des e-mails depuis Google Mail, Yahoo, Outlook.com ou tout autre compte IMAP standard.
- Système antispam puissant et léger avec rspamd et accès optionnel à l'interface web.
- Antivirus pour les e-mails entrants et sortants avec ClamAV.
- Configuration automatique pour Thunderbird et Outlook à l'aide de XML publié et d'autres clients avec des enregistrements DNS spéciaux (RFC 6186).
- Détection automatique des comportements inhabituels, avec avertissement en temps réel en utilisant XMPP et e-mail à une adresse externe.

## Calendrier et carnet d'adresses

- Installation et configuration un serveur CalDAV / CardDAV, avec détection automatique (RFC 6186).
- Fonctionnalité de Groupware dans une interface Web, avec SOGo.
- Événements récurrents, alertes par e-mail, carnets d'adresses et calendriers partagés.
- Compatibilité des appareils mobiles: Android, Apple iOS, BlackBerry 10 et Windows mobile via Microsoft ActiveSync.

## Autres fonctionnalités optionnelles

- Sauvegardes incrémentielles, chiffrées, sur plusieurs destinations (SFTP, S3, partage Samba ou clé USB), avec rapports par e-mail et Jabber.
- Serveur Jabber, utilisant ejabberd, avec authentification LDAP, transfert de fichiers direct ou hors ligne et communication facultative de serveur à serveur.
- Installation Tor et personnalisation possible.
- Installation facile de Privoxy, avec synchronisation quotidienne des règles adblock et chaînage Tor en option.
- Création d'un site web minimal statique, avec certificats https et niveau de sécurité A+ par défaut.
- Serveur de sauvegarde personnel pour chaque utilisateur, à l'aide de borgbackup.
- Gogs git server, un serveur git rapide et léger écrit en Golang.
- Démon de transmission, accessible avec https, public ou privé sur votre LAN. Les fichiers peuvent être téléchargés directement avec un navigateur Web possible.
- Surveillance avec Zabbix, avec des alertes par e-mail et Jabber.
- Masquez le serveur SSH avec Single Packet Authorization, à l'aide de fwknop.

## Développement

- Validation des fichiers YAML sur chaque commit, en utilisant travis-ci.
- Intégration continue à l'aide de Jenkins.
- Ansible lint (en cours)
- Tests d'intégration de bout en bout pour la majorité des composants.
- Playbooks pour faciliter l'installation ou la suppression des packages de développement.
- Indicateur de débogage global pour activer le mode de débogage de tous les composants.
- Scripts Ansible entièrement open source sous licence GPLv3.

Merci à Debian.

## À venir

- La version Buster sera développée en 2020.
- Utilisation d'un VPN pour fournir deux adresses IP statiques, IPv4 et IPv6, aux personnes utilisant une adresse IP dynamique.
- Plus de sécurité

## Liens:

- Page GitHub: https://github.com/progmaticltd/homebox
- Documentation: https://homebox.readthedocs.io/en/latest/
- Page d'intégration continue: https://jenkins.homebox.space/

## Listes de diffusion

- Questions générales: https://framalistes.org/sympa/info/homebox-general
- Dévelopement: https://framalistes.org/sympa/info/homebox-dev
