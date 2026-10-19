
+++
date = '2026-06-25T16:02:29-04:00'
draft = false
weight = 9
pre="9. "
title = 'API & BD'
+++


Dans ce cours, nous allons voir comment capturer une photo à l'aide de l'appareil photo, la stocker de manière permanente dans le système de fichiers de l'appareil, et enregistrer ses références dans une base de données relationnelle locale (`SQLite`). 

Cette architecture est idéale pour les applications mobiles fonctionnant en mode *offline-first* avec Expo (compatible **Expo Go**).

---

## 1. Pourquoi séparer le fichier et la base de données ?

Lorsqu'on développe une application mobile, une erreur fréquente consiste à vouloir stocker l'image directement dans la base de données (sous forme de texte brut ou de BLOB). 

* **Le problème :** Les bases de données SQLite ne sont pas optimisées pour stocker de gros volumes binaires. Cela alourdit le fichier de la base, provoque des lenteurs et peut saturer la mémoire de l'appareil.
* **La solution :** On stocke le fichier image brut dans le dossier sécurisé du téléphone via **`expo-file-system`**, et on enregistre uniquement le chemin d'accès (URI) ainsi que des métadonnées (titre, date) dans **`expo-sqlite`**.

---

## 2. Installation des dépendances

Ouvrez votre terminal à la racine de votre projet Expo et installez les trois packages nécessaires :

```bash
npx expo install expo-sqlite expo-file-system expo-image-picker

```

* **`expo-image-picker`** : Permet d'ouvrir la caméra ou la galerie pour sélectionner ou prendre une photo.
* **`expo-file-system`** : Permet de manipuler le système de fichiers pour copier l'image dans un répertoire persistant.
* **`expo-sqlite`** : Fournit une base de données SQL locale performante.

---

## 3. Implémentation pas à pas

Voici un exemple de composant React complet qui initialise la base de données, prend une photo, la sauvegarde localement et l'enregistre en base.

```jsx
import React, { useState, useEffect } from 'react';
import { StyleSheet, Text, View, Button, Image, FlatList } from 'react-native';
import * as ImagePicker from 'expo-image-picker';
import { File, Paths } from 'expo-file-system';
import * as SQLite from 'expo-sqlite';

export default function App() {
    const [db, setDb] = useState(null);
    const [images, setImages] = useState([]);

    // 1. Initialisation de la base de données au chargement
    useEffect(() => {
        async function setupDatabase() {
            const database = await SQLite.openDatabaseAsync('gallery.db');
            setDb(database);

            await database.execAsync(`
                CREATE TABLE IF NOT EXISTS photos (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    uri TEXT NOT NULL,
                    title TEXT
                );
            `);
            loadPhotos(database);
        }
        setupDatabase();
    }, []);

    // Charger les photos depuis SQLite
    const loadPhotos = async (database) => {
        const result = await database.getAllAsync('SELECT * FROM photos;');
        setImages(result);
    };

    // Retirer les photos de la mémoire puis de la bd
    const deletePhoto = async (id) => {
        if (!db) return;

        const row = await db.getFirstAsync('SELECT uri FROM photos WHERE id = ?;', [id]);

        if (row && row.uri) {
            try {
                // 2. Retirer la photo
                const file = new File(row.uri);
                if (file.exists) {
                    file.delete();
                }
            } catch (error) {
                console.log("Error deleting file:", error);
            }
        }

        // 3. Retirer l'entrée de la bd
        await db.runAsync('DELETE FROM photos WHERE id = ?;', [id]);

        // 4. Rafraichir les données
        loadPhotos(db);
    };
    // 2. Prendre une photo et la stocker avec l'API FileSystem
    const takeAndSavePhoto = async () => {

        // Ne pas oublier de demander les permissions
        const permissionResult = await ImagePicker.requestCameraPermissionsAsync();
        if (!permissionResult.granted) {
            alert("Permission d'accès à la caméra requise !");
            return;
        }

        // Prendre la photo
        const result = await ImagePicker.launchCameraAsync({
            mediaTypes: ['images'],
            quality: 0.7,
        });


        // Enregistrement
        if (!result.canceled) {
            const tempUri = result.assets[0].uri;
            const fileName = tempUri.split('/').pop();

            // Créer une instance de fichier source et une instance de destination
            const sourceFile = new File(tempUri);
            const destinationFile = new File(Paths.document, fileName);

            // Déplacer le fichier en utilisant la nouvelle méthode orientée objet
            sourceFile.move(destinationFile);
            const permanentUri = destinationFile.uri;

            // Enregistrer l'URI dans SQLite
            if (db) {
                await db.runAsync(
                    'INSERT INTO photos (uri, title) VALUES (?, ?);',
                    [permanentUri, 'Ma super photo']
                );
                loadPhotos(db);
            }
        }
    };

    return (
        <View style={styles.container}>
            <Button onPress={takeAndSavePhoto} title="Prendre une photo" />

            <Text style={styles.title}>Galerie locale ({images.length})</Text>

            <FlatList
                data={images}
                keyExtractor={(item) => item.id.toString()}
                renderItem={({ item }) => (
                    <View style={styles.item}>
                        <Image source={{ uri: item.uri }} style={styles.image} />
                        <Text>{item.title}</Text>
                        <Button title="Supprimer" color="red" onPress={() => deletePhoto(item.id)} />
                    </View>
                )}
            />
        </View>
    );
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 40,
        backgroundColor: '#fff',
        marginTop: 30
    },

    title: {
        fontSize: 18,
        fontWeight: 'bold',
        marginVertical: 15
    },

    item: {
        marginBottom: 20,
        alignItems: 'center'
    },

    image: {
        width: 200,
        height: 200,
        borderRadius: 10,
        marginBottom: 5
    }
});
```

