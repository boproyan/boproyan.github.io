+++
title = 'SimplyTranslate-Web pour la traduction'
date = 2023-07-15 00:00:00 +0100
categories = ['outils']
+++
## Traduction

*[SimplyTranslate Web](https://codeberg.org/SimpleWeb/SimplyTranslate-Web) est un frontend qui peut regrouper des traducteurs comme Google Traduction, Libre Translate (qui est lui aussi un logiciel libre), Reverso, ICIBA et DeepL. Son interface est volontairement minimaliste Il est écrit en Python.[SimplyTranslate (Framalibre)](https://framalibre.org/content/simplytranslate)*

![](simplytranslate.png)  

### Installer SimplyTranslate Web

Installation application web pour la traduction

Prérequis

    sudo apt install python3-pip python3-msgpack virtualenv

Mise à jour pip

    python3 -m pip install --upgrade pip

Installer environnement virtuel

    pip3 install virtualenv

Cloner le dépôt 

    git clone https://gitea.xoyaz.xyz/yann/simplytranslatefr SimplyTranslate-Web
    cd ~/SimplyTranslate-Web

Créer un environnement pour l'application

    virtualenv SimplyTranslateDev

activer l'environnement virtuel  

    source SimplyTranslateDev/bin/activate

On arrive sur un prompt `(SimplyTranslateDev) lxcyan@lxcbullseye:/home/lxcyan/SimplyTranslate-Web$`

Mettre à jour pip dans l'environnement

    /home/lxcyan/SimplyTranslate-Web/SimplyTranslateDev/bin/python -m pip install --upgrade pip

Installer les dépendances

    pip install -r requirements.txt

### Test via SSH

Pour le test direct, exécuter 

    /home/lxcyan/SimplyTranslate-Web/SimplyTranslateDev/bin/hypercorn main:app --bind 127.0.0.1:5000

Test local sur un ordinateur qui a l'accès SSH

    ssh -L 9500:localhost:5000 bullsvm@192.168.0.210 -p 55210 -i /home/yann/.ssh/vm-bullseyes

Sur ce même ordinateur lancer le navigateur avec le lien <http://localhost:9500>  

Une fois que vous avez terminé de tester l'application, appuyez sur ctrl + c pour arrêter le processus et désactiver l'environnement virtuel.

    deactivate

### Créer service systemd

Créer un service pour s'assurer que notre application fonctionne juste après le démarrage du système.

    sudo nano /etc/systemd/system/simplytranslate.service

```
[Unit]
Description=hypercorn instance to serve SimplyTranslate
After=network.target

[Service]
WorkingDirectory=/home/lxcyan/SimplyTranslate-Web
ExecStart=/home/lxcyan/SimplyTranslate-Web/SimplyTranslateDev/bin/hypercorn \
          --bind 127.0.0.1:5000 \
          main:app

[Install]
WantedBy=multi-user.target
```

Le fichier d'unité, le fichier de configuration source ou les drop-ins de `simplytranslate.service` ont changé sur le disque. Exécutez 'sudo systemctl daemon-reload' pour recharger les unités.

    sudo systemctl start simplytranslate
    sudo systemctl status simplytranslate

```
● simplytranslate.service - hypercorn instance to serve SimplyTranslate
     Loaded: loaded (/etc/systemd/system/simplytranslate.service; disabled; vendor preset: enabled)
    Drop-In: /run/systemd/system/service.d
             └─zzz-lxc-service.conf
     Active: active (running) since Fri 2023-02-24 17:19:41 CET; 1min 48s ago
   Main PID: 4178 (hypercorn)
      Tasks: 3 (limit: 38341)
     Memory: 128.8M
        CPU: 1.050s
     CGroup: /system.slice/simplytranslate.service
             ├─4178 /home/lxcyan/SimplyTranslate-Web/SimplyTranslateDev/bin/python /home/lxcyan/SimplyTranslate-Web/SimplyTranslateDev/bin/hypercorn --bind 127.0.0.1:5000 main:app
             ├─4179 /home/lxcyan/SimplyTranslate-Web/SimplyTranslateDev/bin/python -c from multiprocessing.resource_tracker import main;main(7)
             └─4180 /home/lxcyan/SimplyTranslate-Web/SimplyTranslateDev/bin/python -c from multiprocessing.spawn import spawn_main; spawn_main(tracker_fd=8, pipe_handle=10) --multiprocessing-fork

févr. 24 17:19:41 lxcbullseye systemd[1]: Started hypercorn instance to serve SimplyTranslate.
févr. 24 17:19:42 lxcbullseye hypercorn[4180]: [2023-02-24 17:19:42 +0100] [4180] [INFO] Running on http://127.0.0.1:5000 (CTRL + C to quit)
```

Activer le service

    sudo systemctl enable simplytranslate

### Configuration

Le fichier `~/SimplyTranslate-Web/config.conf`

```ini
[libre]
# LibreTranslate is disabled by default. If it is enabled, `Instance` is required.
Enabled = False
Instance = https://libretranslate.com
# Not all instances need an API key; if the one you use don't, remove this
# line.
# ApiKey = [REDACTED]

[google]
# Google translate is enabled by default.
Enabled = True

[deepl]
# Deepl Translate does not support async as of right now, it will block all other requests
# while it's processing a Deepl Requests, please enable this with caution!
Enabled = True 

[iciba]
# ICIBA Translate (a.k.a. PowerWord) is disabled by default.
Enabled = True 

[reverso]
Enabled = True

[network]
port = 5000
host = 127.0.0.1
```

Redémarrer le service après les modifications

    sudo systemctl restart simplytranslate

### Proxy nginx

Le fichier de configuration nginx `/etc/nginx/conf.d/traduction.conf`

```
server {
  listen  80;
  server_name traduction.lxcyan.local;
  
  location / { 
      proxy_pass              http://127.0.0.1:5000;
  } 

}
```

Vérifier nginx

    sudo nginx -t

Redémarrer nginx

    sudo systemctl reload nginx

Lien http://traduction.lxcyan.local

