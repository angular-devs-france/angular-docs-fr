<!-- source: 7d4750e45642d4ef3400d9687e196b757e4ee8af -->
# Compiler des applications Angular

Vous pouvez compiler votre application ou votre library Angular CLI avec la commande `ng build`.
Celle-ci compile votre code TypeScript en JavaScript, puis optimise, regroupe en bundles et minifie la sortie selon les besoins.

`ng build` exécute uniquement le builder de la cible `build` du projet par défaut, tel que spécifié dans `angular.json`.
L'Angular CLI inclut quatre builders généralement utilisés comme cibles `build` :

| Builder                                         | Rôle                                                                                                                                                                                                                                        |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `@angular/build:application`                    | Compile une application avec un bundle côté client, un serveur Node et des routes prérendues au moment du build, avec [esbuild](https://esbuild.github.io/).                                                                                                  |
| `@angular-devkit/build-angular:browser-esbuild` | Regroupe en bundle une application côté client destinée au navigateur avec [esbuild](https://esbuild.github.io/). Consultez la [documentation de `browser-esbuild`](tools/cli/build-system-migration#manual-migration-to-the-compatibility-builder) pour plus d'informations. |
| `@angular-devkit/build-angular:browser`         | Regroupe en bundle une application côté client destinée au navigateur avec [webpack](https://webpack.js.org/).                                                                                                                                                |
| `@angular/build:ng-packagr`                     | Compile une library Angular conforme à l'[Angular Package Format](tools/libraries/angular-package-format).                                                                                                                                        |

Les applications générées par `ng new` utilisent `@angular/build:application` par défaut.
Les libraries générées par `ng generate library` utilisent `@angular/build:ng-packagr` par défaut.

Vous pouvez déterminer quel builder est utilisé pour un projet donné en consultant la cible `build` de ce projet.

```json
{
  "projects": {
    "my-app": {
      "architect": {
        // `ng build` invokes the Architect target named `build`.
        "build": {
          "builder": "@angular/build:application",
          …
        },
        "serve": { … }
        "test": { … }
        …
      }
    }
  }
}
```

Cette page présente l'utilisation et les options de `@angular/build:application`.

## Répertoire de sortie {#output-directory}

Le résultat de ce processus de build est écrit dans un répertoire (`dist/${PROJECT_NAME}` par défaut).

## Configurer les budgets de taille {#configuring-size-budgets}

À mesure que les applications gagnent en fonctionnalités, leur taille augmente aussi.
La CLI vous permet de définir des seuils de taille dans votre configuration pour garantir que certaines parties de votre application restent dans les limites de taille que vous définissez.

Définissez vos limites de taille dans le fichier de configuration de la CLI, `angular.json`, dans une section `budgets` pour chaque [environnement configuré](tools/cli/environments).

```json
{
  …
  "configurations": {
    "production": {
      …
      "budgets": [
        {
          "type": "initial",
          "maximumWarning": "250kb",
          "maximumError": "500kb"
        },
      ]
    }
  }
}
```

Vous pouvez spécifier des budgets de taille pour l'application entière et pour des parties précises.
Chaque entrée de budget configure un budget d'un type donné.
Par exemple, le budget `initial` mesure le JavaScript et le CSS utilisés pour le bootstrap de l'application, ce qui correspond à la valeur `Initial Total` affichée dans le récapitulatif de sortie du build.
Spécifiez les valeurs de taille dans les formats suivants :

| Valeur de taille | Détails                                                                     |
| :-------------- | :-------------------------------------------------------------------------- |
| `123` ou `123b` | Taille en octets.                                                              |
| `123kb`         | Taille en kilo-octets.                                                          |
| `123mb`         | Taille en mégaoctets.                                                          |
| `12%`           | Pourcentage de la taille par rapport à la référence. \(Non valide pour les valeurs de référence.\) |

Lorsque vous configurez un budget, le builder émet un avertissement ou signale une erreur lorsqu'une partie donnée de l'application atteint ou dépasse une taille limite que vous avez définie.

Chaque entrée de budget est un objet JSON avec les propriétés suivantes :

| Propriété       | Valeur                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| type           | Le type de budget. L'une des valeurs suivantes : <table> <thead> <tr> <th> Valeur </th> <th> Détails </th> </tr> </thead> <tbody> <tr> <td> <code>bundle</code> </td> <td> La taille d'un bundle spécifique. Utilisez ce type avec <code>name</code> pour budgéter un bundle spécifique, y compris un bundle chargé en lazy loading. </td> </tr> <tr> <td> <code>initial</code> </td> <td> La taille du JavaScript et du CSS nécessaires au bootstrap de l'application. Correspond à la valeur <code>Initial Total</code> affichée dans le récapitulatif de sortie du build. Par défaut, avertissement à 500kb et erreur à 1mb. </td> </tr> <tr> <td> <code>allScript</code> </td> <td> La taille de tous les scripts. </td> </tr> <tr> <td> <code>all</code> </td> <td> La taille de l'application entière. </td> </tr> <tr> <td> <code>anyComponentStyle</code> </td> <td> La taille de n'importe quelle feuille de style de composant. Par défaut, avertissement à 4kb et erreur à 8kb. </td> </tr> <tr> <td> <code>anyScript</code> </td> <td> La taille de n'importe quel script. </td> </tr> <tr> <td> <code>any</code> </td> <td> La taille de n'importe quel fichier. </td> </tr> </tbody> </table> |
| name           | Le nom du bundle pour `type=bundle`. Il s'agit du nom de bundle indiqué par le builder, et non du nom du fichier de sortie généré.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| baseline       | La taille de référence pour la comparaison. Si elle n'est pas définie, la référence vaut `0` par défaut, et les valeurs de seuil sont calculées par rapport à cette référence.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| maximumWarning | Le seuil maximal d'avertissement par rapport à la référence.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| maximumError   | Le seuil maximal d'erreur par rapport à la référence.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| minimumWarning | Le seuil minimal d'avertissement par rapport à la référence.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| minimumError   | Le seuil minimal d'erreur par rapport à la référence.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| warning        | Le seuil d'avertissement par rapport à la référence (min et max).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| error          | Le seuil d'erreur par rapport à la référence (min et max).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

Pour configurer un budget pour un bundle chargé en lazy loading, utilisez `type: "bundle"` et donnez à `name` le nom de ce bundle.

```json
{
  "budgets": [
    {
      "type": "bundle",
      "name": "admin",
      "maximumWarning": "250kb",
      "maximumError": "300kb"
    }
  ]
}
```

Le champ `name` correspond au nom du bundle, et non au nom du fichier émis : il n'accepte donc ni caractères génériques ni expressions régulières comme `admin.*.js`.

L'exemple suivant montre un budget qui utilise une référence :

```json
{
  "type": "bundle",
  "name": "main",
  "baseline": "200kb",
  "maximumWarning": "10%",
  "maximumError": "20%"
}
```

Dans cet exemple, le builder émet un avertissement lorsque le bundle dépasse `220kb` et une erreur lorsqu'il dépasse `240kb`.

## Configurer les dépendances CommonJS {#configuring-commonjs-dependencies}

Privilégiez toujours les [modules ECMAScript](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/import) (ESM) natifs dans toute votre application et ses dépendances.
ESM est un standard du web entièrement spécifié et une fonctionnalité du langage JavaScript qui offre une solide prise en charge de l'analyse statique. Cela rend les optimisations de bundle plus efficaces qu'avec les autres formats de module.

L'Angular CLI prend aussi en charge l'import de dépendances [CommonJS](https://nodejs.org/api/modules.html) dans votre projet et les regroupe automatiquement dans les bundles.
Cependant, les modules CommonJS peuvent empêcher les bundlers et les minifieurs de les optimiser efficacement, ce qui produit des bundles plus volumineux.
Pour plus d'informations, consultez [How CommonJS is making your bundles larger](https://web.dev/commonjs-larger-bundles).

L'Angular CLI émet des avertissements si elle détecte que votre application navigateur dépend de modules CommonJS.
Lorsque vous rencontrez une dépendance CommonJS, envisagez de demander au mainteneur de prendre en charge les modules ECMAScript, d'y contribuer vous-même ou d'utiliser une dépendance alternative qui répond à vos besoins.
Si la meilleure option est d'utiliser une dépendance CommonJS, vous pouvez désactiver ces avertissements en ajoutant le nom du module CommonJS à l'option `allowedCommonJsDependencies` dans les options de `build` situées dans `angular.json`.

```json
"build": {
  "builder": "@angular/build:application",
  "options": {
     "allowedCommonJsDependencies": [
        "lodash"
     ]
     …
   }
   …
},
```

## Configurer la compatibilité des navigateurs {#configuring-browser-compatibility}

L'Angular CLI utilise [Browserslist](https://github.com/browserslist/browserslist) pour garantir la compatibilité avec les différentes versions de navigateurs.
Selon les navigateurs pris en charge, Angular transforme automatiquement certaines fonctionnalités JavaScript et CSS pour que l'application compilée n'utilise pas une fonctionnalité qui n'est pas implémentée par un navigateur pris en charge. En revanche, l'Angular CLI n'ajoute pas automatiquement de polyfills pour compléter les API web manquantes. Utilisez l'option `polyfills` dans `angular.json` pour ajouter des polyfills.

Par défaut, l'Angular CLI utilise une configuration `browserslist` qui [correspond aux navigateurs pris en charge par Angular](reference/versions#browser-support) pour la version majeure courante.

Pour remplacer la configuration interne, exécutez [`ng generate config browserslist`](cli/generate/config), qui génère dans le répertoire du projet un fichier de configuration `.browserslistrc` correspondant aux navigateurs pris en charge par Angular.

Consultez le [dépôt browserslist](https://github.com/browserslist/browserslist) pour d'autres exemples de ciblage de navigateurs et de versions spécifiques.
Évitez d'étendre cette liste à davantage de navigateurs. Même si le code de votre application est compatible plus largement, Angular lui-même pourrait ne pas l'être.
Vous ne devriez que _réduire_ l'ensemble des navigateurs ou des versions de cette liste.

HELPFUL: Utilisez [browsersl.ist](https://browsersl.ist) pour afficher les navigateurs compatibles avec une requête `browserslist`.

## Configurer Tailwind {#configuring-tailwind}

Angular prend en charge [Tailwind CSS](https://tailwindcss.com/), un framework CSS orienté utilitaires (utility-first).

Pour intégrer Tailwind CSS à l'Angular CLI, consultez [Utiliser Tailwind CSS avec Angular](guide/tailwind)

## Inlining du CSS critique {#critical-css-inlining}

Angular peut intégrer directement (inline) les définitions CSS critiques de votre application pour améliorer le [First Contentful Paint (FCP)](https://web.dev/first-contentful-paint).
Cette option est activée par défaut. Vous pouvez désactiver cet inlining dans les [options de personnalisation `styles`](reference/configs/workspace-config#styles-optimization-options).

Cette optimisation extrait le CSS nécessaire au rendu du viewport initial et l'intègre directement dans le HTML généré, ce qui permet au navigateur d'afficher le contenu plus rapidement sans attendre le chargement complet des feuilles de style. Le reste du CSS se charge ensuite de manière asynchrone en arrière-plan. L'Angular CLI utilise [Beasties](https://github.com/danielroe/beasties) pour analyser le HTML et les styles de votre application.
