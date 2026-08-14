+++
date = '2026-06-26T10:15:09-04:00'
draft = true
title = 'Axios'
+++

### Étape 1 : Créez le fichier de contenu

Créez un fichier nommé `axios.md` (ou `_index.md` si c'est une section) dans votre dossier `content/` (par exemple : `content/node-crud/axios.md`).

### Étape 2 : Collez le code Markdown suivant

```markdown
---
title: "Intégration d'Axios"
linkTitle: "Axios"
weight: 20
description: "Apprendre à connecter une application mobile React Native à un backend Node.js avec Axios."
---

## Axios

L'objectif de cette section est de lier votre application mobile à votre base de données en passant par un serveur d'API Node.js/Express à l'aide de la librairie **Axios**.

---

### 1. Préparer la Base de Données

Créez une table nommée `user` qui contiendra les informations d'authentification de vos utilisateurs.

::: {.expand .button title="🗔 Afficher le script SQL de création"}
```sql
CREATE TABLE user (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(64),
    email VARCHAR(128),
    password VARCHAR(64)
);

```

:::

---

### 2. Préparer le Backend

Ajoutez d'abord une fonction de requête SQL dans votre fichier `database.js` pour valider l'authentification soit par le pseudonyme (*username*), soit par l'adresse courriel (*email*).

::: {.expand .button title="📄 Afficher la fonction dans database.js"}

```js
import mysql from 'mysql2';
import dotenv from 'dotenv';
dotenv.config();

const pool = mysql.createPool({
    host: process.env.MYSQL_HOST,
    user: process.env.MYSQL_USER,
    password: process.env.MYSQL_PASSWORD,
    database: process.env.MYSQL_DATABASE
}).promise();

export async function getUserByUsernameOrEmailAndPassword(usernameOrEmail, password) {
    // Vérification de l'existence du username ou du email
    const [potentialUsers] = await pool.query(
        `SELECT * FROM user WHERE username=? OR email=?;`, 
        [usernameOrEmail, usernameOrEmail]
    );
    
    let goodUser = false;
    
    potentialUsers.forEach((user) => {
        if (user.password === password) {
            goodUser = user;
        }
    });
    
    return goodUser;
}

```

:::

#### Création de l'Endpoint API

Créez un point de terminaison (endpoint) de type `POST` pour la route `/auth/signin` afin de recevoir et de traiter les informations transmises par l'application mobile.

```js
import express from 'express';
// Assurez-vous que l'importation de votre fonction SQL réussit                
import { getUserByUsernameOrEmailAndPassword } from './database.js';

const app = express();

// app.use(express.json()) est une instruction cruciale pour configurer Express afin de gérer 
// et d'interpréter automatiquement les données JSON envoyées via les requêtes HTTP.
app.use(express.json());

app.post("/auth/signin", async (req, res) => {
    const { usernameOrEmail, password } = req.body;
    try {
        const user = await getUserByUsernameOrEmailAndPassword(usernameOrEmail, password);
        if (!user) {
            return res.status(404).json({ error: "User not found" });
        }
        res.status(200).json(user);
    } catch (error) {
        console.error('Error retrieving user: ', error);
        res.status(500).json({ error: 'Internal server error' });
    }
});

// Middleware d'erreur centralisé
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).send('Something broke!');
});

// Lance le serveur sur le port 8080
app.listen(8080, () => {
    console.log('Server is running on port 8080');
});

```

---

### 3. Vérifier le tout avec Thunder Client

**Thunder Client** est une extension intégrée pour VSCode très utile pour tester son backend rapidement sans quitter l'éditeur (similaire à Postman).

Pour tester votre API :

1. Lancez votre serveur : dans le dossier `BACKEND`, exécutez la commande `npm run dev`.
2. Configurez la méthode HTTP à **POST**.
3. Saisissez l'URL correspondante (ex: `http://127.0.0.1:8080/auth/signin`). *Modifiez l'adresse IP si votre serveur ne tourne pas localement sur la même machine.*

