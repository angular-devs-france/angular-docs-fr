<!-- source: a49a0f5a3e41a8f76406168615974fb21104cf68 -->
# Dépendances npm du workspace

Le framework Angular, l'Angular CLI et les composants utilisés par les applications Angular sont distribués sous forme de [packages npm](https://docs.npmjs.com/getting-started/what-is-npm 'What is npm?') via le [registre npm](https://docs.npmjs.com).

Vous pouvez télécharger et installer ces packages npm avec le [client CLI npm](https://docs.npmjs.com/cli/install).
Par défaut, l'Angular CLI utilise le client npm.

HELPFUL: Consultez [Configuration de l'environnement local](tools/cli/setup-local 'Setting up for Local Development') pour connaître les versions requises de `Node.js` et `npm` et la façon de les installer.

Si des projets utilisant d'autres versions de Node.js et npm tournent déjà sur votre machine, envisagez d'utiliser [nvm](https://github.com/creationix/nvm) pour gérer les différentes versions de Node.js et npm.

## `package.json`

`npm` installe les packages listés dans un fichier [`package.json`](https://docs.npmjs.com/files/package.json).

La commande CLI `ng new` crée un fichier `package.json` lorsqu'elle crée le nouveau workspace.
Ce `package.json` est utilisé par tous les projets du workspace, y compris le projet d'application initial que la CLI crée en même temps que le workspace.
Les libraries créées avec `ng generate library` incluent leur propre fichier `package.json`.

Au départ, ce `package.json` contient _un ensemble de packages de démarrage_, dont certains sont requis par Angular et d'autres prennent en charge des scénarios d'application courants.
Vous ajoutez des packages au `package.json` à mesure que votre application évolue.

## Dépendances par défaut {#default-dependencies}

Les packages Angular suivants sont inclus comme dépendances dans le fichier `package.json` par défaut d'un nouveau workspace Angular.
Pour la liste complète des packages Angular, consultez la [référence de l'API](api).

| Nom du package | Détails |
| :--------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`@angular/common`](api#angular_common) | Les services, pipes et directives d'usage courant fournis par l'équipe Angular. |
| `@angular/compiler` | Le compilateur de templates d'Angular. Il comprend les templates Angular et peut les convertir en code qui fait fonctionner l'application. |
| `@angular/compiler-cli` | Le compilateur d'Angular, appelé par les commandes `ng build` et `ng serve` de l'Angular CLI. Il traite les templates Angular avec `@angular/compiler` au sein d'une compilation TypeScript standard. |
| [`@angular/core`](api#angular_core) | Les parties critiques du framework à l'exécution, nécessaires à toute application. Inclut tous les décorateurs de métadonnées comme `@Component`, l'injection de dépendances et les lifecycle hooks des composants. |
| [`@angular/forms`](api#angular_forms) | Prise en charge des [template-driven forms](guide/forms) et des [reactive forms](guide/forms/reactive-forms). Consultez [Introduction aux formulaires](guide/forms). |
| [`@angular/platform-browser`](api#angular_platform-browser) | Tout ce qui concerne le DOM et le navigateur, en particulier les éléments qui aident au rendu dans le DOM. |
| [`@angular/router`](api#angular_router) | Le module du router navigue entre les pages de votre application lorsque l'URL du navigateur change. Pour plus d'informations, consultez [Routing et navigation](guide/routing). |
| [`@angular/cli`](https://github.com/angular/angular-cli) | Contient le binaire de l'Angular CLI qui permet d'exécuter les commandes `ng`. |
| [`@angular/build`](https://www.npmjs.com/package/@angular/build) | Contient les builders CLI par défaut pour bundler, tester et servir les applications et libraries Angular. |
| [`rxjs`](https://www.npmjs.com/package/rxjs) | Une library de programmation réactive basée sur les `Observables`. |
| [`typescript`](https://www.npmjs.com/package/typescript) | Le compilateur TypeScript, le serveur de langage et les définitions de types intégrées. |
