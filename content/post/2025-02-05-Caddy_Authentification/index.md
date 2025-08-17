+++
title = 'Caddy Authentification'
date = 2025-02-05 00:00:00 +0100
categories = ['caddy']
+++
## Authentification basique

*Lors de la visite d'un site Web protégé par l'authentification de base, le navigateur demandera à l'utilisateur de saisir un nom d'utilisateur et un mot de passe avant le chargement des ressources.*

* Le client enverra ensuite un en-tête d'autorisation avec chaque requête adressée au site Web, afin de maintenir l'authentification.
* Cette authentification doit être effectuée via une connexion HTTPS pour être sécurisée.

Vous trouverez plus d'informations sur l'authentification de base dans les [documents Mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication) 

### Générer un hachage de votre mot de passe

Entrez ceci dans le terminal pour générer un hachage de mot de passe (cela utilise l'algorithme de hachage bcrypt par défaut) :

```bash
caddy hash-password # mot de passe à saisir , Enter password:
```

Exemple de sortie :  
JDJhJDE0JElab2ZPM25zdU40bE5SSURlTHd3OHVBeVJvYTlMN3dMOEFMdFVCRzNYS1l5ODl6TlVyQllH

### Configurer Caddyfile

Ci-dessous se trouve un modèle, remplacez simplement le nom d'utilisateur et le hachage, puis placez-le dans votre Caddyfile.

```
basicauth * {
	bob JDJhJDE0JElab2ZPM25zdU40bE5SSURlTHd3OHVBeVJvYTlMN3dMOEFMdFVCRzNYS1l5ODl6TlVyQllH
}
```

Si vous souhaitez sécuriser un certain chemin, la syntaxe suivante peut être utilisée :

```
basicauth /homework/* {
	bob JDJhJDE0JElab2ZPM25zdU40bE5SSURlTHd3OHVBeVJvYTlMN3dMOEFMdFVCRzNYS1l5ODl6TlVyQllH
}
```

Exemple domaine xoyize.xyz avec **root xoyize.xyz** (`/var/caddy/www/xoyize.xyz`)  

```
xoyize.xyz {
    root xoyize.xyz
    basicauth * {
     xouser JDJhJDE0JElab2ZPM25zdU40bE5SSURlTHd3OHVBeVJvYTlMN3dMOEFMdFVCRzNYS1l5ODl6TlVyQllH
    }
    encode gzip
    file_server
}
```

### Recharger Caddy

Deux commandes au choix

```bash
caddy reload
# ou
sudo systemctl reload caddy
```
