+++
date = '2026-06-25T14:43:31-04:00'
draft = false
title = 'Composants de bases'
pre = "2. "
weight = "2"
+++


## Style

Vous pouvez écrire des styles directement comme ceci :

```jsx
import { StyleSheet, Text, View } from 'react-native'
import React from 'react'

const RootLayout = () => {
  return (
    <View style={{flex: 1,justifyContent:'center'}}>
      <Text>RootLayout</Text>
    </View>
  )
}

export default RootLayout

const styles = StyleSheet.create({})

```

```jsx
import { StyleSheet, Text, View } from 'react-native'
import React from 'react'

const RootLayout = () => {
  return (
    <View style={styles.container}>
      <Text>RootLayout</Text>
    </View>
  )
}

export default RootLayout

const styles = StyleSheet.create({
    container:{
        display: 'flex',
        flex: 1,
        alignItems:'center',
        justifyContent: 'center'
    }
})

```

### `const styles = StyleSheet.create({ ... })`

Cette ligne crée un objet de styles à l'aide de la méthode `create` de `StyleSheet`. Cela permet de définir des styles que vous pouvez appliquer à vos composants React Native.

* **`container: { ... }`** : Ici, `container` est un nom de style que vous pouvez utiliser pour styliser un composant. Le nom est choisi librement et peut être utilisé comme une clé pour accéder à ces styles.
* **`display: 'flex'`** : La propriété `display: 'flex'` est utilisée pour spécifier que le conteneur utilise le modèle de mise en page Flexbox. En Flexbox, les éléments enfants sont disposés de manière flexible selon les propriétés de style définies.
* **`flex: 1`** : La propriété `flex: 1` indique que le conteneur doit occuper tout l'espace disponible dans son parent. Cela signifie que le conteneur se dilatera pour remplir l'espace restant.
* **`alignItems: 'center'`** : La propriété `alignItems: 'center'` aligne les éléments enfants du conteneur au centre de l'axe transversal (l'axe perpendiculaire à la direction principale du Flexbox). Pour un conteneur en colonne (axe principal vertical), cela aligne les éléments horizontalement au centre.
* **`justifyContent: 'center'`** : La propriété `justifyContent: 'center'` aligne les éléments enfants du conteneur au centre de l'axe principal (l'axe dans la direction de la flexibilité). Pour un conteneur en colonne, cela aligne les éléments verticalement au centre.

