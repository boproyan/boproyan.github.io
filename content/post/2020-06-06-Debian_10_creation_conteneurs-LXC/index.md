+++
title = 'Debian 10, création conteneurs - LXC'
date = 2020-06-06 00:00:00 +0100
categories = ['virtuel']
+++
![lxc](Linux_Containers_logo.png){:width="100"}

## LXC

**testé sur Debian Buster 10.2**

**Présentation:**  
*(source: Wikipédia) LXC, contraction de l’anglais Linux Containers est un système de virtualisation, utilisant l'isolation comme méthode de cloisonnement au niveau du système d'exploitation. Il est utilisé pour faire fonctionner des environnements Linux isolés les uns des autres dans des conteneurs, partageant le même noyau et une plus ou moins grande partie du système hôte. Le conteneur apporte une virtualisation de l'environnement d'exécution (processeur, mémoire vive, réseau, système de fichier…) et non pas de la machine. Pour cette raison, on parle de « conteneur » et non de « machine virtuelle ». LXC est le système de conteneurisation, sur lequel s'appuie le logiciel Docker.*

**Objectif de cette documentation:**
Installer, configurer avec le réseau libvirt ou bridge et utiliser LXC.

### AVEC LIBVIRT 

(pour obtenir une adresse en 192.168.X.X)

**Installez les paquets requis**

```bash
# apt-get install lxc libvirt-daemon-system
```

**Editez le fichier /etc/lxc/default.conf**

```bash
# vim /etc/lxc/default.conf
```

**Assurez-vous d'avoir ces lignes:**

```bash
lxc.net.0.type = empty
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
lxc.net.0.type = veth
lxc.net.0.link = virbr0
lxc.net.0.flags = up
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
```

**Vérifiez la configuration**

```bash
# lxc-checkconfig
```

**Démarrez le réseau**

```bash
# virsh net-start default
```

**Configurez le démarrage automatique du réseau**

```bash
# virsh net-autostart default
```

**Redémarrez le service LXC**

```bash
# systemctl restart lxc-net
# systemctl status lxc-net
```

**Affichez les interfaces réseaux virtuelles**

```bash
# brctl show
```

### EN BRIDGE ### 

(pour obtenir une adresse en 10.0.3.X)

**Installez les paquets requis**

```bash
# apt-get install lxc
```

**Editez le fichier /etc/default/lxc-net**

```bash
# vim /etc/default/lxc-net
```

**Ajoutez cette ligne:**

```bash
USE_LXC_BRIDGE="true"
```

**Editez le fichier /etc/lxc/default.conf**

```bash
# vim /etc/lxc/default.conf
```

**Ajoutez ces lignes:**

```bash
lxc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
```

**Vérifiez la configuration**

```bash
# lxc-checkconfig
```

**Redémarrez le service LXC**

```bash
# systemctl restart lxc-net
# systemctl status lxc-net
```

### EXPOSER UN PORT 

**Router un port dans un conteneur**

```bash
# iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 10.0.3.11:8080
```
ou Editez le fichier /etc/network/interfaces

```bash
up iptables -t nat -A PREROUTING -i <host-iface> -p tcp --dport <source-port> -j DNAT --to <container-ip>:<container-port>
```

### UTILISER LXC

**Créer un conteneur nommé "c1" avec la distribution Debian**

```bash
# lxc-create -t debian -n c1
```
ou si vous avez une erreur réseau:

```bash
# lxc-create -t download -n debian -- --keyserver hkp://keyserver.ubuntu.com:80
```

**Affichez la liste des conteneurs**

```bash
# lxc-ls -f
```

**Démarrez le conteneur nommé "debian"**

```bash
# lxc-start -n debian
```

**Se connecter au conteneur nommé "c1"**

```bash
# lxc-console -t 0 -n c1
```
ou

```bash
# lxc-attach -n debian --clear-env
```

**Démarrer automatiquement un contenur**

```bash
# lxc config set debian boot.autostart true
```
ou Editez le fichier /var/lib/lxc/$containername/config pour avoir cette ligne:

```bash
lxc.start.auto = 1
```

**Arrêter brutalement un conteneur**

```bash
# lxc-stop -n NOMDUCONTENEUR
```

**Arrêter proprement un conteneur**

```bash
# lxc-halt -n NOMDUCONTENEUR
```

**Détruire un conteneur**

```bash
# lxc-destroy NOMDUCONTENEUR
```

**Affichez les informations d'un conteneur**

```bash
# lxc-info NOMDUCONTENEUR
```

**Copier un conteneur**

```bash
# lxc-copy -n debian1 -N debian2
```

**Relier un point de montage à l'intérieur du conteneur**

```bash
lxc.mount.entry=/CHEMIN /var/lib/lxc/NOMDUCONTENEUR/rootfs/mount_point none bind 0 0
```

**Changer la configuration d'un conteneur**

```bash
# vim /var/lib/lxc/debian/config
```
