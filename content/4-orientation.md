+++
date = '2026-06-25T15:18:46-04:00'
draft = false
weight = 4
pre = "4. "
title = 'Orientation'
+++


## 1. Gestion de l'orientation de l'écran avec Expo
[Voir la doc](https://docs.expo.dev/versions/latest/sdk/screen-orientation/)

Par défaut, les applications mobiles sont souvent bloquées en mode portrait. Pour permettre à une application de pivoter dynamiquement entre le mode portrait et paysage, des configurations et une bibliothèque externe sont requises.

### Étape A : Installation
Utilisez toujours l'interface de commande d'Expo (CLI) plutôt que le `npm` standard pour garantir la compatibilité des versions avec votre SDK Expo actuel.
```bash
npx expo install expo-screen-orientation

```

### Étape B : Configuration (`app.json`)

```json
{
  "expo": {
    "ios": {
      "requireFullScreen": true
    },
    "plugins": [
      [
        "expo-screen-orientation",
        {
          "initialOrientation": "DEFAULT"
        }
      ]
    ]
  }
}

```