Pour plus de style, fiez-vous à la [documentation d'Expo](https://docs.expo.dev/develop/user-interface/color-themes/), la [documentation de React Native](https://reactnative.dev/docs/style), mais aussi à [Gemeni](https://gemini.google.com/app?hl=fr).

<!-- ::: note
*Note pour plus tard : parler de Nativewind un jour.*
::: -->

---

## Stack

En ce moment, c'est uniquement `_layout.jsx` qui est affiché. Nous voulons incorporer à cette page `index.js`. Nous allons utiliser les `Stack` de `expo-router`.

```jsx
import { StyleSheet, Text, View } from 'react-native'
import { Stack } from 'expo-router'

const RootLayout = () => {
  return (
    // Il doit y avoir 1 seul élément retourné, 
    // on peut mettre des balises vides qui entourent le reste du code.
    <>
        <Text>Hello from layout</Text>
        <Stack>
            <Stack.Screen name="index" options={{headerShown: false}} >
        </Stack>
    </>
  )
}

export default RootLayout

```

---

## SafeAreaView

Nous allons régler le fait que le layout sort possiblement de votre écran tout dépendant de votre téléphone.

Il va falloir utiliser `SafeAreaView` en installant le paquet suivant :

```bash
npx expo install react-native-safe-area-context 

```

Dans `_layout.jsx`, remplacez `View` par `SafeAreaView` et observez la différence en haut de l'écran :

```jsx
import { StyleSheet, Text, View } from 'react-native'
import { Stack } from 'expo-router'
import { SafeAreaView } from "react-native-safe-area-context";

const RootLayout = () => {
  return (
    <>
        <SafeAreaView>
            <Text>Hello from layout</Text>
        </SafeAreaView>
        <Stack>
            <Stack.Screen name="index" options={{headerShown: false}} /&gt;
        </Stack>
    </>
  )
}

export default RootLayout

```

---

## [Core Components](https://reactnative.dev/docs/components-and-apis)

* **View** : Conteneur principal pour le layout et le style. Utilisé pour organiser et envelopper d'autres composants. Crée des mises en page flexibles et gère les dispositions des éléments enfants.
* **Text** : Affiche du texte à l'écran. Permet de personnaliser le texte avec différentes propriétés de style.
* **Image** : Affiche des images dans l'application. Supporte différents formats d'images et peut gérer les images locales ou distantes. Peut être redimensionnée et stylisée.
* **ScrollView** : Permet le défilement du contenu lorsque celui-ci dépasse la taille visible de l'écran. Utile pour les longues listes ou contenus volumineux.
* **TextInput** : Permet à l'utilisateur de saisir du texte. Supporte différents types de clavier et comportements de saisie. Utilisé pour les champs de formulaire, les barres de recherche, ou tout autre endroit nécessitant une entrée de texte.

---

## Structure de projet
Une bonne structure et hiérarchie dans un projet permet d'améliorer la qualité, la maintenabilité, et l'évolutivité du code, tout en facilitant la collaboration et en rendant le débogage et les tests plus simples.

Voici les quelques règles que je vous demande de suivre pour l'instant :

<div class="row pb-30">
    <div class="col-sm-4 col-12 pt-30">
        <img src="/420512/images/hierarchie.png" alt="exemple hierarchie">
    </div>
    <div class="col-sm-8 col-12 pt-30">

- **app :** pages(screen) et _layout
- **assets :** Les images sans exceptions se trouvent ici. Si vous avez des données en format json elles seront enregistrées ici. Si vous avez n'importe quoi qui n'est pas du code et qui n'est pas un fichier de configuration alors il trouvera ça place ici.
- **components :** Tous les composants qui seront exportés ailleurs

    </div>
</div>

---

{{% notice style="exo"%}}

Reproduisez une application selon le UI suivant :



<div class="row">
    <div class="col-12 col-sm-4 image-container">
        <img src="/420512/images/golden.jpg">
    </div>
    <div class="col-12 col-sm-4 image-container">
        <img src="/420512/images/corgie.jpg">
    </div>
    <div class="col-12 col-sm-4 image-container">
        <img src="/420512/images/rottweiler.jpg">
    </div>
</div>

Les images proviennents du site [unsplash](https://unsplash.com/fr/licence)

{{% expand title="Découvrir la Solution" %}}

```jsx
import { StatusBar } from 'expo-status-bar';
import { StyleSheet, Text, View, Image, ScrollView} from 'react-native';

export default function App() {
  return (
    <ScrollView 
        horizontal
        contentContainerStyle={{flexDirection:'row'}}
        style={styles.scroll}
    >
        <View style={styles.container}>
          <Image 
              style={styles.image}
              source={require('../assets/chiot1.jpg')} />
          <Text style={styles.text}>Golden</Text>
        </View>
        <View style={styles.container}>
              <Image 
                  style={styles.image}
                  source={require('../assets/chiot2.jpg')} />
              <Text style={styles.text}>Corgie?</Text>
        </View>
        <View style={styles.container}>
              <Image 
                  style={styles.image}
                  source={require('../assets/chiot3.jpg')} />
              <Text style={styles.text}>Rottweiler</Text>
        </View>
    </ScrollView>
    
    );
}

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
      justifyContent: 'center',
      padding:10,
    },
      image: {
        height:400,
      width: 300, 
      resizeMode: 'contain'
    },
    scroll:{
      flex:1,
      backgroundColor:'#fff',
    },
    text: {
      fontSize: 18,
      textAlign: 'center',
      marginTop: 10,
    },
});

```

{{% /expand %}}


{{% /notice %}}