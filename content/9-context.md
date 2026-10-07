+++
date = '2026-06-25T16:02:29-04:00'
draft = false
weight = 9
pre="9. "
title = 'Context'
+++

Le **Context API**, c'est comme installer un réseau Wi-Fi dans votre application. Au lieu de faire passer des données manuellement de composant en composant (le *props drilling* 😵), vous créez une zone de partage où n'importe quel composant peut se connecter instantanément pour récupérer ou modifier des informations !

---

## 🗂️ Problématique : La navigation imbriquée

Copier ce code dans votre projet :

```jsx
// _layout.jsx
import { Drawer } from 'expo-router/drawer';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import "../global.css"
import { StatusBar } from 'expo-status-bar';
export default function RootLayout() {
  return (
    // 1. Envelopper l'application pour gérer les gestes
    <GestureHandlerRootView style={{ flex: 1 }}>
      <StatusBar style="light" />
      {/* 2. Initialiser le Drawer avec des options globales */}
      <Drawer
        screenOptions={{
          headerStyle: { backgroundColor: '#6200ee' },
          headerTintColor: '#fff',
          drawerActiveTintColor: '#6200ee',
          drawerType: 'front', // 'front', 'back', ou 'slide'
          
        }}
      >
        {/* 3. Déclarer chaque écran */}
        <Drawer.Screen
          name="index"
          options={{
            drawerLabel: 'Accueil',
            title: 'Bienvenue',
          }}
        />
        <Drawer.Screen
          name="(pokemon)"
          options={{
            drawerLabel: 'Pokemon',
          }}
          />
      </Drawer>

    </GestureHandlerRootView>
  );
}
```

