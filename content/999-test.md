+++
date = '2026-06-25T15:43:23-04:00'
draft = true
title = 'Test'
weight = 1001
+++

### Notes Détailées pour les Tests React Native avec Jest et React Testing Library

#### 1. **Configuration du Projet pour les Tests avec Jest et React Testing Library**

Lorsque vous écrivez des tests pour des applications React Native, nous utilisons **Jest** comme framework de tests et **React Testing Library** pour le rendu des composants et la simulation d’interactions utilisateur.

##### Dépendances nécessaires :
Assurez-vous que les dépendances suivantes sont installées dans votre projet :
- `jest`: Framework de tests
- `@testing-library/react-native`: Fournit des utilitaires pour tester React Native (rendu, événements, etc.)
- `expo-router`: Pour la gestion de la navigation dans les applications Expo
- `axios`: Pour les requêtes HTTP, généralement pour les appels API
- `@testing-library/jest-native`: Étend Jest pour bien fonctionner avec les tests React Native
- `jest-mock`: Fonctions de simulation pour les tests

Installez les dépendances avec la commande suivante :
```bash
npx expo install --save-dev jest @testing-library/react-native @testing-library/jest-native
npx expo install axios expo-router
```

#### 2. **Compréhension du Code Exemple**

Ce fichier de test se concentre sur le test du composant `SignIn`, qui inclut des fonctions de simulation et le rendu des composants via **React Testing Library**.

##### Composants concernés :
1. **Composant SignIn** : Un composant React Native qui gère la fonctionnalité de connexion de l'utilisateur.
2. **ThemeProvider** : Fournit un contexte pour gérer le thème de l'application, contrôlant probablement l'apparence des composants.
3. **useRouter** : De `expo-router`, il est utilisé pour la navigation dans une application Expo.

##### Dépendances simulées :
1. **expo-router** : Simuler `useRouter` nous permet de tester la navigation sans effectuer de navigation réelle.
2. **axios (`signIn`)** : Simuler la fonction `signIn` permet de contrôler l'appel réseau et sa réponse.

#### 3. **Simulation des Modules Externes**

Jest permet de simuler des modules et des fonctions pour isoler le comportement du composant testé. Par exemple :
- **Simulation de `useRouter`** :
  ```js
  jest.mock('expo-router', () => ({
    useRouter: jest.fn(),
    Link: jest.fn(({ children }) => <>{children}</>),
  }));
  ```
  Cette simulation permet à `SignIn` d'utiliser une version simulée de `useRouter`, où nous contrôlons la fonction `push` utilisée pour la navigation.

- **Simulation d’`axios`** :
  ```js
  jest.mock('../lib/axios', () => ({
    signIn: jest.fn(),
  }));
  ```
  Cette simulation empêche les appels réels à l'API pendant les tests et nous permet de simuler différentes réponses (succès, erreur).

#### 4. **Provider de Thème Personnalisé pour les Tests**

Ce provider personnalisé (`MockThemeProvider`) permet d'envelopper les composants avec un contexte de thème simulé pendant les tests. Cela garantit que le composant testé reçoit un contexte de thème cohérent sans dépendre du thème réel.

```js
const MockThemeProvider = ({ children }) => {
  const mockTheme = { theme: 'light' }; // Vous pouvez ajuster cela selon votre configuration de thème
  return <ThemeProvider value={mockTheme}>{children}</ThemeProvider>;
};
```

#### 5. **Cas de Test et Assertions**

Les cas de test sont organisés sous `describe` et contiennent différents scénarios :

- **Test de Rendu** :
  Teste si le composant se rend correctement avec les textes essentiels visibles.
  ```js
  it('renders correctly', () => {
    const { getByText } = render(
      <MockThemeProvider>
        <SignIn />
      </MockThemeProvider>
    );
    expect(getByText('ChatMV')).toBeTruthy();
    expect(getByText('Connectez-vous')).toBeTruthy();
  });
  ```

- **Validation des Champs Vides** :
  Simule l'appui sur le bouton "Se connectez" sans entrer de données. Le test vérifie si les messages de validation appropriés sont affichés pour les champs vides.
  ```js
  it('shows validation messages for empty fields', async () => {
    const { getByText, getByPlaceholderText } = render(
      <MockThemeProvider>
        <SignIn />
      </MockThemeProvider>
    );
    fireEvent.press(getByText('Se connectez'));
    expect(getByText('Identifiant : Ce champs doit être rempli')).toBeTruthy();
    expect(getByText('Mot de passe : Ce champs doit être rempli')).toBeTruthy();
  });
  ```

- **Connexion Réussie** :
  Simule une réponse réussie de l'API `signIn` et vérifie si l'utilisateur est redirigé vers la page de profil après avoir soumis des informations valides.
  ```js
  it('handles successful sign-in', async () => {
    signIn.mockResolvedValue({ id: 1 });
    const { getByPlaceholderText, getByText } = render(
      <MockThemeProvider>
        <SignIn />
      </MockThemeProvider>
    );
    fireEvent.changeText(getByPlaceholderText('Entrez l\'identifiant'), 'test@example.com');
    fireEvent.changeText(getByPlaceholderText('Entrez le mot de passe'), 'password123');
    fireEvent.press(getByText('Se connectez'));
    await waitFor(() => {
      expect(mockRouter.push).toHaveBeenCalledWith('../1/profile');
    });
  });
  ```

- **Échec de la Connexion** :
  Simule une réponse d’échec de la connexion (par exemple, identifiants incorrects) et vérifie si le message d'erreur est affiché.
  ```js
  it('handles sign-in failure', async () => {
    signIn.mockRejectedValue(new Error('AxiosError: Request failed with status code 401'));
    const { getByPlaceholderText, getByText } = render(
      <MockThemeProvider>
        <SignIn />
      </MockThemeProvider>
    );
    fireEvent.changeText(getByPlaceholderText('Entrez l\'identifiant'), 'test@example.com');
    fireEvent.changeText(getByPlaceholderText('Entrez le mot de passe'), 'wrongpassword');
    fireEvent.press(getByText('Se connectez'));
    await waitFor(() => {
      expect(getByText('Identifiant ou mot de passe incorrect')).toBeTruthy();
    });
  });
  ```

#### 6. **Concepts de Test Utilisés**

- **`render`** : Cette fonction est utilisée pour rendre le composant et renvoyer des requêtes comme `getByText`, `getByPlaceholderText`, utilisées ensuite pour interagir avec le composant.
- **`fireEvent`** : Simule des actions utilisateur (par exemple, appuyer sur des boutons, entrer du texte).
- **`waitFor`** : Utilisé pour tester des opérations asynchrones. Dans ce cas, il est utilisé pour attendre que la navigation se produise ou qu'un message d'erreur apparaisse après des actions asynchrones.

#### 7. **Bonnes Pratiques pour les Tests**
- **Isolation** : Simulez les dépendances externes comme les requêtes réseau et la navigation pour isoler le comportement du composant testé.
- **Couverture** : Testez à la fois les cas positifs et négatifs, tels que le rendu, la validation, les appels API réussis et échoués.
- **Nettoyage** : Utilisez `jest.clearAllMocks()` pour réinitialiser les simulations entre les tests afin de garantir qu'il n'y a pas de fuite de données entre les cas de tests.

### 8. **Exécution des Tests**

Pour exécuter vos tests, utilisez la commande suivante :
```bash
npm test
```

Cela invoque Jest et lance tous les tests dans votre projet. Assurez-vous que Jest est configuré dans votre fichier `package.json` ou via un fichier de configuration Jest.

---

