<!-- source: 0869946e19fa27e3827ee5be5635156d6e37db98 -->
# Structure des fichiers du workspace et des projets

Vous développez des applications dans le contexte d'un workspace Angular.
Un workspace contient les fichiers d'un ou plusieurs projets.
Un projet est l'ensemble des fichiers qui constituent une application ou une library partageable.

La commande `ng new` de l'Angular CLI crée un workspace.

```shell
ng new my-project
```

Lorsque vous lancez cette commande, la CLI installe les packages npm Angular nécessaires et les autres dépendances dans un nouveau workspace, avec une application au niveau racine nommée _my-project_.

Par défaut, `ng new` crée une application squelette initiale au niveau racine du workspace.
Ce squelette correspond à une application d'accueil simple, prête à être lancée et facile à modifier.
L'application au niveau racine porte le même nom que le workspace, et ses fichiers source se trouvent dans le sous-dossier `src/` du workspace.

Ce comportement par défaut convient à un style de développement « multi-repo » classique, où chaque application réside dans son propre workspace.
Nous encourageons les débutants et les utilisateurs intermédiaires à utiliser `ng new` pour créer un workspace distinct pour chaque application.

Angular prend aussi en charge les workspaces à [plusieurs projets](#multiple-projects).
Ce type d'environnement de développement convient aux utilisateurs avancés qui développent des libraries partageables,
ainsi qu'aux entreprises qui adoptent un style de développement « monorepo », avec un dépôt unique et une configuration globale pour tous les projets Angular.

Pour mettre en place un workspace monorepo, ne créez pas l'application racine.
Consultez [Configurer un workspace multi-projets](#multiple-projects) ci-dessous.

## Fichiers de configuration du workspace {#workspace-configuration-files}

Tous les projets d'un workspace partagent une [configuration](reference/configs/workspace-config).
Le niveau racine du workspace contient les fichiers de configuration communs à tout le workspace, les fichiers de configuration de l'application racine, ainsi que des sous-dossiers pour les fichiers source et de test de l'application racine.

| Fichiers de configuration du workspace | Rôle |
| :---------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `.editorconfig` | Configuration pour les éditeurs de code. Consultez [EditorConfig](https://editorconfig.org). |
| `.gitignore` | Indique les fichiers volontairement non suivis que [Git](https://git-scm.com) doit ignorer. |
| `README.md` | Documentation du workspace. |
| `angular.json` | Configuration de la CLI pour tous les projets du workspace, y compris les options de configuration qui déterminent comment compiler, servir et tester chaque projet. Pour plus de détails, consultez [Configuration du workspace Angular](reference/configs/workspace-config). |
| `package.json` | Configure les [dépendances de packages npm](reference/configs/npm-packages) disponibles pour tous les projets du workspace. Consultez la [documentation npm](https://docs.npmjs.com/files/package.json) pour le format et le contenu précis de ce fichier. |
| `package-lock.json` | Fournit les informations de version de tous les packages installés dans `node_modules` par le client npm. Consultez la [documentation npm](https://docs.npmjs.com/files/package-lock.json) pour plus de détails. |
| `src/` | Fichiers source du projet d'application racine. |
| `public/` | Contient les images et autres ressources statiques servies par le serveur de développement et copiées telles quelles lors du build de votre application. |
| `node_modules/` | [Packages npm](reference/configs/npm-packages) installés pour l'ensemble du workspace. Les dépendances `node_modules` du workspace sont visibles par tous les projets. |
| `tsconfig.json` | La configuration [TypeScript](https://www.typescriptlang.org) de base pour les projets du workspace. Tous les autres fichiers de configuration héritent de ce fichier de base. Pour plus d'informations, consultez la [documentation TypeScript correspondante](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html#tsconfig-bases). |

## Fichiers du projet d'application {#application-project-files}

Par défaut, la commande CLI `ng new my-app` crée un dossier de workspace nommé « my-app » et génère un nouveau squelette d'application dans un dossier `src/` au niveau racine du workspace.
Une application nouvellement générée contient les fichiers source d'un composant racine et de son template.

Une fois la structure de fichiers du workspace en place, vous pouvez utiliser la commande `ng generate` en ligne de commande pour ajouter des fonctionnalités et des données à l'application.
Cette application racine initiale est l'_application par défaut_ des commandes CLI (sauf si vous changez ce défaut après avoir créé [d'autres applications](#multiple-projects)).

Dans un workspace à application unique, le sous-dossier `src` du workspace contient les fichiers source (logique applicative, données et ressources) de l'application racine.
Dans un workspace multi-projets, les projets supplémentaires du dossier `projects` contiennent un sous-dossier `project-name/src/` doté de la même structure.

### Fichiers source de l'application {#application-source-files}

Les fichiers au niveau racine de `src/` permettent de faire tourner votre application.
Les sous-dossiers contiennent le code source de l'application et sa configuration spécifique.

| Fichiers de support de l'application | Rôle |
| :------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `app/` | Contient les fichiers de composants dans lesquels la logique et les données de votre application sont définies. Voir les détails ci-dessous. |
| `index.html` | La page HTML principale servie lorsqu'une personne visite votre site. La CLI ajoute automatiquement tous les fichiers JavaScript et CSS lors du build de votre application : vous n'avez donc généralement pas besoin d'ajouter manuellement de balises `<script>` ou`<link>` ici. |
| `main.ts` | Le point d'entrée principal de votre application. |
| `styles.css` | Les styles CSS globaux appliqués à toute l'application. |

Dans le dossier `src`, le dossier `app` contient la logique et les données de votre projet.
C'est là que se trouvent les composants, templates et styles Angular.

| Fichiers de `src/app/` | Rôle |
| :--------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `app.config.ts` | Définit la configuration de l'application, qui indique à Angular comment assembler l'application. Les providers que vous ajoutez à l'application doivent être déclarés ici.<br><br>_Généré uniquement avec l'option `--standalone`._ |
| `app.ts` | Définit le composant racine de l'application, nommé `App`. La vue associée à ce composant racine devient la racine de la hiérarchie des vues à mesure que vous ajoutez des composants et des services à votre application. |
| `app.html` | Définit le template HTML associé à `App`. |
| `app.css` | Définit la feuille de style CSS de `App`. |
| `app.spec.ts` | Définit un test unitaire pour `App`. |
| `app-module.ts` | Définit le module racine, nommé `AppModule`, qui indique à Angular comment assembler l'application. Au départ, il ne déclare que le composant `App`. Les composants que vous ajoutez à l'application doivent être déclarés ici.<br><br>_Généré uniquement avec l'option `--standalone false`._ |
| `app.routes.ts` | Définit la configuration du routing de l'application. |

### Fichiers de configuration de l'application {#application-configuration-files}

Les fichiers de configuration spécifiques à l'application racine se trouvent au niveau racine du workspace.
Dans un workspace multi-projets, les fichiers de configuration spécifiques à un projet se trouvent à la racine du projet, sous `projects/project-name/`.

Les fichiers de configuration [TypeScript](https://www.typescriptlang.org) spécifiques à un projet héritent du fichier `tsconfig.json` commun au workspace.

| Fichiers de configuration spécifiques à l'application | Rôle |
| :--------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tsconfig.app.json` | [Configuration TypeScript](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) spécifique à l'application, y compris les [options du compilateur Angular](reference/configs/angular-compiler-options). |
| `tsconfig.spec.json` | [Configuration TypeScript](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) pour les tests de l'application. |

## Plusieurs projets {#multiple-projects}

Un workspace multi-projets convient à une organisation qui utilise un dépôt unique et une configuration globale pour plusieurs projets Angular (le modèle « monorepo »).
Un workspace multi-projets permet aussi de développer des libraries.

### Configurer un workspace multi-projets {#setting-up-for-a-multi-project-workspace}

Si vous prévoyez d'avoir plusieurs projets dans un workspace, vous pouvez ignorer la génération de l'application initiale lors de la création du workspace, et donner au workspace un nom unique.
La commande suivante crée un workspace avec tous les fichiers de configuration communs au workspace, mais sans application au niveau racine.

```shell
ng new my-workspace --no-create-application
```

Vous pouvez ensuite générer des applications et des libraries avec des noms uniques au sein du workspace.

```shell
cd my-workspace
ng generate application my-app
ng generate library my-lib
```

### Structure de fichiers à plusieurs projets {#multiple-project-file-structure}

La première application générée explicitement va dans le dossier `projects`, avec tous les autres projets du workspace.
Les libraries nouvellement générées sont également ajoutées sous `projects`.
Lorsque vous créez des projets de cette façon, la structure de fichiers du workspace correspond entièrement à la structure du [fichier de configuration du workspace](reference/configs/workspace-config), `angular.json`.

```markdown
my-workspace/
├── … (workspace-wide configuration files)
└── projects/ (applications and libraries)
├── my-app/ (an explicitly generated application)
│ └── … (application-specific code and configuration)
└── my-lib/ (a generated library)
└── … (library-specific code and configuration)
```

## Fichiers d'un projet de library {#library-project-files}

Lorsque vous générez une library avec la CLI (avec une commande comme `ng generate library my-lib`), les fichiers générés vont dans le dossier `projects/` du workspace.
Pour plus d'informations sur la création de vos propres libraries, consultez [Créer des libraries](tools/libraries/creating-libraries).

Contrairement à une application, une library possède son propre fichier de configuration `package.json`.

Dans le dossier `projects/`, le dossier `my-lib` contient le code de votre library.

| Fichiers source de la library | Rôle |
| :----------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `src/lib` | Contient la logique et les données de votre projet de library. Comme un projet d'application, un projet de library peut contenir des composants, des services, des modules, des directives et des pipes. |
| `src/public-api.ts` | Indique tous les fichiers exportés par votre library. |
| `ng-package.json` | Fichier de configuration utilisé par [ng-packagr](https://github.com/ng-packagr/ng-packagr) pour compiler votre library. |
| `package.json` | Configure les [dépendances de packages npm](reference/configs/npm-packages) requises par cette library. |
| `tsconfig.lib.json` | [Configuration TypeScript](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) spécifique à la library, y compris les [options du compilateur Angular](reference/configs/angular-compiler-options). |
| `tsconfig.lib.prod.json` | [Configuration TypeScript](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) spécifique à la library, utilisée lors du build de la library en mode production. |
| `tsconfig.spec.json` | [Configuration TypeScript](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) pour les tests unitaires de la library. |
