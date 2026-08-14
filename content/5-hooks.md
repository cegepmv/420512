+++
date = '2026-06-25T15:44:32-04:00'
draft = false
weight = 5
pre = "5. "
title = 'Hooks'
+++

Bienvenue dans le monde merveilleux des **Hooks** ! Les hooks sont de petites fonctions magiques qui permettent à vos composants fonctionnels de faire tout ce que faisaient les anciennes classes, et bien plus encore.

---

## Les Incontournables

Ces deux-là, ce sont vos meilleurs amis pour débuter. Vous allez les croiser *partout* !

### 1. `useState`
Il permet d'ajouter une **mémoire locale** (un état) à un composant. Il vous donne une variable pour stocker la valeur actuelle et une fonction pour la modifier.

```javascript
const [compteur, setCompteur] = useState(0);

```

### 2. `useEffect` ⚡

Il vous permet de déclencher des **effets secondaires** (du code qui s'exécute en réaction à certains événements, comme le montage du composant ou la modification d'une variable).

```javascript
useEffect(() => {
  console.log('Composant monté ou mis à jour !');
}, [compteur]); // Se déclenche dès que `compteur` change de valeur

```

---

## Les autres Hooks

Voici un petit guide pour savoir à quoi servent tous ces autres hooks que nous croiserons au fil de notre aventure ! 

| Hook | Son super-pouvoir 🦸‍♂️ | Un petit aperçu... |
| :--- | :--- | :--- |
| **`useContext`** | Partage des données dans toute l'app sans passer par les props. | `const theme = useContext(ThemeContext);` |
| **`useCallback`** | Garde une fonction en mémoire pour éviter de la recréer inutilement. | `const handleClick = useCallback(() => {...}, [compteur]);` |
| **`useMemo`** | Garde le *résultat d'un calcul lourd* au chaud dans la mémoire. | `const valeurMemo = useMemo(() => calculLourd(c), [c]);` |
| **`useRef`** 📌 | Pointe directement vers un élément du DOM ou stocke une valeur sans relancer de rendu. | `const inputRef = useRef(null);` |
| **`useReducer`** 🕹️ | Gère des états super complexes avec une logique d'actions (façon Redux). | `const [state, dispatch] = useReducer(reducer, init);` |
| **`useLayoutEffect`** 📐 | Identique à `useEffect`, mais s'exécute de façon synchrone *avant* l'affichage à l'écran. | `useLayoutEffect(() => { /* synchrone */ }, []);` |
| **`useImperativeHandle`** 🔑 | Permet de choisir et personnaliser ce qu'un composant parent peut faire avec une `ref`. | `useImperativeHandle(ref, () => ({ focus: () => {} }));` |
| **`useDebugValue`** 🛠️ | Donne une jolie étiquette personnalisée à vos propres hooks dans les DevTools. | `useDebugValue(isOnline ? 'Online' : 'Offline');` |
---

## Zoom sur : `useState`

`useState` est le roi de la réactivité. Quand sa valeur change, React redessine automatiquement le composant à l'écran ! ✨

### syntaxe :

```javascript
const [monEtat, setMonEtat] = useState(valeurInitiale);

```

### 📱 Exemple concret (Un joli compteur) :

```javascript
import React, { useState } from 'react';
import { Text, Button, View } from 'react-native';

const Compteur = () => {
  // 0 est la valeur de départ 
  const [compteur, setCompteur] = useState(0);

  return (
    <View style={{ padding: 20, alignItems: 'center' }}>
      <Text>Vous avez cliqué {compteur} fois</Text>
      <Button
        title="Cliquez-moi !"
        onPress={() => setCompteur(compteur + 1)}
      />
    </View>
  );
};

export default Compteur;

```

**À retenir :**

* **Persistant** : L'état ne s'efface pas quand le composant se re-déclenche.
* **Indépendant** : Si vous mettez ce `<Compteur />` deux fois dans votre page, chacun aura sa propre vie !

{{% notice style="exo" title="Exo 1" %}}
<img style="max-width:45%;display:inline-block;margin-right:5%; margin-left:2%" src="/420512/images/exo1Hooks.png"/>
<img style="max-width:45%;display:inline-block;margin-right:2%" src="/420512/images/exo1Hooks2.png"/>

En vous basant sur l'exemple ci-dessus, modifiez le code pour que le bouton devienne **bleu** lorsque le compteur est **pair**, et **vert** lorsqu'il est **impair**.
{{% expand title="💡 Afficher la Solution de l'Exercice 1" %}}

#### Option 1 : Avec la magie de `useEffect` ⚡

