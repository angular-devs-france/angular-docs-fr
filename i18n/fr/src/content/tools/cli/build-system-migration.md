<!-- source: e5a3bc8f65f0bdd1d2b701b7d3b3bc90c18a7785 -->
# Système de build des applications Angular

À partir de la v17, le nouveau système de build offre une meilleure façon de compiler les applications Angular. Ce nouveau système de build comprend :

- Un format de sortie moderne basé sur ESM, avec des expressions d'import dynamique pour prendre en charge le lazy loading des modules.
- De meilleures performances au moment du build, aussi bien pour les builds initiaux que pour les rebuilds incrémentaux.
- Des outils plus récents de l'écosystème JavaScript comme [esbuild](https://esbuild.github.io/) et [Vite](https://vitejs.dev/).
- Des fonctionnalités intégrées de SSR et de prérendu.
- Le remplacement à chaud automatique des feuilles de style globales et de composant.

Ce nouveau système de build est stable et entièrement pris en charge pour les applications Angular.
Vous pouvez migrer vers le nouveau système de build les applications qui utilisent le builder `browser`.
Si vous utilisez un builder personnalisé, consultez la documentation de ce builder pour connaître les options de migration possibles.

IMPORTANT: Le système de build existant basé sur webpack et le builder `browser` sont dépréciés.
Les applications peuvent temporairement continuer à utiliser le builder `browser` et les projets peuvent refuser la migration lors d'une mise à jour, mais l'équipe Angular recommande de migrer vers le nouveau système de build.

## Pour les nouvelles applications {#for-new-applications}

Les nouvelles applications utilisent ce nouveau système de build par défaut, via le builder `application`.

## Pour les applications existantes {#for-existing-applications}

