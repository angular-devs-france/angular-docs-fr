<!-- source: 95d73f29e4fd3976cbbb64327e4083da22696a08 -->
# Écrire des schematics

Vous pouvez créer vos propres schematics pour agir sur des projets Angular.
Les développeurs de libraries incluent généralement des schematics dans leurs libraries pour les intégrer à l'Angular CLI.
Vous pouvez aussi créer des schematics autonomes pour manipuler les fichiers et les structures des applications Angular, afin de les adapter à votre environnement de développement et de les rendre conformes à vos standards et à vos contraintes.
Les schematics peuvent être chaînés, en exécutant d'autres schematics pour réaliser des opérations complexes.

Manipuler le code d'une application peut s'avérer à la fois très puissant et, en conséquence, dangereux.
Par exemple, créer un fichier qui existe déjà serait une erreur et, si l'opération était appliquée immédiatement, elle annulerait toutes les autres modifications appliquées jusque-là.
L'outillage Angular Schematics protège contre les effets de bord et les erreurs en créant un système de fichiers virtuel.
Un schematic décrit un pipeline de transformations qui peuvent être appliquées au système de fichiers virtuel.
Lorsqu'un schematic s'exécute, les transformations sont enregistrées en mémoire, et ne sont appliquées au système de fichiers réel qu'une fois leur validité confirmée.

## Concepts des schematics {#schematics-concepts}

L'API publique des schematics définit des classes qui représentent les concepts de base.

- Le système de fichiers virtuel est représenté par un `Tree`.
  La structure de données `Tree` contient une _base_ \(un ensemble de fichiers qui existent déjà\) et une _zone de préparation_ \(staging area : une liste de modifications à appliquer à la base\).
  Lorsque vous effectuez des modifications, vous ne changez pas réellement la base : vous ajoutez ces modifications à la zone de préparation.

- Un objet `Rule` définit une fonction qui prend un `Tree`, applique des transformations et renvoie un nouveau `Tree`.
  Le fichier principal d'un schematic, `index.ts`, définit un ensemble de règles qui implémentent la logique du schematic.

- Une transformation est représentée par une `Action`.
  Il existe quatre types d'actions : `Create`, `Rename`, `Overwrite` et `Delete`.

- Chaque schematic s'exécute dans un contexte, représenté par un objet `SchematicContext`.

L'objet de contexte passé à une règle donne accès à des fonctions utilitaires et aux métadonnées dont le schematic peut avoir besoin, notamment une API de journalisation pour faciliter le débogage.
Le contexte définit aussi une _stratégie de fusion_ qui détermine comment les modifications sont fusionnées depuis l'arbre de préparation vers l'arbre de base.
Une modification peut être acceptée, ignorée, ou lever une exception.

### Définir des règles et des actions {#defining-rules-and-actions}