```jsx
//(pokemon)/_layout.jsx
import { NativeTabs } from 'expo-router/unstable-native-tabs';

export default function TabLayout() {
  return (
    <NativeTabs>
      <NativeTabs.Trigger name="pokemon-start">
        <NativeTabs.Trigger.Label>Start</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Icon sf="flag.fill" md="flag" />
      </NativeTabs.Trigger>
      
      <NativeTabs.Trigger name="pokemon-end">
        <NativeTabs.Trigger.Label>End</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Icon sf="flag.checkered" md="check_box" />
      </NativeTabs.Trigger>
    </NativeTabs>
  );
}
```
```jsx
//(pokemon)/pokemon-start.jsx
import React, { useState } from 'react';
import { StyleSheet, Text, View, ScrollView, TouchableOpacity } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';

// Définition des types en français et correspondance des indices
const TYPES = [
  "Normal", "Combat", "Vol", "Poison", "Sol", 
  "Roche", "Insecte", "Spectre", "Acier", "Feu", 
  "Eau", "Plante", "Électrik", "Psy", "Glace", 
  "Dragon", "Ténèbres"
];

// Couleurs officielles des types Pokémon
const TYPE_COLORS = {
  Normal: "#A8A878",
  Combat: "#C03028",
  Vol: "#A890F0",
  Poison: "#A040A0",
  Sol: "#E0C068",
  Roche: "#B8A038",
  Insecte: "#A8B820",
  Spectre: "#705898",
  Acier: "#B8B8D0",
  Feu: "#F08030",
  Eau: "#6890F0",
  Plante: "#78C850",
  Électrik: "#F8D030",
  Psy: "#F85888",
  Glace: "#98D8D8",
  Dragon: "#7038F8",
  Ténèbres: "#705848"
};

// Matrice des types FireRed (17x17) (Lignes : Attaquant, Colonnes : Défenseur)
const TYPE_MATRIX = [
  /* Nor */ [1, 1, 1, 1, 1, 0.5, 1, 0, 0.5, 1, 1, 1, 1, 1, 1, 1, 1],
  /* Com */ [2, 1, 0.5, 0.5, 1, 2, 0.5, 0, 2, 1, 1, 1, 1, 0.5, 2, 1, 2],
  /* Vol */ [1, 2, 1, 1, 1, 0.5, 2, 1, 0.5, 1, 1, 2, 0.5, 1, 1, 1, 1],
  /* Poi */ [1, 1, 1, 0.5, 0.5, 0.5, 1, 0.5, 0, 1, 1, 2, 1, 1, 1, 1, 1],
  /* Sol */ [1, 1, 0, 2, 1, 2, 0.5, 1, 2, 2, 1, 0.5, 2, 1, 1, 1, 1],
  /* Roc */ [1, 0.5, 2, 1, 0.5, 1, 2, 1, 0.5, 2, 1, 1, 1, 1, 2, 1, 1],
  /* Ins */ [1, 0.5, 0.5, 0.5, 1, 1, 1, 0.5, 0.5, 0.5, 1, 2, 1, 2, 1, 1, 2],
  /* Spe */ [0, 1, 1, 1, 1, 1, 1, 2, 0.5, 1, 1, 1, 1, 2, 1, 1, 0.5],
  /* Aci */ [1, 1, 1, 1, 1, 2, 1, 1, 0.5, 0.5, 0.5, 1, 0.5, 1, 2, 1, 1],
  /* Feu */ [1, 1, 1, 1, 1, 0.5, 2, 1, 2, 0.5, 0.5, 2, 1, 1, 2, 0.5, 1],
  /* Eau */ [1, 1, 1, 1, 2, 2, 1, 1, 1, 2, 0.5, 0.5, 1, 1, 1, 0.5, 1],
  /* Pla */ [1, 1, 0.5, 0.5, 2, 2, 0.5, 1, 0.5, 0.5, 2, 0.5, 1, 1, 1, 0.5, 1],
  /* Ele */ [1, 1, 2, 1, 0, 1, 1, 1, 1, 1, 2, 0.5, 0.5, 1, 1, 0.5, 1],
  /* Psy */ [1, 2, 1, 2, 1, 1, 1, 1, 0.5, 1, 1, 1, 1, 0.5, 1, 1, 0],
  /* Gla */ [1, 1, 2, 1, 2, 1, 1, 1, 0.5, 0.5, 0.5, 2, 1, 1, 0.5, 2, 1],
  /* Dra */ [1, 1, 1, 1, 1, 1, 1, 1, 0.5, 1, 1, 1, 1, 1, 1, 2, 1],
  /* Tén */ [1, 0.5, 1, 1, 1, 1, 1, 2, 0.5, 1, 1, 1, 1, 2, 1, 1, 0.5]
];

const TypeCalculator = () => {
  return (
    <>
      <Text>🔥 Table des Types Pokémon 🍃</Text>
      <Text>COMBAT ! ⚔️</Text>
      <Text>👑</Text>
    </>
  );
}

export default TypeCalculator
```
```jsx
//(pokemon)/pokemon-end.jsx
import React, { useState } from 'react';
import { StyleSheet, Text, View, ScrollView, TouchableOpacity } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';

// Définition des types en français et correspondance des indices
const TYPES = [
  "Normal", "Combat", "Vol", "Poison", "Sol", 
  "Roche", "Insecte", "Spectre", "Acier", "Feu", 
  "Eau", "Plante", "Électrik", "Psy", "Glace", 
  "Dragon", "Ténèbres"
];

// Couleurs officielles des types Pokémon
const TYPE_COLORS = {
  Normal: "#A8A878",
  Combat: "#C03028",
  Vol: "#A890F0",
  Poison: "#A040A0",
  Sol: "#E0C068",
  Roche: "#B8A038",
  Insecte: "#A8B820",
  Spectre: "#705898",
  Acier: "#B8B8D0",
  Feu: "#F08030",
  Eau: "#6890F0",
  Plante: "#78C850",
  Électrik: "#F8D030",
  Psy: "#F85888",
  Glace: "#98D8D8",
  Dragon: "#7038F8",
  Ténèbres: "#705848"
};

// Matrice des types FireRed (17x17) (Lignes : Attaquant, Colonnes : Défenseur)
const TYPE_MATRIX = [
  /* Nor */ [1, 1, 1, 1, 1, 0.5, 1, 0, 0.5, 1, 1, 1, 1, 1, 1, 1, 1],
  /* Com */ [2, 1, 0.5, 0.5, 1, 2, 0.5, 0, 2, 1, 1, 1, 1, 0.5, 2, 1, 2],
  /* Vol */ [1, 2, 1, 1, 1, 0.5, 2, 1, 0.5, 1, 1, 2, 0.5, 1, 1, 1, 1],
  /* Poi */ [1, 1, 1, 0.5, 0.5, 0.5, 1, 0.5, 0, 1, 1, 2, 1, 1, 1, 1, 1],
  /* Sol */ [1, 1, 0, 2, 1, 2, 0.5, 1, 2, 2, 1, 0.5, 2, 1, 1, 1, 1],
  /* Roc */ [1, 0.5, 2, 1, 0.5, 1, 2, 1, 0.5, 2, 1, 1, 1, 1, 2, 1, 1],
  /* Ins */ [1, 0.5, 0.5, 0.5, 1, 1, 1, 0.5, 0.5, 0.5, 1, 2, 1, 2, 1, 1, 2],
  /* Spe */ [0, 1, 1, 1, 1, 1, 1, 2, 0.5, 1, 1, 1, 1, 2, 1, 1, 0.5],
  /* Aci */ [1, 1, 1, 1, 1, 2, 1, 1, 0.5, 0.5, 0.5, 1, 0.5, 1, 2, 1, 1],
  /* Feu */ [1, 1, 1, 1, 1, 0.5, 2, 1, 2, 0.5, 0.5, 2, 1, 1, 2, 0.5, 1],
  /* Eau */ [1, 1, 1, 1, 2, 2, 1, 1, 1, 2, 0.5, 0.5, 1, 1, 1, 0.5, 1],
  /* Pla */ [1, 1, 0.5, 0.5, 2, 2, 0.5, 1, 0.5, 0.5, 2, 0.5, 1, 1, 1, 0.5, 1],
  /* Ele */ [1, 1, 2, 1, 0, 1, 1, 1, 1, 1, 2, 0.5, 0.5, 1, 1, 0.5, 1],
  /* Psy */ [1, 2, 1, 2, 1, 1, 1, 1, 0.5, 1, 1, 1, 1, 0.5, 1, 1, 0],
  /* Gla */ [1, 1, 2, 1, 2, 1, 1, 1, 0.5, 0.5, 0.5, 2, 1, 1, 0.5, 2, 1],
  /* Dra */ [1, 1, 1, 1, 1, 1, 1, 1, 0.5, 1, 1, 1, 1, 1, 1, 2, 1],
  /* Tén */ [1, 0.5, 1, 1, 1, 1, 1, 2, 0.5, 1, 1, 1, 1, 2, 1, 1, 0.5]
];

const TypeCalculator = () => {
  const [typeAttaquant, setTypeAttaquant] = useState([])
  const [typeDefenseur, setTypeDefenseur] = useState([])
  const [isResAffiche, setIsResAffiche] = useState(false)
  const [battleResult, setBattleResult] = useState(null)
  const handlePress = (type, liste, setListe) => {
    if (liste.includes(type)) {
      setListe(liste.filter((t) => t != type))
      setIsResAffiche(false)
    }
    else if (liste.length < 2){
      setListe([...liste, type])
      setIsResAffiche(false)
    }
    
  }
  const handleCombat = () => {
    if (typeAttaquant.length < 1 || typeDefenseur.length < 1) {
      setBattleResult({"error" : "Veuillez choisir au moins 1 type attaquant et 1 défenseur !"})
      setIsResAffiche(true)
      return;
    }
    if(isResAffiche) {
      return;
    }
    let multiplicateurTotal = 1
    let msgCombat = []
    let result = ""

    typeAttaquant.forEach((typeAtt) => {
      const indexAtt = TYPES.indexOf(typeAtt)
      typeDefenseur.forEach((typeDef) => {
        const indexDef = TYPES.indexOf(typeDef)
        multiplicateurTotal = multiplicateurTotal * TYPE_MATRIX[indexAtt][indexDef]
        msgCombat.push(`• ${typeAtt} contre ${typeDef} = ${TYPE_MATRIX[indexAtt][indexDef]}x`)
      });
    });

    if (multiplicateurTotal > 1) {
      result = "Victoire de l'attaquant"
    }
    else if (multiplicateurTotal == 1) {
      result = "Égalité / Neutre"
    }
    else {
      result = "Victoire du défenseur"
    }
    setBattleResult({
      multiplicateurTotal,
      result,
      msgCombat
    })
    setIsResAffiche(true)

  }
  return (
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scroll}>
        <Text style={styles.titre}>🔥 Table des Types Pokémon 🍃</Text>
        <Text style={styles.titreType}>1. Type(s) Attaquant(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type, index) => {
            const isSelected = typeAttaquant.includes(type);
            return (
              <TouchableOpacity key={`att-${index}`} onPress={() => handlePress(type, typeAttaquant, setTypeAttaquant)} style={[styles.btnType, isSelected && {backgroundColor : TYPE_COLORS[type]}]}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <Text style={styles.titreType}>2. Type(s) Defenseur(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type, index) => {
            const isSelected = typeDefenseur.includes(type)
            return (
              <TouchableOpacity key={`def-${index}`} onPress={() => handlePress(type, typeDefenseur, setTypeDefenseur)} style={[styles.btnType, isSelected && {backgroundColor : TYPE_COLORS[type]}]}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <TouchableOpacity onPress={handleCombat} style={styles.btnCombat}>
          <Text style={styles.textCombat}>
            COMBAT ! ⚔️
          </Text>
        </TouchableOpacity>
        {isResAffiche && (
            battleResult["error"] ? (
                <View style={styles.sectionResultats}>
                  <Text style={styles.errorMsg}> {battleResult["error"]}</Text>
                </View>
              )
              :
              (

                <View style={styles.sectionResultats}>
                  <Text style={styles.couronne}>👑</Text>
                  <Text style={styles.result}>{battleResult["result"]}</Text>
                  <Text style={styles.multi}>Multiplicateur total : {battleResult["multiplicateurTotal"]}x</Text>
                  <View style={styles.containerExplication}>
                    <Text style={styles.pourquoi}>Pourquoi ?</Text>
                    {battleResult["msgCombat"].map((msg, index) => (
                      <Text style={styles.explications} key={`msgCombat-${index}`}>{msg}</Text>
                    ))}
                  </View>
                </View>
              )

             
          )
        }
      </ScrollView>
    </SafeAreaView>
  );
}

export default TypeCalculator



const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#161616',
    padding: 25,
  },
  titre: {
    color: "#ffffff",
    textAlign: "center",
    fontSize: 20,
    fontWeight: "700",
    paddingVertical: 30,
  },
  btnCombat: {
    backgroundColor: "#15ff00c7",
    paddingVertical: 15,
    borderRadius: 10
  },
  scroll: {

  },
  sectionResultats: {
    backgroundColor: "#49494981",
    borderColor: "#494949",
    borderRadius:10,
    borderWidth:1,
    marginVertical: 20,
    padding: 20
  },
  textCombat: {
    textAlign: "center",
    fontSize: 20,
    fontWeight: "500",
    color: "#ffffff",
    

  },
  couronne: {
    textAlign: "center",
    fontSize: 40
  },
  titreType: {
    color: "#918d8d",
    fontWeight: "600",

  },
  textBtnType: {
    color: "#ffffff"
  },
  containerType: {
    flexDirection: "row",
    flexWrap: "wrap",
    paddingBottom: 20,
    
  },
  btnType: {
    padding: 5,
    backgroundColor: "#49494981",
    borderColor: "#494949",
    borderRadius:10,
    borderWidth:1,
    margin: 5
  },
  errorMsg: {
    color: "#c56a6a",
    fontSize: 18
  },
  result: {
    color: "#ffe600",
    textAlign: "center",
    fontSize: 20,
    fontWeight:"700"
  },
  containerExplication: {
    backgroundColor: '#161616',
    borderRadius: 10,
    padding: 10
  },
  explications: {
    color: "#ffffff"
  },
  multi: {
    color: "#ffffff",
    fontSize: 18,
    fontWeight: "500",
    textAlign: "center",
    padding: 10
  },
  pourquoi: {
    color: "#ffffff",
    fontWeight: "600",
    padding:5,
    fontSize: 15
  }
})
```


