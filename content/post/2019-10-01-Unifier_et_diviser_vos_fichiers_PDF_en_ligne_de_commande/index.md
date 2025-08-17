+++
title = 'Unifier et diviser vos fichiers PDF en ligne de commande'
date = 2019-10-01 00:00:00 +0100
categories = pdf
+++
Unifier et diviser vos fichiers PDF en ligne de commande sous GNU/Linux

## pdf-unit-separate

Il existe un bon nombre d’outils graphiques permettant de concaténer ou de diviser un fichier PDF sous GNU/Linux, comme par exemple « pdfmod » ou « pdfchain ». Néanmoins, il peut être utile de savoir qu’il existe des commandes généralement intégrées nativement dans la plupart des distributions GNU/Linux pour réaliser ce genre d’opérations. Je vous expliquerai donc très brièvement comment utiliser les commandes « pdfunite » et « pdfseparate »  (testé sur Manjaro et Ubuntu), que vous pourrez éventuellement utiliser avant de vous tourner vers d’autres outils.

### Unifier des fichiers PDF à l’aide de « pdfunite » 

Dans l’exemple ci-dessous les fichiers pdf1, pdf2 et pdf3 vont être fusionnés dans cet ordre, au sein d’un fichier de sortie unique, nommé out.pdf.

	pdfunite pdf1.pdf pdf2.pdf pdf3.pdf out.pdf

### Séparer des fichiers PDF à l’aide de « pdfseparate » 

Légèrement plus compliqué mais toujours trivial d’utilisation, le fichier PDF fourni ici en entrée (in.pdf) va être divisé en x fichiers PDF,  ou : x représente le nombre de pages du fichier PDF et ou %d est remplacé par le numéro de page.

	pdfseparate in.pdf out%d.pdf

On peut aussi éventuellement spécifier une page de début et de fin :

	pdfseparate -f 3 -l 5 in.pdf out%d.pdf

Dans ce dernier cas seules les pages 3 a 5 seront extraites.

### Utiliser « qpdf » dans certains cas:  

Le bémol de ces deux commandes est qu’elles ne fonctionnent pas des lors que le fichier PDF est encrypté. Vous pouvez alors éventuellement décrypter un fichier avec la commande « qpdf », elle aussi disponible out of the box sous Manjaro et Ubuntu.

Exemple :

	qpdf in.pdf --decrypt --password=mot_de_passe_du_pdf out.pdf


