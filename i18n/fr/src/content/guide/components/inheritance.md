<!-- source: 233db69a47d8a1d52e07ff12ecce2b902999f050 -->
# Héritage

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Commencez par celui-ci si vous découvrez Angular.

Les composants Angular sont des classes TypeScript et suivent la sémantique standard de l'héritage
en JavaScript.

Un composant peut étendre n'importe quelle classe de base :

```ts
export class ListboxBase {
  value: string;
}

@Component(/* ... */)
export class CustomListbox extends ListboxBase {
  // CustomListbox inherits the `value` property.
}
```

## Étendre d'autres composants et directives {#extending-other-components-and-directives}

Quand un composant étend un autre composant ou une directive, il hérite d'une partie des métadonnées définies dans
le décorateur de la classe de base ainsi que des membres décorés de la classe de base. Cela comprend
les liaisons d'hôte, les inputs, les outputs et les méthodes de cycle de vie.

```angular-ts
@Component({
  selector: 'base-listbox',
  template: ` ... `,
  host: {
    '(keydown)': 'handleKey($event)',
  },
})
export class ListboxBase {
  value = input.required<string>();
  handleKey(event: KeyboardEvent) {
    /* ... */
  }
}

@Component({
  selector: 'custom-listbox',
  template: ` ... `,
  host: {
    '(click)': 'focusActiveOption()',
  },
})
export class CustomListbox extends ListboxBase {
  disabled = input(false);
  focusActiveOption() {
    /* ... */
  }
}
```

Dans l'exemple ci-dessus, `CustomListbox` hérite de toutes les informations associées à `ListboxBase`,
en remplaçant le sélecteur et le template par ses propres valeurs. `CustomListbox` possède deux inputs (`value`
et `disabled`) et deux écouteurs d'événements (`keydown` et `click`).

Les classes enfants obtiennent au final l'_union_ des inputs, des outputs et des liaisons d'hôte de tous leurs ancêtres
et des leurs.

### Transmettre les dépendances injectées {#forwarding-injected-dependencies}

Quand une classe de base utilise `inject()` comme initialiseur de propriété, la classe enfant hérite automatiquement de la propriété. Aucune transmission via `super` n'est nécessaire.

```ts
@Component(/* ... */)
export class ListboxBase {
  protected element = inject(ElementRef);
}

@Component(/* ... */)
export class CustomListbox extends ListboxBase {
  // `element` is inherited from `ListboxBase`.
}
```

Si une classe de base injecte des dépendances en tant que paramètres du constructeur, la classe enfant doit explicitement transmettre ces dépendances à `super`.

```ts
@Component(/* ... */)
export class ListboxBase {
  constructor(private element: ElementRef) {}
}

@Component(/* ... */)
export class CustomListbox extends ListboxBase {
  constructor(element: ElementRef) {
    super(element);
  }
}
```

### Redéfinir les méthodes de cycle de vie {#overriding-lifecycle-methods}

Si une classe de base définit une méthode de cycle de vie, comme `ngOnInit`, une classe enfant qui
implémente aussi `ngOnInit` _redéfinit_ l'implémentation de la classe de base. Si vous voulez conserver la méthode de cycle de vie
de la classe de base, appelez explicitement la méthode avec `super` :

```ts
@Component(/* ... */)
export class ListboxBase {
  protected isInitialized = false;
  ngOnInit() {
    this.isInitialized = true;
  }
}

@Component(/* ... */)
export class CustomListbox extends ListboxBase {
  override ngOnInit() {
    super.ngOnInit();
    /* ... */
  }
}
```