---



## 🛠️ Étape par étape : Le TabNameContext

### 📁 Étape 1 : Créer le réseau (Le fichier Context)

Créez un fichier nommé `tabNameContext.js` dans votre dossier `components/`.

```jsx
import React, { createContext, useContext, useState } from 'react';

// 1. On crée notre boîte de contexte
const TabNameContext = createContext();

// 2. On crée le Provider qui va distribuer les données
export const TabNameProvider = ({ children }) => {
    const [tabName, setTabName] = useState('Nested Tabs');

    return (
        <TabNameContext.Provider value={{ tabName, setTabName }}>
            {children}
        </TabNameContext.Provider>
    );
};

// 3. Un petit hook personnalisé pour se connecter super facilement !
export const useTabNameContext = () => {
    return useContext(TabNameContext);
};

```

### 🔌 Étape 2 : Brancher le Provider à la racine

Il faut envelopper (*wrap*) les écrans qui ont besoin de ces données avec notre `TabNameProvider`. Modifiez votre `_layout.jsx` racine :

```jsx
import React from 'react';
import { Drawer } from 'expo-router/drawer';
import { TabNameProvider, useTabNameContext } from '../contexts/tabNameContext';

const RootLayout = () => {
  return (
      <TabNameProvider>
        <Layout />
      </TabNameProvider>
  );
};

const Layout = () => {
  // On récupère la variable globale contenant le titre
  const { tabName } = useTabNameContext();

  return (
      <Drawer 
        screenOptions={{ 
          headerStyle: { backgroundColor: 'lightblue' } 
        }}
      >
        <Drawer.Screen name="index" />
        <Drawer.Screen name="speedTest" />
        {/* Le titre de cette section s'adaptera dynamiquement ! */}
        <Drawer.Screen 
          name="(tabs)" 
          options={{ 
            drawerLabel: 'Tabs', 
            title: tabName 
          }} 
        />
      </Drawer>
  );
};

export default RootLayout;

```

