<!-- source: c81ff0a2d7889263a56b4cdef9169386e7238df9 -->
# Ajouter le package localize

Pour tirer parti des fonctionnalités de localisation d'Angular, utilisez l'[Angular CLI][CliMain] pour ajouter le package `@angular/localize` à votre projet.

Pour ajouter le package `@angular/localize`, utilisez la commande suivante pour mettre à jour le fichier `package.json` et les fichiers de configuration TypeScript de votre projet.

<docs-code language="shell" path="adev/src/content/examples/i18n/doc-files/commands.sh" region="add-localize"/>

Elle ajoute `types: ["@angular/localize"]` dans les fichiers de configuration TypeScript.
Elle ajoute aussi la ligne `/// <reference types="@angular/localize" />` en haut du fichier `main.ts`, qui est la référence à la définition de types.

HELPFUL: Pour en savoir plus sur les fichiers `package.json` et `tsconfig.json`, consultez [Dépendances npm du workspace][GuideNpmPackages] et [Configuration TypeScript][GuideTsConfig]. Pour en savoir plus sur les directives triple-slash, consultez le [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html#-reference-types-).

Si `@angular/localize` n'est pas installé et que vous essayez de construire une version localisée de votre projet (par exemple, en utilisant l'attribut `i18n` dans les templates), l'[Angular CLI][CliMain] génère une erreur qui indique les étapes à suivre pour activer l'i18n dans votre projet.

## Options {#options}

| OPTION             | DESCRIPTION                                                                                                                                                                                   | TYPE DE VALEUR | VALEUR PAR DÉFAUT |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------- | :------------ |
| `--project`        | Le nom du projet.                                                                                                                                                                      | `string`   |
| `--use-at-runtime` | Si cette option est définie, `$localize` peut être utilisé à l'exécution. De plus, `@angular/localize` est inclus dans la section `dependencies` du `package.json` plutôt que dans `devDependencies`, qui est la valeur par défaut. | `boolean`  | `false`       |

Pour connaître les autres options disponibles, consultez `ng add` dans l'[Angular CLI][CliMain].

## Et ensuite ? {#whats-next}

<docs-pill-row>
  <docs-pill href="guide/i18n/locale-id" title="Désigner les locales par leur identifiant"/>
</docs-pill-row>

[CliMain]: cli 'CLI Overview and Command Reference | Angular'
[GuideNpmPackages]: reference/configs/npm-packages 'Workspace npm dependencies | Angular'
[GuideTsConfig]: https://www.typescriptlang.org/docs/handbook/tsconfig-json.html 'TypeScript Configuration'