```javascript
import { StyleSheet, Text, View } from 'react-native'
import React, { useState, useEffect } from 'react'
import ButtonAddOne from '../components/buttonAddOne';

const Test = () => {
  const [count, setCount] = useState(0);
  const [colorButton, setColorButton] = useState("blue")

  const handlePress = () => {
    setCount(count + 1);
  }

  // On surveille le "count"
  useEffect(() => {
    if (count % 2 === 0) {
      setColorButton("blue")
    } else {
      setColorButton("green")
    }
  }, [count])

  return (
    <View style={styles.container}>
      <Text style={styles.label}>Vous avez appuyé { count } fois</Text>
      <ButtonAddOne onPress={handlePress} style={[styles, { backgroundColor: colorButton }]} />
    </View>
  )
}

export default Test

const styles = StyleSheet.create({
  container: { justifyContent: "center", alignItems: "center", flex: 1 },
  label: { marginVertical: 60 }
})

```

#### Option 2 : La version courte (Opérateur ternaire ⚡)

```javascript
import { StyleSheet, Text, View } from 'react-native'
import React, { useState } from 'react'
import ButtonAddOne from '../components/buttonAddOne';

const Test = () => {
  const [count, setCount] = useState(0);
  const [colorButton, setColorButton] = useState("blue")

  const handlePress = () => {
    // Changement de couleur direct au clic !
    count % 2 === 0 ? setColorButton('green') : setColorButton('blue')
    setCount(count + 1);
  }
  
  return (
    <View style={styles.container}>
      <Text style={styles.label}>Vous avez appuyé { count } fois</Text>
      <ButtonAddOne onPress={handlePress} style={[styles, { backgroundColor: colorButton }]} />
    </View>
  )
}
export default Test

```

{{% /expand %}}
{{% /notice%}}

---

## `useEffect`

`useEffect` est le chef d'orchestre de vos composants. 🎻

```javascript
useEffect(() => {
  // 1. J'exécute mon code ici
  
  return () => {
    // 2. Optionnel : Je nettoie avant de partir 
  };
}, [lesDépendances]);

```

### 🚦 Comment le contrôler avec le tableau de dépendances ?

* **`[ ]` Vide :** Ne s'exécute qu'**une seule fois** (quand le composant apparaît). Parfait pour charger des données !
* **`[variable]` Rempli :** S'exécute à chaque fois que la `variable` change de valeur.
* **Pas de tableau :** S'exécute à **chaque** rendu. (Attention aux boucles infinies ! 🚨)

{{% notice style="exo" title="Exo 2" %}}

**Mission :** Créer une vraie calculatrice qui s'adapte automatiquement si l'écran passe en mode **Paysage** ou **Portrait** grâce à `useEffect` !

**Code de départ du bouton :**

```javascript
// Composant boutonCalculatrice
// 
import { StyleSheet, Text, View,TouchableOpacity, Dimensions} from 'react-native'
import React from 'react'

const BoutonCalculatrice = ({value, handlePress}) => {
  const screen = Dimensions.get("window")

  return (
    
    <TouchableOpacity onPress={handlePress} style={[styles.container,styles.numberButton, {height:((screen.width - 50) / 4)}]}>
      <Text style={styles.text}>
        {value}
      </Text>
    </TouchableOpacity>
    
  )
}

export default BoutonCalculatrice

const styles = StyleSheet.create({
  container: {
    margin: 10,
    borderRadius: 10,
    justifyContent: 'center',
    alignItems: 'center',
  },
  numberButton: {
    backgroundColor: '#333333',  // Dark gray for numeric buttons
  },
  functionButton: {
    backgroundColor: '#00e676',  // Neon green for function buttons
    
  },
  text: {
    color: '#e0e0e0',  // Light gray 
    fontSize: 20,
    fontWeight: '500',
  },
  textFonction: {
    color: '#C60709',  // Dark orange
    fontSize: 20,
    fontWeight: '500',
  },
})
```
```jsx
//calculatrice.jsx

import { StyleSheet, Text, View, Dimensions } from 'react-native'
import React, {useState} from 'react'
import BoutonCalculatrice from '../components/boutonCalculatriceCours'
import { SafeAreaView } from 'react-native-safe-area-context';
const Calculatrice = () => {


  
  return (


    <SafeAreaView style={styles.container}>
      <View style={styles.containerLabel}>
        <Text style={styles.label}>Something</Text>
      </View>
      <View style={styles.row}>
        <View style={styles.column}>
          <BoutonCalculatrice value="1" />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="2" />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="3" />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="+" />
        </View>
      </View>
      <View style={styles.row}>
        
        <View style={styles.column}>
          <BoutonCalculatrice value="4" />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="5" />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="6" />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="-" />
        </View>
      </View>
      <View style={styles.row}>
        <View style={styles.column}>
          <BoutonCalculatrice value="7" />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="8" />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="9" />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="x" />
        </View>
      </View>
      <View style={styles.row}>
        <View style={styles.column}>
          <BoutonCalculatrice value="0" />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="." />
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="<-"/>
        </View>
        <View style={styles.column}>
          <BoutonCalculatrice value="=" />
        </View>
      </View>
    </SafeAreaView>

  );
};



export default Calculatrice

const styles = StyleSheet.create({
  container: {
    flex:1,
    backgroundColor:"grey",
    padding: 10,
    alignItems:"flex-end",
    justifyContent:"flex-end"

  },
  row: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 0,
  },
  column: {
    flex: 1,
    marginHorizontal: 0,
  },
  label:{
    textAlign:"right",
    fontSize:30,
    paddingHorizontal:12,
    width:"90%"
  },
  containerLabel:{
    height:60,
    width:'100%',
    flexDirection: 'row',
    marginBottom: 10,
    justifyContent:'flex-end',
    alignItems:"center"
  }
});
```
{{% expand title="📱 Solution de l'Exercice 2 : Step 1" %}}
En premier, on va s'arranger pour que la caluclatrice s'affiche et fonctionne.