### ⚡ Étape 3 : Mettre à jour le titre depuis les onglets

On utilise `useFocusEffect` pour changer le titre dès que l'utilisateur clique sur l'onglet !

```jsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { useFocusEffect } from 'expo-router';
import { useTabNameContext } from '../../contexts/tabNameContext';

export default function TabOneScreen() {
  const { setTabName } = useTabNameContext();

  useFocusEffect(
    React.useCallback(() => {
      // 1. On change le nom dès que l'écran devient actif 🎯
      setTabName('Mon Super Onglet 1');

      // 2. Optionnel : Nettoyage quand on quitte l'onglet
      return () => {};
    }, [setTabName])
  );

  return (
    <View style={styles.container}>
      <Text style={styles.text}>Bienvenue sur l'onglet 1 !</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  text: { fontSize: 18 }
});

```

{{% expand title="Solution" %}}
```jsx
//_layout.jsx
import { Drawer } from 'expo-router/drawer';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import "../global.css"
import { StatusBar } from 'expo-status-bar';
import { TabNameProvider, useTabNameContext } from '../contexts/tabNameContext';
const RootLayout = () => {
  return (
      <GestureHandlerRootView style={{ flex: 1 }}>
        <TabNameProvider>
          <Layout />
        </TabNameProvider>
      </GestureHandlerRootView>
  );
};

export default RootLayout;

const Layout = () => {
  const { tabName } = useTabNameContext()
  return (
    <>
      {/* // 1. Envelopper l'application pour gérer les gestes */}
      <StatusBar style="light" />
      {/* 2. Initialiser le Drawer avec des options globales */}
      <Drawer
        screenOptions={{
          headerStyle: { backgroundColor: '#6200ee' },
          headerTintColor: '#fff',
          drawerActiveTintColor: '#6200ee',
          drawerType: 'front', // 'front', 'back', ou 'slide'
          
        }}
        >
        {/* 3. Déclarer chaque écran */}
        <Drawer.Screen
          name="index"
          options={{
            drawerLabel: 'Accueil',
            title: 'Bienvenue',
          }}
          />
        <Drawer.Screen
          name="(pokemon)"
          options={{
            drawerLabel: 'Pokemon',
            title: tabName
          }}
          />
      </Drawer>
    </>
  
  );
}
```

