+++
title = 'Javascript exemples'
date = 2021-06-10 00:00:00 +0100
categories = javascript
+++
## Trier un tableau d'objets en JavaScript

Pour trier un tableau d'objets, vous utilisez la méthode sort() et fournissez une fonction de comparaison qui détermine l'ordre des objets.
Supposons que vous ayez un tableau d'objets employés comme suit :

```js
let employees = [
    {
        firstName: 'John',
        lastName: 'Doe',
        age: 27,
        joinedDate: 'December 15, 2017'
    },

    {
        firstName: 'Ana',
        lastName: 'Rosy',
        age: 25,
        joinedDate: 'January 15, 2019'
    },

    {
        firstName: 'Zion',
        lastName: 'Albert',
        age: 30,
        joinedDate: 'February 15, 2011'
    }
];
```

### par numéros

L'extrait d'instruction suivant trie le tableau des employés par âge dans l'ordre croissant :

```js
employees.sort((a, b) => {
    return a.age - b.age;
});
```

Pour afficher les employés, vous utilisez la méthode forEach() :

```js
employees.forEach((e) => {
    console.log(`${e.firstName} ${e.lastName} ${e.age}`);
});
```

Sortie:

```
Ana Rosy 25
John Doe 27
Zion Albert 30
```

Pour trier les employés par âge en ordre décroissant, il suffit d'inverser l'ordre dans la fonction de comparaison. Par exemple :

```js
employees.sort((a, b) => b.age - a.age);

employees.forEach((e) => {
    console.log(`${e.firstName} ${e.lastName} ${e.age}`);
});
```

Sortie:

```
Zion Albert 30
John Doe 27
Ana Rosy 25
```

### par chaînes de caractères

L'extrait suivant montre comment trier les employés par prénom dans l'ordre décroissant, sans tenir compte de la casse :

```js
employees.sort((a, b) => {
    let fa = a.firstName.toLowerCase(),
        fb = b.firstName.toLowerCase();

    if (fa < fb) {
        return -1;
    }
    if (fa > fb) {
        return 1;
    }
    return 0;
});
```

Dans cet exemple :

1.    Premièrement, convertir les noms des employés en minuscules.
2.    Ensuite, comparer les noms et renvoyer -1, 1 et 0, selon la comparaison des chaînes de caractères.

Le code suivant montre le résultat :

```js
employees.forEach((e) => {
    console.log(`${e.firstName} ${e.lastName}`);
});
```

Sortie:

```
Ana Rosy
John Doe
Zion Albert    
```

### par date

Pour trier les employés par dates jointes, vous devez :

1.    Convertir les dates jointes de chaînes de caractères en objets de type date.
2.    Trier les employés par dates.

Le code suivant illustre l'idée :

```js
employees.sort((a, b) => {
    let da = new Date(a.joinedDate),
        db = new Date(b.joinedDate);
    return da - db;
});
```

Et le code suivant montre la sortie :

```js
employees.forEach((e) => {
    console.log(`${e.firstName} ${e.lastName} ${e.joinedDate}`);
});
```

Sortie:

```
Zion Albert Feb 15, 2011
John Doe December 15, 2017
Ana Rosy January 15, 2019
```

Autre exemple de tri par date

Il existe de nombreuses façons de faire du tri en JavaScript mais dans cet article, nous allons utiliser la fonction par défaut sort() d'un tableau.  
Prenons un exemple pour trier un tableau, nous allons donc utiliser l'exemple de tableau suivant qui contient un objet date.

```js
const students = [
  { name: 'Liam', joinDate: new Date('2019-06-28') },
  { name: 'Noah', joinDate: new Date('2019-06-10') },
  { name: 'William', joinDate: new Date('2019-06-12') },
  { name: 'James', joinDate: new Date('2019-06-08') },
  { name: 'Lucas', joinDate: new Date('2019-06-21') }
];
```

### par date en ordre décroissant

Tout d'abord, nous allons trier le tableau ci-dessus en ordre décroissant par l'attribut joinDate. Utilisez le code suivant pour le tri décroissant.

```js
const sortedStudents = students.sort((a, b) => b.joinDate > a.joinDate ? 1 : -1) ;
sortedStudents.forEach((e) => {
    console.log(`${e.name} ${e.joinDate}`);
});
```

### par date en ordre croissant

Dans le deuxième exemple, nous allons trier le tableau de l'échantillon dans l'ordre croissant par l'attribut joinDate et pour cela utiliser le code suivant.

```js
const sortedStudents = students.sort((a, b) => b.joinDate < a.joinDate ? 1 : -1) ;
sortedStudents.forEach((e) => {
    console.log(`${e.name} ${e.joinDate}`);
});
```

### Utiliser la méthode slice() avec sort()

Lorsque nous utilisons la méthode `sort()`, elle renvoie un tableau trié, mais elle trie également un tableau existant. Ainsi, les variables `students` et `sortedStudents` deviennent identiques.

Pour empêcher le tri d'un tableau existant, nous utiliserons la méthode `slice()`  
Regardez le code ci-dessous pour trier un tableau par date en ordre décroissant.

```js
const sortedStudents = students.slice().sort((a, b) => b.joinDate > a.joinDate ? 1: -1);
sortedStudents.forEach((e) => {
    console.log(`${e.name} ${e.joinDate}`);
});
```

