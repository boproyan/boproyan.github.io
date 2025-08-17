+++
title = 'nmap'
date = 2020-11-15 00:00:00 +0100
categories = divers
+++
## nmap

Trouver l'adresse avec **nmap** ,exemple

    sudo nmap -T4 -sP 192.168.0.0/24


## What is Dracnmap ?

[![Version](https://img.shields.io/badge/Dracnmap-2.2.0-brightgreen.svg?maxAge=259200)]()
[![Version](https://img.shields.io/badge/Codename-Redline-red.svg?maxAge=259200)]()
[![Stage](https://img.shields.io/badge/Release-Stable-brightgreen.svg)]()
[![Build](https://img.shields.io/badge/Supported_OS-Linux-orange.svg)]()

Le logiciel nmap comprend plusieurs options qui peuvent paraître complexes et même difficiles à retenir notamment pour les débutants.
Dracnmap vous épargne la nécéssité de retenir ces commandes et réalise vos scans de réseaux en quelques secondes.
Ce programme utilise nmap pour scanner les réseaux et collecter des informations sur des cibles.

Pour télécharger Dracnmap, clonez son repo GitHub:

```
git clone https://github.com/Screetsec/Dracnmap.git
cd Dracnmap
chmod +x dracnmap-v2.2.sh
sudo ./dracnmap-v2.2.sh # or sudo su ./dracnmap-v2.2.sh
```

