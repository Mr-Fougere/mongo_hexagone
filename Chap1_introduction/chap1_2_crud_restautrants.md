# CRUD create/read/update/delete avec MongoDB avec la base de données ny

## Création d'une base de données

Pour créer une nouvelle base de données taper la ligne de commande suivante :

```bash
mongo
> use school
```

Si vous avez installé Robo 3T vous pouvez également créer la base de données à l'aide de cet outils.

## Insertion de données

Créez la collection authors dans la base de données school

```js
db.createCollection("authors");
```

Insérer une donnée ou plusieurs données en même temps à l'aide de la méthode **insert** :

```js
// Un seul document
db.authors.insert({
  name: "Alan",
  grade: "master 5",
  notes: [11, 20, 18, 19],
  status: "A++",
});

// Plusieurs dans un tableau
db.authors.insert([
  {
    name: "Alan",
    grade: "master 5",
    notes: [11, 20, 18, 19],
    status: "A++",
  },
  {
    name: "Alice",
    grade: "master 4",
    notes: [11, 17, 19, 13],
    status: "A+",
  },
]);
```

Remarques : si on ne précise pas de propriété \_id dans le document, il sera automatiquement créé. Celui-ci est de type ObjectId (voir ci-après pour sa définition précise).

Les méthodes **insertMany** et **insertOne** permettent respectivement d'insérer plusieurs ou un document unique.

Création d'un document avec un ObjectId :

```js
db.authors.insert({
  _id: ObjectId("5063114bd386d8fadbd6b004"),
  name: "Naoudi",
  grade: "master 5",
});
```

Vous pouvez créer votre propre \_id avec une valeur de type scalaire (non mutable).

- Précisions sur l'objet ObjectId

Il est codé sur 12 bytes :

