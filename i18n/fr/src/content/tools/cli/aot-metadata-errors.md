<!-- source: fbe9007f840b1df393ac59320c82991ac2713fe5 -->
# Erreurs de métadonnées AOT

Voici les erreurs de métadonnées que vous pouvez rencontrer, avec des explications et des corrections suggérées.

## Forme d'expression non prise en charge {#expression-form-not-supported}

HELPFUL: Le compilateur a rencontré une expression qu'il n'a pas comprise lors de l'évaluation des métadonnées Angular.

Les fonctionnalités du langage en dehors de la [syntaxe d'expression restreinte](tools/cli/aot-compiler) du compilateur
peuvent produire cette erreur, comme le montre l'exemple suivant :

```ts
// ERROR
export class Fooish { … }
…
const prop = typeof Fooish; // typeof is not valid in metadata
  …
  // bracket notation is not valid in metadata
  { provide: 'token', useValue: { [prop]: 'value' } };
  …
```

Vous pouvez utiliser `typeof` et la notation entre crochets dans le code applicatif normal.
Vous ne pouvez simplement pas utiliser ces fonctionnalités dans les expressions qui définissent des métadonnées Angular.

Évitez cette erreur en vous en tenant à la [syntaxe d'expression restreinte](tools/cli/aot-compiler) du compilateur
lorsque vous écrivez des métadonnées Angular,
et méfiez-vous des fonctionnalités TypeScript nouvelles ou inhabituelles.

## Référence à un symbole local (non exporté) {#reference-to-a-local-non-exported-symbol}

HELPFUL: Reference to a local \(non-exported\) symbol 'symbol name'. Consider exporting the symbol.

Le compilateur a rencontré une référence à un symbole défini localement qui n'était soit pas exporté, soit pas initialisé.

Voici un exemple du problème avec un `provider`.

```ts

// ERROR
let foo: number; // neither exported nor initialized

@Component({
  selector: 'my-component',
  template: … ,
  providers: [
    { provide: Foo, useValue: foo }
  ]
})
export class MyComponent {}

```

Le compilateur génère la fabrique du composant, qui inclut le code du provider `useValue`, dans un module séparé. _Ce_ module de fabrique ne peut pas remonter jusqu'à _ce_ module source pour accéder à la variable locale \(non exportée\) `foo`.

Vous pouvez corriger le problème en initialisant `foo`.

```ts
let foo = 42; // initialized
```

Le compilateur va [replier](tools/cli/aot-compiler#code-folding) l'expression dans le provider comme si vous aviez écrit ceci.

```ts
providers: [{provide: Foo, useValue: 42}];
```

Vous pouvez aussi le corriger en exportant `foo`, en partant du principe que `foo` sera assigné à l'exécution, lorsque vous connaîtrez réellement sa valeur.

```ts
// CORRECTED
export let foo: number; // exported

@Component({
  selector: 'my-component',
  template: … ,
  providers: [
    { provide: Foo, useValue: foo }
  ]
})
export class MyComponent {}
```

Ajouter `export` fonctionne souvent pour les variables référencées dans des métadonnées comme `providers` et `animations`, car le compilateur peut générer des _références_ aux variables exportées dans ces expressions. Il n'a pas besoin des _valeurs_ de ces variables.

Ajouter `export` ne fonctionne pas lorsque le compilateur a besoin de la _valeur réelle_
pour générer du code.
Par exemple, cela ne fonctionne pas pour la propriété `template`.

```ts
// ERROR
export let someTemplate: string; // exported but not initialized

@Component({
  selector: 'my-component',
  template: someTemplate,
})
export class MyComponent {}
```

Le compilateur a besoin de la valeur de la propriété `template` _immédiatement_ pour générer la fabrique du composant.
La seule référence à la variable ne suffit pas.
Préfixer la déclaration par `export` ne fait que produire une nouvelle erreur, « [`Only initialized variables and constants can be referenced`](#only-initialized-variables-and-constants) ».

## Seules les variables et constantes initialisées {#only-initialized-variables-and-constants}

HELPFUL: _Only initialized variables and constants can be referenced because the value of this variable is needed by the template compiler._

Le compilateur a trouvé une référence à une variable exportée ou à un champ statique qui n'était pas initialisé.
Il a besoin de la valeur de cette variable pour générer du code.

L'exemple suivant tente de définir la propriété `template` du composant avec la valeur de la variable exportée `someTemplate`, qui est déclarée mais _non assignée_.

```ts
// ERROR
export let someTemplate: string;

@Component({
  selector: 'my-component',
  template: someTemplate,
})
export class MyComponent {}
```

Vous obtiendriez aussi cette erreur si vous importiez `someTemplate` depuis un autre module en oubliant de l'y initialiser.

```ts
import {Component} from '@angular/core';
// ERROR - not initialized there either
import {someTemplate} from './config';

@Component({
  selector: 'my-component',
  template: someTemplate,
})
export class MyComponent {}
```

Le compilateur ne peut pas attendre l'exécution pour obtenir les informations du template.
Il doit déduire statiquement la valeur de la variable `someTemplate` à partir du code source afin de pouvoir générer la fabrique du composant, qui contient les instructions pour construire l'élément à partir du template.

Pour corriger cette erreur, fournissez la valeur initiale de la variable dans une clause d'initialisation _sur la même ligne_.

```ts
// CORRECTED
export let someTemplate = '<h1>Greetings from Angular</h1>';

@Component({
  selector: 'my-component',
  template: someTemplate,
})
export class MyComponent {}
```

## Référence à une classe non exportée {#reference-to-a-non-exported-class}

HELPFUL: _Reference to a non-exported class `<class name>`._
_Consider exporting the class._

Des métadonnées ont référencé une classe qui n'était pas exportée.

Par exemple, vous avez peut-être défini une classe et l'avez utilisée comme jeton d'injection dans un tableau de providers, en oubliant d'exporter cette classe.

```ts
// ERROR
abstract class MyStrategy { }

  …
  providers: [
    { provide: MyStrategy, useValue: … }
  ]
  …
```

Angular génère une fabrique de classe dans un module séparé, et cette fabrique [ne peut accéder qu'aux classes exportées](tools/cli/aot-compiler#public-or-protected-symbols).
Pour corriger cette erreur, exportez la classe référencée.

```ts
// CORRECTED
export abstract class MyStrategy { }

  …
  providers: [
    { provide: MyStrategy, useValue: … }
  ]
  …
```

## Variable ou constante déstructurée non prise en charge {#destructured-variable-or-constant-not-supported}

HELPFUL: _Referencing an exported destructured variable or constant is not supported by the template compiler. Consider simplifying this to avoid destructuring._

Le compilateur ne prend pas en charge les références à des variables assignées par [déstructuration](https://www.typescriptlang.org/docs/handbook/variable-declarations.html#destructuring).

Par exemple, vous ne pouvez pas écrire quelque chose comme ceci :

```ts
// ERROR
import { configuration } from './configuration';

// destructured assignment to foo and bar
const {foo, bar} = configuration;
  …
  providers: [
    {provide: Foo, useValue: foo},
    {provide: Bar, useValue: bar},
  ]
  …
```

Pour corriger cette erreur, faites référence à des valeurs non déstructurées.

```ts
// CORRECTED
import { configuration } from './configuration';
  …
  providers: [
    {provide: Foo, useValue: configuration.foo},
    {provide: Bar, useValue: configuration.bar},
  ]
  …
```

## Impossible de résoudre le type {#could-not-resolve-type}

HELPFUL: _Le compilateur a rencontré un type et ne parvient pas à déterminer quel module exporte ce type._

Cela peut arriver si vous faites référence à un type ambiant.
Par exemple, le type `Window` est un type ambiant déclaré dans le fichier global `.d.ts`.

Vous obtiendrez une erreur si vous le référencez dans le constructeur du composant, que le compilateur doit analyser statiquement.

```ts
// ERROR
@Component({ })
export class MyComponent {
  constructor (private win: Window) { … }
}
```

TypeScript comprend les types ambiants, vous ne les importez donc pas.
Le compilateur Angular ne comprend pas un type que vous omettez d'exporter ou d'importer.

Dans ce cas, le compilateur ne sait pas comment injecter quelque chose avec le jeton `Window`.

Ne faites pas référence à des types ambiants dans les expressions de métadonnées.

Si vous devez injecter une instance d'un type ambiant,
vous pouvez contourner le problème en quatre étapes :

1. Créez un jeton d'injection pour une instance du type ambiant.
1. Créez une fonction de fabrique qui renvoie cette instance.
1. Ajoutez un provider `useFactory` avec cette fonction de fabrique.
1. Utilisez `@Inject` pour injecter l'instance.

Voici un exemple illustratif.

```ts
// CORRECTED
import {Inject, InjectionToken, Component} from '@angular/core';

export const WINDOW = new InjectionToken('Window');
export function _window() { return window; }

@Component({
  …
  providers: [
    { provide: WINDOW, useFactory: _window }
  ]
})
export class MyComponent {
  constructor (@Inject(WINDOW) private win: Window) { … }
}
```

Le type `Window` dans le constructeur ne pose plus de problème au compilateur, car celui-ci
utilise `@Inject(WINDOW)` pour générer le code d'injection.

Angular fait quelque chose de similaire avec le jeton `DOCUMENT`, ce qui vous permet d'injecter l'objet `document` du navigateur \(ou une abstraction de celui-ci, selon la plateforme sur laquelle l'application s'exécute\).

```ts
import {Inject, Component} from '@angular/core';
import { DOCUMENT } from '@angular/common';

@Component({ … })
export class MyComponent {
  constructor (@Inject(DOCUMENT) private doc: Document) { … }
}
```

## Nom attendu {#name-expected}

HELPFUL: _Le compilateur attendait un nom dans une expression qu'il évaluait._

Cela peut arriver si vous utilisez un nombre comme nom de propriété, comme dans l'exemple suivant.

```ts
// ERROR
provider: [{provide: Foo, useValue: {0: 'test'}}];
```

Changez le nom de la propriété pour une valeur non numérique.

```ts
// CORRECTED
provider: [{provide: Foo, useValue: {'0': 'test'}}];
```

## Nom de membre d'enum non pris en charge {#unsupported-enum-member-name}

HELPFUL: _Angular n'a pas pu déterminer la valeur du [membre d'enum](https://www.typescriptlang.org/docs/handbook/enums.html) que vous avez référencé dans les métadonnées._

Le compilateur comprend les valeurs d'enum simples, mais pas les valeurs complexes comme celles dérivées de propriétés calculées.

```ts
// ERROR
enum Colors {
  Red = 1,
  White,
  Blue = "Blue".length // computed
}

  …
  providers: [
    { provide: BaseColor,   useValue: Colors.White } // ok
    { provide: DangerColor, useValue: Colors.Red }   // ok
    { provide: StrongColor, useValue: Colors.Blue }  // bad
  ]
  …
```

Évitez de faire référence à des enums dotés d'initialiseurs complexes ou de propriétés calculées.

## Les expressions de template étiquetées ne sont pas prises en charge {#tagged-template-expressions-are-not-supported}

HELPFUL: _Tagged template expressions are not supported in metadata._

Le compilateur a rencontré une [expression de template étiquetée](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Template_literals#Tagged_template_literals) JavaScript ES2015, comme la suivante.

```ts

// ERROR
const expression = 'funky';
const raw = String.raw`A tagged template ${expression} string`;
 …
 template: '<div>' + raw + '</div>'
 …

```

[`String.raw()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String/raw) est une _fonction d'étiquetage_ native de JavaScript ES2015.

Le compilateur AOT ne prend pas en charge les expressions de template étiquetées ; évitez-les dans les expressions de métadonnées.

## Référence à un symbole attendue {#symbol-reference-expected}

HELPFUL: _Le compilateur attendait une référence à un symbole à l'emplacement indiqué dans le message d'erreur._

Cette erreur peut se produire si vous utilisez une expression dans la clause `extends` d'une classe.

<!--todo: Chuck: After reviewing your PR comment I'm still at a loss. See [comment there](https://github.com/angular/angular/pull/17712#discussion_r132025495). -->
