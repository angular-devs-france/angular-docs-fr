<!-- source: 52db021a0b90fa4f35daa5f4b9001a7262328208 -->
# Options du compilateur Angular

Lorsque vous utilisez la [compilation ahead-of-time (AOT)](tools/cli/aot-compiler), vous pouvez contrôler la façon dont votre application est compilée en spécifiant des options du compilateur Angular dans le [fichier de configuration TypeScript](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html).

L'objet d'options Angular, `angularCompilerOptions`, se place au même niveau que l'objet `compilerOptions`, qui fournit les options standard au compilateur TypeScript.

<docs-code header="tsconfig.json" path="adev/src/content/examples/angular-compiler-options/tsconfig.json" region="angular-compiler-options"/>

## Héritage de configuration avec `extends` {#configuration-inheritance-with-extends}

Comme le compilateur TypeScript, le compilateur AOT d'Angular prend aussi en charge `extends` dans la section `angularCompilerOptions` du fichier de configuration TypeScript.
La propriété `extends` se trouve au niveau racine, au même niveau que `compilerOptions` et `angularCompilerOptions`.

Une configuration TypeScript peut hériter des paramètres d'un autre fichier grâce à la propriété `extends`.
Les options de configuration du fichier de base sont chargées en premier, puis remplacées par celles du fichier de configuration qui en hérite.

Par exemple :

<docs-code header="tsconfig.app.json" path="adev/src/content/examples/angular-compiler-options/tsconfig.app.json" region="angular-compiler-options-app"/>

Pour plus d'informations, consultez le [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html).

## Options de template {#template-options}

Les options suivantes permettent de configurer le compilateur AOT d'Angular.

### `annotationsAs`

Modifie la façon dont les annotations propres à Angular sont émises afin d'améliorer le tree-shaking.
Les annotations non Angular ne sont pas affectées.
Valeur parmi `static fields` ou `decorators`. La valeur par défaut est `static fields`.

