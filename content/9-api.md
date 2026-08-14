+++
date = '2026-06-25T15:40:06-04:00'
draft = false
pre="9. "
weight=9
title = 'API'
+++


## Différence entre Bibliothèques Externes et API

**Bibliothèque Externe :**
C'est un package ou un module que vous installez dans votre projet. Il contient des fonctionnalités réutilisables pour réaliser des tâches spécifiques (comme afficher une carte ou utiliser la caméra).
*Exemples : `expo-camera`, `react-native-maps`, `axios`, `redux`.*

**API (Application Programming Interface) :**
C'est un ensemble d'outils ou de services que vous consommez dans votre code, souvent via des requêtes HTTP ou WebSocket. Une API peut être utilisée pour communiquer avec des services externes, comme une base de données distante ou un service de cartographie (par exemple, l'API de Mapbox ou Google Maps).
*Exemples : API de géolocalisation, API de traduction, API météo, API de paiement.*


### En résumé
* **Bibliothèques externes :** Des modules prêts à l'emploi qui ajoutent des fonctionnalités supplémentaires à votre application (accès au matériel, gestion d'état, etc.).
* **API :** Des interfaces qui permettent à votre application de communiquer avec des services externes, d'obtenir des données ou d'interagir avec des systèmes distants.

Les deux concepts sont essentiels dans le développement d'applications modernes, car ils permettent d'étendre rapidement les fonctionnalités sans avoir à réinventer la roue.


![Schéma explicatif API](/420512/images/exo1API-1.png?width=300px)

---

## Une carte

### `react-native-maps`
* Allez voir la documentation en ligne.
* La carte doit être centrée sur **Marie-Victorin**.
* Ajouter un marqueur avec le logo de Marie-Victorin.
* Ajouter un rayon de 5 km autour du marqueur.

---

## Caméra

### `expo-camera`
* Allez voir la documentation en ligne.
* Créez une nouvelle page caméra.
* Elle doit permettre de prendre une photo, de choisir la caméra (avant/arrière) et de retourner à la page précédente.
* Doit enregistrer localement la photo.

---

## Localisation

### `expo-location`
* Allez voir la documentation en ligne.