Lorsque vous créez un nouveau schematic vierge avec la [CLI Schematics](#schematics-cli), la fonction d'entrée générée est une _fabrique de règles_.
Un objet `RuleFactory` définit une fonction d'ordre supérieur qui crée une `Rule`.

```ts {header: "index.ts"}
import {Rule, SchematicContext, Tree} from '@angular-devkit/schematics';

// You don't have to export the function as default.
// You can also have more than one rule factory per file.
export function helloWorld(_options: any): Rule {
  return (tree: Tree, _context: SchematicContext) => {
    return tree;
  };
}
```

Vos règles peuvent modifier vos projets en appelant des outils externes et en implémentant de la logique.
Vous avez par exemple besoin d'une règle pour définir comment un template du schematic doit être fusionné dans le projet hôte.

Les règles peuvent utiliser les utilitaires fournis avec le package `@schematics/angular`.
Recherchez les fonctions utilitaires pour manipuler les modules, les dépendances, TypeScript, l'AST, le JSON, les workspaces et projets Angular CLI, et plus encore.

```ts {header: "index.ts"}
import {
  JsonAstObject,
  JsonObject,
  JsonValue,
  Path,
  normalize,
  parseJsonAst,
  strings,
} from '@angular-devkit/core';
```

### Définir les options d'entrée avec un schéma et des interfaces {#defining-input-options-with-a-schema-and-interfaces}

Les règles peuvent récupérer des valeurs d'options auprès de l'appelant et les injecter dans des templates.
Les options disponibles pour vos règles, avec leurs valeurs autorisées et leurs valeurs par défaut, sont définies dans le fichier de schéma JSON du schematic, `<schematic>/schema.json`.
Définissez les types de données variables ou énumérés du schéma à l'aide d'interfaces TypeScript.

Le schéma définit les types et les valeurs par défaut des variables utilisées dans le schematic.
Par exemple, l'hypothétique schematic « Hello World » pourrait avoir le schéma suivant.

```json {header: "schema.json"}
{
  "properties": {
    "name": {
      "type": "string",
      "minLength": 1,
      "default": "world"
    },
    "useColor": {
      "type": "boolean"
    }
  }
}
```

Consultez des exemples de fichiers de schéma pour les schematics des commandes de l'Angular CLI dans [`@schematics/angular`](https://github.com/angular/angular-cli/blob/main/packages/schematics/angular/application/schema.json).

### Prompts de schematic {#schematic-prompts}

Les _prompts_ de schematic introduisent une interaction avec l'utilisateur dans l'exécution d'un schematic.
Configurez les options du schematic pour afficher une question personnalisable à l'utilisateur.
Les prompts sont affichés avant l'exécution du schematic, qui utilise ensuite la réponse comme valeur de l'option.
Cela permet aux utilisateurs d'orienter le fonctionnement du schematic sans avoir besoin de connaître en détail tout l'éventail des options disponibles.

Le schematic « Hello World » pourrait par exemple demander son nom à l'utilisateur, puis afficher ce nom à la place du nom par défaut « world ».
Pour définir un tel prompt, ajoutez une propriété `x-prompt` au schéma de la variable `name`.

De même, vous pouvez ajouter un prompt pour laisser l'utilisateur décider si le schematic utilise la couleur lors de l'exécution de son action hello.
Le schéma avec les deux prompts serait le suivant.

```json {header: "schema.json"}
{
  "properties": {
    "name": {
      "type": "string",
      "minLength": 1,
      "default": "world",
      "x-prompt": "What is your name?"
    },
    "useColor": {
      "type": "boolean",
      "x-prompt": "Would you like the response in color?"
    }
  }
}
```

#### Syntaxe courte des prompts {#prompt-short-form-syntax}

Ces exemples utilisent une forme abrégée de la syntaxe des prompts, qui ne fournit que le texte de la question.
Dans la plupart des cas, c'est tout ce qui est nécessaire.
Remarquez cependant que les deux prompts attendent des types d'entrée différents.
Avec la forme abrégée, le type le plus approprié est sélectionné automatiquement en fonction du schéma de la propriété.
Dans l'exemple, le prompt `name` utilise le type `input` parce qu'il s'agit d'une propriété de type chaîne.
Le prompt `useColor` utilise un type `confirmation` parce qu'il s'agit d'une propriété booléenne.
Dans ce cas, « yes » correspond à `true` et « no » correspond à `false`.

Trois types d'entrée sont pris en charge.

| Type d'entrée | Détails                                            |
| :----------- | :------------------------------------------------- |
| confirmation | Une question à répondre par oui ou non ; idéal pour les options booléennes.   |
| input        | Saisie de texte ; idéal pour les options de type chaîne ou nombre. |
| list         | Un ensemble prédéfini de valeurs autorisées.                |

Dans la forme courte, le type est déduit du type et des contraintes de la propriété.

| Schéma de la propriété | Type de prompt                                  |
| :---------------- | :------------------------------------------- |
| "type": "boolean" | confirmation \("yes"=`true`, "no"=`false`\)  |
| "type": "string"  | input                                        |
| "type": "number"  | input \(seuls les nombres valides sont acceptés\)        |
| "type": "integer" | input \(seuls les nombres valides sont acceptés\)        |
| "enum": […]       | list \(les membres de l'enum deviennent les choix de la liste\) |

Dans l'exemple suivant, la propriété prend une valeur énumérée : le schematic choisit donc automatiquement le type list et crée un menu à partir des valeurs possibles.

```json {header: "schema.json"}
{
  "style": {
    "description": "The file extension or preprocessor to use for style files.",
    "type": "string",
    "default": "css",
    "enum": ["css", "scss", "sass", "less", "styl"],
    "x-prompt": "Which stylesheet format would you like to use?"
  }
}
```

Le runtime des prompts valide automatiquement la réponse fournie par rapport aux contraintes définies dans le schéma JSON.
Si la valeur n'est pas acceptable, une nouvelle valeur est demandée à l'utilisateur.
Cela garantit que toutes les valeurs passées au schematic répondent aux attentes de son implémentation, sans que vous ayez à ajouter de vérifications supplémentaires dans le code du schematic.

#### Syntaxe longue des prompts {#prompt-long-form-syntax}

La syntaxe du champ `x-prompt` prend en charge une forme longue pour les cas où vous avez besoin de davantage de personnalisation et de contrôle sur le prompt.
Dans cette forme, la valeur du champ `x-prompt` est un objet JSON dont les sous-champs personnalisent le comportement du prompt.

| Champ   | Valeur                                                                  |
| :------ | :-------------------------------------------------------------------------- |
| type    | `confirmation`, `input` ou `list` \(sélectionné automatiquement dans la forme courte\) |
| message | chaîne \(obligatoire\)                                                         |
| items   | chaîne et/ou paire label/value sous forme d'objet \(valide uniquement avec le type `list`\)       |

L'exemple suivant de forme longue provient du schéma JSON du schematic que la CLI utilise pour [générer des applications](https://github.com/angular/angular-cli/blob/ba8a6ea59983bb52a6f1e66d105c5a77517f062e/packages/schematics/angular/application/schema.json#L56).
Il définit le prompt qui permet aux utilisateurs de choisir le préprocesseur de styles à utiliser pour l'application en cours de création.
Grâce à la forme longue, le schematic peut formater plus explicitement les choix du menu.

```json {header: "schema.json"}
{
  "style": {
    "description": "The file extension or preprocessor to use for style files.",
    "type": "string",
    "default": "css",
    "enum": ["css", "scss", "sass", "less"],
    "x-prompt": {
      "message": "Which stylesheet format would you like to use?",
      "type": "list",
      "items": [
        {"value": "css", "label": "CSS"},
        {"value": "scss", "label": "SCSS [ https://sass-lang.com/documentation/syntax#scss ]"},
        {
          "value": "sass",
          "label": "Sass [ https://sass-lang.com/documentation/syntax#the-indented-syntax ]"
        },
        {"value": "less", "label": "Less [ https://lesscss.org/ ]"}
      ]
    }
  }
}
```

#### Schéma x-prompt {#x-prompt-schema}

Le schéma JSON qui définit les options d'un schematic prend en charge des extensions permettant de définir de manière déclarative les prompts et leur comportement respectif.
Aucune logique supplémentaire ni modification du code d'un schematic n'est nécessaire pour prendre en charge les prompts.
Le schéma JSON suivant décrit complètement la syntaxe longue du champ `x-prompt`.

```json {header: "x-prompt schema"}
{
  "oneOf": [
    {
      "type": "string"
    },
    {
      "type": "object",
      "properties": {
        "type": {
          "type": "string"
        },
        "message": {
          "type": "string"
        },
        "items": {
          "type": "array",
          "items": {
            "oneOf": [
              {
                "type": "string"
              },
              {
                "type": "object",
                "properties": {
                  "label": {
                    "type": "string"
                  },
                  "value": {}
                },
                "required": ["value"]
              }
            ]
          }
        }
      },
      "required": ["message"]
    }
  ]
}
```

## CLI Schematics {#schematics-cli}

Les schematics disposent de leur propre outil en ligne de commande.
Installez l'outil en ligne de commande Schematics globalement :

```shell

npm install -g @angular-devkit/schematics-cli

```

Cela installe l'exécutable `schematics`, que vous pouvez utiliser pour créer une nouvelle collection de schematics dans son propre dossier de projet, ajouter un nouveau schematic à une collection existante ou étendre un schematic existant.

Dans les sections suivantes, vous allez créer une nouvelle collection de schematics à l'aide de la CLI afin de découvrir les fichiers, la structure des fichiers et quelques concepts de base.

L'usage le plus courant des schematics reste toutefois l'intégration d'une library Angular à l'Angular CLI.
Pour cela, créez les fichiers du schematic directement dans le projet de la library, au sein d'un workspace Angular, sans utiliser la CLI Schematics.
Consultez [Schematics pour les libraries](tools/cli/schematics-for-libraries).

### Créer une collection de schematics {#creating-a-schematics-collection}

La commande suivante crée un nouveau schematic nommé `hello-world` dans un nouveau dossier de projet du même nom.

```shell

schematics blank --name=hello-world

```

Le schematic `blank` est fourni par la CLI Schematics.
La commande crée un nouveau dossier de projet \(le dossier racine de la collection\) et un premier schematic nommé dans la collection.

Allez dans le dossier de la collection, installez vos dépendances npm et ouvrez votre nouvelle collection dans votre éditeur favori pour voir les fichiers générés.
Par exemple, si vous utilisez VS Code :

```shell

cd hello-world
npm install
npm run build
code .

```

Le schematic initial reçoit le même nom que le dossier du projet et est généré dans `src/hello-world`.
Ajoutez des schematics liés à cette collection et modifiez le code squelette généré pour définir les fonctionnalités de votre schematic.
Chaque nom de schematic doit être unique au sein de la collection.

### Exécuter un schematic {#running-a-schematic}

Utilisez la commande `schematics` pour exécuter un schematic nommé.
Indiquez le chemin du dossier du projet, le nom du schematic et les options obligatoires, au format suivant.

```shell

schematics <path-to-schematics-project>:<schematics-name> --<required-option>=<value>

```

Le chemin peut être absolu ou relatif au répertoire de travail courant dans lequel la commande est exécutée.
Par exemple, pour exécuter le schematic que vous venez de générer \(qui n'a pas d'option obligatoire\), utilisez la commande suivante.

```shell

schematics .:hello-world

```

### Ajouter un schematic à une collection {#adding-a-schematic-to-a-collection}

Pour ajouter un schematic à une collection existante, utilisez la même commande que pour démarrer un nouveau projet de schematics, mais exécutez-la dans le dossier du projet.

```shell

cd hello-world
schematics blank --name=goodbye-world

```

La commande génère le nouveau schematic nommé dans votre collection, avec un fichier principal `index.ts` et son fichier de spec de test associé.
Elle ajoute aussi le nom, la description et la fonction de fabrique du nouveau schematic au schéma de la collection, dans le fichier `collection.json`.

## Contenu d'une collection {#collection-contents}

Le niveau supérieur du dossier racine du projet d'une collection contient des fichiers de configuration, un dossier `node_modules` et un dossier `src/`.
Le dossier `src/` contient des sous-dossiers pour les schematics nommés de la collection, ainsi qu'un schéma, `collection.json`, qui décrit les schematics regroupés.
Chaque schematic est créé avec un nom, une description et une fonction de fabrique.

```json
{
  "$schema": "../node_modules/@angular-devkit/schematics/collection-schema.json",
  "schematics": {
    "hello-world": {
      "description": "A blank schematic.",
      "factory": "./hello-world/index#helloWorld"
    }
  }
}
```

- La propriété `$schema` indique le schéma que la CLI utilise pour la validation.
- La propriété `schematics` liste les schematics nommés qui appartiennent à cette collection.
  Chaque schematic possède une description en texte brut et pointe vers la fonction d'entrée générée dans le fichier principal.

- La propriété `factory` pointe vers la fonction d'entrée générée.
  Dans cet exemple, vous invoquez le schematic `hello-world` en appelant la fonction de fabrique `helloWorld()`.

- La propriété facultative `schema` pointe vers un fichier de schéma JSON qui définit les options de ligne de commande disponibles pour le schematic.
- Le tableau facultatif `aliases` indique une ou plusieurs chaînes qui peuvent être utilisées pour invoquer le schematic.
  Par exemple, le schematic de la commande « generate » de l'Angular CLI a un alias « g », qui vous permet d'utiliser la commande `ng g`.

### Schematics nommés {#named-schematics}

Lorsque vous utilisez la CLI Schematics pour créer un projet de schematics vierge, le nouveau schematic vierge est le premier membre de la collection et porte le même nom que celle-ci.
Lorsque vous ajoutez un nouveau schematic nommé à cette collection, il est automatiquement ajouté au schéma `collection.json`.

En plus du nom et de la description, chaque schematic possède une propriété `factory` qui identifie son point d'entrée.
Dans l'exemple, vous invoquez la fonctionnalité définie par le schematic en appelant la fonction `helloWorld()` dans le fichier principal, `hello-world/index.ts`.

<img alt="vue d'ensemble" src="assets/images/guide/schematics/collection-files.gif">

Chaque schematic nommé de la collection comporte les parties principales suivantes.

| Parties       | Détails                                                           |
| :------------ | :---------------------------------------------------------------- |
| `index.ts`    | Code qui définit la logique de transformation d'un schematic nommé. |
| `schema.json` | Définition des variables du schematic.                                    |
| `schema.d.ts` | Variables du schematic.                                              |
| `files/`      | Fichiers de composants/templates facultatifs à répliquer.                   |

Un schematic peut fournir toute sa logique dans le fichier `index.ts`, sans templates supplémentaires.
Vous pouvez toutefois créer des schematics dynamiques pour Angular en fournissant des composants et des templates dans le dossier `files`, comme ceux des projets Angular standalone.
La logique du fichier index configure ces templates en définissant des règles qui injectent des données et modifient des variables.
