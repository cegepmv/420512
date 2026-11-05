+++
date = '2026-06-25T16:02:29-04:00'
draft = false
weight = 15
title = 'Formatif 2'
+++

Voici une version mise à jour et structurée de votre cahier des charges, intégrant l'ensemble de vos nouvelles exigences (thèmes, animation de la voiture et anticipation de l'API d'examen) :

---

# Cahier des Charges : Application Mobile "Click Speed Test"

## 1. Contexte et Objectif du Projet

Développer une application mobile (iOS/Android) de test de vitesse de clic (*Click Speed Test*). L'utilisateur doit effectuer 20 clics le plus rapidement possible. L'application mesure le temps total écoulé et gère un classement local et distant.

---

## 2. Fonctionnalités Principales & Logique de Jeu

* **Le Test de Clic :**
* L'utilisateur effectue 20 clics successifs.
* L'application mesure le temps écoulé du premier au vingtième clic.


* **Animation de la Voiture (Progression) :**
* Une barre de progression représente l'avancement des clics à l'aide d'une **voiture animée de manière fluide** (pas de téléportation).
* **0 clic :** La voiture est tout à **gauche** (0 %).
* **10 clics :** La voiture est au **centre** (50 %).
* **20 clics :** La voiture est à **droite** (100 %).


* **Saisie et Enregistrement du Score :**
* À la fin du test, si le score entre dans le **Top 5 local**, l'utilisateur peut saisir son nom pour enregistrer son temps, sinon le bouton est désactivé.



---

## 3. Architecture Offline-First & Synchronisation

* **Base de Données Locale (SQLite) :**
* Stockage local des scores et du leaderboard pour garantir un fonctionnement hors-ligne (*offline-first*).


* **Base de Données Distante (Backend Express) :**
* Un serveur Express gère la synchronisation globale et le classement partagé (*leaderboard*).


* **Stratégie de Synchronisation :**
* Mise à jour automatique des données **toutes les minutes** et **à la fin de chaque partie**.
* **Gestion des conflits :** Aucune logique complexe (*last-write-wins* ou *last-delete*) n'est nécessaire, les collisions étant impossibles.
* **Rôle du Front-End :** Enregistrer les nouveaux scores localement, puis les transmettre au backend Express. Le Front-End envoie uniquement les modifications lorsqu'elles se produisent. Le backend se charge entièrement de la gestion du leaderboard global.



---

## 4. Expérience Utilisateur & Design

* **Gestion des Thèmes :**
* Prise en charge complète des **thèmes Light et Dark** disponibles dynamiquement et uniformément dans toute l'application.



---

## 5. Note Contextuelle (Préparation Examen)

* *Bien qu'aucune API native ne soit requise pour le cœur du projet actuel, le code doit être conçu de manière modulaire pour pouvoir intégrer facilement **une API native lors de l'examen** (ex: Caméra, Capteurs/Sensors).*

---

{{% notice style="warning" %}}
Attention, le code ci-dessous comporte des erreurs. Il est une très bonne base pour commencer par contre.
{{% /notice %}}

```js
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
            CREATE TABLE IF NOT EXISTS scores (
                id INT AUTO_INCREMENT PRIMARY KEY,
                name VARCHAR(100) NOT NULL,
                score FLOAT NOT NULL,
                updated_at DATETIME NOT NULL
            );
        `;
        await pool.query(query);
        console.log("✅ Table 'scores' vérifiée / créée avec succès.");
    } catch (error) {
        console.error("❌ Erreur lors de la création de la table :", error);
    }
}

