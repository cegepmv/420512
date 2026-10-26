+++
date = '2026-06-25T16:02:29-04:00'
draft = false
weight = 12
pre="12. "
title = 'Backend'
+++

## 1. Mise en place de Node.js

Pour débuter avec Node.js, nous utilisons **fnm** (Fast Node Manager) :

1. Installez `curl` et `unzip` :
```bash
apt update && apt install -y curl unzip

```


2. Installez et configurez `fnm` et Node.js (version 22 LTS) :
```bash
curl -fsSL https://fnm.vercel.app/install | bash
source ~/.bashrc
fnm use --install-if-missing 22
node -v
npm -v

```


3. Créez un dossier `CRUD` et ouvrez-le dans VS Code.
4. Initialisez le projet :
```bash
npm init -y

```


5. Configurez `package.json` pour les modules ES et nodemon :
```json
{
  "type": "module",
  "scripts": {
    "dev": "npx nodemon app.js"
  }
}

```


6. Installez les dépendances :
```bash
npm install --save-dev nodemon
npm i mysql2 dotenv express

```


---

## Variables d'environnement

Les variables d'environnement sont utilisées pour sécuriser et paramétrer les applications sans stocker directement ces informations dans le code source. Elles permettent de séparer la configuration des applications (comme les clés d'API, les identifiants de base de données, etc.) de leur logique métier, facilitant ainsi la gestion et le déploiement dans différents environnements (développement, test, production). En outre, l'utilisation de variables d'environnement améliore la sécurité en évitant l'exposition de données sensibles dans le code source et en permettant des ajustements rapides et sûrs de la configuration sans nécessiter de modifications de code.

### Préparation

1. Créez un fichier appelé `.env` à la racine de votre projet
2. Ajoutez-y :
```env
MYSQL_HOST='127.0.0.1'
MYSQL_USER='root'
MYSQL_PASSWORD='`votre mdp`'
MYSQL_DATABASE='`Le nom de votre db`'

```


3. `npm i dotenv` : installe la librairie qui saura gérer les variables d'environnement
4. Lorsque vous voudrez utiliser des variables d'environnement il suffira d'ajouter ce code à votre fichier `.js` :
```javascript
import dotenv from 'dotenv'
dotenv.config()

```


5. Ensuite vous n'aurez qu'à préfixer les variables par `process.env` pour y avoir accès (Ex : `process.env.MYSQL_HOST`)

---


## Connexion à la DB

Créez un fichier `database.js` :
```javascript
import mysql from 'mysql2';
import dotenv from 'dotenv';

dotenv.config();

const pool = mysql.createPool({
    host: process.env.MYSQL_HOST,
    user: process.env.MYSQL_USER,
    password: process.env.MYSQL_PASSWORD,
    database: process.env.MYSQL_DATABASE
}).promise();

export default pool;

```

Exemple d'utilisation : 

```js

const [resultat] = await pool.query("SELECT * FROM etudiant")
console.log(resultat)
                
```

{{% notice style="exo"%}}

1. Créez la table `parc_attraction` :
```sql
CREATE TABLE parc_attraction ( 
    id INT AUTO_INCREMENT PRIMARY KEY,
    nom VARCHAR(100), 
    ville VARCHAR(500), 
    cout_billet DECIMAL(6,2) 
);

```


2. Écrivez un script Node.js permettant de récupérer tous les parcs, d'en insérer deux nouveaux, de réafficher la liste, puis de les supprimer proprement.
{{% /notice %}}

---


## 3. Programmation asynchrone et Promesses


