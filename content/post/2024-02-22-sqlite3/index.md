+++
title = 'Base de données Sqlite3'
date = 2024-02-22 00:00:00 +0100
categories = ['divers']
+++
*Une base SQLite3 a la particularité d'être contenue dans un fichier qui porte le même nom. Le moteur de base de données SQLite3 est une bibliothèque, libsqlite3, qu'il est possible d'utiliser avec l'interface utilisateur en ligne de commande **sqlite3**   
SQLite3 présente l'avantage de n'avoir rien à configurer, rien à maintenir ou à administrer. C'est aussi son objectif. En contrepartie, certaines fonctionnalités sont absentes, comme la gestion des utilisateurs ou la possibilité de se connecter à distance à la base (en TCP/IP par exemple).*

![](sqlite-logo.png)

## Installation

Pour installer SQLite3

```
sudo apt-get install sqlite3
```

## Utilisation en ligne de commande

### Lancer le terminal SQLite

Dans un terminal, lancer la commande suivante avant de taper les commandes propres à SQLite:

```
sqlite3
```


Le curseur indique que vous êtes maintenant dans le "terminal" SQlite comme ici:

```
SQLite version 3.6.22
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite>
```

*Toute commande SQLite sera précédée de son curseur afin de bien distinguer les commandes SQLite des autres commandes*

### Quitter le terminal SQLite

```
sqlite>.quit
```

### Lister les commandes SQLite

```
sqlite>.help
```

Affichera:

```
.backup ?DB? FILE		Backup DB (default "main") to FILE
.bail ON|OFF			Stop after hitting an error.  Default OFF
.databases			List names and files of attached databases
.dump ?TABLE? ...		Dump the database in an SQL text format
				     If TABLE specified, only dump tables matching
				     LIKE pattern TABLE.
.echo ON|OFF			Turn command echo on or off
.exit				Exit this program
.explain ?ON|OFF?		Turn output mode suitable for EXPLAIN on or off.
				     With no args, it turns EXPLAIN on.
.genfkey ?OPTIONS?		Options are:
				     --no-drop: Do not drop old fkey triggers.
				     --ignore-errors: Ignore tables with fkey errors
				     --exec: Execute generated SQL immediately
				See file tool/genfkey.README in the source 
				distribution for further information.
.header(s) ON|OFF		Turn display of headers on or off
.help				Show this message
.import FILE TABLE		Import data from FILE into TABLE
.indices ?TABLE?		Show names of all indices
				     If TABLE specified, only show indices for tables
				     matching LIKE pattern TABLE.
.load FILE ?ENTRY?		Load an extension library
.mode MODE ?TABLE?		Set output mode where MODE is one of:
				     csv      Comma-separated values
				     column   Left-aligned columns.  (See .width)
				     html     HTML <table> code
				     insert   SQL insert statements for TABLE
				     line     One value per line
				     list     Values delimited by .separator string
				     tabs     Tab-separated values
				     tcl      TCL list elements
.nullvalue STRING		Print STRING in place of NULL values
.output FILENAME		Send output to FILENAME
.output stdout			Send output to the screen
.prompt MAIN CONTINUE		Replace the standard prompts
.quit				Exit this program
.read FILENAME			Execute SQL in FILENAME
.restore ?DB? FILE		Restore content of DB (default "main") from FILE
.schema ?TABLE?			Show the CREATE statements
				     If TABLE specified, only show tables matching
				     LIKE pattern TABLE.
.separator STRING		Change separator used by output mode and .import
.show				Show the current values for various settings
.tables ?TABLE?			List names of tables
				     If TABLE specified, only list tables matching
				     LIKE pattern TABLE.
.timeout MS			Try opening locked tables for MS milliseconds
.width NUM1 NUM2 ...		Set column widths for "column" mode
.timer ON|OFF			Turn the CPU timer measurement on or off

```


### Modifier le format de sortie

Vous pouvez par exemple:
  * Afficher le nom des colonnes / Changer l'aspect des colonnes  
  * Modifier le séparateur
  * Modifier la largeur des colonnes
  * Modifier la sortie en code html
  

#### Afficher le nom des colonnes 

