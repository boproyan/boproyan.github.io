+++
title = 'SSO Authelia'
date = 2025-02-14 00:00:00 +0100
categories = authentification
+++
*Authelia est un serveur d'authentification et d'autorisation open source fournissant une authentification à deux facteurs et une authentification unique (SSO) pour vos applications via un portail Web.* 


## Authelia

![](authelia-logo.png){:width="150" .left}    
Site web officiel : <https://www.authelia.com/>

*    Open source
*    SSO
*    Authentification 2F avec Yubikey
*    OTP basé sur le temps avec Google Authenticator
*    Prend en charge LDAP
*    Prend en charge Active Directory
*    Fonctionne avec les proxys inverses
*    Notifications push avec duo
*    Bloquer les attaques de force brute
*    Vérification d’identité
*    Une meilleure gestion des mots de passe
*    Flux de travail flexible
*    UI intuitif

### Prérequis et liens

Un serveur debian + nginx

* [Installer Authelia comme SSO - Guide complet](https://blog.cyril.by/fr/software/single-sign-on/installing-authelia-for-sso-all-in-one-guide)
* [[TUTO] [Docker] Authelia : serveur d'authentification](https://www.nas-forum.com/forum/topic/71875-tuto-docker-authelia-serveur-dauthentification/)
* [Tutoriel Authelia – Protégez votre pile Docker Traefik avec une MFA privée](https://mediacenterz.com/tutoriel-authelia-protegez-votre-pile-docker-traefik-avec-une-mfa-privee/)
* [Authelia, authentification au niveau du reverse proxy](https://blog.xataz.net/posts/authelia-authentification-au-niveau-du-reverse-proxy/)


### Installer Authelia

Ajout clés de signature Authelia + dépôt 

```shell
curl -s https://apt.authelia.com/organization/signing.asc | sudo tee /usr/share/keyrings/authelia.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/authelia.asc] https://apt.authelia.com/stable/debian/debian/ all main" | sudo tee /etc/apt/sources.list.d/authelia-stable-debian.list
```

Mise à jour des paquets et installation

    sudo apt update
    sudo apt install authelia

### Configurer Authelia

Clé de configuration | Signification | Exemple
 ------------------- | ------------- | -----------
jwt_secret | Une chaîne aléatoire qui ne peut pas être devinée par un attaquant | Générez-en une en tapant `head /dev/random | tr -dc A-Za-z0-9 | head -c64 > jwt_secret.txt` et collez-la ici entre guillemets.
default_redirection_url |  Si l'utilisateur navigue directement vers auth.myhome.com, Authelia ne peut pas déduire où le rediriger. Il s'agira donc probablement d'une page lister-toutes-les-applications-disponibles  | https://www.myhome.com
default_2fa_method | 2FA est l'abréviation de 2 factor authentication (authentification à deux facteurs). Il s'agit du processus ennuyeux qui vous demande d'entrer le code que vous avez reçu par SMS ou par l'intermédiaire d'une application d'authentification. | "totp", au moins c'est gratuit
server/host | Attention! Le serveur d'authentification est un serveur qui ne peut pas être utilisé par les utilisateurs de l'Internet. Si vous laissez la valeur par défaut 0.0.0.0 alors Authelia écoutera sur l'interface réseau de votre serveur. Mais ce n'est pas ce que vous voulez une fois que le serveur est en production, puisque nous allons le faire fonctionner derrière un reverse proxy pour HTTPS. | Lors de la configuration, laissez 0.0.0.0 mais changez pour 127.0.0.1 une fois que tout est configuré.
log/level | Changez pour info une fois que tout est en cours d'exécution | info
authentication_backend | Activez les sous-clés **file** et **file/path** pour faciliter l'utilisation dans un premier temps. Ci-dessous, vous trouverez ce qu'il faut stocker dans ce chemin d'accès | Changez path en `/etc/authelia/users_db.yml` 
access_control | Cette partie est un peu complexe, donc regardez ci-dessous pour une explication | Voir ci-dessous
access_control/rule/.../policy | N'importe lequel de deny (rejette la requête), bypass (relai transparent de la connexion), one_factor (demande de login & mots de passe), two_factor (demande de login & mot de passe + preuve de possession d'un appareil) | Voir ci-dessous pour un exemple
session/domain | Typiquement, définir le domaine de base | myhost.com
session/secret | Une autre chaîne aléatoire qui ne peut pas être devinée par un attaquant | Générez-en une en tapant `head /dev/random | tr -dc A-Za-z0-9 | head -c64 > session_secret.txt` et collez-la ici entre guillemets.
storage | Activez cette option, car elle est nécessaire pour stocker les données de session. |  
storage/encryption_key | Une autre chaîne aléatoire qui ne peut pas être devinée par un attaquant | Générez-en une en tapant `head /dev/random | tr -dc A-Za-z0-9 | head -c64 > session_secret.txt` et collez-la ici entre guillemets
notifier | Vous devez définir au moins un notificateur pour que Authelia puisse communiquer avec vous et ses utilisateurs |  

### Redis

Installer redis

    sudo apt install redis

Fichier de configuration

```yaml
session:

  redis:
    host: localhost
    port: 6379
    #password: authelia
    database_index: 0
    maximum_active_connections: 8
    minimum_idle_connections: 0
```

### Alternative gestion utilisateurs

#### A - LLDAP (UTILISE)

* [LLdap - Serveur virtuel d'authentification (VM+Docker)](/posts/Light_LDAP_simple_serveur_authentification/)

```yaml
  ldap:
    implementation: custom
    address: ldap://127.0.0.1:3890
    timeout: 5s
    start_tls: false
    base_dn: dc=rnmkcy,dc=eu
    additional_users_dn: ou=people
    users_filter: "(&({username_attribute}={input})(objectClass=person))"
    additional_groups_dn: ou=groups
    groups_filter: "(member={dn})"
    user: uid=admin,ou=people,dc=rnmkcy,dc=eu
    password: 'xxxxxxxxxxxxxxxx'
    attributes:
      mail: 'mail'
      username: 'uid'
      group_name: 'cn'
      display_name: 'displayName'
```

#### B - Fichier

<u>Fichier yml des utilisateurs</u>  
*Créer un fichier utilisateur ou utiliser un serveur LDAP.* 

La base de données des utilisateurs est un fichier appelé `/etc/authelia/users_db.yml` 

```yaml
users:
  yannick:
    displayname: "Yannick"
    # Générer avec: authelia hash-password <your-password>, et copier depuis le '$' jusqu'à la fin du  digest.
    password: "$argon2id$v=19$m=65536,t=3,p=4$xpIAeb39WEW/PfHVkTFlXA$OjNhaBViFckNiOaKkl2qqdBQGV6G7GP2SshX5GCpiks"
    email: yannick@cinay.eu
    groups:
      - admins
```

### Alternative storage

#### Sqlite

Installer sqlite3

    sudo apt install sqlite3

Fichier de configuration

```yaml
storage:
  encryption_key: XXXXXXXXXXXXXXXXXXXX
  local:
    path: /etc/authelia/db.sqlite3
```

#### Mysql

<u>Créer une base mysql</u>  
Host : 127.0.0.1  
Base : authelia  
Utilisateur : authelia  
Mot de passe : Mot_de_passe_base_authelia

En ligne de commande

```shell
sudo mysql -e "create database authelia; grant all privileges on authelia.* to authelia@'localhost' identified by 'Mot_de_passe_base_authelia'; flush privileges;"
```
 La configuration yml

```yaml
storage:
  encryption_key: "zQMZFuTB36PYCMJeqPTYsAnb3BwSOO"
  mysql:
    address: '127.0.0.1:3306'
    database: authelia
    username: authelia
    password: "Mot_de_passe_base_authelia"
```

### Générateur secret base 64

La génération des des secrets se fait en ligne de commande

```shell
# Commande 
head /dev/random | tr -dc A-Za-z0-9 | head -c64 
# OU
tr -cd '[:alnum:]' < /dev/urandom | fold -w 64 | head -n 1 | tr -d '\n' ; echo
```

## Authelia configuration.yml

### Configuration de base

Modifier le fichier configuration `/etc/authelia/configuration.yml`

```
access_control:
  default_policy: deny
  rules:
    ## bypass rule
    - domain: 
        - "auth.rnmkcy.eu"
      policy: bypass
    ## catch-all
    - domain:
        - "*.rnmkcy.eu"
      subject: 
        - "group:admins"
      policy: one_factor
    - domain:
        - "dice.rnmkcy.eu"
      subject: 
        - "user:usera"
      policy: two_factor

```


WebAuthn Authelia

```
webauthn:
  disable: false
  display_name: Authelia
  attestation_conveyance_preference: indirect
  user_verification: preferred
  timeout: 60s
```

user_verification :

* **discouraged** : Le client sera découragé de demander une vérification à l'utilisateur.
* **preferred** : Le client, s'il est conforme, demandera à l'utilisateur une vérification si l'appareil la prend en charge.
* **required** : Le client demandera à l'utilisateur une vérification ou échouera si l'appareil ne prend pas en charge la vérification.

attestation_conveyance_preference:

* **none** : Le client recevra l'instruction de ne pas effectuer de transfert de propriété.
* **indirect** : Le client reçoit l'instruction d'effectuer le transfert de propriété, mais il peut choisir la manière de le faire, y compris en utilisant une AC d'anonymisation tierce.
* **direct** : Le client recevra l'instruction d'effectuer le transfert avec une déclaration d'attestation directement signée par le dispositif.

Exemple

```yaml
##
## WebAuthn Configuration
##
## Paramètres utilisés pour WebAuthn.
webauthn :
  ## Désactiver Webauthn.
  disable : false

  ## Ajuster le délai d'interaction pour les dialogues Webauthn.
  timeout : 60s

  ## Le nom d'affichage que le navigateur doit montrer à l'utilisateur lorsqu'il utilise Webauthn pour se connecter ou s'enregistrer.
  display_name : Authelia

  ## La préférence de transmission contrôle si nous collectons la déclaration d'attestation, y compris l'AAGUID, à partir de l'appareil.
  ## Les options sont none, indirect, direct.
  attestation_conveyance_preference : indirect

  ## User verification contrôle si l'utilisateur doit faire un geste ou une action pour confirmer sa présence.
  ## Les options sont : required, preferred, discouraged.
  user_verification : preferred
```

### Utilisateurs

Ajouter 2 utilisateurs dans un groupe "users", fichier `/etc/authelia/users_db.yml`

usera usera49600  
userb userb49600 

```yaml
  usera:
    displayname: "usera"
    password: "$argon2id$v=19$m=65536,t=3,p=4$wdL22Uvrbb/A9bbnk5xStQ$zilbUyV/E9BDXiz0mC0L3C7jalqQiPvJ8mpXB8y7Wl8"
    email: usera@rnmkcy.eu
    groups:
      - users
  userb:
    displayname: "userb"
    password: "$argon2id$v=19$m=65536,t=3,p=4$Smi2zDT3UkMGqXOWTZq9DQ$agEy2AvRiKE1HThdXkEba2Ca/rHWcgxA81yfne+zias"
    email: userb@rnmkcy.eu
    groups:
      - users
```

[Firefox, comment activer l'U2F/Fido2 mais aussi comment supporter le passwordless](https://forum.linuxos.ovh/d/182-activer-u2f-sur-firefox)

Normalement, le protocole exige que votre clé de sécurité soit enregistrée sur chaque site avant de pouvoir s'y authentifier. Comme Authelia propose le Single Sign-On, vos utilisateurs n'auront besoin d'enregistrer leur appareil qu'une seule fois pour avoir accès à toutes vos applications.


### Configuration serveur cwwk

*Configuration authelia sur serveur cwwk rnmkcy.eu 192.168.0.205*

Créer le fichier de configuration authelia `/etc/authelia/configuration.yml` en se basant sur l'[exemple ici](https://docs.ibracorp.io/authelia/configuration-files/configuration.yml)

* Serveur: 127.0.0.1:9091
* Gestion utilisateurs: LLDAP
* Authentification: TOTP + Webauthnn 
* Stockage: Base mysql
* Notification: SMTP

<details>
<summary>Etendre Réduire <b>configuration.yml</b></summary>
{% highlight yaml %} 
###############################################################################
#                           Authelia Configuration                            #
###############################################################################

theme: dark
##
## Identity Validation Configuration
##
## This configuration tunes the identity validation flows.
identity_validation:
  ## Reset Password flow. Adjusts how the reset password flow operates.
  reset_password:
    jwt_secret: "IkVHLPgMDV62YJ40lOUqe1ySxSDAvLV7ljWwbav9JrMO1E84gYLxotG0j1EjuVV4"

server:
  #host: 127.0.0.1
  #port: 9091
  #path: ""
  address: 'tcp://127.0.0.1:9091/'
  disable_healthcheck: false
  tls:
    key: ""
    certificate: ""
  ## Server Endpoints configuration.
  ## This section is considered advanced and it SHOULD NOT be configured unless you've read the relevant documentation.
  endpoints:
    ## Enables the pprof endpoint.
    enable_pprof: false
    ## Enables the expvars endpoint.
    enable_expvars: false

log:
  level: info
  file_path: '/etc/authelia/authelia.log'

totp:
  issuer: rnmkcy.eu
  period: 30
  skew: 1

##
## WebAuthn Configuration
##
webauthn:
  disable: false
  enable_passkey_login: false
  display_name: 'Authelia'
  attestation_conveyance_preference: 'indirect'
  timeout: '60 seconds'
  filtering:
    permitted_aaguids: []
    prohibited_aaguids: []
    prohibit_backup_eligibility: false
  selection_criteria:
    attachment: ''
    discoverability: 'preferred'
    user_verification: 'preferred'
  metadata:
    enabled: false
    validate_trust_anchor: true
    validate_entry: true
    validate_entry_permit_zero_aaguid: false
    validate_status: true
    validate_status_permitted: []
    validate_status_prohibited:
      - 'REVOKED'
      - 'USER_KEY_PHYSICAL_COMPROMISE'
      - 'USER_KEY_REMOTE_COMPROMISE'
      - 'USER_VERIFICATION_BYPASS'
      - 'ATTESTATION_KEY_COMPROMISE'

authentication_backend:
  password_reset:
    disable: false
  refresh_interval: 1m
  ldap:
    implementation: custom
    address: ldap://127.0.0.1:3890
    timeout: 5s
    start_tls: false
    base_dn: dc=rnmkcy,dc=eu
    additional_users_dn: ou=people
    users_filter: "(&({username_attribute}={input})(objectClass=person))"
    additional_groups_dn: ou=groups
    groups_filter: "(member={dn})"
    user: uid=admin,ou=people,dc=rnmkcy,dc=eu
    password: '35UKayRwjGijBiVANELn'
    attributes:
      mail: 'mail'
      username: 'uid'
      group_name: 'cn'
      display_name: 'displayName'

access_control:
  default_policy: deny
  rules:
    ## bypass rule
    - domain:
        - "auth.rnmkcy.eu"
        - "cloud.rnmkcy.eu"
        - "searx.rnmkcy.eu"
      policy: bypass
    ## catch-all
    - domain:
        - "calibre.rnmkcy.eu"
        - "immich.rnmkcy.eu"
      policy: one_factor
    - domain:
        - "site.rnmkcy.eu"
        - "cockpit.rnmkcy.eu"
      policy: two_factor

session:
  secret: 'dRBndWsA4G6tR8dkWfdocDrJa13yP0Wxze4GMONWkUGaNM3bsARi6sms7UzKzuKV'
  name: 'authelia_session'
  same_site: 'lax'
  inactivity: '45m'
  expiration: '12h'
  remember_me: '2M'
  cookies:
    - domain: 'rnmkcy.eu'
      authelia_url: 'https://auth.rnmkcy.eu'
      default_redirection_url: 'https://rnmkcy.eu'
      name: 'authelia_session'
      same_site: 'lax'
      inactivity: '45m'
      expiration: '12h'
      remember_me: '1d'
  redis:
    host: localhost
    port: 6379
    #password: ""
    database_index: 0
    maximum_active_connections: 10
    minimum_idle_connections: 0

regulation:
  max_retries: 3
  find_time: 10m
  ban_time: 12h

storage:
  encryption_key: "zQkAOYfMvvEmvrkd9bGYCMJeqMZFuTB36PYn8ItXQ7oehqzTL1lPTYsAnb3BwSOO"
  mysql:
    address: '127.0.0.1:3306'
    database: authelia
    username: authelia
    password: "DistyleKyrieParieJouffluCostauxCalme"

identity_providers:
  oidc:
    jwks:
      - key_id: 'authelia'
        algorithm: 'RS256'
        use: 'sig'
        certificate_chain: |
          -----BEGIN CERTIFICATE-----
          MIIDFzCCAf+gAwIBAgIQBEcLOQcy50Hf4c6PC8PIGDANBgkqhkiG9w0BAQsFADAsMREwDwYDVQQKEwhBdXRoZWxpYTEXMBUGA1UEAxMOYXV0aC5ybm1rY3kuZXUwHhcNMjUwNjEwMTYxMzQ2WhcNMjYwNjEwMTYxMzQ2WjAsMREwDwYDVQQKEwhBdXRoZWxpYTEXMBUGA1UEAxMOYXV0aC5ybm1rY3kuZXUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCwOgcozZHcQcI+UalNWU8Ir48q0HmdHxyWCAVRlnMUqfLWcqiMqAf4hs3qXemASjTAwNmtVyueX1eanIm9iFBa47y5130Buf3/itm8Q40CV0XF0hBJRB/6qOIUYZydcZkfJrPC5zPAJw6sPJ97B1+oOJLc8XPAx2y/Ce0Fpk8HJUZ1cbZY+YgS1VFm/6aGacjy0/YUc+/YvydzpLgOWDye/XfsxIXvMuHtVep3z7StrUagNYVmO5f1E3mYGlWvYcJLxIxgOwGmF+UclV/7s0cA2Z74dmATb2fEDVGktD/40FXX5b5wksgnzcEHgzT07mlxzsIUOA7x+pM+aFz116VnAgMBAAGjNTAzMA4GA1UdDwEB/wQEAwIFoDATBgNVHSUEDDAKBggrBgEFBQcDATAMBgNVHRMBAf8EAjAAMA0GCSqGSIb3DQEBCwUAA4IBAQBIvmqQMRj/Z30Qn6M+XpG4LN9byqkZKJpmuFpGx276hweSm0oFdEMznoJTSVZRQIFUooW8T5kxzusi/T807YyYzhD3f+oe3IMnkOealzkR/+29CeAx5Ps+osaEfRMQsZGfuWdBpxs11hQYZ8JpBM80QOdxU/bflWahiI7+PhRqdwrKxKI2ONxXdzBDBKA9VEb5mtYn02Qjb3d/Kys5rGuksuvQMQ4545wMNfcueLuc3DAG24vEewop4a/vE5QnL0MeUdDRufQq6RTYWlY7WQQW+5aHasATnmT45mkFoYqYrvVcYDa0A4ZzOymxFIE49PUtCq1BKM/S5FzLjZBfZWOV
          -----END CERTIFICATE-----
        key: |
          -----BEGIN PRIVATE KEY-----
          MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCwOgcozZHcQcI+UalNWU8Ir48q0HmdHxyWCAVRlnMUqfLWcqiMqAf4hs3qXemASjTAwNmtVyueX1eanIm9iFBa47y5130Buf3/itm8Q40CV0XF0hBJRB/6qOIUYZydcZkfJrPC5zPAJw6sPJ97B1+oOJLc8XPAx2y/Ce0Fpk8HJUZ1cbZY+YgS1VFm/6aGacjy0/YUc+/YvydzpLgOWDye/XfsxIXvMuHtVep3z7StrUagNYVmO5f1E3mYGlWvYcJLxIxgOwGmF+UclV/7s0cA2Z74dmATb2fEDVGktD/40FXX5b5wksgnzcEHgzT07mlxzsIUOA7x+pM+aFz116VnAgMBAAECggEAAw9e5HR9Xee+abqdR/yPBMWXpNSt5+6MXoaRIRONuAQjdVnUV2zrhoNAvBsZXOLs+5Pds8bZuQxKxyKEucfOLWXa9xco0ojaQj+0/DHoFuxRB9hX9A28yGt6UYRIZet9Dy9q262MFoAXHyKGWmqMXK1uz2tZ6TUNTFvtw7VZlfQmXdXT4BpM79omcMicWVIqXeeKJCfTMVZVy2aXqpCGcIZ7I4aFCzRdIN47gjx1v+01+xI7xw7YCB1kHU6I11zyJd6l6lHbjpRcULxB1MPyFPi8SnMlcUtbNWLtIQRDS/F6m+vH7zLv1dHSVc7lPM7b+KewitM2HpNtInGOV/qCQQKBgQDa4Gbr7Q4adbE0m64GlUSVTenIbVftM55LpTQqbYg+1qGAOnrXDl3IXDilDZgld/i10SGdIkMpoNlD9XCYsjWwrbrLGLNj8eIj5E7pd6vH3hA4yBwnO6KprBCQWB5hsWdHGWAKKw/qRf3eLB50ebFX71iRZoPprIRmdcaHbmAiFwKBgQDOHcWRv5BP4qIFo312HZImlL0tRXxwKYI8JaK0D3KiNtSenMTUu598RTkUuA8SDBjqulVnTNQxYA4RCRjVTqd0ah5/0w23s1QlwOt0CUDlD0GxDrxTvRNEAByVOHnnOxaZc7/Gf1qXho6dNDMGvAEOBQMxdmt/TTZaQQ2iyzY5MQKBgQDDVNalICag+H2PvIufNmFBeYn0o5h/iLxxN9sJLj0UD0mL/SSZfim14I9wxWQNtDaSVDPSXrxfjzu2SXkV3JzLsw9rpvyZakltzJecBX+ZXBVQB04QuUjecT9LAkkC2GaZ7gxmpq2KXrJEL01ylxPsrR52jG2XAa4bHmpxfNBwOwKBgQCS8UKir5Ewgn/DyVRuWpKREsTze3BFfjIssyAVglXVM4XNNjJU/xWqQKd7D+UelqVE4mZ0aJ92O+aLWpLrr1FOe0EXG93RsvUwnSK+7MHpC0Fn+vtkL42GkxXNhB/UK5nm+FAmMTDiibkudcJS8MFHv2cRYj0s2lz/ZxVTk9AqYQKBgDbMd8MWactlpcDeT7QihzMXE2smq6aZCjuU+p5tlQY8njqu6deOSsO5NQHWzGLRazRzErQg5UOhNLLkosyIteRpwGWw8WWMyxflodIp8BIS0gaGvsFc4WgMz24aE6COIVxmgPTKL+PsRXf5nzhqB/W+NcVEmBU7kWWv5DfHJsqA
          -----END PRIVATE KEY-----
    ## The other portions of the mandatory OpenID Connect 1.0 configuration go here.
    ## See: https://www.authelia.com/c/oidc
    clients:
      - client_id: 'immich'
        client_name: 'immich'
        client_secret: '$pbkdf2-sha512$310000$XbCUhR7hmJGtsByOD03LJA$ZaQCQ8AU1l65nQGlXIHJoXEuwiHGhMm66PFjCrTXg3GZJ.Q.lDITZEIKuSox6Def2GNLpMEPePpNPSVqQLrGMQ'  # The digest of 'insecure_secret'.
        public: false
        authorization_policy: 'two_factor'
        redirect_uris:
          - 'https://immich.rnmkcy.eu/auth/login'
          - 'https://immich.rnmkcy.eu/user-settings'
          - 'app.immich:///oauth-callback'
        scopes:
          - 'openid'
          - 'profile'
          - 'email'
        userinfo_signed_response_alg: 'none'
        token_endpoint_auth_method: 'client_secret_post'

#notifier:
#  disable_startup_check: false
#
#  filesystem:
#    filename: /etc/authelia/notification.txt

notifier:
  disable_startup_check: false

  smtp:
    username: "yanfi@yanfi.net"
    ## Password can also be set using a secret: https://www.authelia.com/docs/configuration/secrets.html
    password: "RocadeLoureSuccubeFrugaux"
    address: "submission://mx1.xoyize.xyz:587"
    sender: "yanfi@yanfi.net"
    ## Subject configuration of the emails sent. {title} is replaced by the text from the notifier.
    subject: "[Authelia] {title}"

{% endhighlight %}
</details>

Remarque sur la configuration smtp :  
Les messages envoyés sont au nom de l'utilisateur connecté, dans notre cas pour "yann", ils seront envoyés à yann@yanfi.net  
Il faut créer un alias dans le serveur de messagerie yanfi.net : yann@yanfi.net --> yanfi@yanfi.net
{: .prompt-info }

Vérifier la configuration en mode su

    sudo authelia validate-config --config /etc/authelia/configuration.yml

Vous devez obtenir le message suivant, si tout est correct  
`Configuration parsed and loaded successfully without errors.`

Lancer, activer et vérifier

    sudo systemctl enable authelia --now
    sudo cat /etc/authelia/authelia.log 

```
time="2024-12-05T08:34:11+01:00" level=info msg="Storage schema is being checked for updates"
time="2024-12-05T08:34:11+01:00" level=info msg="Storage schema is already up to date"
time="2024-12-05T08:34:11+01:00" level=info msg="Startup complete"
time="2024-12-05T08:34:11+01:00" level=info msg="Listening for non-TLS connections on '127.0.0.1:9091' path '/'" server=main service=server
```

### Préparer votre application pour le SSO

*Authelia affiche une page de connexion/authentification, il doit être exécuté sur un canal de transport https*

L'application doit être adaptée pour traiter avec un serveur d'authentification externe.  
Lorsque **Authelia a validé l'utilisateu**r, il redirige vers votre application.   
Cette redirection contient des en-têtes spécifiques que votre application doit traiter pour procéder à sa création de session habituelle (typiquement, votre application émettra son `Set-Cookie` ou une requête de stockage local ou un jeton JWT ou ...).  
Il s'agit simplement de **contourner la page de connexion**.


#### Authelia proxy nginx

* [Authelia proxy nginx](https://www.authelia.com/integration/proxies/nginx/)

Création dossier **snippets**

    sudo mkdir -p /etc/nginx/snippets

Les fichiers de configuration nginx 

> Le premier extrait est une route interne `/authelia` qui n'est pas accessible depuis l'internet, qui est utilisée dans vos applications pour affirmer que la session Authelia est valide.
{: .prompt-tip }

/etc/nginx/snippets/authelia-location.conf

```nginx
# Virtual endpoint created by nginx to forward auth requests.
location /authelia {
    internal;
    set $upstream_authelia http://127.0.0.1:9091/api/verify;
    proxy_pass_request_body off;
    proxy_pass $upstream_authelia;    
    proxy_set_header Content-Length "";

    # Timeout if the real server is dead
    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;

    # [REQUIRED] Needed by Authelia to check authorizations of the resource.
    # Provide either X-Original-URL and X-Forwarded-Proto or
    # X-Forwarded-Proto, X-Forwarded-Host and X-Forwarded-Uri or both.
    # Those headers will be used by Authelia to deduce the target url of the     user.
    # Basic Proxy Config
    client_body_buffer_size 128k;
    proxy_set_header Host $host;
    proxy_set_header X-Original-URL $scheme://$http_host$request_uri;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr; 
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Forwarded-Uri $request_uri;
    proxy_set_header X-Forwarded-Ssl on;
    proxy_redirect  http://  $scheme://;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    proxy_cache_bypass $cookie_session;
    proxy_no_cache $cookie_session;
    proxy_buffers 4 32k;

    # Advanced Proxy Config
    send_timeout 5m;
    proxy_read_timeout 240;
    proxy_send_timeout 240;
    proxy_connect_timeout 240;
}
```

> Vous aurez aussi besoin d'un snippet pour l'authentification (interne) elle-même, donc sauvegardez ce snippet sous `/etc/nginx/snippets/authelia-authrequest.conf`:  
Typiquement, ce snippets fait en sorte que NGINX génère une requête vers l'emplacement interne `/authelia` à chaque fois qu'une requête d'utilisateur est faite vers votre application. Il vérifie le code de résultat HTTP de la requête interne. Si c'est 200, la requête de l'utilisateur est acceptée, sinon il est redirigé vers la page de login Authelia.
{: .prompt-tip }


/etc/nginx/snippets/authelia-authrequest.conf

```nginx
# Basic Authelia Config
# Send a subsequent request to Authelia to verify if the user is authenticated
# and has the right permissions to access the resource.
auth_request /authelia;
# Set the `target_url` variable based on the request. It will be used to build the portal
# URL with the correct redirection parameter.
auth_request_set $target_url $scheme://$http_host$request_uri;
# Set the X-Forwarded-User and X-Forwarded-Groups with the headers
# returned by Authelia for the backends which can consume them.
# This is not safe, as the backend must make sure that they come from the
# proxy. In the future, it's gonna be safe to just use OAuth.
auth_request_set $user $upstream_http_remote_user;
auth_request_set $groups $upstream_http_remote_groups;
auth_request_set $name $upstream_http_remote_name;
auth_request_set $email $upstream_http_remote_email;
proxy_set_header Remote-User $user;
proxy_set_header Remote-Groups $groups;
proxy_set_header Remote-Name $name;
proxy_set_header Remote-Email $email;
# If Authelia returns 401, then nginx redirects the user to the login portal.
# If it returns 200, then the request pass through to the backend.
# For other type of errors, nginx will handle them as usual.
error_page 401 =302 https://auth.domain.eu/?rd=$target_url;
```


### Exemple de protection

* [Un exemple de SSO avec Authelia et Owntone](https://blog.cyril.by/fr/software/single-sign-on/example-sso-with-authelia-and-owntone)
* [Security key](https://www.authelia.com/overview/authentication/security-key/)

> Enfin, dans la description de votre site d'application, vous devrez inclure ces extraits. Le fichier  `authelia.conf` va dans le bloc serveur et le fichier `authelia-authrequest.conf` va dans n'importe quel endroit que vous voulez protéger.  
Vous éviterez typiquement de protéger les actifs statiques et les redirections, puisqu'il n'y a pas d'intérêt à vérifier l'authentification pour ceux-ci. 
{: .prompt-tip }


Un exemple d'application pourrait ressembler à ceci :

```nginx
server {
    server_name app.myhome.com;
    listen 80;
    return 301 https://$server_name$request_uri;
}

server {
    server_name app.myhome.com;
    listen 443 ssl http2;
    include snippet/ssl.conf;

    include snippets/authelia.conf; # Authelia auth endpoint

    location / {
        proxy_pass http://proxied:service;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        include snippets/authelia-authrequest.conf; # Protect this endpoint
    }
}
```

## Authentification 

### Yubikeys

Configuration authelia prérequise

```yaml
## WebAuthn Actif
##
webauthn:
  ## Désactiver Webauthn avec true.
  disable: false

# l'application web utilisant la double authentification
access_control:
  default_policy: deny
  rules:
    - domain:
        - "cockpit.rnmkcy.eu"
      policy: two_factor

# Pour la transmission des messages par smtp
notifier:
  smtp:
    username: 'yako@xoyize.xyz'
    # Password can also be set using a secret: https://www.authelia.com/configuration/methods/secrets/
    password: 'xxxxxxxxxxxxxxxxx'
    sender: 'yako@xoyize.xyz'
    address: 'submission://xoyize.xyz:587'

```

Clé yubikey insérée dans un port USB

Se connecter sur le lien https://auth.rnmkcy.eu avec Firefox  
Saisir utilisateur "yann" et son mot de passe  

![](authelia01.png){:width="300"}  
Choisir la méthode  

![](authelia02.png){:width="300"}  

![](authelia03.png){:width="300"}  
![](authelia10.png){:width="400"}  

Un message est envoyé  
![](authelia11.png){:width="300"}  

Sur la boîte de réception  
![](authelia12.png){:width="600"}  

Saisir le code ID  
![](authelia13.png){:width="400"}  
Une nouvelle fenêtre s'ouvre  
![](authelia14.png){:width="300"}  
![](authelia15.png){:width="300"}  

La clé est enregistrée  
![](authelia16.png){:width="300"}  
et un message de confirmation  
![](authelia17.png){:width="600"}  

Se connecter de nouveau à cockpit

Page de connexion auhelia  
![](authelia06.png){:width="500"}  
Validation par clé  
![](authelia07.png){:width="500"}  
Page authentification cockpit  
![](authelia08.png){:width="500"}  

Ajouter une autre clé yubikey  
La seconde clé est enregistrée  
![](authelia18.png){:width="600"}  
et un message de confirmation  
![](authelia19.png){:width="600"}  

### Totp

On va ajouter une authentification TOTP  
Se connecter sur authelia et cliquer sur authelia  
![](authelia20.png){:width="600"}  
Mot de passe usage unique
![](authelia21.png){:width="600"}  

Sur la boîte de réception  
![](authelia12.png){:width="600"}  

Saisir le code ID  
![](authelia13.png){:width="400"}  
Suivre la procédure  
![](authelia22.png){:width="400"}  
Si vous n"avez pas google authenticator  
Prendre une photo avec l'appareil android pour décoder   
![](authelia23.png){:width="400"}  
Cliquer sur le lien pour copier le code   

Copier le contenu de secret dans le totp de cockpit keepassxc  
![](authelia24.png){:width="200"}  

Cliquer sur suivant et décocher qrcode  
![](authelia25.png){:width="300"}  
Vérification fonctionnement avec un code généré par cockpi keepassxc   
![](authelia26.png){:width="400"}  
Si tout est OK   
![](authelia27.png){:width="500"}  
et un message de confirmation  
![](authelia28.png){:width="600"}  

## Authelia Nextcloud OpenID

### Versions testées

Authelia v4.38.0  
Nextcloud 22.1.0

### Notes communes

1. Le paramètre OpenID Connect 1.0 `client_id` :
    1.        Cela doit être une valeur unique pour chaque client.
    *        La valeur utilisée dans ce guide est uniquement à des fins de lisibilité et de démonstration et vous ne devez pas utiliser cette valeur. Nous recommandons 64 caractères aléatoires, mais vous pouvez utiliser n'importe quelle valeur arbitraire répondant aux autres critères.
    *        Celui-ci ne doit contenir que des caractères non réservés RFC3986 ([RFC3986 Unreserved Characters](https://datatracker.ietf.org/doc//rfc3986#section-2.3)) .
    *        Celui-ci ne doit pas comporter plus de 100 caractères.
*    Le paramètre OpenID Connect 1.0 `client_secret` :
     1.   La valeur utilisée dans ce guide est uniquement à des fins de démonstration et vous ne devez absolument pas l'utiliser en production et devez plutôt utiliser la FAQ Comment générer des secrets client ([How Do I Generate Client Secrets](https://www.authelia.com/integration/openid-connect/frequently-asked-questions/#how-do-i-generate-client-secrets)) .
     *  Cette chaîne peut être stockée sous forme de texte brut dans la configuration Authelia, mais ce comportement est obsolète et il n'est pas garanti qu'il soit pris en charge à l'avenir. Consultez le guide Texte brut ([Plaintext](https://www.authelia.com/integration/openid-connect/frequently-asked-questions/#plaintext)) pour plus d’informations.
     *  Lorsque le secret est stocké sous forme hachée dans la configuration authelia ( fortement recommandé ), le coût du hachage peut, s'il est trop élevé, entraîner des timeouts pour les clients. Consultez le guide Réglage des facteurs de travail pour plus d’informations ()[Tuning the work factors](https://www.authelia.com/integration/openid-connect/frequently-asked-questions/#tuning-work-factors).
* L'exemple de configuration pour Authelia ne représente qu'une partie de la configuration requise et il doit être utilisé comme guide en conjonction avec les guides  [OpenID Connect 1.0 Provider Configuration](https://www.authelia.com/configuration/identity-providers/openid-connect/provider/) et [OpenID Connect 1.0 Clients Configuration](https://www.authelia.com/configuration/identity-providers/openid-connect/clients/)

### Hypothèses

Cet exemple fait les hypothèses suivantes :

*    URL racine de l'application : https://cloud.rnmkcy.eu/
*    URL racine d'Authelia : https://auth.rnmkcy.eu/
*    Identité du client: nextcloud
*    Secret client : insecure_secret

Remarque importante : il a été signalé que certains plugins Nextcloud n'encodent pas correctement le secret client. en tant que tel, il est important de n'utiliser que des caractères alphanumériques ainsi que les autres caractères non réservés de la RFC3986 . Nous vous recommandons d'utiliser les instructions de génération de secrets client ci-dessus.

### Générer jwks

Pour utiliser OpenID (oidc),Authelia demande l'option jwks  

Pour générer ou acquérir les valeurs nécessaires à la section jwks pour OpenID Connect (OIDC) dans Authelia, vous devez créer un ensemble de clés Web JSON (JWKs). Voici les étapes pour générer ces clés

**1 - Générer une clé privée** : Vous pouvez utiliser un outil comme OpenSSL pour générer une clé privée RSA.

    openssl genrsa -out private.key 2048

**2 - Créer le JWK** : convertir la clé privée RSA au format JWK. Vous pouvez utiliser un outil comme `node-jose-tools` ou un convertisseur en ligne.  
[node-jose-tools for JOSE key management, token signing and encryption](https://github.com/phish108/node-jose-tools?tab=readme-ov-file#node-jose-tools-for-jose-key-management-token-signing-and-encryption) : Installation : `npm install -g node-jose-tools`

Si vous utilisez node-jose-tools

    jose newkey -s 256 -t oct -a HS256

```json
{
"kty":"oct",
"kid":"ORYf0WHq5XAcpwHFQ-OQ4Oo265REyyc7p6EsWRB7D6s",
"alg":"HS256",
"k":"zqsIYRYQ5Zn1kIRSWTqTkG-t2B7xvke7t9pbdFQr4dA"
}
```

En ligne : <https://russelldavies.github.io/jwk-creator/>



**3 - Extraire la clé publique** de la clé privée à l'aide d'OpenSSL :

    openssl rsa -in private.key -pubout -out public.key


### Configuration Authelia

La configuration YAML suivante est un exemple de configuration de client Authelia à utiliser avec Nextcloud qui fonctionnera avec l'exemple ci-dessus :

```yaml
identity_providers:
  oidc:
    ## The other portions of the mandatory OpenID Connect 1.0 configuration go here.
    ## See: https://www.authelia.com/c/oidc
    clients:
      - client_id: 'nextcloud'
        client_name: 'NextCloud'
        client_secret: '$pbkdf2-sha512$310000$c8p78n7pUMln0jzvd4aK4Q$JNRBzwAo0ek5qKn50cFzzvE9RXV88h1wJn5KGiHrD0YKtZaR/nCb2CJPOsKaPK0hjf.9yHxzQGZziziccp6Yng'  # The digest of 'insecure_secret'.
        public: false
        authorization_policy: 'two_factor'
        redirect_uris:
          - 'https://cloud.rnmkcy.eu/apps/oidc_login/oidc'
        scopes:
          - 'openid'
          - 'profile'
          - 'email'
          - 'groups'
        userinfo_signed_response_alg: 'none'
```

### Application

Pour configurer Nextcloud pour utiliser Authelia en tant que fournisseur OpenID Connect 1.0 :

1.    Installez l' application **OpenID Connect Login** ([Nextcloud OpenID Connect Login app](https://apps.nextcloud.com/apps/oidc_login))
*    Ajoutez les éléments suivants à la configuration Nextcloud config.php :

```
$CONFIG = array (
    'allow_user_to_change_display_name' => false,
    'lost_password_link' => 'disabled',
    'oidc_login_provider_url' => 'https://auth.rnmkcy.eu',
    'oidc_login_client_id' => 'nextcloud',
    'oidc_login_client_secret' => 'insecure_secret',
    'oidc_login_auto_redirect' => false,
    'oidc_login_end_session_redirect' => false,
    'oidc_login_button_text' => 'Log in with Authelia',
    'oidc_login_hide_password_form' => false,
    'oidc_login_use_id_token' => true,
    'oidc_login_attributes' => array (
        'id' => 'preferred_username',
        'name' => 'name',
        'mail' => 'email',
        'groups' => 'groups',
    ),
    'oidc_login_default_group' => 'oidc',
    'oidc_login_use_external_storage' => false,
    'oidc_login_scope' => 'openid profile email groups',
    'oidc_login_proxy_ldap' => false,
    'oidc_login_disable_registration' => true,
    'oidc_login_redir_fallback' => false,
    'oidc_login_alt_login_page' => 'assets/login.php',
    'oidc_login_tls_verify' => true,
    'oidc_create_groups' => false,
    'oidc_login_webdav_enabled' => false,
    'oidc_login_password_authentication' => false,
    'oidc_login_public_key_caching_time' => 86400,
    'oidc_login_min_time_between_jwks_requests' => 10,
    'oidc_login_well_known_caching_time' => 86400,
    'oidc_login_update_avatar' => false,
);
```

### Voir également

* [Nextcloud OpenID Connect Login app](https://apps.nextcloud.com/apps/oidc_login)
* [Nextcloud OpenID Connect Login Documentation](https://github.com/pulsejet/nextcloud-oidc-login)

### Générer identifiant ou secret

Authelia fournit un moyen simple d'effectuer de telles actions.

#### ID client

Les utilisateurs peuvent facilement générer un identifiant/identifiant client en suivant le guide [Génération d'une chaîne alphanumérique aléatoire](https://www.authelia.com/reference/guides/generating-secure-values/#generating-a-random-alphanumeric-string)  
Par exemple, les utilisateurs peuvent exécuter la commande  

    authelia crypto rand --length 72 --charset rfc3986

pour générer un identifiant/identifiant client de 72 caractères qui est imprimé.  
Cette commande aléatoire évite également les problèmes avec une application client/partie de confiance codant correctement les caractères car elle utilise les caractères non réservés RFC3986 .

Si un jeu de caractères différent est utilisé et que la valeur est différente lors du codage de l'URL, il imprimera également cette valeur séparément.

#### secret client

Les utilisateurs peuvent facilement générer un secret client en suivant le guide [Génération d'un hachage de mot de passe aléatoire](https://www.authelia.com/reference/guides/generating-secure-values/#generating-a-random-password-hash)  
Par exemple, les utilisateurs peuvent exécuter la commande 

    authelia crypto hash generate pbkdf2 --variant sha512 --random --random.length 72 --random.charset rfc3986

pour à la fois générer un secret client de 72 caractères qui est imprimé et doit être utilisé avec la partie utilisatrice et le hacher à l'aide de PBKDF2 qui peut être stocké dans la configuration Authelia. Cette commande aléatoire évite également les problèmes avec une application client/partie de confiance codant correctement les caractères car elle utilise les caractères non réservés RFC3986 .

Si un jeu de caractères différent est utilisé et que la valeur est différente lors du codage de l'URL, il imprimera également cette valeur séparément.


#### Réglage des facteurs de travail

Lors du hachage des secrets du client, Authelia effectue l'opération de hachage pour authentifier le client lors de la réception des demandes. Cette opération de hachage prend du temps, de par sa conception (la partie travail du facteur travail) pour empêcher un attaquant de tenter d'obtenir le secret client. Le temps nécessaire dépend de votre matériel et du facteur de travail.

Si les opérations de votre client expirent, vous devrez peut-être réduire le facteur de travail à un niveau approprié pour votre client et les capacités de votre matériel.

Pour tester la durée de différents facteurs de travail, vous pouvez la mesurer ainsi : 

    time authelia crypto hash generate pbkdf2 --variant sha512 --iterations 310000 --password insecure_password 

Remarque : Vous ne devez pas utiliser vos mots de passe réels pour ce test. Le temps nécessaire doit être le même pour toute longueur de mot de passe raisonnable.

Vous pouvez en savoir plus sur le réglage du hachage de mot de passe dans le guide de référence des mots de passe .

