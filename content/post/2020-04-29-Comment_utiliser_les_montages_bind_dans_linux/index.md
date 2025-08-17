+++
title = 'Comment utiliser les montages bind dans linux'
date = 2020-04-29 00:00:00 +0100
categories = ['outils']
+++
Avez-vous déjà eu affaire à un système qui n'était pas correctement cloisonné lors de sa construction et qui est maintenant entré en production ?  
Il vous sera probablement difficile de trouver le temps et la patience nécessaires pour reconstruire le système dans un avenir proche.  
Heureusement, il existe un moyen de contourner les nombreuses limites d'un système mal cloisonné en utilisant des montages "bind".

Ces supports sont assez simples. Au lieu de monter un périphérique (avec un système de fichiers) sur un chemin particulier, vous montez un chemin dans un autre chemin.

Par exemple : Supposons que vous ayez une petite **/var** mais une très grande partition **/opt** et que vous ayez besoin d'espace supplémentaire pour vos fichiers de log qui se développent.

D'abord, arrêtez les services qui écrivent dans les fichiers journaux, puis...

```
mv /var/log /opt/var_log
mkdir /var/log
mount -o bind /opt/var_log /var/log
```

Vous verrez maintenant cela se refléter lors de l'exécution de la commande mount :

```
# mount | grep var
/opt/var_log on /var/log type none (rw,bind)
```

À ce stade, vous êtes prêt à redémarrer les services précédemment interrompus.

Si vous voulez que cela persiste lors des redémarrages, il vous suffit de mettre à jour votre **/etc/fstab** avec le montage de liaison également.

```
# /etc/fstab
/opt/var_log /var/log none bind 0 0
```

Et voilà ! Ce n'est pas très beau, mais cela vous aidera à garder les lumières allumées jusqu'à ce que vous puissiez mettre en place une solution à long terme.