**boutonCalculatrice.jsx**
```jsx
import { StyleSheet, Text, View,TouchableOpacity, Dimensions} from 'react-native'
import React from 'react'
import { evaluate } from 'mathjs';


const BoutonCalculatrice = ({value, affichage, setAffichage}) => {
  const screen = Dimensions.get("window")
  const handlePress = () => {
    switch (value) {
      case "<-":
        setAffichage((prec) => {
          if(prec.length === 1 || prec === "ERR 0"){
            return "0"
          }
          else {
            prec = prec.slice(0, -1)
            return prec
          }
        })
        break;
      case "=":
        setAffichage((prec) => {
          try {
            return `${evaluate(prec)}`
          } catch (error) {
            return "ERR 0"
          }
        })
        break;
      default:
        setAffichage((prec) => {
          if(prec === "0" || prec === "ERR 0"){
            return value
          }
          else {
            return `${prec}${value}`

          }
        })
        break;
    }
  }
  return (
    
    <TouchableOpacity onPress={handlePress} style={[styles.container,styles.numberButton, {height:((screen.width - 50) / 4)}]}>
      <Text style={styles.text}>
        {value}
      </Text>
    </TouchableOpacity>
    
  )
}

export default BoutonCalculatrice

const styles = StyleSheet.create({
  container: {
    margin: 10,
    borderRadius: 10,
    justifyContent: 'center',
    alignItems: 'center',
  },
  numberButton: {
    backgroundColor: '#333333',  // Dark gray for numeric buttons
  },
  functionButton: {
    backgroundColor: '#00e676',  // Neon green for function buttons
    
  },
  text: {
    color: '#e0e0e0',  // Light gray 
    fontSize: 20,
    fontWeight: '500',
  },
  textFonction: {
    color: '#C60709',  // Dark orange
    fontSize: 20,
    fontWeight: '500',
  },
})
```

**calculatrice.jsx**
```javascript

import { StyleSheet, Text, View, Dimensions } from 'react-native'
import React, {useState} from 'react'
import ButtonCalculatrice from '../components/boutonCalculatrice'
import { SafeAreaView } from 'react-native-safe-area-context';
const Calculatrice = () => {
  const [affichage, setAffichage] = useState("0")
  const values = [["1","2","3","+"],["4","5","6","-"],["7","8","9","*"],["0",".","<-","="]];
  
  return (


    <SafeAreaView style={styles.container}>
      <View style={styles.containerLabel}>
        <Text style={styles.label}>{affichage}</Text>
      </View>
      {values.map((data, index) => {
        return (
          <View key={`r-${index}`} style={styles.row}>
            {data.map((value, index2) => {
              return (
                <View key={`r-${index}-c-${index2}`} style={styles.column}>
                  <ButtonCalculatrice value={value} affichage={affichage} setAffichage={setAffichage}/>
                </View>
              )
            })}
          </View>
        )
      })}
    </SafeAreaView>

  );
};



export default Calculatrice

const styles = StyleSheet.create({
  container: {
    flex:1,
    backgroundColor:"grey",
    padding: 10,
    alignItems:"flex-end",
    justifyContent:"flex-end"

  },
  row: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 0,
  },
  column: {
    flex: 1,
    marginHorizontal: 0,
  },
  label:{
    textAlign:"right",
    fontSize:30,
    paddingHorizontal:12,
    width:"90%"
  },
  containerLabel:{
    height:60,
    width:'100%',
    flexDirection: 'row',
    marginBottom: 10,
    justifyContent:'flex-end',
    alignItems:"center"
  }
});

```

