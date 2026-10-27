<!-- source: aca4d2caa3ba0c5bb11e1c46ee83fc5641fbf15e -->
# Variables dans les templates

Angular propose deux types de déclarations de variables dans les templates : les variables de template locales et les variables de référence de template.

HELPFUL: Dans ce guide, « template » ne désigne pas l'ensemble du fichier de template HTML. Le terme désigne uniquement une construction ou une expression de template précise au sein du fichier.

## Variables de template locales avec `@let` {#local-template-variables-with-let}

La syntaxe `@let` d'Angular vous permet de définir une variable locale et de la réutiliser dans un template, de manière similaire à la [syntaxe `let` de JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let).

### Utiliser `@let` {#using-let}

Utilisez `@let` pour déclarer une variable dont la valeur repose sur le résultat d'une expression de template. Angular maintient automatiquement la valeur de la variable à jour avec l'expression donnée, de manière similaire aux [liaisons](/guide/templates/binding).

```angular-html
@let name = user.name;
@let greeting = 'Hello, ' + name;
@let data = data$ | async;
@let pi = 3.14159;
@let coordinates = {x: 50, y: 100};
@let longExpression =
  'Lorem ipsum dolor sit amet, consectetur adipiscing elit ' +
  'sed do eiusmod tempor incididunt ut labore et dolore magna ' +
  'Ut enim ad minim veniam...';
```

Chaque bloc `@let` peut déclarer exactement une variable. Vous ne pouvez pas déclarer plusieurs variables dans le même bloc en les séparant par une virgule.

### Référencer la valeur de `@let` {#referencing-the-value-of-let}

Une fois que vous avez déclaré une variable avec `@let`, vous pouvez la réutiliser dans le même template :

```angular-html
@let user = user$ | async;

@if (user) {
  <h1>Hello, {{ user.name }}</h1>
  <user-avatar [photo]="user.photo" />

  <ul>
    @for (snack of user.favoriteSnacks; track snack.id) {
      <li>{{ snack.name }}</li>
    }
  </ul>

  <button (click)="update(user)">Update profile</button>
}
```

### Réassignation {#assignability}

Une différence essentielle entre `@let` et le `let` de JavaScript est que `@let` ne peut pas être réassignée après sa déclaration. En revanche, Angular maintient automatiquement la valeur de la variable à jour avec l'expression donnée.

```angular-html
@let value = 1;

<!-- Invalid - This does not work! -->
<button (click)="value = value + 1">Increment the value</button>
```

### Portée des variables {#variable-scope}

Les déclarations `@let` ont pour portée la vue courante et ses descendants. Angular crée une nouvelle vue aux frontières des composants et partout où un template peut contenir du contenu dynamique, comme les blocs de flux de contrôle, les blocs `@defer` ou les directives structurelles.

Comme les déclarations `@let` ne sont pas remontées (hoisting), elles ne sont **pas** accessibles depuis les vues parentes ni depuis les éléments frères :

```angular-html
@let topLevel = value;

<div>
  @let insideDiv = value;
</div>

<!-- Valid -->
{{ topLevel }}
<!-- Valid -->
{{ insideDiv }}

@if (condition) {
  <!-- Valid -->
  {{ topLevel + insideDiv }}

  @let nested = value;

  @if (condition) {
    <!-- Valid -->
    {{ topLevel + insideDiv + nested }}
  }
}

<!-- Error, not hoisted from @if -->
{{ nested }}
```

### Syntaxe complète {#full-syntax}

La syntaxe `@let` est formellement définie ainsi :

- Le mot-clé `@let`.
- Suivi d'un ou plusieurs espaces, sans compter les retours à la ligne.
- Suivi d'un nom JavaScript valide et de zéro ou plusieurs espaces.
- Suivi du symbole = et de zéro ou plusieurs espaces.
- Suivi d'une expression Angular, qui peut s'étendre sur plusieurs lignes.
- Terminé par le symbole `;`.

## Variables de référence de template {#template-reference-variables}

Les variables de référence de template vous permettent de déclarer une variable qui référence une valeur provenant d'un élément de votre template.

Une variable de référence de template peut faire référence à :

- un élément du DOM dans un template (y compris les [éléments personnalisés](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements))
- un composant ou une directive Angular
- un [TemplateRef](/api/core/TemplateRef) issu d'un [ng-template](/api/core/ng-template)

Vous pouvez utiliser les variables de référence de template pour lire, dans une partie du template, des informations provenant d'une autre partie de ce même template.

### Déclarer une variable de référence de template {#declaring-a-template-reference-variable}

Vous pouvez déclarer une variable sur un élément d'un template en ajoutant un attribut qui commence par le caractère dièse (`#`) suivi du nom de la variable.

