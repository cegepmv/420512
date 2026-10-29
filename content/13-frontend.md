+++
date = '2026-06-25T16:02:29-04:00'
draft = false
weight = 13
pre="13. "
title = 'Frontend'
+++

{{% expand title="Code Backend"%}}

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

{{% /expand %}}


# FrontEnd

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

const API_URL = `http://${process.env.EXPO_PUBLIC_BACKEND_IP}:3000/parcs`;

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
    paddingHorizontal: 20
  },
  headerTitle: {
    fontSize: 22,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
    color: '#333'
  },
  formContainer: {
    backgroundColor: '#fff',
    padding: 15,
    borderRadius: 10,
    marginBottom: 20,
    shadowColor: '#000',
    shadowOpacity: 0.05,
    elevation: 2
  },
  input: {
    borderWidth: 1,
    borderColor: '#ddd',
    padding: 10,
    borderRadius: 6,
    marginBottom: 10,
    backgroundColor: '#fdfdfd'
  },
  addButton: {
    backgroundColor: '#007AFF',
    padding: 12,
    borderRadius: 6,
    alignItems: 'center'
  },
  addButtonText: {
    color: '#fff',
    fontWeight: 'bold',
    fontSize: 16
  },
  center: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  card: {
    flexDirection: 'row',
    backgroundColor: '#fff',
    padding: 15,
    borderRadius: 8,
    marginBottom: 10,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOpacity: 0.05,
    elevation: 2
  },
  cardInfo: {
    flex: 1
  },
  parcName: {
    fontSize: 16,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 4
  },
  parcDetails: {
    fontSize: 14,
    color: '#666'
  },
  deleteButton: {
    padding: 10
  },
  deleteButtonText: {
    fontSize: 18
  },
  emptyText: {
    textAlign: 'center',
    color: '#888',
    marginTop: 20
  }
});

```



{{% notice style="exo" %}}
Reprenez la solution de l'exercice du Gestionnaire de tâche. Remplacer l'enregistrement local par de l'enregistrement sur bd distante.

```sql
CREATE TABLE IF NOT EXISTS tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title TEXT NOT NULL,
    completed TINYINT DEFAULT 0
);
```
{{% expand title="Solution"%}}
```js
import express from 'express';
import dotenv from 'dotenv';
import pool from './database.js';

dotenv.config();

const app = express();
const port = process.env.PORT || 3000;

app.use(express.json());

// CREATE
app.post('/tasks', async (req, res) => {
    try {
        const { title, completed } = req.body;
        const [result] = await pool.query(
            "INSERT INTO tasks (title, completed) VALUES (?, ?)",
            [title, completed]
        );
        res.status(201).json({ id: result.insertId, title, completed });
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// READ (Tous)
app.get('/tasks', async (req, res) => {
    try {
        const [tasks] = await pool.query("SELECT * FROM tasks");
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
        const { title, completed } = req.body;
        const [result] = await pool.query(
            "UPDATE tasks SET nom = ?, ville = ?, cout_billet = ? WHERE id = ?",
            [title, completed, req.params.id]
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
              {data.title}
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

  // 1. READ - Récupérer toutes les tâches (GET)
  const fetchTasks = async () => {
    try {
      const response = await axios.get(API_URL);
      setTasks(response.data);
    } catch (error) {
      Alert.alert("Erreur", error.message || "Erreur lors du chargement des tâches.");
    } finally {
      // On arrête le loading seulement au premier chargement initial
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchTasks();
  }, []);

  // 3. DELETE - Mise à jour locale sans recharger toute la liste
  const supprimerTask = async (id) => {
    // Suppression optimiste (met à jour l'UI instantanément)
    setTasks((prevTasks) => prevTasks.filter((task) => task.id !== id));

    try {
      await axios.delete(`${API_URL}/${id}`);
    } catch (error) {
      Alert.alert("Erreur", "Erreur lors de la suppression.");
      fetchTasks(); // En cas d'erreur serveur, on resynchronise
    }
  };

  // 2. CREATE - Ajout de la nouvelle tâche directement dans le state
  const ajouterTask = async () => {
    if (curInput.trim() == "") {
      Alert.alert("Attention", "Veuillez remplir tous les champs.");
      return;
    }

    try {
      const response = await axios.post(API_URL, {
        "title": curInput,
        "completed": 0
      });

      // Le backend renvoie généralement la nouvelle tâche créée avec son ID
      setTasks((prevTasks) => [...prevTasks, response.data]);
      setCurInput("");
    } catch (error) {
      Alert.alert("Erreur", error.response?.data?.erreur || "Erreur lors de l'ajout de la tâche.");
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
{{% /expand%}}
{{% /notice %}}