La page de [w3school](https://www.w3schools.com/js/js_promise.asp) explique bien ce que c'est qu'une `promise`, mais voici ma traduction francaise :

Une `promise` est séparé en 2 parties: action et réaction.

* *Action* : code qui peux prendre un certain temps.
* *Réaction* : code qui doit attendre la fin de l'exécution pour s'effectuer.

```javascript
let myPromise = new Promise(function(resolve, reject) {
    // "action" (Peux prendre un certain temps)
    
        resolve(); // Fonction pour retourner un succès
        reject();  // Fonction pour retourner une erreur
    });
    
    // "Réaction" (Doit attendre le retour de l'action)
    myPromise.then(
        function(value) { /* code effectué si action terminé avec resolve() */ },
        function(error) { /* code effectué si action terminé avec reject()  */ }
    );

```

À quoi ça sert d'avoir une séparation entre le code qui doit être effectué avant et après la terminaison de l'action, car le code s'effectue de haut en bas? Dans le cas de code synchrone il est inutile d'utiliser des `promise`, mais il y a plusieurs moments où vous serez obliger d'utiliser de l'asynchrone pour des raisons de performances.

---

## Programmation asynchrone

La programmation asynchrone est utilisée principalement pour améliorer l'efficacité et la réactivité des applications, en particulier dans des environnements où des opérations d'entrée-sortie (I/O) sont fréquentes. Voici quelques utilisations principales de la programmation asynchrone :

1. *Gestion des Entrées/Sorties (I/O)* : Dans les applications web, les opérations telles que les requêtes réseau (API, base de données), la lecture/écriture de fichiers, et autres, peuvent prendre du temps. Utiliser des fonctions asynchrones permet à l'application de continuer à fonctionner pendant que ces opérations sont en cours, évitant ainsi le blocage du thread principal.
2. *Réactivité de l'Interface Utilisateur* : Dans les interfaces utilisateur (UI), les opérations longues, comme le traitement de données ou les calculs intensifs, peuvent rendre l'application non réactive si elles sont exécutées de manière synchrone. En rendant ces opérations asynchrones, l'UI reste fluide et réactive pour l'utilisateur.
3. *Traitement des Événements* : Dans les environnements où des événements peuvent se produire de manière asynchrone (par exemple, des clics utilisateur, des notifications), la programmation asynchrone permet de gérer efficacement ces événements sans interrompre le flux principal de l'application.
4. *Performances et Scalabilité* : En permettant à plusieurs tâches d'être exécutées simultanément ou en parallèle, la programmation asynchrone améliore les performances globales de l'application. Cela est particulièrement important dans les applications nécessitant une gestion efficace des ressources et une scalabilité pour gérer de grandes quantités de données ou de trafic.
5. *Optimisation du Temps de Chargement* : Sur les applications web, le chargement asynchrone des ressources telles que les images, les scripts ou les styles permet d'améliorer la vitesse de chargement global de la page, en chargeant les éléments de manière non bloquante.

En résumé, la programmation asynchrone est essentielle pour améliorer la réactivité, l'efficacité et les performances des applications modernes, en particulier celles qui interagissent fréquemment avec des ressources externes ou nécessitent un traitement intensif sans bloquer le thread principal.

### Exemple

Exécutons d'abors une simulation d'aquisition de données. Créez vous un fichier `test.html` et copiez-y le code. Ensuite utiliser l'extenstion Live Server pour lancer ce fichier dans votre navigateur. Sinon faites un fichier `test.js` retirer les balise script et tester avec `node test.js`

```javascript
function fetchDataSync() {
    try {
        // Pour transformer des promesses asynchrones en une simulation de code synchrone il faut les traiters en cascade.        
        // Ceci est à la fois horrible à lire et tout à fait inefficace dans le cas où les fonctions ne dépendent pas l'une de l'autre.
        fetchWeatherData().then(res => {
            const weatherData = res;
            fetchNewsData().then(resp => {
                const newsData = resp;
                fetchStocksData().then(response =>{
                    const stocksData = response;
                    // Process data
                    console.log("Sync : ", weatherData, newsData, stocksData);
                })})})
        
    } catch (error) {
        console.error('Error fetching data:', error);
    }
}
 // Sim-1
function fetchWeatherData() {
    
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ temperature: 25, condition: 'Sunny' });
        }, 1000);
    });
}
 // Sim-2
function fetchNewsData() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(['Article 1', 'Article 2', 'Article 3']);
        }, 1500);
    });
}
 // Sim-3
function fetchStocksData() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ symbol: 'AAPL', price: 150.25 });
        }, 2000);
    });
}


// La meilleure solution beacoup plus rapide est d'utiliser async/await
// Vous remarquerez la lisibilité accrue et en testant vous verrez la différence en rapidité.
async function fetchDataAsync() {
    try {
        const weatherPromise = fetchWeatherDataAsync();
        const newsPromise = fetchNewsDataAsync();
        const stocksPromise = fetchStocksDataAsync();

        const [weatherData, newsData, stocksData] = await Promise.all([weatherPromise, newsPromise, stocksPromise]);

        // Process data
        
        console.log("Async : ", weatherData, newsData, stocksData);
    } catch (error) {
        console.error('Error fetching data:', error);
    }
}

async function fetchWeatherDataAsync() {
    // Simulate fetching weather data asynchronously
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ temperature: 25, condition: 'Sunny' });
        }, 1000);
    });
}

async function fetchNewsDataAsync() {
    // Simulate fetching news data asynchronously
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(['Article 1', 'Article 2', 'Article 3']);
        }, 1500);
    });
}

async function fetchStocksDataAsync() {
    // Simulate fetching stocks data asynchronously
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ symbol: 'AAPL', price: 150.25 });
        }, 2000);
    });
}
fetchDataSync();
fetchDataAsync();

```

*async* : Déclare qu'une fonction retourne une promesse. Une fonction marquée async peut contenir des expressions await, qui mettent en pause l'exécution de la fonction asynchrone jusqu'à ce que la promesse soit résolue ou rejetée.

*await* : Met en pause l'exécution de la fonction asynchrone jusqu'à ce que la promesse soit résolue ou rejetée. On utilise await uniquement à l'intérieur des fonctions déclarées avec async.

*Promise.all* est une méthode statique qui prend un tableau de promesses en entrée et retourne une nouvelle promesse qui est résolue lorsque toutes les promesses du tableau d'entrée sont résolues, ou rejetée si l'une des promesses est rejetée.



{{% notice style="exo" %}}

Créez 4 méthodes qui simulent la connexion à une base de données. Elles auront pour nom `fetchLongueur`, `fetchLargeur`, `fetchProfondeur` et `fetchMasseVolumique`. Chacune de ces fonctions retournera une promesse qui se résoudra en 1 seconde et qui retournera respectivement pour chaque fonction : `2`, `1`, `4` et `5`.

1. Faites un programme nommé `promise.js` qui calcule la masse sans utiliser `async`, `await` ou de callback.
2. Faites un programme nommé `async.js` qui calcule la masse avec `async` et `await`.

{{% expand title="solution" %}}

#### 1. `promise.js`

```javascript
function calculMasse() {
    try {
        // Pour transformer des promesses asynchrones en une simulation de code synchrone il faut les traiter en cascade.        
        // Ceci est à la fois horrible à lire et tout à fait inefficace dans le cas où les fonctions ne dépendent pas l'une de l'autre.
        fetchLongueur().then(res => {
            let masse = res;
            fetchLargeur().then(resp => {
                masse *= resp;
                fetchProfondeur().then(response => {
                    masse *= response;
                    fetchMasseVolumique().then(response2 => {
                        masse *= response2;
                        // Process data
                        console.log("Sync : ", masse);
                    });
                });
            });
        });
        
    } catch (error) {
        console.error('Error fetching data:', error);
    }
}

// Sim-1
function fetchLongueur() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(2);
        }, 1000);
    });
}

// Sim-2
function fetchLargeur() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(1);
        }, 1000);
    });
}

// Sim-3
function fetchProfondeur() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(4);
        }, 1000);
    });
}

// Sim-4
function fetchMasseVolumique() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(5);
        }, 1000);
    });
}

calculMasse();

```

#### 2. `async.js`

```javascript
async function calculMasseAsync() {
    try {
        const longueur = fetchAsyncLongueur();
        const largeur = fetchAsyncLargeur();
        const profondeur = fetchAsyncProfondeur();
        const masseVolumique = fetchAsyncMasseVolumique();

        const [longueurData, largeurData, profondeurData, masseVolumiqueData] = await Promise.all([longueur, largeur, profondeur, masseVolumique]);
        console.log("Async : ", longueurData * largeurData * profondeurData * masseVolumiqueData);
    } catch (error) {
        console.error('Error fetching data:', error);
    }
}

// Sim-1
async function fetchAsyncLongueur() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(2);
        }, 1000);
    });
}

// Sim-2
async function fetchAsyncLargeur() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(1);
        }, 1000);
    });
}

// Sim-3
async function fetchAsyncProfondeur() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(4);
        }, 1000);
    });
}

// Sim-4
async function fetchAsyncMasseVolumique() {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(5);
        }, 1000);
    });
}

calculMasseAsync();

```

{{% /expand %}}
{{% /notice %}}

---

## 4. Requêtes backend



```sql
CREATE TABLE parc_attraction ( 
    id INT AUTO_INCREMENT PRIMARY KEY,
    nom VARCHAR(100), 
    ville VARCHAR(500), 
    cout_billet DECIMAL(6,2) 
);

```

```sql
INSERT INTO parc_attraction (nom, ville, cout_billet) VALUES ('La Ronde','Montréal','55.99'),('Disney','Orlando','99.49');"

```

---

Dans `database.js` nous allons maintenant créer des requêtes simples :

```javascript
// Ce code fonctionne car await peut être utilisé au `Top-level`
const [resultat] = await pool.query("SELECT * FROM parc_attraction");
console.log(resultat);

// Si vous prévoyez utiliser ce code plus d'une fois, il est plus propre de le mettre dans une fonction async
async function getParcAttraction() {
     const [rows] = await pool.query("SELECT * FROM parc_attraction");
     return rows;
}
const parc = await getParcAttraction();
console.log(parc);

```

## Pratique - Requêtes

1. Faites un code qui affiche les lignes de la table `parc_attraction`, en ajoute 2 nouvelles, affiche encore une fois toutes les lignes, puis supprime les 2 lignes créées et affiche une dernière fois toutes les lignes de la table.
2. En utilisant le code suivant :
```javascript
async function getParcAttraction(id) {
    return new Promise(resolve => {
        setTimeout(async () => {
            const demande = "SELECT * FROM parc_attraction WHERE id=" + id;
            const [rows] = await pool.query(demande);
            resolve(rows[0]);
        }, 4000);
    });
}

```


Vous devez utiliser ce code pour afficher `"Disney La Ronde"` en utilisant seulement leur identifiant (`id`), cette fonction, et ce en moins de 5 secondes. *(Indice : vous avez droit à `Promise.all`)*

### Solution

#### 1. Manipulation des parcs (Affichage, Ajout, Suppression)

```javascript
// Si vous prévoyez utiliser ce code plus d'une fois, il est plus propre de le mettre dans une fonction async
async function getParcAttraction() {
     const [rows] = await pool.query("SELECT * FROM parc_attraction;");
     return rows;
}

async function addParcAttraction(nom, ville, cout) {
    const [rows] = await pool.query(`INSERT INTO parc_attraction (nom, ville, cout_billet) VALUES ('${nom}','${ville}','${cout}');`);
    return rows;
}

async function removeParcAttractionByName(nom) {
    const [rows] = await pool.query(`DELETE FROM parc_attraction WHERE nom='${nom}';`);
    return rows;
}

const parc = await getParcAttraction();
console.log(parc);

const ajout1 = await addParcAttraction('Super Aqua Club', 'Pointe-Calumet', 60);
const ajout2 = await addParcAttraction('Bromont', 'Bromont', 50);

const parc2 = await getParcAttraction();
console.log(parc2);

const ret1 = await removeParcAttractionByName("Super Aqua Club");
const ret2 = await removeParcAttractionByName("Bromont");

const parc3 = await getParcAttraction();
console.log(parc3);

```

#### 2. Affichage optimisé avec `Promise.all`

```javascript
import mysql from 'mysql2';
import dotenv from 'dotenv';
dotenv.config();

const pool = mysql.createPool({
    host: process.env.MYSQL_HOST,
    user: process.env.MYSQL_USER,
    password: process.env.MYSQL_PASSWORD,
    database: process.env.MYSQL_DATABASE
}).promise();

async function getParcAttraction(id) {
    return new Promise(resolve => {
        setTimeout(async () => {
            const demande = "SELECT * FROM parc_attraction WHERE id=" + id;
            const [rows] = await pool.query(demande);
            resolve(rows[0]);
        }, 4000);
    });    
}

const parc1 = getParcAttraction(1);
const parc2 = getParcAttraction(2);
const [parc1Data, parc2Data] = await Promise.all([parc1, parc2]);
console.log(parc2Data.nom + " " + parc1Data.nom);

```

## Requête préparée

1. **Sécurité contre les injections SQL** : Les requêtes préparées aident à prévenir les attaques par injection SQL en séparant les données des instructions SQL. Avec les requêtes préparées, les valeurs des paramètres sont transmises séparément de la requête SQL elle-même, ce qui empêche les attaquants d'injecter du code SQL malveillant via les données utilisateur. Cela garantit une meilleure sécurité des données et protège contre les vulnérabilités courantes comme les attaques par injection SQL.
2. **Performance optimisée** : Les bases de données peuvent optimiser l'exécution de requêtes préparées en les compilant une seule fois et en réutilisant le plan d'exécution pour les requêtes similaires avec des paramètres différents. Cela peut réduire la charge sur la base de données et améliorer les performances, surtout lorsque les mêmes requêtes sont exécutées fréquemment avec différents paramètres.
3. **Clarté du code et maintenance simplifiée** : Les requêtes préparées rendent le code plus lisible et plus facile à maintenir en séparant clairement la logique de la requête SQL de ses données. Cela facilite la gestion des modifications ultérieures de requêtes sans avoir à réécrire le code SQL principal.

Reprenons l'exemple précédent sans le timeout et transformons la requête en requête préparée :

```javascript
async function getParcAttraction(id) {
     const [rows] = await pool.query(`SELECT * FROM parc_attraction WHERE id=?`, [id]);
     return rows[0];
}

```

---

## Exercices

Faites un CRUD basique avec les fonctions async suivantes :

1. Création d'un parc d'attraction en prenant les données en paramètre.
2. Lire une ligne en donnant l'id en paramètre.
3. Lire toutes les lignes de la table.
4. Modifier une ligne de la table en prenant en paramètre toutes les données (id, nom, ville, coût du billet).
5. Supprimer une ligne en prenant en paramètre l'id.

{{% expand title="Solution"%}}

```javascript
import mysql from 'mysql2';
import dotenv from 'dotenv';
dotenv.config();

const pool = mysql.createPool({
    host: process.env.MYSQL_HOST,
    user: process.env.MYSQL_USER,
    password: process.env.MYSQL_PASSWORD,
    database: process.env.MYSQL_DATABASE
}).promise();

// Si vous prévoyez d'utiliser ce code plus d'une fois, il est plus propre de le mettre dans une fonction async
async function addParcAttraction(nom, ville, cout) {
    const [rows] = await pool.query(`INSERT INTO parc_attraction (nom, ville, cout_billet) VALUES (?,?,?);`, [nom, ville, cout]);
    return rows;
}

async function getParcAttraction(id) {
    const [rows] = await pool.query("SELECT * FROM parc_attraction where id=?;", [id]);
    return rows;
}

// Si vous prévoyez d'utiliser ce code plus d'une fois, il est plus propre de le mettre dans une fonction async
async function getParcAttractions() {
     const [rows] = await pool.query("SELECT * FROM parc_attraction;");
     return rows;
}

// Si vous prévoyez d'utiliser ce code plus d'une fois, il est plus propre de le mettre dans une fonction async
async function updateParcAttraction(id, nom, ville, cout) {
    const [rows] = await pool.query(`UPDATE parc_attraction 
                                     SET nom=?, ville=?, cout_billet=?
                                     WHERE id=?;`, [nom, ville, cout, id]);
    return rows;
}

// Si vous prévoyez d'utiliser ce code plus d'une fois, il est plus propre de le mettre dans une fonction async
async function removeParcAttractionByName(nom) {
    const [rows] = await pool.query(`DELETE FROM parc_attraction 
                                     WHERE nom=?;`, [nom]);
    return rows;
}

const step1 = await getParcAttraction(1);
console.log(step1);

const step2 = await addParcAttraction("Bidon", "lol", 1000);
const step3 = await getParcAttractions();
console.log(step3);

const step4 = await updateParcAttraction(1, "changed", "Montréal", 10);
const step5 = await getParcAttractions();
console.log(step5);

const step6 = await updateParcAttraction(1, "La Ronde", "Montréal", 55.99);
const step7 = await removeParcAttractionByName("Bidon");
const step8 = await getParcAttractions();
console.log(step8);

```

{{% /expand %}}

---

## 5. Démo d'un CRUD complet avec Express (`app.js`)

```javascript
import express from 'express';
import dotenv from 'dotenv';
import pool from './database.js';

dotenv.config();

const app = express();
const port = process.env.PORT || 3000;

app.use(express.json());

// CREATE
app.post('/parcs', async (req, res) => {
    try {
        const { nom, ville, cout_billet } = req.body;
        const [result] = await pool.query(
            "INSERT INTO parc_attraction (nom, ville, cout_billet) VALUES (?, ?, ?)",
            [nom, ville, cout_billet]
        );
        res.status(201).json({ id: result.insertId, nom, ville, cout_billet });
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// READ (Tous)
app.get('/parcs', async (req, res) => {
    try {
        const [parcs] = await pool.query("SELECT * FROM parc_attraction");
        res.json(parcs);
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// READ (Un seul)
app.get('/parcs/:id', async (req, res) => {
    try {
        const [parcs] = await pool.query("SELECT * FROM parc_attraction WHERE id = ?", [req.params.id]);
        if (parcs.length === 0) return res.status(404).json({ erreur: "Non trouvé" });
        res.json(parcs[0]);
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// UPDATE
app.put('/parcs/:id', async (req, res) => {
    try {
        const { nom, ville, cout_billet } = req.body;
        const [result] = await pool.query(
            "UPDATE parc_attraction SET nom = ?, ville = ?, cout_billet = ? WHERE id = ?",
            [nom, ville, cout_billet, req.params.id]
        );
        if (result.affectedRows === 0) return res.status(404).json({ erreur: "Non trouvé" });
        res.json({ message: "Mis à jour avec succès !" });
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// DELETE
app.delete('/parcs/:id', async (req, res) => {
    try {
        const [result] = await pool.query("DELETE FROM parc_attraction WHERE id = ?", [req.params.id]);
        if (result.affectedRows === 0) return res.status(404).json({ erreur: "Non trouvé" });
        res.json({ message: "Supprimé avec succès !" });
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// Lorsqu'une erreur se produit dans l'application (par exemple, une exception non gérée), Express appelle automatiquement
// ce middleware d'erreur avec l'objet d'erreur (err), ce qui permet de la gérer de manière centralisée et uniforme.
app.use((err, req, res, next) => {
    console.error(err.stack)
    res.status(1045).send('Something broke!')
})

app.listen(port, () => {
    console.log(`Serveur démarré sur http://localhost:${port}`);
});