Changer l'aspect des colonnes

Pour présenter les résultat d'une requête sous forme de tableau dans le terminal on utilisera:

```
sqlite> .headers on
sqlite> .mode column
```
 
Et à l'issue d'une requête les résultats s'afficheront de cette manière:

```
nom        age    	membre
----------  ----------  ----------
dan         23         oui
bob         45         non
```

Pour revenir au mode précédent:

```
sqlite> .header off		# Mode par défaut
sqlite> .mode list		# Mode par défaut
```

Affichera:

```
dan|23|oui
bob|45|non
```


#### Modifier le séparateur

Il est possible de modifier le séparateur dans le mode list:

```
sqlite> .separator ", "
```

Affichera:

```
dan, 23, oui
bob, 45, non
```


#### Modifier la largeur des colonnes

Par défaut, chaque colonne fait 10 caractères de largeur. Il est possible d'ajuster la taille ainsi:

```
sqlite> .width 2 15 10 20 3
```

Affichera:

```
id  titre            auteur      resume           num  date_creation    éditeur  
--  ---------------  ----------  ---------------  ---  --------------  ----------
1   tintin au congo  hergé      Tintin est au c  5.0                   casterman 
2   le nid des mars  franquin    Un reportage in  6.0                        
3   la déesse       moebius     une aventure ex  7.0  2011-02-03  
```

#### Modifier la sortie en code html

Il est possible de sortir directement les résultats en html:

```
sqlite> .mode html
```

Affichera:

```
<TR><TH>id</TH>
<TH>titre</TH>
<TH>auteur</TH>
<TH>resume</TH>
<TH>num</TH>
<TH>timenter</TH>
<TH>éditeur</TH>
</TR>
<TR><TD>1</TD>
<TD>tintin au congo</TD>
<TD>hergé</TD>
<TD>Tintin est au congo.</TD>
<TD>5.0</TD>
<TD></TD>
<TD>casterman</TD>
</TR>
<TR><TD>2</TD>
<TD>le nid des marsupilamis</TD>
<TD>franquin</TD>
<TD>Un reportage incroyable</TD>
<TD>6.0</TD>
<TD></TD>
<TD></TD>
</TR>
<TR><TD>3</TD>
<TD>la déesse</TD>
<TD>moebius</TD>
<TD>une aventure extraordinaire</TD>
<TD>7.0</TD>
<TD>2011-02-03</TD>
<TD></TD>
</TR>
```


### Rappel des paramètres

```
sqlite> .show
```

Affichera:

```
     echo: off
  explain: off
  headers: off
     mode: list
nullvalue: ""
   output: stdout
separator: "|"
    width: 
```


### Manipuler une base

* Créer une base - ouvrir une base
* Créer une table
* Lister les tables
* Insérer des valeurs dans la table
* Faire une simple requête pour visualiser le contenu de la table
* Effacer une valeur dans la table
* Ajouter une colonne à la table
* Mettre à jour une valeur à la table
* Modifier le nom d'une table
* Écrire la sortie des résultats dans un fichier
* Dumper une table depuis SQLite en format SQL pour sauvegarder la structure et les données sur un disque
* Dumper une base en format SQL pour sauvegarder sa structure, ses tables et ses données

#### Créer une base - ouvrir une base

```
sqlite3 livres.db
```

Si la base **livres.db** n'existe pas déjà, elle sera créée. Cette commande, non seulement créé ou ouvre la base, mais place le curseur immédiatement dans l'environnement terminal de SQLite. Toute commande tapée par la suite concernera cette base. Un fichier **livres.db** est créé.

#### Détruire une base 

Il suffit de détruire le fichier correspondant. Par exemple 

```
rm livres.db
```

#### Créer une table

Il faut avoir ouvert ou créé une base avant de taper les commandes suivantes:

```
sqlite> CREATE TABLE bandedessinée (id integer primary key, titre VARCHAR(30), auteur VARCHAR(30), resume TEXT, num double, date_creation date);
```


`Si …> apparait après avoir tapé la commande, c'est qu'il manque tout simplement le ; à la fin de la requête. Ajoutez-le juste après le …>; et validez.`  
Les types de données SQLite3 sont tels qu'expliqué [ici](https:www.sqlite.org/datatype3.html) 

