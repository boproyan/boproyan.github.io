+++
title = 'Arrêt en douceur des machines virtuelles lorsque la machine hôte est bloquée, mise hors tension ou redémarrée'
date = 2022-09-04 00:00:00 +0100
categories = ['virtuel']
+++
*Le service libvirt-guests possède des paramètres qui peuvent être configurés pour s'assurer que l'invité est arrêté correctement.* 

**libvirt-guests** fait partie de l'installation de libvirt et est installé par défaut. Ce service enregistre automatiquement les invités sur le disque lorsque l'hôte s'éteint, et les restaure dans leur état antérieur à l'arrêt lorsque l'hôte redémarre. Par défaut, ce paramètre est défini pour suspendre l'invité. Si vous voulez que l'invité soit arrêté, vous devrez modifier l'un des paramètres du fichier de configuration libvirt-guests.  
`LE SERVICE libvirt-guests est INACTIF PAR DEFAUT`{: .prompt-info }

Modifier les paramètres du service **libvirt-guests** pour permettre l'arrêt gracieux des invités  

La procédure décrite ici permet l'arrêt en douceur des machines virtuelles invitées lorsque la machine physique hôte est bloquée, mise hors tension ou doit être redémarrée.([Manipulating the libvirt-guests Configuration Settings](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/virtualization_administration_guide/sub-sect-shutting_down_rebooting_and_force_shutdown_of_a_guest_virtual_machine-manipulating_the_libvirt_guests_configuration_settings))
{: .prompt-info }

Ouvrir ou créer le fichier de configuration `/etc/conf.d/libvirt-guests`  

```
# URIs to check for running guests
# example: URIS='default xen:/// vbox+tcp://host/system lxc:///'
#URIS=default

# action taken on host boot
# - start   all guests which were running on shutdown are started on boot
#           regardless on their autostart settings  
# - ignore  libvirt-guests init script won't start any guest on boot, however, 
#           guests marked as autostart will still be automatically started by 
#           libvirtd 
ON_BOOT=ignore  

# Number of seconds to wait between each guest start. Set to 0 to allow   
# parallel startup.
#START_DELAY=0

# action taken on host shutdown
# - suspend   all running guests are suspended using virsh managedsave
# - shutdown  all running guests are asked to shutdown. Please be careful with
#             this settings since there is no way to distinguish between a
#             guest which is stuck or ignores shutdown requests and a guest
#             which just needs a long time to shutdown. When setting
#             ON_SHUTDOWN=shutdown, you must also set SHUTDOWN_TIMEOUT to a
#             value suitable for your guests.
ON_SHUTDOWN=shutdown

# If set to non-zero, shutdown will suspend guests concurrently. Number of
# guests on shutdown at any time will not exceed number set in this variable.
#PARALLEL_SHUTDOWN=0

# Number of seconds we're willing to wait for a guest to shut down. If parallel
# shutdown is enabled, this timeout applies as a timeout for shutting down all
# guests on a single URI defined in the variable URIS. If this is 0, then there
# is no time out (use with caution, as guests might not respond to a shutdown
# request). The default value is 300 seconds (5 minutes).
SHUTDOWN_TIMEOUT=240

# If non-zero, try to bypass the file system cache when saving and
# restoring guests, even though this may give slower operation for
# some file systems.
#BYPASS_CACHE=0
```

**libvirt-guests.service**

* Configuré afin qu'il arrête les machines virtuelles invitées lorsque l'hôte s'arrête.
* défini le délai d'expiration sur moins de 5 minutes

**Définition des options**

