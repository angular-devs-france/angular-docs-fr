<!-- source: e7d4ca1718a2190491b307f6154c93745d7b6155 -->
# Schematics pour les libraries

Lorsque vous créez une library Angular, vous pouvez la fournir et la packager avec des schematics qui l'intègrent à l'Angular CLI.
Grâce à vos schematics, vos utilisateurs peuvent utiliser `ng add` pour installer une version initiale de votre library,
`ng generate` pour créer des artefacts définis dans votre library, et `ng update` pour adapter leur projet à une nouvelle version de votre library qui introduit des breaking changes.

Ces trois types de schematics peuvent faire partie d'une collection que vous packagez avec votre library.

## Créer une collection de schematics {#creating-a-schematics-collection}

Pour démarrer une collection, vous devez créer les fichiers du schematic.
Les étapes suivantes vous montrent comment ajouter une prise en charge initiale sans modifier aucun fichier du projet.

1. Dans le dossier racine de votre library, créez un dossier `schematics`.
1. Dans le dossier `schematics/`, créez un dossier `ng-add` pour votre premier schematic.
1. À la racine du dossier `schematics`, créez un fichier `collection.json`.
1. Modifiez le fichier `collection.json` pour définir le schéma initial de votre collection.

   <docs-code header="projects/my-lib/schematics/collection.json (Collection de schematics)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/collection.1.json"/>
   - Le chemin `$schema` est relatif au schéma de collection de l'Angular Devkit.
   - L'objet `schematics` décrit les schematics nommés qui font partie de cette collection.
   - La première entrée concerne un schematic nommé `ng-add`.
     Elle contient la description et pointe vers la fonction de fabrique appelée lors de l'exécution de votre schematic.

1. Dans le fichier `package.json` du projet de votre library, ajoutez une entrée « schematics » avec le chemin vers votre fichier de schéma.
   L'Angular CLI utilise cette entrée pour trouver les schematics nommés de votre collection lorsqu'elle exécute des commandes.

<docs-code header="projects/my-lib/package.json (Référence à la collection de schematics)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/package.json" region="collection"/>

Le schéma initial que vous avez créé indique à la CLI où trouver le schematic qui prend en charge la commande `ng add`.
Vous êtes maintenant prêt à créer ce schematic.

## Prendre en charge l'installation {#providing-installation-support}

Un schematic pour la commande `ng add` peut améliorer le processus d'installation initiale pour vos utilisateurs.
Les étapes suivantes définissent ce type de schematic.

1. Accédez au dossier `<lib-root>/schematics/ng-add`.
1. Créez un fichier `schema.json` pour définir les options acceptées par le schematic.

   <docs-code header="projects/my-lib/schematics/ng-add/schema.json (Schéma ng-add)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/ng-add/schema.json"/>

1. Créez un fichier `schema.ts` pour définir une interface pour les options définies dans le fichier `schema.json`.

   <docs-code header="projects/my-lib/schematics/ng-add/schema.ts (Interface du schéma ng-add)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/ng-add/schema.ts"/>

1. Créez le fichier principal, `index.ts`, et ajoutez le code source de la fonction de fabrique de votre schematic.

   <docs-code header="projects/my-lib/schematics/ng-add/index.ts (Fabrique de règle ng-add)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/ng-add/index.ts"/>

L'Angular CLI installe automatiquement la dernière version de la library, et cet exemple va plus loin en ajoutant `MyLibModule` à la racine de l'application. La fonction `addRootImport` accepte un callback qui doit renvoyer un bloc de code. Vous pouvez écrire n'importe quel code dans la chaîne étiquetée avec la fonction `code`, et tout symbole externe doit être enveloppé dans la fonction `external` pour garantir que les instructions d'import appropriées sont générées.

### Définir le type de dépendance {#define-dependency-type}

Utilisez l'option `save` de `ng-add` pour indiquer si la library doit être ajoutée aux `dependencies`, aux `devDependencies`, ou ne pas être enregistrée du tout dans le fichier de configuration `package.json` du projet.

<docs-code header="projects/my-lib/package.json (Référence ng-add)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/package.json" region="ng-add"/>

Les valeurs possibles sont :

| Valeurs             | Détails                                 |
| :------------------ | :-------------------------------------- |
| `false`             | N'ajoute pas le package à `package.json` |
| `true`              | Ajoute le package aux dependencies      |
| `"dependencies"`    | Ajoute le package aux dependencies      |
| `"devDependencies"` | Ajoute le package aux devDependencies   |

## Compiler vos schematics {#building-your-schematics}

