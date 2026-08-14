+++
date = '2026-06-25T16:02:29-04:00'
draft = false
weight = 8
pre="8. "
title = 'Context'
+++


Le **Context API**, c'est comme installer un réseau Wi-Fi dans votre application 📶. Au lieu de faire passer des données manuellement de composant en composant (le *props drilling* 😵), vous créez une zone de partage où n'importe quel composant peut se connecter instantanément pour récupérer ou modifier des informations !

---

## 🗂️ Problématique : La navigation imbriquée

Ajoutez un **Drawer** (menu latéral).

Dans ce **Drawer** il y aura 2 onglets. 

Le premier contiendra une page bidon appelé APis que nous utiliserons dans le prochain chapitre. 

Le deuxième contiendra la solution de l'exercice de la section navigation. Désactivez le header des onglets pour éviter d'avoir deux barres en haut. 

**Le problème ?** Le titre du header principal (celui du Drawer) reste figé et ne sait pas sur quel onglet du tab l'utilisateur se trouve. 

💡 **La solution :** Utiliser un Context pour mettre à jour le titre du Drawer en temps réel depuis les onglets !

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
import { TabNameProvider, useTabNameContext } from '../components/tabNameContext';

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

On utilise `useFocusEffect` pour changer le titre dès que l'étudiant clique sur l'onglet !

```jsx
import React from 'react';
import { useFocusEffect } from 'expo-router';
import { useTabNameContext } from '../components/tabNameContext';

// À mettre à l'intérieur de votre composant de page :
const { setTabName } = useTabNameContext();

useFocusEffect(
  React.useCallback(() => {
    // 1. On change le nom dès que l'écran devient actif 🎯
    setTabName('Nom personnalisé de la Tab');

    // 2. Optionnel : Nettoyage quand on quitte l'onglet
    return () => {};
  }, [setTabName])
);

```

---

## 🌗 Mission Mode Sombre : Un Header Customisé

Allons plus loin ! Créons un bouton magique dans notre barre supérieure pour basculer entre le **Light Mode** et le **Dark Mode**. Pour cela, on va remplacer le header par défaut par un composant personnalisé.

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

### 🧱 2. Création du composant `customDrawerHeader.js`

Ce composant va afficher le bouton menu (hamburger), le titre de l'application et la petite lune/soleil cliquable !

```jsx
import React from 'react';
import { View, Text, StyleSheet, Switch, TouchableOpacity} from 'react-native';
import { useTheme } from './contexts/ThemeContext';
import { SafeAreaView } from 'react-native-safe-area-context';
import { colorsPalette } from '../assets/colorsPalette';
import Icon from 'react-native-vector-icons/FontAwesome';


const CustomDrawerHeader = ({navigation}) => {
  const { theme, toggleTheme} = useTheme();
  const colors = colorsPalette[theme];
  return (
    <SafeAreaView style={[styles.header,{backgroundColor:colors.background}]}>
        
        <TouchableOpacity style={[styles.content]} onPress={() => {navigation.openDrawer();}}>
        <Text>
            <Icon name="bars" size={30} color={colors.text}/>
        </Text>
      </TouchableOpacity>
      
      <Text style={[styles.title,{color:colors.text}]}>App Title</Text>
      <TouchableOpacity style={[styles.content]} onPress={() => {toggleTheme()}}>
        <Text>
            <Icon name={theme == 'light' ? "moon-o" : "sun-o"} size={30} color={colors.text}/>
        </Text>
      </TouchableOpacity>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    width:'100%',
    justifyContent:'space-between'
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
  },
  content:{
    heigth:56,
    width:56,
    justifyContent:"center",
    alignItems:"center",
    fontSize:30
  }
});

export default CustomDrawerHeader;   

```

### 🔌 3. Injection du Header dans votre Drawer

Pour appliquer ce superbe en-tête à votre application, modifiez l'option `header` dans les `screenOptions` de votre composant `<Drawer>` :

```jsx
<Drawer screenOptions={
                        {
                          header: ({navigation}) => <CustomDrawerHeader navigation={navigation} toggleTheme={toggleTheme} />
                        }
                      }
>
</Drawer>

```
{{% notice style="exo" %}}

C'est à vous de jouer ! Assurez-vous d'implémenter correctement votre `ThemeContext` pour stocker la valeur `'light'` ou `'dark'`, et lisez les couleurs de votre fichier `colorsPalette.js` dans le reste de vos écrans pour que toute l'application change de look d'un seul coup ! 🎨✨
{{% /notice %}}

