+++
title = 'Débogage des plages de pression du pavé tactile'
date = 2019-08-16 00:00:00 +0100
categories = []
+++
## Débogage des plages de pression du pavé tactile

https://wayland.freedesktop.org/libinput/doc/latest/touchpad-pressure-debugging.html#touchpad-pressure-hwdb

Les plages de pression du pavé tactile dépendent de l'entrée des particularités de l' appareil spécifiques à chaque modèle d'ordinateur portable. Pour vérifier si une plage de pression  est déjà définie pour votre appareil, utilisez l'outil libinput quirks :

    $ libinput quirks list /dev/input/event19

Si votre appareil ne répertorie aucune anomalie, il a probablement besoin d'une plage de pression / taille au toucher, d'un seuil pour la paume de la main et d'un seuil pour le pouce. Commencez par les plages de pression du pavé tactile pour le débogage , puis pour les plages de taille tactiles pour le débogage . Les outils respectifs se fermeront si l’axe requis n’est pas pris en charge.

### Débogage des plages de pression du pavé tactile

Cette section explique comment déterminer les plages de pression du pavé tactile requises pour un périphérique à pavé tactile et comment ajouter localement les aléas de périphérique requis. Notez que cette erreur n’est pas une API publique et peut changer à tout moment . Les utilisateurs sont invités à signaler un bogue avec les plages de pression mises à jour une fois les tests terminés.

Outils sur archlinux

    yaourt -S libevdev python-libevdev python-pyudev

Utilisez l'outil **libinput measure touchpad-pressure** fourni par libinput. Cet outil recherchera votre appareil à pavé tactile et imprimera des statistiques de pression, indiquant notamment si une touche est / était considérée logiquement comme étant en panne.

>Remarque : Cet outil ne fonctionnera que sur les touchpads avec pression.

Voici un exemple de sortie de l'outil:

$ sudo libinput measure touchpad-pressure

```
Using AlpsPS/2 ALPS GlidePoint: /dev/input/event11

Ready for recording data.
Pressure range used: 15:12
Palm pressure range used: 130
Thumb pressure range used: 127
Place a single finger on the touchpad to measure pressure values.
Ctrl+C to exit

Sequence 34 pressure: min:   7 max:  20 avg:  15 median:  16 tags: down
Sequence 35 pressure: min:   9 max:  21 avg:  15 median:  16 tags: down
Sequence 36 pressure: min:   9 max:  21 avg:  16 median:  16 tags: down
Sequence 37 pressure: min:   7 max:  23 avg:  15 median:  17 tags: down
Sequence 38 pressure: min:  15 max:  28 avg:  22 median:  24 tags: down
Sequence 39 pressure: min:  11 max:  27 avg:  21 median:  24 tags: down
Sequence 40 pressure: min:  22 max:  30 avg:  26 median:  27 tags: down
Sequence 41 pressure: min:  10 max:  30 avg:  24 median:  28 tags: down
Sequence 42 pressure: min:  24 max:  33 avg:  28 median:  27 tags: down
Sequence 42 pressure: min:   0 max:  33 avg:  23 median:  27 tags: down
```

L'exemple de sortie montre cinq séquences tactiles terminées et une séquence en cours. Pour chacune d'elles, les valeurs de pression minimale et maximale respectives sont imprimées, ainsi que certaines statistiques. Les tags montrent que cette séquence a été considérée logiquement à un moment donné. Il s’agit d’un outil interactif dont le résultat peut changer fréquemment.

Par défaut, cet outil utilise les particularités de l' appareil pour la plage de pression. Pour affiner les meilleures valeurs pour votre appareil, spécifiez les seuils de pression «logiquement bas» et «logiquement haut» avec l'argument `--touch-thresholds` :

    $ sudo libinput measure touchpad-pressure --touch-thresholds=10:8 --palm-threshold=20

Interagissez avec le pavé tactile et vérifiez si la sortie de cet outil correspond à vos attentes.

>Remarque :   
C'est un processus interactif. Vous devrez réexécuter l'outil avec différents seuils jusqu'à ce que vous trouviez la plage appropriée pour votre pavé tactile. La connexion de journaux de sortie à un bogue ne vous aidera pas. Seul un accès au matériel peut déterminer les plages correctes.

Une fois que les seuils sont définis (par exemple 10 et 8), ils peuvent être activés avec une entrée Device Quirks similaire à celle-ci:

$> cat /etc/libinput/local-overrides.quirks

```
[Touchpad pressure override]
MatchUdevType=touchpad
MatchName=*SynPS/2 Synaptics TouchPad
MatchDMIModalias=dmi:*svnLENOVO:*:pvrThinkPadX230*
AttrPressureRange=10:8
```

Le nom du fichier doit être **/etc/libinput/local-overrides.quirks** . La première ligne est le nom de la section et peut être de forme libre. Les directives **Match** limitent le caprice à votre pavé tactile, assurez-vous que le nom du périphérique correspond au nom de votre périphérique. La correspondance dmi modalias doit être basée sur les informations contenues dans **/sys/class/dmi/id/modalias** . Ces modalias doivent être abrégés en informations du système spécifique, généralement le fournisseur du système (svn) et le nom du produit (pn).