```angular-html
<!-- Create a template reference variable named "taskInput", referring to the HTMLInputElement. -->
<input #taskInput placeholder="Enter task name" />
```

### Assigner des valeurs aux variables de référence de template {#assigning-values-to-template-reference-variables}

Angular assigne une valeur aux variables de template en fonction de l'élément sur lequel la variable est déclarée.

Si vous déclarez la variable sur un composant Angular, la variable fait référence à l'instance du composant.

```angular-html
<!-- The `startDate` variable is assigned the instance of `MyDatepicker`. -->
<my-datepicker #startDate />
```

Si vous déclarez la variable sur un élément `<ng-template>`, la variable fait référence à une instance de TemplateRef qui représente le template. Pour plus d'informations, consultez [Comment Angular utilise la syntaxe avec astérisque, \*,](/guide/directives/structural-directives#structural-directive-shorthand) dans [Directives structurelles](/guide/directives/structural-directives).

```angular-html
<!-- The `myFragment` variable is assigned the `TemplateRef` instance corresponding to this template fragment. -->
<ng-template #myFragment>
  <p>This is a template fragment</p>
</ng-template>
```

Si vous déclarez la variable sur tout autre élément affiché, la variable fait référence à l'instance de `HTMLElement`.

```angular-html
<!-- The "taskInput" variable refers to the HTMLInputElement instance. -->
<input #taskInput placeholder="Enter task name" />
```

#### Assigner une référence à une directive Angular {#assigning-a-reference-to-an-angular-directive}

Les directives Angular peuvent avoir une propriété `exportAs` qui définit un nom par lequel la directive peut être référencée dans un template :

```angular-ts
@Directive({
  selector: '[dropZone]',
  exportAs: 'dropZone',
})
export class DropZone {
  /* ... */
}
```

Lorsque vous déclarez une variable de template sur un élément, vous pouvez assigner à cette variable une instance de directive en indiquant ce nom `exportAs` :

```angular-html
<!-- The `firstZone` variable refers to the `DropZone` directive instance. -->
<section dropZone #firstZone="dropZone">...</section>
```

Vous ne pouvez pas faire référence à une directive qui ne définit pas de nom `exportAs`.

### Utiliser les variables de référence de template avec les requêtes {#using-template-reference-variables-with-queries}

En plus d'utiliser les variables de template pour lire des valeurs depuis une autre partie du même template, vous pouvez aussi utiliser ce style de déclaration de variable pour « marquer » un élément pour les [requêtes de composant et de directive](/guide/components/queries).

Lorsque vous voulez interroger un élément précis d'un template, vous pouvez déclarer une variable de template sur cet élément, puis interroger l'élément à partir du nom de la variable.

```angular-html
<input #description value="Original description" />
```

```angular-ts
@Component({
  /* ... */,
  template: `<input #description value="Original description">`,
})
export class AppComponent {
  // Query for the input element based on the template variable name.
  @ViewChild('description') input: ElementRef | undefined;
}
```

Consultez [Référencer des enfants avec des requêtes](/guide/components/queries) pour plus d'informations sur les requêtes.

### Portée des variables de template {#template-variable-scope}

Tout comme les variables dans du code JavaScript ou TypeScript, les variables de template ont pour portée le template qui les déclare.

De même, les [blocs de flux de contrôle](guide/templates/control-flow) comme `@if` et `@for`, les [directives structurelles](guide/directives/structural-directives) et les déclarations `<ng-template>` créent une nouvelle portée de template imbriquée, tout comme les instructions de flux de contrôle de JavaScript telles que `if` et `for` créent de nouvelles portées lexicales. Vous ne pouvez pas accéder aux variables de template d'un de ces templates imbriqués depuis l'extérieur de ses limites.

HELPFUL: Ne définissez une variable qu'une seule fois dans le template afin que sa valeur à l'exécution reste prévisible.

#### Accès depuis un template imbriqué {#accessing-in-a-nested-template}

Un template interne peut accéder aux variables de template définies par le template externe.

Dans l'exemple suivant, modifier le texte de l'`<input>` modifie la valeur du `<span>`, car Angular répercute immédiatement les changements via la variable de template `ref1`.

```angular-html
<input #ref1 type="text" [(ngModel)]="firstExample" />

@if (true) {
  <span>Value: {{ ref1.value }}</span>
}
```

Dans ce cas, le bloc `@if` crée une nouvelle portée de template, qui inclut la variable `ref1` de sa portée parente.

En revanche, accéder dans le template parent à une variable de template d'une portée enfant ne fonctionne pas :

```angular-html {avoid}
@if (true) {
  <input #ref2 type="text" [(ngModel)]="secondExample" />
}

<span>Value: {{ ref2?.value }}</span>
```

Ici, `ref2` est déclarée dans la portée enfant créée par `@if` et n'est pas accessible depuis le template parent.
