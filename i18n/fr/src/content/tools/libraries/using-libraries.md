<!-- source: f44204331f961d92d84b058bbbb61d299339356f -->
# Utiliser des libraries Angular publiées sur npm

Quand vous construisez votre application Angular, tirez parti de libraries officielles sophistiquées, ainsi que d'un riche écosystème de libraries tierces.
[Angular Material][AngularMaterialMain] est un exemple de library officielle sophistiquée.

## Installer des libraries {#install-libraries}

Les libraries sont publiées sous forme de [packages npm][GuideNpmPackages], généralement accompagnées de schematics qui les intègrent à l'Angular CLI.
Pour intégrer le code réutilisable d'une library dans une application, vous devez installer le package et importer la fonctionnalité fournie à l'endroit où vous l'utilisez.
Pour la plupart des libraries Angular publiées, utilisez la commande Angular CLI `ng add <lib_name>`.

La commande Angular CLI `ng add` utilise un gestionnaire de packages pour installer le package de la library et invoque les schematics inclus dans le package pour ajouter d'autres éléments de scaffolding au code du projet.
Parmi les gestionnaires de packages, on trouve par exemple [npm][NpmjsMain] ou [yarn][YarnpkgMain].
Le scaffolding supplémentaire dans le code du projet comprend des instructions d'import, des polices et des thèmes.

Une library publiée fournit généralement un fichier `README` ou une autre documentation expliquant comment ajouter cette library à votre application.
Pour un exemple, consultez la documentation d'[Angular Material][AngularMaterialMain].

### Typages des libraries {#library-typings}

En général, les packages de libraries incluent des typages dans des fichiers `.d.ts` ; voyez des exemples dans `node_modules/@angular/material`.
Si le package de votre library n'inclut pas de typages et que votre IDE s'en plaint, vous devrez peut-être installer le package `@types/<lib_name>` avec la library.

Par exemple, supposons que vous ayez une library nommée `d3` :

```shell

npm install d3 --save
npm install @types/d3 --save-dev

```

Les types définis dans un package `@types/` pour une library installée dans le workspace sont automatiquement ajoutés à la configuration TypeScript du projet qui utilise cette library.
Par défaut, TypeScript cherche les types dans le répertoire `node_modules/@types`, vous n'avez donc pas à ajouter chaque package de types individuellement.

Si une library ne dispose pas de typages dans `@types/`, vous pouvez quand même l'utiliser en lui ajoutant des typages manuellement.
Pour cela :

1. Créez un fichier `typings.d.ts` dans votre répertoire `src/`.
   Ce fichier est automatiquement inclus comme définition de types globale.

1. Ajoutez le code suivant dans `src/typings.d.ts` :

   ```ts
   declare module 'host' {
     export interface Host {
       protocol?: string;
       hostname?: string;
       pathname?: string;
     }
     export function parse(url: string, queryString?: string): Host;
   }
   ```

1. Dans le composant ou le fichier qui utilise la library, ajoutez le code suivant :

   ```ts
   import * as host from 'host';
   const parsedUrl = host.parse('https://angular.dev');
   console.log(parsedUrl.hostname);
   ```

Définissez d'autres typages selon vos besoins.

## Mettre à jour des libraries {#updating-libraries}

Une library peut être mise à jour par son éditeur, et possède aussi ses propres dépendances qu'il faut maintenir à jour.
Pour vérifier les mises à jour de vos libraries installées, utilisez la commande Angular CLI [`ng update`][CliUpdate].

Utilisez la commande Angular CLI `ng update <lib_name>` pour mettre à jour la version d'une library précise.
L'Angular CLI vérifie la dernière version publiée de la library et, si elle est plus récente que la version installée, la télécharge et met à jour votre `package.json` pour correspondre à la dernière version.

Quand vous mettez à jour Angular vers une nouvelle version, vous devez vous assurer que toutes les libraries que vous utilisez sont à jour.
Si des libraries dépendent les unes des autres, vous devrez peut-être les mettre à jour dans un ordre particulier.
Consultez l'[Angular Update Guide][AngularUpdateMain] pour vous aider.

