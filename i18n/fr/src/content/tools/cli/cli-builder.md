<!-- source: 059576883c63273c59eede28b25e01cab391f9d0 -->
# Les builders de l'Angular CLI

Un certain nombre de commandes de l'Angular CLI exécutent un processus complexe sur votre code, comme compiler, tester ou servir votre application.
Ces commandes utilisent un outil interne appelé Architect pour exécuter des _builders de la CLI_, qui invoquent à leur tour un autre outil (bundler, test runner, serveur) pour accomplir la tâche souhaitée.
Des builders personnalisés peuvent réaliser une tâche entièrement nouvelle, ou changer l'outil tiers utilisé par une commande existante.

Ce document explique comment les builders de la CLI s'intègrent au fichier de configuration du workspace, et montre comment créer votre propre builder.

HELPFUL: Retrouvez le code des exemples utilisés ici dans ce [dépôt GitHub](https://github.com/mgechev/cli-builders-demo).

## Les builders de la CLI {#cli-builders}

L'outil interne Architect délègue le travail à des fonctions de traitement appelées _builders_.
La fonction de traitement d'un builder reçoit deux arguments :

| Argument  | Type             |
| :-------- | :--------------- |
| `options` | `JSONObject`     |
| `context` | `BuilderContext` |

La séparation des responsabilités est ici la même qu'avec les [schematics](tools/cli/schematics-authoring), utilisés par d'autres commandes de la CLI qui modifient votre code (comme `ng generate`).

- L'objet `options` est fourni à partir des options et de la configuration de l'utilisateur de la CLI, tandis que l'objet `context` est fourni automatiquement par l'API CLI Builder.
- En plus des informations contextuelles, l'objet `context` donne aussi accès à une méthode de planification, `context.scheduleTarget()`.
  Le planificateur exécute la fonction de traitement du builder avec une configuration de cible donnée.

La fonction de traitement du builder peut être synchrone (renvoyer une valeur), asynchrone (renvoyer une `Promise`), ou observer et renvoyer plusieurs valeurs (renvoyer un `Observable`).
Les valeurs renvoyées doivent toujours être de type `BuilderOutput`.
Cet objet contient un champ booléen `success` et un champ facultatif `error` qui peut contenir un message d'erreur.

Angular fournit des builders utilisés par la CLI pour des commandes comme `ng build` et `ng test`.
Les configurations de cibles par défaut de ces builders et des autres builders intégrés à la CLI se trouvent et se configurent dans la section « architect » du [fichier de configuration du workspace](reference/configs/workspace-config), `angular.json`.
Vous pouvez aussi étendre et personnaliser Angular en créant vos propres builders, que vous pouvez exécuter directement avec la [commande CLI `ng run`](cli/run).

### Structure d'un projet de builder {#builder-project-structure}

Un builder se trouve dans un dossier de « projet » dont la structure est semblable à celle d'un workspace Angular, avec les fichiers de configuration globaux au niveau supérieur, et une configuration plus spécifique dans un dossier source contenant les fichiers de code qui définissent le comportement.
Par exemple, votre dossier `myBuilder` pourrait contenir les fichiers suivants.

| Fichiers                 | Rôle                                                                                                   |
| :----------------------- | :-------------------------------------------------------------------------------------------------------- |
| `src/my-builder.ts`      | Fichier source principal de la définition du builder.                                                              |
| `src/my-builder.spec.ts` | Fichier source des tests.                                                                                    |
| `src/schema.json`        | Définition des options d'entrée du builder.                                                                      |
| `builders.json`          | Définition des builders.                                                                                      |
| `package.json`           | Dépendances. Consultez [https://docs.npmjs.com/files/package.json](https://docs.npmjs.com/files/package.json). |
| `tsconfig.json`          | [Configuration TypeScript](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html).              |

Les builders peuvent être publiés sur `npm` : consultez [Publier votre library](tools/libraries/creating-libraries).

## Créer un builder {#creating-a-builder}

À titre d'exemple, créez un builder qui copie un fichier vers un nouvel emplacement.
Pour créer un builder, utilisez la fonction `createBuilder()` de CLI Builder et renvoyez un objet `Promise<BuilderOutput>`.

<docs-code header="src/my-builder.ts (skeleton du builder)" path="adev/src/content/examples/cli-builder/src/my-builder.ts" region="builder-skeleton"/>

Ajoutons-y maintenant un peu de logique.
Le code suivant récupère les chemins des fichiers source et de destination dans les options de l'utilisateur, puis copie le fichier de la source vers la destination \(à l'aide de la [version promise de la fonction Node.js intégrée `copyFile()`](https://nodejs.org/api/fs.html#fs_fspromises_copyfile_src_dest_mode)\).
Si l'opération de copie échoue, il renvoie une erreur avec un message décrivant le problème sous-jacent.

<docs-code header="src/my-builder.ts (builder)" path="adev/src/content/examples/cli-builder/src/my-builder.ts" region="builder"/>

### Gérer la sortie {#handling-output}

Par défaut, `copyFile()` n'affiche rien sur la sortie standard ni sur la sortie d'erreur du processus.
Si une erreur survient, il peut être difficile de comprendre exactement ce que le builder tentait de faire au moment du problème.
Ajoutez du contexte en journalisant des informations supplémentaires à l'aide de l'API `Logger`.
Cela permet aussi d'exécuter le builder lui-même dans un processus séparé, même si la sortie standard et la sortie d'erreur sont désactivées.

Vous pouvez récupérer une instance de `Logger` depuis le contexte.

<docs-code header="src/my-builder.ts (gestion de la sortie)" path="adev/src/content/examples/cli-builder/src/my-builder.ts" region="handling-output"/>

### Signaler la progression et le statut {#progress-and-status-reporting}

L'API CLI Builder inclut des outils de signalement de la progression et du statut, qui peuvent fournir des indications pour certaines fonctions et interfaces.

Pour signaler la progression, utilisez la méthode `context.reportProgress()`, qui prend en arguments une valeur courante, un total facultatif et une chaîne de statut.
Le total peut être n'importe quel nombre. Par exemple, si vous savez combien de fichiers vous devez traiter, le total pourrait être le nombre de fichiers, et la valeur courante le nombre de fichiers traités jusqu'ici.
La chaîne de statut reste inchangée tant que vous ne passez pas une nouvelle valeur de chaîne.

Dans notre exemple, l'opération de copie est soit terminée, soit encore en cours : un rapport de progression est donc inutile, mais vous pouvez signaler le statut pour qu'un builder parent ayant appelé notre builder sache ce qui se passe.
Utilisez la méthode `context.reportStatus()` pour générer une chaîne de statut de n'importe quelle longueur.

HELPFUL: Rien ne garantit qu'une longue chaîne sera affichée en entier ; elle peut être tronquée pour s'adapter à l'interface qui l'affiche.

Passez une chaîne vide pour supprimer le statut.

<docs-code header="src/my-builder.ts (signalement de la progression)" path="adev/src/content/examples/cli-builder/src/my-builder.ts" region="progress-reporting"/>

## Entrées d'un builder {#builder-input}

Vous pouvez invoquer un builder indirectement via une commande de la CLI comme `ng build`, ou directement avec la commande `ng run` de l'Angular CLI.
Dans les deux cas, vous devez fournir les entrées obligatoires, mais vous pouvez laisser les autres entrées prendre des valeurs par défaut préconfigurées pour une _cible_ spécifique, définies par une [configuration](tools/cli/environments), ou indiquées sur la ligne de commande.

### Validation des entrées {#input-validation}

Vous définissez les entrées d'un builder dans un schéma JSON associé à ce builder.
Comme pour les schematics, l'outil Architect regroupe les valeurs d'entrée résolues dans un objet `options`, et valide leurs types par rapport au schéma avant de les passer à la fonction du builder.

Pour notre builder d'exemple, `options` doit être un `JsonObject` avec deux clés :
`source` et `destination`, chacune étant une chaîne.

Vous pouvez fournir le schéma suivant pour valider le type de ces valeurs.

```json {header: "schema.json"}
{
  "$schema": "https://json-schema.org/schema",
  "type": "object",
  "properties": {
    "source": {
      "type": "string"
    },
    "destination": {
      "type": "string"
    }
  }
}
```

HELPFUL: Il s'agit d'un exemple minimal, mais l'utilisation d'un schéma pour la validation peut s'avérer très puissante.
Pour plus d'informations, consultez le [site des schémas JSON](https://json-schema.org).

Pour relier l'implémentation de notre builder à son schéma et à son nom, vous devez créer un fichier de _définition de builder_, vers lequel vous pouvez pointer dans `package.json`.

Créez un fichier nommé `builders.json` qui ressemble à ceci :

```json {header: "builders.json"}
{
  "builders": {
    "copy": {
      "implementation": "./dist/my-builder.js",
      "schema": "./src/schema.json",
      "description": "Copies a file."
    }
  }
}
```

Dans le fichier `package.json`, ajoutez une clé `builders` qui indique à l'outil Architect où trouver notre fichier de définition de builder.

```json {header: "package.json"}
{
  "name": "@example/copy-file",
  "version": "1.0.0",
  "description": "Builder for copying files",
  "builders": "builders.json",
  "dependencies": {
    "@angular/build": "^21.2.0"
  }
}
```

Le nom officiel de notre builder est désormais `@example/copy-file:copy`.
La première partie correspond au nom du package et la seconde au nom du builder tel que défini dans le fichier `builders.json`.

Ces valeurs sont accessibles via `options.source` et `options.destination`.

<docs-code header="src/my-builder.ts (signalement du statut)" path="adev/src/content/examples/cli-builder/src/my-builder.ts" region="report-status"/>

### Configuration de la cible {#target-configuration}

Un builder doit avoir une cible définie qui l'associe à une configuration d'entrée et à un projet spécifiques.

Les cibles sont définies dans le [fichier de configuration de la CLI](reference/configs/workspace-config) `angular.json`.
Une cible indique le builder à utiliser, sa configuration d'options par défaut et des configurations alternatives nommées.
Architect, dans l'Angular CLI, utilise la définition de la cible pour résoudre les options d'entrée d'une exécution donnée.

Le fichier `angular.json` comporte une section pour chaque projet, et la section « architect » de chaque projet configure les cibles des builders utilisés par les commandes de la CLI comme « build », « test » et « serve ».
Par défaut, par exemple, la commande `ng build` exécute le builder `@angular/build:application` pour réaliser la tâche de build, et lui passe les valeurs d'options par défaut indiquées pour la cible `build` dans `angular.json`.

```json {header: "angular.json"}
{
  "myApp": {
    "...": "...",
    "architect": {
      "build": {
        "builder": "@angular/build:application",
        "options": {
          "outputPath": "dist/myApp",
          "index": "src/index.html",
          "...": "..."
        },
        "configurations": {
          "production": {
            "fileReplacements": [
              {
                "replace": "src/environments/environment.ts",
                "with": "src/environments/environment.prod.ts"
              }
            ],
            "optimization": true,
            "outputHashing": "all",
            "...": "..."
          }
        }
      },
      "...": "..."
    }
  }
}
```

La commande passe au builder l'ensemble des options par défaut indiquées dans la section « options ».
Si vous passez l'option `--configuration=production`, elle utilise les valeurs de remplacement indiquées dans la configuration `production`.
Indiquez d'autres remplacements d'options individuellement sur la ligne de commande.

#### Chaînes de cible {#target-strings}

La commande générique `ng run` de la CLI prend comme premier argument une chaîne de cible de la forme suivante.

```shell

project:target[:configuration]

```

|               | Détails                                                                                                               |
| :------------ | :-------------------------------------------------------------------------------------------------------------------- |
| project       | Le nom du projet Angular CLI auquel la cible est associée.                                               |
| target        | Une configuration de builder nommée, issue de la section `architect` du fichier `angular.json`.                                |
| configuration | (facultatif) Le nom d'une configuration de remplacement spécifique pour la cible donnée, telle que définie dans le fichier `angular.json`. |

Si votre builder appelle un autre builder, il peut avoir besoin de lire une chaîne de cible qui lui est passée.
Analysez cette chaîne pour la convertir en objet à l'aide de la fonction utilitaire `targetFromTargetString()` de `@angular-devkit/architect`.

## Planifier et exécuter {#schedule-and-run}

Architect exécute les builders de manière asynchrone.
Pour invoquer un builder, vous planifiez une tâche qui sera exécutée une fois la résolution de toute la configuration terminée.

La fonction du builder n'est pas exécutée tant que le planificateur n'a pas renvoyé un objet de contrôle `BuilderRun`.
La CLI planifie généralement les tâches en appelant la fonction `context.scheduleTarget()`, puis résout les options d'entrée à l'aide de la définition de la cible dans le fichier `angular.json`.

Architect résout les options d'entrée d'une cible donnée en prenant l'objet des options par défaut, puis en écrasant ses valeurs avec celles de la configuration, puis en les écrasant à nouveau avec celles de l'objet de remplacements passé à `context.scheduleTarget()`.
Pour l'Angular CLI, l'objet de remplacements est construit à partir des arguments de la ligne de commande.

Architect valide les valeurs d'options obtenues par rapport au schéma du builder.
Si les entrées sont valides, Architect crée le contexte et exécute le builder.

Pour plus d'informations, consultez [Configuration du workspace](reference/configs/workspace-config).

HELPFUL: Vous pouvez aussi invoquer un builder directement depuis un autre builder ou un test en appelant `context.scheduleBuilder()`.
Vous passez directement un objet `options` à la méthode, et ces valeurs d'options sont validées par rapport au schéma du builder sans autre ajustement.

Seule la méthode `context.scheduleTarget()` résout la configuration et les remplacements via le fichier `angular.json`.

### Configuration architect par défaut {#default-architect-configuration}

Créons un fichier `angular.json` simple qui met en contexte les configurations de cibles.

Vous pouvez publier le builder sur npm (consultez [Publier votre library](tools/libraries/creating-libraries#publishing-your-library)), et l'installer avec la commande suivante :

```shell

npm install @example/copy-file

```

Si vous créez un nouveau projet avec `ng new builder-test`, le fichier `angular.json` généré ressemble à ceci, avec uniquement les configurations de builders par défaut.

```json {header: "angular.json"}
{
  "projects": {
    "builder-test": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "outputPath": "dist/builder-test",
            "index": "src/index.html",
            "browser": "src/main.ts",
            "tsConfig": "tsconfig.app.json"
          },
          "configurations": {
            "production": {
              "optimization": true,
              "aot": true
            }
          }
        }
      }
    }
  }
}
```

### Ajouter une cible {#adding-a-target}

Ajoutez une nouvelle cible qui exécutera notre builder pour copier un fichier.
Cette cible indique au builder de copier le fichier `package.json`.

- Nous allons ajouter une nouvelle section de cible à l'objet `architect` de notre projet
- La cible nommée `copy-package` utilise notre builder, que vous avez publié sous `@example/copy-file`.
- L'objet options fournit des valeurs par défaut pour les deux entrées que vous avez définies.
  - `source` - Le fichier existant que vous copiez.
  - `destination` - Le chemin vers lequel vous voulez copier.

```json {header: "angular.json"}
{
  "projects": {
    "builder-test": {
      "architect": {
        "copy-package": {
          "builder": "@example/copy-file:copy",
          "options": {
            "source": "package.json",
            "destination": "package-copy.json"
          }
        }
        // Existing targets...
      }
    }
  }
}
```

### Exécuter le builder {#running-the-builder}

Pour exécuter notre builder avec la configuration par défaut de la nouvelle cible, utilisez la commande CLI suivante.

```shell

ng run builder-test:copy-package

```

Cela copie le fichier `package.json` vers `package-copy.json`.

Utilisez des arguments de ligne de commande pour remplacer les valeurs par défaut configurées.
Par exemple, pour exécuter le builder avec une autre valeur de `destination`, utilisez la commande CLI suivante.

```shell

ng run builder-test:copy-package --destination=package-other.json

```

Cela copie le fichier vers `package-other.json` au lieu de `package-copy.json`.
Comme vous n'avez pas remplacé l'option _source_, la copie se fait toujours à partir du fichier par défaut `package.json`.

## Tester un builder {#testing-a-builder}

Utilisez des tests d'intégration pour votre builder, afin de pouvoir utiliser le planificateur d'Architect pour créer un contexte, comme dans cet [exemple](https://github.com/mgechev/cli-builders-demo).
Dans le répertoire source du builder, créez un nouveau fichier de test `my-builder.spec.ts`. Le test crée de nouvelles instances de `JsonSchemaRegistry` (pour la validation du schéma), `TestingArchitectHost` (une implémentation en mémoire de `ArchitectHost`) et `Architect`.

Voici un exemple de test qui exécute le builder de copie de fichier.
Le test utilise le builder pour copier le fichier `package.json` et vérifie que le contenu du fichier copié est identique à celui de la source.

<docs-code header="src/my-builder.spec.ts" path="adev/src/content/examples/cli-builder/src/my-builder.spec.ts"/>

HELPFUL: Pour exécuter ce test dans votre dépôt, vous avez besoin du package [`ts-node`](https://github.com/TypeStrong/ts-node).
Vous pouvez éviter cela en renommant `my-builder.spec.ts` en `my-builder.spec.js`.

### Mode watch {#watch-mode}

La plupart des builders s'exécutent une seule fois puis rendent la main. Toutefois, ce comportement n'est pas entièrement compatible avec un builder qui surveille les modifications (comme un serveur de développement, par exemple).
Architect peut prendre en charge le mode watch, mais il faut faire attention à certains points.

- Pour être utilisée avec le mode watch, la fonction de traitement d'un builder doit renvoyer un `Observable`.
  Architect s'abonne à l'`Observable` jusqu'à ce qu'il se termine, et peut le réutiliser si le builder est de nouveau planifié avec les mêmes arguments.

- Le builder doit toujours émettre un objet `BuilderOutput` après chaque exécution.
  Une fois exécuté, il peut passer en mode watch, pour être déclenché par un événement externe.
  Si un événement le relance, le builder doit exécuter la fonction `context.reportRunning()` pour indiquer à Architect qu'il est de nouveau en cours d'exécution.
  Cela empêche Architect d'arrêter le builder si une autre exécution est planifiée.

Lorsque votre builder appelle `BuilderRun.stop()` pour quitter le mode watch, Architect se désabonne de l'`Observable` du builder et appelle la logique de libération du builder pour faire le nettoyage.
Ce comportement permet aussi d'arrêter et de nettoyer les builds de longue durée.

En général, si votre builder surveille un événement externe, vous devez séparer votre exécution en trois phases.

| Phases     | Détails                                                                                                                                                                                                                                       |
| :--------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Exécution  | La tâche en cours, comme l'invocation d'un compilateur. Elle se termine lorsque le compilateur a fini et que votre builder émet un objet `BuilderOutput`.                                                                                                  |
| Surveillance | Entre deux exécutions, surveiller un flux d'événements externes. Par exemple, surveiller les modifications du système de fichiers. Elle se termine lorsque le compilateur redémarre et que `context.reportRunning()` est appelée.                                                          |
| Achèvement | Soit la tâche est entièrement terminée, comme un compilateur qui doit s'exécuter un certain nombre de fois, soit l'exécution du builder a été arrêtée (avec `BuilderRun.stop()`). Architect exécute la logique de libération et se désabonne de l'`Observable` de votre builder. |

## Résumé {#summary}

L'API CLI Builder permet de modifier le comportement de l'Angular CLI en utilisant des builders pour exécuter une logique personnalisée.

- Les builders peuvent être synchrones ou asynchrones, s'exécuter une seule fois ou surveiller des événements externes, et peuvent planifier d'autres builders ou cibles.
- Les builders ont des options par défaut indiquées dans le fichier de configuration `angular.json`, qui peuvent être écrasées par une configuration alternative de la cible, puis à nouveau par les options de la ligne de commande
- L'équipe Angular vous recommande d'utiliser des tests d'intégration pour tester les builders Architect. Utilisez des tests unitaires pour valider la logique exécutée par le builder.
- Si votre builder renvoie un `Observable`, il doit nettoyer le builder dans la logique de libération (teardown) de cet `Observable`.
