+++
title = 'GoLang exécuter un binaire Go en tant que service systemd'
date = 2020-04-30 00:00:00 +0100
categories = go
+++
![go](go-logo.png){:width="70"}

Article original : [GoLang: Running a Go binary as a systemd service on Ubuntu 16.04](https://fabianlee.org/2017/05/21/golang-running-a-go-binary-as-a-systemd-service-on-ubuntu-16-04/)

Le langage Go  avec sa simplicité, sa prise en charge de la concurrence, son riche écosystème de packages et sa capacité à compiler en un seul binaire est une solution intéressante pour écrire des services sur Ubuntu.

Cependant, le langage Go ne fournit pas nativement un moyen fiable de se démoniser. Dans cet article, je décrirai comment prendre quelques programmes de langue Go simples et les exécuter en utilisant un   fichier de service systemd qui les démarre au démarrage sur Ubuntu 16.04/Debian 10.

Si vous n'avez pas installé Go sur Ubuntu, lisez d'abord [mon article ici](https://fabianlee.org/2017/05/13/golang-installing-the-go-programming-language-on-ubuntu-14-04/) .


## Considérations de service

Avant de commencer, considérons les problèmes que nous devons résoudre lorsque nous passons de l'exécution d'une tâche de premier plan à celle d'un démon.

Tout d'abord, l'application doit s'exécuter en arrière-plan. En raison des interactions complexes avec le pool de threads Go et les autorisations forks / drop [ 1 , 2 , 3 , 4 ], l'exécution d'un simple nohup ou double fork du programme n'est pas une option - mais à vrai dire, cela ne devrait pas être de  toute façon étant donné l'ensemble riche des alternatives disponibles aujourd'hui.

Il existe de nombreux systèmes de contrôle de processus tels que Supervisor  et monit , mais avec Ubuntu 16.04/Debian 10, nous pouvons utiliser le systemd  qui est le système d'initialisation par défaut.

Les processus d'arrière-plan sont détachés du terminal, mais peuvent toujours recevoir des signaux, nous aimerions donc un moyen de les capturer afin de pouvoir quitter sans problème si nécessaire.

Pour des raisons de sécurité, nous devons faire en sorte que le démon s'exécute en tant que son propre utilisateur afin que nous puissions contrôler exactement les privilèges et les autorisations d'accès aux fichiers.

Ensuite, nous devons nous assurer que la journalisation est disponible. Bien que 'journalctl' fournisse les journaux, ce que nous voulons vraiment, c'est que les journaux soient disponibles à l'emplacement standard `/var/log/<service>`   
Nous allons donc dire à systemd d'envoyer à syslog, puis demander à syslog d'écrire nos fichiers sur le disque.

Enfin, le service doit faire partie du processus de démarrage, afin qu'il démarre automatiquement après le redémarrage.

## SleepService au premier plan

Commençons par un simple programme Go qui entre dans une boucle infinie, imprimant «hello world» sur le terminal avec un retard aléatoire entre les deux. La logique réelle du programme est mise en évidence ci-dessous, le reste est configuré pour capter tous les signaux reçus.

```go
package main

import (
        "time"
        "log"
        "flag"
        "math/rand"
        "os"
        "os/signal"
        //"syscall"
)

func main() {

        // load command line arguments
        name := flag.String("name","world","name to print")
        flag.Parse()

        log.Printf("Starting sleepservice for %s",*name)

        // setup signal catching
        sigs := make(chan os.Signal, 1)

        // catch all signals since not explicitly listing
        signal.Notify(sigs)
        //signal.Notify(sigs,syscall.SIGQUIT)

        // method invoked upon seeing signal
        go func() {
          s := <-sigs
          log.Printf("RECEIVED SIGNAL: %s",s)
          AppCleanup()
          os.Exit(1)
        }()

        // infinite print loop
        for {
          log.Printf("hello %s",*name)

          // wait random number of milliseconds
          Nsecs := rand.Intn(3000)
          log.Printf("About to sleep %dms before looping again",Nsecs)
          time.Sleep(time.Millisecond * time.Duration(Nsecs))
        }

}

func AppCleanup() {
        log.Println("CLEANUP APP BEFORE EXIT!!!")
}
```

D'abord, nous l'exécuterons au premier plan en tant qu'utilisateur actuel. Voici les commandes pour Linux:

```bash
mkdir -p $GOPATH/src/sleepservice
cd $GOPATH/src/sleepservice
wget https://raw.githubusercontent.com/fabianlee/blogcode/master/golang/sleepservice/sleepservice.go
go get
go build
./sleepservice
```

Ce qui devrait produire une sortie qui ressemble à quelque chose comme ci-dessous qui se termine lorsque vous faites Ctrl-C sur l'exécution:

```bash
2017/05/20 13:41:15 Starting sleepservice for world
2017/05/20 13:41:15 hello world
2017/05/20 13:41:15 About to sleep 2081ms before looping again
2017/05/20 13:41:17 hello world
2017/05/20 13:41:17 About to sleep 1887ms before looping again
2017/05/20 13:41:19 hello world
2017/05/20 13:41:19 About to sleep 1847ms before looping again
^C2017/05/20 13:41:20 RECEIVED SIGNAL: interrupt
2017/05/20 13:41:20 CLEANUP APP BEFORE EXIT!!!
```

Notez que la demande ne s'est pas arrêtée brutalement. Il a détecté le Control-C ( signal SIGINT ), effectué un nettoyage personnalisé de l'application, puis est sorti.

Si vous deviez démarrer sleepservice dans un terminal, accédez à un autre terminal et envoyez divers signaux au processus avec killall:

```bash
$ sudo killall --signal SIGTRAP sleepservice
$ sudo killall --signal SIGINT sleepservice
$ sudo killall --signal SIGTERM sleepservice
```

Vous verriez l'application refléter ces différents signaux, comme ci-dessous où un SIGTRAP a été envoyé:

```bash
2017/05/20 13:35:23 RECEIVED SIGNAL: trace/breakpoint trap
2017/05/20 13:35:23 CLEANUP APP BEFORE EXIT!!!
```

## SleepService en tant que service systemd

Pour transformer cela en un service pour systemd, nous devons créer un fichier de service d'unité à **/lib/systemd/system/sleepservice.service** comme ci-dessous:

```ini
[Unit]
Description=Sleep service
ConditionPathExists=/home/ubuntu/work/src/sleepservice/sleepservice
After=network.target
 
[Service]
Type=simple
User=sleepservice
Group=sleepservice
LimitNOFILE=1024

Restart=on-failure
RestartSec=10

WorkingDirectory=/home/ubuntu/work/src/sleepservice
ExecStart=/home/ubuntu/work/src/sleepservice/sleepservice --name=foo

# make sure log directory exists and owned by syslog
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /var/log/sleepservice
ExecStartPre=/bin/chown syslog:adm /var/log/sleepservice
ExecStartPre=/bin/chmod 755 /var/log/sleepservice
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=sleepservice
 
[Install]
WantedBy=multi-user.target
```

Les chemins absolus dans «`ConditionPathExists`», «`WorkingDirectory`» et «`ExecStart`» doivent tous être modifiés en fonction de votre environnement. Notez que nous avons demandé à systemd d'exécuter le processus en tant qu'utilisateur «*sleepservice*», nous devons donc également créer cet utilisateur.

Vous trouverez ci-dessous des instructions pour créer l'utilisateur et déplacer le fichier de service de l'unité systemd vers l'emplacement correct:

```bash
cd /tmp
sudo useradd sleepservice -s /sbin/nologin -M
wget https://raw.githubusercontent.com/fabianlee/blogcode/master/golang/sleepservice/systemd/sleepservice.service
sudo mv sleepservice.service /lib/systemd/system/
sudo chmod 755 /lib/systemd/system/sleepservice.service
```

Maintenant, vous devriez pouvoir activer le service, le démarrer, puis surveiller les journaux en suivant le journal systemd:

    sudo systemctl enable sleepservice.service
    sudo systemctl daemon-reload
    sudo systemctl start sleepservice
    sudo journalctl -f -u sleepservice

```
May 21 16:20:43 xenial1 sleepservice[4037]: 2017/05/21 16:20:43 hello foo
May 21 16:20:43 xenial1 sleepservice[4037]: 2017/05/21 16:20:43 About to sleep 1526ms before looping again
May 21 16:20:45 xenial1 sleepservice[4037]: 2017/05/21 16:20:45 hello foo
May 21 16:20:45 xenial1 sleepservice[4037]: 2017/05/21 16:20:45 About to sleep 196ms before looping again
```

Le journal est stocké sous forme de fichier binaire, il ne peut donc pas être suivi directement. Mais nous avons le <u>transfert syslog activé</u> du côté systemd, donc maintenant il s'agit juste de configurer notre serveur syslog.

Pour des instructions complètes sur la [configuration de syslog sur Ubuntu, lisez mon article ici](https://fabianlee.org/2017/05/24/ubuntu-enabling-syslog-on-ubuntu-hosts-and-custom-templates/) . Mais voici des instructions rapides pour Ubuntu 16.04/Debian 10.

Modifiez d'abord «/etc/rsyslog.conf» et décommentez les lignes ci-dessous qui indiquent au serveur d'écouter les messages syslog sur le port 514 / TCP.

```
module(load="imtcp")
input(type="imtcp" port="514")
```

Ensuite, créez «/etc/rsyslog.d/30-sleepservice.conf» avec le contenu suivant:

```
if $programname == 'sleepservice' or $syslogtag == 'sleepservice' then /var/log/sleepservice/sleepservice.log
& stop
```

Redémarrez maintenant le service rsyslog et vous devriez voir l'écouteur syslog sur le port 514, redémarrez le service de sommeil et maintenant vous devriez voir les événements de journal envoyés au fichier toutes les quelques secondes.

```
$ sudo systemctl restart rsyslog
$ netstat -an | grep "LISTEN "
$ sudo systemctl restart sleepservice
$ tail -f /var/log/sleepservice/sleepservice.log

May 21 16:30:12 xenial1 sleepservice[4196]: 2017/05/21 16:30:12 hello foo
May 21 16:30:12 xenial1 sleepservice[4196]: 2017/05/21 16:30:12 About to sleep 2211ms before looping again
May 21 16:30:14 xenial1 sleepservice[4196]: 2017/05/21 16:30:14 hello foo
May 21 16:30:14 xenial1 sleepservice[4196]: 2017/05/21 16:30:14 About to sleep 1445ms before looping again
```

La liste des processus en cours d'exécution montre que le processus s'exécute en tant qu'utilisateur «sleepservice».

```
$ ps -ef | grep sleepservice

sleepse+  4196     1  0 16:29 ?        00:00:00 /home/ubuntu/work/src/sleepservice/sleepservice --name=foo
```

L'arrêt du service montrera que le signal SIGTERM a été envoyé à l'application et qu'il a été nettoyé avant l'arrêt.

```
$ sudo service sleepservice stop

$ tail -n2 /var/log/sleepservice/sleepservice.log

May 21 16:32:30 xenial1 sleepservice[4196]: 2017/05/21 16:32:30 RECEIVED SIGNAL: terminated
May 21 16:32:30 xenial1 sleepservice[4196]: 2017/05/21 16:32:30 CLEANUP APP BEFORE EXIT!!!
```

Mais, si vous deviez envoyer un signal SIGINT (interruption), notez que le service redémarre à cause du «Restart = on-failure» que nous avons indiqué dans le fichier de service).

```
$ sudo killall -s SIGNINT sleepservice

$ tail -n 10 -f /var/log/sleepser

May 21 16:34:59 xenial1 sleepservice[4231]: 2017/05/21 16:34:59 RECEIVED SIGNAL: interrupt
May 21 16:34:59 xenial1 sleepservice[4231]: 2017/05/21 16:34:59 CLEANUP APP BEFORE EXIT!!!
May 21 16:35:09 xenial1 sleepservice[4255]: 2017/05/21 16:35:09 hello foo
May 21 16:35:09 xenial1 sleepservice[4255]: 2017/05/21 16:35:09 About to sleep 2081ms before looping again
```

Par défaut, le service sera exécuté au démarrage par le paramètre «WantedBy = multi-user.target», et il y a un lien sous «/etc/systemd/system/multi-user.target.wants/».

## EchoService au premier plan

Passons maintenant à la construction d'un service REST simple qui écoute sur le port 8080 et répond aux requêtes HTTP. Vous trouverez ci-dessous un extrait des principales fonctionnalités qui configurent un routeur et un gestionnaire:

```
func main() {

  router := mux.NewRouter().StrictSlash(true)
  router.HandleFunc("/hello/{name}", hello).Methods("GET")

  // want to start server, BUT
  // not on loopback or internal "10.x.x.x" network
  DoesNotStartWith := "10."
  IP := GetLocalIP(DoesNotStartWith)

  // start listening server
  log.Printf("creating listener on %s:%d",IP,8080)
  log.Fatal(http.ListenAndServe(fmt.Sprintf("%s:8080",IP), router))
}

func hello(w http.ResponseWriter, r *http.Request) {
  log.Println("Responding to /hello request")
  log.Println(r.UserAgent())

  // request variables
  vars := mux.Vars(r)
  log.Println("request:",vars)

  // query string parameters
  rvars := r.URL.Query()
  log.Println("query string",rvars)

  name := vars["name"]
  if name == "" {
    name = "world"
  }

  w.WriteHeader(http.StatusOK)
  fmt.Fprintf(w, "Hello %s\n", name)
}
```

Voici un exemple de création et d'exécution du service:

```bash
mkdir -p $GOPATH/src/echoservice
cd $GOPATH/src/echoservice
wget https://raw.githubusercontent.com/fabianlee/blogcode/master/golang/echoservice/echoservice.go
go get
go build
sudo ufw allow 8080/tcp
./echoservice
```

Ce qui devrait produire une sortie sur le serveur qui ressemble à quelque chose comme:

    2020/04/30 10:35:15 creating listener on 51.91.249.57:8080

Nous pouvons voir que le serveur écoute sur le port 8080. Alors maintenant, en passant à un hôte client, nous exécutons curl contre le service «/hello» (ou utilisons un navigateur), en envoyant un paramètre de «foo».

    sudo apt-get install curl -y
    curl "http://domaine.tld:8080/hello/foo"  # ouvrir port 8080
Bonjour foo

Et côté serveur, la sortie ressemble à:

```
2020/04/30 10:35:15 creating listener on 51.91.249.57:8080
2020/04/30 10:50:31 Responding to /hello request
2020/04/30 10:50:31 curl/7.64.0
2020/04/30 10:50:31 request: map[name:foo]
2020/04/30 10:50:31 query string map[]
```

## EchoService en tant que service systemd

Pour transformer cela en un service pour systemd, nous devons créer un fichier de service d'unité à «**/lib/systemd/system/echoservice.service**» comme ci-dessous:

```ini
[Unit]
Description=Echo service
ConditionPathExists=/home/vpsadmin/work/src/echoservice/echoservice
After=network.target

[Service]
Type=simple
User=echoservice
Group=echoservice
LimitNOFILE=1024

Restart=on-failure
RestartSec=10
startLimitIntervalSec=60

WorkingDirectory=/home/vpsadmin/work/src/echoservice
ExecStart=/home/vpsadmin/work/src/echoservice/echoservice

# make sure log directory exists and owned by syslog
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /var/log/echoservice
ExecStartPre=/bin/chown syslog:adm /var/log/echoservice
ExecStartPre=/bin/chmod 755 /var/log/echoservice
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=echoservice

[Install]
WantedBy=multi-user.target
```

Les chemins absolus dans «ConditionPathExists», «WorkingDirectory» et «ExecStart» doivent tous être modifiés en fonction de votre environnement. Notez que nous avons demandé à systemd d'exécuter le processus en tant qu'utilisateur «échoservice», nous devons donc également créer cet utilisateur.

Vous trouverez ci-dessous des instructions pour créer l'utilisateur et déplacer le fichier de service de l'unité systemd vers l'emplacement correct:

```bash
$ cd /tmp
$ sudo useradd echoservice -s /sbin/nologin -M
$ wget https://raw.githubusercontent.com/fabianlee/blogcode/master/golang/echoservice/systemd/echoservice.service
$ sudo mv echoservice.service /lib/systemd/system/.
$ sudo chmod 755 /lib/systemd/system/echoservice.service
```

Maintenant, vous devriez pouvoir activer le service, le démarrer, puis surveiller les journaux en suivant le journal systemd:

```
$ sudo systemctl enable echoservice.service

$ sudo systemctl start echoservice

$ sudo journalctl -f -u echoservice

May 21 16:56:25 xenial1 systemd[1]: Started Echo service.
May 21 16:56:25 xenial1 echoservice[4450]: 2017/05/21 16:56:25 creating listener on 192.168.2.66:8080
```

Le journal est stocké sous forme de fichier binaire, il ne peut donc pas être suivi directement. Mais si nous configurons syslog, le transfert syslog est activé afin que nous puissions envoyer notre journal à «/var/log/echoservice/echoservice.log».

La section sleepservice ci-dessus a montré comment écouter rsyslog sur le port 514, il nous suffit donc maintenant de créer «/etc/rsyslog.d/30-echoservice.conf» avec le contenu suivant:

```
if $programname == 'echoservice' or $syslogtag == 'echoservice' then /var/log/echoservice/echoservice.log
& stop
```

Redémarrez maintenant le service rsyslog et vous devriez voir l'écouteur syslog sur le port 514, redémarrez l'échoservice, et maintenant vous devriez voir les événements de journal envoyés au fichier toutes les quelques secondes.

```
$ sudo systemctl restart rsyslog
$ netstat -an | grep "LISTEN "
$ sudo systemctl restart echoservice
$ tail -f /var/log/echoservice/echoservice.log

May 21 17:00:53 xenial1 echoservice[4499]: 2017/05/21 17:00:53 creating listener on 192.168.2.66:8080
```

La liste des processus en cours d'exécution montre que le processus s'exécute en tant qu'utilisateur «échoservice».

```
$ ps -ef | grep echoservice

echoser+  4499     1  0 17:00 ?        00:00:00 /home/ubuntu/work/src/echoservice/echoservice
```

## Ports privilégiés

Dans l'exemple ci-dessus, nous avons l'écoservice à l'écoute sur le port 8080. Mais si nous avons utilisé un port inférieur à 1024, des privilèges spéciaux devraient être accordés pour que cela s'exécute en tant que service (ou au premier plan d'ailleurs).

```
May 21 17:03:47 xenial1 echoservice[4560]: 2017/05/21 17:03:47 creating listener on 192.168.2.66:80
May 21 17:03:47 xenial1 echoservice[4560]: 2017/05/21 17:03:47 listen tcp 192.168.2.66:80: bind: permission denied
```

Le moyen de résoudre ce problème n'est pas d'exécuter l'application en tant que root, mais de définir les capacités du binaire. Cela peut être fait avec setcap:

```
$ sudo apt-get install libcap2-bin -y

$ sudo setcap 'cap_net_bind_service=+ep' /your/path/gobinary
```
 

 

RÉFÉRENCES

https://freedesktop.org/wiki/Software/systemd/

https://blog.xyzio.com/2016/06/14/setting-up-a-golang-website-to-autorun-on-ubuntu-using-systemd/

https://serverfault.com/questions/479434/service-file-for-golang-app

https://vincent.bernat.im/en/blog/2017-systemd-golang

https://denbeke.be/blog/servers/running-caddy-server-as-a-service-with-systemd/

https://github.com/coreos/go-systemd

https://fabianlee.org/2017/05/20/golang-running-a-go-binary-as-a-sysv-service-on-ubuntu-14-04/

https://www.loggly.com/ultimate-guide/linux-logging-with-systemd/

https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files

https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units

https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/chap-Managing_Services_with_systemd.html#sect-Managing_Services_with_systemd-Introduction-Features

 

systemctl dae