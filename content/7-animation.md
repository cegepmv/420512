+++
date = '2026-06-25T15:22:35-04:00'
draft = false
weight = 7
pre = "7. "
title = 'Animations'
+++

Dans cette section, nous verrons comment dynamiser l'interface utilisateur grâce aux animations. En React Native, la gestion des animations repose sur une combinaison entre la **gestion des états (`state`)** pour contrôler les données à afficher, et la bibliothèque **`react-native-reanimated`** qui permet d'exécuter des transitions fluides et performantes directement sur le thread natif.


### Installation 

[Animation](https://docs.swmansion.com/react-native-reanimated/docs/fundamentals/getting-started)

[Gesture handler](https://docs.swmansion.com/react-native-gesture-handler/docs/2.x/fundamentals/installation)



### Exemple avec longpress


```jsx
import { View, StyleSheet } from 'react-native';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

export default function App() {
  const longPressGesture = Gesture.LongPress().onEnd((e, success) => {
    if (success) {
      console.log(`Long pressed for ${e.duration} ms!`);
    }
  });

  return (
    <GestureDetector gesture={longPressGesture}>
      <View style={styles.box} />
    </GestureDetector>
  );
}

const styles = StyleSheet.create({
  box: {
    height: 120,
    width: 120,
    backgroundColor: '#b58df1',
    borderRadius: 20,
    marginBottom: 30,
  },
});
```

```jsx
import { View, StyleSheet } from 'react-native';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

export default function App() {
  const longPressGesture = Gesture.LongPress().onEnd((e, success) => {
    if (success) {
      console.log(`Long pressed for ${JSON.stringify(e)} ms!`);
    }
  });

  return (
    <GestureDetector gesture={longPressGesture}>
      <View style={styles.box} />
    </GestureDetector>
  );
}

const styles = StyleSheet.create({
  box: {
    height: 120,
    width: 120,
    backgroundColor: '#b58df1',
    borderRadius: 20,
    marginBottom: 30,
  },
});
```

### withTiming

[Doc transforms](https://reactnative.dev/docs/transforms)

```jsx
import React from 'react';
import { StyleSheet, View, useWindowDimensions } from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withRepeat,
} from 'react-native-reanimated';

export default function App() {
  const { width } = useWindowDimensions();
  
  // Calculate a safe range for the box to move back and forth
  const initialOffset = width / 4;
  const offset = useSharedValue(initialOffset);

  const animatedStyles = useAnimatedStyle(() => ({
    transform: [{ translateX: offset.value }],
  }));

  React.useEffect(() => {
    // Animate to the negative counterpart and repeat infinitely with reverse (true)
    offset.value = withRepeat(
      withTiming(-initialOffset, { duration: 1750 }),
      -1,
      true
    );
  }, [initialOffset]);

  return (
    <View style={styles.container}>
      <Animated.View style={[styles.box, animatedStyles]} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  box: {
    height: 120,
    width: 120,
    backgroundColor: '#b58df1',
    borderRadius: 20,
  },
});
```
---

### 1. Gestion des Gestes (`Gesture.LongPress`)

Dans ces deux exemples, tu définis un comportement de appui long (`LongPress`).

```javascript
const longPressGesture = Gesture.LongPress().onEnd((e, success) => {
  if (success) {
    console.log(`Long pressed for ${e.duration} ms!`);
  }
});

```

* **`Gesture.LongPress()`** : C'est un *builder*. Il crée une configuration de geste. Contrairement aux événements React classiques (`onResponderGrant`, etc.), celui-ci gère le cycle de vie complet du geste en natif.
* **`.onEnd((e, success) => ...)`** : C'est un callback déclenché lorsque l'utilisateur relève son doigt.
* `success` : Un booléen qui indique si le long press a bien atteint sa durée requise (pour éviter les faux positifs si l'utilisateur glisse son doigt trop vite).
* `e` (Event) : Contient les métadonnées (durée, coordonnées X/Y, etc.). Dans ton deuxième snippet, `JSON.stringify(e)` te permet de logger l'objet brut pour explorer toutes les propriétés disponibles.


* **`<GestureDetector gesture="{longPressGesture}">`** : C'est le composant conteneur (le "wrapper"). Il écoute les touches sur l'élément enfant (`View`) et attache le geste de manière déclarative, un peu comme un `onClick`, mais géré nativement.

---

### 2. Le coeur de Reanimated : `useSharedValue`

Passons à l'animation avec `withTiming`. C'est ici que la différence avec React se fait sentir.

```javascript
const offset = useSharedValue(initialOffset);

```

* **`useSharedValue` vs `useState`** : En React, si tu modifies un state pour animer une valeur, tu déclenches un re-render complet du composant à chaque frame (ex: 60 fois par seconde), ce qui fait ramer l'application.
* `useSharedValue` ressemble plutôt à un **`useRef`** : modifier sa valeur (`offset.value = ...`) **ne déclenche aucun re-render React**. La valeur vit entièrement sur le thread natif UI, ce qui garantit une fluidité totale.

---

### 3. Lier la logique au style : `useAnimatedStyle`

```javascript
const animatedStyles = useAnimatedStyle(() => ({
  transform: [{ translateX: offset.value }],
}));

```

* **`useAnimatedStyle`** : C'est le pont entre ta `sharedValue` et le style de ton composant.
* La fonction que tu passes à l'intérieur est un **worklet** (du code JavaScript qui s'exécute directement sur le thread UI natif). Elle s'abonne aux changements de `offset.value` et met à jour la propriété `transform` à chaque frame de manière ultra-rapide, sans repasser par le bridge React Native.
* Pour l'utiliser dans le JSX, tu l'passes simplement dans le tableau de styles de ton `Animated.View` : `style={[styles.box, animatedStyles]}`.

---

### 4. Piloter l'animation : `withTiming` et `withRepeat`

```javascript
React.useEffect(() => {
  offset.value = withRepeat(
    withTiming(-initialOffset, { duration: 1750 }),
    -1,
    true
  );
}, [initialOffset]);

```

* **`React.useEffect`** : Utilisé ici uniquement pour lancer l'animation au montage (ou lorsque `initialOffset` change). Tu modifies directement `offset.value`, ce qui déclenche l'animation.
* **`withTiming(targetValue, options)`** : Indique à Reanimated comment passer de la valeur actuelle à `targetValue` de manière fluide (avec une transition progressive sur une durée de 1750 ms).
* **`withRepeat(animation, numberOfReps, reverse)`** :
* Il enveloppe ton `withTiming`.
* Le premier argument est l'animation à répéter.
* `-1` signifie **répétition infinie** (comme une boucle).
* `true` signifie **reverse (ping-pong)** : l'élément va vers `-initialOffset`, puis revient vers sa position initiale en sens inverse, en boucle infinie.

{{% notice style="exo"%}}
# Exercice 1

Faites un programme qui consiste en un bouton qui change de couleur après un longpress. Le changement de couleur doit être progressif sur 2 secondes.


{{% expand title="Solution 1"%}}
## Minimum

```jsx
import React from 'react';
import { StyleSheet, View, useWindowDimensions } from 'react-native';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
} from 'react-native-reanimated';

export default function App() {
  const { width } = useWindowDimensions();
  
  // // Calculate a safe range for the box to move back and forth
  // const initialOffset = width / 4;
  // const offset = useSharedValue(initialOffset);
  const isPressed = useSharedValue(false)

  const longPressGesture = Gesture.LongPress().onEnd((e, success) => {
    isPressed.value = !isPressed.value
  });

  const animatedStyles = useAnimatedStyle(() => ({
      backgroundColor: withTiming(isPressed.value ? '#FF5252' : '#6200EE', {
        duration: 200,
      }),
    }
  )
  )



  return (
    <GestureDetector gesture={longPressGesture}>
      <Animated.View style={[styles.box, animatedStyles]} />
    </GestureDetector>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  box: {
    height: 120,
    width: 120,
    backgroundColor: '#b58df1',
    borderRadius: 20,
  },
});
```

## Plus sophistiqué

```jsx
import React from 'react';
import { StyleSheet, Text, View } from 'react-native';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, { 
  useSharedValue, 
  useAnimatedStyle, 
  withTiming 
} from 'react-native-reanimated';

export default function LongPressButton() {
  // Shared value to track press state on the UI thread
  const isPressed = useSharedValue(false);

  // Define the long press gesture
  const longPressGesture = Gesture.LongPress()
    .minDuration(500) // Duration in milliseconds (0.5 seconds)
    .onStart(() => {
      isPressed.value = true;
    })
    .onEnd(() => {
      isPressed.value = false;
    })
    .onFinalize(() => {
      isPressed.value = false;
    });

  // Create smooth animated styles for color change
  const animatedStyle = useAnimatedStyle(() => {
    return {
      backgroundColor: withTiming(isPressed.value ? '#FF5252' : '#6200EE', {
        duration: 200,
      }),
    };
  });

  return (
    <View style={styles.container}>
      <GestureDetector gesture={longPressGesture}>
        <Animated.View style={[styles.button, animatedStyle]}>
          <Text style={styles.text}>Hold to Change Color</Text>
        </Animated.View>
      </GestureDetector>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f5f5f5',
  },
  button: {
    paddingVertical: 16,
    paddingHorizontal: 32,
    borderRadius: 12,
    alignItems: 'center',
    justifyContent: 'center',
    elevation: 3, // Android shadow
    shadowColor: '#000', // iOS shadow
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: { ios: 0.25 },
    shadowRadius: 3.84,
  },
  text: {
    color: '#FFFFFF',
    fontSize: 16,
    fontWeight: '600',
  },
});
```
{{% /expand %}}
{{% /notice %}}

---


[Flatlist](https://reactnative.dev/docs/flatlist)  

---

### 1. Le rendu des éléments (La stratégie de chargement)

* **`ScrollView` (Le bourrin) :**
Il rend **tous** ses enfants d'un seul coup, dès le montage du composant, qu'ils soient affichés à l'écran ou non.
* *Conséquence :* Si tu as 500 éléments dans une liste, React Native va instancier 500 composants d'un coup. Résultat : gros pic de CPU, saccades (*dropped frames*) au montage, et consommation excessive de mémoire.


* **`FlatList` (L'intelligent) :**
Il utilise une stratégie de **virtualisation**. Il ne rend à l'écran que les éléments qui sont actuellement visibles (plus une petite marge de sécurité appelée *windowing*). Au fur et à mesure que l'utilisateur scroll, les éléments qui sortent de l'écran sont détruits ou recyclés pour afficher les nouveaux.
* *Conséquence :* Performances constantes, peu importe que ta liste contienne 10 ou 100 000 éléments.



---

### 2. Comment on les utilise (L'API)

* **`ScrollView` :**
Tu lui passes des enfants de manière classique, comme une `View` :
```jsx
<ScrollView>
  {items.map(item => <Item key={item.id} data={item} />)}
</ScrollView>

```


* **`FlatList` :**
Il est conçu spécifiquement pour des listes de données. Tu lui passes un tableau via la prop `data` et une fonction de rendu via `renderItem` :
```jsx
<FlatList
  data={items}
  keyExtractor={item => item.id}
  renderItem={({ item }) => <Item data={item} />}
/>

```



---

### 3. Tableau comparatif synthétique

| Critère | `ScrollView` | `FlatList` |
| --- | --- | --- |
| **Cas d'usage idéal** | Petits contenus statiques (ex: un formulaire avec quelques inputs, une page de paramètres). | Listes dynamiques, grandes ou infinies (ex: feed de réseaux sociaux, catalogue de produits). |
| **Performance sur grande liste** | Très mauvaise (risque de crash / lag sévère). | Excellente (grâce au recyclage des composants). |
| **Props principales** | `contentContainerStyle`, `onScroll` | `data`, `renderItem`, `keyExtractor`, `onEndReached` (pour la pagination). |

---




{{% notice tip "Tableau dans un state en React" %}}
En React, il ne faut jamais modifier directement un tableau existant dans l'état (comme faire data.push(nouvelleTache)), car React ne détectera pas le changement et ne rafraîchira pas l'écran.

On utilise l'opérateur de décomposition (...) pour créer une nouvelle copie du tableau avec l'élément ajouté :

```JavaScript


// ❌ À NE PAS FAIRE (Mutation directe)
data.push(nouvelleTache);
setData(data); // React ne détecte pas la modification !

// ✅ BONNE PRATIQUE (Immutabilité avec Spread)
const nouveauTableau = [...data, nouvelleTache]; // Copie l'ancien tableau et ajoute le nouvel élément à la fin
setData(nouveauTableau);

// ✅ ENCORE MIEUX : Mise à jour fonctionnelle
setData((prevData) => [...prevData, nouvelleTache]);
```
{{% /notice %}}

{{% notice style="exo"%}}
## Exercice

À l'aide des valeurs de couleur ci-dessous :

```jsx
const color1 = "#000000";
const color2 = "#282A3A";
const color3 = '#735F32';
const color4 = '#C69749';

```

Recréez l'affichage suivant : 

![Exo_anim](/420512/images/animation.png)


1. Faites en sorte que le bouton **+** ajoute une tâche. *Indice : utiliser le spread operator `...` et une mise à jour fonctionnelle d'état (Ex : `setCount((prev) => prev + 1)`)*.
1. Faites en sorte qu'un swipe à droite sur une tâche entraîne sa suppression. **Indice : `Swipeable`**.

---

{{% expand title="Afficher la Solution" %}}

### Layout

```jsx
import { View, Text } from 'react-native'
import React from 'react'
import {Stack} from 'expo-router'
import { GestureHandlerRootView } from 'react-native-gesture-handler'

const RootLayout = () => {
    return (
    <GestureHandlerRootView>
        <Stack>
            <Stack.Screen name="index" options={{headerShown: false}}/>
        </Stack>
    </GestureHandlerRootView>
    )
}

export default RootLayout

```

### Index
To be added
```jsx


```

{{% /expand %}}



{{% /notice %}}


<!-- import { StyleSheet, Text, View, TextInput, TouchableOpacity, Dimensions, FlatList, Keyboard} from 'react-native'
import React, {useState} from 'react'
import { SafeAreaView } from 'react-native-safe-area-context';
import  { Swipeable } from 'react-native-gesture-handler'
import Animated, {LinearTransition , Easing} from 'react-native-reanimated';

const color1 = "#000000";
const color2 = "#282A3A";
const color3 = '#735F32';
const color4 = '#C69749';

const index = () => {
    const [textInput,setTextInput] = useState('')
    const [data,setData] = useState(["Pratiquer mon lancer de frisbee", "Me questionner sur la vie", "Corriger les examens"])

    const handlePressPlus = () => {
        setData((prev) => [...prev,textInput])
        setTextInput('')
        Keyboard.dismiss()
    }
    
    const handleDelete = (index) => {
        setData((prev) => prev.filter((_,id) => id !== index))
    }
    const RenderItem = ({index, item}) => {
        const afficheText = () =>(
                <View style={{width:150,height:10}}></View>
            );
        
        return(
            <Swipeable renderLeftActions={afficheText} onSwipeableWillOpen={() => handleDelete(index)}>
                <View style={[styles.task, { width: Dimensions.get('window').width }]}>
                    <Text style={styles.taskText}>{item}</Text>
                </View>
            </Swipeable>
        );
    }

    return (
        <SafeAreaView style={styles.container}>
            <View style={{flexDirection:"row"}}>
                <TextInput
                    style={styles.textInput}
                    onChangeText={setTextInput}
                    placeholder='Entrez la tâche à accomplir'
                    placeholderTextColor={color3}
                    value={textInput}
                />
                <TouchableOpacity onPress={handlePressPlus} style={styles.btnPlusContainer}>
                    <Text style={styles.btnPlus}>+</Text>
                </TouchableOpacity>
            </View>
            <Animated.FlatList
                style={styles.flatList}
                data={data}
                renderItem={({item,index}) => <RenderItem index={index} item={item}/>}
                keyExtractor={(item, index) => index.toString()}
                contentContainerStyle={{alignItems:'center'}}
            />
        </SafeAreaView>
    )
}

export default index

const styles = StyleSheet.create({
    container:{
        backgroundColor:color1,
        flex:1,
    },
    textInput:{
        backgroundColor:color2,
        width:Dimensions.get('window').width - 56,
        height:56,
        paddingHorizontal:20,
        textAlign:'center',
        fontSize:16,
        color:color4,
    },
    btnPlus:{
        color:color1,
        fontSize:26
    },
    btnPlusContainer:{
        backgroundColor:color4,
        height:56,
        width:56,
        justifyContent:"center",
        alignItems:"center"
    },
    flatList:{
        paddingVertical:30
    },
    task:{
        paddingVertical:20,
        paddingHorizontal:10,
        marginVertical:5,
        backgroundColor:color2,
        width:Dimensions.get('window').width * 0.9,
    },
    taskText:{
        color:color4,
    },
    minusBtn:{
        color:color1,
        fontSize:26,
    },
    btnMinusContainer:{
        justifyContent:'center',
        alignItems:'center',
        backgroundColor:'red',
        width:56,
        height:56,
        marginVertical:5,
    }
}) -->