* **URIS** - vérifie les connexions spécifiées pour un invité en cours d'exécution. Le **paramètre par défaut** fonctionne de la même manière que virsh lorsqu'aucun URI explicite n'est défini. En outre, on peut définir explicitement l'URI à partir de /etc/libvirt/libvirt.conf. Il convient de noter que lors de l'utilisation du paramètre par défaut du fichier de configuration libvirt, aucun sondage ne sera utilisé.
* **ON_BOOT** - spécifie l'action à effectuer sur les invités lorsque l'hôte démarre. L'option **start** démarre tous les invités qui étaient en cours d'exécution avant l'arrêt, quels que soient leurs paramètres de démarrage automatique. L'option ignore ne démarrera pas l'invité en cours d'exécution au démarrage, cependant, tout invité marqué comme démarrant automatiquement sera toujours démarré automatiquement par libvirtd.
* L'option **START_DELAY** - définit un intervalle de temps entre le démarrage des invités. Cette période de temps est définie en secondes. Utilisez le paramètre 0 pour vous assurer qu'il n'y a pas de délai et que tous les invités sont démarrés simultanément.
* **ON_SHUTDOWN** - spécifie l'action prise lorsqu'un hôte s'arrête. Les options qui peuvent être définies comprennent : **suspend** qui suspend tous les invités en cours d'exécution en utilisant **virsh managedsave** et **shutdown** qui arrête tous les invités en cours d'exécution. Il est préférable d'être prudent avec l'utilisation de l'option shutdown car il n'y a aucun moyen de distinguer entre un invité qui est bloqué ou qui ignore les demandes d'arrêt et un invité qui a juste besoin d'un temps plus long pour s'arrêter. Lorsque vous définissez l'option **ON_SHUTDOWN=shutdown**, vous devez également définir **SHUTDOWN_TIMEOUT** à une valeur appropriée pour les invités.
* **PARALLEL_SHUTDOWN** Indique que le nombre d'invités à l'arrêt à tout moment ne dépassera pas le nombre défini dans cette variable et que les invités seront suspendus simultanément. Si la valeur est 0, les clients ne sont pas arrêtés simultanément.
* Nombre de secondes à attendre pour qu'un invité s'arrête. Si **SHUTDOWN_TIMEOUT** est activé, ce délai s'applique comme un délai d'arrêt de tous les invités sur un seul URI défini dans la variable URIS. Si **SHUTDOWN_TIMEOUT** a la valeur **0**, il n'y a pas de délai d'attente (à utiliser avec prudence, car les invités peuvent ne pas répondre à une demande d'arrêt). La valeur par défaut est de 300 secondes (5 minutes).
* **BYPASS_CACHE** peut avoir 2 valeurs, 0 pour désactiver et 1 pour activer. Si cette option est activée, le cache du système de fichiers sera contourné lors de la restauration des invités. Notez que ce paramètre peut avoir un effet sur les performances et peut entraîner un fonctionnement plus lent pour certains systèmes de fichiers. 

Activer et démarrer le service **libvirt-guests**

    sudo systemctl enable libvirt-guests
    sudo systemctl start libvirt-guests

Etat

    sudo systemctl status libvirt-guests

```
     Loaded: loaded (/usr/lib/systemd/system/libvirt-guests.service; enabled; preset: disabled)
     Active: active (exited) since Sat 2022-09-03 09:18:26 CEST; 1min 19s ago
       Docs: man:libvirt-guests(8)
             https://libvirt.org
   Main PID: 6497 (code=exited, status=0/SUCCESS)
        CPU: 9ms

sept. 03 09:18:26 archyan systemd[1]: Starting Suspend/Resume Running libvirt Guests...
sept. 03 09:18:26 archyan systemd[1]: Finished Suspend/Resume Running libvirt Guests.
```

`Ne redémarrez pas le service car cela entraînerait l'arrêt de tous les domaines en cours d'exécution.`{: .prompt-warning } 

Service qui permet d'exécuter des commandes avant l'arrêt complet de la machine hôte , voir le lien suivant pour un exemple de script 
{: .prompt-info }


Comment configurer un serveur hôte afin que, lorsqu'il est arrêté ([Run a script on virtual machines when the host is shut down](https://www.brianlinkletter.com/2019/09/run-a-script-on-virtual-machines-when-the-host-is-shut-down/)):

* il exécute un script qui exécute des commandes sur toutes les machines virtuelles en cours d'exécution avant que l'hôte n'essaie de les arrêter. 
* il attend que le script termine la configuration des machines virtuelles avant de poursuivre le processus d'arrêt, d'arrêter les machines virtuelles et éventuellement de les éteindre.

Le service libvirt-guests est déjà démarré et activé, et il est configuré de manière appropriée (`ON_SHUTDOWN=shutdown`). 

Créer un nouveau service Systemd nommé **graceful-shutdown** qui exécute un script **stopvm.sh** lorsque le système hôte s'arrête, mais avant que *Libvirt* n'arrête les machines virtuelles.

    sudo nano /etc/systemd/system/graceful-shutdown.service

```
[Unit]
Description=Graceful Shutdown Service
DefaultDependencies=no
Requires=libvirt-guests.service
After=libvirt-guests.service

[Service]
Type=oneshot
RemainAfterExit=true
ExecStop=/usr/local/bin/stopvm.sh

[Install]
WantedBy=multi-user.target
```

Le service "Graceful Shutdown" obligera Systemd à exécuter un script shell Linux, `/usr/local/bin/stopvm.sh` avant que Systemd n'arrête le service *libvirt-guests*.  
Le type de service est **oneshot** , Systemd attendra que le script se termine avant de poursuivre son processus d'arrêt standard.

Le service "Graceful Shutdown" s'appuie sur le service *libvirt-guests*  
Si le service *libvirt-guests* n'est pas en cours d'exécution, le service "Graceful Shutdown" ne lancera pas le script *stopvm.sh* et Libvirt éteindra les machines virtuelles, ce qui peut entraîner une corruption des processus ou des bases de données s'exécutant sur ces machines virtuelles.
{: .prompt-warning }

Exemple de script "stopvm.sh" pour arrêter une VM

    sudo nano /usr/local/bin/stopvm.sh

```bash
#!/bin/bash
# Shut down the VM

virsh shutdown yunobulls
exit 0
```

Le rendre exécutable

    sudo chmod +x /usr/local/bin/stopvm.sh

Activer et démarrer le service "Graceful Shutdown"

```bash
sudo systemctl enable graceful-shutdown
sudo systemctl start graceful-shutdown
```
