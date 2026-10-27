<!-- source: 47b132a262676fbfe9c21991d301feea291beda2 -->
# API de composition de directives

Les directives Angular offrent un excellent moyen d'encapsuler des comportements réutilisables. Les directives peuvent appliquer des attributs, des classes CSS et des event listeners à un élément.

L'_API de composition de directives_ vous permet d'appliquer des directives au host element d'un composant _depuis_ la classe TypeScript du composant.

## Ajouter des directives à un composant {#adding-directives-to-a-component}

Vous appliquez des directives à un composant en ajoutant une propriété `hostDirectives` au décorateur du composant. Ces directives sont appelées _host directives_.

Dans cet exemple, la directive `MenuBehavior` est appliquée au host element de `AdminMenu`. Cela fonctionne de la même manière que l'application de `MenuBehavior` à l'élément `<admin-menu>` dans un template.

```ts
@Component({
  selector: 'admin-menu',
  templateUrl: './admin-menu.html',
  hostDirectives: [MenuBehavior],
})
export class AdminMenu {}
```

Quand le framework rend un composant, Angular crée aussi une instance de chaque host directive. Les host bindings des directives s'appliquent au host element du composant. Par défaut, les inputs et les outputs des host directives ne sont pas exposés dans l'API publique du composant. Consultez [Inclure des inputs et des outputs](#including-inputs-and-outputs) ci-dessous pour plus d'informations.

Les host directives s'accompagnent des contraintes suivantes :

- **Angular applique les host directives de manière statique, à la compilation.** Vous ne pouvez pas ajouter
  dynamiquement des directives à l'exécution.
- **Les directives utilisées dans `hostDirectives` ne peuvent pas spécifier `standalone: false`.**
- **Angular ignore le `selector` des directives appliquées dans la propriété `hostDirectives`.**

## Inclure des inputs et des outputs {#including-inputs-and-outputs}

Quand vous appliquez `hostDirectives` à votre composant, les inputs et les outputs des host directives ne sont pas inclus par défaut dans l'API de votre composant. Vous pouvez inclure explicitement des inputs et des outputs dans l'API de votre composant en développant l'entrée dans `hostDirectives` :

```ts
@Component({
  selector: 'admin-menu',
  templateUrl: './admin-menu.html',
  hostDirectives: [
    {
      directive: MenuBehavior,
      inputs: ['menuId'],
      outputs: ['menuClosed'],
    },
  ],
})
export class AdminMenu {}
```

En spécifiant explicitement les inputs et les outputs, les consommateurs du composant qui utilise `hostDirectives` peuvent les lier dans un template :

```angular-html
<admin-menu menuId="top-menu" (menuClosed)="logMenuClosed()"></admin-menu>
```

De plus, vous pouvez définir des alias pour les inputs et les outputs d'une host directive afin de personnaliser l'API de votre composant :

```ts
@Component({
  selector: 'admin-menu',
  templateUrl: './admin-menu.html',
  hostDirectives: [
    {
      directive: MenuBehavior,
      inputs: ['menuId: id'],
      outputs: ['menuClosed: closed'],
    },
  ],
})
export class AdminMenu {}
```

```angular-html
<admin-menu id="top-menu" (closed)="logMenuClosed()"></admin-menu>
```

## Ajouter des directives à une autre directive {#adding-directives-to-another-directive}

Vous pouvez aussi ajouter `hostDirectives` à d'autres directives, en plus des composants. Cela permet l'agrégation transitive de plusieurs comportements.

L'exemple suivant définit deux directives, `Menu` et `Tooltip`, puis compose leur comportement dans `MenuWithTooltip`. Enfin, il applique `MenuWithTooltip` à `SpecializedMenuWithTooltip`.

Quand `SpecializedMenuWithTooltip` est utilisée dans un template, elle crée des instances de `Menu`, `Tooltip` et `MenuWithTooltip`. Les host bindings de chacune de ces directives s'appliquent au host element de `SpecializedMenuWithTooltip`.

```ts
@Directive({/* ... */})
export class Menu {}

@Directive({/* ... */})
export class Tooltip {}

// MenuWithTooltip can compose behaviors from multiple other directives
@Directive({
  hostDirectives: [Tooltip, Menu],
})
export class MenuWithTooltip {}

// CustomWidget can apply the already-composed behaviors from MenuWithTooltip
@Directive({
  hostDirectives: [MenuWithTooltip],
})
export class SpecializedMenuWithTooltip {}
```

## Sémantique des host directives {#host-directive-semantics}

### Ordre d'exécution des directives {#directive-execution-order}

Les host directives suivent le même cycle de vie que les composants et les directives utilisés directement dans un template. Cependant, les host directives exécutent toujours leur constructeur, leurs lifecycle hooks et leurs bindings _avant_ le composant ou la directive auquel elles sont appliquées.

L'exemple suivant montre une utilisation minimale d'une host directive :

```ts
@Component({
  selector: 'admin-menu',
  templateUrl: './admin-menu.html',
  hostDirectives: [MenuBehavior],
})
export class AdminMenu {}
```

L'ordre d'exécution est ici le suivant :

1. `MenuBehavior` est instanciée
2. `AdminMenu` est instancié
3. `MenuBehavior` reçoit ses inputs (`ngOnInit`)
4. `AdminMenu` reçoit ses inputs (`ngOnInit`)
5. `MenuBehavior` applique ses host bindings
6. `AdminMenu` applique ses host bindings

Cet ordre des opérations signifie que les composants qui utilisent `hostDirectives` peuvent surcharger n'importe quel host binding spécifié par une host directive.

Cet ordre des opérations s'étend aux chaînes imbriquées de host directives, comme le montre l'exemple suivant.

```ts
@Directive({...})
export class Tooltip { }

@Directive({
  hostDirectives: [Tooltip],
})
export class CustomTooltip { }

@Directive({
  hostDirectives: [CustomTooltip],
})
export class EvenMoreCustomTooltip { }
```

Dans l'exemple ci-dessus, l'ordre d'exécution est le suivant :

1. `Tooltip` est instanciée
2. `CustomTooltip` est instanciée
3. `EvenMoreCustomTooltip` est instanciée
4. `Tooltip` reçoit ses inputs (`ngOnInit`)
5. `CustomTooltip` reçoit ses inputs (`ngOnInit`)
6. `EvenMoreCustomTooltip` reçoit ses inputs (`ngOnInit`)
7. `Tooltip` applique ses host bindings
8. `CustomTooltip` applique ses host bindings
9. `EvenMoreCustomTooltip` applique ses host bindings

### Injection de dépendances {#dependency-injection}

Un composant ou une directive qui spécifie `hostDirectives` peut injecter les instances de ces host directives, et inversement.

Quand vous appliquez des host directives à un composant, le composant et les host directives peuvent tous définir des providers.

Si un composant ou une directive utilisant `hostDirectives` et ses host directives fournissent le même jeton d'injection, les providers définis par la classe qui utilise `hostDirectives` ont la priorité sur ceux définis par les host directives.

### Dédoublonnage des host directives {#host-directive-de-duplication}

Quand la même directive apparaît plusieurs fois dans l'arbre résolu des host directives, elle est automatiquement dédoublonnée au lieu de lever une erreur. Deux règles déterministes décident quelle correspondance est conservée.

#### La correspondance dans le template a la priorité {#template-match-takes-precedence}

Si une directive correspond à un élément une fois via un **selector de template** et apparaît aussi comme **host directive**, Angular ne conserve que la correspondance dans le template et écarte toutes les correspondances en tant que host directive.

Le modèle mental est le suivant : une correspondance en tant que host directive représente `Partial<YourDirective>`, une application partielle où seuls les inputs et les outputs explicitement listés dans `hostDirectives` sont exposés, tandis qu'une correspondance dans le template représente la directive complète avec toute son API publique.

@Directive({selector: '[hoverable]'})
export class Hoverable {}

```ts
@Component({
  selector: 'app-button',
  hostDirectives: [Hoverable],
})
export class Button {}
```

```angular-html
<!-- Hoverable is matched by selector AND as a host directive of Button. -->
<!-- Angular keeps only the selector match, which has the full public API. -->
<app-button hoverable></app-button>
```

#### Les correspondances multiples en tant que host directive sont fusionnées {#multiple-host-directive-matches-are-merged}

Si la même directive apparaît **plusieurs fois en tant que host directive**, par exemple quand deux directives déclarent une dépendance commune dans leurs `hostDirectives`, Angular fusionne toutes les instances en une seule instance de directive. Les mappings d'inputs et d'outputs de toutes les instances sont combinés.

Cela résout le classique [problème du diamant](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem) dans la composition de host directives :

```ts
// A shared behavior that both triggers need
@Directive({
  host: {
    '[attr.data-trigger-id]': 'triggerId()',
  },
})
export class TriggerRef {
  readonly triggerId = input(`trigger-${crypto.randomUUID()}`);
}

// Two separate triggers, each declaring TriggerRef as a host directive
@Directive({
  selector: '[popoverTrigger]',
  hostDirectives: [TriggerRef],
})
export class PopoverTrigger {
  readonly triggerRef = inject(TriggerRef);
}

@Directive({
  selector: '[dropdownTrigger]',
  hostDirectives: [TriggerRef],
})
export class DropdownTrigger {
  readonly triggerRef = inject(TriggerRef);
}
```

```angular-html
<!-- Angular keeps one TriggerRef instance, shared by both triggers. -->
<button popoverTrigger dropdownTrigger>Actions</button>
```

HELPFUL: Comme Angular ne produit qu'une seule instance de la directive partagée, `PopoverTrigger` et `DropdownTrigger` reçoivent tous deux la même instance de `TriggerRef` quand ils l'injectent.

#### Alias en conflit {#conflicting-aliases}

Quand Angular fusionne des correspondances de host directive en double, il fusionne aussi leurs mappings d'inputs et d'outputs.

Si deux instances de la même host directive exposent le **même input ou output sous des alias différents**, Angular lève une erreur à la compilation ([NG8024](errors/NG8024)).

```ts
@Directive({
  selector: '[popoverTrigger]',
  hostDirectives: [{directive: TriggerRef, inputs: ['triggerId: popoverTriggerId']}],
})
export class PopoverTrigger {}

@Directive({
  selector: '[dropdownTrigger]',
  hostDirectives: [
    {directive: TriggerRef, inputs: ['triggerId: dropdownTriggerId']}, // different alias!
  ],
})
export class DropdownTrigger {}
```

```angular-html
<!-- Error: triggerId is exposed as both "popoverTriggerId" and "dropdownTriggerId". -->
<button popoverTrigger dropdownTrigger></button>
```

Pour résoudre ce problème, assurez-vous que les deux chemins exposent l'input ou l'output partagé sous le même alias, ou ne l'exposent pas du tout.

## Et ensuite ? {#whats-next}

<docs-pill-row>
  <docs-pill href="guide/directives/structural-directives" title="Directives structurelles"/>
  <docs-pill href="guide/components/host-elements" title="Host elements des composants"/>
</docs-pill-row>
