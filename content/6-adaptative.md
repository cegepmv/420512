+++
date = '2026-09-08T14:53:03-04:00'
draft = false
weight = 6
pre = "6. "
title = 'Design réactif'
+++

---

# Notes de cours : Design Réactif en React Native

## useWindowDimensions

Le Hook `useWindowDimensions` de React Native est l'outil indispensable pour rendre vos composants réactifs aux changements de dimensions de l'écran (rotation de l'appareil, mode fenêtré sur tablette, pliables, etc.).

* **Fonctionnement :** Il retourne un objet contenant la `width` et la `height` actuelles de la fenêtre, ainsi que la `scale` et la `fontScale`.
* **Avantage clé :** Contrairement à l'objet `Dimensions` (qui nécessite des écouteurs d'événements complexes pour se mettre à jour), `useWindowDimensions` se met à jour **automatiquement et instantanément** lors d'une modification de l'orientation ou de la taille de la fenêtre.
* **Exemple d'utilisation :**
```javascript
import { useWindowDimensions, StyleSheet, Text, View } from 'react-native';

export default function App() {
  const { width, height } = useWindowDimensions();

  const isLandscape = width > height;

  return (
    <View style={[styles.container, { flexDirection: isLandscape ? 'row' : 'column' }]}>
      <Text>Largeur actuelle : {width}px</Text>
    </View>
  );
}



const styles = StyleSheet.create({
    container: {
        flex:1
    }
})
```



---

## FlexBox et width en %

En React Native, le système de mise en page repose entièrement sur **Flexbox**. Contrairement au CSS web où la propriété `flex` peut être complexe (`flex-grow`, `flex-shrink`, `flex-basis`), React Native simplifie grandement son utilisation en se basant principalement sur des **ratios proportionnels**.

* **Le principe du ratio :**
Lorsque vous attribuez une valeur numérique à la propriété `flex` d'un composant, vous lui donnez un **poids** par rapport aux autres enfants du même conteneur. React Native calcule l'espace disponible et le distribue en additionnant tous les `flex` des éléments frères.
* **Comment le calcul s'effectue :**
* Si vous avez deux blocs, l'un avec `flex: 1` et l'autre avec `flex: 3`, le total des parts est de **4** ($1 + 3$).
* Le premier bloc occupera **1/4** ($25\%$) de l'espace disponible.
* Le second bloc occupera **3/4** ($75\%$) de l'espace disponible.


* **Exemple concret (Mise en page type Sidebar + Contenu principal) :**
```javascript
import { StyleSheet, View } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      {/* Sidebar qui prend 1 part (25%) */}
      <View style={[styles.box, { flex: 1, backgroundColor: '#ffcccc' }]} />

      {/* Contenu principal qui prend 3 parts (75%) */}
      <View style={[styles.box, { flex: 3, backgroundColor: '#ccffcc' }]} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1, // Le conteneur prend tout l'écran
    flexDirection: 'row', // Dispose les enfants côte à côte
  },
  box: {
    height: '100%',
  },
});

```

* **Utilisation des pourcentages (`%`) :**
* Vous pouvez attribuer des valeurs en pourcentage sous forme de chaînes de caractères (ex: `'50%'`) pour les propriétés `width` et `height`.
* Le pourcentage se réfère à la taille du **parent direct**. Si le parent n'a pas de dimension fixe, le comportement peut être imprévisible.

---

## NativeWind

**NativeWind** utilise Tailwind CSS pour styliser vos applications React Native. C'est une solution puissante pour industrialiser le design réactif grâce à l'utilisation de classes utilitaires.

* **Breakpoints intégrés :**
* Par défaut, NativeWind intègre les breakpoints standard de Tailwind (inspirés du mobile-first) :
* `sm` : 640px
* `md` : 768px
* `lg` : 1024px
* `xl` : 1280px


* **Syntaxe :** Vous préfixez simplement la classe par le breakpoint souhaité (ex: `w-full md:w-1/2` appliquera `w-full` par défaut, et `w-1/2` à partir de 768px de largeur).



