+++
date = '2026-06-25T16:02:29-04:00'
draft = false
weight = 14
pre="14. "
title = 'Offline First'
+++

```jsx
import { SafeAreaView } from 'react-native-safe-area-context';
import ReanimatedSwipeable from 'react-native-gesture-handler/ReanimatedSwipeable';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
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

const panGesture = Gesture.Pan();

const Card = ({ data, supprimerTask }) => {
    return (
        <Animated.View
            entering={FadeIn.duration(300)}
            exiting={FadeOut.duration(300)}
            layout={LinearTransition.duration(200)}
        >
            <GestureDetector gesture={panGesture}>
                <ReanimatedSwipeable
                    simultaneousWithExternalGesture={panGesture}
                    friction={2}
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
            </GestureDetector>
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
```