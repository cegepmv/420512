+++
date = '2026-06-25T10:40:56-04:00'
draft = false
title = "Mise en place"
pre="1. "
weight = 1
+++

{{% notice note "À faire à la maison" %}}


## Installation

---

### Linux
```bash
sudo apt install curl
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.5/install.sh | bash
source ~/.bashrc
nvm install 24
```

---

### Windows
Installer Node ([lien](https://nodejs.org/en))

---

## Vérifiez l'installation
```bash
npm -v
node -v
```

{{% /notice %}}

{{% notice note "À faire à l'école" %}}
Connectez-vous sur votre container avec vos informations :

```bash
ssh root@fourniermax-mob.lan
```
mdp : Numéro d'admission

{{% /notice %}}

## Créez le projet
```bash
# Choisissez blank comme template
# SDK 54
npx create-expo-app@latest premierProjet --template
cd 'premierProjet'
```



## Lancer le code
```bash
npx expo start -c
```

Si tout s'est bien passé, vous devriez voir un **code QR**. Il est maintenant temps d'installer **Expo Go** sur votre appareil mobile.

Après l'installation, vous pouvez scanner le code QR et voir l'application de base se déployer sur votre mobile.

Installez l'extension sur VS Code : `ES7+ React/Redux/React-Native snippets`

{{% notice info "Commandes utiles" %}}

## Installer les dépendances sans tout briser
```bash
npx expo install
```

## Mettre à jours les dépendances sans tout briser
```bash
npx expo install --check
```

## Tout réinstaller les dépendances, "clean reinstall"
```bash
rm -rf node_modules yarn.lock package-lock.json

npm cache clean --force
npx expo install --check
```

## Forcer un peu la main à npm s'il est difficile avec les dépendances
```
--legacy-peer-deps
```
{{% /notice %}}

Nous allons éventuellement avoir besoin de `expo-router` pour du "routing" :

```bash
npx expo install expo-router

```

Ensuite, créez un dossier nommé `app` et copiez le contenu de `App.js` dans un fichier nommé `index.jsx`. Vous devez également ajouter `"main": "expo-router/entry"` dans le fichier `package.json`.

Créez maintenant un fichier dans `app` et nommez-le `_layout.jsx`. Il contiendra l'information qui doit être dans toutes les pages tel que la navbar, le header, le footer... Dans ce fichier, faites `rnfes + enter` pour rapidement mettre le code de base d'un `.jsx`.

Dans ce fichier, sélectionnez le `_layout` qui se trouve après `const`. Une fois sélectionné, faites `ctrl + d` x2 pour sélectionner les 2 autres dans le fichier puis tapez `RootLayout`. Vous remarquerez que tous les `_layout` sont devenus `RootLayout`.

<!-- 
later

npx expo install expo-dev-client # Installer le client qui permet de développer en live sur le cell

```

Dans `app.json`
```json
{
  "expo": {
    "name": "Your App Name",
    "slug": "your-app-slug",
    "version": "1.0.0",
    "ios": {
      "bundleIdentifier": "com.yourname.yourapp"
    },
    "android": {
      "package": "com.yourname.yourapp"
    }
  }
}
```

```bash
npx expo prebuild
```




Let's get your local machine fully configured.Here is the exact setup sequence to get your computer ready to build and install the app directly over USB.

## Step 1: 
Install Java 17 and Linux 32-bit LibrariesAndroid's build tools require Java 17 and specific 32-bit compatibility libraries on 64-bit Linux environments. Run this in your terminal:Bash

```
sudo apt update
sudo apt install openjdk-17-jdk libc6-dev-i386 lib32z1 adb -y
```

## Step 2: 
Manually Download the Android Command Line ToolsWe need to download the official minimal CLI package from Google, extract it, and structure it exactly how Gradle expects it.Bash

### 1. Create the structured directory layout
```
mkdir -p ~/Android/Sdk/cmdline-tools
```

### 2. Navigate to it and download the official package
```
cd ~/Android/Sdk/cmdline-tools
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
```

### 3. Unzip and tidy up files
```
unzip commandlinetools-linux-11076708_latest.zip
mv cmdline-tools latest
rm commandlinetools-linux-11076708_latest.zip
```


## Step 3: 
Map Out Your Environment VariablesNow we tell your system and Expo exactly where to find these tools. Open your environment profile:Bash
```
nano ~/.bashrc
```


Scroll to the very bottom of the file and paste these paths:

```
export ANDROID_HOME=$HOME/Android/Sdk
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```


Save and exit (Ctrl+O, Enter, Ctrl+X), then reload your shell:

```
source ~/.bashrc
```


## Step 4: Accept Licenses and Fetch Build Components

Now that the paths are linked, we can use the sdkmanager tool we just downloaded to accept Google's licensing agreements and download the compilation utilities.

## 1. Accept the licenses (Press 'y' to accept each prompt)
sdkmanager --licenses

## 2. Download the platform and build components React Native needs
sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"

Connecting Your Physical PhoneWith your computer configured, prepare your phone to receive the binary payload:

1. Unlock Developer Mode :On your Android phone, navigate to Settings > About Phone. Scroll down to Build Number and tap it rapidly 7 times until a toast message says "You are now a developer!"

2. Enable USB Debugging :Go back to the main settings page, search for Developer Options, locate the USB Debugging toggle, and switch it on.

3. Connect and Authorize :Plug your phone into your computer via a USB cable. A prompt will appear on your phone screen asking to Allow USB Debugging from this computer? Check the box for "Always allow" and hit Allow.

4. Verify Link Connection :Back in your Linux terminal, verify the handshake by typing adb devices. You should see your phone listed with the status labeled as device.

## Step 5: 
Wipe the Cache and Run!Head back into your project root folder. Before compiling, flush out the old failed configurations so Gradle starts with a completely blank slate:

```
cd android && ./gradlew clean && cd ..

EXPO_OFFLINE=1 npx expo run:android
```

Your computer will compile the customized development build and sideload it directly onto your physical







That is a fantastic way to teach! Sharing a pre-built APK saves your students hours of agonizing over environment setup (installing Android Studio, configuring SDKs, and managing Gradle versions).Here are the answers to your questions and exactly how to make it work seamlessly for your class.

1. Is it legal?Yes, 100% legal.Since you built this .apk yourself from your project, you own the distribution rights to it. Android is open-source, and Google deliberately allows the sharing of raw APKs for testing and development. Because you aren't publishing it to the Google Play Store, you don't have to go through any official review or licensing hurdles. Your students just have to allow "Install from Unknown Sources" on their phones or emulators to install it.

2. Can they all work on different projects at the same time using this ONE APK?Yes, but with one crucial requirement: Your app.json configuration must support it.An Expo Development Build APK works exactly like an "advanced launcher." When a student opens the app on their phone, they can scan a QR code or type in a local URL (like [http://192.168.0.](http://192.168.0.)X:8081) to load their own JavaScript code into the app.However, they can only load code for a project that shares the same URL scheme as the APK.

3. The Modif you need to make (The Launcher Hack)By default, an Expo development build restricts itself to listening only to its original project identifier. To turn your APK into a universal class launcher that can load anyone's code, you need to explicitly define a generic deep-linking URL scheme in your app.json before you compile the final APK.Add a "scheme" property to the root of your app.json like this:

```JSON
{
  "expo": {
    "name": "premierProjet",
    "slug": "premierProjet",
    "scheme": "classroom-launcher", 
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "android": {
      "package": "com.maxime.premierProjet"
    }
  }
}
```
How your students must structure their projects:For your students to use your APK on their phones without compiling their own, their local app.json files must match your "scheme" and "package" identifier exactly.Provide them with a boilerplate/template folder where their app.json looks like this:JSON{
  "expo": {
    "name": "Student-Project-Name",
    "slug": "student-project-slug",
    "scheme": "classroom-launcher",
    "android": {
      "package": "com.maxime.premierProjet"
    }
  }
}
The Classroom WorkflowYou build the universal APK (after adding the scheme) and give the app-debug.apk file to your students via a USB drive, Google Drive, or your school's LMS.Students install the APK on their personal Android phones or their local emulators.  Students write their own code in their own unique project folders.When they want to test, they run:BashEXPO_OFFLINE=1 npx expo start
They open your launcher app on their phone, tap "Enter URL manually," and type in their own computer's Metro bundler IP address (e.g., [http://192.168.0.](http://192.168.0.)X:8081).The APK will pull their unique code and run it instantly!⚠️ One pedagogical catch: If a student installs a new npm package that requires native Android code (like modifying the device's actual OS architecture or using a specialized native library), it won't work unless you rebuild the APK to include it. For standard React Native components, UI libraries, and JavaScript code, your single master APK will work perfectly for all of them all semester long!


 MAke some changes to the debug apk and build it again. Dont forget to change the app.json and maybe the package.json
 cd android && ./gradlew clean && cd ..
 npx expo prebuild --platform android --clean
 EXPO_OFFLINE=1 npx expo run:android

---


Here is the complete guide, formatted beautifully in clean Markdown so you can easily copy, paste, and distribute it directly to your students (via a README.md file, GitHub, or your school's online portal).

Markdown


# 🚀 Guide d'installation et d'utilisation de l'application (Étudiants)

Ce guide vous explique comment installer l'application de base de la classe sur votre téléphone Android et la connecter à votre propre code sur votre ordinateur.

---

## 📱 Étape 1 : Installer l'application (APK) sur votre téléphone

Avant de coder, vous devez installer le lanceur d'application qui servira de pont entre votre téléphone et votre ordinateur.

1. **Téléchargez le fichier :** Récupérez le fichier `classroom-launcher.apk` fourni par l'enseignant et transférez-le sur votre téléphone Android (via Google Drive, clé USB, courriel, etc.).
2. **Ouvrez le fichier :** Sur votre téléphone, allez dans votre application **Fichiers** (ou Téléchargements) et appuyez sur `classroom-launcher.apk`.
3. **Autorisez les sources inconnues :** 
   * Android affichera un message de sécurité indiquant que l'installation d'applications inconnues est bloquée.
   * Appuyez sur **Paramètres** (Settings) directement sur l'alerte.
   * Activez l'option **"Autoriser à partir de cette source"** (Allow from this source).
   * Revenez en arrière et appuyez sur **Installer**.

---

## 💻 Étape 2 : Lancer votre projet sur l'ordinateur

Dans votre dossier de projet (qui contient le fichier `app.json` configuré avec le bon modèle fourni en classe), ouvrez votre terminal et lancez le serveur de développement :

```bash
npx expo start
```
Cette commande va démarrer le serveur de développement (Metro Bundler) et affichera un Code QR ainsi qu'une adresse locale dans votre terminal (ex: exp://192.168.2.50:8081).

🔗 Étape 3 : Connecter l'application à votre code
Ouvrez l'application premierProjet (ou le nom donné par l'enseignant) qui vient d'être installée sur votre téléphone. Vous verrez un menu de développement Expo Dev Client.

Option A (Le plus simple) : Appuyez sur "Scan QR Code" dans l'application et scannez le code QR qui s'affiche dans le terminal de votre ordinateur.

Option B (Alternative manuelle) : Si votre caméra ne fonctionne pas, appuyez sur "Enter URL manually" et entrez l'adresse http de votre terminal (ex: http://192.168.2.50:8081).

Dès que la connexion est établie, l'application va charger votre code JavaScript local. Vous pouvez maintenant modifier votre code sur votre ordinateur et voir les changements en direct sur votre écran de téléphone !

⚠️ Le piège numéro 1 à éviter !
Pour que le téléphone puisse lire le code de l'ordinateur, votre téléphone et votre ordinateur doivent obligatoirement être connectés sur le même réseau Wi-Fi (le Wi-Fi de la classe ou le partage de connexion de votre téléphone). Si votre téléphone utilise vos données cellulaires (4G/5G), la connexion échouera.








 EXPO_OFFLINE=1 npx expo run:android --variant release

 -->