Pour intégrer vos schematics au bundle de votre library, vous devez configurer la library pour qu'elle compile les schematics séparément, puis les ajouter au bundle.
Vous devez compiler vos schematics _après_ avoir compilé votre library, afin qu'ils soient placés dans le bon répertoire.

- Votre library a besoin d'un fichier de configuration TypeScript personnalisé contenant les instructions pour compiler vos schematics dans votre library distribuée
- Pour ajouter les schematics au bundle de la library, ajoutez des scripts au fichier `package.json` de la library

Supposons que vous ayez un projet de library `my-lib` dans votre workspace Angular.
Pour indiquer à la library comment compiler les schematics, ajoutez un fichier `tsconfig.schematics.json` à côté du fichier `tsconfig.lib.json` généré, qui configure le build de la library.

1. Modifiez le fichier `tsconfig.schematics.json` pour y ajouter le contenu suivant.

   <docs-code header="projects/my-lib/tsconfig.schematics.json (Configuration TypeScript)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/tsconfig.schematics.json"/>

   | Options   | Détails                                                                                                          |
   | :-------- | :--------------------------------------------------------------------------------------------------------------- |
   | `rootDir` | Indique que votre dossier `schematics` contient les fichiers d'entrée à compiler.                                |
   | `outDir`  | Correspond au dossier de sortie de la library. Par défaut, il s'agit du dossier `dist/my-lib` à la racine de votre workspace. |

1. Pour vous assurer que les fichiers source de vos schematics sont compilés dans le bundle de la library, ajoutez les scripts suivants au fichier `package.json` situé dans le dossier racine du projet de votre library \(`projects/my-lib`\).

   <docs-code header="projects/my-lib/package.json (Scripts de build)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/package.json"/>
   - Le script `build` compile votre schematic à l'aide du fichier personnalisé `tsconfig.schematics.json`
   - Le script `postbuild` copie les fichiers du schematic une fois le script `build` terminé
   - Les scripts `build` et `postbuild` nécessitent tous deux les dépendances `copyfiles` et `typescript`.
     Pour installer les dépendances, placez-vous dans le chemin défini dans `devDependencies` et exécutez `npm install` avant de lancer les scripts.

## Prendre en charge la génération {#providing-generation-support}

Vous pouvez ajouter à votre collection un schematic nommé qui permet à vos utilisateurs d'utiliser la commande `ng generate` pour créer un artefact défini dans votre library.

Supposons que votre library définisse un service, `my-service`, qui nécessite une certaine configuration.
Vous voulez que vos utilisateurs puissent le générer avec la commande CLI suivante.

```shell

ng generate my-lib:my-service

```

Pour commencer, créez un nouveau sous-dossier, `my-service`, dans le dossier `schematics`.

### Configurer le nouveau schematic {#configure-the-new-schematic}

Lorsque vous ajoutez un schematic à la collection, vous devez le référencer dans le schéma de la collection et fournir des fichiers de configuration pour définir les options qu'un utilisateur peut passer à la commande.

1. Modifiez le fichier `schematics/collection.json` pour pointer vers le sous-dossier du nouveau schematic, et ajoutez une référence vers un fichier de schéma qui spécifie les entrées du nouveau schematic.

   <docs-code header="projects/my-lib/schematics/collection.json (Collection de schematics)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/collection.json"/>

1. Accédez au dossier `<lib-root>/schematics/my-service`.
1. Créez un fichier `schema.json` et définissez les options disponibles pour le schematic.

   <docs-code header="projects/my-lib/schematics/my-service/schema.json (Schéma JSON du schematic)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/schema.json"/>
   - _id_ : un identifiant unique pour le schéma dans la collection.
   - _title_ : une description du schéma lisible par un humain.
   - _type_ : un descripteur du type fourni par les propriétés.
   - _properties_ : un objet qui définit les options disponibles pour le schematic.

   Chaque option associe une clé à un type, une description et un alias facultatif.
   Le type définit la forme de la valeur attendue, et la description s'affiche lorsque l'utilisateur demande l'aide d'utilisation de votre schematic.

   Consultez le schéma du workspace pour d'autres personnalisations des options de schematic.

1. Créez un fichier `schema.ts` et définissez une interface qui stocke les valeurs des options définies dans le fichier `schema.json`.

   <docs-code header="projects/my-lib/schematics/my-service/schema.ts (Interface du schematic)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/schema.ts"/>

   | Options | Détails                                                                                                                                     |
   | :------ | :------------------------------------------------------------------------------------------------------------------------------------------ |
   | name    | Le nom que vous voulez donner au service créé.                                                                                              |
   | path    | Remplace le chemin fourni au schematic. La valeur par défaut du chemin est basée sur le répertoire de travail courant.                       |
   | project | Indique un projet précis sur lequel exécuter le schematic. Dans le schematic, vous pouvez fournir une valeur par défaut si l'utilisateur ne fournit pas l'option. |

