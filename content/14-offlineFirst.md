+++
date = '2026-06-25T16:02:29-04:00'
draft = false
weight = 14
pre="14. "
title = 'Offline First'
+++

L'approche **Offline-First** (le hors-ligne d'abord) est un paradigme de conception logicielle qui consiste à concevoir une application en partant du principe que la connexion Internet est **instable, lente, voire absente**, plutôt que de la considérer comme un acquis permanent.

Contrairement aux applications traditionnelles dites "Online-First" (qui affichent une erreur "Pas de connexion" dès que le réseau tombe), une application Offline-First fonctionne de manière fluide et transparente, que l'utilisateur soit dans le métro, en zone blanche ou connecté à la fibre.

Voici les principes fondamentaux à connaître et les principaux enjeux de cette approche.

---

## 1. Les principes clés de l'Offline-First

### A. La persistance locale des données (Local-First)

L'application ne dépend pas d'un serveur distant pour stocker ou lire les données initiales. Elle utilise une base de données locale (ex: **SQLite** ).

* **Principe :** L'utilisateur lit et écrit toujours dans sa base locale. Le serveur distant n'est plus qu'une simple copie de sauvegarde ou un point de synchronisation.

### B. La réactivité instantanée (Optimistic UI)

Puisque les données sont lues et écrites localement, les interactions de l'utilisateur (cliquer sur un bouton, valider un formulaire, supprimer un élément) sont **immédiates**.

* **Principe :** L'interface utilisateur se met à jour instantanément sans attendre qu'un serveur lointain valide l'action, offrant une fluidité accrue.

### C. La synchronisation bidirectionnelle et intelligente

C'est le cœur technique du système. Lorsque la connexion est rétablie, l'application synchronise les modifications locales avec le serveur distant et récupère les nouveautés.

* **Principe :** La synchronisation doit être **transparente** pour l'utilisateur et gérer les cas de connectivité intermittente sans bloquer l'application.

### D. La gestion des conflits (Conflict Resolution)

Que se passe-t-il si un utilisateur modifie la même information sur son téléphone en mode hors-ligne et depuis son ordinateur au bureau ?

* **Principe :** L'application doit prévoir des stratégies pour fusionner les données (ex. : *Last-Write-Wins*, mais il en existe d'autres) pour éviter d'écraser des données par erreur.

---

## 2. Les enjeux majeurs de l'Offline-First

### 🎯 Enjeu 1 : L'expérience utilisateur (UX) ultime

* **L'enjeu :** Offrir une résilience totale.
* **Bénéfice :** Fini les pages blanches, les messages d'erreur frustrants "Impossible de se connecter au serveur" ou la perte de données si la connexion coupe au mauvais moment. L'application reste **toujours utilisable**, ce qui augmente considérablement l'engagement.

### 🏗️ Enjeu 2 : La complexité architecturale

* **L'enjeu :** C'est le revers de la médaille. Développer en Offline-First demande beaucoup plus d'efforts d'ingénierie qu'une architecture client-serveur classique.
* **Complexité :** Il faut gérer deux sources de vérité (le client et le serveur), mettre en place des files d'attente de requêtes (*sync queues*), écrire des algorithmes de résolution de conflits et concevoir une gestion fine du cache.

### 💾 Enjeu 3 : La gestion de l'espace de stockage et de la sécurité sur le client

* **L'enjeu :** Stocker des données sur l'appareil de l'utilisateur implique des contraintes techniques et légales.
* **Défis :**
* *Stockage :* Les navigateurs ou les téléphones ont des limites d'espace. Il faut savoir purger intelligemment les données anciennes ou volumineuses.
* *Sécurité/Confidentialité :* Si l'appareil est volé, les données stockées localement en clair peuvent être compromises. Le chiffrement local (ex. *SQLCipher*) devient souvent indispensable, notamment pour respecter le RGPD.

---

# Concurrence

