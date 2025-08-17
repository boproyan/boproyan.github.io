+++
title = 'Portail Yunohost'
date = 2020-07-25 00:00:00 +0100
categories = ['yunohost']
+++
![](yunohost.png){:width="60"}

## Thèmes

[Yunohost Themes](https://github.com/yunohost-themes)


### Créer un thème

Vous pouvez créer votre propre thème en copiant un thème existant 

```bash
sudo cp -r /usr/share/ssowat/portal/assets/themes/{clouds,yann}
```

Ensuite, éditez les fichiers CSS et JS dans `/usr/share/ssowat/portal/assets/themes/votre_theme` selon ce que vous voulez faire : 

- `custom_portal.css` peut être utilisé pour ajouter des règles CSS personnalisées au portail utilisateur ;
- `custom_overlay.css` peut être utilisé pour personnaliser le petit bouton YunoHost, présent sur les apps qui l'intègrent ;
- `custom_portal.js` peut être utilisé pour ajouter du code JS personnalisé à exécuter à la fois sur le portail utilisateur ou lors de l'injection du petit bouton YunoHost ("overlay").

Vous pouvez également ajouter vos propres images et ressources qui peuvent ensuite être utilisées par les fichiers CSS et JS.

### Personnaliser le logo

Vous pouvez créer votre propre thème simplement pour changer le "branding" du portail utilisateur YunoHost et remplacer le logo YunoHost par votre propre logo !

Pour ce faire, téléversez votre logo dans `/usr/share/ssowat/portal/assets/themes/yann/` et ajoutez les règles CSS suivantes : 

```css
/* Dans custom_portal.css */
#ynh-logo {
  background-image : url("./votre_logo.png");
}

/* Dans custom_overlay.css */
#ynh-overlay-switch {
  background-image : url("./votre_logo.png");
}
```

### Utiliser un thème  

Depuis YunoHost 3.5, il est possible de changer le thème du portail utilisateur - bien que pour l'instant il faille encore faire cette opération via la ligne de commande.

Vous pouvez lister les thèmes disponibles avec : 

```bash
ls /usr/share/ssowat/portal/assets/themes/
```

Ensuite, vous pouvez utiliser `nano /etc/ssowat/conf.json.persistent` pour activer le thème que vous choisissez comme ceci :

```json
{
    "theme" : "yann",
    ...autres lignes.....
}
```

<div class="alert alert-info" markdown="1">
Vous devrez peut-être forcer le rafraîchissement du cache de votre navigateur pour que le thème se propage complètement. Vous pouvez le faire avec Ctrl+Maj+R sur Firefox.
</div>

### Ajouter un thème externe

Ajout du thème "Nature-Mount"

    git clone https://github.com/yunohost-themes/Nature-Mount.git
    sudo mv Nature-Mount/ /usr/share/ssowat/portal/assets/themes/

Activer le thème

    sudo nano /etc/ssowat/conf.json.persistent

```json
{
"theme" : "Nature-Mount"
}
