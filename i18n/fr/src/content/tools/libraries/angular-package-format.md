<!-- source: 553af5b1e1559a1f69eaca0b18f2927f444031fe -->
# Angular Package Format

Ce document décrit l'Angular Package Format \(APF\).
L'APF est une spécification propre à Angular pour la structure et le format des packages npm. Elle est utilisée par tous les packages Angular officiels \(`@angular/core`, `@angular/material`, etc.\) et par la plupart des libraries Angular tierces.

L'APF permet à un package de fonctionner sans accroc dans la plupart des scénarios courants qui utilisent Angular.
Les packages qui utilisent l'APF sont compatibles avec l'outillage proposé par l'équipe Angular ainsi qu'avec l'écosystème JavaScript au sens large.
Il est recommandé aux développeurs de libraries tierces de suivre le même format de package npm.

HELPFUL: L'APF est versionné en même temps que le reste d'Angular, et chaque version majeure améliore le format de package.
Vous trouverez les versions de la spécification antérieures à la v13 dans ce [document Google](https://docs.google.com/document/d/1CZC2rcpxffTDfRDs6p1cfbmKNLA6x5O-NtkJglDaBVs/preview).

## Pourquoi spécifier un format de package ? {#why-specify-a-package-format}

Dans le paysage JavaScript actuel, les développeurs consomment des packages de nombreuses façons, avec de nombreuses chaînes d'outils différentes \(webpack, Rollup, Rolldown, esbuild, etc.\).
Ces outils peuvent comprendre et exiger des entrées différentes : certains outils savent traiter la dernière version du langage ES, tandis que d'autres gagnent à consommer directement une version ES plus ancienne.

Le format de distribution d'Angular prend en charge tous les outils et workflows de développement couramment utilisés, et met l'accent sur les optimisations qui produisent soit une application plus légère, soit un cycle d'itération de développement plus rapide \(temps de build\).

Les développeurs peuvent s'appuyer sur l'Angular CLI et sur [ng-packagr](https://github.com/ng-packagr/ng-packagr) \(un outil de build qu'utilise l'Angular CLI\) pour produire des packages au format Angular Package Format.
Consultez le guide [Créer des libraries](tools/libraries/creating-libraries) pour plus de détails.

## Organisation des fichiers {#file-layout}

L'exemple suivant montre une version simplifiée de l'organisation des fichiers du package `@angular/core`, avec une explication pour chaque fichier du package.

```markdown
node_modules/@angular/core
├── README.md
├── package.json
├── fesm2022
│ ├── core.mjs
│ ├── core.mjs.map
│ ├── testing.mjs
│ └── testing.mjs.map
└── types
│ ├── core.d.ts
│ ├── testing.d.ts
```

Ce tableau décrit l'organisation des fichiers sous `node_modules/@angular/core`, en précisant le rôle des fichiers et des répertoires :

| Fichiers                                                                                                                                                  | Rôle                                                                                                                                                                                                           |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `README.md`                                                                                                                                               | README du package, utilisé par l'interface web de npmjs.                                                                                                                                                      |
| `package.json`                                                                                                                                            | `package.json` principal, qui décrit le package lui-même ainsi que tous les points d'entrée et formats de code disponibles. Ce fichier contient le mapping « exports » utilisé par les runtimes et les outils pour effectuer la résolution de modules. |
| `fesm2022/` <br /> &nbsp;&nbsp;─ `core.mjs` <br /> &nbsp;&nbsp;─ `core.mjs.map` <br /> &nbsp;&nbsp;─ `testing.mjs` <br /> &nbsp;&nbsp;─ `testing.mjs.map` | Le code de tous les points d'entrée au format ES2022 aplati \(FESM\), accompagné des source maps.                                                                                                              |
| `types/` <br /> &nbsp;&nbsp;─ `core.d.ts` <br /> &nbsp;&nbsp;─ `testing.d.ts`                                                                             | Les définitions de types TypeScript bundlées pour tous les points d'entrée publics.                                                                                                                            |

### `package.json`

Le `package.json` principal contient des métadonnées importantes sur le package, notamment :

- Il [déclare](#esm-declaration) que le package est au format EcmaScript Module \(ESM\)
- Il contient un champ `"exports"` qui définit les formats de code source disponibles pour tous les points d'entrée
- Il contient des [clés](#legacy-resolution-keys) qui définissent les formats de code source disponibles pour le point d'entrée principal `@angular/core`, à destination des outils qui ne comprennent pas `"exports"`.
  Ces clés sont considérées comme dépréciées et pourraient être supprimées à mesure que la prise en charge de `"exports"` se généralise dans l'écosystème.

- Il déclare si le package contient des [effets de bord](#side-effects)

### Déclaration ESM {#esm-declaration}

Le `package.json` de premier niveau contient la clé :

```js
{
  "type": "module"
}
```

Cela indique aux résolveurs que le code du package utilise des modules EcmaScript, et non des modules CommonJS.

### `"exports"`

Le champ `"exports"` a la structure suivante :

```js
"exports": {
  "./schematics/*": {
    "default": "./schematics/*.js"
  },
  "./package.json": {
    "default": "./package.json"
  },
  ".": {
    "types": "./types/core.d.ts",
    "default": "./fesm2022/core.mjs"
  },
  "./testing": {
    "types": "./types/testing.d.ts",
    "default": "./fesm2022/testing.mjs"
  }
}
```

Les clés les plus importantes sont `"."` et `"./testing"`, qui définissent respectivement les formats de code disponibles pour le point d'entrée principal `@angular/core` et pour le point d'entrée secondaire `@angular/core/testing`.
Pour chaque point d'entrée, les formats disponibles sont :

| Formats                   | Détails                                                                 |
| :------------------------ | :---------------------------------------------------------------------- |
| Typages \(fichiers `.d.ts`\) | Les fichiers `.d.ts` sont utilisés par TypeScript quand on dépend d'un package donné. |
| `default`                 | Du code ES2022 aplati en une seule source.                              |

Les outils qui connaissent ces clés peuvent choisir en priorité un format de code adapté dans `"exports"`.

Les libraries peuvent vouloir exposer des fichiers statiques supplémentaires qui ne sont pas couverts par les exports des points d'entrée JavaScript, comme des mixins Sass ou du CSS précompilé.

Pour plus d'informations, consultez [Gérer les assets dans une library](tools/libraries/creating-libraries#managing-assets-in-a-library).

### Clés de résolution historiques {#legacy-resolution-keys}

En plus de `"exports"`, le `package.json` de premier niveau définit aussi des clés historiques de résolution de modules pour les résolveurs qui ne prennent pas en charge `"exports"`.
Pour `@angular/core`, ce sont :

```js
{
  "module": "./fesm2022/core.mjs",
  "typings": "./types/core.d.ts",
}
```

Comme le montre l'extrait de code précédent, un résolveur de modules peut utiliser ces clés pour charger un format de code précis.

### Effets de bord {#side-effects}

La dernière fonction du `package.json` est de déclarer si le package a des [effets de bord](#sideeffects-flag).

```js
{
  "sideEffects": false
}
```

La plupart des packages Angular ne devraient pas dépendre d'effets de bord de premier niveau, et devraient donc inclure cette déclaration.

### Points d'entrée et code splitting {#entrypoints-and-code-splitting}

Les packages au format Angular Package Format contiennent un point d'entrée principal et zéro ou plusieurs points d'entrée secondaires \(par exemple, `@angular/common/http`\).
Les points d'entrée remplissent plusieurs fonctions.

1. Ils définissent les spécificateurs de module depuis lesquels les utilisateurs importent du code \(par exemple, `@angular/core` et `@angular/core/testing`\).

   Les utilisateurs perçoivent généralement ces points d'entrée comme des groupes de symboles distincts, avec des finalités ou des capacités différentes.

   Certains points d'entrée ne servent qu'à des usages particuliers, comme les tests.
   De telles API peuvent être séparées du point d'entrée principal pour réduire le risque qu'elles soient utilisées par accident ou de manière incorrecte.

1. Ils définissent la granularité à laquelle le code peut être chargé en lazy loading.

   De nombreux outils de build modernes ne savent faire du « code splitting » \(c'est-à-dire du lazy loading\) qu'au niveau du module ES.
   L'Angular Package Format utilise principalement un seul module ES « aplati » par point d'entrée. La plupart des outils de build ne peuvent donc pas découper le code d'un même point d'entrée en plusieurs chunks de sortie.

La règle générale pour les packages APF est d'utiliser des points d'entrée pour les plus petits ensembles possibles de code logiquement lié.
Par exemple, le package Angular Material publie chaque composant logique, ou ensemble de composants, comme un point d'entrée séparé : un pour Button, un pour Tabs, etc.
Cela permet de charger chaque composant Material séparément en lazy loading, si on le souhaite.

Toutes les libraries n'ont pas besoin d'une telle granularité.
La plupart des libraries qui ont une seule finalité logique devraient être publiées avec un seul point d'entrée.
`@angular/core`, par exemple, utilise un seul point d'entrée pour le runtime, car le runtime Angular est généralement utilisé comme une seule entité.

#### Résolution des points d'entrée secondaires {#resolution-of-secondary-entry-points}

Les points d'entrée secondaires peuvent être résolus via le champ `"exports"` du `package.json` du package.

Pour savoir comment configurer des points d'entrée secondaires dans une library, consultez [Points d'entrée](tools/libraries/creating-libraries#entry-points).

### README.md

Le fichier README au format Markdown, utilisé pour afficher la description d'un package sur npm et GitHub.

Exemple de contenu du README du package @angular/core :

```html
Angular &equals;&equals;&equals;&equals;&equals;&equals;&equals; The sources for this package are in
the main [Angular](https://github.com/angular/angular) repo.Please file issues and pull requests
against that repo. License: MIT
```

## Compilation des libraries et optimisations des packages {#library-compilation-and-package-optimizations}

Cette section décrit le processus de compilation des libraries et les optimisations qui s'appliquent au package npm avant la publication de l'artefact sur un registre.

### Compilation partielle {#partial-compilation}

Les libraries au format Angular Package Format doivent être publiées en mode « compilation partielle ».
Il s'agit d'un mode de compilation de `ngc` qui produit du code Angular compilé non lié à une version précise du runtime Angular, contrairement à la compilation complète utilisée pour les applications, où les versions du compilateur et du runtime Angular doivent correspondre exactement.

Pour compiler partiellement du code Angular, utilisez l'option `compilationMode` dans la propriété `angularCompilerOptions` de votre `tsconfig.json` :

```js
{
  …
  "angularCompilerOptions": {
    "compilationMode": "partial",
  }
}
```

Le code de library compilé partiellement est ensuite converti en code entièrement compilé par l'Angular CLI pendant le build de l'application.

Si votre pipeline de build n'utilise pas l'Angular CLI, reportez-vous au guide [Consommer du code partial-Ivy en dehors de l'Angular CLI](tools/libraries/creating-libraries#consuming-partial-ivy-code-outside-the-angular-cli).

### Aplatissement des modules ES {#flattening-of-es-modules}

L'Angular Package Format spécifie que le code doit être publié au format de module ES « aplati ».
Cela réduit considérablement le temps de build des applications Angular, ainsi que le temps de téléchargement et d'analyse du bundle final de l'application.
Consultez l'excellent article [« The cost of small modules »](https://nolanlawson.com/2016/08/15/the-cost-of-small-modules) de Nolan Lawson.

Le compilateur Angular peut générer des fichiers de module ES d'index. Des outils comme Rollup / Rolldown peuvent utiliser ces fichiers pour générer des modules aplatis au format de fichier _Flattened ES Module_ (FESM).

Le FESM est un format de fichier créé en aplatissant tous les modules ES accessibles depuis un point d'entrée en un seul module ES.
Il est obtenu en suivant tous les imports d'un package et en copiant ce code dans un seul fichier, tout en conservant tous les exports ES publics et en supprimant tous les imports privés.
Cependant, dans certains cas, le FESM peut dépendre de chunks partagés entre plusieurs points d'entrée.

L'abréviation FESM, prononcée _phe-som_, peut être suivie d'un nombre, comme FESM2020.
Ce nombre désigne le niveau de langage du JavaScript contenu dans le module.
Ainsi, un fichier FESM2022 serait en ESM+ES2022 et contiendrait des instructions import/export et du code source ES2022.

Pour générer un fichier d'index de module ES aplati, utilisez les options de configuration suivantes dans votre fichier tsconfig.json :

```js
{
  "compilerOptions": {
    …
    "module": "esnext",
    "target": "es2022",
    …
  },
  "angularCompilerOptions": {
    …
    "flatModuleOutFile": "my-ui-lib.js",
    "flatModuleId": "my-ui-lib"
  }
}
```

Une fois le fichier d'index \(par exemple, `my-ui-lib.js`\) généré par ngc, des bundlers et des optimiseurs comme Rollup et Rolldown peuvent être utilisés pour produire le fichier ESM aplati.

### L'option « sideEffects » {#sideeffects-flag}

Par défaut, les modules EcmaScript ont des effets de bord : importer depuis un module garantit que tout le code au niveau supérieur de ce module s'exécute.
C'est souvent indésirable, car la plupart du code à effets de bord dans les modules courants n'en a pas réellement, mais n'affecte que des symboles précis.
Si ces symboles ne sont pas importés et utilisés, il est souvent souhaitable de les supprimer grâce à un processus d'optimisation appelé tree-shaking, et le code à effets de bord peut l'empêcher.

Des outils de build comme webpack prennent en charge une option qui permet aux packages de déclarer qu'ils ne dépendent pas de code à effets de bord au niveau supérieur de leurs modules, ce qui laisse aux outils plus de liberté pour appliquer le tree-shaking au code du package.
Le résultat final de ces optimisations devrait être un bundle plus petit et une meilleure répartition du code dans les chunks après le code splitting.
Cette optimisation peut casser votre code s'il contient des effets de bord non locaux ; c'est toutefois rare dans les applications Angular, et c'est généralement le signe d'une mauvaise conception.
La recommandation est que tous les packages se déclarent sans effets de bord en définissant la propriété `sideEffects` à `false`, et que les développeurs suivent le [guide de style Angular](/style-guide), qui produit naturellement du code sans effets de bord non locaux.

Plus d'informations : [documentation webpack sur les effets de bord](https://github.com/webpack/webpack/tree/main/examples/side-effects)

### Niveau de langage ES2022 {#es2022-language-level}

Le niveau de langage ES2022 est désormais le niveau de langage par défaut consommé par l'Angular CLI et les autres outils.
L'Angular CLI abaisse le niveau de langage du bundle vers un niveau pris en charge par tous les navigateurs ciblés, au moment du build de l'application.

### Aplatissement des définitions de types (bundling des `.d.ts`) {#flattening-of-type-definitions-dts-bundling}

Depuis l'APF v8, il est recommandé de bundler les définitions TypeScript.
Le bundling des définitions de types peut accélérer considérablement les compilations pour les utilisateurs, surtout si votre library contient de nombreux fichiers source `.ts` individuels.

Angular utilise [`rolldown-plugin-dts`](https://github.com/sxzz/rolldown-plugin-dts) pour aplatir les fichiers `.d.ts` (avec `rolldown`, de la même manière que les fichiers FESM sont créés).

Utiliser rolldown pour le bundling des `.d.ts` est avantageux, car il prend en charge le code splitting entre points d'entrée.
Par exemple, si plusieurs points d'entrée reposent sur le même type partagé, un fichier `.d.ts` partagé est créé en plus des fichiers `.d.ts` aplatis plus volumineux.
C'est souhaitable et cela évite la duplication des types.

### Tslib

Depuis l'APF v10, il est recommandé d'ajouter tslib comme dépendance directe de votre point d'entrée principal.
En effet, la version de tslib est liée à la version de TypeScript utilisée pour compiler votre library.

## Exemples {#examples}

Les packages npm du framework Angular constituent une bonne référence pour des packages de libraries conformes à l'APF.
Parcourez l'organisation des fichiers et le package npm sur UNPKG :

<docs-pill-row>
  <docs-pill href="https://app.unpkg.com/@angular/core@latest" title="Package @angular/core"/>
  <docs-pill href="https://app.unpkg.com/@angular/material@latest" title="Package @angular/material"/>
</docs-pill-row>

## Définition des termes {#definition-of-terms}

Les termes suivants sont utilisés à dessein tout au long de ce document.
Cette section en donne la définition pour plus de clarté.

### Package {#package}

Le plus petit ensemble de fichiers publiés sur npm et installés ensemble, par exemple `@angular/core`.
Ce package comprend un manifeste appelé package.json, du code source compilé, des fichiers de définitions TypeScript, des source maps, des métadonnées, etc.
Le package s'installe avec `npm install @angular/core`.

### Symbole {#symbol}

Une classe, une fonction, une constante ou une variable contenue dans un module et, éventuellement, rendue visible au monde extérieur via un export de module.

### Module {#module}

Abréviation de ECMAScript Modules.
Un fichier contenant des instructions qui importent et exportent des symboles.
C'est identique à la définition des modules dans la spécification ECMAScript.

### ESM

Abréviation de ECMAScript Modules \(voir ci-dessus\).

### FESM

Abréviation de Flattened ES Modules : un format de fichier créé en aplatissant tous les modules ES accessibles depuis un point d'entrée en un seul module ES.
Notez qu'un FESM est généralement un seul fichier, mais qu'il peut dépendre d'un chunk partagé avec d'autres FESM.

### Identifiant de module {#module-id}

L'identifiant d'un module utilisé dans les instructions d'import \(par exemple, `@angular/core`\).
L'identifiant correspond souvent directement à un chemin sur le système de fichiers, mais ce n'est pas toujours le cas, en raison des différentes stratégies de résolution de modules.

### Spécificateur de module {#module-specifier}

Un identifiant de module \(voir ci-dessus\).

### Stratégie de résolution de modules {#module-resolution-strategy}

Algorithme utilisé pour convertir les identifiants de module en chemins sur le système de fichiers.
Node.js en possède une, bien spécifiée et largement utilisée ; TypeScript prend en charge plusieurs stratégies de résolution de modules ; [Closure Compiler](https://developers.google.com/closure/compiler) en a encore une autre.

### Format de module {#module-format}

Spécification de la syntaxe des modules qui couvre au minimum la syntaxe d'import et d'export depuis un fichier.
Les formats de module courants sont CommonJS \(CJS, généralement utilisé pour les applications Node.js\) et ECMAScript Modules \(ESM\).
Le format de module indique uniquement le packaging des modules individuels, mais pas les fonctionnalités du langage JavaScript utilisées pour constituer le contenu du module.
C'est pourquoi l'équipe Angular utilise souvent le spécificateur de niveau de langage comme suffixe du format de module \(par exemple, ESM+ES2022 indique que le module est au format ESM et contient du code ES2022\).

### Bundle {#bundle}

Un artefact sous la forme d'un unique fichier JS, produit par un outil de build \(par exemple, [webpack](https://webpack.js.org) ou [Rollup](https://rollupjs.org)\), qui contient des symboles provenant d'un ou plusieurs modules.
Les bundles sont un contournement propre aux navigateurs, qui réduit la charge réseau qu'entraînerait le téléchargement par les navigateurs de centaines, voire de dizaines de milliers de fichiers.
Node.js n'utilise généralement pas de bundles.
Les formats de bundle courants sont UMD et System.register.

### Niveau de langage {#language-level}

Le langage du code \(ES2022\).
Indépendant du format de module.

### Point d'entrée {#entry-point}

Un module destiné à être importé par l'utilisateur.
Il est référencé par un identifiant de module unique et exporte l'API publique référencée par cet identifiant de module.
`@angular/core` ou `@angular/core/testing` en sont des exemples.
Ces deux points d'entrée existent dans le package `@angular/core`, mais ils exportent des symboles différents.
Un package peut avoir de nombreux points d'entrée.

### Import profond {#deep-import}

Le fait de récupérer des symboles depuis des modules qui ne sont pas des points d'entrée.
Ces identifiants de module sont généralement considérés comme des API privées, susceptibles de changer au cours de la vie du projet ou pendant la création du bundle du package concerné.

### Import de premier niveau {#top-level-import}

Un import provenant d'un point d'entrée.
Les imports de premier niveau disponibles sont ce qui définit l'API publique ; ils sont exposés dans des modules « @angular/name », comme `@angular/core` ou `@angular/common`.

### Tree-shaking {#tree-shaking}

Le processus d'identification et de suppression du code non utilisé par une application, aussi appelé élimination du code mort.
Il s'agit d'une optimisation globale effectuée au niveau de l'application à l'aide d'outils comme [Rollup](https://rollupjs.org), [Rolldown](https://rolldown.rs/), [Closure Compiler](https://developers.google.com/closure/compiler) ou [Terser](https://github.com/terser/terser).

### Compilateur AOT {#aot-compiler}

Le compilateur Ahead-of-Time d'Angular.

### Définitions de types aplaties {#flattened-type-definitions}

Les définitions TypeScript bundlées, générées avec des outils comme [API Extractor](https://api-extractor.com) ou [rollup-plugin-dts](https://github.com/Swatinem/rollup-plugin-dts).