En architecture **Offline-First**, la gestion de la concurrence et des actions destructrices (comme la suppression) est un sujet critique. Puisque plusieurs appareils peuvent modifier ou supprimer les mêmes données en même temps sans être connectés, on utilise des mécanismes de **verrouillage** et des stratégies de **suppression** très précis.

Voici comment s'articulent la **suppression optimiste / pessimiste** et les contrôles de concurrence associés dans ce contexte.

---

## 1. Concurrence Optimiste vs Pessimiste : Le choix de l'Offline-First

Avant de parler de suppression, il faut comprendre comment ces deux philosophies gèrent l'accès aux données :

* **Le Verrouillage Pessimiste (Pessimistic Locking) :**
* *Principe :* "Je bloque la ressource dès que je commence à la modifier pour empêcher quiconque d'autre d'y toucher."
* *En mobile :* **Incompatible avec l'Offline-First pur.** Si un utilisateur prend le métro et passe en mode hors-ligne avec un enregistrement "verrouillé", personne d'autre ne pourra y accéder tant qu'il ne sera pas ressorti du tunnel. Cela bloque complètement la collaboration.


* **Le Verrouillage Optimiste (Optimistic Locking) :**
* *Principe :* "Je laisse tout le monde modifier les données en même temps en local. Je vérifie juste au moment de la synchronisation si la donnée a changé entre-temps."
* *En mobile :* **C'est le standard de l'Offline-First.** On utilise généralement un horodatage (`updated_at`) afin de régler les conflits. 



---

## 2. Application à la suppression : Suppression Optimiste vs Pessimiste

Dans une application mobile hors-ligne, la suppression pose un piège redoutable : **le "problème du fantôme"** (supprimer une donnée en local alors qu'un autre utilisateur l'a modifiée sur le serveur, ou inversement).

### A. La Suppression Optimiste (Soft Delete + Sync asynchrone)

C'est l'approche reine en Offline-First mobile.

* **Comment ça fonctionne :**
1. L'utilisateur clique sur "Supprimer" sur son téléphone mobile en mode hors-ligne.
2. L'interface réagit **instantanément** : l'élément disparaît de l'écran (UX fluide).
3. En base locale, on ne détruit pas physiquement la ligne (pas de `DELETE SQL` brut), mais on applique un **Soft Delete** en marquant un drapeau, par exemple `is_deleted = true` ou `deleted_at = timestamp`.
4. L'action est ajoutée dans la file d'attente de synchronisation (*sync queue*).
5. Quand le réseau revient, le mobile envoie l'ordre au serveur : *"Supprime l'élément ID 42 (version 2)"*.


* **Avantages :** Zéro latence pour l'utilisateur, fonctionne parfaitement hors-ligne.
* **Le piège (Conflit) :** Que se passe-t-il si un autre utilisateur a modifié cet élément sur le web pendant que notre mobile était hors-ligne ?
* *Règle générale :* La suppression lamine souvent la modification (*Last-Delete-Wins*), ou bien le serveur refuse la suppression et renvoie un conflit si la règle métier l'interdit (ex: "Impossible de supprimer un document qui a été validé par un manager entre-temps").


---

### B. La Suppression Pessimiste (Verrou de suppression)

Plus rare et délicate sur mobile, elle s'applique à des cas métiers très stricts (ex: applications bancaires, validation de transactions, dossiers médicaux partagés).

* **Comment ça fonctionne :**
* Avant d'autoriser la suppression, l'application mobile doit s'assurer (si elle a du réseau) qu'aucun autre processus ne verrouille l'objet.
* Si l'appareil est **hors-ligne**, l'application **refuse purement et simplement** d'initier la suppression si elle touche à des données critiques partagées, ou elle place la demande dans un état "En attente de vérification stricte" qui ne sera exécuté qu'une fois la connexion retrouvée et le verrou validé par le serveur.


* **Avantages :** Évite les erreurs de suppression catastrophiques sur des données partagées.
* **Inconvénients :** Brise l'expérience Offline-First ("Impossible de supprimer cet élément car vous êtes hors-ligne").

---

Voici plusieurs propositions de reformulation, de la plus académique à la plus directe, corrigées et prêtes à intégrer dans vos notes de cours :