```



## RESTful

Pour la version longue des explications, visitez le [site](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design) par Microsoft.

Les API REST (Representational State Transfer) sont un ensemble de conventions pour concevoir des services web qui permettent aux applications de communiquer entre elles. Voici les concepts clés :

### 1. Ressources

Les API REST manipulent des "ressources", qui sont des objets ou des données. Chaque ressource est identifiée par une URL unique (ex. : `/utilisateurs/123`).

### 2. Méthodes HTTP

Les API REST utilisent les méthodes HTTP pour effectuer des opérations sur les ressources :

* **GET** : Récupérer une ressource.
* **POST** : Créer une nouvelle ressource.
* **PUT** : Mettre à jour une ressource existante.
* **DELETE** : Supprimer une ressource.

### 3. Formats de données

Les réponses des API REST sont généralement formatées en JSON (JavaScript Object Notation) ou XML. JSON est le plus courant en raison de sa simplicité et de sa légèreté.

### 4. Utilisation des Codes de Statut HTTP

Les API REST utilisent des codes de statut HTTP pour indiquer le résultat d'une requête :

* **200 OK** : Requête réussie.
* **201 Created** : Nouvelle ressource créée avec succès.
* **204 No Content** : Requête réussie, mais aucune donnée à retourner.
* **400 Bad Request** : Erreur de la part du client.
* **404 Not Found** : La ressource demandée n'existe pas.
* **500 Internal Server Error** : Erreur côté serveur.


# FE

### 1. Installation d'Axios dans votre projet Expo

Avant de lancer le code, assurez-vous d'avoir installé Axios :

```bash
npm i axios

