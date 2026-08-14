+++
date = '2026-06-26T10:07:43-04:00'
draft = true
title = 'Setup'
+++


### Étape 1 : Créez le fichier de contenu

Créez un fichier nommé `index.md` (ou `_index.md` selon votre structure de section) dans votre dossier `content/` (par exemple : `content/node-crud/index.md`).

### Étape 2 : Collez le code suivant

```markdown
---
title: "Mise en place de Node"
linkTitle: "Node & CRUD"
weight: 10
description: "Guide d'apprentissage sur Node.js, les variables d'environnement, les promesses, l'asynchronisme et les requêtes MySQL2."
---

## Mise en place de Node

1. Installer curl et unzip : `apt install curl unzip`
2. Installer et activer le gestionnaire de versions (Fast Node Manager) :
   ```bash
   # installs fnm (Fast Node Manager)
   curl -fsSL [https://fnm.vercel.app/install](https://fnm.vercel.app/install) | bash

   # activate fnm
   source ~/.bashrc

   # download and install Node.js
   fnm use --install-if-missing 22

   # verifies the right Node.js version is in the environment
   node -v # should print `v22.5.1`

   # verifies the right npm version is in the environment
   npm -v # should print `10.8.2`

```

3. Créez vous un dossier nommé **CRUD** et ouvrez-le dans VSCode.
4. `npm init -y` : démarrer un nouveau projet Node.js, en sautant les étapes interactives de configuration grâce à l'acceptation automatique des valeurs par défaut.
5. Pour pouvoir utiliser `import` et `export` ainsi que permettre le redémarrage automatique, modifiez ces 2 lignes dans le fichier `package.json` :
```json
"type": "module",
"scripts": {
    "dev": "npx nodemon app.js"
},

```


6. `npm install --save-dev nodemon` : On utilisera cette librairie pour redémarrer automatiquement le serveur.
7. `npm i mysql2` : `mysql2` est une librairie pour Node.js qui permet aux applications d'interagir avec des bases de données MySQL.

---

## Variables d'environnement

Les variables d'environnement sont utilisées pour sécuriser et paramétrer les applications sans stocker directement ces informations dans le code source. Elles permettent de séparer la configuration des applications (comme les clés d'API, les identifiants de base de données, etc.) de leur logique métier, facilitant ainsi la gestion et le déploiement dans différents environnements (développement, test, production).

En outre, l'utilisation de variables d'environnement améliore la sécurité en évitant l'exposition de données sensibles dans le code source et en permettant des ajustements rapides et sûrs de la configuration sans nécessiter de modifications de code.

### Préparation

1. Créez un fichier appelé `.env` à la racine de votre projet.
2. Ajoutez-y :
```ini
MYSQL_HOST='127.0.0.1'
MYSQL_USER='root'
MYSQL_PASSWORD='votre mdp'
MYSQL_DATABASE='Le nom de votre db'

```


3. `npm i dotenv` : installe la librairie qui saura gérer les variables d'environnement.
4. Lorsque vous voudrez utiliser des variables d'environnement, il suffira d'ajouter ce code à votre fichier `.js` :
```js
import dotenv from 'dotenv'
dotenv.config()

```


5. Ensuite vous n'aurez qu'à préfixer les variables par `process.env` pour y avoir accès (Ex : `process.env.MYSQL_HOST`).

---

## Connexion à la BD

Voici le code vous permettant de vous connecter à la BD, mettez-le dans un fichier nommé *database.js*. Vous devez au préalable avoir créé vos variables dans le fichier `.env` qui se situe à la racine du projet.

```js
import mysql from 'mysql2';
import dotenv from 'dotenv';
dotenv.config();

const pool = mysql.createPool({
    host: process.env.MYSQL_HOST,
    user: process.env.MYSQL_USER,
    password: process.env.MYSQL_PASSWORD,
    database: process.env.MYSQL_DATABASE
}).promise()

```

Tapez `node database.js` pour tester le fichier.

Tester en faisant une requête. Ajouter simplement le code suivant pour le test, puis retirez-le :

```js
const [resultat] = await pool.query("SELECT * FROM etudiant")
console.log(resultat)

```

---

## Promise

La page de [W3Schools](https://www.w3schools.com/js/js_promise.asp) explique bien ce qu'est une `Promise`, mais voici une explication rapide :

Une `Promise` est séparée en 2 parties : **Action** et **Réaction**.

> **Action** : code qui peut prendre un certain temps.
> **Réaction** : code qui doit attendre la fin de l'exécution pour s'effectuer.

```js
let myPromise = new Promise(function(resolve, reject) {
    // "action" (Peut prendre un certain temps)
    
    resolve(); // Fonction pour retourner un succès
    reject();  // Fonction pour retourner une erreur
});

// "Réaction" (Doit attendre le retour de l'action)
myPromise.then(
    function(value) { /* code effectué si action terminé avec resolve() */ },
    function(error) { /* code effectué si action terminé avec reject()  */ }
);

```

À quoi ça sert d'avoir une séparation entre le code qui doit être effectué avant et après la terminaison de l'action, vu que le code s'exécute habituellement de haut en bas ? Dans le cas de code synchrone, il est inutile d'utiliser des `Promise`, mais il y a plusieurs moments où vous serez obligé d'utiliser de l'asynchronisme pour des raisons de performances.

---

## Programmation asynchrone

La programmation asynchrone est utilisée principalement pour améliorer l'efficacité et la réactivité des applications, en particulier dans des environnements où des opérations d'entrée-sortie (I/O) sont fréquentes.

1. **Gestion des Entrées/Sorties (I/O)** : Dans les applications web, les opérations telles que les requêtes réseau (API, base de données) peuvent prendre du temps. Utiliser des fonctions asynchrones permet à l'application de continuer à fonctionner pendant ce temps.
2. **Réactivité de l'Interface Utilisateur** : Évite le gel de l'affichage de l'application pendant les lourds traitements.
3. **Traitement des Événements** : Permet de gérer les actions de l'utilisateur en tâche de fond.
4. **Performances et Scalabilité** : Permet à plusieurs tâches d'être exécutées simultanément.
5. **Optimisation du Temps de Chargement** : Chargement non bloquant des ressources.

### Exemple avec Async/Await et Promise.all

```js
// Exemple d'utilisation de async/await pour une lisibilité et rapidité accrue
async function fetchDataAsync() {
    try {
        const weatherPromise = fetchWeatherDataAsync();
        const newsPromise = fetchNewsDataAsync();
        const stocksPromise = fetchStocksDataAsync();

        const [weatherData, newsData, stocksData] = await Promise.all([weatherPromise, newsPromise, stocksPromise]);
        
        console.log("Async : ", weatherData, newsData, stocksData);
    } catch (error) {
        console.error('Error fetching data:', error);
    }
}

```

* **async** : Déclare qu'une fonction retourne une promesse.
* **await** : Met en pause l'exécution de la fonction asynchrone jusqu'à ce que la promesse soit résolue.
* **Promise.all** : Méthode statique qui prend un tableau de promesses et s'exécute en parallèle.

::::::: {.notice .info}

### Pratique : Exercice de Masse

Créez 4 méthodes qui simulent la connexion à une BD : `fetchLongueur`, `fetchLargeur`, `fetchProfondeur`, `fetchMasseVolumique`. Chacune résoudra sa promesse en 1s et retournera respectivement 2, 1, 4, 5.

1. Faites un programme nommé `promise.js` qui calcule la masse sans utiliser de async/await.
2. Faites un programme nommé `async.js` qui calcule la masse avec async/await.
:::::::

::: {.expand .button title="👉 Afficher la Solution de l'Exercice"}

#### 1. Version synchrone imbriquée (Sans Async/Await)

```js
function calculMasse() {
    fetchLongueur().then(res => {
        let masse = res;
        fetchLargeur().then(resp => {
            masse *= resp;
            fetchProfondeur().then(response =>{
                masse *= response;
                fetchMasseVolumique().then(response2 => {
                    masse *= response2;
                    console.log("Sync : ", masse);
                })
            })
        })
    }).catch(err => console.error(err));
}

```

#### 2. Version optimisée (Avec Async/Await et Promise.all)

```js
async function calculMasseAsync() {
    try {
        const longueur = fetchAsyncLongueur();
        const largeur = fetchAsyncLargeur();
        const profondeur = fetchAsyncProfondeur();
        const masseVolumique = fetchAsyncMasseVolumique();

        const [lon, larg, prof, mv] = await Promise.all([longueur, largeur, profondeur, masseVolumique]);
        console.log("Async : ", lon * larg * prof * mv);
    } catch (error) {
        console.error('Error:', error);
    }
}

```

:::

---

## Requêtes

::::::: {.notice .warning}

### Exercice : Structure de la base de données

Créez la table suivante dans votre base de données :


*(Assurez-vous de placer l'image dans le dossier `static/images/` de votre projet Hugo)*
:::::::

::: {.expand .button title="👉 Afficher la Solution SQL"}

```sql
CREATE TABLE parc_attraction ( 
    id INT AUTO_INCREMENT PRIMARY KEY,
    nom VARCHAR(100), 
    ville VARCHAR(500), 
    cout_billet DECIMAL(6,2) 
);

-- Insertion de test
INSERT INTO parc_attraction (nom, ville, cout_billet) 
VALUES ('La Ronde','Montréal','55.99'),('Disney','Orlando','99.49');

```

:::

Dans `database.js`, nous pouvons maintenant créer des fonctions de requêtes propres :

```js
// Modèle de fonction asynchrone réutilisable
async function getParcAttraction() {
     const [rows] = await pool.query("SELECT * FROM parc_attraction;")
     return rows
}
const parc = await getParcAttraction()
console.log(parc)

```

