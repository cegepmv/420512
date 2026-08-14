+++
date = '2026-06-25T15:22:35-04:00'
draft = false
weight = 7
pre = "7. "
title = 'Navigation'
+++

React Navigation est la bibliothèque standard pour gérer le changement d'écrans et la navigation dans une application React Native.

---

## 1. Les Concepts Clés de React Navigation

Avant d'utiliser le Drawer, il faut comprendre les 3 piliers de la navigation :

* **Le Navigator (Navigateur) :** Le composant parent qui contrôle l'historique et les transitions d'écrans (ex: `Drawer`, `Stack`, `Tabs`).
* **Les Screens (Écrans) :** Les composants React représentant chaque page de votre application.
* **Les Options (`screenOptions` & `options`) :** La configuration visuelle de l'en-tête (Header), des icônes, des titres et des gestes.

Dans **Expo Router**, la structure des dossiers définit automatiquement les routes de l'application (File-based Routing).

---

## 2. Le fonctionnement du Drawer Navigator

Le **Drawer** (ou menu coulissant) est un panneau caché sur le côté de l'écran qui s'ouvre par un glissement (swipe) ou en appuyant sur l'icône « hamburger » (☰).

### Composants principaux du Drawer :
1. **`GestureHandlerRootView`** : Requis par `react-native-gesture-handler` pour intercepter les gestes de glissement.
2. **`Drawer`** : Le conteneur principal configuré dans `_layout.js`.
3. **`Drawer.Screen`** : Définit la présence d'un écran dans le menu coulissant.

---

## 3. Structure minimale d'un Drawer

Voici comment s'articule le fichier layout principal (`app/_layout.js`) :

```jsx
import { Drawer } from 'expo-router/drawer';
import { GestureHandlerRootView } from 'react-native-gesture-handler';

export default function RootLayout() {
  return (
    // 1. Envelopper l'application pour gérer les gestes
    <GestureHandlerRootView style={{ flex: 1 }}>
      
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
          name="profile"
          options={{
            drawerLabel: 'Profil',
            title: 'Mon Profil',
          }}
        />
      </Drawer>

    </GestureHandlerRootView>
  );
}
```

---

## 4. Options de personnalisation essentielles

Tu peux modifier l'apparence du Drawer à deux niveaux :

### A. Options globales (`screenOptions` sur `<Drawer>`)
Applique le style à tous les écrans du Drawer :
* `drawerType`: Définit comment le tiroir s'ouvre (`front` repasse par-dessus, `slide` pousse l'écran, `permanent` reste fixe).
* `drawerPosition`: `'left'` (par défaut) ou `'right'`.
* `headerShown`: `true` ou `false` pour afficher ou masquer la barre supérieure.

### B. Options par écran (`options` sur `<Drawer.Screen>`)
Applique la configuration à un écran spécifique :
* `drawerLabel`: Le texte affiché dans le menu déroulant (s'il est différent du `title`).
* `title`: Le titre affiché dans la barre supérieure (Header).
* `drawerIcon`: Une fonction qui retourne une icône (ex: Vector Icons).

Exemple avec icône :

```jsx
import { Ionicons } from '@expo/vector-icons';

<Drawer.Screen
  name="settings"
  options={{
    drawerLabel: 'Paramètres',
    title: 'Configuration',
    drawerIcon: ({ color, size }) => (
      <Ionicons name="settings-outline" size={size} color={color} />
    ),
  }}
/>
```

---

## 5. Contrôler le Drawer par le code

Bien que le bouton "hamburger" soit créé automatiquement, tu peux ouvrir ou fermer le Drawer manuellement dans n'importe quel écran à l'aide du hook `useNavigation` ou de l'objet `navigation`.

Exemple dans un composant d'écran :

```jsx
import { View, Button } from 'react-native';
import { useNavigation } from 'expo-router';
import { DrawerActions } from '@react-navigation/native';

export default function HomeScreen() {
  const navigation = useNavigation();

  const openMenu = () => {
    // Ouvre le tiroir par le code
    navigation.dispatch(DrawerActions.openDrawer());
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Button title="Ouvrir le Drawer" onPress={openMenu} />
    </View>
  );
}
```

---

## 6. Résumé des étapes d'implémentation

1. Installe les dépendances : `@react-navigation/drawer`, `react-native-gesture-handler`, `react-native-reanimated`.
2. Ajoute `'react-native-reanimated/plugin'` dans `babel.config.js`.
3. Englobe ton composant racine dans `<GestureHandlerRootView>`.
4. Configure `<Drawer>` et ses `<Drawer.Screen>` dans `_layout.js`.
5. Ajuste le design via `screenOptions` ou les `options` individuelles.

{{% notice style="exo" %}}

Avec l'aide de la <a href="https://docs.expo.dev/router/advanced/native-tabs/">documentation</a>. Créez un drawer contenant la calculatrice et le gestionnaire de tâche. Le rendu final doit ressembler à ceci :


{{% /notice %}}