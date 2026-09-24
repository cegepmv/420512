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
import { StyleSheet, View } from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
  withRepeat,
} from 'react-native-reanimated';

export default function App({ width }) {
  const offset = useSharedValue(width / 2 - 160);

  const animatedStyles = useAnimatedStyle(() => ({
    transform: [{ translateX: offset.value }],
  }));

  React.useEffect(() => {
    offset.value = withRepeat(
      // highlight-next-line
      withTiming(-offset.value, { duration: 1750 }),
      -1,
      true
    );
  }, []);

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
    height: '100%',
  },
  box: {
    height: 120,
    width: 120,
    backgroundColor: '#b58df1',
    borderRadius: 20,
  },
});

```

{{% expand title="Solution 1"%}}
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

---

[Flatlist](https://reactnative.dev/docs/flatlist)  
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