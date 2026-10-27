<!-- source: fa21d3682bbac932b4cd776ebb9b8d4e39d6f4de -->
# Profilage avec Chrome DevTools

Angular s'intègre à l'[API d'extensibilité de Chrome DevTools](https://developer.chrome.com/docs/devtools/performance/extension) pour présenter des données et des analyses propres au framework directement dans le [panneau Performance de Chrome DevTools](https://developer.chrome.com/docs/devtools/performance/overview).

Une fois l'intégration activée, vous pouvez [enregistrer un profil de performances](https://developer.chrome.com/docs/devtools/performance#record) contenant deux ensembles de données :

- les entrées de performances standard, fondées sur la compréhension qu'a Chrome de l'exécution de votre code dans un navigateur, et
- les entrées propres à Angular, fournies par le runtime du framework.

Les deux ensembles de données sont présentés ensemble dans le même onglet, mais sur des pistes distinctes :

<img alt="Piste personnalisée Angular dans le profiler de Chrome DevTools" src="assets/images/best-practices/runtime-performance/angular-perf-in-chrome.png">

Les données propres à Angular sont exprimées en termes de concepts du framework (composants, détection de changements, lifecycle hooks, etc.), à côté des appels de fonctions et de méthodes de plus bas niveau capturés par le navigateur. Ces deux ensembles de données sont corrélés, et vous pouvez passer d'une vue et d'un niveau de détail à l'autre.

Vous pouvez utiliser la piste Angular pour mieux comprendre comment votre code s'exécute dans le navigateur, notamment pour :

- Déterminer si un bloc de code donné fait partie de l'application Angular ou s'il appartient à un autre script exécuté sur la même page.
- Identifier les goulots d'étranglement de performances et les attribuer à des composants ou des services précis.
- Mieux comprendre le fonctionnement interne d'Angular grâce à une décomposition visuelle de chaque cycle de détection de changements.

## Enregistrer un profil {#recording-a-profile}

### Activer l'intégration {#enable-integration}

Vous pouvez activer le profilage Angular de deux manières :

1. Exécutez [`ng.enableProfiling()`](api/core/enableProfiling) dans le panneau Console de Chrome, ou
1. Ajoutez un appel à [`enableProfiling()`](api/core/enableProfiling) dans le code de démarrage de votre application (importé depuis `@angular/core`).

NOTE: Le profilage Angular fonctionne uniquement en mode développement.

Voici un exemple montrant comment activer l'intégration lors du bootstrap de l'application pour capturer tous les événements possibles :

```ts
import {enableProfiling} from '@angular/core';
import {bootstrapApplication} from '@angular/platform-browser';
import {MyApp} from './my-app';

// Turn on profiling *before* bootstrapping your application
// in order to capture all of the code run on start-up.
enableProfiling();
bootstrapApplication(MyApp);
```

### Enregistrer un profil {#record-a-profile}

Utilisez le bouton **Record** du panneau Performance de Chrome DevTools :

<img alt="Enregistrement d'un profil" src="assets/images/best-practices/runtime-performance/recording-profile-in-chrome.png">

Consultez la [documentation de Chrome DevTools](https://developer.chrome.com/docs/devtools/performance#record) pour plus de détails sur l'enregistrement des profils.

## Ouvrir un composant dans Angular DevTools {#open-a-component-in-angular-devtools}

Après avoir enregistré un profil, sélectionnez un événement de composant dans la piste **Angular**.
L'onglet **Summary** peut contenir un lien **Component** qui utilise le schéma d'URL `angular-devtools://component/...`.

<img alt="Panneau Performance de Chrome DevTools affichant une piste personnalisée Angular avec un événement _MainComponent sélectionné. L'onglet Summary affiche un lien Component qui utilise le schéma d'URL angular-devtools://component." src="assets/images/best-practices/runtime-performance/chrome-performance-deep-link.png">

Cliquez sur le lien pour ouvrir Angular DevTools et sélectionner le composant correspondant dans l'onglet **Components**.
Cela vous permet de passer d'un profil au niveau du navigateur à l'état et aux métadonnées du composant pour un événement sélectionné.

NOTE: L'ouverture des liens de composants nécessite Angular DevTools pour Chrome et le flag expérimental `chrome://flags/#enable-devtools-deep-link-via-extensibility-api` de Chrome.

## Interpréter un profil enregistré {#interpreting-a-recorded-profile}

Vous pouvez utiliser la piste personnalisée « Angular » pour identifier et diagnostiquer rapidement les problèmes de performances. Les sections suivantes décrivent quelques scénarios de profilage courants.

### Distinguer votre application Angular des autres tâches de la même page {#differentiating-between-your-angular-application-and-other-tasks-on-the-same-page}

Comme les données d'Angular et de Chrome sont présentées sur des pistes distinctes mais corrélées, vous pouvez voir quand le code de l'application Angular s'exécute, par opposition à d'autres traitements du navigateur (généralement la mise en page et le rendu graphique) ou à d'autres scripts exécutés sur la même page (dans ce cas, la piste personnalisée Angular ne contient aucune donnée) :

<img alt="Données de profil : exécution d'Angular et de scripts tiers" src="assets/images/best-practices/runtime-performance/profile-angular-vs-3rd-party.png">

Cela vous permet de déterminer si les investigations suivantes doivent porter sur le code de l'application Angular ou sur d'autres parties de votre base de code ou de vos dépendances.

### Code couleur {#color-coding}

Angular utilise des couleurs dans le flame chart pour distinguer les types de tâches :

- 🟦 Le bleu représente le code TypeScript écrit par le développeur de l'application (par exemple : services, constructeurs de composants, lifecycle hooks, etc.).
- 🟪 Le violet représente le code des templates écrit par le développeur de l'application et transformé par le compilateur Angular.
- 🟩 Le vert représente les points d'entrée dans le code de l'application et identifie les _raisons_ de l'exécution du code.

Les exemples suivants illustrent ce code couleur dans divers enregistrements réels.

#### Exemple : bootstrap de l'application {#example-application-bootstrapping}

Le processus de bootstrap de l'application se compose généralement :

- de déclencheurs marqués en bleu, comme l'appel à `bootstrapApplication`, l'instanciation du composant racine et la détection de changements initiale
- de divers services DI instanciés pendant le bootstrap, marqués en vert.

<img alt="Données de profil : bootstrap de l'application" src="assets/images/best-practices/runtime-performance/profile-bootstrap-application.png">

#### Exemple : exécution d'un composant {#example-component-execution}

Le traitement d'un composant est généralement représenté par un point d'entrée (bleu) suivi de l'exécution de son template (violet). Un template peut à son tour déclencher l'instanciation de directives et l'exécution de lifecycle hooks (vert) :

<img alt="Données de profil : traitement d'un composant" src="assets/images/best-practices/runtime-performance/profile-component-processing.png">

#### Exemple : détection de changements {#example-change-detection}

Un cycle de détection de changements se compose généralement d'une ou plusieurs passes de synchronisation des données (bleu), chaque passe parcourant un sous-ensemble de composants.

<img alt="Données de profil : détection de changements" src="assets/images/best-practices/runtime-performance/profile-change-detection.png">

Grâce à cette visualisation des données, il est possible d'identifier immédiatement les composants impliqués dans la détection de changements et ceux qui ont été ignorés (généralement les composants `OnPush` qui n'ont pas été marqués comme modifiés (dirty)).

De plus, vous pouvez examiner le nombre de passes de synchronisation pour une détection de changements. Plus d'une passe de synchronisation indique que l'état est mis à jour pendant la détection de changements. Évitez cela, car cela ralentit les mises à jour de la page et peut même provoquer des boucles infinies dans le pire des cas.
