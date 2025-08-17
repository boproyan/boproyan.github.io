+++
title = 'Alpine Linux - Guide d'écriture de service OpenRC'
date = 2025-04-20 06:45:00
categories = ['alpine']
+++
*Ce document s'adresse aux développeurs ou aux packagers pour 
écrire des scripts de service OpenRC, soit pour leurs propres projets, soit pour
les packages qu'ils gèrent. Il contient des conseils, des suggestions et des astuces*

* [OpenRC Handbook:X86/Working/Initscripts](https://wiki.gentoo.org/wiki/Handbook:X86/Working/Initscripts)

## Syntaxe des scripts de service

Les scripts de service sont des scripts shell. OpenRC vise à n'utiliser que les scripts standardisés.
Sous-ensemble POSIX sh pour des raisons de portabilité. L'interpréteur par défaut (au moment de la compilation)
toggle) est `/bin/sh`, donc utiliser par exemple mksh n'est pas un problème.

OpenRC a été testé avec busybox sh, ash, dash, bash, mksh, zsh et éventuellement
d'autres. L'utilisation de busybox sh a été difficile car il remplace les commandes par
des éléments intégrés qui n'offrent pas les fonctionnalités attendues.

L'interpréteur pour les scripts de service est `#!/sbin/openrc-run`.
Ne pas utiliser cet interpréteur interrompra l'utilisation des dépendances et n'est pas
pris en charge. (iow : si vous insistez pour utiliser `#!/bin/sh`, vous êtes seul)

Une fonction « depend » déclare les dépendances de ce script de service.
Tous les scripts doivent avoir des fonctions de start/stop/status, mais les valeurs par défaut sont
fournies et doivent être utilisées à moins que vous n'ayez une très bonne raison de ne pas le faire,
utiliser les.

Des fonctions supplémentaires peuvent être ajoutées facilement :

```
extra_commands="checkconfig"
checkconfig() {
	doSomething
}
```

Cela exporte la fonction checkconfig afin que `/etc/init.d/someservice
checkconfig` sera disponible et il exécute « simplement » cette fonction.

Bien que les commandes définies dans `extra_commands` soient toujours disponibles, les commandes
défini dans `extra_started_commands` ne fonctionnera que lorsque le service est démarré
et ceux définis dans `extra_stopped_commands` ne fonctionneront que lorsque le service est
arrêté. Ceci peut être utilisé pour implémenter un rechargement progressif et des opérations similaires.

L'ajout d'une fonction de redémarrage ne fonctionnera pas, il s'agit d'une décision de conception au sein de
OpenRC. Étant donné qu'il peut y avoir des dépendances (par exemple, network -> Apache),
la fonction de redémarrage ne fonctionnera généralement pas.
restart est mappé en interne à `stop()` + `start()` (plus la gestion des dépendances).
Si un service doit se comporter différemment lors de son redémarrage ou
démarré ou arrêté, il doit tester la variable `$RC_CMD`, par exemple :

```
[ "$RC_CMD" = restart ] && do_something
```

## La fonction Depend

Cette fonction déclare les dépendances d'un script de service.
détermine l'ordre dans lequel les scripts de service démarrent.

```
depend() {
	need net
	use dns logger netmount
	want coolservice
}
```

`need` déclare une dépendance dure - net doit toujours être démarré avant ce que fait le service

`use` est une dépendance logicielle - si DNS, logger ou netmount est dans ce niveau d'exécution
	démarrez-le avant, mais cela ne nous dérange pas s'il n'est pas dans ce niveau d'exécution.
	
`want` est entre le besoin et l'utilisation - essayez de démarrer coolservice si c'est le cas
	installé sur le système, qu'il soit ou non dans le
	niveau d'exécution, mais nous ne nous soucions pas de savoir s'il démarre.

`before` déclare que nous devons être démarrés avant un autre service

`after` déclare que nous devons être démarrés après un autre service, sans
	créer une dépendance (donc en appelant stop les deux sont indépendants)

`provide` permet à plusieurs implémentations de fournir un type de service, par exemple :
	`provide cron` est défini dans tous les daemons cron, donc n'importe lequel d'entre eux a démarré
	satisfait une dépendance cron

