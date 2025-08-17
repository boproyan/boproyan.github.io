+++
title = 'Alpine Linux - Mises à jour automatique'
date = 2025-03-12 00:00:00 +0100
categories = serveur
+++
*De nombreuses distributions Linux ont un moyen de configurer les mises à jour automatiques, mais de manière assez surprenante, Alpine Linux ne le fait pas.*

![](alpine-linux-logo.png){:height="50"}  

## Mises à jour automatique

`Solution minimale pour maintenir vos systèmes Alpine Linux à jour`{: .prompt-info } 

Pour chaque nouveau serveur Alpine Linux, créer un script shell nommé **apk-autoupgrade** dans le dossier `/etc/periodic/daily/` avecles permissions suivantes : 700

Vous pouvez le créer en une seule commande 

```shell
echo -e "#!/bin/sh\napk upgrade --update | sed \"s/^/[\`date\`] /\" >> /var/log/apk-autoupgrade.log" > /etc/periodic/daily/apk-autoupgrade && \
	chmod 700 /etc/periodic/daily/apk-autoupgrade
```

Votre serveur Alpine Linux se met désormais à jour automatiquement à condition que les tâches cron soient en cours d'exécution.  
Activer les tâches cron

```shell
rc-service crond start
rc-update add crond
```

Le script exécute la commande `apk upgrade --update` une fois par jour, **apk** par défaut ne demande jamais l'intervention de l'utilisateur. Le reste du script est là pour vous aider en fournissant des journaux. Vous pouvez consulter le fichier `/var/log/apk-autoupgrade.log`  pour vous assurer que tout fonctionne correctement. 

Modification script pour envoi notication ntfy

```
#!/bin/sh
apk upgrade --update | sed "s/^/[`date`] /" >> /var/log/apk-autoupgrade.log
#
curl \
    -H "Title: Alpine Linux serveur ntfy" \
    -H "Authorization: Bearer tk_fjh5bfo3zu2cpibgi2jyfkif49xws" \
    -H prio:low \
    -H tags:information_source \
    -d "Fin exécution script /etc/periodic/daily/apk-autoupgrade" \
    https://noti.rnmkcy.eu/yan_infos
```

## Journal 

Voici un exemple de résultat obtenu sur l'un des serveurs 

```
[Wed Mar 12 14:07:35 CET 2025] fetch http://dl-cdn.alpinelinux.org/alpine/v3.21/main/x86_64/APKINDEX.tar.gz
[Wed Mar 12 14:07:35 CET 2025] fetch http://dl-cdn.alpinelinux.org/alpine/v3.21/community/x86_64/APKINDEX.tar.gz
[Wed Mar 12 14:07:35 CET 2025] Upgrading critical system libraries and apk-tools:
[Wed Mar 12 14:07:35 CET 2025] (1/1) Upgrading apk-tools (2.14.6-r2 -> 2.14.6-r3)
[Wed Mar 12 14:07:35 CET 2025] Executing busybox-1.37.0-r9.trigger
[Wed Mar 12 14:07:35 CET 2025] Continuing the upgrade transaction with new apk-tools:
[...]
[Wed Mar 12 14:07:35 CET 2025] (35/42) Upgrading openssh (9.9_p1-r2 -> 9.9_p2-r0)
[Wed Mar 12 14:07:35 CET 2025] (36/42) Upgrading openssl (3.3.2-r4 -> 3.3.3-r0)
[Wed Mar 12 14:07:35 CET 2025] (37/42) Upgrading libffi (3.4.6-r0 -> 3.4.7-r0)
[Wed Mar 12 14:07:35 CET 2025] (38/42) Upgrading python3 (3.12.8-r1 -> 3.12.9-r0)
[Wed Mar 12 14:07:35 CET 2025] (39/42) Upgrading python3-pycache-pyc0 (3.12.8-r1 -> 3.12.9-r0)
[Wed Mar 12 14:07:35 CET 2025] (40/42) Upgrading pyc (3.12.8-r1 -> 3.12.9-r0)
[Wed Mar 12 14:07:35 CET 2025] (41/42) Upgrading python3-pyc (3.12.8-r1 -> 3.12.9-r0)
[Wed Mar 12 14:07:35 CET 2025] (42/42) Upgrading blkid (2.40.2-r4 -> 2.40.4-r0)
[Wed Mar 12 14:07:35 CET 2025] Executing busybox-1.37.0-r12.trigger
[Wed Mar 12 14:07:35 CET 2025] Executing kmod-33-r2.trigger
[Wed Mar 12 14:07:35 CET 2025] Executing mkinitfs-3.11.1-r0.trigger
[Wed Mar 12 14:07:35 CET 2025] ==> initramfs: creating /boot/initramfs-lts for 6.12.18-0-lts
[Wed Mar 12 14:07:35 CET 2025] Executing syslinux-6.04_pre1-r16.trigger
[Wed Mar 12 14:07:35 CET 2025] OK: 224 MiB in 93 packages
```

<u>Ce script ne redémarre pas les services une fois mis à jour</u>.  
Il faut envisager d'étendre le script en vérifiant les mises à jour du noyau ou des services en cours d'exécution, puis en redémarrant le système ou le service en fonction des journaux.  
En attendant , soit redémarrer automatiquement les serveurs à un moment donné  (les redémarrages sont quasi instantanés).

## Liens

* [Comment configurer une tâche de mise à jour hebdomadaire et de redémarrage automatique sur Alpine Linux](https://blog.romaindasilva.fr/comment-configurer-une-tache-de-mise-a-jour-hebdomadaire-et-de-redemarrage-automatique-sur-alpine-linux/)
* [How to see what packages updates available on Alpine Linux](https://www.cyberciti.biz/faq/list-show-what-packages-updates-available-on-alpine-linux/)
* [13 commandes Apk pour la gestion des packages Alpine Linux](https://fr.techtribune.net/linux/13-commandes-apk-pour-la-gestion-des-packages-alpine-linux/289810/)