Voici les étapes visuelles de la validation :

Ajoutez vos paramètres au format JSON dans le corps (*body*) de la requête. La requête doit renvoyer l'utilisateur, peu importe si vous fournissez l'adresse courriel ou le pseudonyme.

---

### 4. Intégration Client (Application Mobile React Native)

Créez un dossier nommé `lib/` à la racine de votre projet React Native, puis ajoutez-y un fichier nommé `axios.js`. Ce fichier centralise la configuration globale d'Axios.

```js
// lib/axios.js
import axios from 'axios';

// Remplacez l'adresse IP par celle de votre serveur backend sur le réseau local
const api = axios.create({
    baseURL: `http://192.168.0.31:8080`
});

export async function signIn(usernameOrEmail, password) {
    try {
        console.log(`trying to signIn with usernameOrEmail: ${usernameOrEmail}`);

        const signInData = {
            usernameOrEmail: usernameOrEmail,
            password: password
        };

        const userAuth = await api.post(`/auth/signin`, signInData, {
            headers: {
                Authorization: 'none',
            },
        });
        
        if (!(userAuth.status === 200)) throw new Error("Authentication failed");
        
        return userAuth.data;
    } catch (error) {
        throw new Error(error.message || error);
    }
}

```

---

### 5. Intégrer la requête dans l'interface de connexion (Sign In)

Voici comment lier la fonction d'appel API créée ci-dessus à votre composant de formulaire React Native.

::: {.expand .button title="📱 Afficher le composant complet React Native (signin.js)"}

```jsx
import { StyleSheet, Text, View, TextInput, Dimensions, TouchableOpacity } from 'react-native';
import React, { useState } from 'react';
import { useTheme } from '../../components/contexts/ThemeContext';
import { colorsPalette } from '../../assets/colorsPalette';
import { useRouter } from 'expo-router';
import { signIn } from '../../lib/axios';

const Signin = () => {
    const { theme } = useTheme();
    const colors = colorsPalette[theme];
    const router = useRouter();
    
    const [form, setForm] = useState({ username: "Harry Potter", password: "iloveGinny" });

    const submit = async () => {
        console.log("Trying to signIn");
        try {
            const result = await signIn(form.username, form.password);
            // Redirection dynamique vers le profil de l'utilisateur authentifié
            router.push(`../${result.username}/profile`);
        } catch (error) {
            console.error("Error : ", error.message);
        }
    };   

    return (
        <View style={styles.container}>
            <TextInput
                style={[styles.textInput, { color: colors.text, backgroundColor: colors.background }]}
                onChangeText={(item) => setForm({ ...form, username: item })}
                placeholder="Entrez votre nom d'utilisateur ou email"
                placeholderTextColor={colors.secondary}
                value={form.username}
            />
            
            <TextInput
                style={[styles.textInput, { color: colors.text, backgroundColor: colors.background }]}
                onChangeText={(item) => setForm({ ...form, password: item })}
                placeholder="Entrez votre mot de passe"
                placeholderTextColor={colors.secondary}
                secureTextEntry={true}
                value={form.password}
            />
            
            <TouchableOpacity style={styles.button} onPress={submit}>
                <Text style={{ color: colors.text }}>Sign-in</Text>
            </TouchableOpacity>
        </View>
    );
};

export default Signin;

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'space-evenly',
        alignItems: 'center',
    },
    title: {
        fontSize: 48,
        fontWeight: 'bold',
        letterSpacing: 2,
        textAlign: 'center',
        textTransform: 'uppercase',
    },
    textInput: {
        width: Dimensions.get('window').width - 56,
        height: 56,
        paddingHorizontal: 20,
        textAlign: 'center',
        fontSize: 16,
    },
    button: {
        padding: 15,
        borderRadius: 8,
        alignItems: 'center',
    }
});

```