```jsx
//(pokemon)/pokemon-start.jsx
import React, { useState } from 'react';
import { StyleSheet, Text, View, ScrollView, TouchableOpacity } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useFocusEffect } from 'expo-router';
import { useTabNameContext } from '../../contexts/tabNameContext';


// Définition des types en français et correspondance des indices
const TYPES = [
  "Normal", "Combat", "Vol", "Poison", "Sol",
  "Roche", "Insecte", "Spectre", "Acier", "Feu",
  "Eau", "Plante", "Électrik", "Psy", "Glace",
  "Dragon", "Ténèbres"
];

// Couleurs officielles des types Pokémon
const TYPE_COLORS = {
  Normal: "#A8A878",
  Combat: "#C03028",
  Vol: "#A890F0",
  Poison: "#A040A0",
  Sol: "#E0C068",
  Roche: "#B8A038",
  Insecte: "#A8B820",
  Spectre: "#705898",
  Acier: "#B8B8D0",
  Feu: "#F08030",
  Eau: "#6890F0",
  Plante: "#78C850",
  Électrik: "#F8D030",
  Psy: "#F85888",
  Glace: "#98D8D8",
  Dragon: "#7038F8",
  Ténèbres: "#705848"
};

// Matrice des types FireRed (17x17) (Lignes : Attaquant, Colonnes : Défenseur)
const TYPE_MATRIX = [
  /* Nor */[1, 1, 1, 1, 1, 0.5, 1, 0, 0.5, 1, 1, 1, 1, 1, 1, 1, 1],
  /* Com */[2, 1, 0.5, 0.5, 1, 2, 0.5, 0, 2, 1, 1, 1, 1, 0.5, 2, 1, 2],
  /* Vol */[1, 2, 1, 1, 1, 0.5, 2, 1, 0.5, 1, 1, 2, 0.5, 1, 1, 1, 1],
  /* Poi */[1, 1, 1, 0.5, 0.5, 0.5, 1, 0.5, 0, 1, 1, 2, 1, 1, 1, 1, 1],
  /* Sol */[1, 1, 0, 2, 1, 2, 0.5, 1, 2, 2, 1, 0.5, 2, 1, 1, 1, 1],
  /* Roc */[1, 0.5, 2, 1, 0.5, 1, 2, 1, 0.5, 2, 1, 1, 1, 1, 2, 1, 1],
  /* Ins */[1, 0.5, 0.5, 0.5, 1, 1, 1, 0.5, 0.5, 0.5, 1, 2, 1, 2, 1, 1, 2],
  /* Spe */[0, 1, 1, 1, 1, 1, 1, 2, 0.5, 1, 1, 1, 1, 2, 1, 1, 0.5],
  /* Aci */[1, 1, 1, 1, 1, 2, 1, 1, 0.5, 0.5, 0.5, 1, 0.5, 1, 2, 1, 1],
  /* Feu */[1, 1, 1, 1, 1, 0.5, 2, 1, 2, 0.5, 0.5, 2, 1, 1, 2, 0.5, 1],
  /* Eau */[1, 1, 1, 1, 2, 2, 1, 1, 1, 2, 0.5, 0.5, 1, 1, 1, 0.5, 1],
  /* Pla */[1, 1, 0.5, 0.5, 2, 2, 0.5, 1, 0.5, 0.5, 2, 0.5, 1, 1, 1, 0.5, 1],
  /* Ele */[1, 1, 2, 1, 0, 1, 1, 1, 1, 1, 2, 0.5, 0.5, 1, 1, 0.5, 1],
  /* Psy */[1, 2, 1, 2, 1, 1, 1, 1, 0.5, 1, 1, 1, 1, 0.5, 1, 1, 0],
  /* Gla */[1, 1, 2, 1, 2, 1, 1, 1, 0.5, 0.5, 0.5, 2, 1, 1, 0.5, 2, 1],
  /* Dra */[1, 1, 1, 1, 1, 1, 1, 1, 0.5, 1, 1, 1, 1, 1, 1, 2, 1],
  /* Tén */[1, 0.5, 1, 1, 1, 1, 1, 2, 0.5, 1, 1, 1, 1, 2, 1, 1, 0.5]
];

const TypeCalculator = () => {
  // À mettre à l'intérieur de votre composant de page :
  const { setTabName } = useTabNameContext();

  useFocusEffect(
    React.useCallback(() => {
      // 1. On change le nom dès que l'écran devient actif 🎯
      setTabName('Pokémon Début');

      // 2. Optionnel : Nettoyage quand on quitte l'onglet
      return () => { };
    }, [setTabName])
  );
  return (
    <>
      <Text>🔥 Table des Types Pokémon 🍃</Text>
      <Text>COMBAT ! ⚔️</Text>
      <Text>👑</Text>
    </>
  );
}

export default TypeCalculator
```
```jsx
//(pokemon)/pokemon-end.jsx
import React, { useState } from 'react';
import { StyleSheet, Text, View, ScrollView, TouchableOpacity } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { useTabNameContext } from '../../contexts/tabNameContext';
import { useFocusEffect } from 'expo-router';

// Définition des types en français et correspondance des indices
const TYPES = [
  "Normal", "Combat", "Vol", "Poison", "Sol",
  "Roche", "Insecte", "Spectre", "Acier", "Feu",
  "Eau", "Plante", "Électrik", "Psy", "Glace",
  "Dragon", "Ténèbres"
];

// Couleurs officielles des types Pokémon
const TYPE_COLORS = {
  Normal: "#A8A878",
  Combat: "#C03028",
  Vol: "#A890F0",
  Poison: "#A040A0",
  Sol: "#E0C068",
  Roche: "#B8A038",
  Insecte: "#A8B820",
  Spectre: "#705898",
  Acier: "#B8B8D0",
  Feu: "#F08030",
  Eau: "#6890F0",
  Plante: "#78C850",
  Électrik: "#F8D030",
  Psy: "#F85888",
  Glace: "#98D8D8",
  Dragon: "#7038F8",
  Ténèbres: "#705848"
};

// Matrice des types FireRed (17x17) (Lignes : Attaquant, Colonnes : Défenseur)
const TYPE_MATRIX = [
  /* Nor */[1, 1, 1, 1, 1, 0.5, 1, 0, 0.5, 1, 1, 1, 1, 1, 1, 1, 1],
  /* Com */[2, 1, 0.5, 0.5, 1, 2, 0.5, 0, 2, 1, 1, 1, 1, 0.5, 2, 1, 2],
  /* Vol */[1, 2, 1, 1, 1, 0.5, 2, 1, 0.5, 1, 1, 2, 0.5, 1, 1, 1, 1],
  /* Poi */[1, 1, 1, 0.5, 0.5, 0.5, 1, 0.5, 0, 1, 1, 2, 1, 1, 1, 1, 1],
  /* Sol */[1, 1, 0, 2, 1, 2, 0.5, 1, 2, 2, 1, 0.5, 2, 1, 1, 1, 1],
  /* Roc */[1, 0.5, 2, 1, 0.5, 1, 2, 1, 0.5, 2, 1, 1, 1, 1, 2, 1, 1],
  /* Ins */[1, 0.5, 0.5, 0.5, 1, 1, 1, 0.5, 0.5, 0.5, 1, 2, 1, 2, 1, 1, 2],
  /* Spe */[0, 1, 1, 1, 1, 1, 1, 2, 0.5, 1, 1, 1, 1, 2, 1, 1, 0.5],
  /* Aci */[1, 1, 1, 1, 1, 2, 1, 1, 0.5, 0.5, 0.5, 1, 0.5, 1, 2, 1, 1],
  /* Feu */[1, 1, 1, 1, 1, 0.5, 2, 1, 2, 0.5, 0.5, 2, 1, 1, 2, 0.5, 1],
  /* Eau */[1, 1, 1, 1, 2, 2, 1, 1, 1, 2, 0.5, 0.5, 1, 1, 1, 0.5, 1],
  /* Pla */[1, 1, 0.5, 0.5, 2, 2, 0.5, 1, 0.5, 0.5, 2, 0.5, 1, 1, 1, 0.5, 1],
  /* Ele */[1, 1, 2, 1, 0, 1, 1, 1, 1, 1, 2, 0.5, 0.5, 1, 1, 0.5, 1],
  /* Psy */[1, 2, 1, 2, 1, 1, 1, 1, 0.5, 1, 1, 1, 1, 0.5, 1, 1, 0],
  /* Gla */[1, 1, 2, 1, 2, 1, 1, 1, 0.5, 0.5, 0.5, 2, 1, 1, 0.5, 2, 1],
  /* Dra */[1, 1, 1, 1, 1, 1, 1, 1, 0.5, 1, 1, 1, 1, 1, 1, 2, 1],
  /* Tén */[1, 0.5, 1, 1, 1, 1, 1, 2, 0.5, 1, 1, 1, 1, 2, 1, 1, 0.5]
];

const TypeCalculator = () => {
  const [typeAttaquant, setTypeAttaquant] = useState([])
  const [typeDefenseur, setTypeDefenseur] = useState([])
  const [isResAffiche, setIsResAffiche] = useState(false)
  const [battleResult, setBattleResult] = useState(null)
  const { setTabName } = useTabNameContext();

  

  useFocusEffect(
    React.useCallback(() => {
      // 1. On change le nom dès que l'écran devient actif 🎯
      setTabName('Pokémon Fin');

      // 2. Optionnel : Nettoyage quand on quitte l'onglet
      return () => { };
    }, [setTabName])
  );
  const handlePress = (type, liste, setListe) => {
    if (liste.includes(type)) {
      setListe(liste.filter((t) => t != type))
      setIsResAffiche(false)
    }
    else if (liste.length < 2) {
      setListe([...liste, type])
      setIsResAffiche(false)
    }

  }
  const handleCombat = () => {
    if (typeAttaquant.length < 1 || typeDefenseur.length < 1) {
      setBattleResult({ "error": "Veuillez choisir au moins 1 type attaquant et 1 défenseur !" })
      setIsResAffiche(true)
      return;
    }
    if (isResAffiche) {
      return;
    }
    let multiplicateurTotal = 1
    let msgCombat = []
    let result = ""

    typeAttaquant.forEach((typeAtt) => {
      const indexAtt = TYPES.indexOf(typeAtt)
      typeDefenseur.forEach((typeDef) => {
        const indexDef = TYPES.indexOf(typeDef)
        multiplicateurTotal = multiplicateurTotal * TYPE_MATRIX[indexAtt][indexDef]
        msgCombat.push(`• ${typeAtt} contre ${typeDef} = ${TYPE_MATRIX[indexAtt][indexDef]}x`)
      });
    });

    if (multiplicateurTotal > 1) {
      result = "Victoire de l'attaquant"
    }
    else if (multiplicateurTotal == 1) {
      result = "Égalité / Neutre"
    }
    else {
      result = "Victoire du défenseur"
    }
    setBattleResult({
      multiplicateurTotal,
      result,
      msgCombat
    })
    setIsResAffiche(true)

  }
  return (
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scroll}>
        <Text style={styles.titre}>🔥 Table des Types Pokémon 🍃</Text>
        <Text style={styles.titreType}>1. Type(s) Attaquant(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type, index) => {
            const isSelected = typeAttaquant.includes(type);
            return (
              <TouchableOpacity key={`att-${index}`} onPress={() => handlePress(type, typeAttaquant, setTypeAttaquant)} style={[styles.btnType, isSelected && { backgroundColor: TYPE_COLORS[type] }]}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <Text style={styles.titreType}>2. Type(s) Defenseur(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type, index) => {
            const isSelected = typeDefenseur.includes(type)
            return (
              <TouchableOpacity key={`def-${index}`} onPress={() => handlePress(type, typeDefenseur, setTypeDefenseur)} style={[styles.btnType, isSelected && { backgroundColor: TYPE_COLORS[type] }]}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <TouchableOpacity onPress={handleCombat} style={styles.btnCombat}>
          <Text style={styles.textCombat}>
            COMBAT ! ⚔️
          </Text>
        </TouchableOpacity>
        {isResAffiche && (
          battleResult["error"] ? (
            <View style={styles.sectionResultats}>
              <Text style={styles.errorMsg}> {battleResult["error"]}</Text>
            </View>
          )
            :
            (

              <View style={styles.sectionResultats}>
                <Text style={styles.couronne}>👑</Text>
                <Text style={styles.result}>{battleResult["result"]}</Text>
                <Text style={styles.multi}>Multiplicateur total : {battleResult["multiplicateurTotal"]}x</Text>
                <View style={styles.containerExplication}>
                  <Text style={styles.pourquoi}>Pourquoi ?</Text>
                  {battleResult["msgCombat"].map((msg, index) => (
                    <Text style={styles.explications} key={`msgCombat-${index}`}>{msg}</Text>
                  ))}
                </View>
              </View>
            )


        )
        }
      </ScrollView>
    </SafeAreaView>
  );
}

export default TypeCalculator



const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#161616',
    padding: 25,
  },
  titre: {
    color: "#ffffff",
    textAlign: "center",
    fontSize: 20,
    fontWeight: "700",
    paddingVertical: 30,
  },
  btnCombat: {
    backgroundColor: "#15ff00c7",
    paddingVertical: 15,
    borderRadius: 10
  },
  scroll: {

  },
  sectionResultats: {
    backgroundColor: "#49494981",
    borderColor: "#494949",
    borderRadius: 10,
    borderWidth: 1,
    marginVertical: 20,
    padding: 20
  },
  textCombat: {
    textAlign: "center",
    fontSize: 20,
    fontWeight: "500",
    color: "#ffffff",


  },
  couronne: {
    textAlign: "center",
    fontSize: 40
  },
  titreType: {
    color: "#918d8d",
    fontWeight: "600",

  },
  textBtnType: {
    color: "#ffffff"
  },
  containerType: {
    flexDirection: "row",
    flexWrap: "wrap",
    paddingBottom: 20,

  },
  btnType: {
    padding: 5,
    backgroundColor: "#49494981",
    borderColor: "#494949",
    borderRadius: 10,
    borderWidth: 1,
    margin: 5
  },
  errorMsg: {
    color: "#c56a6a",
    fontSize: 18
  },
  result: {
    color: "#ffe600",
    textAlign: "center",
    fontSize: 20,
    fontWeight: "700"
  },
  containerExplication: {
    backgroundColor: '#161616',
    borderRadius: 10,
    padding: 10
  },
  explications: {
    color: "#ffffff"
  },
  multi: {
    color: "#ffffff",
    fontSize: 18,
    fontWeight: "500",
    textAlign: "center",
    padding: 10
  },
  pourquoi: {
    color: "#ffffff",
    fontWeight: "600",
    padding: 5,
    fontSize: 15
  }
})
```
{{% /expand %}}