`keyword` permet des remplacements spécifiques à la plate-forme, par exemple `keyword -lxc` rend cela
	Script de service noop dans les conteneurs LXC. Utile pour des éléments comme les keymaps,
	chargement de modules, etc. qui sont spécifiques à la plate-forme ou non disponibles
	dans les conteneurs/virtualisation/...

## Les fonctions par défaut

Tous les scripts de service sont supposés avoir les fonctions suivantes :

```
start()
stop()
status()
```

Il existe des implémentations par défaut dans `lib/rc/sh/openrc-run.sh` - cela permet de très
Scripts de service compacts. Ces fonctions peuvent être remplacées par un service script, si
nécessaire.

Les fonctions par défaut supposent que les variables suivantes sont définies dans le scénario du service
:

```
command=
command_args=
pidfile=
```

Ainsi, les « plus petits » scripts de service peuvent comporter une demi-douzaine de lignes.

## N'écrivez pas vos propres fonctions de démarrage/arrêt

OpenRC est capable d'arrêter et de démarrer la plupart des daemons en fonction des
informations que vous lui donnez. Pour un daemon bien élevé qui
s'exécute en arrière-plan et écrit son propre fichier PID par défaut, 
les variables OpenRC suivantes sont probablement tout ce dont vous aurez besoin :

  * commande
  * command_args
  * pidfile

