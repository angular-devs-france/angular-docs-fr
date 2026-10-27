<!-- source: 5425e5b1963f6a84448f5b39f1cb4861539cbf80 -->
# Flux de contrôle

Les templates Angular prennent en charge des blocs de flux de contrôle qui vous permettent d'afficher, de masquer et de répéter des éléments de manière conditionnelle.

## Afficher du contenu de manière conditionnelle avec `@if`, `@else if` et `@else` {#conditionally-display-content-with-if-else-if-and-else}

Le bloc `@if` affiche son contenu de manière conditionnelle quand son expression de condition est truthy :

```angular-html
@if (a > b) {
  <p>{{ a }} is greater than {{ b }}</p>
}
```

Si vous souhaitez afficher un contenu alternatif, vous pouvez le faire en fournissant un nombre quelconque de blocs `@else if` et un unique bloc `@else`.

```angular-html
@if (a > b) {
  {{ a }} is greater than {{ b }}
} @else if (b > a) {
  {{ a }} is less than {{ b }}
} @else {
  {{ a }} is equal to {{ b }}
}
```

### Référencer le résultat de l'expression conditionnelle {#referencing-the-conditional-expressions-result}

La condition `@if` permet d'enregistrer le résultat de l'expression conditionnelle dans une variable pour la réutiliser à l'intérieur du bloc.

```angular-html
@if (user.profile.settings.startDate; as startDate) {
  {{ startDate }}
}
```

Cela peut être utile pour référencer des expressions plus longues, qui seraient plus faciles à lire et à maintenir dans le template.

## Répéter du contenu avec le bloc `@for` {#repeat-content-with-the-for-block}

Le bloc `@for` parcourt une collection et rend le contenu d'un bloc de manière répétée. La collection peut être n'importe quel [itérable](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Iteration_protocols) JavaScript, mais Angular dispose d'optimisations de performances supplémentaires pour les valeurs de type `Array`.

Une boucle `@for` typique ressemble à ceci :

```angular-html
@for (item of items; track item.id) {
  {{ item.name }}
}
```

Le bloc `@for` d'Angular ne prend pas en charge les instructions qui modifient le flux, comme `continue` ou `break` en JavaScript.

### Pourquoi `track` est-il important dans les blocs `@for` ? {#why-is-track-in-for-blocks-important}

L'expression `track` permet à Angular de maintenir une relation entre vos données et les nœuds DOM de la page. Angular peut ainsi optimiser les performances en exécutant le minimum d'opérations DOM nécessaires quand les données changent.

Bien utiliser track peut améliorer considérablement les performances de rendu de votre application quand vous bouclez sur des collections de données.

Dans l'expression `track`, choisissez une propriété qui identifie de manière unique chaque élément. Si votre modèle de données inclut une propriété d'identification unique, généralement `id` ou `uuid`, utilisez cette valeur. Si vos données n'incluent pas de champ de ce type, envisagez sérieusement d'en ajouter un.

Pour les collections statiques qui ne changent jamais, vous pouvez utiliser `$index` pour indiquer à Angular de suivre chaque élément par son index dans la collection.

Si aucune autre option n'est disponible, vous pouvez utiliser l'élément lui-même comme clé de suivi. Cela indique à Angular de suivre l'élément par l'identité de sa référence, avec l'opérateur d'égalité stricte (`===`). Évitez cette option autant que possible, car elle peut ralentir considérablement les mises à jour du rendu : Angular n'a aucun moyen de savoir quel élément de données correspond à quels nœuds DOM.

```angular-html
@for (item of items; track item) {
  {{ item.name }}
}
```

NOTE: Contrairement à `*ngFor`, le bloc `@for` privilégie la réutilisation des vues. Si une propriété suivie change mais que la référence de l'objet reste la même, Angular met à jour les liaisons de la vue (y compris les inputs des composants) au lieu de détruire et de recréer l'élément entier.

### Variables contextuelles dans les blocs `@for` {#contextual-variables-in-for-blocks}

À l'intérieur des blocs `@for`, plusieurs variables implicites sont toujours disponibles :

| Variable | Signification |
| -------- | --------------------------------------------- |
| `$count` | Nombre d'éléments de la collection parcourue |
| `$index` | Index de la ligne courante |
| `$first` | Indique si la ligne courante est la première |
| `$last`  | Indique si la ligne courante est la dernière |
| `$even`  | Indique si l'index de la ligne courante est pair |
| `$odd`   | Indique si l'index de la ligne courante est impair |