---

## 🌗 Mission Mode Sombre : Un Header Customisé

Allons plus loin ! Créons un bouton magique dans notre barre supérieure pour basculer entre le **Light Mode** et le **Dark Mode**. Pour cela, nous allons créer un `ThemeContext`, un fichier de couleurs et un header personnalisé.

### 🎨 1. Le dictionnaire de couleurs (`assets/colorsPalette.js`)

Préparez vos thèmes graphiques à un seul endroit :

```javascript
// assets/colorsPalette.js
export const colorsPalette = {
  light: {
    primary: '#3498db',
    secondary: '#2ecc71',
    accent: '#e74c3c',
    background: '#ecf0f1',
    text: '#2c3e50',
  },
  dark: {
    primary: '#C69749',
    secondary: '#735F32',
    accent: '#282A3A',
    background: 'black',
    text: 'white',
  }
};

```

---

### Création du `ThemeContext` (Code de base)


Pour que le bouton de changement de thème fonctionne, vous devez créer un fichier `contexts/themeContext.js`.



```jsx
// contexts/themeContext.js
import React, { createContext, useContext, useState } from 'react';

// 1. Création du contexte du thème
const ThemeContext = createContext();

// 2. Création du Provider associé
export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light'); // 'light' par défaut

  // Fonction pour basculer d'un mode à l'autre
  const toggleTheme = () => {
    setTheme((prevTheme) => (prevTheme === 'light' ? 'dark' : 'light'));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// 3. Hook personnalisé pour consommer le thème facilement
export const useTheme = () => {
  return useContext(ThemeContext);
};

```