### Ajouter les fichiers de template {#add-template-files}

Pour ajouter des artefacts à un projet, votre schematic a besoin de ses propres fichiers de template.
Les templates de schematic prennent en charge une syntaxe spéciale pour exécuter du code et substituer des variables.

1. Créez un dossier `files/` dans le dossier `schematics/my-service/`.
1. Créez un fichier nommé `__name@dasherize__.service.ts.template` qui définit un template à utiliser pour générer des fichiers.
   Ce template génère un service dans lequel le `HttpClient` d'Angular est déjà injecté dans une propriété `http`.

   ```ts {header: "projects/my-lib/schematics/my-service/files/__name@dasherize__.service.ts.template (Schematic Template)"}

   import { Service } from '@angular/core';
   import { HttpClient } from '@angular/common/http';

   @Service()
   export class <%= classify(name) %>Service {
      private http = inject(HttpClient);
   }

   ```

   - Les méthodes `classify` et `dasherize` sont des fonctions utilitaires que votre schematic utilise pour transformer votre template source et le nom de fichier.
   - Le `name` est fourni sous forme de propriété par votre fonction de fabrique.
     Il s'agit du même `name` que celui défini dans le schéma.

### Ajouter la fonction de fabrique {#add-the-factory-function}

Maintenant que l'infrastructure est en place, vous pouvez définir la fonction principale qui effectue les modifications nécessaires dans le projet de l'utilisateur.

Le framework Schematics fournit un système de templating de fichiers, qui prend en charge à la fois les templates de chemin et de contenu.
Le système opère sur des placeholders définis dans les fichiers ou les chemins chargés dans l'arbre (`Tree`) d'entrée.
Il les remplit à l'aide des valeurs passées à la règle (`Rule`).