### Option 1 : Style "Notes de cours" (Recommandé)

Maintenant que les concepts clés sont posés, passons à la mise en pratique à travers un cas concret : une application de type *To-Do List*.
Nous développerons une architecture *offline-first* dotée d'une interface réactive (*Optimistic UI*). Pour gérer les conflits de manière simple et locale, la résolution s'effectuera directement côté client selon l'ordre de priorité suivant :
1. **`Last-delete-wins`** (la suppression l'emporte)
2. **`Last-write-wins`** (la modification la plus récente l'emporte)


---



<!-- 
```jsx
import { SafeAreaView } from 'react-native-safe-area-context';
import ReanimatedSwipeable from 'react-native-gesture-handler/ReanimatedSwipeable';
import Animated, { FadeIn, FadeOut, LinearTransition } from 'react-native-reanimated';
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
} from 'react-native';
import * as SQLite from 'expo-sqlite';
import axios from 'axios';

const API_URL = `http://${process.env.EXPO_PUBLIC_BACKEND_IP}:3000/tasks`;

const COLOR_1 = "#000000";
const COLOR_2 = "#282A3A";
const COLOR_3 = '#735F32';
const COLOR_4 = '#C69749';

const Card = ({ data, supprimerTask }) => {
    const renderRightActions = (progress, dragX) => {
        return (
            <View>
                <Text>Supprimer</Text>
            </View>
        );
    };
    return (
        <Animated.View
            entering={FadeIn.duration(300)}
            exiting={FadeOut.duration(300)}
            layout={LinearTransition.duration(200)}
        >
                <ReanimatedSwipeable
                    friction={2}
                    leftThreshold={40}
                    renderRightActions={renderRightActions}
                    onSwipeableWillOpen={(direction) => {
                        if (direction === "left") {
                            supprimerTask(data.id);
                        }
                    }}
                >
                    <View style={styles.cardContainer}>
                        <Text style={{ color: COLOR_4 }}>
                            {data.title} {data.synced === 0 && " (En attente...)"}
                        </Text>
                    </View>
                </ReanimatedSwipeable>
        </Animated.View>
    );
};

const Index = () => {
    const [tasks, setTasks] = useState([]);
    const [curInput, setCurInput] = useState("");
    const [loading, setLoading] = useState(true);
    const [db, setDb] = useState(null);

    useEffect(() => {
        async function setupDatabase() {
            try {
                const database = await SQLite.openDatabaseAsync('tasks.db');
                setDb(database);

                // Ajout de la colonne is_deleted
                await database.execAsync(`
                    CREATE TABLE IF NOT EXISTS local_tasks (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        server_id INTEGER,
                        title TEXT NOT NULL,
                        completed INTEGER DEFAULT 0,
                        synced INTEGER DEFAULT 0,
                        is_deleted INTEGER DEFAULT 0
                    );
                    `);

                // Charger uniquement les tâches non supprimées
                const allTasks = await database.getAllAsync('SELECT * FROM local_tasks WHERE is_deleted = 0');
                setTasks(allTasks);
                setLoading(false);

                syncWithServer(database);
                // 2. SYNCHRO PÉRIODIQUE : Vérifier la base toutes les 30 secondes
                const intervalId = setInterval(() => {
                    console.log("Vérification périodique de la synchronisation...");
                    syncWithServer(database);
                }, 30000); // 30000 ms = 30 secondes
                // Nettoyage de l'intervalle si le composant est démonté
                return () => clearInterval(intervalId);
            } catch (error) {
                console.error("Erreur init SQLite:", error);
                setLoading(false);
            }
            
        }
        setupDatabase();
    }, []);

    const syncWithServer = async (database) => {
        if (!database) return;
        try {
            let hasChanges = false;

            // 1. Envoyer les suppressions en attente au serveur
            const pendingDeletions = await database.getAllAsync('SELECT * FROM local_tasks WHERE is_deleted = 1 AND server_id IS NOT NULL');
            for (let task of pendingDeletions) {
                try {
                    await axios.delete(`${API_URL}/${task.server_id}`);
                    // Une fois supprimée du serveur, on la nettoie définitivement de SQLite
                    await database.runAsync('DELETE FROM local_tasks WHERE id = ?', [task.id]);
                    hasChanges = true;
                } catch (err) {
                    // Serveur injoignable, on laisse is_deleted = 1 pour la prochaine fois
                }
            }

            // 2. Envoyer les créations en attente (synced = 0 et non supprimées)
            const unsyncedTasks = await database.getAllAsync('SELECT * FROM local_tasks WHERE synced = 0 AND is_deleted = 0');
            for (let task of unsyncedTasks) {
                try {
                    const response = await axios.post(API_URL, {
                        title: task.title,
                        completed: task.completed
                    });

                    await database.runAsync(
                        'UPDATE local_tasks SET server_id = ?, synced = 1 WHERE id = ?',
                        [response.data.id, task.id]
                    );
                    hasChanges = true;
                } catch (err) {
                    // Hors ligne
                }
            }

            // 3. Récupérer les données du serveur
            const response = await axios.get(API_URL);
            const serverTasks = response.data;

            for (let sTask of serverTasks) {
                const existing = await database.getFirstAsync('SELECT * FROM local_tasks WHERE server_id = ?', [sTask.id]);
                if (!existing) {
                    await database.runAsync(
                        'INSERT INTO local_tasks (server_id, title, completed, synced, is_deleted) VALUES (?, ?, ?, 1, 0)',
                        [sTask.id, sTask.title, sTask.completed]
                    );
                    hasChanges = true;
                }
            }

            if (hasChanges) {
                const updatedTasks = await database.getAllAsync('SELECT * FROM local_tasks WHERE is_deleted = 0');
                setTasks(updatedTasks);
            }
        } catch (error) {
            console.log("Mode hors-ligne actif (synchronisation ignorée)");
        }
    };

    const ajouterTask = async () => {
        if (curInput.trim() == "") {
            Alert.alert("Attention", "Veuillez remplir tous les champs.");
            return;
        }

        try {
            const result = await db.runAsync(
                'INSERT INTO local_tasks (title, completed, synced, is_deleted) VALUES (?, ?, 0, 0)',
                [curInput, 0]
            );

            const newTask = {
                id: result.lastInsertRowId,
                title: curInput,
                completed: 0,
                synced: 0,
                is_deleted: 0
            };

            setTasks((prevTasks) => [...prevTasks, newTask]);
            setCurInput("");
            syncWithServer(db);
        } catch (error) {
            Alert.alert("Erreur", "Erreur lors de l'ajout local.");
        }
    };

    const supprimerTask = async (id) => {
        const taskToDelete = tasks.find(t => t.id === id);

        // Suppression optimiste immédiate de l'UI
        setTasks((prevTasks) => prevTasks.filter((task) => task.id !== id));

        try {
            if (!db) return;

            if (!taskToDelete.server_id) {
                // Si elle n'a jamais été envoyée au serveur, on la supprime carrément de SQLite
                await db.runAsync('DELETE FROM local_tasks WHERE id = ?', [id]);
            } else {
                // Si elle existe sur le serveur, on la marque comme "à supprimer" (Soft Delete local)
                await db.runAsync('UPDATE local_tasks SET is_deleted = 1 WHERE id = ?', [id]);
                // Et on tente de supprimer immédiatement sur le serveur
                await axios.delete(`${API_URL}/${taskToDelete.server_id}`);
                // Si ça réussit, on nettoie définitivement de SQLite
                await db.runAsync('DELETE FROM local_tasks WHERE id = ?', [id]);
            }
        } catch (error) {
            console.log("Hors-ligne : la suppression sur le serveur est mise en attente.");
        }
    };

    return (
        <SafeAreaView style={styles.container}>
            {loading ? (
                <View style={styles.center}>
                    <ActivityIndicator size="large" color="#007AFF" />
                </View>
            ) : (
                <>
                    <View style={styles.inputsContainer}>
                        <TextInput
                            style={styles.input}
                            placeholder='Entrez une tâche à accomplir'
                            placeholderTextColor={COLOR_3}
                            value={curInput}
                            onChangeText={setCurInput}
                        />
                        <TouchableOpacity onPress={ajouterTask} style={styles.addBtnContainer}>
                            <Text style={styles.addBtnText}>+</Text>
                        </TouchableOpacity>
                    </View>
                    <FlatList
                        data={tasks}
                        renderItem={({ item }) => <Card data={item} supprimerTask={supprimerTask} />}
                        keyExtractor={(item) => item.id.toString()}
                        style={{ flex: 1, marginVertical: 20 }}
                    />
                </>
            )}
        </SafeAreaView>
    );
};

export default Index;

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: COLOR_1,
        padding: 20
    },
    center: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center'
    },
    inputsContainer: {
        flexDirection: "row",
        marginTop: 10
    },
    input: {
        backgroundColor: COLOR_2,
        flex: 1,
        textAlign: "center",
        color: COLOR_4,
        paddingHorizontal: 10,
        height: 50
    },
    addBtnContainer: {
        justifyContent: "center",
        alignItems: "center",
        backgroundColor: COLOR_4,
    },
    addBtnText: {
        fontSize: 30,
        fontWeight: "600",
        paddingVertical: 5,
        paddingHorizontal: 20
    },
    cardContainer: {
        backgroundColor: COLOR_2,
        marginVertical: 8,
        padding: 20,
    }
});
``` -->

Voici un exercice guidé pour transformer un composant de To-Do List basique (en mémoire volatile) en une application **Offline-First** complète avec SQLite, UI optimiste et synchronisation en arrière-plan.

---

### 🚀 Le Code de Base (Point de départ)

Voici le squelette de l'application. Pour l'instant, les tâches sont stockées uniquement dans un état React (`useState`) en mémoire : si vous fermez l'app, tout est perdu, et il n'y a aucune communication avec un serveur.

{{% expand title="BE" %}}
```jsx
import express from 'express';
import dotenv from 'dotenv';
import pool from './database.js';