{{% /expand %}}

{{% expand title="📱 Solution de l'Exercice 2 : Step 2" %}}
Ensuite, on règle l'orientation et on s'assure que l'affichage reste beau dans les 2 orientations.


**boutonCalculatrice.jsx**
```jsx
import { StyleSheet, Text, View,TouchableOpacity, Dimensions} from 'react-native'
import React from 'react'
import { evaluate } from 'mathjs';


const BoutonCalculatrice = ({value, affichage, setAffichage}) => {
  const isNumber = (value) => {
    return Number.isFinite(Number(value));
  }
  const handlePress = () => {
    switch (value) {
      case "<-":
        setAffichage((prec) => {
          if(prec.length === 1 || prec === "ERR 0"){
            return "0"
          }
          else {
            prec = prec.slice(0, -1)
            return prec
          }
        })
        break;
      case "=":
        setAffichage((prec) => {
          try {
            return `${evaluate(prec)}`
          } catch (error) {
            return "ERR 0"
          }
        })
        break;
      default:
        setAffichage((prec) => {
          if(prec === "0" || prec === "ERR 0"){
            return value
          }
          else {
            return `${prec}${value}`

          }
        })
        break;
    }
  }
  return (
    
    <TouchableOpacity onPress={handlePress} style={[styles.container,isNumber(value) ? styles.numberButton : styles.functionButton]}>
      <Text style={isNumber(value) ? styles.text : styles.textFonction}>
        {value}
      </Text>
    </TouchableOpacity>
    
  )
}

export default BoutonCalculatrice

const styles = StyleSheet.create({
  container: {
    flex:1,
    justifyContent: 'center',
    alignItems: 'center',
    borderRadius:20,
  },
  numberButton: {
    backgroundColor: '#333333',  // Dark gray for numeric buttons
  },
  functionButton: {
    backgroundColor: '#00e676',  // Neon green for function buttons
    
  },
  text: {
    color: '#e0e0e0',  // Light gray 
    fontSize: 20,
    fontWeight: '500',
  },
  textFonction: {
    color: '#000000',  // Dark orange
    fontSize: 20,
    fontWeight: '500',
  },
})
```

**calculatrice.jsx**
```javascript
import { StyleSheet, Text, View, Dimensions, useWindowDimensions } from 'react-native'
import React, {useState} from 'react'
import ButtonCalculatrice from '../components/boutonCalculatrice'
import { SafeAreaView } from 'react-native-safe-area-context';
import { min } from 'mathjs';
const Calculatrice = () => {
  const [affichage, setAffichage] = useState("0")
  const { width, height } = useWindowDimensions()
  const isHorizontal = width > height
  const values = isHorizontal ? [["1","2","3","4","+","-"],["5","6","7","8","*", "/"],["9","0",".","%","<-","="]] :
                                [["1","2","3","+"],["4","5","6","-"],["7","8","9","*"],["0",".","<-","="]] 
  const sizeButton = min(width, height) / values[0].length - 20
  return (


    <SafeAreaView style={styles.container}>
      <View style={styles.containerLabel}>
        <Text style={styles.label}>{affichage}</Text>
      </View>
      <View style={styles.containerKeypad}>
        {values.map((data, index) => {
          return (
            <View key={`r-${index}`} style={styles.row}>
              {data.map((value, index2) => {
                return (
                  <View key={`r-${index}-c-${index2}`} style={[{width:sizeButton, height:sizeButton}, styles.column, isHorizontal ? {width:sizeButton * 2} : {}]}>
                    <ButtonCalculatrice value={value} affichage={affichage} setAffichage={setAffichage}/>
                  </View>
                )
              })}
            </View>
          )
        })}
      </View>
    </SafeAreaView>

  );
};



export default Calculatrice

const styles = StyleSheet.create({
  container: {
    flex:1,
    backgroundColor:"grey",
    padding: 10,
    alignItems:"flex-end",
    justifyContent:"flex-end"

  },
  containerKeypad:{
    width:"100%",
    paddingVertical:10,
  },
  row: {
    width:'100%',
    marginVertical:10,
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 0,
  },
  column: {
    marginHorizontal: 0,
  },
  label:{
    textAlign:"right",
    fontSize:30,
    paddingHorizontal:12,
    width:"90%"
  },
  containerLabel:{
    height:60,
    width:'100%',
    flexDirection: 'row',
    marginBottom: 10,
    justifyContent:'flex-end',
    alignItems:"center"
  }
});


```

{{% /expand %}}
{{% /notice %}}

