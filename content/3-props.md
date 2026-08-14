+++
date = '2026-06-25T15:15:19-04:00'
draft = false
pre="3. "
title = 'Map & Props'
weight=3
+++


```jsx
import React from 'react';
import { Text, View, StyleSheet } from 'react-native';

const Index = () => {
  const names = ['Alice', 'Bob', 'Charlie', 'Guy'];

  return (
    <View style={styles.container}>
      {names.map((name, index) => (
        <Text key={index} style={styles.greeting}>
          Hello, {name}!
        </Text>
      ))}
    </View>
  );
};
export default Index;

// Styles
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  greeting: {
    fontSize: 18,
    margin: 5,
  },
});
```
En JSX (la syntaxe utilisée par React et React Native), les accolades `{}` ont un rôle très précis : **elles indiquent au compilateur qu'il faut sortir du monde HTML/JSX pour exécuter du code JavaScript classique.**

Sans ces accolades, React traiterait votre code comme du simple texte brut.

Voici exactement pourquoi elles sont indispensables dans votre exemple :

## 1. Évaluer du code JavaScript dans le rendu
Lorsque vous écrivez du JSX, vous décrivez une structure visuelle. Si vous écriviez directement `names.map(...)` au milieu de votre `<View>`, React afficherait littéralement le texte `"names.map(...)"` sur l'écran de votre téléphone.

Les accolades forcent React à **évaluer** l'expression JavaScript :
* Elles exécutent la fonction `.map()`.
* Elles génèrent le tableau de composants `<Text>`.
* Elles injectent ensuite ce tableau de composants dans la `<View>`.

## 2. Le double rôle des accolades dans votre code
Si vous regardez attentivement votre composant, vous utilisez les accolades à deux endroits différents pour cette même raison :

1. **Autour du `.map()` :** Pour dire à React *"Exécute cette fonction JavaScript pour boucler sur mon tableau"*.
2. **Autour de `{name}` et `{index}` :** À l'intérieur du composant `<Text>`, les accolades permettent d'insérer la valeur de la variable JavaScript `name` au milieu du texte "Hello, !", et la variable `index` dans la prop `key`.

---

### Un parallèle simple pour comprendre
Imaginez que le JSX est une lettre que vous écrivez en français. Les accolades sont des zones "à remplir" dynamiquement :

```jsx
// En bon français :
<Text>Hello, {name}!</Text>

// Ce que React comprend :
// "Écris 'Hello, ', va chercher la variable JavaScript 'name', puis écris '!'"
```

# Props

Les props permettent de passez en argument des valeurs à nos composants comme ceci :

```jsx
import React from 'react';
import { Text, View, StyleSheet } from 'react-native';

// 1. Reusable Child Component
const Greeting = (props) => {
  return (
    <View style={styles.greeting}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
};

// 2. Main Parent Component
const LotsOfGreetings = () => {
  return (
    <View style={styles.container}>
      <Greeting name="Rexxar" />
      <Greeting name="Jaina" />
      <Greeting name="Valeera" />
    </View>
  );
};

// 3. Clean Export
export default LotsOfGreetings;

// 4. Proper Stylesheet (instead of inline styles)
// Styles
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  greeting: {
    fontSize: 18,
    margin: 5,
  },
});

```

Une méthode encore meilleure serait d'utiliser `map` :

```jsx
import React from 'react';
import { Text, View, StyleSheet } from 'react-native';


// Define a component that takes an array of names as a prop
const GreetingList = ({ names }) => {
  return (
    <View style={styles.container}>
      {names.map((name, index) => (
        <Text key={index} style={styles.greeting}>
          Hello, {name}!
        </Text>
      ))}
    </View>
  );
};

// Parent component that uses GreetingList
const Index = () => {
  // Array of names to pass as a prop
  const names = ['Alice', 'Bob', 'Charlie', 'Guy'];

  return (
    <GreetingList names={names} />
  );
};
export default Index;

// Styles
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    backgroundColor: '#f5f5f5',
  },
  greeting: {
    fontSize: 18,
    margin: 5,
  },
});

```

{{% notice style="exo" %}}
Créez un composant pour afficher des cartes de chien dans l'exercice d'introduction à React Native. Utilisez la méthode `map` pour passer en paramètre les noms des images à afficher.

Vous rencontrerez un problème lié à l'importation dynamique des images. Pour résoudre ce problème, utilisez les ressources disponibles sur Internet pour trouver la solution appropriée.


{{% expand title="Afficher la Solution" %}}
```jsx
import React from 'react';
import { Text, View, StyleSheet, ScrollView, Image } from 'react-native';


const HScrollViewImageName = ({paths, names}) => {
  return (
    <ScrollView 
                horizontal
                contentContainerStyle={{flexDirection:'row'}}
                style={styles.scroll}
                >
                {names.map((name, index) => (
                  
                  <View style={styles.container} key={index}>
                        <Image 
                            style={styles.image}
                            source={paths[index]} 
                            />
                        <Text style={styles.text}>name</Text>
                    </View>
                ))}
            </ScrollView>
        
    );
};

const Index = () => {
  // Array of names to pass as a prop
  const names = ['Golden', 'Corgie', 'Rottweiler'];
  const paths = [require('../assets/chiot1.jpg'), require('../assets/chiot2.jpg'), require('../assets/chiot3.jpg')];

  return (
    <HScrollViewImageName names={names} paths={paths} />
  );
};
export default Index;


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