dotenv.config();

const app = express();
const port = process.env.PORT || 3000;

app.use(express.json());

// Fonction pour initialiser la base de données (Création de la table si elle n'existe pas)
async function initializeDatabase() {
    try {
        const query = `
            CREATE TABLE IF NOT EXISTS tasks (
                id INT AUTO_INCREMENT PRIMARY KEY,
                title VARCHAR(255) NOT NULL,
                updated_at DATETIME NOT NULL
            );
        `;
        await pool.query(query);
        console.log("✅ Table 'tasks' vérifiée / créée avec succès.");
    } catch (error) {
        console.error("❌ Erreur lors de la création de la table :", error);
    }
}

app.post('/tasks', async (req, res) => {
    try {
        const { title, updated_at } = req.body;
        
        // Convertit la date au format string compatible MySQL
        const dateToSave = updated_at 
            ? new Date(updated_at).toISOString().slice(0, 19).replace('T', ' ')
            : new Date().toISOString().slice(0, 19).replace('T', ' ');

        const [result] = await pool.query(
            "INSERT INTO tasks (title, updated_at) VALUES (?, ?)",
            [title, dateToSave]
        );
        
        res.status(201).json({ id: result.insertId, title, updated_at: dateToSave });
    } catch (error) {
        console.error(error);
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// READ (Tous - avec filtrage intelligent via lastSync optionnel)
app.get('/tasks', async (req, res) => {
    try {
        const { lastSync } = req.query;
        let query = "SELECT * FROM tasks";
        let params = [];

        // Si le client fournit une date de dernière synchro, on ne renvoie que ce qui a changé depuis
        if (lastSync) {
            query += " WHERE updated_at >= ?";
            params.push(lastSync);
        }

        const [tasks] = await pool.query(query, params);
        res.json(tasks);
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// READ (Un seul)
app.get('/tasks/:id', async (req, res) => {
    try {
        const [tasks] = await pool.query("SELECT * FROM tasks WHERE id = ?", [req.params.id]);
        if (tasks.length === 0) return res.status(404).json({ erreur: "Non trouvé" });
        res.json(tasks[0]);
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// UPDATE
app.put('/tasks/:id', async (req, res) => {
    try {
        const { title, updated_at } = req.body;
        const [result] = await pool.query(
            "UPDATE tasks SET title = ?, updated_at = ? WHERE id = ?",
            [title, updated_at || new Date(), req.params.id]
        );
        if (result.affectedRows === 0) return res.status(404).json({ erreur: "Non trouvé" });
        res.json({ message: "Mis à jour avec succès !" });
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// DELETE
app.delete('/tasks/:id', async (req, res) => {
    try {
        const [result] = await pool.query("DELETE FROM tasks WHERE id = ?", [req.params.id]);
        if (result.affectedRows === 0) return res.status(404).json({ erreur: "Non trouvé" });
        res.json({ message: "Supprimé avec succès !" });
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// Middleware d'erreur centralisé
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).send('Something broke!');
});

// Lancement de l'initialisation de la DB puis démarrage du serveur
initializeDatabase().then(() => {
    app.listen(port, () => {
        console.log(`🚀 Serveur démarré sur http://localhost:${port}`);
    });
});
```
{{% /expand %}}
{{% expand title="FE" %}}
```jsx
import { SafeAreaView } from 'react-native-safe-area-context';
import ReanimatedSwipeable from 'react-native-gesture-handler/ReanimatedSwipeable';
import Animated, { FadeIn, FadeOut, LinearTransition } from 'react-native-reanimated';
import React, { useState } from 'react';
import {
    StyleSheet,
    Text,
    View,
    FlatList,
    TextInput,
    TouchableOpacity,
} from 'react-native';

const COLOR_1 = "#000000";
const COLOR_2 = "#282A3A";
const COLOR_3 = '#735F32';
const COLOR_4 = '#C69749';

const Card = ({ data, supprimerTask }) => {
    const renderRightActions = (progress, dragX) => {
        return (
            <View>
                <Text>Supprimer</Text>
            </View>
        );
    };
    return (
        <Animated.View
            entering={FadeIn.duration(300)}
            exiting={FadeOut.duration(300)}
            layout={LinearTransition.duration(200)}
        >
                <ReanimatedSwipeable
                    friction={2}
                    leftThreshold={40}
                    renderRightActions={renderRightActions}
                    onSwipeableWillOpen={(direction) => {
                        if (direction === "left") {
                            supprimerTask(data.id);
                        }
                    }}
                >
                    <View style={styles.cardContainer}>
                        <Text style={{ color: COLOR_4 }}>
                            {data.title} {data.synced === 0 && " (En attente...)"}
                        </Text>
                    </View>
                </ReanimatedSwipeable>
        </Animated.View>
    );
};
export default function Index() {
    const [tasks, setTasks] = useState([]);
    const [curInput, setCurInput] = useState("");

    const ajouterTask = () => {
        if (curInput.trim() === "") return;
        setTasks([...tasks, { id: Date.now().toString(), title: curInput, completed: 0 }]);
        setCurInput("");
    };

    const supprimerTask = (id) => {
        setTasks(tasks.filter(t => t.id !== id));
    };

    return (
        <SafeAreaView style={styles.container}>
            <View style={styles.inputsContainer}>
                <TextInput
                    style={styles.input}
                    placeholder='Entrez une tâche à accomplir'
                    placeholderTextColor={COLOR_3}
                    value={curInput}
                    onChangeText={setCurInput}
                />
                <TouchableOpacity onPress={ajouterTask} style={styles.addBtnContainer}>
                    <Text style={styles.addBtnText}>+</Text>
                </TouchableOpacity>
            </View>
            <FlatList
                data={tasks}
                renderItem={({ item }) => <Card data={item} supprimerTask={supprimerTask} />}
                keyExtractor={(item) => item.id.toString()}
                style={{ flex: 1, marginVertical: 20 }}
            />
        </SafeAreaView>
    );
}

const styles = StyleSheet.create({
    container: { flex: 1, backgroundColor: COLOR_1, padding: 20 },
    inputsContainer: { flexDirection: "row", marginTop: 10 },
    input: { backgroundColor: COLOR_2, flex: 1, textAlign: "center", color: COLOR_4, paddingHorizontal: 10, height: 50 },
    addBtnContainer: { justifyContent: "center", alignItems: "center", backgroundColor: COLOR_4 },
    addBtnText: { fontSize: 30, fontWeight: "600", paddingVertical: 5, paddingHorizontal: 20 },
    cardContainer: { backgroundColor: COLOR_2, marginVertical: 8, padding: 20 }
});

```
{{% /expand %}}
---

### Étape 1 : Ajouter la persistance locale (`SQLite`) et la théorie du *Local-First*

> 💡 **Théorie :** Dans une architecture *Offline-First*, le serveur n'est plus la source principale de vérité. L'application lit et écrit d'abord dans une base de données embarquée sur l'appareil (ici SQLite avec `expo-sqlite`).

#### Objectif de l'étape :

1. Importer `expo-sqlite`.
2. Initialiser la base de données au chargement du composant (`useEffect`).
3. Créer une table `local_tasks` avec des champs spécifiques à l'offline :
* `server_id` : Pour faire le lien avec l'ID distant du serveur.
* `synced` : Un booléen (0 ou 1) pour savoir si la tâche a été envoyée au serveur.
* `is_deleted` : Un drapeau pour le **Soft Delete** (suppression logique en cas de hors-ligne).



```javascript
import * as SQLite from 'expo-sqlite';

// ... à l'intérieur du composant Index :
const [db, setDb] = useState(null);

useEffect(() => {
    async function setupDatabase() {
        const database = await SQLite.openDatabaseAsync('tasks.db');
        setDb(database);
        
        await database.execAsync(`
            CREATE TABLE IF NOT EXISTS local_tasks (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                server_id INTEGER,
                title TEXT NOT NULL,
                updated_at TEXT NOT NULL,
                synced INTEGER DEFAULT 0,
                is_deleted INTEGER DEFAULT 0
            );
        `);

        const allTasks = await database.getAllAsync('SELECT * FROM local_tasks WHERE is_deleted = 0');
        setTasks(allTasks);
    }
    setupDatabase();
}, []);

```

{{% notice style="info" title="Solution" %}}
Simplement tester, ça marche? Si oui, passe à la prochaine question.
{{% /notice %}}

---

### Étape 2 : Implémenter l'UI Optimiste (`Optimistic UI`)

> 💡 **Théorie :** L'UI optimiste consiste à mettre à jour l'interface utilisateur instantanément, sans attendre la réponse du réseau. Si l'utilisateur est hors-ligne, l'action est enregistrée localement avec un indicateur (ex: *"En attente..."*).

#### Objectif de l'étape :

Modifier les fonction `ajouterTask` et `supprimerTask` pour intégrer la bd locale. 

{{% expand title="Solution" %}}
```jsx
    const ajouterTask = async () => {
        if (curInput.trim() === "") return;
        try {
            const resp = await db.runAsync(
                        'INSERT INTO local_tasks (server_id, title, updated_at) VALUES (?, ?, ?);',
                        [null, curInput, Date.now()]
                    );
            const newTask = {
                id: resp.lastInsertRowId,
                title: curInput,
                updated_at: updatedAt,
                synced: 0,
                is_deleted: 0
            };

            setTasks([...tasks, newTask]);
            setCurInput("");

        }
        catch {
            console.log("Something went wrong in ajouterTask")
        }
    };

    const supprimerTask = async (id) => {
        try {
            await db.runAsync(`DELETE FROM local_tasks WHERE id=?`, [id])
            setTasks(tasks.filter(t => t.id !== id));
        } catch {
            Alert("Local Error")
        }
    };
```
{{% /expand %}}

---

### Étape 2.5 : Soft Delete local

Règle : effectuez toujours un soft delete des tâches en local préalablement à la synchronisation.

{{% expand title="Solution" %}}
```jsx
    const supprimerTask = async (id) => {
        try {
            await db.runAsync(`UPDATE local_tasks SET is_deleted = 1 WHERE id=?`, [id])
            setTasks(tasks.filter(t => t.id !== id));
        } catch {
            Alert("Local Error")
        }
    };
```
{{% /expand %}} 
---

### Étape 3 : Mettre en place la synchronisation en arrière-plan (`Push / Pull`)

> 💡 **Théorie :** Dès que le réseau est disponible (ou de manière périodique via un `setInterval`), l'application déclenche une synchronisation :
> 1. **Push :** Elle envoie les créations et suppressions en attente (`synced = 0` ou `is_deleted = 1`) vers le serveur.
> 2. **Pull :** Elle va chercher les nouveautés du serveur pour enrichir la base locale.
> 
> 

#### Objectif de l'étape :

Intégrer la fonction `syncWithServer` à l'aide d'`axios` et planifier une vérification périodique toutes les 30 secondes pour garantir la résilience de l'application.

---


### Étape 4 : Implémenter le soft Delete (`Soft Delete`)

 💡 **Rappel Théorique :**
 Dans une architecture *Offline-First*, dès que le réseau revient (ou de façon périodique), l'application synchronise ses données :
 1. **Le Push :** Elle envoie les créations et les suppressions en attente (`synced = 0` ou `is_deleted = 1`) au serveur.
 2. **Le Pull :** Elle récupère les nouveautés du serveur pour mettre à jour la base locale.
 3. **Le Soft Delete :** En cas de suppression hors-ligne, on marque la ligne avec `is_deleted = 1` pour que le serveur sache qu'il doit lui aussi la supprimer lors de la reconnexion.
 
---

### Étape 5 : Optimiser le Pull avec un horodatage (`updated_at`)

> 💡 **Théorie :**
> Dans une application standard, télécharger *toutes* les données du serveur à chaque synchronisation finit par saturer la bande passante et ralentir l'application.
> La **synchronisation incrémentale** consiste à demander au serveur uniquement ce qui a changé depuis la dernière synchronisation réussie, grâce à un champ `updated_at`.

#### 1. Mettre à jour la structure de la table SQLite

Il faut d'abord ajouter un champ `updated_at` dans notre table locale pour suivre les modifications :

```javascript
await database.execAsync(`
    CREATE TABLE IF NOT EXISTS local_tasks (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        server_id INTEGER,
        title TEXT NOT NULL,
        synced INTEGER DEFAULT 0,
        is_deleted INTEGER DEFAULT 0,
        updated_at TEXT
    );
`);

```

#### 2. Adapter la fonction de Pull avec un paramètre de date

Au lieu de faire un `axios.get(API_URL)` global, on passe la date de la dernière mise à jour connue (ou on stocke la dernière date de synco dans une table dédiée ou une variable) :

```javascript
// Exemple de Pull optimisé dans syncWithServer :
// Supposons qu'on récupère la dernière date de synco (ex: stockée en local)
const lastSyncTime = "2026-01-01T00:00:00.000Z"; 

// On interroge l'API avec un filtre
const response = await axios.get(`${API_URL}?updated_after=${lastSyncTime}`);
const serverTasks = response.data; // Uniquement les tâches modifiées !

for (let sTask of serverTasks) {
    // Si la tâche existe déjà (via server_id), on la met à jour (Upsert)
    // Sinon, on l'insère
    await database.runAsync(`
        INSERT INTO local_tasks (server_id, title, completed, synced, is_deleted, updated_at) 
        VALUES (?, ?, ?, 1, 0, ?)
        ON CONFLICT(server_id) DO UPDATE SET 
            title = excluded.title,
            completed = excluded.completed,
            updated_at = excluded.updated_at
    `, [sTask.id, sTask.title, sTask.completed, sTask.updated_at]);
    
    hasChanges = true;
}

```

---