---

## 4. Points clés à retenir

1. **Le cycle de vie du fichier :** L'image renvoyée par `ImagePicker` se trouve souvent dans un dossier temporaire du système (`Cache`). Si vous ne la déplacez pas vers `FileSystem.documentDirectory`, elle risque d'être supprimée par l'OS.
2. **Compatibilité Expo Go :** Tout le code présenté ci-dessus s'exécute directement dans l'application **Expo Go** sur Android et iOS, sans nécessiter de compilation native personnalisée (`expo run:android`).
3. **Bonnes pratiques :** Pensez à gérer la suppression des fichiers physiques sur le disque si l'utilisateur supprime une entrée de la base de données SQLite, afin de ne pas saturer l'espace de stockage du téléphone.

{{% notice style="exo" %}}
# 1 - Modal

Implémenter une fenêtre modale qui s'ouvre lors de la tentative d'enregistrement d'une image afin d'interrompre l'action immédiate et de proposer les options suivantes :

1. **Saisir un nom** pour l'image.
2. **Valider** l'enregistrement avec le nom choisi.
3. **Annuler** complètement l'opération.

> **Référence visuelle :** Se fier à la maquette / capture d'écran fournie pour le design et l'ergonomie de la modale.

Si le titre est laisser vide, ajouter vous même sans titre.

![alt text](/420512/images/modal.png)

# 2 - 2 image de large

Mettre à jour les paramètres de la FlatList pour configurer un affichage sur 2 colonnes d'images de large.

# 3 - Theme

Assurer la compatibilité complète de l'interface avec le changement dynamique de thèmes (mode clair / mode sombre).

{{% /notice %}}