```
NULL => null
INT => 0 1 2 3
VARCHAR(64) => CHAINE
TEXT => TEXTE
BOOLEAN => TRUE/FALSE
DATETIME => YYYY-MM-JJ HH:MM:SS
DATE => YYYY-MM-JJ
FLOAT => -0,123, 1.2345
```

Ce qui donne par exemple :

```
sqlite> CREATE TABLE bandedessinée (id integer primary key, titre TEXT, auteur TEXT, resume TEXT, num REAL, date_creation INTEGER);
```

#### Lister les tables

```
sqlite> .tables
```

#### Insérer des valeurs dans la table

Un exemple d'insertion de valeurs:

```
sqlite> INSERT INTO "bandedessinée" VALUES(1, 'tintin au congo', 'hergé', 'Tintin est au congo.', 5.0, NULL);
sqlite> INSERT INTO "bandedessinée" VALUES(2, 'le nid des marsupilamis', 'franquin', 'Un reportage incroyable', 6.0, date('now'));
sqlite> INSERT INTO "bandedessinée" VALUES(3, 'la déesse', 'moebius', 'une aventure géniale', 7.0, strftime("%Y-%m-%d %H:%M:%S",'now','localtime'));
```

#### Simple requête pour visualiser le contenu de la table

```
sqlite> select * from bandedessinée;
```

Affichera:

```
1|tintin au congo|hergé|Tintin est au congo.|5.0|
2|le nid des marsupilamis|franquin|Un reportage incroyable|6.0|2011-02-03
3|la déesse|moebius|une aventure extraordinaire|7.0|2011-02-03 18:36:25
```

#### Requête de visualisation d'une table formatée

en sortie COMME une insertion de valeur

```
sqlite> .mode insert bandedessinée
sqlite> select * from bandedessinée;
```

Affichera:

```
INSERT INTO 'bandedessinée' VALUES(1,'tintin au congo','hergé','Tintin est au congo.',5.0,NULL);
INSERT INTO 'bandedessinée' VALUES(2,'le nid des marsupilamis','franquin','Un reportage incroyable',6.0,'2011-02-03');
INSERT INTO 'bandedessinée' VALUES(3,'la déesse','moebius','une aventure extraordinaire',7.0,'2011-02-03');
```


#### Quelques exemples de requêtes

Limiter une requête par nombre d'éléments: 

```
sqlite> sqlite> select * from bandedessinée limit 2;
```

résultat

```
1|tintin au congo|hergé|Tintin est au congo.|5.0||casterman
2|le nid des marsupilamis|franquin|Un reportage incroyable|6.0||
```


Sélectionner les titres de la table bandedessinée enregistrés depuis février: 

```
sqlite> select titre from bandedessinée where strftime('%m', date_creation)='02';
```

résultat

```
le nid des marsupilamis
la déesse
```

#### Effacer une valeur dans la table

```
sqlite> DELETE FROM "bandedessinée" WHERE id = 3;
```

#### Ajouter une colonne à la table

```
sqlite> ALTER TABLE "bandedessinée" add column "éditeur";
```

#### Mettre à jour une valeur à la table

```
sqlite> UPDATE "bandedessinée" SET éditeur ='casterman' WHERE id = 1;
```

#### Modifier le nom d'une table

```
alter table 'bandedessinée' rename to 'bd';
```

#### Écrire la sortie des résultats dans un fichier

```
sqlite> .output bd.txt
sqlite> select * from bd;
sqlite> .quit
```

Visualiser dans le terminal le fichier créé:

```
cat bd.txt
```

#### Dumper une table depuis SQLite en format SQL 

pour sauvegarder la structure et les données sur un disque

résultat de la commande .dump

```
sqlite> .dump bd

PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE "bd" (id integer primary key, titre VARCHAR(30), auteur VARCHAR(30), resume TEXT, num double, date_creation date);
INSERT INTO "bd" VALUES(1,'tintin au congo','hergé','Tintin est au congo.',5.0,NULL);
INSERT INTO "bd" VALUES(2,'le nid des marsupilamis','franquin','Un reportage incroyable',6.0,'2011-02-04');
INSERT INTO "bd" VALUES(3,'la déesse','moebius','une aventure géniale',7.0,'2011-02-04 17:06:23');
COMMIT;
```