- Par défaut, le compilateur remplace les décorateurs par un champ statique dans la classe, ce qui permet à des outils de tree-shaking avancés comme [Closure compiler](https://github.com/google/closure-compiler) de supprimer les classes inutilisées
- La valeur `decorators` laisse les décorateurs en place, ce qui accélère la compilation.
  TypeScript émet des appels au helper `__decorate`.
  Utilisez `--emitDecoratorMetadata` pour la réflexion à l'exécution.

  HELPFUL: Notez que le code produit ne se prête pas correctement au tree-shaking.

### `annotateForClosureCompiler`

<!-- vale Angular.Angular_Spelling = NO -->

Lorsque cette option vaut `true`, utilise [Tsickle](https://github.com/angular/tsickle) pour annoter le JavaScript émis avec les commentaires [JSDoc](https://jsdoc.app) requis par le [Closure Compiler](https://github.com/google/closure-compiler).
La valeur par défaut est `false`.

<!-- vale Angular.Angular_Spelling = YES -->

### `compilationMode`

Indique le mode de compilation à utiliser.
Les modes suivants sont disponibles :

| Modes | Détails |
| :---------- | :-------------------------------------------------------------------------------------------------- |
| `'full'` | Génère du code entièrement compilé en AOT selon la version d'Angular actuellement utilisée. |
| `'partial'` | Génère du code sous une forme stable mais intermédiaire, adaptée à une library publiée. |

La valeur par défaut est `'full'`.

Pour la plupart des applications, `'full'` est le bon mode de compilation.

Utilisez `'partial'` pour les libraries publiées indépendamment, comme les packages npm.
Les compilations `'partial'` produisent un format intermédiaire stable, mieux adapté à une utilisation par des applications compilées avec des versions d'Angular différentes de celle de la library.
Les libraries compilées à « HEAD » avec leurs applications et utilisant la même version d'Angular, comme dans un monodépôt, peuvent utiliser `'full'` puisqu'il n'y a aucun risque de décalage de versions.

### `disableTypeScriptVersionCheck`

Lorsque cette option vaut `true`, le compilateur ne vérifie pas la version de TypeScript et ne signale pas d'erreur lorsqu'une version non prise en charge de TypeScript est utilisée.
Déconseillé, car les versions non prises en charge de TypeScript peuvent avoir un comportement indéfini.
La valeur par défaut est `false`.

### `enableI18nLegacyMessageIdFormat`

Demande au compilateur de templates d'Angular de créer des identifiants hérités pour les messages marqués dans les templates par l'attribut `i18n`.
Consultez [Marquer du texte pour la traduction][GuideI18nCommonPrepareMarkTextInComponentTemplate] pour plus d'informations sur le marquage des messages pour la localisation.

Définissez cette option à `false`, sauf si votre projet repose sur des traductions créées auparavant avec des identifiants hérités.
La valeur par défaut est `true`.

Les outils d'extraction de messages antérieurs à Ivy créaient divers formats hérités pour les identifiants des messages extraits.
Ces formats de messages présentent certains problèmes, comme la gestion des espaces et la dépendance aux informations contenues dans le HTML d'origine d'un template.

Le nouveau format de message résiste mieux aux changements d'espaces, il est identique pour tous les formats de fichiers de traduction et peut être créé directement à partir d'appels à `$localize`.
Cela permet aux messages `$localize` du code de l'application d'utiliser le même identifiant que les messages `i18n` identiques des templates de composants.

IMPORTANT: Cette option n'est prise en charge que par le builder `@angular-devkit/build-angular:browser`.
Avec le builder `@angular/build:application` (esbuild), cette option n'a aucun effet et le nouveau format décimal d'identifiant de message est toujours utilisé, quel que soit ce paramètre.

### `enableResourceInlining`

Lorsque cette option vaut `true`, remplace les propriétés `templateUrl` et `styleUrls` de tous les décorateurs `@Component` par du contenu inline dans les propriétés `template` et `styles`.

Lorsqu'elle est activée, la sortie `.js` de `ngc` n'inclut aucune URL de template ou de style chargée en lazy loading.

Pour les projets de library créés avec l'Angular CLI, la valeur par défaut de la configuration de développement est `true`.

### `flatModuleId`

L'identifiant de module à utiliser pour importer un module plat \(lorsque `flatModuleOutFile` vaut `true`\).
Les références créées par le compilateur de templates utilisent ce nom de module pour importer des symboles depuis le module plat.
Ignorée si `flatModuleOutFile` vaut `false`.

### `flatModuleOutFile`

Lorsque cette option vaut `true`, génère un index de module plat portant le nom de fichier indiqué, ainsi que les métadonnées de module plat correspondantes.
Utilisez-la pour créer des modules plats packagés de la même façon que `@angular/core` et `@angular/common`.
Lorsque cette option est utilisée, le `package.json` de la library doit référencer l'index de module plat créé plutôt que le fichier d'index de la library.

Produit un seul fichier `.metadata.json`, qui contient toutes les métadonnées nécessaires aux symboles exportés depuis l'index de la library.
Dans les fichiers `.ngfactory.js` créés, l'index de module plat est utilisé pour importer les symboles. Ces symboles incluent à la fois l'API publique de l'index de la library et des symboles internes masqués.

Par défaut, le fichier `.ts` fourni dans le champ `files` est considéré comme l'index de la library.
Si plusieurs fichiers `.ts` sont spécifiés, `libraryIndex` sert à sélectionner le fichier à utiliser.
Si plusieurs fichiers `.ts` sont fournis sans `libraryIndex`, une erreur est produite.

Un index de module plat `.d.ts` et `.js` est créé avec le nom `flatModuleOutFile` indiqué, au même emplacement que le fichier `.d.ts` d'index de la library.

Par exemple, si une library utilise le fichier `public_api.ts` comme index de library du module, le champ `files` du `tsconfig.json` vaudrait `["public_api.ts"]`.
L'option `flatModuleOutFile` pourrait alors être définie, par exemple, à `"index.js"`, ce qui produit les fichiers `index.d.ts` et `index.metadata.json`.
Le champ `module` du `package.json` de la library vaudrait `"index.js"` et le champ `typings` vaudrait `"index.d.ts"`.

### `generateCodeForLibraries`

Lorsque cette option vaut `true`, crée des fichiers de fabrique \(`.ngfactory.js` et `.ngstyle.js`\) pour les fichiers `.d.ts` accompagnés d'un fichier `.metadata.json` correspondant. La valeur par défaut est `true`.

Lorsqu'elle vaut `false`, les fichiers de fabrique ne sont créés que pour les fichiers `.ts`.
Faites-le lorsque vous utilisez des résumés de fabrique.

### `preserveWhitespaces`

Lorsque cette option vaut `false` (valeur par défaut), supprime les nœuds de texte vides des templates compilés, ce qui produit des modules de fabrique de templates plus légers.
Définissez-la à `true` pour conserver les nœuds de texte vides.

HELPFUL: Lorsque vous utilisez l'hydratation, il est recommandé d'utiliser `preserveWhitespaces: false`, qui est la valeur par défaut. Si vous choisissez de conserver les espaces en ajoutant `preserveWhitespaces: true` à votre tsconfig, vous risquez de rencontrer des problèmes d'hydratation. Cette configuration n'est pas encore entièrement prise en charge. Veillez aussi à ce que ce paramètre soit cohérent entre les fichiers tsconfig du serveur et du client. Consultez le [guide de l'hydratation](guide/hydration#preserve-whitespaces-configuration) pour plus de détails.

### `skipMetadataEmit`

Lorsque cette option vaut `true`, ne produit pas de fichiers `.metadata.json`.
La valeur par défaut est `false`.

Les fichiers `.metadata.json` contiennent les informations d'un fichier `.ts` dont le compilateur de templates a besoin et qui ne figurent pas dans le fichier `.d.ts` produit par le compilateur TypeScript.
Ces informations comprennent par exemple le contenu des annotations, comme le template d'un composant, que TypeScript émet dans le fichier `.js` mais pas dans le fichier `.d.ts`.

Vous pouvez la définir à `true` lorsque vous utilisez des résumés de fabrique, car ces résumés contiennent une copie des informations présentes dans le fichier `.metadata.json`.

Définissez-la à `true` si vous utilisez l'option `--outFile` de TypeScript, car les fichiers de métadonnées ne sont pas valides pour ce type de sortie TypeScript.
La communauté Angular déconseille d'utiliser `--outFile` avec Angular.
Utilisez plutôt un bundler, comme [webpack](https://webpack.js.org).

### `skipTemplateCodegen`

Lorsque cette option vaut `true`, n'émet pas de fichiers `.ngfactory.js` ni `.ngstyle.js`.
Cela désactive la majeure partie du compilateur de templates ainsi que le signalement des diagnostics de templates.

Peut servir à demander au compilateur de templates de produire des fichiers `.metadata.json` à distribuer avec un package `npm`. Cela évite de produire des fichiers `.ngfactory.js` et `.ngstyle.js` qui ne peuvent pas être distribués sur `npm`.

Pour les projets de library créés avec l'Angular CLI, la valeur par défaut de la configuration de développement est `true`.

### `strictMetadataEmit`

Lorsque cette option vaut `true`, signale une erreur dans le fichier `.metadata.json` si `"skipMetadataEmit"` vaut `false`.
La valeur par défaut est `false`.
À utiliser uniquement lorsque `"skipMetadataEmit"` vaut `false` et `"skipTemplateCodegen"` vaut `true`.

Cette option sert à vérifier les fichiers `.metadata.json` émis pour être bundlés avec un package `npm`.
La validation est stricte et peut émettre des erreurs pour des métadonnées qui ne produiraient jamais d'erreur lors de leur utilisation par le compilateur de templates.
Vous pouvez désactiver l'erreur émise par cette option pour un symbole exporté en ajoutant `@dynamic` dans le commentaire qui documente ce symbole.

Il est valide que des fichiers `.metadata.json` contiennent des erreurs.
Le compilateur de templates signale ces erreurs si les métadonnées servent à déterminer le contenu d'une annotation.
Le collecteur de métadonnées ne peut pas prédire quels symboles sont destinés à être utilisés dans une annotation. Il inclut donc par anticipation des nœuds d'erreur dans les métadonnées des symboles exportés.
Le compilateur de templates peut ensuite utiliser ces nœuds d'erreur pour signaler une erreur si ces symboles sont utilisés.

Si le client d'une library prévoit d'utiliser un symbole dans une annotation, le compilateur de templates ne le signale normalement pas. L'erreur n'est signalée qu'une fois que le client utilise réellement le symbole.
Cette option permet de détecter ces erreurs pendant la phase de build de la library ; elle est utilisée, par exemple, pour produire les libraries Angular elles-mêmes.

Pour les projets de library créés avec l'Angular CLI, la valeur par défaut de la configuration de développement est `true`.

### `strictInjectionParameters`

Lorsque cette option vaut `true`, signale une erreur pour tout paramètre fourni dont le type d'injection ne peut pas être déterminé.
Lorsqu'elle vaut `false`, les paramètres de constructeur des classes marquées avec `@Injectable` dont le type ne peut pas être résolu produisent un avertissement.
La valeur recommandée est `true`, mais la valeur par défaut est `false`.

Lorsque vous utilisez la commande `ng new --strict` de l'Angular CLI, elle est définie à `true` dans la configuration du projet créé.

### `strictTemplates`

Lorsque cette option vaut `true`, active la [vérification de types stricte des templates](tools/cli/template-typecheck#strict-mode).

Les flags de rigueur qu'active cette option vous permettent d'activer ou de désactiver des types précis de vérification de types stricte des templates.
Consultez [Résoudre les erreurs de templates](tools/cli/template-typecheck#troubleshooting-template-errors).
La valeur par défaut est `true`.

### `strictStandalone`

Lorsque cette option vaut `true`, signale une erreur si un composant, une directive ou un pipe n'est pas standalone.

### `trace`

Lorsque cette option vaut `true`, affiche des informations supplémentaires pendant la compilation des templates.
La valeur par défaut est `false`.

### `typeCheckHostBindings`

Lorsque cette option vaut `true`, active la vérification de types des expressions de l'objet littéral `host` et des décorateurs `@HostBinding`/`@HostListener` des composants et des directives.
La valeur par défaut est `true`.

## Options en ligne de commande {#command-line-options}

La plupart du temps, vous interagissez avec le compilateur Angular indirectement via l'[Angular CLI](reference/configs/angular-compiler-options). Pour déboguer certains problèmes, il peut être utile d'appeler directement le compilateur Angular.
Vous pouvez utiliser la commande `ngc` fournie par le package npm `@angular/compiler-cli` pour appeler le compilateur en ligne de commande.

La commande `ngc` est une surcouche de la commande `tsc` du compilateur TypeScript. Le compilateur Angular se configure principalement via `tsconfig.json`, tandis que l'Angular CLI se configure principalement via `angular.json`.

En plus du fichier de configuration, vous pouvez aussi utiliser les [options en ligne de commande de `tsc`](https://www.typescriptlang.org/docs/handbook/compiler-options.html) pour configurer `ngc`.

[GuideI18nCommonPrepareMarkTextInComponentTemplate]: guide/i18n/prepare#mark-text-in-component-template 'Mark text in component template - Prepare component for translation | Angular'
