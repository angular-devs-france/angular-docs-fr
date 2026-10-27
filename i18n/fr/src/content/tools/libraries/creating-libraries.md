<!-- source: 9773b96cfdf8b992bcd8eb8fc5f48c96642dda80 -->
# Créer des libraries

Cette page présente une vue d'ensemble conceptuelle de la création et de la publication de nouvelles libraries pour étendre les fonctionnalités d'Angular.

Si vous constatez que vous devez résoudre le même problème dans plus d'une application \(ou que vous souhaitez partager votre solution avec d'autres développeurs\), vous avez là un bon candidat pour une library.
Un exemple simple pourrait être un bouton qui envoie les utilisateurs vers le site web de votre entreprise, et qui serait inclus dans toutes les applications que votre entreprise développe.

## Premiers pas {#getting-started}

Utilisez l'Angular CLI pour générer le squelette d'une nouvelle library dans un nouveau workspace avec les commandes suivantes.

```shell

ng new my-workspace --no-create-application
cd my-workspace
ng generate library my-lib

```

<docs-callout title="Nommer votre library">

Choisissez le nom de votre library avec beaucoup de soin si vous prévoyez de la publier plus tard sur un registre de packages public comme npm.
Consultez [Publier votre library](tools/libraries/creating-libraries#publishing-your-library).

Évitez les noms préfixés par `ng-`, comme `ng-library`.
Le préfixe `ng-` est un mot-clé réservé, utilisé par le framework Angular et ses libraries.
Le préfixe `ngx-` est préféré, par convention, pour indiquer que la library est adaptée à une utilisation avec Angular.
C'est aussi une excellente indication pour les utilisateurs du registre, qui peuvent ainsi distinguer les libraries des différents frameworks JavaScript.

</docs-callout>

La commande `ng generate` crée le dossier `projects/my-lib` dans votre workspace, qui contient un composant.

HELPFUL: Pour plus de détails sur la structure d'un projet de library, reportez-vous à la section [Fichiers d'un projet de library](reference/configs/file-structure#library-project-files) du [guide de la structure des fichiers du projet](reference/configs/file-structure).

Utilisez le modèle monorepo pour utiliser le même workspace pour plusieurs projets.
Consultez [Configurer un workspace multi-projets](reference/configs/file-structure#multiple-projects).

Quand vous générez une nouvelle library, le fichier de configuration du workspace, `angular.json`, est mis à jour avec un projet de type `library`.

```json

"projects": {
  …
  "my-lib": {
    "root": "projects/my-lib",
    "sourceRoot": "projects/my-lib/src",
    "projectType": "library",
    "prefix": "lib",
    "architect": {
      "build": {
        "builder": "@angular/build:ng-packagr",
        …

```

Compilez, testez et analysez (lint) le projet avec les commandes de la CLI :

```shell

ng build my-lib --configuration development
ng test my-lib
ng lint my-lib

```

Notez que le builder configuré pour le projet est différent du builder par défaut des projets d'application.
Ce builder garantit notamment que la library est toujours compilée avec le [compilateur AOT](tools/cli/aot-compiler).

Pour rendre le code d'une library réutilisable, vous devez lui définir une API publique.
Cette « couche utilisateur » définit ce qui est disponible pour les consommateurs de votre library.
Un utilisateur de votre library doit pouvoir accéder aux fonctionnalités publiques \(comme les providers de services et les fonctions utilitaires générales\) via un seul chemin d'import.

L'API publique de votre library est maintenue dans le fichier `public-api.ts` du dossier de votre library.
Tout ce qui est exporté depuis ce fichier devient public quand votre library est importée dans une application.

Votre library doit fournir une documentation \(généralement un fichier README\) pour son installation et sa maintenance.

## Points d'entrée {#entry-points}

Un _point d'entrée_ est un spécificateur de module depuis lequel les consommateurs importent, associé à l'API publique que ce spécificateur expose.
Chaque library possède un _point d'entrée principal_ et peut ajouter autant de _points d'entrée secondaires_ que nécessaire.

Le point d'entrée principal est le package lui-même.
Le fichier `ng-package.json` à la racine de la library le configure, et `lib.entryFile` désigne le fichier qui définit son API publique.

```json {header: 'projects/my-lib/ng-package.json'}
{
  "$schema": "../../node_modules/ng-packagr/ng-package.schema.json",
  "dest": "../../dist/my-lib",
  "lib": {
    "entryFile": "src/public-api.ts"
  }
}
```

Les consommateurs importent le point d'entrée principal par le nom du package :

```ts
import {ThemeService} from 'my-lib';
```

Un point d'entrée secondaire est un répertoire à l'intérieur de la library, doté de son propre fichier `ng-package.json`.
Le chemin de ce répertoire relatif à la racine de la library définit le sous-chemin d'import.
Par exemple, un répertoire `button` met `my-lib/button` à la disposition des consommateurs :

```ts
import {ButtonComponent} from 'my-lib/button';
```

Les packages Angular utilisent cette même structure.
Par exemple, `@angular/core` est le point d'entrée principal et `@angular/core/testing` est un point d'entrée secondaire du même package.

HELPFUL: Pour une vue d'ensemble architecturale de la façon dont les points d'entrée permettent le code splitting et définissent les limites des chunks dans l'Angular Package Format, consultez [Points d'entrée et code splitting](tools/libraries/angular-package-format#entrypoints-and-code-splitting).

### Ajouter un point d'entrée secondaire {#adding-a-secondary-entry-point}

Créez un répertoire à l'intérieur de la library avec son propre `ng-package.json` et son propre fichier d'API publique :

```text
projects/my-lib/
├── ng-package.json      (primary entry point: my-lib)
├── package.json
├── src/
│   ├── public-api.ts
│   └── lib/ …
└── button/
    ├── ng-package.json  (secondary entry point: my-lib/button)
    └── src/
        ├── public-api.ts
        └── button.ts
```

Le `ng-package.json` secondaire configure uniquement le point d'entrée lui-même :

```json {header: 'projects/my-lib/button/ng-package.json'}
{
  "$schema": "../../../node_modules/ng-packagr/ng-entrypoint.schema.json",
  "lib": {
    "entryFile": "src/public-api.ts"
  }
}
```

IMPORTANT: Un `ng-package.json` secondaire n'accepte que les options `lib`. Les options qui concernent tout le package, comme `dest` et `assets`, se placent dans le `ng-package.json` à la racine de la library et s'appliquent à tous les points d'entrée.

Vous n'avez pas besoin d'enregistrer le point d'entrée secondaire ailleurs.
Pendant le build, `ng-packagr` découvre automatiquement chaque `ng-package.json` sous la racine de la library et déduit le sous-chemin d'import de son chemin de répertoire (par exemple, `button/` devient `my-lib/button` et `testing/harness/` devient `my-lib/testing/harness`).

Pour importer un point d'entrée secondaire depuis une application du même workspace, ajoutez un mapping de chemin avec wildcard dans le `tsconfig.json` du workspace.
Par défaut, `ng generate library` ne mappe que la racine du package, ce qui ne couvre pas les sous-chemins :

```json {header: 'tsconfig.json'}
{
  "compilerOptions": {
    "paths": {
      "my-lib": ["./dist/my-lib"],
      "my-lib/*": ["./dist/my-lib/*"]
    }
  }
}
```

IMPORTANT: Quand vous importez du code depuis un autre point d'entrée de la même library, utilisez toujours son chemin d'import de package (par exemple, `import {ThemeService} from 'my-lib'`) plutôt qu'un chemin relatif. `ng-packagr` compile chaque point d'entrée séparément, dans l'ordre des dépendances : les imports relatifs entre points d'entrée et les dépendances circulaires entre points d'entrée font donc échouer le build.

## Extraire des parties d'une application dans une library {#refactoring-parts-of-an-application-into-a-library}

Pour rendre votre solution réutilisable, vous devez l'adapter afin qu'elle ne dépende pas de code propre à l'application.
Voici quelques points à prendre en compte pour migrer des fonctionnalités d'une application vers une library.

- Les déclarations comme les composants et les pipes doivent être conçues sans état, c'est-à-dire qu'elles ne dépendent pas de variables externes et ne les modifient pas.
  Si vous dépendez d'un état, vous devez évaluer chaque cas et décider s'il s'agit d'un état de l'application ou d'un état que la library gérerait.

- Tous les observables auxquels les composants s'abonnent en interne doivent être nettoyés et libérés au cours du cycle de vie de ces composants
- Les composants doivent exposer leurs interactions via des inputs pour fournir le contexte, et des outputs pour communiquer des événements aux autres composants

- Vérifiez toutes les dépendances internes.
  - Pour les classes ou interfaces personnalisées utilisées dans les composants ou les services, vérifiez si elles dépendent d'autres classes ou interfaces qui doivent aussi être migrées
  - De même, si le code de votre library dépend d'un service, ce service doit être migré
  - Si le code de votre library ou ses templates dépendent d'autres libraries \(comme Angular Material, par exemple\), vous devez configurer votre library avec ces dépendances

- Réfléchissez à la façon dont vous fournissez des services aux applications clientes.
  - Les services doivent déclarer leurs propres providers, plutôt que de déclarer les providers dans le NgModule ou dans un composant.
    Déclarer un provider rend ce service _compatible avec le tree-shaking_.
    Cette pratique permet au compilateur d'exclure le service du bundle s'il n'est jamais injecté dans l'application qui importe la library.
    Pour en savoir plus, consultez [Providers compatibles avec le tree-shaking](guide/di/lightweight-injection-tokens).

  - Si vous enregistrez des providers de services globaux, exposez une fonction provider `provideXYZ()`.
  - Si votre library fournit des services optionnels que certaines applications clientes n'utiliseront peut-être pas, prenez correctement en charge le tree-shaking dans ce cas en utilisant le [design pattern des jetons légers](guide/di/lightweight-injection-tokens)

## S'intégrer à la CLI avec des schematics de génération de code {#integrating-with-the-cli-using-code-generation-schematics}

Une library comprend généralement du _code réutilisable_ qui définit des composants, des services et d'autres artefacts Angular \(pipes, directives\) que vous importez dans un projet.
Une library est packagée dans un package npm pour être publiée et partagée.
Ce package peut aussi inclure des schematics qui fournissent des instructions pour générer ou transformer du code directement dans votre projet, de la même façon que la CLI crée un nouveau composant générique avec `ng generate component`.
Un schematic packagé avec une library peut, par exemple, fournir à l'Angular CLI les informations dont elle a besoin pour générer un composant qui configure et utilise une fonctionnalité particulière, ou un ensemble de fonctionnalités, définie dans cette library.
Le [schematic de navigation d'Angular Material](https://material.angular.dev/guide/schematics#navigation-schematic) en est un exemple : il configure le [BreakpointObserver](https://material.angular.dev/cdk/layout/overview#breakpointobserver) du CDK et l'utilise avec les composants [MatSideNav](https://material.angular.dev/components/sidenav/overview) et [MatToolbar](https://material.angular.dev/components/toolbar/overview) de Material.

Créez et incluez les types de schematics suivants :

- Incluez un schematic d'installation pour que `ng add` puisse ajouter votre library à un projet
- Incluez des schematics de génération dans votre library pour que `ng generate` puisse générer dans un projet les artefacts que vous avez définis \(composants, services, tests\)
- Incluez un schematic de mise à jour pour que `ng update` puisse mettre à jour les dépendances de votre library et fournir des migrations pour les changements cassants des nouvelles versions

Ce que vous incluez dans votre library dépend de votre besoin.
Par exemple, vous pourriez définir un schematic qui crée une liste déroulante préremplie avec des données d'exemple, pour montrer comment l'ajouter à une application.
Si vous voulez une liste déroulante qui contienne à chaque fois des valeurs différentes passées en paramètre, votre library pourrait définir un schematic qui la crée avec une configuration donnée.
Les développeurs pourraient alors utiliser `ng generate` pour configurer une instance pour leur propre application.

Supposons que vous vouliez lire un fichier de configuration puis générer un formulaire à partir de cette configuration.
Si ce formulaire nécessite une personnalisation supplémentaire par le développeur qui utilise votre library, un schematic sera sans doute la meilleure solution.
En revanche, si le formulaire est toujours le même et ne nécessite pas beaucoup de personnalisation de la part des développeurs, vous pourriez créer un composant dynamique qui prend la configuration et génère le formulaire.
En général, plus la personnalisation est complexe, plus l'approche par schematic est utile.

Pour plus d'informations, consultez [Générer du code avec les schematics](tools/cli/schematics) et [Schematics pour les libraries](tools/cli/schematics-for-libraries).

## Publier votre library {#publishing-your-library}

Utilisez l'Angular CLI et le gestionnaire de packages npm pour compiler et publier votre library sous forme de package npm.

L'Angular CLI utilise un outil appelé [ng-packagr](https://github.com/ng-packagr/ng-packagr/blob/main/README.md) pour créer, à partir de votre code compilé, des packages publiables sur npm.
Consultez [Compiler des libraries avec Ivy](tools/libraries/creating-libraries#publishing-libraries) pour en savoir plus sur les formats de distribution pris en charge par `ng-packagr` et sur la façon
de choisir le bon format pour votre library.

Compilez toujours les libraries destinées à la distribution avec la configuration `production`.
Cela garantit que la sortie générée utilise les optimisations appropriées et le bon format de package pour npm.

```shell

ng build my-lib
cd dist/my-lib
npm publish

```

## Gérer les assets dans une library {#managing-assets-in-a-library}

Dans votre library Angular, le package distribuable peut inclure des assets supplémentaires comme des fichiers de thème, des mixins Sass ou de la documentation \(comme un changelog\).
Pour plus d'informations, consultez [copier des assets dans votre library pendant le build](https://github.com/ng-packagr/ng-packagr/blob/main/docs/copy-assets.md) et [intégrer des assets dans les styles des composants](https://github.com/ng-packagr/ng-packagr/blob/main/docs/embed-assets-css.md).

IMPORTANT: Si vous incluez des assets supplémentaires comme des mixins Sass ou du CSS précompilé,
vous devez les ajouter manuellement aux [« exports »](tools/libraries/angular-package-format#exports) conditionnels du `package.json` du point d'entrée principal.

`ng-packagr` fusionne les `"exports"` écrits à la main avec ceux générés automatiquement, ce qui permet aux auteurs de libraries de configurer des sous-chemins d'export supplémentaires ou des conditions personnalisées.

```json

"exports": {
  ".": {
    "sass": "./_index.scss",
  },
  "./theming": {
    "sass": "./_theming.scss"
  },
  "./prebuilt-themes/indigo-pink.css": {
    "style": "./prebuilt-themes/indigo-pink.css"
  }
}

```

L'exemple ci-dessus est extrait du package distribuable [@angular/material](https://unpkg.com/browse/@angular/material/package.json).

## Dépendances peer {#peer-dependencies}

Les libraries Angular doivent lister toutes les dépendances `@angular/*` dont elles dépendent comme dépendances peer.
Cela garantit que, quand des modules demandent Angular, ils obtiennent tous exactement le même module.
Si une library liste `@angular/core` dans `dependencies` au lieu de `peerDependencies`, elle risque d'obtenir un autre module Angular, ce qui casserait votre application.

## Utiliser votre propre library dans des applications {#using-your-own-library-in-applications}

Vous n'avez pas besoin de publier votre library sur le gestionnaire de packages npm pour l'utiliser dans le même workspace, mais vous devez d'abord la compiler.

Pour utiliser votre propre library dans une application :

- Compilez la library.
  Vous ne pouvez pas utiliser une library avant qu'elle soit compilée.

```shell
  ng build my-lib
```

- Dans vos applications, importez depuis la library par son nom :

```ts
import {myExport} from 'my-lib';
```

### Compiler et recompiler votre library {#building-and-rebuilding-your-library}

L'étape de build est importante si vous n'avez pas publié votre library sous forme de package npm puis réinstallé ce package depuis npm dans votre application.
Par exemple, si vous clonez votre dépôt git et lancez `npm install`, votre éditeur signale les imports de `my-lib` comme manquants si vous n'avez pas encore compilé votre library.

HELPFUL: Quand vous importez quelque chose depuis une library dans une application Angular, Angular cherche une correspondance entre le nom de la library et un emplacement sur le disque.
Quand vous installez le package d'une library, la correspondance se trouve dans le dossier `node_modules`.
Quand vous compilez votre propre library, la correspondance doit se trouver dans les chemins (paths) de votre `tsconfig`.

Générer une library avec l'Angular CLI ajoute automatiquement son chemin au fichier `tsconfig`.
L'Angular CLI utilise les chemins du `tsconfig` pour indiquer au système de build où trouver la library.

Pour plus d'informations, consultez [Path mapping overview](https://www.typescriptlang.org/docs/handbook/module-resolution.html#path-mapping).

Vous pouvez recompiler votre library chaque fois que vous la modifiez, mais cette étape supplémentaire prend du temps.
La fonctionnalité de _builds incrémentaux_ améliore l'expérience de développement des libraries.
À chaque modification d'un fichier, un build partiel est effectué et émet les fichiers modifiés.

Les builds incrémentaux peuvent s'exécuter en tâche de fond dans votre environnement de développement.
Pour profiter de cette fonctionnalité, ajoutez l'option `--watch` à la commande de build :

```shell

ng build my-lib --watch

```

IMPORTANT: La commande `build` de la CLI utilise un builder différent et invoque un outil de build différent pour les libraries et pour les applications.

- Le système de build des applications, `@angular/build`, est basé sur `esbuild` et est inclus dans tous les nouveaux projets Angular CLI
- Le système de build des libraries est basé sur `ng-packagr`.
  Il n'est ajouté à vos dépendances que lorsque vous ajoutez une library avec `ng generate library my-lib`.

Les deux systèmes de build prennent en charge des choses différentes, et même lorsqu'ils prennent en charge les mêmes choses, ils les font différemment.
Cela signifie que le même code source TypeScript peut produire un code JavaScript différent dans une library compilée et dans une application compilée.

Pour cette raison, une application qui dépend d'une library ne doit utiliser que des mappings de chemins TypeScript qui pointent vers la _library compilée_.
Les mappings de chemins TypeScript ne doivent _pas_ pointer vers les fichiers source `.ts` de la library.

### Lier des libraries pour le développement local {#linking-libraries-for-local-development}

Cette section explique comment utiliser la fonctionnalité de liaison locale de votre gestionnaire de packages
(comme [`npm link`](https://docs.npmjs.com/cli/v11/commands/npm-link) ou [`pnpm link`](https://pnpm.io/cli/link)) pour tester une library Angular standalone avec une application externe pendant
le développement local, sans dépendre de la structure de workspace monorepo ni publier sur le registre npm.

NOTE: Si votre library et votre application se trouvent dans le même workspace Angular (une configuration monorepo), le workflow monorepo standard gère automatiquement la liaison et est généralement plus efficace. Cette approche de liaison locale est la plus adaptée quand :

- Vous développez une library standalone et devez tester les modifications avec une application externe qui la consomme.
- Vous testez des modifications de la library dans une application consommatrice située en dehors du workspace monorepo.

#### Configurer l'application consommatrice {#configuring-the-consuming-application}

Pour utiliser des libraries liées, vous devez configurer le fichier `angular.json` de votre application avec les paramètres suivants :

```json
{
  "projects": {
    "your-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "preserveSymlinks": true
          },
          "configurations": {
            "development": {
              "sourceMap": {
                "scripts": true,
                "styles": true,
                "vendor": true
              }
            }
          }
        },
        "serve": {
          "builder": "@angular/build:dev-server",
          "options": {
            "prebundle": {
              "exclude": ["my-lib"]
            }
          }
        }
      }
    }
  }
}
```

**Options de configuration expliquées :**

- `preserveSymlinks: true` : indique au système de build de suivre les liens symboliques créés par la commande de liaison de votre gestionnaire de packages au lieu de les résoudre vers leur emplacement d'origine. C'est indispensable pour éviter d'avoir plusieurs copies des packages node dont dépend le projet.
- `sourceMap.vendor` : active les source maps des dépendances tierces (en particulier `vendor: true`) pour faciliter le débogage du code de la library liée.
- `prebundle.exclude` : par défaut, l'Angular CLI peut pré-bundler toutes les dépendances node. Exclure votre library garantit que le code source lié est correctement surveillé et recompilé en cas de modification.

## Publier des libraries {#publishing-libraries}

Il existe deux formats de distribution utilisables pour publier une library :

| Formats de distribution     | Détails                                                                                                                                                                                                                                                                                                                                  |
| :-------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Partial-Ivy \(recommandé\)  | Contient du code portable qui peut être consommé par des applications Ivy compilées avec n'importe quelle version d'Angular à partir de la v12.                                                                                                                                                                                          |
| Full-Ivy                    | Contient des instructions Angular Ivy privées, dont le fonctionnement n'est pas garanti d'une version d'Angular à l'autre. Ce format exige que la library et l'application soient compilées avec _exactement_ la même version d'Angular. Ce format est utile dans les environnements où tout le code des libraries et des applications est compilé directement depuis les sources. |

Pour publier sur npm, utilisez le format partial-Ivy, car il est stable entre les versions patch d'Angular.

Évitez de compiler des libraries en code full-Ivy si vous publiez sur npm, car les instructions Ivy générées ne font pas partie de l'API publique d'Angular et peuvent donc changer entre les versions patch.

## Garantir la compatibilité des versions de library {#ensuring-library-version-compatibility}

La version d'Angular utilisée pour compiler une application doit toujours être identique ou supérieure aux versions d'Angular utilisées pour compiler les libraries dont elle dépend.
Par exemple, si vous avez une library qui utilise Angular 13, l'application qui dépend de cette library doit utiliser Angular 13 ou une version ultérieure.
Angular ne prend pas en charge l'utilisation d'une version antérieure pour l'application.

Si vous prévoyez de publier votre library sur npm, compilez en code partial-Ivy en définissant `"compilationMode": "partial"` dans `tsconfig.prod.json`.
Ce format partiel est stable entre les différentes versions d'Angular, et peut donc être publié sans risque sur npm.
Le code dans ce format est traité pendant le build de l'application avec la même version du compilateur Angular, ce qui garantit que l'application et toutes ses libraries utilisent une seule et même version d'Angular.

Évitez de compiler des libraries en code full-Ivy si vous publiez sur npm, car les instructions Ivy générées ne font pas partie de l'API publique d'Angular et peuvent donc changer entre les versions patch.

Si vous n'avez jamais publié de package sur npm, vous devez créer un compte utilisateur.
Pour en savoir plus, consultez [Publishing npm Packages](https://docs.npmjs.com/getting-started/publishing-npm-packages).

## Consommer du code partial-Ivy en dehors de l'Angular CLI {#consuming-partial-ivy-code-outside-the-angular-cli}

Une application installe de nombreuses libraries Angular depuis npm dans son répertoire `node_modules`.
Cependant, le code de ces libraries ne peut pas être bundlé directement avec l'application compilée, car il n'est pas entièrement compilé.
Pour terminer la compilation, utilisez le linker Angular.

Pour les applications qui n'utilisent pas l'Angular CLI, le linker est disponible sous forme de plugin [Babel](https://babeljs.io).
Le plugin s'importe depuis `@angular/compiler-cli/linker/babel`.

Le plugin Babel du linker Angular prend en charge la mise en cache du build : les libraries n'ont besoin d'être traitées par le linker qu'une seule fois, indépendamment des autres opérations npm.

Exemple d'intégration du plugin dans un build [webpack](https://webpack.js.org) personnalisé, en enregistrant le linker comme plugin [Babel](https://babeljs.io) via [babel-loader](https://webpack.js.org/loaders/babel-loader/#options).

<docs-code header="webpack.config.mjs" path="adev/src/content/examples/angular-linker-plugin/webpack.config.mjs" region="webpack-config"/>

HELPFUL: L'Angular CLI intègre automatiquement le plugin du linker : si les consommateurs de votre library utilisent la CLI, ils peuvent installer depuis npm des libraries Ivy natives sans aucune configuration supplémentaire.