#### Rediriger la sortie vers un fichier 

puis dumper la table depuis SQLite

```
sqlite> .output bd.sql
sqlite> .dump bd
```

Le résultat n'est plus affiché dans le terminal, mais redirigé vers le fichier **bd.sql**.  
Pour le vérifier il suffit d'afficher le contenu du fichier:

```
sqlite> .quit
cat bd.sql
```


#### Lire directement un fichier dumpé depuis sqlite

Tout d'abord effacez la table de la base:

```
sqlite> drop table bd;
```

Puis lisez le fichier sauvegardé:

```
sqlite> .read bd.sql
sqlite> select * from bd;
```

Affichera:

```
1|tintin au congo|hergé|Tintin est au congo.|5.0|
2|le nid des marsupilamis|franquin|Un reportage incroyable|6.0|2011-02-04
3|la déesse|moebius|une aventure géniale|7.0|2011-02-04 17:06:23
```

#### Dumper une base en format SQL pour sauvegarder sa structure, ses tables et ses données

Dumper la base

```
sqlite3 livres.db .dump > livres.sql
```


Récupérer un fichier dumpé pour recréer la base

Effacez le fichier original avant de procéder à la récupération de la base:

```
rm -r livres.db				# Effacer la base originale
sqlite3 livres.db < livres.sql		# Récuperer la base depuis le fichier de sql
sqlite3 livres.db			# Se connecter à la base

sqlite> select * from bd;		# Faire un requête pour vérification
```

Affichera:

```
1|tintin au congo|hergé|Tintin est au congo.|5.0|
2|le nid des marsupilamis|franquin|Un reportage incroyable|6.0|2011-02-04
3|la déesse|moebius|une aventure géniale|7.0|2011-02-04 17:06:23
```


## Utilisation avec un client graphique

Liste de clients graphiques libres disponibles sous Linux :

* [[https:sqlitebrowser.org/|Sqlite Browser
* [[https:mbg-sqlclient.developpez.com/|Ohraimeur.
* [[https:extendsclass.com/sqlite-browser.html|ExtendsClass (interface web).
* [[https:www.phpliteadmin.org/|phpLiteAdmin (interface web).

## Utiliser Sqlite avec Python

Exemple d'utilisation de SQLite avec un script python.  
Créez un fichier **test-python.py**. 

```
gedit test-python.py
```


Et ajoutez le code ci-dessous:

```python
#!/usr/bin/env python
#-*- coding: utf-8 -*-
         
import sqlite3
        
class Mabase():
        
	def __init__(self):
	
		self.conn = sqlite3.connect('mabase.db')
                 
	def creer(self):
		"""	Créer une simple base.
			Renvoi True si reussie, False si déjà créée. """
                  
		# Obtention d'un curseur
		c = self.conn.cursor()
                  
		# Créer une table
		try:
			c.execute('create table comptes (id INTEGER PRIMARY KEY,signature VARCHAR(50), compteur INTEGER)')
			# Inserer deux lignes de données
			c.execute('insert into comptes values (null,"gffgdfgd","0")')
			c.execute('insert into comptes values (null,"Martin","1")')
                                    
			# Sauvegarder les modifications
			self.conn.commit()
                                    
			# Fermer le curseur
			c.close()
			print "Création de la base réussie."
			return True
                                    
		except:
			# Fermer le curseur
			c.close()
			return False
                                    
	def lire(self):
		"""	"""
		c = self.conn.cursor()
		c.execute("SELECT * FROM comptes")
		for row in c:
			print row
		c.close()
                          
# Création de l'instance de classe
mabase = Mabase()
if not mabase.creer():	# Si la méthode creer() renvoi False, lire la base
	mabase.lire()
```

Modifiez les permissions du script **test-python.py** afin de le rendre exécutable, puis lancez-le dans le terminal en tapant:

```
./test-python.py
```