app.post('/scores', async (req, res) => {
    try {
        const { title, updated_at } = req.body;
        
        // Convertit la date au format string compatible MySQL
        const dateToSave = updated_at 
            ? new Date(updated_at).toISOString().slice(0, 19).replace('T', ' ')
            : new Date().toISOString().slice(0, 19).replace('T', ' ');

        const [result] = await pool.query(
            "INSERT INTO scores (title, updated_at) VALUES (?, ?)",
            [title, dateToSave]
        );
        
        res.status(201).json({ id: result.insertId, title, updated_at: dateToSave });
    } catch (error) {
        console.error(error);
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// READ (Tous - avec filtrage intelligent via lastSync optionnel)
app.get('/scores', async (req, res) => {
    try {
        const { lastSync } = req.query;
        let query = "SELECT * FROM scores";
        let params = [];

        // Si le client fournit une date de dernière synchro, on ne renvoie que ce qui a changé depuis
        if (lastSync) {
            query += " WHERE updated_at >= ?";
            params.push(lastSync);
        }

        const [scores] = await pool.query(query, params);
        res.json(scores);
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// READ (Un seul)
app.get('/scores/:id', async (req, res) => {
    try {
        const [scores] = await pool.query("SELECT * FROM scores WHERE id = ?", [req.params.id]);
        if (scores.length === 0) return res.status(404).json({ erreur: "Non trouvé" });
        res.json(scores[0]);
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// UPDATE
app.put('/scores/:id', async (req, res) => {
    try {
        const { title, updated_at } = req.body;
        const [result] = await pool.query(
            "UPDATE scores SET title = ?, updated_at = ? WHERE id = ?",
            [title, updated_at || new Date(), req.params.id]
        );
        if (result.affectedRows === 0) return res.status(404).json({ erreur: "Non trouvé" });
        res.json({ message: "Mis à jour avec succès !" });
    } catch (error) {
        res.status(500).json({ erreur: "Erreur serveur" });
    }
});

// DELETE
app.delete('/scores/:id', async (req, res) => {
    try {
        const [result] = await pool.query("DELETE FROM scores WHERE id = ?", [req.params.id]);
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

{{% notice style="info" %}}
Code à compléter
{{% /notice %}}

```jsx
import React, { useState, useEffect, useRef } from 'react';
import { StyleSheet, Text, View, TouchableOpacity, TextInput, useWindowDimensions, Alert } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import Animated, { useSharedValue, useAnimatedStyle, withTiming, Easing } from 'react-native-reanimated';
import * as SQLite from 'expo-sqlite';
import NetInfo from '@react-native-community/netinfo';

const TARGET_CLICKS = 20;
const API_URL = `http://${process.env.EXPO_PUBLIC_BACKEND_IP}:3000/`; // Remplacez par l'IP de votre serveur Express

// Initialisation de la base de données SQLite locale unique
const db = SQLite.openDatabaseSync('grand_prix_v2.db');

export default function App() {
  const isOnline = false
  const time = 0
  const carAnimatedStyle = null
  const isFinished = false
  const handlePress = null
  const clicksCount = 0
  const topScores = []
  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Grand Prix Expo</Text>
        <Text style={[styles.statusText, { color: isOnline ? '#00ffcc' : '#ff4757' }]}>
          {isOnline ? '🟢 En ligne' : '🔴 Hors-ligne (Table unique SQLite)'}
        </Text>
      </View>

      <View style={styles.chronoContainer}>
        <Text style={styles.chronoText}>{time.toFixed(2)} s</Text>
      </View>

      <View style={styles.trackContainer}>
        <View style={styles.track}>
          <View style={styles.startLine} />
          <View style={styles.finishLine} />
          <Animated.View style={[styles.car, carAnimatedStyle]}>
            <View style={styles.windshield} />
            <View style={styles.wheelFront} />
            <View style={styles.wheelBack} />
          </Animated.View>
        </View>
      </View>

      <View style={styles.controls}>
        {!isFinished ? (
          <>
            <TouchableOpacity style={styles.button} onPress={handlePress} activeOpacity={0.7}>
              <Text style={styles.buttonText}>ACCÉLÉRER !</Text>
            </TouchableOpacity>
            <Text style={styles.counter}>
              Clics : <Text style={styles.highlight}>{clicksCount} / {TARGET_CLICKS}</Text>
            </Text>
          </>
        ) : (
          <View style={styles.winContainer}>
            <Text style={styles.winText}>🏁 Temps : {time.toFixed(2)}s !</Text>
            <TextInput
              style={styles.input}
              placeholder="Votre nom / pseudo"
              placeholderTextColor="#888"
              value={playerName}
              onChangeText={setPlayerName}
            />
            <TouchableOpacity style={[styles.button, styles.saveButton]} onPress={saveScore} activeOpacity={0.8}>
              <Text style={styles.buttonText}>Enregistrer le score</Text>
            </TouchableOpacity>
          </View>
        )}
      </View>

      <View style={styles.leaderboard}>
        <Text style={styles.leaderboardTitle}>🏆 Top 5 des Meilleurs Scores</Text>
        {topScores.map((item, index) => (
          <View key={index} style={styles.scoreRow}>
            <Text style={styles.scoreText}>{index + 1}. {item.name}</Text>
            <Text style={styles.scoreTime}>{item.score} s</Text>
          </View>
        ))}
      </View>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: '#0f1020', justifyContent: 'space-between', alignItems: 'center', paddingVertical: 20 },
  header: { alignItems: 'center' },
  title: { fontSize: 26, fontWeight: '900', color: '#ff2e63', textTransform: 'uppercase', letterSpacing: 2 },
  statusText: { fontSize: 12, marginTop: 4, fontWeight: 'bold' },
  chronoContainer: { backgroundColor: '#1b1c31', paddingVertical: 8, paddingHorizontal: 25, borderRadius: 15, borderWidth: 2, borderColor: '#3d3d5c' },
  chronoText: { fontSize: 32, fontWeight: 'bold', color: '#00ffcc', fontVariant: ['tabular-nums'] },
  trackContainer: { width: '92%', backgroundColor: '#1b1c31', borderRadius: 20, padding: 15, borderWidth: 2, borderColor: '#e94560' },
  track: { position: 'relative', height: 90, backgroundColor: '#12121e', borderRadius: 12, justifyContent: 'center', overflow: 'hidden' },
  startLine: { position: 'absolute', left: 0, top: 0, bottom: 0, width: 8, backgroundColor: '#e74c3c' },
  finishLine: { position: 'absolute', right: 0, top: 0, bottom: 0, width: 16, backgroundColor: '#ffffff' },
  car: { position: 'absolute', left: 10, width: 70, height: 38, backgroundColor: '#ff2e63', borderRadius: 8, justifyContent: 'center' },
  windshield: { position: 'absolute', right: 12, top: 6, width: 20, height: 14, backgroundColor: '#0ff', borderTopRightRadius: 6 },
  wheelFront: { position: 'absolute', bottom: -6, left: 12, width: 14, height: 14, backgroundColor: '#111', borderRadius: 7, borderWidth: 2, borderColor: '#fff' },
  wheelBack: { position: 'absolute', bottom: -6, right: 12, width: 14, height: 14, backgroundColor: '#111', borderRadius: 7, borderWidth: 2, borderColor: '#fff' },
  controls: { alignItems: 'center', width: '100%' },
  button: { backgroundColor: '#ff2e63', paddingVertical: 15, paddingHorizontal: 35, borderRadius: 40, alignItems: 'center' },
  saveButton: { backgroundColor: '#00b894', marginTop: 10 },
  buttonText: { color: '#ffffff', fontSize: 18, fontWeight: '900', letterSpacing: 1 },
  counter: { fontSize: 16, color: '#ffffff', marginTop: 15 },
  highlight: { color: '#ff2e63', fontWeight: 'bold', fontSize: 18 },
  winContainer: { alignItems: 'center', width: '100%' },
  winText: { fontSize: 22, fontWeight: '900', color: '#00b894', marginBottom: 10 },
  input: { width: '80%', backgroundColor: '#1b1c31', color: '#fff', padding: 12, borderRadius: 10, textAlign: 'center', fontSize: 16, borderWidth: 1, borderColor: '#3d3d5c', marginBottom: 10 },
  leaderboard: { width: '85%', backgroundColor: '#1b1c31', padding: 12, borderRadius: 12, borderWidth: 1, borderColor: '#3d3d5c' },
  leaderboardTitle: { color: '#ff2e63', fontSize: 16, fontWeight: 'bold', textAlign: 'center', marginBottom: 8 },
  scoreRow: { flexDirection: 'row', justifyContent: 'space-between', paddingVertical: 4, borderBottomWidth: 1, borderBottomColor: '#252642' },
  scoreText: { color: '#fff', fontSize: 14 },
  scoreTime: { color: '#00ffcc', fontSize: 14, fontWeight: 'bold' }
});
```