- 4 bytes représentant le timestamp courant (nombre de secondes depuis epoch, naissance d'UNIX).
- 3 bytes pour idenfitication de la machine.
- 2 bytes pour représenter l’identifiant du processus.
- 3 bytes qui représentent un compteur qui démarre à un numéro aléatoire.

Tapez les lignes de code suivantes :

```js
const _id = ObjectId();
print(_id);
// ObjectId("5eef0c14591a8edc333898dd")
print(_id.getTimestamp());
// Sun Jun 21 2020 09:28:20 GMT+0200 (CEST)
```

La méthode insertMany :

```js
try {
  db.authors.insertMany([
    {
      name: "Alan",
      grade: "master 5",
      notes: [11, 20, 18, 19],
      status: "A++",
    },
    {
      name: "Alice",
      grade: "master 3",
      notes: [20, 18, 11, 13],
      status: "A+",
    },
  ]);
} catch (e) {
  print(e);
}
```

Méthode insertOne :

```js
db.authors.insertOne({ name: "Bernard", grade: "professor" });
```

## Méthode find lecture des données

### Installez les données restaurants

Récupérez les données dans un dossier **DataExamples** :

https://raw.githubusercontent.com/mongodb/docs-assets/primer-dataset/primer-dataset.json

Dans le dossier **DataExamples** et dans un terminal ouvert dans ce dossier tapez la ligne de commande ci-dessous :

--db pour donner un nom à votre base de données.
--collection indique le nom de votre collection
--file indique le nom du fichier à intégrer dans la base de données
--drop supprimera au préalable les collections existantes.

```bash
# Import de données csv dans une base de données que l'on va créer train
mongoimport --db ny --collection restaurants --file primer-dataset.json --drop
```

Vérifiez que vos données sont bien importées :

```bash
show dbs

use ny

show collections
restaurants

db.restaurants.count()
```

Pour faire une sauvegarde d'une collection au format BJSON tapez la ligne de commande suivante, la sauvegarde se fera dans un dossier dump, dans le dossier où votre terminal a été ouvert.

```bash
mongodump --collection restaurants --db ny
```

L'instruction suivante correspond à un SELECT \* FROM restaurants en SQL :

```js
db.restaurants.find({});
```

En SQL on peut faire des sélections précises à l'aide d'une restriction partie WHERE :

```sql
SELECT *
FROM restaurants
WHERE cuisine = "Delicatessen";
```

En MongoDB cela donnerait :

```js
db.restaurants.find({ cuisine: "Delicatessen" });
```

Plus généralement la structure de la méthode find ressemble à :

```js
db.collection.findOne(restriction, projection);
```

Par exemple on sélectionne les restaurants qui font de la cuisine Delicatessen en affichant que les champs : cuisine et address :

```js
db.restaurants
  .find({ cuisine: "Delicatessen" }, { _id: 0, cuisine: 1, address: 1 })
  .pretty();
```

### Opérateur IN

Vous pouvez également utiliser les query operators comme dans l'exemple suivant, ici on cherche à sélectionner les types de cuisines Delicatessen ou American dans la collection restaurants

```js
db.restaurants.find({ cuisine: { $in: ["Delicatessen", "American"] } });
```

Cet opérateur est similaire à l'opérateur IN de SQL.

### Opérateurs AND et OR

- On peut également utiliser un opérateur logique ET comme suit :

```js
db.restaurants.find({ borough: "Brooklyn", cuisine: "Hamburgers" });

// De manière équivalente
db.restaurants.find({
  $and: [{ borough: "Brookyn" }, { cuisine: "Hamburgers" }],
});
```

- Syntaxe de l'opérateur or :

```js
// { $or: [ { <expression1> }, { <expression2> }, ... , { <expressionN> } ] }

// Exemple sur la table authors
db.authors.find({ $or: [{ name: "Alan" }, { name: "Alice" }] });
```

Voici un exemple de condition logique en utilisant OR et AND. Remarquez le deuxième argument de la méthode find, il permet de faire une projection, c'est-à-dire de sélectionner uniquement certaine(s) propriété(s) du document :

```js
db.restaurants.find(
  {
    borough: "Brooklyn",
    $or: [{ name: /^B/ }, { name: /^W/ }],
  },
  { name: 1, borough: 1 }
);
```

Cela correspondrait (...) en SQL à la requête suivante :

```sql
SELECT
`name`,
borough
FROM restaurants
WHERE borough = "Brooklyn"
AND ( `name` LIKE '/^B/' OR `name` LIKE '/^W/')
```

## 1. Exercice compter le nombre de restaurants

Sans utiliser la méthode count dans un premier temps comptez le nombre de restaurants dans le quartier de Brooklyn.

Pour itérer sur une requête vous utiliserez l'une des deux syntaxes suivantes :

```js
// 1
db.collection.find().forEach((doc) => print(tojson(doc)));

// 2
const myCursor = db.users.find(restriction);
while (myCursor.hasNext()) {
  print(tojson(myCursor.next()));
}
```

Puis comparez le résultat avec la méthode count :

```js
db.collection.findOne(query, restriction).count();
```

### Correction

```js
// première version
const cursor = db.restaurants.find({ borough: "Brooklyn" }, { _id: 1 });

let count = 0;
while (cursor.hasNext()) {
  cursor.next();
  count = count + 1;
}
print("next---- une dernière fois");
// print(cursor.next()); // plus de doc
print(count);

// deuxième version
let count = 0;
function counter(doc) {
  count += 1;
  print(doc._id);
}
// on peut passer en JS une fonction en paramètre ici forEach de Mongo
// passera le paramètre doc à la fonction
db.restaurants.find({ borough: "Brooklyn" }, { _id: 1 }).forEach(counter);

print(count);
```

### 2. Exercices sur la notion de filtrage

Exemple de filtres classiques :

```js
// plus grand que
$gt, $gte;

// Plus petit que
$lt, $lte;

// collection inventory  quantité < 10
db.inventory.find({ quantity: { $lt: 20 } });
```

D'autres filtres :

```js
// différent de
$ne
"number" : {"$ne" : 10}

// fait partie de ...
$in, $nin
"notes" : {"$in" : [10, 12, 15, 18] }
"notes" : {"$nin" : [10, 12, 15, 18] }

// Ou
$or
"notes : { "$or": [{"$gt" : 10}, {"$lt" : 5} ] }
// and
$and

"notes : { "$and": [{"$gt" : 10}, {"$lt" : 5} ] }

// négation
$not
"notes" : {"$not" : {"$lt" : 10} }

// existe
$exists
"notes" : {"$exists" : true}

// tous les documents qui possède(nt) la propriété level
db.inventory.find( { level : { $exists: true } } )

// tous les documents qui ne possède(nt) pas la propriété level
db.inventory.find( { level : { $exists: false } } )

// test sur la taille d'une liste
$size
"notes" : {"$size" : 4}

// element match

/*
{
    "content" : [
        { "name" : <string>, year: <number>, by: <string> }
        ...
    ]
}
*/

{ "content": { $elemMatch: { "name": "Turing Award", "year": { $gt: 1980 } } } }

// recherche avec une Regex simple
$regex
{ "name": /^A/  }

```
## Regex présentation

Regex signifie en réalité Regular Expressions. Mais c'est un mot issu d'une fusion d'autres mots dont Regular Expressions en fait parti.

Les expressions rationnelles (regex) sont issues de la théorie des mathématiques et spécialementdes des langages formels définies dans les années 1940. Le regex ont la capacité à décrire **avec concision** des ensembles réguliers ou pattern.

Le standard POSIX a permi de normaliser la syntaxe et fonctionnalités des différents éditeurs de Regex.

Vous trouverez une librairie PCRE (Perl Compatible Regex) implémentée dans de nombreux langages comme dans Mongo et JS par exemple.

Pour utiliser une Regex complexe avec Mongo il faudra utiliser la syntaxe suivante :

```js
{ <field>: { $regex: /pattern/, $options: '<options>' } }
```

MongoDB utilise Perl compatible regular expressions (i.e. "PCRE" ) version 8.42 en 2021 avec le support UTF-8.

## Exercice

1. Combien y a t il de restaurants qui font de la cuisine italienne et qui ont eu un score de 10 au moins ?
   Affichez également le nom, les scores et les coordonnées GPS de ces restaurants. Ordonnez les résultats
   par ordre décroissant sur les noms des restaurants.

Remarque pour la dernière partie de la question utilisez la méthode sort :

```js
db.collection.findOne(query, restriction).sort({ key: 1 }); // 1 pour ordre croissant et -1 pour décroissant
```

2.1 Quels sont les restaurants qui ont eu un grade A avec un score supérieur ou égal à 20 en même temps ? Affichez uniquement les noms et ordonnez les par ordre décroissant. Affichez le nombre de résultat.

### correction 2.1

```js
db.restaurants
  .find({
    grades: {
      $elemMatch: {
        score: { $gte: 20 },
        grade: "A",
      },
    },
  },
  { _id : 0, name : 1 }
  )
  .sort({
    name: -1,
  })
  .pretty();
```

2.2 Quels sont les restaurants qui ont eu un grade A et un score supérieur ou égal à 20 ? Affichez uniquement les noms et ordonnez les par ordre décroissant. Affichez le nombre de résultat.

```js
db.restaurants
  .find({
    "grades.grade": "A",
    "grades.score": { $gte: 20 },
  })
  .sort({
    name: -1,
  })
  .pretty();
```

Remarque pour la dernière partie de la question utilisez la méthode count :

```js
db.collection.findOne(query, restriction).count();
```

- 3. A l'aide de la méthode distinct trouvez tous les quartiers distincts de NY.

```js
db.restaurants.distinct("borough");
```

- 4. Trouvez tous les types de restaurants dans le quartiers du Bronx. Vous pouvez là encore utiliser distinct et un deuxième paramètre pour préciser sur quel ensemble vous voulez appliquer cette close :

```js
db.restaurants.distinct("field", { key: "value" });
```

### correction

```js
db.restaurants.distinct("cuisine", { borough: "Bronx" });
```

- 5. Sélectionnez les restaurants dont le grade est A ou B dans le Bronx.

### Correction

```js
db.restaurants
  .find(
    {
      $or: [{ "grades.grade": "A" }, { "grades.grade": "B" }],
      borough: "Bronx",
    },
    {
      name: 1,
      _id: 0,
      "grades.grade": 1,
      borough: 1,
    }
  )
  .pretty();
```

- 6. Même question mais, on aimerait récupérer les restaurants qui ont eu à la dernière inspection un A ou B. Vous pouvez utilisez la notion d'indice sur la clé grade :

```js
"grades.2.grade";

```

### Correction

```js
db.restaurants
  .find(
    {
      $or: [{ "grades.0.grade": "A" }, { "grades.0.grade": "B" }],
      borough: "Bronx",
    },
    {
      name: 1,
      _id: 0,
      grades: 1,
      borough: 1,
    }
  )
  .pretty();
```

_Rechercher tous les grades distincts dans la collection._

```js
db.restaurants.distinct("grades.grade");
```

On aimerait maintenant avoir que des A ou que des B dans les notations des restaurants.

```js
db.restaurants.find(
    {
        "grades.grade" : "A",
        "grades.grade" : { $nin : ["B", "C", "Not Yet Graded", "P", "Z"]},
        "grades": { $not: { $size: 0 } },
        "borough": "Bronx"
    },
    {
        "name" : 1,
        "_id" : 0,
        "grades" : 1,
        "borough": 1
    }
).forEach(doc => {
  const json = tojson( doc.grades ) ;
  print(json);
})
```

- 7. Sélectionnez maintenant tous les restaurants qui ont le mot "Coffee" ou "coffee" dans la propriété name du document. Puis, même question mais uniquement dans le quartier du Bronx.

- 8. Trouvez tous les restaurants avec les mots Coffee ou Restaurant et qui ne contiennent pas le mot Starbucks. Puis, même question mais uniquement dans le quartier du Bronx.

- 9. Nouvelle question : Trouvez tous les restaurants qui ont dans leur nom le mot clé coffee, qui sont dans le bronx ou dans Brooklyn, qui ont eu exactement 4 appréciations (grades).

- 10. Affichez tous les noms de ces restaurants en majuscule avec leur dernière date et permière date d'évaluation.

- 11. Précisez également le quartier dans lequel ce restaurent se trouve.

Indications : utilisez les opérateurs suivants :

```js
{
  $size: 4;
}
ISODate("2012-10-24T00:00:00Z"); // UTC -2h par rapport à l'heure française
"bonjour".toUpperCase();
```

### Correction exercice 11

```js
db.restaurants.find({
    name: /coffee/i,
    borough: { $in: [/bronx/i, /Brooklyn/i] },
    grades: { $size: 4 }
},
    { "_id": 0, "name": 1, "grades.date": 1, borough: 1 }
).forEach(
    doc => {
        const { name, grades, borough } = doc;
        print(`Borough: ${borough}`);
        print();
        print(name.toUpperCase());
        grades.slice(0, 1).forEach(({ date }) => {
            print(`Last date : ${date.toDateString()}`);
        });
        grades.slice(-1).forEach(({ date }) => print(`First date ${date.toDateString()}`));
        print("----------------------------------")
    }
);
```

## Recherche de restaurents à proximité d'un lieu

MongoDB permet de gérér des points GPS. Dans la collection restaurants nous avons un champ address.coord qui correspond à des coordonnées GPS (longitude & latitude).

Nous allons utiliser les coordonnées sphériques de MongoDB. Pour l'implémenter dans la collection vous devez créer un index particulier sur le champ coord :

```js
db.restaurants.createIndex({ "address.coord": "2dsphere" });
```

### Exercice GPS

Après avoir créer l'index 2dsphere ci-dessus, trouvez tous les restaurants qui sont à 5 miles autour du point GPS suivant, donnez leurs noms, leur quartier ainsi que les coordonnées GPS en console, aidez-vous des indications ci-après :

```js
const coordinate = [-73.961704, 40.662942];
```

Indications : vous utiliserez la syntaxe suivante avec les opérateurs MongoDB :

```js
// opérateur
{ $nearSphere: { $geometry: { type: "Point", coordinates: coordinate }, $maxDistance: VOTRE_DISTANCE_EN_METRE } }
```

### Correction 

```js
const DISTANCE = 5 * 1609.34; // rayon 
const COORDINATE  = [-73.961704, 40.662942];

db.restaurants.find({
  "address.coord": { 
      $nearSphere: { $geometry: { type: "Point", coordinates: COORDINATE }, $maxDistance: DISTANCE } }
  }
)

```

## Recherche par rapport à la date

Sans exécutez la requête suivante, qu'affiche-t-elle ?

```js
db.restaurants.find(
  {
    "grades.0.date": ISODate("2013-12-30T00:00:00Z"),
  },
  { _id: 0, name: 1, borough: 1, "grades.date": 1 }
);
```

## Exercice 

Affichez tous les noms  des restaurants qui ont une appréciation (grades) dont toutes les dates sont supérieures à la date suivante:

```js
 ISODate("2012-10-24T00:00:00Z")
```

### Correction 

```js
 db.restaurants.find(
    { 
      "grades.date" : { $gte: ISODate("2012-10-24T00:00:00Z"), $not: { $lt: ISODate("2012-10-24T00:00:00Z") } } 
    }
  ,
  { _id: 0, name: 1, borough: 1, "grades.date": 1 }
).pretty();
```

### Exercices supplémentaires

1. Affichez la liste des restaurants dont le nom commence et se termine par une voyelle.

### Correction

```js
db.restaurants.find(
    { name: /^[ aeiouy ].*[ aeiouy ]$/i },
    {
        _id: 0,
        name: 1
    }
).forEach(doc => {
    const { name } = doc;
    print(`Name : ${name}`);
    print("----------------------------------");
});
```

2. Affichez la liste des restaurants dont le nom commence et se termine par une même lettre. Vous ferez attention à ne pas récupérer dans votre requête les restaurants n'ayant pas de nom. 

Remarque vous pouvez soit programmer cet affichage, soit directement utiliser une regex. Dans ce cas lisez les indications suivantes :

```js
() // les parenthèses captures une chaîne de caractère(s)
\1 // permet de récupérer la première chaîne de caractère(s) capturée(s)
\2 // permet de récupérer la deuxième chaîne de caractère(s) capturée(s)
```

### Correction

```js
// première correction
db.restaurants.find(
    { name: { $nin: [""] } },
    { name: 1, _id: 0 }
).forEach(doc => {
    const { name } = doc;
    if (name.toLowerCase().substr(0, 1) === name.toLowerCase().substr(-1)) {
        print(name);
        print("----------------------------------");
    }
})

// deuxième correction
db.restaurants.find(
    {
        name: { $nin: [""] },
        // regex : commence par une lettre de a à z en minuscule capturé
        // suivi de n'importe quel caractère(s) (0, infini) et se termine par
        // ce que les parenthèses ont capturé
        name: { $regex: /^([a-z]).*\1$/, $options: "i" }
    },
    { name: 1, _id: 0 }
).forEach(doc => {
    const { name } = doc;
    print(name);
    print("----------------------------------");
});
```

## Lire un document entièrement résumé

La méthode find permet de lire les documents dans une collection, par défaut elle ne retournera que 20 documents au maximum.

```js
db.restaurants.find();
```

Dans le terminal vous pouvez utiliser la commande it pour avancer dans la lecture du document.

- Utilisation d'un curseur pour lire le document :

```js
const resCursor1 = db.restaurants.find();

while (resCursor1.hasNext()) {
  print(tojson(resCursor1.next()));
}
```

Avec la méthode **foreEach** :

```js
const resCursor2 = db.restaurants.find();

resCursor2.forEach(printjson);
```

Vous pouvez également récupérez l'ensemble des documents dans un array :

```js
const resCursor3 = db.restaurants.find();
const resArray = resCursor3.toArray();
print(resArray[3].name);
```