---

## Custom breakpoints

Si les breakpoints par défaut de Tailwind/NativeWind ne correspondent pas à vos maquettes ou à vos besoins spécifiques sur mobile/tablette, vous pouvez les personnaliser.

* **Configuration :** La personnalisation s'effectue généralement dans le fichier de configuration de Tailwind (`tailwind.config.js`).
* **Exemple de configuration :**
```javascript
// tailwind.config.js
module.exports = {
  content: ["./App.{js,jsx,ts,tsx}", "./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {
      screens: {
        'tablet': '640px',
        'laptop': '1024px',
        'desktop': '1280px',
      },
    },
  },
  plugins: [],
}

```


* **Utilisation dans le code :** Vous pouvez dès lors utiliser vos propres préfixes directement dans vos composants :
```javascript
<View className="w-full tablet:w-1/2 laptop:w-1/3">
  {/* Contenu responsive */}
</View>

```




{{% notice style="exo" %}}
En partant de ce code, faites en sorte que le visuel soit adaptatif au fait de passer l'écran en mode paysage et portrait. Utiliser d'abord useWindowDimensions, puis NativeWind.
```jsx
//calculatrice.jsx

import { StyleSheet, Text, View, Dimensions } from 'react-native'
import React, {useState} from 'react'
import BoutonCalculatrice from '../components/boutonCalculatrice'
import { SafeAreaView } from 'react-native-safe-area-context';
const Calculatrice = () => {
    const [resultat, setResultat] = useState("0")
    const touches = [
                        ["1", "2", "3", "+"],
                        ["4", "5", "6", "-"],
                        ["7", "8", "9", "x"],
                        ["0", ".", "<-", "="]
                    ]
  
    return (

        <SafeAreaView style={styles.container}>
        <View style={styles.containerLabel}>
            <Text style={styles.label}>{resultat}</Text>
        </View>
       

       {touches.map((row, indexRow) => (

           <View key={indexRow} style={styles.row}>
           
                {row.map((value, indexCol) => (
                    
                    <View key={indexCol} style={styles.column}>
                        <BoutonCalculatrice value={value} resultat={resultat} setResultat={setResultat}/>
                    </View>
                )
                )}
            </View>
        ))}
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
```jsx
// Composant boutonCalculatrice
// 
import { StyleSheet, Text, View,TouchableOpacity, Dimensions} from 'react-native'
import React from 'react'