---

### 🧱 2. Création du composant `customDrawerHeader.jsx`

Ce composant va afficher le bouton menu (hamburger), le titre de l'application et la petite lune/soleil cliquable !

```jsx
import React from 'react';
import { Text, StyleSheet, TouchableOpacity, View } from 'react-native';
import { useTheme } from '../contexts/themeContext';
import { useSafeAreaInsets } from 'react-native-safe-area-context'; // 1. On importe le hook
import { colorsPalette } from '../assets/colorsPalette';
import Icon from 'react-native-vector-icons/FontAwesome';

const CustomDrawerHeader = ({ navigation }) => {
  const { theme, toggleTheme } = useTheme();
  const colors = colorsPalette[theme];
  const insets = useSafeAreaInsets();
  const borderColor = theme === 'light' ? '#e2e8f0' : '#2a2a2a';

  return (
    <View 
      style={[
        styles.header, 
        { 
            backgroundColor: colors.background, 
            borderBottomColor: borderColor,
            paddingTop: insets.top,
            paddingBottom: 15
        }
      ]}
    >
      <TouchableOpacity 
        style={styles.iconButton} 
        onPress={() => navigation.openDrawer()}
        activeOpacity={0.7}
      >
        <Icon name="bars" size={18} color={colors.text} />
      </TouchableOpacity>
      
      <Text style={[styles.title, { color: colors.text }]} numberOfLines={1}>
        App Title
      </Text>
      
      <TouchableOpacity 
        style={styles.iconButton} 
        onPress={toggleTheme}
        activeOpacity={0.7}
      >
        <Icon name={theme === 'light' ? "moon-o" : "sun-o"} size={18} color={colors.text} />
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between',
    paddingHorizontal: 12,
    paddingVertical: 6, 
    borderBottomWidth: 1,
  },
  title: {
    flex: 1,
    fontSize: 16, // More compact text
    fontWeight: '600',
    textAlign: 'center',
    marginHorizontal: 8,
  },
  iconButton: {
    height: 32, // Smaller touch footprint
    width: 32,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default CustomDrawerHeader;  

```

