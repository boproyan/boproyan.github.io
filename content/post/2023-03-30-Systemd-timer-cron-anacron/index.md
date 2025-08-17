+++
title = 'Utiliser les temporisateurs (Timers Oncalendar) Systemd pour remplacer Cron'
date = 2023-03-30 00:00:00 +0100
categories = outils timer
+++
*Linux cron a fonctionné comme le planificateur de tâches basé sur le temps Unix pendant de nombreuses années mais les minuteries Systemd commencent à remplacer cron*

- [Cron](#cron)
    - [Exécuter cron](#exécuter-cron)
- [Systemd Timer](#systemd-timer)
    - [Création service et timer](#création-service-et-timer)
    - [Spécifications des événements du calendrier](#spécifications-des-événements-du-calendrier)
    - [Tester les spécifications du calendrier](#tester-les-spécifications-du-calendrier)
        - [Syntaxe date "OnCalendar"](#syntaxe-date-oncalendar)
        - [Explication du format des temporisateurs systemd onCalendar (cron)](#explication-du-format-des-temporisateurs-systemd-oncalendar-cron)
    - [Activation et démarrage du timer](#activation-et-démarrage-du-timer)
    - [Gestion et suivi d’un timer](#gestion-et-suivi-d’un-timer)
    - [Archlinux Nettoyage du cache (systemd-timer)](#archlinux-nettoyage-du-cache-systemd-timer)
- [Tutoriel Systemd Timer avec un exemple d'envoi automatique d'e-mails](#tutoriel-systemd-timer-avec-un-exemple-denvoi-automatique-de-mails)
    - [Unité de service (Service Unit)](#unité-de-service-service-unit)
    - [Unité de temporisarion (Timer Unit)](#unité-de-temporisarion-timer-unit)
    - [Installer  et cible ([Install] and target)](#installer-et-cible-install-and-target)
    - [Commandes relatives à la minuterie](#commandes-relatives-à-la-minuterie)
    - [Commandes relatives au journal](#commandes-relatives-au-journal)
- [Liens](#liens)


## Cron

### Exécuter cron 

*un jour spécifique dans le mois (e.g. deuxième lundi)*

Comment exécuter un cron sur un jour spécifique de la semaine une fois par mois ?  
Ceci pourrait sembler simple au premier abord, puisque cette ligne pourrait semblait faire l’affaire :

```
# Run on every second Tuesday of the month
15 3 8-14 * 2  /usr/bin/bash /opt/myscriptfortuesday.sh
```

Mais ceci ne marcherait pas car le ‘2’ pour vérifier que nous sommes bien un mardi vient comme une condition OR, et donc la commande pourrait s’exécuter du jour 8 au jour 14 et tous les mardis du mois.

Pour contourner cela, vous pouvez utiliser cette commande :

```
# Run on every second Tuesday of the month
15 3 8-14 * * test $(date +\%u) -eq 2 && /usr/bin/bash /opt/myscriptfortuesday.sh
```

Voici l’explication de cette ligne de cron :

```
15   = 15th minute
3    = 3am
8-14 = between day 8 and day 14 (second week)
*    = every month
*    = every day of the week
test $(date +\%u) -eq 2 && /usr/bin/bash /opt/myscriptfortuesday.sh = the command to execute with a check on the date
```

En effectuant cette vérification, nous vérifions alors d’abord que nous sommes bien un mardi avant d’exécuter la commande. N’oubliez pas d’ajouter un antislash avant le caractère ‘%’ pour l’échapper.

## Systemd Timer

Le fonctionnement de **systemd** impose d'avoir deux fichiers :  
**service**, qui contient la définition du programme  
**timer**, qui dit "quand" le lancer.  

>Ils doivent porter le même nom et se situer dans **/etc/systemd/system/**.

### Création service et timer

Si vous gérez déjà vos services via systemd, vous avez déjà utilisé des “unit” systemd de type “service”.  
Ces “unit” permettent de définir un process et son mode d’éxécution.  
Pour implémenter un “timer” sous systemd, il va nous falloir un fichier “service”.

Pour notre tâche à planifier, nous allons avoir au final 3 fichiers :

    Le script à exécuter
    Le fichier “service” qui va dire quel script exécuter
    Le fichier “timer” qui va indiquer quand il doit être exécuté.

>A noter que par convention, les fichiers service et timer doivent avoir le même nom

Nous devons exécuter ,une fois par jour , un script de sauvegarde **/home/yannick/scripts/savarch** sur un ordinateur qui n'est pas sous tension 24/24h.

Pour le fichier service **/etc/systemd/system/savarch.service**, une base simple

```
[Unit]
Description=Sauvegarde jour

[Service]
Type=simple
ExecStart=/bin/bash /home/yannick/scripts/savarch
StandardError=journal
Restart=on-abort


[Install]
WantedBy=multi-user.target
```

Je fournis une description à mon service, indique que c’est un process de type simple, le chemin vers mon script et je rajoute que le flux d’erreur est envoyé dans le journal.Il ne faut pas de section [Install] car le script va être piloté par le fichier timer.  
La ligne `Type=oneshot` est importante, c'est elle qui dit à systemd de ne pas relancer le service en boucle.

Le fichier “timer” **/etc/systemd/system/savarch.timer**  

```
[Unit]
Description=Sauvegarde jour

[Timer]
# lisez le man systemd.timer(5) pour tout ce qui est disponible
OnCalendar=daily
# Autoriser la persistence entre les reboot
Persistent=true
Unit=savarch.service

[Install]
WantedBy=timers.target
```

* **OnCalendar** permet d’indiquer l’occurrence et la fréquence d’exécution du script. Il y a les abréviations classiques ("minutely", "hourly", "daily", "monthly", "weekly", "yearly", "quarterly", "semiannually", etc) mais vous pouvez avoir des choses plus complexes comme "Mon,Tue *-*-01..04 12:00:00" - voir **systemd.time**
* **Persistent** va forcer l’exécution du script si la dernière exécution a été manquée suite à un reboot de serveur ou autre événement.
* **Install** va créer la dépendance pour que votre “timer” soit bien exécuté et pris en compte par systemd.

### Spécifications des événements du calendrier

Les spécifications des événements du calendrier sont un élément clé du déclenchement des minuteries aux moments répétitifs souhaités. Commencez par examiner certaines spécifications utilisées avec le paramètre **OnCalendar**.

*systemd et ses minuteries utilisent un style différent pour les spécifications d'heure et de date que le format utilisé dans crontab. Il est plus flexible que crontab et permet des dates et des heures floues à la manière de la commande at. Il devrait également être suffisamment familier pour être facile à comprendre.*

Le format de base des minuteries systemd utilisant `OnCalendar=` est `DOW YYYY-MM-DD HH:MM:SS`  
DOW (jour de la semaine) est facultatif, et les autres champs peuvent utiliser un astérisque (*) pour correspondre à toute valeur pour cette position. Toutes les formes d'heure du calendrier sont converties en une forme normalisée. Si l'heure n'est pas spécifiée, elle est supposée être 00:00:00. Si la date n'est pas spécifiée mais que l'heure l'est, la prochaine correspondance peut être aujourd'hui ou demain, selon l'heure actuelle. Des noms ou des nombres peuvent être utilisés pour le mois et le jour de la semaine. Des listes séparées par des virgules peuvent être spécifiées pour chaque unité. Des plages d'unités peuvent être spécifiées avec .. entre les valeurs de début et de fin.

Il existe quelques options intéressantes pour spécifier les dates. Le tilde (~) peut être utilisé pour indiquer le dernier jour du mois ou un certain nombre de jours avant le dernier jour du mois. Le "/" peut être utilisé pour spécifier un jour de la semaine comme modificateur.

Voici quelques exemples de spécifications temporelles typiques utilisées dans les instructions **OnCalendar**.

Evénement                     | Description
------------------------------|-------------
`DOW YYYY-MM-DD HH:MM:SS    ` |	 
`*-*-* 00:15:30             ` | Chaque jour de chaque mois de chaque année à 15 minutes et 30 secondes après minuit.
`Weekly                     ` | hebdomadaire, chaque lundi à 00:00:00
`Mon *-*-* 00:00:00         ` | Identique à hebdomadaire (weekly)
`Mon                        ` | Identique à la semaine (weekly)
`Wed 2020-*-*               ` |Tous les mercredis de 2020 à 00:00:00
`Mon..Fri 2021-*-*          ` |Tous les jours de la semaine en 2021 à 00:00:00 (hors samedi et dimanche)
`2022-6,7,8-1,15 01:15:00   ` | Les 1er et 15 juin, juillet et août de l'année 2022 à 01:15:00
`Mon *-05~03                ` | La prochaine occurrence d'un lundi en mai de n'importe quelle année qui est aussi le 3ème jour à partir de la fin du mois.
`Mon..Fri *-08~04           ` | Le 4ème jour précédant la fin du mois d'août pour toutes les années où il tombe également un jour de semaine.
`*-05~03/2                  ` | Le 3ème jour à partir de la fin du mois de mai, puis à nouveau deux jours plus tard. Se répète chaque année. Notez que cette expression utilise le tilde (`~`). 
`*-05-03/2                  ` | Le troisième jour du mois de mai et ensuite tous les deux jours pour le reste du mois de mai. Se répète chaque année. Notez que cette expression utilise le tiret (`-`).

### Tester les spécifications du calendrier

**systemd-analyze calendar**  
systemd fournit un excellent outil pour valider et examiner les spécifications des événements temporels d'un calendrier dans une minuterie.  
L'outil `systemd-analyze calendar` analyse une spécification d'événement temporel de calendrier et fournit la forme normalisée ainsi que d'autres informations intéressantes telles que la date et l'heure du prochain "elapse", c'est-à-dire la correspondance, et le temps approximatif avant que l'heure de déclenchement ne soit atteinte.

Tout d'abord, regardez une date dans le futur sans heure (notez que les heures pour Next elapse et UTC seront différentes en fonction de votre fuseau horaire local) :

    systemd-analyze calendar 2030-06-17

```
  Original form: 2030-06-17
Normalized form: 2030-06-17 00:00:00
    Next elapse: Mon 2030-06-17 00:00:00 CEST
       (in UTC): Sun 2030-06-16 22:00:00 UTC
       From now: 7 years 11 months left
```

Ajoutez maintenant une heure. Dans cet exemple, la date et l'heure sont analysées séparément comme des entités non liées :

    systemd-analyze calendar 2030-06-17 15:21:16

```
  Original form: 2030-06-17
Normalized form: 2030-06-17 00:00:00
    Next elapse: Mon 2030-06-17 00:00:00 CEST
       (in UTC): Sun 2030-06-16 22:00:00 UTC
       From now: 7 years 11 months left

  Original form: 15:21:16
Normalized form: *-*-* 15:21:16
    Next elapse: Thu 2022-06-23 15:21:16 CEST
       (in UTC): Thu 2022-06-23 13:21:16 UTC
       From now: 6h left
```

Pour analyser la date et l'heure comme une seule unité, mettez-les entre guillemets. Veillez à supprimer les guillemets lorsque vous les utilisez dans la spécification de l'événement OnCalendar= dans une unité de temps, sinon vous obtiendrez des erreurs :

    systemd-analyze calendar "2030-06-17 15:21:16"

Testez maintenant les entrées du tableau 2. J'aime particulièrement la dernière :

    systemd-analyze calendar "2022-6,7,8-1,15 01:15:00"

```
Normalized form: 2030-06-17 15:21:16
    Next elapse: Mon 2030-06-17 15:21:16 CEST
       (in UTC): Mon 2030-06-17 13:21:16 UTC
       From now: 7 years 11 months left
```

Examinons un exemple dans lequel nous listons les cinq prochains écoulements pour l'expression timestamp.

    systemd-analyze calendar --iterations=5 "Mon *-05~3"

```
  Original form: Mon *-05~3
Normalized form: Mon *-05~03 00:00:00
    Next elapse: Mon 2023-05-29 00:00:00 CEST
       (in UTC): Sun 2023-05-28 22:00:00 UTC
       From now: 11 months 4 days left
       Iter. #2: Mon 2028-05-29 00:00:00 CEST
       (in UTC): Sun 2028-05-28 22:00:00 UTC
       From now: 5 years 11 months left
       Iter. #3: Mon 2034-05-29 00:00:00 CEST
       (in UTC): Sun 2034-05-28 22:00:00 UTC
       From now: 11 years 11 months left
       Iter. #4: Mon 2045-05-29 00:00:00 CEST
       (in UTC): Sun 2045-05-28 22:00:00 UTC
       From now: 22 years 11 months left
       Iter. #5: Mon 2051-05-29 00:00:00 CEST
       (in UTC): Sun 2051-05-28 22:00:00 UTC
       From now: 28 years 11 months left
```

Cela devrait vous donner suffisamment d'informations pour commencer à tester vos spécifications temporelles **OnCalendar**.

#### Syntaxe date "OnCalendar"

Forme minimale               | &rarr; |      Forme normalisée
-------------------------- |------ | --------------------------------
`Sat,Thu,Mon-Wed,Sat-Sun   `  | &rarr; | `Mon-Thu,Sat,Sun *-*-* 00:00:00`
`Mon,Sun 12-*-* 2,1:23     `  | &rarr; | `Mon,Sun 2012-*-* 01,02:23:00`
`Wed *-1                   `  | &rarr; | `Wed *-*-01 00:00:00`
`Wed-Wed,Wed *-1           `  | &rarr; | `Wed *-*-01 00:00:00`
`Wed, 17:48                `  | &rarr; | `Wed *-*-* 17:48:00`
`Wed-Sat,Tue 12-10-15 1:2:3`  | &rarr; | `Tue-Sat 2012-10-15 01:02:03`
`*-*-7 0:0:0               `  | &rarr; | `*-*-07 00:00:00`
`10-15                     `  | &rarr; | `*-10-15 00:00:00`
`monday *-12-* 17:00       `  | &rarr; | `Mon *-12-* 17:00:00`
`Mon,Fri *-*-3,1,2 *:30:45 `  | &rarr; | `Mon,Fri *-*-01,02,03 *:30:45`
`12,14,13,12:20,10,30      `  | &rarr; | `*-*-* 12,13,14:10,20,30:00`
`mon,fri *-1/2-1,3 *:30:45 `  | &rarr; | `Mon,Fri *-01/2-01,03 *:30:45`
`03-05 08:05:40            `  | &rarr; | `*-03-05 08:05:40`
`08:05:40                  `  | &rarr; | `*-*-* 08:05:40`
`05:40                     `  | &rarr; | `*-*-* 05:40:00`
`Sat,Sun 12-05 08:05:40    `  | &rarr; | `Sat,Sun *-12-05 08:05:40`
`Sat,Sun 08:05:40          `  | &rarr; | `Sat,Sun *-*-* 08:05:40`
`2003-03-05 05:40          `  | &rarr; | `2003-03-05 05:40:00`
`2003-03-05                `  | &rarr; | `2003-03-05 00:00:00`
`03-05                     `  | &rarr; | `*-03-05 00:00:00`
`hourly                    `  | &rarr; | `*-*-* *:00:00`
`daily                     `  | &rarr; | `*-*-* 00:00:00`
`monthly                   `  | &rarr; | `*-*-01 00:00:00`
`weekly                    `  | &rarr; | `Mon *-*-* 00:00:00`
`*:20/15                   `  | &rarr; | `*-*-* *:20/15:00`

`Note : *:20/15 signifie *:20:00, *:35:00, *:50:00, en recommençant l'heure suivante à *:20:00`

#### Explication du format des temporisateurs systemd onCalendar (cron)

[Systemd timers onCalendar (cron) format explained](https://silentlad.com/systemd-timers-oncalendar-(cron)-format-explained)

<u>Format de tâche cron</u>

```
* * * * *
minute heures jour du mois mois jour de la semaine
```

Ainsi, par exemple, si nous voulons qu'un travail s'exécute chaque fois qu'il est minuit. Nous mettrons minuteà 00 et hoursà 00 et tout le reste tel quel, cela nous donnera -

```
00 00 * * *
# tous les jours minuit
```

De même, si nous voulons que ce soit tous les jours de la semaine (plage) minuit, ce sera

```
00 00 * * 1-5
# tous les jours minuit
```

Et le dernier concept est de répéter ce qui se passe si nous voulons qu'un travail s'exécute toutes nles minutes. Ensuite, nous utilisons ce format.

```
*/n * * * *
# toutes les n minutes
```

De même, ces répétitions peuvent être utilisées pour l'un des 5 champs avec le temps absolu.

<u>Format Systemd Timer OnCalendar</u>

Maintenant que vous connaissez les bases du format de cron, il vous sera plus facile de comprendre le format OnCalendar qui vous donne plus de granularité et de contrôle.  
Donc, le format de base de l'événement Oncalnedar est le suivant   
`* *-*-* *:*:*`  
Il est divisé en 3 parties  

* `*-` Pour signifier le jour de la semaine, par exemple : - Sat, Thu, Mon
* `*-*-*-` Pour signifier la date du calendrier. Ce qui signifie qu'il se décompose en - `year-month-date`.
    * `2021-10-15` est le 15 octobre
    * `*-10-15` signifie chaque année au 15 octobre
    * `*-01-01` signifie chaque nouvelle année.
    * `*:*:*` est de signifier la composante temporelle de l'événement calendaire. Donc c'est -`hour:minute:second`

>Remarque- Contrairement à cronjob, nous pouvons ignorer un composant de oncalendar event si nous n'avons aucune modification à lui apporter.

Ce qui signifie

```
Mer *-*-* 17:48:00
#Peut aussi s'écrire ->
mer, 17:48
```

Désormais, onCalendar respecte également toutes les règles du format cron. Nous allons donc aborder chaque concept un par un.

<u>Minuteries Systemd qui s'exécutent à un moment précis</u>

C'est assez simple, il suffit de remplir les champs ci-dessous avec le moment souhaité.  

```
* *-*-* *:*:*
#Jour de la semaine Année-Mois-Date Heure :Minute :Seconde
```

<u>Systemd Timers Oncalendar Exemples qui s'exécutent à une fréquence donnée</u>

Donc, comme nous avons utilisé /la fréquence dans le travail cron, nous faisons la même chose ici.
Ainsi, par exemple, pour tous les 2 jours, nous disons

`*/1 *-*-* *:*:*`

De même, nous pouvons également définir la période initiale. Donc, tous les jours à partir de lundi  

`Lun/1 *-*-* *:*:*`

Et nous pouvons définir une plage pour la fréquence à exécuter, donc tous les jours mais seulement les jours de la semaine

`Lun,Mar,Mer,Jeu,Ven *-*-* *:*:*`

Ainsi, des exemples de gamme Systemd Timers Oncalendar et des exemples normaux sont donnés dans le tableau ci-dessous.  
Utilisez la commande calendar de systemd-analyze ([Analyzing systemd calendar and timespans](https://opensource.com/article/20/7/systemd-calendar-timespans)) pour valider et normaliser les spécifications d'heure calendaire à des fins de test. L'outil calcule également quand un événement de calendrier spécifié se produirait ensuite.

<u>Exemples OnCalendar, exemples Systemd Timer</u>

Explication | Systemd Timer
:-----------  |:---------------
Toutes les minutes | `*-*-* *:*:00`
Toutes les 2 minutes | `*-*-* *:*/2:00`
Toutes les 5 minutes | `*-*-* *:*/5:00`
Toutes les 15 minutes | `*-*-* *:*/15:00`
Tous les quarts d'heure | `*-*-* *:*/15:00`
Toutes les 30 minutes | `*-*-* *:*/30:00`
Toutes les demi-heures | `*-*-* *:*/30:00`
Toutes les 60 minutes | `*-*-* */1:00:00`
Toutes les 1 heures | `*-*-* *:00:00`
Toutes les 2 heures | `*-*-* */2:00:00`
Toutes les 3 heures | `*-*-* */3:00:00`
Toutes les autres heures | `*-*-* */2:00:00`
Toutes les 6 heures | `*-*-* */6:00:00`
Toutes les 12 heures | `*-*-* */12:00:00`
Plage horaire  | `*-*-* 9-17:00:00`
Entre certaines heures | `*-*-* 9-17:00:00`
Tous les jours | `*-*-* 00:00:00`
Tous les jours | `*-*-* 00:00:00`
Une fois par jour | `*-*-* 00:00:00`
Chaque nuit | `*-*-* 01:00:00`
Tous les jours à 1 heure du matin | `*-*-* 01:00:00`
Tous les jours à 2 heures du matin | `*-*-* 02:00:00`
Tous les matins | `*-*-* 07:00:00`
Tous les jours à minuit | `*-*-* 00:00:00`
Tous les jours à minuit | `*-*-* 00:00:00`
Chaque nuit à minuit | `*-*-* 00:00:00`
Tous les dimanches dim | `*-*-* 00:00:00`
Tous les vendredis Ven | `*-*-* 01:00:00`
Tous les vendredis à minuit Ven | `*-*-* 00:00:00`
Tous les samedis sam | `*-*-* 00:00:00`
Tous les jours de la semaine Lun...Ven | `*-*-* 00:00:00`
en semaine uniquement Lun...Ven | `*-*-* 00:00:00`
du lundi au vendredi Lun...Ven | `*-*-* 00:00:00`
Tous les week-ends sam, dim | `*-*-* 00:00:00`
week-end uniquement sam,dim | `*-*-* 00:00:00`
Tous les 7 jours | `* *-*-* 00:00:00`
Chaque semaine Dim | `*-*-* 00:00:00`
chaque semaine dim | `*-*-* 00:00:00`
une fois par semaine Dim | `*-*-* 00:00:00`
Chaque mois | `* *-*-01 00:00:00`
mensuel | `* *-*-01 00:00:00`
une fois par mois | `* *-*-01 00:00:00`
Tous les trimestres | `* *-01,04,07,10-01 00:00:00`
Tous les 6 mois | `* *-01,07-01 00:00:00`
Chaque année | `* *-01-01 00:00:00`

### Activation et démarrage du timer

Il est possible de tester le service avec un simple `systemctl start savarch.service`, de regarder les logs avec `journalctl -u savarch.service`.

Ensuite, pour qu'il soit actif, il faut prévenir systemd

    sudo systemctl enable savarch.timer
    sudo systemctl start savarch.timer

### Gestion et suivi d’un timer

Pour voir la liste des “timers” actifs et la date de leur dernière et prochaine exécution

    systemctl list-timers

```
NEXT                         LEFT     LAST                         PASSED    UNIT                         ACTIVATES
Fri 2018-03-02 00:00:00 CET  15h left Thu 2018-03-01 07:49:43 CET  56min ago logrotate.timer              logrotate.service
Fri 2018-03-02 00:00:00 CET  15h left Thu 2018-03-01 07:49:43 CET  56min ago man-db.timer                 man-db.service
Fri 2018-03-02 00:00:00 CET  15h left Thu 2018-03-01 07:49:43 CET  56min ago savarch.timer                savarch.service
Fri 2018-03-02 00:00:00 CET  15h left Thu 2018-03-01 07:49:43 CET  56min ago shadow.timer                 shadow.service
Fri 2018-03-02 00:00:00 CET  15h left Thu 2018-03-01 07:49:43 CET  56min ago updatedb.timer               updatedb.service
Fri 2018-03-02 08:04:45 CET  23h left Thu 2018-03-01 08:04:45 CET  41min ago systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service
```

et accéder aux logs de vos “timers” :

    journalctl -u savarch.service

```
...
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:15 Départ sauvegarde
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:15 synchronisation source partagée /home/yannick/media/Musique cible locale /home/yannick/media/savlocal/Musique
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:21 synchronisation partagée /home/yannick/media/devel cible locale /home/yannick/media/savlocal/devel
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:33 synchronisation partagée /home/yannick/media/BiblioCalibre cible locale /home/yannick/media/savlocal/BiblioCalibre
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:41 SAUVEGARDE /home/yannick -> /home/yannick/media/savlocal/yannick-pc/yannick
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:42 SAUVEGARDE /home/yannick/media/dplus -> /home/yannick/media/savlocal/dplus
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:47 SAUVEGARDE /home/yannick/media/yanspm-md -> /home/yannick/media/savlocal/yanspm-md
mars 01 08:43:47 yannick-pc bash[1932]: 2018-03-01 08:43:47 FIN sauvegarde
mars 01 08:43:47 yannick-pc systemd[1]: Started Sauvegarde jour.
```

En cas de modification du **.timer** ou du **.service**, ne pas oublier de faire un **systemctl daemon-reload** pour que la version actualisée de vos fichiers soit prise en compte par systemd.

### Archlinux Nettoyage du cache (systemd-timer)

**pkgcacheclean** est un petit programme qui nettoie le cache pacman mais conserve n versions du paquet dans le cache. 

    yaourt -S pkgcacheclean

**Manuellement**  

    pkgcacheclean -nv 3

Cette commande va afficher (et seulement afficher, aucun fichier n'est modifié) ce qu'il ferait si vous vouliez conserver 3 versions pour chaque paquet installé.  
Pour effectuer les changements sur le disque :

    pkgcacheclean -v 3

**Automatiquement**  
Nettoyage du cache pacman une fois par mois avec conservation de 3 versions (2 précédentes + 1 en cours)  

Fichier **/etc/systemd/system/pkgcacheclean.timer**

```
[Unit]
Description=Minuterie mensuelle pour pkgcacheclean (nettoyage cache pacman)

[Timer]
OnCalendar=monthly
AccuracySec=5d
Persistent=true
Unit=pkgcacheclean.service

[Install]
WantedBy=multi-user.target
```

Fichier **/etc/systemd/system/pkgcacheclean.service**

```
[Unit]
Description=Nettoyage cache pacman avec conservation de 3 versions

[Service]
Nice=19
IOSchedulingClass=2
IOSchedulingPriority=7
Type=oneshot
ExecStart=/usr/bin/pkgcacheclean -v 3
```

Ensuite, pour qu'il soit actif, il faut prévenir systemd

    sudo systemctl enable pkgcacheclean.timer
    sudo systemctl start pkgcacheclean.timer

>Ne pas oublier un `sudo systemctl daemon-reload` après avoir modifier un fichier, sinon ce n'est pas pris en compte.

## Tutoriel Systemd Timer avec un exemple d'envoi automatique d'e-mails


### Unité de service (Service Unit)

Comme mentionné précédemment, l'unité de service est la tâche à effectuer, comme l'envoi d'un e-mail.  
La création d'un nouveau Service est très simple, qui consiste à créer un nouveau fichier dans le répertoire /usr/lib/systemd/system, tel que le fichier mytimer.service. Par exemple, vous pouvez taper le code suivant.

```
[Unit]
Description=MyTimer

[Service]
ExecStart=/bin/bash /path/to/mail.sh
```

Comme vous pouvez le voir, le fichier de l'unité de service est divisé en deux parties.

La partie[Unit] décrit les informations de base de l'unité (c'est-à-dire les métadonnées) et le champ Description donne une brève introduction de l'unité (c'est-à-dire MyTimer).

La partie[Service] sert à personnaliser les actions et Systemd fournit de nombreux champs.

* ExecStart : commandes à exécuter par systemctl start
* ExecStop : commandes à exécuter par systemctl stop
* ExecReload : commandes à exécuter par rechargement systemctl
* ExecStartPre : commandes à exécuter automatiquement avant ExecStartPre
* ExecStartPost : commandes à exécuter automatiquement après ExecStartPost
* ExecStopPost : commandes à exécuter après ExecStop automatiquement

>Notez que lors de la définition, tous les chemins doivent être écrits en tant que chemins absolus. Par exemple, bash doit être écrit comme /bin/bash, sinon Systemd ne le trouvera pas.

Démarrez maintenant le Service.

	sudo systemctl démarrer mytimer.service

Si tout va bien, vous recevrez un email.

### Unité de temporisarion (Timer Unit)

L'unité de service (Service Unit) définit simplement comment exécuter la tâche.  
Vous devez définir l'unité Minuterie (Timer Unit) afin d'exécuter le Service périodiquement.  
Créez un nouveau fichier mytimer.timer dans le répertoire /usr/lib/systemd/system, et tapez le code suivant.  

```
[Unit]
Description=Runs mytimer every hour

[Timer]
OnUnitActiveSec=1h
Unit=mytimer.service

[Install]
WantedBy=multi-user.target
```

Le fichier de l'unité de temporisation (Timer) est divisé en plusieurs parties.

La partie[Unit] définit les métadonnées.

La partie[Timer] personnalise la minuterie.    
Et Systemd fournit les champs suivants.

* OnActiveSec : Combien de temps faut-il pour démarrer la tâche après que la minuterie ait pris effet ?
* OnBootSec : Combien de temps faut-il pour lancer la tâche après le démarrage du système ?
* OnStartupSec : Combien de temps faut-il pour démarrer la tâche après le démarrage du processus Systemd ?
* OnUnitActiveSec : Combien de temps faut-il pour que l'unité exécute à nouveau après la dernière exécution de l'unité ?
* OnUnitInactiveSec : Combien de temps faut-il pour exécuter une nouvelle fois depuis que la minuterie a été désactivée la dernière fois ?
* OnCalendar : Exécuter en fonction du temps absolu et non du temps relatif.
* AccuracySec : Si la tâche doit être reportée pour une raison quelconque, le nombre maximum de secondes à reporter est de 60 secondes par défaut.
* Unit : L'intervention à exécuter. L'unité par défaut est l'unité avec le suffixe .service du même nom.
* Persistent : Si le champ est défini, l'unité correspondante sera automatiquement exécutée même si la minuterie ne démarre pas.
* WakeSystem : S'il faut réveiller le système automatiquement si le système se met en veille.

Dans le script ci-dessus, OnUnitActiveSec=1h indique que la tâche sera effectuée par heure.  
D'autres façons d'écrire sont : OnUnitActiveSec=*-*-*-* 02:00:00:00 signifie être exécuté à deux heures du matin, et OnUnitActiveSec=Mon *-*-* 02:00:00:00 signifie être exécuté à deux heures du matin chaque lundi. Pour plus de détails, veuillez vous référer à la documentation officielle.

### Installer  et cible ([Install] and target)

Dans le fichier mytimer.timer, il y a une partie [Install] qui définit les commandes à exécuter lors de l'activation ou de la désactivation de systemctl de l'appareil.  
Dans le script ci-dessus, la partie [Install] ne définit qu'un seul champ, qui est WantedBy=multi-user.target.  
Cela signifie que si systemctl enable mytimer.timer est exécuté (tant que Systemd est démarré, le timer prend automatiquement effet), alors le timer appartient à multi-user.target.  

La cible se réfère à un groupe de processus apparentés, un peu comme le niveau de démarrage sous le mode de processus init. Lorsqu'une Cible est démarrée, tous les processus appartenant à cette Cible seront également démarrés.  
multi-user.target est la cible la plus couramment utilisée. Lorsqu'un système est démarré en mode multi-utilisateur, mytimer.timer est démarré avec. La façon de travailler sous le capot est assez simple. Lors de l'exécution de la commande systemctl enable mytimer.timer, un lien symbolique sera créé dans le répertoire multi-user.target.wants, pointant vers mytimer.timer.

### Commandes relatives à la minuterie

démarrez la minuterie nouvellement créée.

	sudo systemctl start mytimer.timer

Vous recevrez le courriel immédiatement et recevrez le même courriel toutes les heures.

Vérifiez l'état de la minuterie.

	systemctl status mytimer.timer

Voir tous les chronomètres (rimers) en cours d'exécution.

	systemctl list-timers

Éteignez la minuterie.

	sudo systemctl stop myscript.timer

Systemctl active automatiquement la minuterie.

	sudo systemctl enable myscript.timer

Systemctl désactive la minuterie.

	sudo systemctl disable myscript.timer

### Commandes relatives au journal

Si une exception survient, vous devez vérifier le protocole. Voici les commandes que Systemd fournit.

```
# view the entire log
$ sudo journalctl

# view the log for mytimer.timer
$ sudo journalctl -u mytimer.timer

# view the logs for mytimer.timer and mytimer.service
$ sudo journalctl -u mytimer

# view the latest log from the end
$ sudo journalctl -f

# view the log for mytimer.timer from the end
$ journalctl -f -u timer.timer
```

## Liens


* [How to Use Systemd Timers](https://jason.the-graham.com/2013/03/06/how-to-use-systemd-timers/), by Jason Graham
* [Using systemd as a better cron](https://medium.com/horrible-hacks/using-systemd-as-a-better-cron-a4023eea996d), by luqmaan
* [Getting started with systemd](https://coreos.com/os/docs/latest/getting-started-with-systemd.html), by CoreOS
* [systemd/Timers](https://wiki.archlinux.org/index.php/Systemd/Timers), by ArchWiki
* [Understanding Systemd Units and Unit Files](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files), by Justin Ellingwood

* [systemd/cron (archlinux fr)](https://wiki.archlinux.fr/Systemd/cron)
* [Systemd timer pour les Cro(o)ners](https://www.cerenit.fr/blog/systemd-timer-pour-les-crooners/)
* [À la découverte de systemd-timer](https://ungeek.fr/systemd-timer/)
* [Systemd Timers for Scheduling Tasks](https://fedoramagazine.org/systemd-timers-for-scheduling-tasks/)
* [Create a Systemd Service to Send Automatic Emails When Arch Linux Restarts](https://www.lisenet.com/2014/create-a-systemd-service-to-send-automatic-emails-when-arch-linux-restarts/)
* [How to Use Systemd Timers as a Cron Replacement](https://www.maketecheasier.com/use-systemd-timers-as-cron-replacement/)