Ces variables sont toujours disponibles sous ces noms, mais vous pouvez leur donner un alias avec un segment `let` :

```angular-html
@for (item of items; track item.id; let idx = $index, e = $even) {
  <p>Item #{{ idx }}: {{ item.name }}</p>
}
```

Les alias sont utiles quand vous imbriquez des blocs `@for` : ils vous permettent de lire, depuis un bloc `@for` interne, les variables du bloc `@for` externe.

### Fournir un contenu de repli pour les blocs `@for` avec le bloc `@empty` {#providing-a-fallback-for-for-blocks-with-the-empty-block}

Vous pouvez inclure une section `@empty` facultative juste après le contenu du bloc `@for`. Le contenu du bloc `@empty` s'affiche quand il n'y a aucun élément :

```angular-html
@for (item of items; track item.name) {
  <li>{{ item.name }}</li>
} @empty {
  <li>There are no items.</li>
}
```

## Afficher du contenu de manière conditionnelle avec le bloc `@switch` {#conditionally-display-content-with-the-switch-block}

Si le bloc `@if` convient à la plupart des scénarios, le bloc `@switch` offre une syntaxe alternative pour rendre des données de manière conditionnelle. Sa syntaxe ressemble beaucoup à l'instruction `switch` de JavaScript.

```angular-html
@switch (userPermissions) {
  @case ('admin') {
    <app-admin-dashboard />
  }
  @case ('reviewer')
  @case ('editor') {
    <app-editor-dashboard />
  }
  @default {
    <app-viewer-dashboard />
  }
}
```

La valeur de l'expression conditionnelle est comparée à l'expression de chaque cas avec l'opérateur d'égalité stricte (`===`).

**`@switch` n'a pas de fallthrough** (passage au cas suivant) : vous n'avez donc pas besoin d'un équivalent à une instruction `break` ou `return` dans le bloc.

Vous pouvez spécifier plusieurs conditions pour un même bloc en enchaînant des instructions `@case` consécutives.

Vous pouvez inclure un bloc `@default` facultatif. Le contenu du bloc `@default` s'affiche si aucune des expressions de cas précédentes ne correspond à la valeur du switch.

Si aucun `@case` ne correspond à l'expression et qu'il n'y a pas de bloc `@default`, rien n'est affiché.

### Vérification de type exhaustive {#exhaustive-type-checking}

`@switch` prend en charge la vérification de type exhaustive, ce qui permet à Angular de vérifier à la compilation que toutes les valeurs possibles d'un type union sont traitées.

En utilisant `@default never;`, vous déclarez explicitement qu'il ne doit rester aucun cas. Si le type union est étendu par la suite et qu'un nouveau cas n'est couvert par aucun @case, le vérificateur de types des templates d'Angular signale une erreur, ce qui vous aide à repérer tôt les branches manquantes.

NOTE: La vérification d'exhaustivité repose sur le rétrécissement de type (type narrowing) de TypeScript, qui ne fonctionne que sur des variables. Elle ne fonctionne pas si la condition du switch est un appel de fonction ou un signal (par exemple, `@switch (state())`). Pour contourner ce problème, affectez le signal à une variable `@let`, par exemple : `@let mySignal = this.mySignal()`.

```angular-ts
@Component({
  template: `
    @switch (state) {
      @case ('loggedOut') {
        <button>Login</button>
      }

      @case ('loggedIn') {
        <p>Welcome back!</p>
      }

      @default never;
      <!-- throws because @case ('loading') is missing -->
    }
  `,
})
export class AppComponent {
  state: 'loggedOut' | 'loading' | 'loggedIn' = 'loggedOut';
}
```

Quand l'expression évaluée par le switch est imbriquée dans une union, vous devez spécifier explicitement l'expression dont l'exhaustivité doit être vérifiée.

<!-- prettier-ignore -->
```angular-ts
@Component({
  template: `
    @switch (state.mode) {
      @case ('show') {
        {{ state.menu }};
      }
      @case ('hide') {}
      @default never(state);
    }
  `,
})
export class App {
  state!: {mode: 'hide'} | {mode: 'show'; menu: number};
}
```
