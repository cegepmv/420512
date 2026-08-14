+++
date = '2026-06-25T15:22:35-04:00'
draft = false
weight = 6
pre = "6. "
title = 'Animations'
+++

Dans cette section, nous verrons comment dynamiser l'interface utilisateur grâce aux animations. En React Native, la gestion des animations repose sur une combinaison entre la **gestion des états (`state`)** pour contrôler les données à afficher, et la bibliothèque **`react-native-reanimated`** qui permet d'exécuter des transitions fluides et performantes directement sur le thread natif.

---

[Flatlist](https://reactnative.dev/docs/flatlist)  
[Animation](https://docs.swmansion.com/react-native-reanimated/docs/fundamentals/glossary/#shared-value)

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

*(Indice : J'utilise `Dimensions.get('window').width` pour déterminer la largeur des rectangles.)*

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

```jsx
import { StyleSheet, Text, View, TextInput, TouchableOpacity, Dimensions, FlatList, Keyboard} from 'react-native'
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
})

```

{{% /expand %}}



{{% /notice %}}