Une fois en place, exécutez la commande suivante pour vérifier que le problème est corrigé et que cela fonctionne pour votre appareil:

    $ sudo libinput list-quirks /dev/input/event10
        AttrPressureRange=10:8

Remplacez le nœud d'événement par celui de votre périphérique. Si le **AttrPressureRange** n'apparaît pas, réexécutez l'exécution avec `--verbose` et vérifiez si le AttrPressureRange contient des messages d'erreur.

Si le décalage de la plage de pression apparaît correctement, redémarrez X ou le compositeur Wayland et libinput devrait maintenant utiliser les seuils de pression corrects. Les outils d'assistance peuvent être utilisés pour vérifier la fonctionnalité correcte sans avoir besoin d'un redémarrage.

Une fois que les plages de pression sont jugées correctes, signalez un bogue pour obtenir les plages de pression dans le référentiel.

### Débogage des plages de tailles tactiles

Cette section explique comment déterminer les plages de tailles de pavé tactile requises pour un périphérique à pavé tactile et comment ajouter localement les défauts de périphérique requis. Notez que cette erreur n’est pas une API publique et peut changer à tout moment . Les utilisateurs sont invités à signaler un bogue avec les plages de pression mises à jour une fois les tests terminés.

>Remarque  
Cet outil ne fonctionnera que sur les pavés tactiles dotés de l'axe ABS_MT_MAJOR .

Voici un exemple de sortie de l'outil:

    $ sudo libinput measure touch-size --touch-thresholds 10:8 --palm-threshold 14

```
Using ELAN Touchscreen: /dev/input/event5
&nbsp;
Ready for recording data.
Touch sizes used: 10:8
Palm size used: 14
Place a single finger on the device to measure touch size.
Ctrl+C to exit
&nbsp;
Sequence: major: [  9.. 11] minor: [  7..  9]
Sequence: major: [  9.. 10] minor: [  7..  7]
Sequence: major: [  9.. 14] minor: [  6..  9]  down
Sequence: major: [ 11.. 11] minor: [  9..  9]  down
Sequence: major: [  4.. 33] minor: [  1..  5]  down palm
```

L'exemple de sortie montre cinq séquences tactiles terminées. Pour chacune d'elles, les valeurs de pression minimale et maximale respectives sont imprimées, ainsi que certaines statistiques. Les balises down et palm indiquent que la séquence a été considérée logiquement comme une palme à un moment donné. Il s’agit d’un outil interactif dont le résultat peut changer fréquemment.  
Par défaut, cet outil utilise les particularités de l' appareil pour la plage de taille tactile. Pour limiter les valeurs optimales pour votre appareil, spécifiez les seuils de pression «logiquement bas» et «logiquement haut» avec les arguments **--touch-thresholds** comme dans l'exemple ci-dessus.

Interagissez avec le pavé tactile et vérifiez si la sortie de cet outil correspond à vos attentes.

>Remarque : C'est un processus interactif. Vous devrez réexécuter l'outil avec différents seuils jusqu'à ce que vous trouviez la plage appropriée pour votre pavé tactile. La connexion de journaux de sortie à un bogue ne vous aidera pas. Seul un accès au matériel peut déterminer les plages correctes.

Une fois que les seuils sont définis (par exemple 10 et 8), ils peuvent être activés avec une entrée Device Quirks similaire à celle-ci:

    $> cat /etc/libinput/local-overrides.quirks

```
[Touchpad touch size override]
MatchUdevType=touchpad
MatchName=*SynPS/2 Synaptics TouchPad
MatchDMIModalias=dmi:*svnLENOVO:*:pvrThinkPadX230*
AttrTouchSizeRange=10:8
```

La première ligne est la ligne de correspondance et doit être ajustée pour le nom du périphérique (voir la sortie de l'enregistrement de libinput ) et pour le système local, en fonction des informations contenues dans **/sys/class/dmi/id/modalias** . Les modalias doivent être abrégés en informations du système spécifique, généralement le fournisseur du système (svn) et le nom du produit (pn).

Une fois en place, exécutez la commande suivante pour vérifier que le problème est valide et fonctionne pour votre appareil:

    $ sudo libinput list-quirks /dev/input/event10
        AttrTouchSizeRange=10:8

Remplacez le nœud d'événement par celui de votre périphérique. Si le AttrTouchSizeRange n'apparaît pas, réexécutez l'exécution avec --verbose et vérifiez si le message de sortie --verbose messages d'erreur.

Si la propriété Plage de tailles tactiles s'affiche correctement, redémarrez X ou le compositeur Wayland et libinput devrait maintenant utiliser les seuils appropriés. Les outils d'assistance peuvent être utilisés pour vérifier la fonctionnalité correcte sans avoir besoin d'un redémarrage.

Une fois que les plages de taille tactile sont considérées comme correctes, les rapports de bogues «signalent un bogue» pour obtenir les seuils dans le référentiel. 