<!-- source: dc1ff607590a9f43d2f8f21e26a296df6e8298fc -->
# Importer les variantes globales des données de locale

L'[Angular CLI][CliMain] inclut automatiquement les données de locale si vous exécutez la commande [`ng build`][CliBuild] avec l'option `--localize`.

```shell
ng build --localize
```

HELPFUL: L'installation initiale d'Angular contient déjà les données de locale pour l'anglais des États-Unis \(`en-US`\).
L'[Angular CLI][CliMain] inclut automatiquement les données de locale et définit la valeur de `LOCALE_ID` lorsque vous utilisez l'option `--localize` avec la commande [`ng build`][CliBuild].

Le package `@angular/common` sur npm contient les fichiers de données de locale.
Les variantes globales des données de locale sont disponibles dans `@angular/common/locales/global`.

## Exemple d'`import` pour le français {#import-example-for-french}

Par exemple, vous pouvez importer les variantes globales pour le français \(`fr`\) dans `main.ts`, là où vous faites le bootstrap de l'application.

<docs-code header="src/main.ts (import de la locale)" path="adev/src/content/examples/i18n/src/main.ts" region="global-locale"/>

HELPFUL: Dans une application à base de `NgModules`, vous l'importeriez dans votre `app.module`.

[CliMain]: cli 'CLI Overview and Command Reference | Angular'
[CliBuild]: cli/build 'ng build | CLI | Angular'