```

---

### 2. Code complet pour `index.jsx` avec Axios

```javascript
import React, { useState, useEffect } from 'react';
import { 
  StyleSheet, 
  Text, 
  View, 
  FlatList, 
  TextInput, 
  TouchableOpacity, 
  ActivityIndicator, 
  Alert, 
  KeyboardAvoidingView, 
  Platform 
} from 'react-native';
import axios from 'axios';

// ⚠️ N'oubliez pas de remplacer par l'IP de votre ordinateur 
const API_URL = 'http://192.168.1.X:3000/parcs';

export default function App() {
  const [parcs, setParcs] = useState([]);
  const [loading, setLoading] = useState(true);

  // États pour le formulaire d'ajout
  const [nom, setNom] = useState('');
  const [ville, setVille] = useState('');
  const [coutBillet, setCoutBillet] = useState('');

  // 1. READ - Récupérer tous les parcs (GET)
  const fetchParcs = async () => {
    try {
      setLoading(true);
      const response = await axios.get(API_URL);
      // Avec Axios, les données sont directement dans response.data
      setParcs(response.data);
    } catch (error) {
      Alert.alert("Erreur", error.message || "Erreur lors du chargement des parcs.");
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchParcs();
  }, []);

  // 2. CREATE - Ajouter un nouveau parc (POST)
  const ajouterParc = async () => {
    if (!nom || !ville || !coutBillet) {
      Alert.alert("Attention", "Veuillez remplir tous les champs.");
      return;
    }

    try {
      // Axios transforme automatiquement l'objet JavaScript en JSON dans le body
      await axios.post(API_URL, {
        nom,
        ville,
        cout_billet: parseFloat(coutBillet)
      });

      // Vider le formulaire et rafraîchir la liste
      setNom('');
      setVille('');
      setCoutBillet('');
      fetchParcs();
    } catch (error) {
      Alert.alert("Erreur", error.response?.data?.erreur || "Erreur lors de l'ajout du parc.");
    }
  };

  // 3. DELETE - Supprimer un parc (DELETE)
  const supprimerParc = async (id) => {
    Alert.alert(
      "Confirmation",
      "Voulez-vous vraiment supprimer ce parc ?",
      [
        { text: "Annuler", style: "cancel" },
        { 
          text: "Supprimer", 
          style: "destructive",
          onPress: async () => {
            try {
              await axios.delete(`${API_URL}/${id}`);

              // Mise à jour locale immédiate de l'interface
              setParcs(parcs.filter(parc => parc.id !== id));
            } catch (error) {
              Alert.alert("Erreur", "Erreur lors de la suppression.");
            }
          }
        }
      ]
    );
  };

  return (
    <KeyboardAvoidingView 
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'} 
      style={styles.container}
    >
      <Text style={styles.headerTitle}>🎢 Gestion des Parcs (Axios)</Text>

      {/* Formulaire d'ajout */}
      <View style={styles.formContainer}>
        <TextInput
          style={styles.input}
          placeholder="Nom du parc"
          value={nom}
          onChangeText={setNom}
        />
        <TextInput
          style={styles.input}
          placeholder="Ville"
          value={ville}
          onChangeText={setVille}
        />
        <TextInput
          style={styles.input}
          placeholder="Coût du billet ($)"
          keyboardType="numeric"
          value={coutBillet}
          onChangeText={setCoutBillet}
        />
        <TouchableOpacity style={styles.addButton} onPress={ajouterParc}>
          <Text style={styles.addButtonText}>Ajouter le parc</Text>
        </TouchableOpacity>
      </View>

      {/* Liste des parcs */}
      {loading ? (
        <View style={styles.center}>
          <ActivityIndicator size="large" color="#007AFF" />
        </View>
      ) : (
        <FlatList
          data={parcs}
          keyExtractor={(item) => item.id.toString()}
          renderItem={({ item }) => (
            <View style={styles.card}>
              <View style={styles.cardInfo}>
                <Text style={styles.parcName}>{item.nom}</Text>
                <Text style={styles.parcDetails}>📍 {item.ville}</Text>
                <Text style={styles.parcDetails}>💰 {item.cout_billet} $</Text>
              </View>
              
              <TouchableOpacity 
                style={styles.deleteButton} 
                onPress={() => supprimerParc(item.id)}
              >
                <Text style={styles.deleteButtonText}>❌</Text>
              </TouchableOpacity>
            </View>
          )}
          ListEmptyComponent={
            <Text style={styles.emptyText}>Aucun parc trouvé.</Text>
          }
        />
      )}
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: { 
    flex: 1, 
    backgroundColor: '#f8f9fa', 
    paddingTop: 60, 
    paddingHorizontal: 20 },
  headerTitle: { 
    fontSize: 22, 
    fontWeight: 'bold', 
    marginBottom: 20, 
    textAlign: 'center', 
    color: '#333' },
  formContainer: { 
    backgroundColor: '#fff', 
    padding: 15, 
    borderRadius: 10, 
    marginBottom: 20, 
    shadowColor: '#000', 
    shadowOpacity: 0.05, 
    elevation: 2 },
  input: { 
    borderWidth: 1, 
    borderColor: '#ddd', 
    padding: 10, 
    borderRadius: 6, 
    marginBottom: 10, 
    backgroundColor: '#fdfdfd' },
  addButton: { 
    backgroundColor: '#007AFF', 
    padding: 12, 
    borderRadius: 6, 
    alignItems: 'center' },
  addButtonText: { 
    color: '#fff', 
    fontWeight: 'bold', 
    fontSize: 16 },
  center: { 
    flex: 1, 
    justifyContent: 'center', 
    alignItems: 'center' },
  card: { 
    flexDirection: 'row', 
    backgroundColor: '#fff', 
    padding: 15, 
    borderRadius: 8, 
    marginBottom: 10, 
    alignItems: 'center', 
    shadowColor: '#000', 
    shadowOpacity: 0.05, 
    elevation: 2 },
  cardInfo: { 
    flex: 1 },
  parcName: { 
    fontSize: 16, 
    fontWeight: 'bold', 
    color: '#333', 
    marginBottom: 4 },
  parcDetails: { 
    fontSize: 14, 
    color: '#666' },
  deleteButton: { 
    padding: 10 },
  deleteButtonText: { 
    fontSize: 18 },
  emptyText: { 
    textAlign: 'center', 
    color: '#888', 
    marginTop: 20 }
});

```



{{% notice style="exo" %}}
Reprenez la solution de l'exercice du Gestionnaire de tâche. Remplacer l'enregistrement local par de l'enregistrement sur bd distante.
{{% /notice %}}