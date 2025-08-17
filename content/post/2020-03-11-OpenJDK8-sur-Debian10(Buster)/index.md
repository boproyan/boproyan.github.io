+++
title = 'OpenJDK 8 sur Debian 10 (Buster)'
date = 2020-03-11 00:00:00 +0100
categories = ['debian']
+++
## OpenJDK 8 sur Debian 10 (Buster)

Le kit de développement Java (JDK) est un environnement de développement qui comprend l'environnement d'exécution Java (JRE), un interpréteur / chargeur (java), un compilateur (javac), un archiveur (jar), un générateur de documentation (Javadoc) et d'autres outils nécessaires au développement Java.

### Conditions préalables

Debian 10 (Buster) avec accès root

	sudo -s
	apt install -y wget gnupg software-properties-common

### Ajout du dépôt AdoptOpenJDK 

pour installer OpenJDK 8 sur Debian Buster

	wget -qO - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public | sudo apt-key add -
	add-apt-repository --yes https://adoptopenjdk.jfrog.io/adoptopenjdk/deb/

### Installer openjdk 8

	apt update 
	apt install adoptopenjdk-8-hotspot -y
	exit	# sortie root

### Vérification version  

	java -version

```	
openjdk version "1.8.0_242"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_242-b08)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.242-b08, mixed mode)
```

### Variable environnement

définir la variable d'environnement Java

	sudo update-alternatives --config java

```
Il n'existe qu'une « alternative » dans le groupe de liens java (qui fournit /usr/bin/java) : /usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/bin/java
Rien à configurer.
```

Définir la variable d'environnement manuellement

	export JAVA_HOME=/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/bin/java

Ajout définition au fichier **.bashrc**


et collez ce qui suit et enregistrez et quittez.

	echo "export JAVA_HOME=/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/bin/java" >> ~/.bashrc

Prise en compte immédiate

	source ~/.bashrc

Vérifier le Java Home

	echo $JAVA_HOME

```
/usr/lib/jvm/adoptopenjdk-8-hotspot-amd64/bin/java
```

