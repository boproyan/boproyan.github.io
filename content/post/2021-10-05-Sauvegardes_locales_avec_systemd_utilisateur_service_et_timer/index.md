+++
title = 'Sauvegardes locales avec systemd utilisateur service et timer'
date = 2021-10-05 00:00:00 +0100
categories = ['outils', 'timer']
+++
## systemd timer utilisateur

Le fonctionnement de systemd impose cependant d’avoir deux fichiers : *service*, qui contient la définition du programme et *timer*, qui dit “quand” le lancer et ils doivent porter le même nom 

Créer le dossier systemd utilisateur

    mkdir -p ~/.config/systemd/user

### Création service et timer

Si vous gérez déjà vos services via systemd, vous avez déjà utilisé des “unit” systemd de type “service”.  
Ces “unit” permettent de définir un process et son mode d’éxécution.  
Pour implémenter un “timer” sous systemd, il va nous falloir un fichier “service”.  

Pour notre tâche à planifier, nous allons avoir au final 3 fichiers :

* Le script à exécuter
* Le fichier “service” qui va dire quel script exécuter
* Le fichier “timer” qui va indiquer quand il doit être exécuté.

>A noter que par convention, les fichiers service et timer doivent avoir le même nom

Nous devons exécuter ,une fois par jour , un script de sauvegarde /home/yann/scripts/savyann sur un ordinateur qui n’est pas sous tension 24/24h.

Pour le fichier service `~/.config/systemd/user/savyann.service`, une base simple

```
[Unit]
Description=Sauvegarde jour

[Service]
Type=simple
ExecStart=/bin/bash /home/yann/scripts/sav-yann-media.sh
StandardError=journal
Type=oneshot
```

Je fournis une description à mon service, indique que c’est un process de type simple, le chemin vers mon script et je rajoute que le flux d’erreur est envoyé dans le journal.Il ne faut pas de section [Install] car le script va être piloté par le fichier timer.
La ligne Type=oneshot est importante, c’est elle qui dit à systemd de ne pas relancer le service en boucle.

Le fichier “timer” `~/.config/systemd/user/savyann.timer`

```
[Unit]
Description=Sauvegarde jour

[Timer]
OnBootSec=15min
OnUnitActiveSec=1d

Unit=savyann.service

[Install]
WantedBy=timers.target
```

>Ceci exécute le fichier .service correspondant 15 minutes après le démarrage et ensuite tous les jours pendant que le système est actif.

Le script `/home/yann/scripts/sav-yann-media.sh`

Activation et démarrage du timer

Il est possible de tester le service avec un simple `systemctl --user start savyann.service`, de regarder les logs avec `journalctl --user -u savyann.service`.

Ensuite, pour qu’il soit actif, il faut prévenir systemd

    systemctl --user enable savyann.timer
    systemctl --user start savyann.timer

Gestion et suivi d’un timer

Pour voir la liste des “timers” actifs et la date de leur dernière et prochaine exécution

    systemctl --user list-timers

```
NEXT                         LEFT     LAST                         PASSED  UNIT          ACTIVATES      
Sat 2021-05-29 11:09:25 CEST 23h left Fri 2021-05-28 11:09:25 CEST 25s ago savyann.timer savyann.service

1 timers listed.
Pass --all to see loaded but inactive timers, too.
```

et accéder aux logs de vos “timers” :

journalctl --user -u savyann.service

```
[...]
mai 28 11:05:42 archyan bash[6648]: Fin sauvegarde
mai 28 11:05:42 archyan systemd[752]: savyann.service: Deactivated successfully.
mai 28 11:05:42 archyan systemd[752]: Finished Sauvegarde jour.
mai 28 11:05:42 archyan systemd[752]: savyann.service: Consumed 28.163s CPU time.
```

En cas de modification du *.timer* ou du *.service*, ne pas oublier de faire un `systemctl --user daemon-reload` pour que la version actualisée de vos fichiers soit prise en compte par systemd.
