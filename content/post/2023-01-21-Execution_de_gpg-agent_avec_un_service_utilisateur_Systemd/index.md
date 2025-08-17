+++
title = 'Exécution de gpg-agent avec un service utilisateur Systemd'
date = 2023-01-21 00:00:00 +0100
categories = ['systemd']
+++
## Gpg Systemd Utilisateur

[Using gpg-agent Effectively](https://eklitzke.org/using-gpg-agent-effectively)  
[Using a GPG key for SSH authentication on macOS and Debian](https://gregrs-uk.github.io/2018-08-06/gpg-key-ssh-mac-debian/)

Il s'agit d'une fonctionnalité de systemd qui vous permet d'exécuter des services sous l'instance systemd gérant votre session de connexion.  
Les avantages de l'utilisation des services utilisateurs de systemd sont:

*    Il existe une seule façon cohérente de gérer les services,  vérifier l'état, activer, désactiver , démarrer et d'arrêter les services.
*    La version des unités (Units) peut être facilement contrôlée . Il est beaucoup plus facile de contrôler des versions comme des unités de temporisation systemd que des crontabs utilisateur.
*    Les données de journaux de ces services sont gérées par journald , ce qui signifie que vous n'avez jamais besoin de deviner où se trouvent les fichiers journaux, et vous pouvez utiliser toutes les fonctionnalités puissantes de `journalctl` pour rechercher ces journaux.
*    Dans le monde non systemd, les processus de démon utilisateur sont parentés au processus d'initialisation du système. Cela provoque des problèmes si vous souhaitez que les démons se terminent lorsque vous vous déconnectez. Il est également difficile de s'assurer que vous n'avez pas d'instances en double des processus démon après la connexion puis la déconnexion. Les services utilisateurs de Systemd résolvent complètement ce problème et vous pouvez contrôler si vous souhaitez que les services «persistent» ou non. 

>**GnuPG** 2.1.16 (sorti en novembre 2016) a ajouté la prise en charge native des services utilisateur systemd.

L'idée de base est que GnuPG comprend un certain nombre de fichiers **.socket** et **.service**  
Les sockets sont gérés par un service utilisateur systemd et un processus **gpg-agent** ou **dirmngr** peut être démarré à la demande lorsqu'un socket est utilisé.  
Le processus agent reste au premier plan et envoie la sortie du journal à **stdout/stderr ** où il est intercepté par **journald**  
**A VERIFIER** &rarr; Vous devez copier ces fichiers dans ~/.config/systemd/user pour les activer et les utiliser. 2

Sur **Fedora**, le paquet gnupg2 placera ces fichiers dans **/usr/share/doc/gnupg2/examples/systemd-user**  

```
$ ls -l /usr/share/doc/gnupg2/examples/systemd-user
total 32
-rw-r--r--. 1 root root  212 Aug 28  2017 dirmngr.service
-rw-r--r--. 1 root root  204 Aug 28  2017 dirmngr.socket
-rw-r--r--. 1 root root  298 Aug 28  2017 gpg-agent-browser.socket
-rw-r--r--. 1 root root  281 Aug 28  2017 gpg-agent-extra.socket
-rw-r--r--. 1 root root  223 Aug 28  2017 gpg-agent.service
-rw-r--r--. 1 root root  234 Aug 28  2017 gpg-agent.socket
-rw-r--r--. 1 root root  308 Aug 28  2017 gpg-agent-ssh.socket
-rw-r--r--. 1 root root 2274 Aug 28  2017 README
```

Sur **Archlinux**, le paquet gnupg placera ces fichiers dans **/usr/share/doc/gnupg/examples/systemd-user**

```
$ ls -l /usr/share/doc/gnupg/examples/systemd-user/
total 32
-rw-r--r-- 1 root root 2274 20 mars  22:01 README
-rw-r--r-- 1 root root  212 20 mars  22:01 dirmngr.service
-rw-r--r-- 1 root root  204 20 mars  22:01 dirmngr.socket
-rw-r--r-- 1 root root  298 20 mars  22:01 gpg-agent-browser.socket
-rw-r--r-- 1 root root  281 20 mars  22:01 gpg-agent-extra.socket
-rw-r--r-- 1 root root  308 20 mars  22:01 gpg-agent-ssh.socket
-rw-r--r-- 1 root root  223 20 mars  22:01 gpg-agent.service
-rw-r--r-- 1 root root  234 20 mars  22:01 gpg-agent.socket
```

Sur **Debian**, il faut trier les fichiers dans **/usr/lib/systemd/user**

```
$ ls -l /usr/lib/systemd/user/{gpg*,dirmngr*}
-rw-r--r-- 1 root root 212 août  28  2017 /usr/lib/systemd/user/dirmngr.service
-rw-r--r-- 1 root root 204 août  28  2017 /usr/lib/systemd/user/dirmngr.socket
-rw-r--r-- 1 root root 298 août  28  2017 /usr/lib/systemd/user/gpg-agent-browser.socket
-rw-r--r-- 1 root root 281 août  28  2017 /usr/lib/systemd/user/gpg-agent-extra.socket
-rw-r--r-- 1 root root 223 août  28  2017 /usr/lib/systemd/user/gpg-agent.service
-rw-r--r-- 1 root root 234 août  28  2017 /usr/lib/systemd/user/gpg-agent.socket
-rw-r--r-- 1 root root 308 août  28  2017 /usr/lib/systemd/user/gpg-agent-ssh.socket
```


Copiez-les dans votre répertoire utilisateur systemd:

```bash
# Créer le répertoire s'il n'existe pas.
mkdir -p ~/.config/systemd/user

# Copier les fichiers.
# cp /usr/share/doc/gnupg2/examples/systemd-user/{*.socket,*.service} ~/.config/systemd/user  # Fedora
# cp /usr/share/doc/gnupg/examples/systemd-user/{*.socket,*.service} ~/.config/systemd/user  # Archlinux
cp /usr/lib/systemd/user/{gpg*,dirmngr*} ~/.config/systemd/user  # Debian 10

# dans mon .bashrc
alias sysu='systemctl --user'

# Rafraîchir systemd
sysu daemon-reload

# Activez et démarrez les prises (sockets)
sysu enable *.socket
sysu enable dirmngr.socket gpg-agent.socket gpg-agent-browser.socket gpg-agent-ssh.socket gpg-agent-extra.socket
sysu start dirmngr.socket gpg-agent.socket gpg-agent-browser.socket gpg-agent-ssh.socket gpg-agent-extra.socket
```

Il y a une dernière étape requise: vous devez dire à **gpg-agent** où demander une entrée **pinentry**.   
Pour le **pinentry** dans X11 ou Wayland, vous pouvez ajouter la ligne suivante à la configuration de votre agent:

```bash
# Set a default display for gpg-agent.
$ echo "display :0" >> ~/.gnupg/gpg-agent.conf
```

Vous pouvez également définir la variable d'environnement **GPG_TTY** si vous n'utilisez pas de session graphique.

### Invoquer gpg-agent

gpg-agent est un démon permettant de gérer des clés secrètes (privées) indépendamment de tout protocole. Il est utilisé comme backend pour gpg et gpgsm ainsi que pour quelques autres utilitaires.

L'agent est automatiquement lancé à la demande par **gpg, gpgsm, gpgconf ou gpg-connect-agent**. Il n'y a donc aucune raison de le lancer manuellement.  

Si vous souhaitez utiliser l'agent Secure Shell inclus, vous pouvez le démarrer à l'aide de l'agent :

    gpg-connect-agent /bye

Si vous souhaitez mettre fin manuellement à l'agent en cours, vous pouvez le faire en toute sécurité avec :

    gpgconf --kill gpg-agent

Vous devez toujours ajouter les lignes suivantes à votre fichier **.bashrc** ou tout autre fichier d'initialisation utilisé pour toutes les invocations du shell (pour la variable d'environnement **GPG_TTY**) :

```bash
GPG_TTY=$(tty)
export GPG_TTY
```

### Utilisation de l'agent SSH

Pour que l'agent SSH fonctionne, vous devez pointer **SSH_AUTH_SOCK** sur le nouveau socket SSH. Le moyen le plus simple consiste à ajouter ceci à votre profil shell:

```bash
# Add this to ~/.profile or ~/.bash_profile
export SSH_AUTH_SOCK="/run/user/$(id -u)/gnupg/S.gpg-agent.ssh"
```

Il y a un dernier problème:l'agent n'agira qu'en déléguant les clés SSH avec les keygrips énumérés dans **~/.gnupg/sshcontrol**   
Vous pouvez également utiliser ce fichier pour définir des stratégies de mise en cache par clé, ce qui peut être utile si vous avez plusieurs clés SSH et souhaitez mettre en cache certaines des clés plus ou moins longtemps que les autres.  
La première fois que vous utilisez une clé SSH avec l'agent, vous devez exécuter manuellement `ssh-add` dans un terminal.  
Cela devrait mettre à jour le fichier **sshcontrol** et ensuite vous devriez voir quelque chose comme ceci:

    cat ~/.gnupg/sshcontrol

```bash
# List of allowed ssh keys.  Only keys present in this file are used
# in the SSH protocol.  The ssh-add tool may add new entries to this
# file to enable them; you may also add them manually.  Comment
# lines, like this one, as well as empty lines are ignored.  Lines do
# have a certain length limit but this is not serious limitation as
# the format of the entries is fixed and checked by gpg-agent. A
# non-comment line starts with optional white spaces, followed by the
# keygrip of the key given as 40 hex digits, optionally followed by a
# caching TTL in seconds, and another optional field for arbitrary
# flags.   Prepend the keygrip with an '!' mark to disable it.

# Ed25519 key added on: 2018-03-21 03:03:49
# Fingerprints:  MD5:39:fe:2b:cf:54:3a:c2:db:e4:b1:d2:70:a2:bc:82:fe
#                SHA256:LKeBCI5/EDJDMKzH4qBBhzJFJaBObn+APfoOzpLRpCI
A8BB036E2A737F56BB0D0B4F9700BC3DC4A9137F 0

```

>Ceci n'est requis que la toute première fois que vous utilisez une clé.  
Une fois la clé présente dans ce fichier, vous pouvez simplement taper `ssh` et le processus de l'agent GnuPG sera automatiquement utilisé. 

---

## Utilisation de gpg-agent

*Vous souhaitez simplifier le cryptage des données (par exemple, les messages électroniques) via GnuPG en utilisant gpg-agent*

### Procédure

Le tutoriel suivant s'adresse aux utilisateurs déjà familiarisés avec GnuPG et est basé sur un document disponible sur la page d'accueil de KMail sous <http://kmail.kde.org/kmail-pgpmime-howto.html>  

Si vous avez effectué une installation standard, tous les packages nécessaires sont déjà installés. Si ce n'est pas le cas (ou si vous avez mis à jour), assurez-vous que les packages suivants sont installés

```
   gpg
   gpgme
   newpg
   cryptplug
   libgcrypt
   libksba
   pinentry
```

Vérifiez que la ligne suivante est incluse dans le fichier de configuration de GnuPG (**~/.gnupg/gpg.conf**):

    use-agent

Si la ligne n'est pas incluse, insérez-la avec un éditeur de votre choix.  

Où est pinentry

     which pinentry

/usr/bin/pinentry

Créez ensuite un fichier **~/.gnupg/gpg-agent.conf** avec le contenu suivant:

```
pinentry-program /usr/bin/pinentry-qt
no-grab
default-cache-ttl 1800
```

Le programme **pinentry** est spécifié et une valeur de délai d'attente est définie.

Lancez ensuite `gpg-agent` en insérant l'entrée eval **"$(gpg-agent --daemon)"** dans le fichier **~/.xinitrc** de votre répertoire HOME:

```
 #
 # Add your own lines here...
 #

 eval "$(gpg-agent --daemon)"
```

Cet appel définit la variable d'environnement "GPG_AGENT_INFO" au démarrage de l'interface graphique.  
Vous devez vous reconnecter pour démarrer `gpg-agent`

Effectuez un court test pour vérifier si GnuPG travaille en collaboration avec l'agent, entrez la commande shell suivante:

  echo "test" | gpg -ase -r 0xMYKEYID | gpg

Remplacez **0xMYKEYID** par votre ID de clé GnuPG.  
Lors de l'exécution de cette commande, l'agent doit ouvrir deux fois une boîte de dialogue graphique de mot de passe: 

* d'abord pour la signature ou le chiffrement ( gpg -ase ) (gpg -ase) 
* puis pour le déchiffrement ou le contrôle de signature ( | gpg ). 

Désormais, chaque fois que GnuPG est utilisé (soit à partir de la ligne de commande, soit intégré à un programme graphique tel que KMail), le mot de passe de l' agent gpg sera transmis automatiquement (jusqu'à l'expiration du délai d'expiration ou la fermeture de l'interface graphique) .

>Remarque: verrouillez toujours votre bureau lorsque vous quittez votre lieu de travail. Quiconque peut accéder à votre session X aura un accès illimité à votre clé secrète.

https://superuser.com/questions/764347/using-systemd-to-start-gpg-agent

La commande gpg-agent lance un démon, mais les programmes qui l'utilisent s'attendent à ce que certaines variables d'environnement (GPG_AGENT_INFO et GPG_TTY) soient définies pour savoir comment communiquer avec l'agent. Vous devez d'une manière ou d'une autre les propager du script de service à vos shells. La page MAN de gpg-agent contient un extrait qui démarre le démon et écrit un fragment de code shell dans un fichier situé dans la maison de l'utilisateur

	gpg-agent --daemon --write-env-file "${HOME}/.gpg-agent-info"

Vous pouvez mettre cette ligne en script shell et l'appeler depuis votre fichier de service

```
[Service]
Type=forking
ExecStart=script-file.sh
<...>
````

Le fichier .gpg-agent-info doit provenir de chaque shell. La page MAN recommande

```
if [ -f "${HOME}/.gpg-agent-info" ]; then
  . "${HOME}/.gpg-agent-info"
  export GPG_AGENT_INFO
fi

GPG_TTY=$(tty)
export GPG_TTY
```

dans votre fichier .profile pour ce faire.  
Vous trouverez des informations sur la manière d'écrire les fichiers de service systemd dans la page MAN systemd.service.