Pour plus de détails sur ces structures de données et cette syntaxe, consultez le [README de Schematics](https://github.com/angular/angular-cli/blob/main/packages/angular_devkit/schematics/README.md).

1. Créez le fichier principal `index.ts` et ajoutez le code source de la fonction de fabrique de votre schematic.
1. Commencez par importer les définitions de schematics dont vous aurez besoin.
   Le framework Schematics propose de nombreuses fonctions utilitaires pour créer et utiliser des règles lors de l'exécution d'un schematic.

   <docs-code header="projects/my-lib/schematics/my-service/index.ts (Imports)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/index.ts" region="schematics-imports"/>

1. Importez l'interface de schéma définie, qui fournit les informations de type pour les options de votre schematic.

   <docs-code header="projects/my-lib/schematics/my-service/index.ts (Import du schéma)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/index.ts" region="schema-imports"/>

1. Pour construire le schematic de génération, commencez par une fabrique de règle vide.

   <docs-code header="projects/my-lib/schematics/my-service/index.ts (Règle initiale)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/index.1.ts" region="factory"/>

Cette fabrique de règle renvoie l'arbre sans modification.
Les options sont les valeurs d'options transmises par la commande `ng generate`.

## Définir une règle de génération {#define-a-generation-rule}

Vous disposez maintenant du cadre nécessaire pour écrire le code qui modifie réellement l'application de l'utilisateur afin de la configurer pour le service défini dans votre library.

Le workspace Angular dans lequel l'utilisateur a installé votre library contient plusieurs projets \(applications et libraries\).
L'utilisateur peut indiquer le projet en ligne de commande, ou laisser la valeur par défaut.
Dans les deux cas, votre code doit identifier le projet précis auquel ce schematic est appliqué, afin de pouvoir récupérer des informations dans la configuration du projet.

Faites-le à l'aide de l'objet `Tree` passé à la fonction de fabrique.
Les méthodes de `Tree` vous donnent accès à l'arbre de fichiers complet de votre workspace, ce qui vous permet de lire et d'écrire des fichiers pendant l'exécution du schematic.

### Obtenir la configuration du projet {#get-the-project-configuration}

1. Pour déterminer le projet de destination, utilisez la méthode `workspaces.readWorkspace` afin de lire le contenu du fichier de configuration du workspace, `angular.json`.
   Pour utiliser `workspaces.readWorkspace`, vous devez créer un `workspaces.WorkspaceHost` à partir du `Tree`.
   Ajoutez le code suivant à votre fonction de fabrique.

   <docs-code header="projects/my-lib/schematics/my-service/index.ts (Import du schéma)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/index.ts" region="workspace"/>

   Veillez à vérifier que le contexte existe et à lever l'erreur appropriée.

1. Maintenant que vous avez le nom du projet, utilisez-le pour récupérer les informations de configuration propres au projet.

   <docs-code header="projects/my-lib/schematics/my-service/index.ts (Projet)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/index.ts" region="project-info"/>

   L'objet `workspace.projects` contient toutes les informations de configuration propres aux projets.

1. `options.path` détermine l'emplacement où les fichiers de template du schematic sont déplacés une fois le schematic appliqué.

   Par défaut, l'option `path` du schéma du schematic est remplacée par le répertoire de travail courant.
   Si `path` n'est pas défini, utilisez le `sourceRoot` de la configuration du projet ainsi que le `projectType`.

   <docs-code header="projects/my-lib/schematics/my-service/index.ts (Informations du projet)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/index.ts" region="path"/>

### Définir la règle {#define-the-rule}

Une `Rule` peut utiliser des fichiers de template externes, les transformer et renvoyer un autre objet `Rule` avec le template transformé.
Utilisez le templating pour générer tous les fichiers personnalisés nécessaires à votre schematic.

1. Ajoutez le code suivant à votre fonction de fabrique.

   <docs-code header="projects/my-lib/schematics/my-service/index.ts (Transformation du template)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/index.ts" region="template"/>

   | Méthodes           | Détails                                                                                                                                                                                                                                          |
   | :----------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | `apply()`          | Applique plusieurs règles à une source et renvoie la source transformée. Elle prend 2 arguments : une source et un tableau de règles.                                                                                                             |
   | `url()`            | Lit les fichiers source depuis votre système de fichiers, relativement au schematic.                                                                                                                                                             |
   | `applyTemplates()` | Reçoit en argument les méthodes et propriétés que vous voulez rendre disponibles pour le template du schematic et les noms de fichiers du schematic. Elle renvoie une `Rule`. C'est ici que vous définissez les méthodes `classify()` et `dasherize()`, ainsi que la propriété `name`. |
   | `classify()`       | Prend une valeur et la renvoie avec des majuscules en début de mot. Par exemple, si le nom fourni est `my service`, il est renvoyé sous la forme `MyService`.                                                                                     |
   | `dasherize()`      | Prend une valeur et la renvoie en minuscules, avec des tirets. Par exemple, si le nom fourni est MyService, il est renvoyé sous la forme `my-service`.                                                                                            |
   | `move()`           | Déplace les fichiers source fournis vers leur destination lorsque le schematic est appliqué.                                                                                                                                                     |

1. Enfin, la fabrique de règle doit renvoyer une règle.

   <docs-code header="projects/my-lib/schematics/my-service/index.ts (Règle chain)" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/index.ts" region="chain"/>

   La méthode `chain()` vous permet de combiner plusieurs règles en une seule, afin d'effectuer plusieurs opérations dans un même schematic.
   Ici, vous fusionnez simplement les règles de template avec le code exécuté par le schematic.

Voici un exemple complet de la fonction de règle de schematic suivante.

<docs-code header="projects/my-lib/schematics/my-service/index.ts" path="adev/src/content/examples/schematics-for-libraries/projects/my-lib/schematics/my-service/index.ts"/>

Pour plus d'informations sur les règles et les méthodes utilitaires, consultez [Provided Rules](https://github.com/angular/angular-cli/tree/main/packages/angular_devkit/schematics#provided-rules).

## Exécuter le schematic de votre library {#running-your-library-schematic}

Après avoir compilé votre library et vos schematics, vous pouvez installer la collection de schematics pour l'exécuter sur votre projet.
Les étapes suivantes vous montrent comment générer un service à l'aide du schematic créé précédemment.

### Compiler votre library et vos schematics {#build-your-library-and-schematics}

Depuis la racine de votre workspace, exécutez la commande `ng build` pour votre library.

```shell

ng build my-lib

```

Placez-vous ensuite dans le répertoire de votre library pour compiler le schematic

```shell

cd projects/my-lib
npm run build

```

### Lier la library {#link-the-library}

Votre library et vos schematics sont packagés et placés dans le dossier `dist/my-lib` à la racine de votre workspace.
Pour exécuter le schematic, vous devez lier la library dans votre dossier `node_modules`.
Depuis la racine de votre workspace, exécutez la commande `npm link` avec le chemin vers votre library distribuable.

```shell

npm link dist/my-lib

```

### Exécuter le schematic {#run-the-schematic}

Maintenant que votre library est installée, exécutez le schematic avec la commande `ng generate`.

```shell

ng generate my-lib:my-service --name my-data

```

Dans la console, vous voyez que le schematic a été exécuté et que le fichier `my-data.service.ts` a été créé dans le dossier de votre application.

```shell {hideCopy}

CREATE src/app/my-data.service.ts (208 bytes)

```
