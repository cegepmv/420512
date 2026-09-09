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