## Ajouter une library à la portée globale d'exécution {#adding-a-library-to-the-runtime-global-scope}

Si une library JavaScript historique n'est pas importée dans une application, vous pouvez l'ajouter à la portée globale d'exécution et la charger comme si elle avait été ajoutée dans une balise script.
Configurez l'Angular CLI pour le faire au moment du build en utilisant les options `scripts` et `styles` de la cible de build dans le fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig].

Par exemple, pour utiliser la library [Bootstrap 4][GetbootstrapDocs40GettingStartedIntroduction]

1. Installez la library et ses dépendances associées avec le gestionnaire de packages npm :

   ```shell
     npm install jquery --save
     npm install popper.js --save
     npm install bootstrap --save
   ```

1. Dans le fichier de configuration `angular.json`, ajoutez les fichiers de script associés au tableau `scripts` :

   ```json
     "scripts": [
         "node_modules/jquery/dist/jquery.slim.js",
         "node_modules/popper.js/dist/umd/popper.js",
         "node_modules/bootstrap/dist/js/bootstrap.js"
       ],
   ```

1. Ajoutez le fichier CSS `bootstrap.css` au tableau `styles` :

   ```json
     "styles": [
         "node_modules/bootstrap/dist/css/bootstrap.css",
         "src/styles.css"
     ],
   ```

1. Lancez ou relancez la commande Angular CLI `ng serve` pour voir Bootstrap 4 fonctionner dans votre application.

### Utiliser des libraries globales d'exécution dans votre application {#using-runtime-global-libraries-inside-your-app}

Après avoir importé une library via le tableau « scripts », ne l'importez **pas** avec une instruction d'import dans votre code TypeScript.
L'extrait de code suivant est un exemple d'instruction d'import.

```ts
import * as $ from 'jquery';
```

Si vous l'importez avec des instructions d'import, vous obtenez deux copies différentes de la library : l'une importée comme library globale, l'autre importée comme module.
C'est particulièrement problématique pour les libraries dotées de plugins, comme jQuery, car chaque copie inclut des plugins différents.

À la place, lancez `npm install @types/jquery` pour télécharger les typages de votre library, ajoutez `"jquery"` au tableau `types` dans `tsconfig.app.json`, puis suivez les étapes d'installation de la library.
Vous avez ainsi accès aux variables globales exposées par cette library.

### Définir des typages pour les libraries globales d'exécution {#defining-typings-for-runtime-global-libraries}

Si la library globale que vous devez utiliser ne dispose pas de typages globaux, vous pouvez les déclarer manuellement comme `any` dans `src/typings.d.ts`.

Par exemple :

```ts
declare var libraryName: any;
```

Certains scripts étendent d'autres libraries, par exemple avec des plugins jQuery :

```ts
$('.test').myPlugin();
```

Dans ce cas, le package `@types/jquery` installé n'inclut pas `myPlugin`, vous devez donc ajouter une interface dans `src/typings.d.ts`.
Par exemple :

```ts
interface JQuery {
  myPlugin(options?: any): any;
}
```

Si vous n'ajoutez pas l'interface pour l'extension définie par le script, votre IDE affiche une erreur :

```text

[TS][Error] Property 'myPlugin' does not exist on type 'JQuery'

```

[CliUpdate]: cli/update 'ng update | CLI |Angular'
[GuideNpmPackages]: reference/configs/npm-packages 'Workspace npm dependencies | Angular'
[GuideWorkspaceConfig]: reference/configs/workspace-config 'Angular workspace configuration | Angular'
[Resources]: resources 'Explore Angular Resources | Angular'
[AngularMaterialMain]: https://material.angular.dev 'Angular Material | Angular'
[AngularUpdateMain]: /update-guide 'Angular Update Guide | Angular'
[GetbootstrapDocs40GettingStartedIntroduction]: https://getbootstrap.com/docs/4.0/getting-started/introduction 'Introduction | Bootstrap'
[NpmjsMain]: https://www.npmjs.com 'npm'
[YarnpkgMain]: https://yarnpkg.com ' Yarn'