Compte tenu de ces trois informations, OpenRC pourra démarrer
et arrêter le daemon tout seul. Ce qui suit est tiré d'un
script de service [OpenNTPD](http://www.openntpd.org/) :

```sh
command="/usr/sbin/ntpd"

# The special RC_SVCNAME variable contains the name of this service.
pidfile="/run/${RC_SVCNAME}.pid"
command_args="-p ${pidfile}"
```

Si le daemon s'exécute au premier plan par défaut mais dispose d'options pour
l'arrière-plan lui-même et pour créer un fichier PID, vous aurez également besoin

  * command_args_background

Cette variable doit contenir les indicateurs nécessaires pour mettre en arrière-plan votre daemon
et lui faire écrire un fichier PID. Prenons par exemple le
extrait suivant d'un script de service [NRPE](https://github.com/NagiosEnterprises/nrpe) :

```sh
command="/usr/bin/nrpe"
command_args="--config=/etc/nagios/nrpe.cfg"
command_args_background="--daemon"
pidfile="/run/${RC_SVCNAME}.pid"
```

Étant donné que NRPE s'exécute en tant que *root* par défaut, il n'a besoin d'aucune autorisation spéciale pour écrire dans `/run/nrpe.pid`. OpenRC se charge du démarrage et de l'arrêt du daemon avec les arguments appropriés, en passant l'indicateur `--daemon` au démarrage pour forcer NRPE en arrière-plan (NRPE sait écrire son propre fichier PID).

Et si le daemon n'était pas si sage ? Et s'il ne savait pas  
Comment se mettre en arrière-plan ou créer un fichier PID ? Si ni l'un ni l'autre ne fonctionne,
puis utiliser,

  * command_background=true

qui passera en plus `--make-pidfile` à start-stop-daemon,
ce qui lui permet de créer le `$pidfile` pour vous (plutôt que le daemon, étant lui-même responsable de la création du fichier PID).

Si votre daemon ne sait pas comment changer son propre utilisateur ou groupe, alors
vous pouvez dire à start-stop-daemon de le lancer en tant qu'utilisateur non privilégié
avec

  * command_user="user:group"

Si votre daemon doit s'exécuter avec des données héritables, ambiantes et capacités de délimitation spécifiques
, vous pouvez alors dire à start-stop-daemon de se lancer
avec ça

  * capabilities="cap-list"

Le format est identique à celui de cap_iab(3). (Uniquement sous Linux)

Par exemple, pour démarrer le daemon avec un environnement ambiant et héritable
`cap_chown`, mais sans `cap_setpcap` dans l'ensemble de délimitation, utilisez
la valeur suivante :

```sh
capabilities="^cap_chown,!cap_setpcap"
```

Enfin, si votre daemon se lance toujours en arrière-plan mais ne parvient pas à
créer un fichier PID, alors votre seule option est d'utiliser

  * procname

Avec `procname`, OpenRC essaiera de trouver le daemon en cours d'exécution en
correspondant au nom de son processus. Ce n'est pas très fiable, mais les daemons
ne devraient pas s'exécuter en arrière-plan sans créer un fichier PID dans le
première place. L'exemple suivant fait partie de la [CA NetConsole
script de service [Daemon](https://oss.oracle.com/projects/cancd/) :

```sh
command="/usr/sbin/cancd"
command_args="-p ${CANCD_PORT}
              -l ${CANCD_LOG_DIR}
              -o ${CANCD_LOG_FORMAT}"
command_user="cancd"

# cancd daemonizes itself, but doesn't write a PID file and doesn't
# have an option to run in the foreground. So, the best we can do
# is try to match the process name when stopping it.
procname="cancd"
```

## Récapitulatif

Pour récapituler, par ordre de préférence :

  1. Si le daemon s'exécute en arrière-plan et crée son propre fichier PID, utilisez
     `pidfile`.
  2. Si le daemon ne s'exécute pas en arrière-plan (ou n'a pas la possibilité de s'exécuter
     au premier plan) et ne crée pas de fichier PID, puis utilisez
     `command_background=true` et `pidfile`.
  3. Si le daemon s'exécute en arrière-plan et ne crée pas de fichier PID,
     utilisez « procname » au lieu de « pidfile ». Cependant, si votre daemon possède l'option pour s'exécuter au premier plan, alors vous devriez plutôt le faire
     (ce serait le cas dans l'élément précédent).
  4. Le dernier cas, où le daemon ne se met pas en arrière-plan mais
     crée un fichier PID, mais ça n'a pas beaucoup de sens. S'il y a un moyen
     pour désactiver le fichier PID du daemon (ou pour l'écrire directement dans le
     "garbage"), puis faites cela et utilisez `command_background=true`.

## Rechargement de la configuration de votre daemon

De nombreux daemons rechargeront leurs fichiers de configuration en réponse à une
signal. Supposons que votre daemon recharge sa configuration en réponse
à un « SIGHUP ». Il est possible d'ajouter une nouvelle commande « reload » à votre
script de service qui effectue cette action. Commencez par indiquer au service
script sur la nouvelle commande.

```sh
extra_started_commands="reload"
```

Nous utilisons `extra_started_commands` par opposition à `extra_commands` car
l'action « reload » n'est valide que lorsque le daemon est en cours d'exécution (c'est-à-dire
est démarré). Maintenant, le daemon start-stop peut être utilisé pour envoyer le signal au processus approprié (en supposant que vous ayez défini par ailleurs la variable « pidfile » ) :

```sh
reload() {
  ebegin "Reloading ${RC_SVCNAME}"
  start-stop-daemon --signal HUP --pidfile "${pidfile}"
  eend $?
}
```

## Ne pas restart ou reload avec une configuration cassée

Souvent, les utilisateurs démarrent un daemon, effectuent des modifications de configuration et
puis essaient de redémarrer le daemon. Si la configuration récente a changé,
contient une erreur, le résultat sera que le daemon est arrêté mais ne peut plus être redémarré (en raison d'une erreur de configuration). C'est possible d'éviter cette situation avec une fonction qui vérifie les erreurs de configuration avec une combinaison de `start_pre` et
hooks `stop_pre`.

```sh
checkconfig() {
  # However you want to check this...
}

start_pre() {
  # If this isn't a restart, make sure that the user's config isn't
  # busted before we try to start the daemon (this will produce
  # better error messages than if we just try to start it blindly).
  #
  # If, on the other hand, this *is* a restart, then the stop_pre
  # action will have ensured that the config is usable and we don't
  # need to do that again.
  if [ "${RC_CMD}" != "restart" ] ; then
    checkconfig || return $?
  fi
}

stop_pre() {
  # If this is a restart, check to make sure the user's config
  # isn't busted before we stop the running daemon.
  if [ "${RC_CMD}" = "restart" ] ; then
      checkconfig || return $?
  fi
}
```

Pour éviter un *reload* avec une configuration cassée, restez simple :

```sh
reload() {
  checkconfig || return $?
  ebegin "Reloading ${RC_SVCNAME}"
  start-stop-daemon --signal HUP --pidfile "${pidfile}"
  eend $?
}
```

## Les fichiers PID ne doivent être accessibles en écriture que par root

Les fichiers PID doivent être accessibles en écriture uniquement par *root*, ce qui signifie en plus
qu'ils doivent résider dans un répertoire appartenant à la racine. Ce répertoire est
normalement /run sous Linux et /var/run sous d'autres systèmes d'exploitation.

Certains daemons s'exécutent en tant que compte utilisateur non privilégié et créent leur PID
fichiers (en tant qu'utilisateur non privilégié) dans un chemin comme
/var/run/foo/foo.pid. Cette faille peut généralement être exploitée par des personnes non privilégiées.
l'utilisateur pour tuer les processus *root*, car lorsqu'un service est arrêté, *root*
envoie généralement un SIGTERM au contenu du fichier PID (qui sont
contrôlé par l'utilisateur non privilégié). Le principal signe d'alerte est d'utiliser `checkpath` pour définir la propriété du répertoire contenant le fichier PID. Par exemple,

```sh
# MAUVAIS MAUVAIS MAUVAIS MAUVAIS MAUVAIS MAUVAIS MAUVAIS MAUVAIS
start_pre() {
  # Assurez-vous que le répertoire pidfile est accessible en écriture par l'utilisateur/groupe foo.
  checkpath --directory --mode 0700 --owner foo:foo "/var/run/foo"
}
# MAUVAIS MAUVAIS MAUVAIS MAUVAIS MAUVAIS MAUVAIS MAUVAIS MAUVAIS
```

Si l'utilisateur *foo* possède `/var/run/foo`, alors il peut mettre ce qu'il veut
dans le fichier `/var/run/foo/foo.pid`. Même si *root* possède le fichier PID, l'utilisateur peut le supprimer et le remplacer par le sien. Pour des raisons de sécurité, le fichier PID doit être créé en tant que *root* et résider dans un répertoire appartenant à la racine. Si votre daemon est responsable du fork, et écrit son propre fichier PID, mais le fichier PID appartient toujours au
utilisateur d'exécution non privilégié, vous pouvez alors avoir un problème en amont.

Une fois le fichier PID créé en tant que *root* (avant de supprimer les privilèges), il peut être écrit directement sur un fichier appartenant à *root*. Par exemple, le daemon *foo* pourrait écrire
`/var/run/foo.pid`. Aucun appel à checkpath n'est nécessaire. 

Remarque : il n'y a rien de techniquement mal à utiliser une structure de répertoire comme
`/var/run/foo/foo.pid`, tant que *root* possède le fichier PID et le
répertoire le contenant.

Idéalement , votre script de service
sera intégré en amont et le système de construction déterminera le
répertoire approprié pour le fichier PID. Par exemple,

```sh
pidfile="@piddir@/${RC_SVCNAME}.pid"
```

Un bon exemple de cela est le [service script principal Nagios
](https://github.com/NagiosEnterprises/nagioscore/blob/HEAD/openrc-init.in),
où le chemin complet vers le fichier PID est spécifié au moment de la construction.

## Ne laissez pas l'utilisateur contrôler l'emplacement du fichier PID

C'est généralement une erreur de laisser l'utilisateur final contrôler le fichier PID
localisation via une variable conf.d, pour plusieurs raisons :

  1. Lorsque le chemin du fichier PID est contrôlé par l'utilisateur, vous devez
     s'assurer que son répertoire parent existe et est accessible en écriture. Ceci
     ajoute du code inutile au script de service.

  2. Si le chemin du fichier PID change pendant l'exécution du service, alors
     vous ne pourrez pas arrêter le service.

  3. Il est préférable de déterminer le répertoire qui doit contenir le fichier PID
     par le système de build en amont (voir « Amonter vos scripts de service »).
     Sous Linux, l'emplacement privilégié est actuellement « /run ». Autres systèmes
     utilisez toujours `/var/run`, cependant, et un script `./configure` est le
     le meilleur endroit pour décider lequel vous voulez.

  4. De toute façon, personne ne se soucie de l'emplacement du fichier PID.

Étant donné que les noms de service OpenRC doivent être uniques, une valeur de

```sh
pidfile="/var/run/${RC_SVCNAME}.pid"
```

garantit que votre fichier PID a un nom unique.

## En amont de vos scripts de service (pour les packagers)

L'emplacement idéal pour un script de service OpenRC est **en amont**. Tout comme les
services systemd, le meilleur endroit pour cela est en amont. Pourquoi ? Pour
deux raisons.  
Premièrement, l'avoir en amont signifie qu'il n'y a qu'un seul
source faisant autorité pour les améliorations.  
Deuxièmement, quelques pistes dans chaque  
Les scripts de service dépendent des indicateurs transmis au système de build.
exemple,

```sh
command=/usr/bin/foo
```

dans un système de construction basé sur Autotools devrait vraiment être

```sh
command=@bindir@/foo
```

afin que la valeur utilisateur de `--bindir` soit respectée. Si vous conservez le
script de service dans le référentiel de votre propre distribution, vous devez alors
gardez vous-même le chemin de commande et le package synchronisés, et ce n'est pas
amusant.

## Méfiez-vous des dépendances « need net »

Il y a deux choses que vous devez savoir sur les dépendances « need net » :

  1. Ils ne sont pas satisfaits par l'interface de bouclage, ils ont donc besoin d'un réseau. et
     nécessite qu'une *autre* interface soit opérationnelle.

  2. En fonction de la valeur de `rc_depend_strict` dans `rc.conf`, le
     « need net » sera satisfait lorsque *n'importe quel* interface non-loopback est active, ou lorsque *toutes* les interfaces non-loopback sooient actives.

Le premier élément signifie que « need net » est incorrect pour les daemons qui sont
heureux avec « 0.0.0.0 », et le deuxième point signifie que « need net » est
incorrect pour les daemons qui ont besoin d'un élément particulier (par exemple, l'interface WAN)
. Nous allons considérer les deux utilisateurs les plus courants de « need net » ;
clients réseau qui accèdent à certaines ressources réseau et serveurs réseau
qui les fournissent.

### Clients réseau

Les clients réseau souhaitent généralement que l'interface WAN soit active. Cela peut
vous inciter à dépendre de l'interface WAN ; mais d'abord, vous devriez demander
Posez-vous une question : est-ce que quelque chose de mal se produit si l'interface WAN est
indisponible ? Autrement dit, si l'administrateur souhaite désactiver
Le WAN, faut-il arrêter le service ? La réponse à la question est généralement
est « non », et dans ce cas, vous devriez renoncer complètement  à la dépendance « net »

### Serveurs réseau

Les serveurs réseau sont généralement plus faciles à gérer que leurs clients
homologues. La plupart des daemons serveurs écoutent sur « 0.0.0.0 » (toutes les adresses)
par défaut, et sont donc satisfaits d'avoir l'interface de bouclage
présent et opérationnel. OpenRC est livré avec le service de bouclage dans le
niveau d'exécution *boot* , et par conséquent la plupart des daemons de serveur ne nécessitent pas d'autres dépendances du réseau.

### En fonction d'une interface particulière

Si vous devez dépendre d'une interface particulière, ce n'est généralement pas le cas.
Il est facile de déterminer par programmation quelle est cette interface.
par exemple, si votre daemon *sshd* écoute sur `192.168.1.100` (plutôt que
`0.0.0.0`), alors vous avez deux problèmes :

  1. Analyser `sshd_config` pour comprendre cela ; et

  2. Déterminer quel nom de service réseau correspond l'interface pour `192.168.1.100`.

C'est généralement une mauvaise idée d'analyser les fichiers de configuration de votre service
scripts, mais le deuxième problème est le plus difficile. Au lieu de cela,
L'approche robuste (c'est-à-dire la plus paresseuse) consiste à demander à l'utilisateur de spécifier
dépendance lorsqu'il modifie sshd_config. Inclure quelque chose
comme ce qui suit dans le fichier de configuration du service,

```sh
# Spécifiez le service réseau qui correspond au paramètre « bind »
# dans votre fichier de configuration. Par exemple, si vous vous connectez à 127.0.0.1,
# cela doit être défini sur « loopback » qui fournit l'interface de bouclage.
rc_need="loopback"
```

Il s'agit d'une valeur par défaut raisonnable pour les daemons qui sont satisfaits de « 0.0.0.0 »,
mais permet à l'utilisateur de spécifier autre chose, comme `rc_need="net.wan"` si
il en a besoin. Il incombe à l'utilisateur de déterminer la solution appropriée au service chaque fois qu'il modifie le fichier de configuration du daemon.