const BoutonCalculatrice = ({value, resultat, setResultat}) => {
  const screen = Dimensions.get("window")
    const handleClick = () => {
        
        if (value == "<-") {
            if (resultat.length == 1 || resultat == "Erreur") {
                setResultat("0")
            }
            else{
                setResultat(resultat.slice(0, -1));
            }
        }
        else if (value =="=") {
            try {
                setResultat(`${eval(resultat.replaceAll("x","*"))}`)
            } catch (error) {
                setResultat("Erreur")
            }
        }
        else {
            if (resultat == "0") {
                setResultat(value)
            }
            else {
                setResultat(resultat + value)
            }
        }
    }
  return (
    
    <TouchableOpacity onPress={handleClick} style={[styles.container,styles.numberButton, {height:((screen.width - 50) / 4)}]}>
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
{{% /notice %}}


{{% notice style="exo" %}}
Code pour recommencer le 10 septembre

```jsx
// Composant boutonCalculatrice
// 
import { StyleSheet, Text, View,TouchableOpacity, Dimensions} from 'react-native'
import React, { useEffect, useState} from 'react'

const BoutonCalculatrice = ({value, resultat, setResultat, width}) => {
    let isSpecial = false

    if(value in ["1", "2", "3", "4", "5", "6", "7", "8", "9", "0", "."]){

    }
    else {
      isSpecial = true;
    }


    const handleClick = () => {
        
        if (value == "<-") {
            if (resultat.length == 1 || resultat == "Erreur") {
                setResultat("0")
            }
            else{
                setResultat(resultat.slice(0, -1));
            }
        }
        else if (value =="=") {
            try {
                setResultat(`${eval(resultat.replaceAll("x","*"))}`)
            } catch (error) {
                setResultat("Erreur")
            }
        }
        else {
            if (resultat == "0") {
                setResultat(value)
            }
            else {
                setResultat(resultat + value)
            }
        }
    }
  return (
    
    <TouchableOpacity onPress={handleClick} style={
                                                  [styles.container,styles.numberButton, 
                                                  {height:((width - 50) / 4)},
                                                  isSpecial ?  styles.functionButton : styles.numberButton
                                                  ]}>
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
    flex:1
  },
  numberButton: {
    backgroundColor: '#333333',  // Dark gray for numeric buttons
  },
  functionButton: {
    backgroundColor: '#e600b0',  // Neon green for function buttons
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

import { StyleSheet, Text, View, Dimensions, useWindowDimensions } from 'react-native'
import React, { useState } from 'react'
import BoutonCalculatrice from '../components/boutonCalculatrice'
import { SafeAreaView } from 'react-native-safe-area-context';
const Calculatrice = () => {
  const { width, height } = useWindowDimensions();
  const [resultat, setResultat] = useState("0")

  const isHorizontal = width > height;

  const touches = isHorizontal ?
    [
      ["1", "2", "3", "4", "+", ")"],
      ["5", "6", "7", "8", "-", "("],
      ["9", "0", ".", "x", "<-", "="],

    ]
    :
    [
      ["1", "2", "3", "+"],
      ["4", "5", "6", "-"],
      ["7", "8", "9", "x"],
      ["0", ".", "<-", "="]
    ]
  // const hauteurBouton = 
  return (

    <SafeAreaView style={styles.container}>
      <View style={styles.containerLabel}>
        <Text style={styles.label}>{resultat}</Text>
      </View>

      <View style={{flex: isHorizontal ? 2 : 1 }}>

        {touches.map((row, indexRow) => (

          <View key={indexRow} style={styles.row}>

            {row.map((value, indexCol) => (

              <View key={indexCol} style={styles.column}>
                <BoutonCalculatrice value={value} width={width} resultat={resultat} setResultat={setResultat} />
              </View>
            )
            )}
          </View>
        ))}
      </View>
    </SafeAreaView>

  );
};



export default Calculatrice

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "grey",
    padding: 10,

  },
  row: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 0,
    flex: 1,
  },
  column: {
    flex: 1,
    marginHorizontal: 0,

  },
  label: {
    textAlign: "right",
    fontSize: 30,
    paddingHorizontal: 12,
    width: "90%"
  },
  containerLabel: {
    flex:1,
    flexDirection: 'row',
    marginBottom: 10,
    justifyContent: 'flex-end',
    alignItems: "center"
  }
});
```

{{% /notice %}}


{{% expand title="Solution Calculatrice" %}}
```jsx
//calculatrice.jsx

import { StyleSheet, Text, View, Dimensions, useWindowDimensions } from 'react-native'
import React, { useState } from 'react'
import BoutonCalculatrice from '../components/boutonCalculatrice'
import { SafeAreaView } from 'react-native-safe-area-context';
const Calculatrice = () => {
  const { width, height } = useWindowDimensions();
  const [resultat, setResultat] = useState("0")

  const isHorizontal = width > height;

  const touches = isHorizontal ?
    [
      ["1", "2", "3", "4", "+", ")"],
      ["5", "6", "7", "8", "-", "("],
      ["9", "0", ".", "x", "<-", "="],

    ]
    :
    [
      ["1", "2", "3", "+"],
      ["4", "5", "6", "-"],
      ["7", "8", "9", "x"],
      ["0", ".", "<-", "="]
    ]
  return (

    <SafeAreaView className="flex-1 justify-center items-center bg-[#525050]">
      <View className="lg:w-1/2" style={styles.container}>

        <View style={styles.containerLabel}>
          <Text style={styles.label}>{resultat}</Text>
        </View>

        <View style={{flex: isHorizontal ? 2 : 1 }}>

          {touches.map((row, indexRow) => (
            
            <View key={indexRow} style={styles.row}>

              {row.map((value, indexCol) => (
                
                <View key={indexCol} style={styles.column}>
                  <BoutonCalculatrice value={value}  resultat={resultat} setResultat={setResultat} />
                </View>
              )
            )}
            </View>
          ))}
        </View>
      </View>
    </SafeAreaView>

  );
};



export default Calculatrice

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "grey",
    padding: 10,

  },
  row: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 0,
    flex: 1,
  },
  column: {
    flex: 1,
    marginHorizontal: 0,

  },
  label: {
    textAlign: "right",
    fontSize: 30,
    paddingHorizontal: 12,
    width: "90%"
  },
  containerLabel: {
    flex:1,
    flexDirection: 'row',
    marginBottom: 10,
    justifyContent: 'flex-end',
    alignItems: "center"
  }
});
```
```jsx
import { StyleSheet, Text, View,TouchableOpacity} from 'react-native'
import React from 'react'


const BoutonCalculatrice = ({value, width, setResultat}) => {

  const handlePress = () => {
    switch (value) {
      case "<-":
        setResultat((prec) => {
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
        setResultat((prec) => {
          try {
            prec = prec.replace("x","*")
            return `${eval(prec)}`
          } catch (error) {
            return "ERR 0"
          }
        })
        break;
      default:
        setResultat((prec) => {
          if((prec === "0" && value != ".") || prec === "ERR 0"){
            return value
          }
          else {
            return `${prec}${value}`

          }
        })
        break;
    }
  }

  let isSpecial = true

  if (["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "."].includes(value)) {
    isSpecial = false
  }
  return (
    
    <TouchableOpacity onPress={handlePress} style={[styles.container,{backgroundColor: isSpecial ? "pink" :  "white", width: width}]}>
      <Text style={isSpecial ? styles.text : styles.textFonction}>
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
    margin:5
  },
  text: {
    fontSize: 20,
    fontWeight: '500',
    color: "white",

  },
  textFonction: {
    fontSize: 20,
    fontWeight: '500',

  },
})
```

{{% /expand %}}



{{% notice style="exo"%}}
Reproduire l'app de cette capture d'écran avec les indications du prof et ces arrays :

<img src="/420512/images/renduReactif.png" style="width: 50%; height: auto;" alt="Theme">



```jsx
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

<Text>🔥 Table des Types Pokémon 🍃</Text>
<Text>COMBAT ! ⚔️</Text>
<Text>👑</Text>
```
# Étape par étape
---

1. Créer un composant nommé TypeCalculator, le lier au `_Layout` et y mettre le code donné ci-dessus.

{{% expand title="Solution 1 (As-tu vraiment essayé?)" %}}
```jsx
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
{{% /expand %}}

2. 
  - Mettre une SafeAreaView et une ScrollView pour rendre le tout plus adaptatif aux différents écrans
  - Entourer le Text de combat d'une TouchableOpacity, car on doit la transformer en bouton
  - Entourer la couronne d'une View qui servira de boite pour la section des résultats
  - Mettre des styles pour chaque composant et créez ce style dans la StyleSheet
{{% expand title="Solution 2" %}}
```jsx
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
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scroll}>
        <Text style={styles.titre}>🔥 Table des Types Pokémon 🍃</Text>
        <TouchableOpacity style={styles.btnCombat}>
          <Text style={styles.textCombat}>
            COMBAT ! ⚔️
          </Text>
        </TouchableOpacity>
        <View style={styles.sectionResultats}>
          <Text style={styles.couronne}>👑</Text>
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

export default TypeCalculator



const styles = StyleSheet.create({
  container: {

  },
  titre: {

  },
  btnCombat: {

  },
  scroll: {

  },
  sectionResultats: {

  },
  textCombat: {

  },
  couronne: {
    
  }
})
```
{{% /expand %}}

3. 
  - Mettre le fond d'écran de la bonne couleur
  - Mettre le titre plus gros, plus gras et blanc
  - Centrer le titre
  - Ajouter "Type(s) Attaquant(s) [Max 2]" et "Type(s) Defenseur(s) [Max 2]" plus leur style.
    - couleur plus poids.
  - Mettre le bouton Combat en vert et sur la largeur de la page avec les bords arrondi. En plus d'ajuster le texte.
  - Faire une boite avec un fond plus pale pour les résulats et ajuster la fonte de la couronne.

{{% expand title="Solution 3" %}}
```jsx
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
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scroll}>
        <Text style={styles.titre}>🔥 Table des Types Pokémon 🍃</Text>
        <Text style={styles.titreType}>1. Type(s) Attaquant(s)[Max 2]</Text>
        <Text style={styles.titreType}>2. Type(s) Defenseur(s)[Max 2]</Text>
        <TouchableOpacity style={styles.btnCombat}>
          <Text style={styles.textCombat}>
            COMBAT ! ⚔️
          </Text>
        </TouchableOpacity>
        <View style={styles.sectionResultats}>
          <Text style={styles.couronne}>👑</Text>
        </View>
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

})
```
{{% /expand %}}

4. 
 - Ajouter les boutons pour les types attaquant et defenseur sans se soucier de la couleur pour l'instant.

{{% expand title="Solution 4"%}}
```jsx
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
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scroll}>
        <Text style={styles.titre}>🔥 Table des Types Pokémon 🍃</Text>
        <Text style={styles.titreType}>1. Type(s) Attaquant(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type) => {
            return (
              <TouchableOpacity style={styles.btnType}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <Text style={styles.titreType}>2. Type(s) Defenseur(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type) => {
            return (
              <TouchableOpacity style={styles.btnType}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <TouchableOpacity style={styles.btnCombat}>
          <Text style={styles.textCombat}>
            COMBAT ! ⚔️
          </Text>
        </TouchableOpacity>
        <View style={styles.sectionResultats}>
          <Text style={styles.couronne}>👑</Text>
        </View>
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
    paddingBottom: 20
  },
  btnType: {
    padding: 5,
    backgroundColor: "#49494981",
    borderColor: "#494949",
    borderRadius:10,
    borderWidth:1,
    margin: 5
  }
})
```
{{% /expand %}}

5. Faire en sorte que lorsqu'on click sur les boutons de l'attaquant, ils changent de couleur pour devenir la couleur de leur type
- Faire une liste en useState nommé typeAttaquant
- Dans le map faire un booléen nommée isSelected qui retient si oui ou non ce type est dans la liste
- dans le style du bouton de type, faites une condition qui ajoute la bonne couleur selon le type si isSelected est vrai
- Ajouter une fonction handlePress dans le onPress du bouton de type. Cette fonction prendra en parametre le type, la liste correspondante à ce type et son setter. 
  - Si le type est dans la liste on ne fait rien pour l'instant
  - S'il n'est pas dans la liste, on l'ajoute 
{{% expand title="Solution 5" %}}
```jsx
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

  const handlePress = ({type, liste, setListe}) => {
    if (liste.includes(type)) {
      
    }
    else {
      setListe([...liste, type])
    }
  }
  return (
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scroll}>
        <Text style={styles.titre}>🔥 Table des Types Pokémon 🍃</Text>
        <Text style={styles.titreType}>1. Type(s) Attaquant(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type) => {
            const isSelected = typeAttaquant.includes(type);
            return (
              <TouchableOpacity onPress={() => handlePress(type, typeAttaquant, setTypeAttaquant)} style={[styles.btnType, isSelected && {backgroundColor : TYPE_COLORS[type]}]}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <Text style={styles.titreType}>2. Type(s) Defenseur(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type) => {
            return (
              <TouchableOpacity style={styles.btnType}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <TouchableOpacity style={styles.btnCombat}>
          <Text style={styles.textCombat}>
            COMBAT ! ⚔️
          </Text>
        </TouchableOpacity>
        <View style={styles.sectionResultats}>
          <Text style={styles.couronne}>👑</Text>
        </View>
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
    paddingBottom: 20
  },
  btnType: {
    padding: 5,
    backgroundColor: "#49494981",
    borderColor: "#494949",
    borderRadius:10,
    borderWidth:1,
    margin: 5
  }
})
```
{{% /expand %}}

6. 
  - Limiter à 2 le nombre de type par liste
  - Permettre le retrait d'un type lorsqu'on click à nouveau sur un type déjà activé.
{{% expand title="Solution 6" %}}
```jsx
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

  const handlePress = (type, liste, setListe) => {
    if (liste.includes(type)) {
      setListe(liste.filter((t) => t != type))
    }
    else if (liste.length < 2){
      setListe([...liste, type])
    }
    
  }
  return (
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scroll}>
        <Text style={styles.titre}>🔥 Table des Types Pokémon 🍃</Text>
        <Text style={styles.titreType}>1. Type(s) Attaquant(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type) => {
            const isSelected = typeAttaquant.includes(type);
            return (
              <TouchableOpacity onPress={() => handlePress(type, typeAttaquant, setTypeAttaquant)} style={[styles.btnType, isSelected && {backgroundColor : TYPE_COLORS[type]}]}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <Text style={styles.titreType}>2. Type(s) Defenseur(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type) => {
            return (
              <TouchableOpacity style={styles.btnType}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <TouchableOpacity style={styles.btnCombat}>
          <Text style={styles.textCombat}>
            COMBAT ! ⚔️
          </Text>
        </TouchableOpacity>
        <View style={styles.sectionResultats}>
          <Text style={styles.couronne}>👑</Text>
        </View>
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
    paddingBottom: 20
  },
  btnType: {
    padding: 5,
    backgroundColor: "#49494981",
    borderColor: "#494949",
    borderRadius:10,
    borderWidth:1,
    margin: 5
  }
})
```
{{% /expand %}}

7. S'arranger pour que l'affichage des résultats soit absent au début et n'apparaisse que si on appui sur combat.

- Combat ne fera apparaitre les résultats que si la boite n'est pas là
- Un click qui modifie mes liste de type causera la boite de disparaitre

{{% expand title="Solution 7" %}}
```jsx
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
    if(!isResAffiche) {
      setIsResAffiche(true)
    }
  }
  return (
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scroll}>
        <Text style={styles.titre}>🔥 Table des Types Pokémon 🍃</Text>
        <Text style={styles.titreType}>1. Type(s) Attaquant(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type) => {
            const isSelected = typeAttaquant.includes(type);
            return (
              <TouchableOpacity onPress={() => handlePress(type, typeAttaquant, setTypeAttaquant)} style={[styles.btnType, isSelected && {backgroundColor : TYPE_COLORS[type]}]}>
                <Text style={styles.textBtnType}>{type}</Text>
              </TouchableOpacity>
            )
          })}
        </View>
        <Text style={styles.titreType}>2. Type(s) Defenseur(s)[Max 2]</Text>
        <View style={styles.containerType}>
          {TYPES.map((type) => {
            const isSelected = typeDefenseur.includes(type);
            return (
              <TouchableOpacity onPress={() => handlePress(type, typeDefenseur, setTypeDefenseur)} style={[styles.btnType, isSelected && {backgroundColor : TYPE_COLORS[type]}]}>
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
            <View style={styles.sectionResultats}>
              <Text style={styles.couronne}>👑</Text>
            </View>
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
  }
})
```
{{% /expand %}}

8. Formatter la boite de résultat

{{% expand title="Solution 8"%}}
```jsx
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
{{% /expand %}}
{{% /notice %}}


