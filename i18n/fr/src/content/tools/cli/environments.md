<!-- source: b89dc0e62dafb4dcfa768e559dcbaeef3e40ad00 -->
# Configurer les environnements de l'application

Vous pouvez définir différentes configurations de build nommées pour votre projet, comme `development` et `staging`, avec des valeurs par défaut différentes.

Chaque configuration nommée peut avoir des valeurs par défaut pour n'importe quelle option qui s'applique aux différentes cibles de builder, comme `build`, `serve` et `test`.
Les commandes `build`, `serve` et `test` de l'[Angular CLI](tools/cli) peuvent alors remplacer des fichiers par les versions appropriées à l'environnement cible.

## Configurations de l'Angular CLI {#angular-cli-configurations}

Les builders de l'Angular CLI prennent en charge un objet `configurations`, qui permet de surcharger des options spécifiques d'un builder en fonction de la configuration fournie sur la ligne de commande.

```json

{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            // By default, disable source map generation.
            "sourceMap": false
          },
          "configurations": {
            // For the `debug` configuration, enable source maps.
            "debug": {
              "sourceMap": true
            }
          }
        },
        …
      }
    }
  }
}

```

Vous pouvez choisir la configuration à utiliser avec l'option `--configuration`.

```shell

ng build --configuration debug

```

Les configurations peuvent être appliquées à n'importe quel builder de l'Angular CLI. Plusieurs configurations peuvent être indiquées, séparées par des virgules. Les configurations sont appliquées dans l'ordre ; en cas de conflit, une option prend la valeur de la dernière configuration.

```shell

ng build --configuration debug,production,customer-facing

```

## Configurer des valeurs par défaut propres à chaque environnement {#configure-environment-specific-defaults}

`@angular/build:application` prend en charge le remplacement de fichiers, une option qui permet de substituer des fichiers source avant d'exécuter un build.
Combinée à `--configuration`, elle fournit un mécanisme pour configurer des données propres à chaque environnement dans votre application.

Commencez par [générer les environnements](cli/generate/environments) pour créer le répertoire `src/environments/` et configurer le projet afin qu'il utilise le remplacement de fichiers.

```shell

ng generate environments

```

Le répertoire `src/environments/` du projet contient le fichier de configuration de base, `environment.ts`, qui fournit la configuration par défaut pour la production.
Vous pouvez surcharger les valeurs par défaut pour d'autres environnements, comme `development` et `staging`, dans des fichiers de configuration propres à chaque cible.

Par exemple :

```text

my-app/src/environments
├── environment.development.ts
├── environment.staging.ts
└── environment.ts

```

Le fichier de base `environment.ts` contient les paramètres d'environnement par défaut.
Par exemple :

```ts
export const environment = {
  production: true,
};
```

La commande `build` l'utilise comme cible de build lorsqu'aucun environnement n'est indiqué.
Vous pouvez ajouter d'autres variables, soit comme propriétés supplémentaires de l'objet d'environnement, soit comme objets distincts.
Par exemple, le code suivant ajoute une valeur par défaut pour une variable à l'environnement par défaut :

```ts
export const environment = {
  production: true,
  apiUrl: 'http://my-prod-url',
};
```

CRITICAL: Les fichiers de `src/environments/` sont intégrés au bundle de votre application côté client et visibles par toute personne qui charge la page. N'y stockez jamais de secrets comme des clés d'API. Utilisez plutôt un proxy côté serveur ou un gestionnaire de secrets.

Vous pouvez ajouter des fichiers de configuration propres à une cible, comme `environment.development.ts`.
Le contenu suivant définit les valeurs par défaut pour la cible de build de développement :

```ts
export const environment = {
  production: false,
  apiUrl: 'http://my-dev-url',
};
```

## Utiliser des variables propres à l'environnement dans votre application {#using-environment-specific-variables-in-your-app}

Pour utiliser les configurations d'environnement que vous avez définies, vos composants doivent importer le fichier d'environnement d'origine :

```ts
import {environment} from './environments/environment';
```

Cela garantit que les commandes build et serve trouvent les configurations des cibles de build spécifiques.

Le code suivant, dans le fichier du composant (`app.ts`), utilise une variable d'environnement définie dans les fichiers de configuration.

```ts
import {environment} from './../environments/environment';

// Fetches from `http://my-prod-url` in production, `http://my-dev-url` in development.
fetch(environment.apiUrl);
```

Le fichier de configuration principal de la CLI, `angular.json`, contient une section `fileReplacements` dans la configuration de chaque cible de build, qui vous permet de remplacer n'importe quel fichier du programme TypeScript par une version de ce fichier propre à la cible.
C'est utile pour inclure du code ou des variables propres à une cible dans un build qui vise un environnement spécifique, comme la production ou le staging.

Par défaut, aucun fichier n'est remplacé, mais `ng generate environments` met en place cette configuration automatiquement.
Vous pouvez modifier ou ajouter des remplacements de fichiers pour des cibles de build spécifiques en éditant directement la configuration `angular.json`.

```json

  "configurations": {
    "development": {
      "fileReplacements": [
          {
            "replace": "src/environments/environment.ts",
            "with": "src/environments/environment.development.ts"
          }
        ],
        …

```

Cela signifie que lorsque vous compilez votre configuration de développement avec `ng build --configuration development`, le fichier `src/environments/environment.ts` est remplacé par la version propre à la cible, `src/environments/environment.development.ts`.

Pour ajouter un environnement de staging, créez une copie de `src/environments/environment.ts` nommée `src/environments/environment.staging.ts`, puis ajoutez une configuration `staging` à `angular.json` :

```json

  "configurations": {
    "development": { … },
    "production": { … },
    "staging": {
      "fileReplacements": [
        {
          "replace": "src/environments/environment.ts",
          "with": "src/environments/environment.staging.ts"
        }
      ]
    }
  }

```

Vous pouvez également ajouter d'autres options de configuration à cet environnement cible.
Toute option prise en charge par votre build peut être surchargée dans la configuration d'une cible de build.

Pour lancer un build avec la configuration de staging, exécutez la commande suivante :

```shell

ng build --configuration staging

```

Par défaut, la cible `build` inclut les configurations `production` et `development`, et `ng serve` utilise le build de développement de l'application.
Vous pouvez aussi configurer `ng serve` pour qu'il utilise la configuration de build ciblée en définissant l'option `buildTarget` :

```json

  "serve": {
    "builder": "@angular/build:dev-server",
    "options": { … },
    "configurations": {
      "development": {
        // Use the `development` configuration of the `build` target.
        "buildTarget": "my-app:build:development"
      },
      "production": {
        // Use the `production` configuration of the `build` target.
        "buildTarget": "my-app:build:production"
      }
    },
    "defaultConfiguration": "development"
  },

```

L'option `defaultConfiguration` indique quelle configuration est utilisée par défaut.
Lorsque `defaultConfiguration` n'est pas défini, les `options` sont utilisées directement, sans modification.
