+++
title = 'Ntfy service de notification'
date = 2025-01-30 00:00:00 +0100
categories = ['divers']
+++
*[Ntfy](https://ntfy.sh/), qui se prononce “notify”, est un service de notification ultra léger, permettant d'envoyer des messages vers un smartphone ou un ordinateur via de simples scripts, sans besoin de compte et totalement gratuitement !*

* [Publishing](https://docs.ntfy.sh/publish/)
* [Emoji reference](https://docs.ntfy.sh/emojis/)
* [UTF-8 emojis in bash or zsh](https://gist.github.com/BuonOmo/77b75349c517defb01ef1097e72227af)
* [Exemples](https://docs.ntfy.sh/examples/)

## Ntfy Installation

Le CLI de ntfy vous permet de publier des messages, de vous abonner à des sujets et d'héberger vous-même votre propre serveur ntfy. C'est très simple. Il suffit d'installer le binaire, le paquet ou l'image Docker, de le configurer et de l'exécuter.

Les étapes suivantes ne sont nécessaires que si vous voulez héberger votre propre serveur ntfy ou si vous voulez utiliser le CLI ntfy. Si vous voulez juste envoyer des messages en utilisant ntfy.sh, vous n'avez pas besoin d'installer quoi que ce soit. Vous pouvez simplement utiliser curl.
{: .prompt-info }

Le **serveur ntfy** se présente sous la forme d'un binaire lié statiquement et est livré sous forme de paquetage tarball, deb/rpm et sous forme d'image Docker. Nous supportons amd64, armv7 et arm64.  
Veuillez consulter la [page des versions](https://github.com/binwiederhier/ntfy/releases) pour les binaires et les paquets deb/rpm.

### Debian 

manuellement 

```shell
wget https://github.com/binwiederhier/ntfy/releases/download/v2.8.0/ntfy_2.8.0_linux_amd64.deb
sudo dpkg -i ntfy_*.deb
sudo systemctl enable ntfy
```

Avec le dépôt

```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://archive.heckel.io/apt/pubkey.txt | sudo gpg --dearmor -o /etc/apt/keyrings/archive.heckel.io.gpg
sudo apt install apt-transport-https
sudo sh -c "echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/archive.heckel.io.gpg] https://archive.heckel.io/apt debian main' \
    > /etc/apt/sources.list.d/archive.heckel.io.list"  
sudo apt update
sudo apt install ntfy
sudo systemctl enable ntfy
sudo systemctl start ntfy
```

### Arch Linux

2 méthodes

**Méthode 1**  
Vous pouvez récupérer le fichier binaire ntfy depuis le [dépôt GitHub](https://github.com/binwiederhier/ntfy/releases) 

Créer un utilisateur système nommé "ntfy"  
Utilisez l’option -r (--system) pour créer un compte d’utilisateur système

    sudo useradd -r ntfy

Télécharger la dernière version, décompresser, copier binaire dans /usr/local/bin et modifier les droits

```bash
# Téléchargement
wget https://github.com/binwiederhier/ntfy/releases/download/v2.11.0/ntfy_2.11.0_linux_amd64.tar.gz
# Décompresser
tar xzvf ntfy_2.11.0_linux_amd64.tar.gz
# copier binaire
sudo cp ntfy_2.11.0_linux_amd64/ntfy /usr/local/bin/
# modifier les droits
sudo chown ntfy:ntfy /usr/local/bin/ntfy
```

**Méthode 2**  
installer le paquet 

    yay -S --noconfirm ntfysh-bin

Création groupe et utilisateur à l'installation

```
Creating group 'ntfy' with GID 951.
Creating user 'ntfy' (ntfy user) with UID 951 and GID 951.
```

## Configuration du serveur ntfy

Le serveur ntfy peut être configuré de trois manières : 

1. Par défaut, en utilisant un fichier de configuration (typiquement dans `/etc/ntfy/server.yml`) 
2. via des arguments de ligne de commande 
3. ou en utilisant des variables d'environnement.

### ntfy derrière un proxy

Si vous exécutez ntfy derrière un proxy, vous devez activer le drapeau behind-proxy. Cela demandera à la logique de limitation de taux d'utiliser l'en-tête X-Forwarded-For comme identifiant principal d'un visiteur, plutôt que l'adresse IP distante. Si l'indicateur "behind-proxy" n'est pas activé, tous les visiteurs seront comptés comme un seul, car du point de vue du serveur ntfy, ils partagent tous l'adresse IP du proxy.

Fichier **/etc/ntfy/server.yml**, les modifications

```yaml
listen-http: "127.0.0.1:8100"

behind-proxy: true
```

Démarrer le service

    sudo systemctl start ntfy

### Proxy Nginx

Le fichier proxy `/etc/nginx/conf.d/noti.rnmkcy.eu.conf` 

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name  noti.rnmkcy.eu;

    # redirect all plain HTTP requests to HTTPS
    return 301 https://noti.rnmkcy.eu$request_uri;
}

server {
    # ipv4 listening port/protocol
    listen       443 ssl http2;
    # ipv6 listening port/protocol
    listen           [::]:443 ssl http2;
    server_name  noti.rnmkcy.eu;

    include /etc/nginx/conf.d/security.conf.inc;

  location / { 
    proxy_pass              http://127.0.0.1:8100;
    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_connect_timeout 3m;
    proxy_send_timeout 3m;
    proxy_read_timeout 3m;

    client_max_body_size 0; # Stream request body to backend
  } 

}
```

Vérifier et recharger nginx

    sudo nginx -t && sudo systemctl reload nginx

Première connexion depuis un navigateur firefox <https://noti.rnmkcy.eu>   
![](ntfy01.png)


Après avoir installé le serveur ntfy, accessibilité lien <https://noti.rnmkcy.eu>

## Instance publique

### Application android ntfy

Installer l'application android ntfy  
Pour s'abonner à un sujet, il suffit de lancer l'application et de cliquer sur le bouton `+`  
![](ntfy06.png){:width="200"}  

Nous écrivons ensuite le nom du sujet auquel nous voulons nous abonner (le nom est complètement arbitraire), et, pour utiliser notre instance Ntfy auto-hébergée, nous cochons « Utiliser un serveur différent » et entrons l'IP de notre serveur ; enfin, nous cliquons sur « ABONNER »  
![](ntfy07.png){:width="200"}  

Pour envoyer une notification au sujet, il suffit d'envoyer une requête POST ou PUT au serveur, en utilisant le langage de programmation de notre choix ou notre outil de ligne de commande préféré (commande `ntfy pub` ou une requête POST via curl). 

    curl -d "Ceci est une notification ntfy" https://noti.rnmkcy.eu/notif_infos
    ntfy pub notif_infos "Ceci est une notification ntfy"

La notification push devrait apparaître sur notre appareil client android  
![](ntfy08.png){:width="200"}  

Affichage des messages au format JSON

```json
{"topic":"monsujet","message":"Ceci est une notification ntfy","time":1622656800}
```

[Exemples d'utilisation](https://docs.ntfy.sh/examples/#cronjobs)  
[Recevoir une alerte à chaque connexion SSH en utilisant ntfy](https://www.pofilo.fr/post/2023/09/10-alerte-connexion-ssh/)

## Instance ntfy privée

### Configurer server.yml

La façon la plus simple de configurer une instance privée est de définir `auth-default-access: deny-all` dans le fichier `/etc/ntfy/server.yml` :

```yaml
auth-file: /var/lib/ntfy/user.db
auth-default-access: "deny-all"
```

Le fichier de configuration `/etc/ntfy/server.yml`

```yaml
listen-http: "127.0.0.1:8100"
auth-file: /var/lib/ntfy/user.db
auth-default-access: "deny-all"
behind-proxy: true
# web-root: "disable"
```

Explications

* Listen-http - Le port sur lequel le serveur Web HTTP écoute (par défaut = 80). Vous pouvez également ajouter une adresse IP à laquelle vous connecter (`listen-http : "127.0.0.1:8100"`)
* **auth-file** - Emplacement du user.dbfichier d'authentification.
* **auth-default-access** - Définir ceci sur **"deny-all"** forcera tous les sujets à être privés par défaut.
* **behind-proxy** - Si ntfy est déployé derrière un proxy comme Caddy, nginx, etc., alors cela doit être défini sur true.
* **web-root** - Emplacement du répertoire racine pour ntfy. Le réglage sur « désactiver » désactive le client Web.

Redémarrer le serveur ntfy

    sudo systemctl restart ntfy

### Créer utilisateur 

Passer en mode su : `sudo -s`

**Créer utilisateur dans NTFY** ([Users and roles](https://docs.ntfy.sh/config/#users-and-roles))  
Puisque nous avons déjà défini l' attribut `auth-file` et `auth-default-access`, nous pouvons passer à l'ajout de notre premier utilisateur administrateur "notif".

    ntfy user add --role=admin notif

L'appel de cette commande avec un nom d'utilisateur vous invitera automatiquement à créer un mot de passe pour le nouvel utilisateur.  
Saisir un mot de passe pour "notif"  
`user leno added with role admin`

Une fois l'utilisateur créé, nous pouvons l'abonner à notre premier sujet de notification.

    ntfy access notif notif_infos rw

Explications 

* **rw** spécifie le niveau d'accès de l'utilisateur **notif** pour le sujet **notif_infos**. Puisque cet utilisateur sera utilisé pour envoyer des notifications, nous devons lui donner la possibilité d'écrire sur le sujet.  
* **wo** "write-only" (écriture seule)  
* **rw** "read-write" (lecture-écriture)  
* **ro** "read-only" (lecture seule)  

Vous pouvez modifier les autorisations d'accès à tout moment, en réexécutant la commande ci-dessus avec un niveau d'accès différent. 

Créez maintenant un deuxième utilisateur **notifmob** avec les autorisations sur le sujet en lecture seul.  
Ce deuxième utilisateur sera utilisé pour notre application mobile.  

    ntfy user add notifmob
    ntfy access notifmob notif_infos ro

Liste des commandes pour la gestion des utilisateurs

```
sudo ntfy user list                     # Shows list of users (alias: 'ntfy access')
sudo ntfy user add phil                 # Add regular user phil  
sudo ntfy user add --role=admin phil    # Add admin user phil
sudo ntfy user del phil                 # Delete user phil
sudo ntfy user change-pass phil         # Change password for user phil
sudo ntfy user change-role phil admin   # Make user phil an admin
sudo ntfy user change-tier phil pro     # Change phil's tier to "pro"
```

Une fois que vous avez fait cela, vous pouvez publier et vous abonner en utilisant l'authentification de base avec le nom d'utilisateur/mot de passe donné. Veillez à utiliser HTTPS pour éviter les écoutes et l'exposition de votre mot de passe. 

## Authentification

[Doc ntfy](https://docs.ntfy.sh)  

### A - Utilisateur et mot de passe

[Username + password](https://docs.ntfy.sh/publish/#username-password)  
Lorsque l'agent utilisateur souhaite envoyer des informations d'authentification au serveur, il peut utiliser le champ d'en-tête **Authorization** 

Le champ d'en-tête **Authorization** est construit comme suit : 

*    Le nom d'utilisateur et le mot de passe sont combinés avec un seul deux-points ( `:`). Cela signifie que le nom d'utilisateur lui-même ne peut pas contenir de deux points.
*    La chaîne résultante est encodée en une séquence d'octets. Le jeu de caractères à utiliser pour cet encodage est par défaut non spécifié, tant qu'il est compatible avec US-ASCII, mais le serveur peut suggérer l'utilisation d'UTF-8 en envoyant le paramètre charset.
*    La chaîne résultante est encodée à l'aide d'une variante de Base64 (+/ et avec remplissage).
*    La méthode d'autorisation et un caractère d'espacement (par exemple "Basic ") sont ensuite ajoutés à la chaîne encodée.

Par exemple, si le navigateur utilise *Aladdin* comme nom d'utilisateur et *open sesame* comme mot de passe, la valeur du champ est le codage Base64 de *Aladdin:open sesame*, ou *QWxhZGRpbjpvcGVuIHNlc2FtZQ==*. Le champ d'en-tête Authorization se présente alors comme suit :

    Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ== 

Pour générer l'en-tête d'autorisation, utilisez la norme base64 pour encoder les `<nom d'utilisateur>:<mot de passe>` séparés par deux points et ajoutez-y le mot Basic, c'est-à-dire Authorization : Basic base64(`<username>:<password>`).  
Voici un pseudo-code qui, je l'espère, l'explique mieux :

```
username = "testuser"
password = "fakepassword"
authHeader = "Basic " + base64(username + " :" + password) // -> Basic dGVzdHVzZXI6ZmFrZXBhc3N3b3Jk
```

La commande suivante génère la valeur appropriée pour vous sur les systèmes *nix :

    echo "Basic $(echo -n 'testuser:fakepassword' | base64)"

INFO: Entête autorisation au format **utilisateur:Mot_de_passe** base 64   
`echo -n 'leno:mot de passe' | base64`  
Renvoie --> `eWFubjptb3QgZGUgcGFzc2U=`  

#### curl

```shell
curl \
    -u leno:mypass \
    -d "accès avec authentification" \
    https://noti.rnmkcy.eu/yan_infos
```

#### POST http

```shell
curl \
 -i -s -X POST -H "Authorization: Basic bGVubzpRpbjpvsZW5vNDk=" \
 -d "POST/HTTP accès avec authentification" "https://noti.rnmkcy.eu/yan_infos"
```

#### JavaScript

```javascript
fetch('https://ntfy.example.com/mysecrets', {
    method: 'POST', // PUT works too
    body: 'Look ma, with auth',
    headers: {
        'Authorization': 'Basic cGhpbDpteXBhc3M='
    }
})
```

#### Go

```go
req, _ := http.NewRequest("POST", "https://ntfy.example.com/mysecrets",
    strings.NewReader("Look ma, with auth"))
req.Header.Set("Authorization", "Basic cGhpbDpteXBhc3M=")
http.DefaultClient.Do(req)
```

#### Python

```python
requests.post("https://ntfy.example.com/mysecrets",
    data="Look ma, with auth",
    headers={
        "Authorization": "Basic cGhpbDpteXBhc3M="
    })
```

#### PHP

```php
file_get_contents('https://ntfy.example.com/mysecrets', false, stream_context_create([
    'http' => [
        'method' => 'POST', // PUT also works
        'header' => 
            'Content-Type: text/plain\r\n' .
            'Authorization: Basic cGhpbDpteXBhc3M=',
        'content' => 'Look ma, with auth'
    ]
]));
```

#### ntfy CLI

```
ntfy publish \
    -u phil:mypass \
    ntfy.example.com/mysecrets \
    "Look ma, with auth"
```

### B - Jetons (Tokens)

*Le jeton peut être utilisé à la place du nom d'utilisateur et du mot de passe lors de l'envoi de messages. ([Access tokens](https://docs.ntfy.sh/config/#access-tokens))*

En plus de l'authentification par nom d'utilisateur/mot de passe, ntfy fournit également une authentification par jetons d'accès. Les jetons d'accès sont utiles pour éviter d'avoir à configurer votre mot de passe dans plusieurs applications de publication/abonnement. Par exemple, vous pouvez utiliser un jeton dédié pour publier à partir de votre hôte de sauvegarde, et un autre à partir de votre système domotique.

Vous pouvez créer des jetons d'accès en utilisant la commande `ntfy token`, ou dans l'application web dans la section "Compte" (lorsque vous êtes connecté). Voir les [jetons d'accès](https://docs.ntfy.sh/config/#access-tokens) pour plus de détails.

Une fois le jeton d'accès créé, vous pouvez l'utiliser pour vous authentifier auprès du serveur ntfy, par exemple lorsque vous publiez ou vous abonnez à des sujets. 

La commande ntfy token permet de gérer les jetons d'accès des utilisateurs. Les jetons peuvent avoir des étiquettes, et ils peuvent expirer automatiquement (ou ne jamais expirer). Chaque utilisateur peut avoir jusqu'à 20 jetons (codés en dur).

Exemples de commandes (tapez ntfy token --help ou ntfy token COMMAND --help pour plus de détails) 

```
ntfy token list # Affiche la liste des jetons pour tous les utilisateurs
ntfy token list phil # Affiche la liste des jetons de l'utilisateur phil
ntfy token add phil # Crée un jeton pour l'utilisateur phil qui n'expire jamais
ntfy token add --expires=2d phil # Crée un jeton pour l'utilisateur phil qui expire dans 2 jours
ntfy token remove phil tk_th2sxr...  # Supprimer le jeton
```

Création d'un jeton d'accès :

```shell
ntfy token add notif
ntfy token add --expires=30d --label="backups" notif
ntfy token list
utilisateur notif
- tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2 (sauvegardes), expire le 15 mars 23 14:33 EDT, accédé depuis 0.0.0.0 le 13 février 23 13:33 EST
```

Une fois le jeton d'accès créé, vous pouvez l'utiliser pour vous authentifier auprès du serveur ntfy, par exemple lorsque vous publiez ou vous abonnez à des sujets.

Voici un exemple utilisant Bearer auth, avec le jeton tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2 :

#### curl

```shell
curl \
  -H "Authorization: Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2" \
  -d "Look ma, with auth" \
  https://ntfy.example.com/mysecrets
```

#### POST http

```shell
POST /mysecrets HTTP/1.1
Host: ntfy.example.com
Authorization: Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2

Look ma, with auth
```

#### JavaScript

```javascript
fetch('https://ntfy.example.com/mysecrets', {
    method: 'POST', // PUT works too
    body: 'Look ma, with auth',
    headers: {
        'Authorization': 'Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2'
    }
})
```

#### Go

```go
req, _ := http.NewRequest("POST", "https://ntfy.example.com/mysecrets",
strings.NewReader("Look ma, with auth"))
req.Header.Set("Authorization", "Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2")
http.DefaultClient.Do(req)
```

#### Python

```python
requests.post("https://ntfy.example.com/mysecrets",
data="Look ma, with auth",
headers={
    "Authorization": "Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2"
})
```

#### PHP

```php
file_get_contents('https://ntfy.example.com/mysecrets', false, stream_context_create([
    'http' => [
        'method' => 'POST', // PUT also works
        'header' =>
            'Content-Type: text/plain\r\n' .
            'Authorization: Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2',
        'content' => 'Look ma, with auth'
    ]
]));
```

#### ntfy CLI

```
ntfy publish \
  --token tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2 \
  ntfy.example.com/mysecrets \
  "Look ma, with auth"
```

## Android ntfy (PRIVE)

### Installer android ntfy

Installer l'application ntfy disponible sur F-Droid

Après ouverture de l'application   
![](ntfy02.png){:width="200" .normal} ![](ntfy03.png){:width="200" .normal} 

![](ntfy03a.png){:width="200" .normal}  ![](ntfy04.png){:width="200" .normal}  

`Il faut saisir utilisateur et mot de passe car notification privée`{: .prompt-info }

### Configurer application mobile ntfy 

**Configuration de l'application mobile Android**  
Téléchargez d’abord l’application depuis [f-droid](https://f-droid.org/en/packages/io.heckel.ntfy/) , et après son installation, lancez-la.  
![](ntfy10.png){:width="200"}  
Au premier lancement, vous serez accueilli par un écran vide sans aucun sujet. Pour configurer l'application pour votre instance auto-hébergée, ouvrez le menu des paramètres (menu à 3 points > paramètres)  
![](ntfy11.png){:width="200"}  
et accédez à « Gérer les utilisateurs » sous Général.   
![](ntfy12.png){:width="200"}  
Appuyez sur « Ajouter un nouvel utilisateur » et dans le Service URLchamp, entrez l'URL que vous avez utilisée pour votre instance ntfy ci-dessus . Les champs Nom d'utilisateur et Mot de passe seront le deuxième utilisateur que vous avez créé précédemment  
Une fois les champs remplis, appuyez sur « AJOUTER UN UTILISATEUR ».   
![](ntfy13.png){:width="200"}  
Votre section utilisateur devrait alors ressembler à l’image suivante  
![](ntfy14.png){:width="200"}  
Appuyez sur la flèche de retour (ou faites glisser votre doigt vers l'arrière), entrez le paramètre **Serveur par défaut** et saisissez à nouveau votre URL ntfy.  
![](ntfy15.png){:width="200"}  
Maintenant, appuyez ou faites glisser votre doigt pour revenir à l'écran d'accueil. Pour désactiver la notification de livraison instantanée, qui s'affiche dans votre tiroir de notification, faites glisser votre doigt vers le sélecteur d'application et appuyez longuement sur l'icône de l'application ntfy ![](ntfy16.png){:width="25" .normal}  
Appuyez ensuite sur « Informations sur l'application ».  
![](ntfy17.png){:width="200"}  
Sur la page des paramètres de l'application, appuyez sur **Notifications** et désactivez la notification « Service d'abonnement » sous « Autre » ???.  
![](ntfy18.png){:width="200"}  

**S'abonner à un sujet**  
`ATTENTION :Pour s'abonner à un sujet, il faut que le sujet existe.`  
Sur l'écran d'accueil de l'application, appuyez sur le bouton vert « + » et saisissez le nom du sujet que vous avez créé précédemment lors de l' étape d'accès utilisateur . Appuyez ensuite sur « S'ABONNER ».  
![](ntfy19.png){:width="200"}  
Si cela est fait correctement, votre nouveau sujet devrait apparaître sur l'écran d'accueil de votre application.

Les paramètres de l'abonnement peuvent être ajustés en appuyant dessus puis en appuyant sur le menu à 3 points. Appuyez ensuite sur « Paramètres d'abonnement » dans le menu. Ici, vous pouvez modifier les préférences de notification ainsi que l'icône et le nom d'affichage de l'abonnement.

## ntfy - Ligne de commande

### Sécuriser 

Pour éviter que n'importe qui puisse envoyer ou recevoir des messages, vous pouvez utiliser un mot de passe.

    ntfy publish -u admin:myp@ssw0rd ntfy.example.com/monsujet "This is a message"
    curl -u admin:myp@ssw0rd -d "This is a message" https://ntfy.example.com/monsujet

Cela implique un hébergement du serveur ntfy


### Envoi 

    ntfy publish -u utilisateur:mot_de_passe  ntfysrv.eu/test "Nouveau test depuis PC1 en ligne de commande" |jq

Affichage message expédié

```json
{
  "id": "WqRBXcEmOoqv",
  "time": 1708692353,
  "expires": 1708735553,
  "event": "message",
  "topic": "test",
  "message": "Nouveau test depuis PC1 en ligne de commande"
}
```

### Réception 

Se mettre en attente d'un message

```shell
# Mot de passe sur serveur privé
ntfy sub -u utilisateur:mot_de_passe  ntfysrv.eu/test |jq
# Clé token tk_xxxxxxxxxxx
ntfy sub -s all --token tk_xxxxxxxxxxx https://ntfysrv.eu/test |jq
```
{: .nolineno }

Affichage à la réception d'un message

```json
{
  "id": "BY9zD5GQqCI9",
  "time": 1708693062,
  "expires": 1708736262,
  "event": "message",
  "topic": "test",
  "title": "Firefox PC1",
  "message": "Nouvel essai depuis le navigateur"
}
```

Utilitaire jq pour un affichage au format json

### Réception avec notify-send

Pour afficher vos notifications directement sur votre environnement de bureau, vous pouvez utiliser la commande `ntfy sub` avec notify-send.

```shell
# Mot de passe sur serveur privé
ntfy sub -u utilisateur:mot_de_passe  ntfysrv.eu/test 'notify-send -t 0 "ntfy" "$m"'
# Clé token tk_xxxxxxxxxxx
ntfy sub --token tk_xxxxxxxxxxx https://ntfysrv.eu/test 'notify-send -t 0 "ntfy" "$m"'
```
{: .nolineno }

![](ntfy05.png)

Voir tous les anciens messages

Si vous souhaitez voir l'historique des messages, vous pouvez utiliser le paramètre `-s all`

    ntfy sub -s all monsujet

### Notification token + curl

L'utilisateur **notif** a les droits en lecture/écriture et possède une clé token qui englobe utilisateur/mot de passe

Ligne de commande

    curl -H "Title: Test envoi notification" -u :token_utilisateur_notif https://noti.rnmkcy.eu/notif_infos -d "ceci est un message de test"

Différentes publications [Publishing](https://docs.ntfy.sh/publish/) 

```shell

curl \
 -H "Title: Alerte!" \
 -H "Authorization: Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2" \
 -H "X-Tags: warning,mailsrv13,daily-backup" \
 -d "Envoi en cas d'erreur!" \
 https://noti.rnmkcy.eu/notif_infos

curl \
  -H "Authorization: Bearer tk_AgQdq7mVBoFD37zQVN29RhuMzNIz2" \
  -d "Look ma, with auth" \
  https://ntfy.example.com/mysecrets
```

### Alerte sur connexion SSH

**Alerte en cas de connexion SSH**  

Le fichier `/etc/pam.d/sshd`

```
# à la fin du fichier
session optional pam_exec.so /usr/bin/ntfy-ssh-login.sh
```

Le fichier `/usr/bin/ntfy-ssh-login.sh`

```bash
#!/bin/bash
if [ "${PAM_TYPE}" = "open_session" ]; then
  curl \
    -H "Title: Alerte connexion SSH!" \
    -H "Authorization: Bearer tk_cpifjh59xo3zu2bgi2jyfkif4wsbf" \
    -H prio:high \
    -H tags:warning \
    -d "SSH ${HOSTNAME} user: ${PAM_USER} from ${PAM_RHOST}" \
    https://noti.rnmkcy.eu/notif_infos
fi
```

Le rendre exécutable : `chmod +x /usr/bin/ntfy-ssh-login.sh`

Message en cas de connexion  
![](ntfy20.png){:width="200"}  

## Web Push (INACTIF)

### Description

Web Push (RFC8030) permet à ntfy de recevoir des notifications push, même lorsque l'application web ntfy (ou même le navigateur, selon la plateforme) est fermée. Une fois activé, l'utilisateur peut activer les notifications de fond pour leurs sujets dans l'application Web sous Paramètres. Une fois activé par l'utilisateur, ntfy transmettra les messages publiés au paramètre push, qui les transmettra ensuite au navigateur.

Pour configurer **Web Push**, vous devez générer et configurer un keypair [VAPID](https://datatracker.ietf.org/doc/html/draft-thomson-webpush-vapid) (via `ntfy webpush keys`), une base de données pour suivre les abonnements du navigateur et une adresse email admin (vous):

* `Web-push-public-key` est la clé publique VAPID générée, exemple:   **AA1234BBCCddvveekaabcdfqwertyuiopasdfghjklzxcvbnm1234567890**  
* `Web-push-private-key` est la clé privée VAPID générée, exemple:   **AA2BB1234567890abcdefzxcvbnm1234567890**
* `web-push-file` est un fichier de base de données pour suivre les paramètres d'abonnement du navigateur,  exemple:  
**/var/lib/ntfy/webpush.db**
* `Web-push-email-address` est l'adresse email d'administration envoyée au fournisseur push, exemple:   
**sysadmin@exemple.com**
* `web-push-startup-queries` est une liste optionnelle de requêtes à exécuter au démarrage 

Limites:

* Comme les notifications de navigateur de premier plan, les notifications de poussée de fond exigent que l'application web soit desservie par HTTPS. Un certificat valide est requis, car les travailleurs de services ne fonctionneront pas sur des origines avec des certificats non fiables.
* Web Push n'est supporté que pour le même serveur. Vous ne pouvez pas utiliser s'abonner à web push sur un sujet sur un autre serveur. Ceci est dû à une limitation de l'API Push, qui n'autorise pas plusieurs serveurs push pour la même origine.

### Mise en place

Sur le serveur ntfy

Pour configurer les clés VAPID, les générer d'abord : `ntfy webpush keys`  
Ensuite copiez les valeurs générées dans le fichier `serveur.yml` ou utilisez les variables d'environnement correspondantes ou les arguments en ligne de commande  

```
web-push-public-key: AA1234BBCCddvveekaabcdfqwertyuiopasdfghjklzxcvbnm1234567890
web-push-private-key: AA2BB1234567890abcdefzxcvbnm1234567890
web-push-file: /var/cache/ntfy/webpush.db
web-push-email-address: sysadmin@example.com
```

* Le **web-push-file** est utilisé pour stocker les abonnements push. Les abonnements inutilisés envoient un avertissement après 7 jours et expirent automatiquement après 9 jours (non configurable). Si la passerelle renvoie une erreur (par exemple 410 Gone quand un utilisateur s'est désabonné), les abonnements sont également supprimés automatiquement.
* L'application web rafraîchit les abonnements au début et régulièrement sur un intervalle, mais ce fichier doit être maintenu à travers les redémarrages. Si le fichier d'abonnement est supprimé ou perdu, les applications web qui ne sont pas ouvertes ne recevront pas de nouvelles notifications de poussée web avant d'ouvrir.
* <u>Il n'est pas recommandé de changer votre paire de clés publique/privée</u>. Les navigateurs n'autorisent qu'une seule identité de serveur (clé publique) par origine, et si vous les modifiez, les clients ne pourront pas s'abonner via la poussée web jusqu'à ce que l'utilisateur efface manuellement la permission de notification.

## ntfy - Application bureau

### Utilisation application web progressive (PWA)

Bien que **ntfy** n'ait pas d'application de bureau native, il est construit comme une application web progressive (PWA) et peut donc être installé sur les appareils de bureau et mobiles.

Cela lui donne son propre lanceur (par exemple raccourci sur Windows, application sur macOS, raccourci de lancement sur Linux, icône d'écran d'accueil sur iOS et icône de lancement sur Android), une fenêtre autonome, des notifications de poussée et un badge d'application avec le compte de notification non lu.

L'installation de l'application Web est prise en charge sur (voir tableau de compatibilité pour plus de détails):

* Chrome: Android, Windows, Linux, macOS
* Safari : iOS 16.4+, macOS 14+
* Edge : Windows
* Firefox: Android, ainsi que sur Windows/Linux via une [extension](https://addons.mozilla.org/en-US/firefox/addon/pwas-for-firefox/)  

Notez que pour les serveurs auto-organisés, **Web Push** doit être configuré pour que le PWA fonctionne.

### Installation

**Chrome sur le bureau**  
Pour installer et enregistrer l'application web via Chrome, cliquez sur l'icône "install app" . Après l'installation, vous pouvez trouver l'application dans votre tiroir :  
![](ntfy-chrome01.png){:width="350" .normal} 
![](ntfy-chrome02.png){:width="350".normal} 
![](ntfy-chrome03.png){:width="350".normal}

**Chrome/Firefox sur Android**  
Pour Chrome sur Android, soit cliquez sur la bannière "Ajouter à l'écran d'accueil" au bas de l'écran, ou sélectionnez "Installer l'application" dans le menu, puis cliquez sur "Installer" dans le menu contextuel. Après l'installation, vous pouvez trouver l'application dans votre tiroir et sur votre écran d'accueil.
![](ntfy-firefox-android01.png){:width="150" .normal} ![](ntfy-firefox-android01.png){:width="150" .normal} ![](ntfy-firefox-android01.png){:width="150" .normal}

Pour Firefox, sélectionnez "Installer" dans le menu, puis cliquez sur "Ajouter" pour ajouter une icône à votre écran d'accueil:  
![](ntfy-firefox-android04.png){:width="150" .normal} ![](ntfy-firefox-android05.png){:width="150" .normal}

**Firefox archlinux bureau**  
via une [extension](https://addons.mozilla.org/en-US/firefox/addon/pwas-for-firefox/)  
![](ntfy-firefox-pwa01.png){:width="250" .normal}  
Setup Progressive Web Apps for Firefox --> Accept agreement  
![](ntfy-firefox-pwa01a.png){:width="250" .normal}  
![](ntfy-firefox-pwa02.png){:width="250" .normal}  
Installer le paquet archlinux : `sudo pacman -S firefoxpwa`  
![](ntfy-firefox-pwa03.png){:width="250" .normal}  
![](ntfy-firefox-pwa04.png){:width="250" .normal}


### Notifications arrière-plan

Les notifications de fond via le web push sont activées par défaut et ne peuvent pas être désactivées lorsque l'application est installée, car les notifications ne seraient pas livrées de manière fiable autrement. Vous pouvez muter des sujets pour lesquels vous ne voulez pas recevoir de notifications.

Sur le bureau, vous avez généralement besoin de votre navigateur ou de l'application Web ouverte pour recevoir des notifications, bien que l'onglet ntfy n'ait pas besoin d'être ouvert. Sur mobile, vous n'avez pas besoin d'avoir l'application Web ouverte pour recevoir des notifications. Regardez les [documents web](https://docs.ntfy.sh/subscribe/web/#background-notifications) pour une ventilation détaillée.