Des procédures automatisées et manuelles sont disponibles selon les besoins du projet.
À partir de la v18, le processus de mise à jour vous demande si vous souhaitez migrer les applications existantes vers le nouveau système de build via la migration automatisée.
Avant de migrer, pensez à consulter la section [Problèmes connus](#known-issues), car elle peut contenir des informations utiles pour votre projet.

HELPFUL: Si vous utilisez le SSR, pensez à supprimer du code serveur de l'application toute hypothèse liée à CommonJS, comme `require`, `__filename`, `__dirname` ou d'autres constructions de la [portée de module CommonJS](https://nodejs.org/api/modules.html#the-module-scope). Tout le code de l'application doit être compatible ESM. Cela ne s'applique pas aux dépendances tierces.

### Migration automatisée (recommandée) {#automated-migration-recommended}

La migration automatisée ajuste à la fois la configuration de l'application dans `angular.json` ainsi que le code et les feuilles de style, afin de supprimer l'usage des fonctionnalités propres à webpack.
Bien que de nombreux changements puissent être automatisés et que la plupart des applications ne nécessitent aucun changement supplémentaire, chaque application est unique et certains changements manuels peuvent être nécessaires.
Après la migration, essayez de compiler l'application, car de nouvelles erreurs pourraient nécessiter des ajustements dans le code.
Les erreurs tentent de proposer des solutions au problème lorsque c'est possible, et les sections suivantes de ce guide décrivent certaines des situations les plus courantes que vous pourriez rencontrer.
Lors de la mise à jour vers Angular v18 via `ng update`, il vous sera demandé d'exécuter la migration.
Cette migration est entièrement facultative pour la v18 et peut aussi être exécutée manuellement à tout moment après une mise à jour, via la commande suivante :

```shell

ng update @angular/cli --name use-application-builder

```

La migration effectue les opérations suivantes :

- Convertit la cible `browser` ou `browser-esbuild` existante en `application`
- Supprime les anciens builders SSR (car `application` s'en charge désormais).
- Met à jour la configuration en conséquence.
- Fusionne `tsconfig.server.json` avec `tsconfig.app.json` et ajoute l'option TypeScript `"esModuleInterop": true` pour garantir que les imports d'`express` sont [conformes à ESM](#esm-default-imports-vs-namespace-imports).
- Met à jour le code serveur de l'application pour utiliser le nouveau bootstrap et la nouvelle structure du répertoire de sortie.
- Supprime l'usage des feuilles de style propre aux builders webpack, comme le tilde ou l'accent circonflexe dans `@import`/`url()`, et met à jour la configuration pour obtenir un comportement équivalent
- Bascule vers le nouveau package Node.js `@angular/build`, plus léger en dépendances, si aucun autre usage de `@angular-devkit/build-angular` n'est détecté.

### Migration manuelle {#manual-migration}

Pour les projets existants, vous pouvez aussi choisir manuellement d'utiliser le nouveau builder, application par application, avec deux options différentes.
Les deux options sont considérées comme stables et entièrement prises en charge par l'équipe Angular.
Le choix de l'option dépend du nombre de changements que vous devrez effectuer pour migrer et des nouvelles fonctionnalités que vous souhaitez utiliser dans le projet.

- Le builder `browser-esbuild` compile uniquement le bundle côté client d'une application ; il est conçu pour être compatible avec le builder `browser` existant, qui fournit l'ancien système de build.
  Ce builder propose des options de build équivalentes et, dans de nombreux cas, il remplace directement le builder `browser` dans les applications existantes.
- Le builder `application` couvre une application entière, comme le bundle côté client, et peut aussi compiler un serveur pour le rendu côté serveur et effectuer le prérendu de pages statiques au moment du build.

Le builder `application` est généralement préféré, car il améliore les builds avec rendu côté serveur (SSR) et facilite l'adoption future du SSR pour les projets rendus côté client.
Il demande cependant un peu plus d'effort de migration, en particulier pour les applications SSR existantes si la migration est faite manuellement.
Si le builder `application` est difficile à adopter pour votre projet, `browser-esbuild` peut être une solution plus simple qui apporte la plupart des gains de performances de build avec moins de changements cassants.

#### Migration manuelle vers le builder de compatibilité {#manual-migration-to-the-compatibility-builder}

Un builder nommé `browser-esbuild` est disponible dans le package `@angular-devkit/build-angular`, présent dans toute application générée par l'Angular CLI.
Vous pouvez essayer le nouveau système de build avec les applications qui utilisent le builder `browser`.
Si vous utilisez un builder personnalisé, consultez la documentation de ce builder pour connaître les options de migration possibles.

L'option de compatibilité a été conçue pour réduire au minimum les changements nécessaires à la migration initiale de vos applications.
Elle est fournie via un builder alternatif (`browser-esbuild`).
Vous pouvez mettre à jour la cible `build` de n'importe quelle application pour migrer vers le nouveau système de build.

Voici ce que vous trouverez généralement dans `angular.json` pour une application :

```json
...
"architect": {
  "build": {
    "builder": "@angular-devkit/build-angular:browser",
...
```

Modifier le champ `builder` est le seul changement que vous aurez à faire.

```json
...
"architect": {
  "build": {
    "builder": "@angular-devkit/build-angular:browser-esbuild",
...
```

#### Migration manuelle vers le nouveau builder `application` {#manual-migration-to-the-new-application-builder}

Un builder nommé `application` est également disponible dans le package `@angular-devkit/build-angular`, présent dans toute application générée par l'Angular CLI.
Ce builder est celui par défaut pour toutes les nouvelles applications créées via `ng new`.

Voici ce que vous trouverez généralement dans `angular.json` pour une application :

```json
...
"architect": {
  "build": {
    "builder": "@angular-devkit/build-angular:browser",
...
```

Modifier le champ `builder` est le premier changement que vous aurez à faire.

```json
...
"architect": {
  "build": {
    "builder": "@angular-devkit/build-angular:application",
...
```

Une fois le nom du builder modifié, les options de la cible `build` devront être mises à jour.
La liste suivante présente toutes les options du builder `browser` qui devront être ajustées.

- `main` doit être renommée en `browser`.
- `polyfills` doit être un tableau, et non un fichier unique.
- `buildOptimizer` doit être supprimée, car elle est couverte par l'option `optimization`.
- `resourcesOutputPath` doit être supprimée : la valeur est désormais toujours `media`.
- `vendorChunk` doit être supprimée, car il s'agissait d'une optimisation de performances qui n'est plus nécessaire.
- `commonChunk` doit être supprimée, car il s'agissait d'une optimisation de performances qui n'est plus nécessaire.
- `ngswConfigPath` doit être renommée en `serviceWorker`.

Si l'application n'utilise pas encore le SSR, cette étape devrait être la dernière pour que `ng build` fonctionne.
Après la première exécution de `ng build`, de nouveaux avertissements ou de nouvelles erreurs peuvent apparaître en raison de différences de comportement ou de l'usage dans l'application de fonctionnalités propres à webpack.
Beaucoup de ces avertissements proposent des pistes pour résoudre le problème.
Si un avertissement semble incorrect ou si la solution n'est pas évidente, ouvrez une issue sur [GitHub](https://github.com/angular/angular-cli/issues).
Les sections suivantes de ce guide fournissent aussi des informations complémentaires sur plusieurs cas spécifiques ainsi que sur les problèmes connus actuels.

Pour les applications qui découvrent le SSR, le [guide sur le rendu serveur et hybride](guide/ssr) fournit des informations complémentaires sur la mise en place du SSR dans une application.

Pour les applications qui utilisent déjà le SSR, des ajustements supplémentaires seront nécessaires pour mettre à jour le serveur de l'application afin de prendre en charge les nouvelles fonctionnalités SSR intégrées.
Le builder `application` fournit désormais la fonctionnalité intégrée de tous les builders préexistants suivants :

- `app-shell`
- `prerender`
- `server`
- `ssr-dev-server`

Le processus `ng update` supprime automatiquement les usages des packages du scope `@nguniversal`, où se trouvaient auparavant certains de ces builders.
Le nouveau package `@angular/ssr` est aussi ajouté et utilisé automatiquement, la configuration et le code étant ajustés pendant la mise à jour.
Le package `@angular/ssr` prend en charge aussi bien le builder `browser` que le builder `application`.

## Lancer un build {#executing-a-build}

Une fois la configuration de l'application mise à jour, les builds peuvent être lancés avec `ng build`, comme auparavant.
Selon le builder choisi pour la migration, certaines options de ligne de commande peuvent être différentes.
Si la commande de build figure dans des scripts `npm` ou autres, veillez à les vérifier et à les mettre à jour.
Pour les applications qui ont migré vers le builder `application` et qui utilisent le SSR et/ou le prérendu, vous pourrez peut-être aussi supprimer des scripts les commandes `ng run` supplémentaires, maintenant que `ng build` intègre la prise en charge du SSR.

```shell

ng build

```

## Démarrer le serveur de développement {#starting-the-development-server}

Le serveur de développement détecte automatiquement le nouveau système de build et l'utilise pour compiler l'application.
Pour démarrer le serveur de développement, aucun changement n'est nécessaire dans la configuration du builder `dev-server` ni dans la ligne de commande.

```shell

ng serve

```

Vous pouvez continuer à utiliser avec le serveur de développement les [options de ligne de commande](/cli/serve) que vous utilisiez auparavant.

HELPFUL: Avec le serveur de développement, vous pouvez observer un léger flash de contenu non stylé (FOUC, Flash of Unstyled Content) au démarrage, pendant l'initialisation du serveur.
Le serveur de développement essaie de différer le traitement des feuilles de style jusqu'à leur première utilisation afin d'améliorer les temps de rebuild.
Cela ne se produit pas dans les builds effectués en dehors du serveur de développement.

### Hot module replacement {#hot-module-replacement}

Le Hot Module Replacement (HMR) est une technique utilisée par les serveurs de développement pour éviter de recharger toute la page lorsque seule une partie de l'application est modifiée.
Dans de nombreux cas, les modifications peuvent s'afficher immédiatement dans le navigateur, ce qui améliore le cycle modification/rafraîchissement pendant le développement d'une application.
Bien que le hot module replacement (HMR) général basé sur JavaScript ne soit pas pris en charge pour le moment, plusieurs formes plus spécifiques de HMR sont disponibles :

- **feuille de style globale** (option de build `styles`)
- **feuille de style de composant** (inline et dans un fichier)
- **template de composant** (inline et dans un fichier)

Les fonctionnalités de HMR sont activées automatiquement et ne nécessitent aucun changement de code ni de configuration.
Angular prend en charge le HMR pour les styles et templates de composant, qu'ils soient dans un fichier (`templateUrl`/`styleUrl`/`styleUrls`) ou inline (`template`/`styles`).
Le système de build tente de compiler et de traiter le minimum de code de l'application lorsqu'il détecte une modification qui ne concerne qu'une feuille de style.

Si vous le souhaitez, les fonctionnalités de HMR peuvent être désactivées en définissant l'option `hmr` du serveur de développement à `false`.
Cela peut aussi être modifié en ligne de commande via :

```shell

ng serve --no-hmr

```

### Vite comme serveur de développement {#vite-as-a-development-server}

Dans l'Angular CLI, Vite est actuellement utilisé _uniquement en tant que serveur de développement_. Même sans utiliser le système de build sous-jacent de Vite, Vite fournit un serveur de développement complet avec une prise en charge côté client, regroupé dans un package npm léger en dépendances. Cela en fait un candidat idéal pour fournir des fonctionnalités complètes de serveur de développement. Le processus actuel du serveur de développement utilise le nouveau système de build pour générer en mémoire un build de développement de l'application, puis transmet les résultats à Vite pour servir l'application. L'usage de Vite, tout comme le serveur de développement basé sur Webpack, est encapsulé dans le builder `dev-server` de l'Angular CLI et ne peut pas être configuré directement pour le moment.

### Prebundling {#prebundling}

Le prebundling améliore les temps de build et de rebuild lors de l'utilisation du serveur de développement.
Vite fournit des [fonctionnalités de prebundling](https://vite.dev/guide/dep-pre-bundling) activées par défaut avec l'Angular CLI.
Le processus de prebundling analyse toutes les dépendances tierces d'un projet et les traite lors de la première exécution du serveur de développement.
Ce processus évite de recompiler et de regrouper en bundles les dépendances du projet à chaque rebuild ou à chaque exécution du serveur de développement.

Dans la plupart des cas, aucune personnalisation supplémentaire n'est nécessaire. Elle peut toutefois l'être dans certaines situations, notamment :

- Personnaliser le comportement du loader pour les imports au sein de la dépendance, par exemple avec l'[option `loader`](#file-extension-loader-customization)
- Créer un lien symbolique d'une dépendance vers du code local pour le développement, par exemple avec [`npm link`](https://docs.npmjs.com/cli/v10/commands/npm-link)
- Contourner une erreur rencontrée pendant le prebundling d'une dépendance

Le processus de prebundling peut être entièrement désactivé, ou certaines dépendances peuvent en être exclues si un projet en a besoin.
L'option `prebundle` du builder `dev-server` permet ces personnalisations.
Pour exclure des dépendances spécifiques, l'option `prebundle.exclude` est disponible :

```json
    "serve": {
      "builder": "@angular/build:dev-server",
      "options": {
        "prebundle": {
          "exclude": ["some-dep"]
        }
      },
```

Par défaut, `prebundle` vaut `true`, mais peut être défini à `false` pour désactiver entièrement le prebundling.
Il est toutefois recommandé d'exclure plutôt des dépendances spécifiques, car les temps de rebuild augmentent lorsque le prebundling est désactivé.

```json
    "serve": {
      "builder": "@angular/build:dev-server",
      "options": {
        "prebundle": false
      },
```

## Nouvelles fonctionnalités {#new-features}

L'un des principaux avantages du système de build des applications est l'amélioration de la vitesse de build et de rebuild.
Le nouveau système de build des applications fournit cependant aussi des fonctionnalités supplémentaires absentes du builder `browser`.

IMPORTANT: Les nouvelles fonctionnalités du builder `application` décrites ici sont incompatibles par défaut avec le builder de test `karma`, car celui-ci utilise le builder `browser` en interne.
Vous pouvez choisir d'utiliser le builder `application` en définissant l'option `builderMode` à `application` pour le builder `karma`.
Cette option est actuellement en developer preview.
Si vous rencontrez des problèmes, signalez-les [ici](https://github.com/angular/angular-cli/issues).

### Remplacement de valeurs au moment du build avec `define` {#build-time-value-replacement-with-define}

L'option `define` permet de remplacer, au moment du build, des identifiants présents dans le code par une autre valeur.
Ce comportement est similaire à celui du `DefinePlugin` de Webpack, utilisé auparavant avec certaines configurations Webpack personnalisées s'appuyant sur des builders tiers.
L'option peut être utilisée soit dans le fichier de configuration `angular.json`, soit en ligne de commande.
Configurer `define` dans `angular.json` est utile lorsque les valeurs sont constantes et peuvent être versionnées dans le gestionnaire de sources.

Dans le fichier de configuration, l'option prend la forme d'un objet.
Les clés de l'objet représentent l'identifiant à remplacer et les valeurs de l'objet représentent la valeur de remplacement correspondante pour cet identifiant.
En voici un exemple :

```json
  "build": {
    "builder": "@angular/build:application",
    "options": {
      ...
      "define": {
          "SOME_NUMBER": "5",
          "ANOTHER": "'this is a string literal, note the extra single quotes'",
          "REFERENCE": "globalThis.someValue.noteTheAbsentSingleQuotes"
      }
    }
  }
```

HELPFUL: Toutes les valeurs de remplacement sont définies sous forme de chaînes dans le fichier de configuration.
Si le remplacement doit être un véritable littéral de chaîne, il doit être entouré d'apostrophes simples.
Cela offre la flexibilité d'utiliser n'importe quel type JSON valide, ou un autre identifiant, comme valeur de remplacement.

L'usage en ligne de commande est préférable pour les valeurs susceptibles de changer à chaque build, comme le hash du commit git ou une variable d'environnement.
La CLI fusionne les valeurs `--define` de la ligne de commande avec les valeurs `define` d'`angular.json` et inclut les deux dans le build.
L'usage en ligne de commande est prioritaire si le même identifiant est présent dans les deux.
En ligne de commande, l'option `--define` utilise le format `IDENTIFIER=VALUE`.

```shell
ng build --define SOME_NUMBER=5 --define "ANOTHER='these will overwrite existing'"
```

Les variables d'environnement peuvent aussi être incluses de manière sélective dans un build.
Pour les shells autres que Windows, les apostrophes autour du littéral de hash peuvent être échappées directement si vous le préférez.
Cet exemple suppose un shell de type bash, mais un comportement similaire est disponible pour les autres shells.

```shell
export MY_APP_API_HOST="http://example.com"
export API_RETRY=3
ng build --define API_HOST=\'$MY_APP_API_HOST\' --define API_RETRY=$API_RETRY
```

Dans les deux cas, TypeScript doit connaître les types des identifiants pour éviter les erreurs de vérification de types pendant le build.
Pour cela, ajoutez un fichier de définitions de types dans le code source de l'application (`src/types.d.ts`, par exemple) avec un contenu similaire :

```ts
declare const SOME_NUMBER: number;
declare const ANOTHER: string;
declare const GIT_HASH: string;
declare const API_HOST: string;
declare const API_RETRY: number;
```

La configuration de projet par défaut est déjà prévue pour utiliser tous les fichiers de définitions de types présents dans les répertoires source du projet.
Si la configuration TypeScript du projet a été modifiée, elle devra peut-être être ajustée pour référencer ce nouveau fichier de définitions de types.

IMPORTANT: Cette option ne remplace pas les identifiants contenus dans les métadonnées Angular, comme un décorateur Component ou Directive.

### Personnalisation du loader par extension de fichier {#file-extension-loader-customization}

IMPORTANT: Cette fonctionnalité est disponible uniquement avec le builder `application`.

Certains projets peuvent avoir besoin de contrôler la façon dont tous les fichiers d'une extension donnée sont chargés et intégrés aux bundles d'une application.
Avec le builder `application`, l'option `loader` permet de gérer ces cas.
Elle permet à un projet de définir le type de loader à utiliser pour une extension de fichier donnée.
Un fichier portant l'extension définie peut alors être utilisé dans le code de l'application via une instruction d'import ou une expression d'import dynamique.
Les loaders disponibles sont :

- `text` : intègre le contenu sous forme de `string`, disponible comme export par défaut
- `binary` : intègre le contenu sous forme de `Uint8Array`, disponible comme export par défaut
- `file` : émet le fichier dans le chemin de sortie de l'application et fournit l'emplacement du fichier à l'exécution comme export par défaut
- `dataurl` : intègre le contenu sous forme de [data URL](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/Data_URIs).
- `base64` : intègre le contenu sous forme de chaîne encodée en Base64.
- `empty` : considère le contenu comme vide et ne l'inclut pas dans les bundles

La valeur `empty`, bien que moins courante, peut être utile pour la compatibilité avec des libraries tierces contenant des imports propres à un bundler qui doivent être supprimés.
C'est le cas par exemple des imports à effet de bord (`import 'my.css';`) de fichiers CSS, qui n'ont aucun effet dans un navigateur.
Le projet peut alors utiliser `empty`, puis ajouter les fichiers CSS à l'option de build `styles` ou utiliser une autre méthode d'injection.

L'option loader est un objet dont les clés définissent l'extension de fichier et les valeurs définissent le type de loader.

Voici un exemple d'utilisation de cette option de build pour intégrer le contenu des fichiers SVG dans l'application regroupée en bundles :

```json
  "build": {
    "builder": "@angular/build:application",
    "options": {
      ...
      "loader": {
        ".svg": "text"
      }
    }
  }
```

Un fichier SVG peut alors être importé :

```ts
import contents from './some-file.svg';

console.log(contents); // <svg>...</svg>
```

De plus, TypeScript doit connaître le type de module de l'import pour éviter les erreurs de vérification de types pendant le build. Pour cela, ajoutez un fichier de définitions de types dans le code source de l'application (`src/types.d.ts`, par exemple) avec le contenu suivant ou un contenu similaire :

```ts
declare module '*.svg' {
  const content: string;
  export default content;
}
```

La configuration de projet par défaut est déjà prévue pour utiliser tous les fichiers de définitions de types (fichiers `.d.ts`) présents dans les répertoires source du projet. Si la configuration TypeScript du projet a été modifiée, le tsconfig devra peut-être être ajusté pour référencer ce nouveau fichier de définitions de types.

### Personnalisation du loader par attribut d'import {#import-attribute-loader-customization}

Lorsque seuls certains fichiers doivent être chargés d'une manière spécifique, un contrôle du chargement fichier par fichier est disponible.
Il s'appuie sur un [attribut d'import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import/with) `loader`, utilisable à la fois avec les instructions et les expressions d'import.
La présence de l'attribut d'import est prioritaire sur tout autre comportement de chargement, y compris JS/TS et les valeurs de l'option de build `loader`.
Pour un chargement général de tous les fichiers d'un type autrement non pris en charge, l'option de build [`loader`](#file-extension-loader-customization) est recommandée.

Pour l'attribut d'import, les valeurs de loader suivantes sont prises en charge :

- `text` : intègre le contenu sous forme de `string`, disponible comme export par défaut
- `binary` : intègre le contenu sous forme de `Uint8Array`, disponible comme export par défaut
- `file` : émet le fichier dans le chemin de sortie de l'application et fournit l'emplacement du fichier à l'exécution comme export par défaut
- `dataurl` : intègre le contenu sous forme de [data URL](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/Data_URIs).
- `base64` : intègre le contenu sous forme de chaîne encodée en Base64.

Pour utiliser les attributs d'import, l'option TypeScript `module` doit en outre être définie à `esnext` afin que TypeScript puisse compiler correctement le code de l'application.
Une fois `ES2025` disponible dans TypeScript, ce changement ne sera plus nécessaire.

Pour le moment, TypeScript ne prend pas en charge les définitions de types basées sur les valeurs des attributs d'import.
L'utilisation de `@ts-expect-error`/`@ts-ignore` ou de fichiers de définitions de types individuels (à condition que le fichier soit toujours importé avec le même attribut de loader) est actuellement nécessaire.
Par exemple, un fichier SVG peut être importé en tant que texte via :

```ts
// @ts-expect-error TypeScript cannot provide types based on attributes yet
import contents from './some-file.svg' with {loader: 'text'};
```

Le même résultat peut être obtenu avec une expression d'import dans une fonction asynchrone.

```ts
async function loadSvg(): Promise<string> {
  // @ts-expect-error TypeScript cannot provide types based on attributes yet
  return import('./some-file.svg', {with: {loader: 'text'}}).then((m) => m.default);
}
```

Pour l'expression d'import, la valeur de `loader` doit être un littéral de chaîne afin de pouvoir être analysée statiquement.
Un avertissement est émis si la valeur n'est pas un littéral de chaîne.

Le loader `file` est utile lorsqu'un fichier est chargé à l'exécution via `fetch()`, en l'affectant au `src` d'un élément image, ou par une autre méthode similaire.

```ts
// @ts-expect-error TypeScript cannot provide types based on attributes yet
import imagePath from './image.webp' with {loader: 'file'};

console.log(imagePath); // media/image-ULK2SIIB.webp
```

Le loader `base64` est utile lorsqu'un fichier doit être intégré directement dans le bundle sous forme de chaîne encodée, qui pourra ensuite servir à construire une Data URL.

```ts
// @ts-expect-error TypeScript cannot provide types based on attributes yet
import logo from './logo.png' with {loader: 'base64'};

console.log(logo); // "iVBORw0KGgoAAAANSUhEUgAA..."
```

Le loader `dataurl` permet d'intégrer des ressources sous forme de Data URL complètes.

```ts
// @ts-expect-error TypeScript cannot provide types based on attributes yet
import icon from './icon.svg' with {loader: 'dataurl'};

console.log(icon); // "data:image/svg+xml;..."
```

Pour les builds de production, comme le montre le commentaire de code ci-dessus, un hash est automatiquement ajouté au chemin pour permettre une mise en cache de longue durée.

HELPFUL: Lorsque vous utilisez le serveur de développement avec un attribut `loader` pour importer un fichier depuis un package Node.js, ce package doit être exclu du prebundling via l'option `prebundle` du serveur de développement.

### Conditions d'import/export {#import-export-conditions}

Les projets peuvent avoir besoin de faire correspondre certains chemins d'import à des fichiers différents selon le type de build.
C'est particulièrement utile lorsque, par exemple, `ng serve` doit utiliser du code spécifique au débogage ou au développement, alors que `ng build` doit utiliser du code sans aucune fonctionnalité ni information de développement.
Plusieurs [conditions](https://nodejs.org/api/packages.html#community-conditions-definitions) d'import/export sont appliquées automatiquement pour répondre à ces besoins :

- Pour les builds optimisés, la condition `production` est activée.
- Pour les builds non optimisés, la condition `development` est activée.
- Pour le code de sortie destiné au navigateur, la condition `browser` est activée.

Un build optimisé est déterminé par la valeur de l'option `optimization`.
Lorsque `optimization` vaut `true`, ou plus précisément lorsque `optimization.scripts` vaut `true`, le build est considéré comme optimisé.
Cette classification s'applique à la fois à `ng build` et à `ng serve`.
Dans un nouveau projet, `ng build` est optimisé par défaut et `ng serve` est non optimisé par défaut.

Une manière pratique de tirer parti de ces conditions dans le code de l'application consiste à les combiner avec les [subpath imports](https://nodejs.org/api/packages.html#subpath-imports).
Avec l'instruction d'import suivante :

```ts
import {verboseLogging} from '#logger';
```

Le fichier peut être basculé dans le champ `imports` de `package.json` :

```json
{
  ...
  "imports": {
    "#logger": {
      "development": "./src/logging/debug.ts",
      "default": "./src/logging/noop.ts"
    }
  }
}
```

Pour les applications qui utilisent aussi le SSR, le code navigateur et le code serveur peuvent être basculés grâce à la condition `browser` :

```json
{
  ...
  "imports": {
    "#crashReporter": {
      "browser": "./src/browser-logger.ts",
      "default": "./src/server-logger.ts"
    }
  }
}
```

Ces conditions s'appliquent aussi aux packages Node.js et à tous les [`exports`](https://nodejs.org/api/packages.html#conditional-exports) définis dans ces packages.

HELPFUL: Si vous utilisez actuellement l'option de build `fileReplacements`, cette fonctionnalité pourra peut-être la remplacer.

## Problèmes connus {#known-issues}

Il existe actuellement plusieurs problèmes connus que vous pourriez rencontrer en essayant le nouveau système de build. Cette liste sera mise à jour pour rester à jour. Si l'un de ces problèmes vous empêche actuellement d'essayer le nouveau système de build, revenez plus tard, car il aura peut-être été résolu.

### Vérification de types du code des Web Workers et traitement des Web Workers imbriqués {#type-checking-of-web-worker-code-and-processing-of-nested-web-workers}

Les Web Workers peuvent être utilisés dans le code de l'application avec la même syntaxe (`new Worker(new URL('<workerfile>', import.meta.url))`) que celle prise en charge par le builder `browser`.
Cependant, le code du Worker n'est pas encore soumis à la vérification de types du compilateur TypeScript. Le code TypeScript est pris en charge, mais sans vérification de types.
De plus, les workers imbriqués ne sont pas traités par le système de build. Un worker imbriqué est un Worker instancié dans un autre fichier de Worker.

### Imports par défaut ESM et imports de namespace {#esm-default-imports-vs-namespace-imports}

Par défaut, TypeScript permet d'importer les exports par défaut sous forme d'imports de namespace, puis de les utiliser dans des expressions d'appel.
Il s'agit malheureusement d'un écart par rapport à la spécification ECMAScript.
Le bundler sous-jacent (`esbuild`) du nouveau système de build attend du code ESM conforme à la spécification.
Le système de build génère désormais un avertissement si votre application utilise un type d'import incorrect pour un package.
Cependant, pour que TypeScript accepte l'usage correct, une option TypeScript doit être activée dans le fichier `tsconfig` de l'application.
Une fois activée, l'option [`esModuleInterop`](https://www.typescriptlang.org/tsconfig#esModuleInterop) assure un meilleur alignement avec la spécification ECMAScript ; elle est d'ailleurs recommandée par l'équipe TypeScript.
Une fois l'option activée, vous pouvez mettre à jour les imports de packages concernés vers une forme conforme à ECMAScript.

En prenant le package [`moment`](https://npmjs.com/package/moment) comme exemple, le code d'application suivant provoque des erreurs à l'exécution :

```ts
import * as moment from 'moment';

console.log(moment().format());
```

Le build génère un avertissement pour vous signaler un problème potentiel. L'avertissement ressemble à ceci :

```text
▲ [WARNING] Calling "moment" will crash at run-time because it's an import namespace object, not a function [call-import-namespace]

    src/main.ts:2:12:
      2 │ console.log(moment().format());
        ╵             ~~~~~~

Consider changing "moment" to a default import instead:

    src/main.ts:1:7:
      1 │ import * as moment from 'moment';
        │        ~~~~~~~~~~~
        ╵        moment

```

Vous pouvez cependant éviter les erreurs à l'exécution et l'avertissement en activant l'option TypeScript `esModuleInterop` pour l'application et en modifiant l'import comme suit :

```ts
import moment from 'moment';

console.log(moment().format());
```

### Imports à effets de bord dépendants de l'ordre dans les modules lazy {#order-dependent-side-effectful-imports-in-lazy-modules}

Les instructions d'import qui dépendent d'un ordre précis et qui sont aussi utilisées dans plusieurs modules chargés en lazy loading peuvent entraîner l'exécution dans le désordre des instructions de premier niveau.
Ce cas est peu courant, car il dépend de l'usage de modules à effets de bord, et il ne s'applique pas à l'option `polyfills`.
Il est causé par un [défaut](https://github.com/evanw/esbuild/issues/399) du bundler sous-jacent, qui sera corrigé dans une future mise à jour.

IMPORTANT: Éviter autant que possible les modules ayant des effets de bord non locaux (en dehors des polyfills) est recommandé, quel que soit le système de build utilisé, et permet d'éviter ce problème particulier. Les modules ayant des effets de bord non locaux peuvent aussi avoir un impact négatif sur la taille de l'application et sur les performances à l'exécution.

### Changements de l'emplacement de sortie {#output-location-changes}

Par défaut, après un build réussi avec le builder application, le bundle se trouve dans un répertoire `dist/<project-name>/browser` (au lieu de `dist/<project-name>` avec le builder browser).
Cela peut casser certaines chaînes d'outils qui dépendent de l'ancien emplacement. Dans ce cas, vous pouvez [configurer le chemin de sortie](reference/configs/workspace-config#output-path-configuration) selon vos besoins.

## Signalement de bugs {#bug-reports}

Signalez les problèmes et les demandes de fonctionnalités sur [GitHub](https://github.com/angular/angular-cli/issues).

Fournissez si possible une reproduction minimale pour aider l'équipe à résoudre les problèmes.