<!-- import React, { useState, useEffect } from 'react';
import { StyleSheet, Text, View, Button, Image, FlatList, TextInput, Modal } from 'react-native';
import * as ImagePicker from 'expo-image-picker';
import { File, Paths } from 'expo-file-system';
import * as SQLite from 'expo-sqlite';
const Photo = () => {
    const [db, setDb] = useState(null);
    const [images, setImages] = useState([]);

    // États pour gérer l'affichage de la modale de saisie du titre
    const [modalVisible, setModalVisible] = useState(false);
    const [tempImageUri, setTempImageUri] = useState(null);
    const [photoTitle, setPhotoTitle] = useState('');

    // 1. Initialisation de la base de données au chargement
    useEffect(() => {
        async function setupDatabase() {
            const database = await SQLite.openDatabaseAsync('gallery.db');
            setDb(database);

            await database.execAsync(`
                CREATE TABLE IF NOT EXISTS photos (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    uri TEXT NOT NULL,
                    title TEXT
                    );
                    `);
            loadPhotos(database);
        }
        setupDatabase();
    }, []);

    // Charger les photos depuis SQLite
    const loadPhotos = async (database) => {
        const result = await database.getAllAsync('SELECT * FROM photos;');
        setImages(result);
    };

    // Retirer les photos de la mémoire puis de la bd
    const deletePhoto = async (id) => {
        if (!db) return;

        const row = await db.getFirstAsync('SELECT uri FROM photos WHERE id = ?;', [id]);

        if (row && row.uri) {
            try {
                const file = new File(row.uri);
                if (file.exists) {
                    file.delete();
                }
            } catch (error) {
                console.log("Error deleting file:", error);
            }
        }

        await db.runAsync('DELETE FROM photos WHERE id = ?;', [id]);
        loadPhotos(db);
    };

    // Étape 1 : Prendre la photo et ouvrir la modale au lieu d'enregistrer direct
    const takeAndSavePhoto = async () => {
        const permissionResult = await ImagePicker.requestCameraPermissionsAsync();
        if (!permissionResult.granted) {
            alert("Permission d'accès à la caméra requise !");
            return;
        }

        const result = await ImagePicker.launchCameraAsync({
            mediaTypes: ['images'],
            quality: 0.7,
        });

        if (!result.canceled) {
            setTempImageUri(result.assets[0].uri);
            setPhotoTitle(''); // Réinitialiser le champ de texte
            setModalVisible(true); // Ouvrir la modale pour demander le titre
        }
    };

    // Étape 2 : Sauvegarder définitivement l'image et le titre dans SQLite
    const savePhotoWithTitle = async () => {
        if (!tempImageUri) return;

        const fileName = tempImageUri.split('/').pop();
        const sourceFile = new File(tempImageUri);
        const destinationFile = new File(Paths.document, fileName);

        // Copier le fichier dans le dossier permanent
        sourceFile.move(destinationFile);
        const permanentUri = destinationFile.uri;

        // Enregistrer l'URI et le titre saisi dans SQLite
        if (db) {
            await db.runAsync(
                'INSERT INTO photos (uri, title) VALUES (?, ?);',
                [permanentUri, photoTitle.trim() || 'Sans titre']
            );
            loadPhotos(db);
        }

        // Fermer la modale et nettoyer
        setModalVisible(false);
        setTempImageUri(null);
        setPhotoTitle('');
    };

    // Function to handle cancellation and clean up the temp file
    const cancelPhoto = async () => {
        if (tempImageUri) {
            try {
                const file = new File(tempImageUri);
                if (file.exists) {
                    file.delete();
                }
            } catch (error) {
                console.log("Error deleting temporary file:", error);
            }
        }
        setModalVisible(false);
        setTempImageUri(null);
        setPhotoTitle('');
    };

    return (
        <View style={styles.container}>
            <Button onPress={takeAndSavePhoto} title="Prendre une photo" />

            <Text style={styles.title}>Galerie locale ({images.length})</Text>

            <FlatList
                data={images}
                numColumns={2}
                columnWrapperStyle={styles.row} 
                keyExtractor={(item) => item.id.toString()}
                renderItem={({ item }) => (
                    <View style={styles.item}>
                        <Image source={{ uri: item.uri }} style={styles.image} />
                        <Text style={styles.imageTitle}>{item.title}</Text>
                        <Button title="Supprimer" color="red" onPress={() => deletePhoto(item.id)} />
                    </View>
                )}
            />

            {/* Modale pour entrer le titre après la prise de photo */}
            <Modal visible={modalVisible} animationType="slide" transparent={true}>
                <View style={styles.modalOverlay}>
                    <View style={styles.modalContent}>
                        <Text style={styles.modalTitle}>Ajouter un titre à la photo</Text>

                        {tempImageUri && (
                            <Image source={{ uri: tempImageUri }} style={styles.modalImage} />
                        )}

                        <TextInput
                            style={styles.input}
                            placeholder="Entrez le titre ici..."
                            value={photoTitle}
                            onChangeText={setPhotoTitle}
                        />

                        <View style={styles.modalButtons}>
                            <Button
                                title="Annuler"
                                color="gray"
                                onPress={() => cancelPhoto()}
                            />
                            <Button
                                title="Valider"
                                onPress={savePhotoWithTitle}
                            />
                        </View>
                    </View>
                </View>
            </Modal>
        </View>
    );
}

export default Photo;

const styles = StyleSheet.create({
    container: {
        flex: 1,
        padding: 40,
        backgroundColor: '#fff',
        marginTop: 30
    },
    title: {
        fontSize: 18,
        fontWeight: 'bold',
        marginVertical: 15
    },
    item: {
        marginBottom: 20,
        alignItems: 'center',
        flex:1
    },
    image: {
        width: 150,
        height: 150,
        borderRadius: 10,
        marginBottom: 5
    },
    imageTitle: {
        fontSize: 16,
        fontWeight: '600',
        marginBottom: 5
    },
    modalOverlay: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: 'rgba(0,0,0,0.5)'
    },
    modalContent: {
        width: '85%',
        backgroundColor: 'white',
        borderRadius: 15,
        padding: 20,
        alignItems: 'center',
        elevation: 5
    },
    modalTitle: {
        fontSize: 18,
        fontWeight: 'bold',
        marginBottom: 15
    },
    modalImage: {
        width: 150,
        height: 150,
        borderRadius: 10,
        marginBottom: 15
    },
    input: {
        width: '100%',
        borderWidth: 1,
        borderColor: '#ccc',
        borderRadius: 8,
        padding: 10,
        marginBottom: 20,
        backgroundColor: '#f9f9f9'
    },
    modalButtons: {
        flexDirection: 'row',
        justifyContent: 'space-between',
        width: '100%'
    },
    row: {
        justifyContent: 'flex-start', 
        gap: 10, 
        marginBottom: 10, 
    }
}); -->