---

### 🔌 3. Injection du Header et du ThemeProvider dans votre Layout

Pour appliquer ce superbe en-tête et activer le mode sombre globalement, enveloppez votre application avec le `ThemeProvider` et configurez l'option `header` dans les `screenOptions` de votre `<Drawer>` :

```jsx
import { Drawer } from 'expo-router/drawer';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import "../global.css"
import { StatusBar } from 'expo-status-bar';
import { TabNameProvider, useTabNameContext } from '../contexts/tabNameContext';
import { ThemeProvider, useTheme } from '../contexts/themeContext';
import CustomDrawerHeader from '../components/customDrawerHeader';
import { colorsPalette } from '../assets/colorsPalette';
import { SafeAreaView } from 'react-native-safe-area-context';

const RootLayout = () => {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <ThemeProvider>
        <TabNameProvider>
          <Layout />
        </TabNameProvider>
      </ThemeProvider>
    </GestureHandlerRootView>
  );
};

export default RootLayout;

const Layout = () => {
  const { tabName } = useTabNameContext();
  const { theme } = useTheme();
  const colors = colorsPalette[theme];

  return (
    <SafeAreaView style={{flex:1, backgroundColor: colors.background}}>
      <StatusBar style={theme === 'light' ? 'dark' : 'light'} />
      <Drawer
        screenOptions={({ navigation }) => ({
          drawerActiveTintColor: colors.primary,
          drawerInactiveTintColor: colors.text,
          drawerStyle: {
            backgroundColor: colors.background,
          },
          header: () => <CustomDrawerHeader navigation={navigation} />,
          sceneContainerStyle: {
            backgroundColor: colors.background, // S'assure que l'écran prend la bonne couleur de fond
          },
        })}
      >
        <Drawer.Screen
          name="index"
          options={{
            drawerLabel: 'Accueil',
            title: 'Bienvenue',
          }}
        />
        <Drawer.Screen
          name="(pokemon)"
          options={{
            drawerLabel: 'Pokemon',
            title: tabName,
          }}
        />
      </Drawer>
    </SafeAreaView>
  );
};

```

---

### 🎨 4. Appliquer les couleurs dynamiques dans vos écrans 

{{% notice style="exo" %}}
C'est à vous de jouer ! Pour terminer, assurez-vous d'utiliser votre `useTheme` et `colorsPalette` à l'intérieur de vos écrans  pour que toute l'application adapte ses couleurs instantanément lorsque l'utilisateur clique sur le bouton lune/soleil 🎨✨.

Votre app doit avoir la page de la calculatrice et les 2 pages de pokémons. Les pages de pokémons doivent conserver leur tab, mais il doit s'adapter au thème en place.
{{% /notice %}}
