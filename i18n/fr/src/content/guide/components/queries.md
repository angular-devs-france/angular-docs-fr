<!-- source: b0bc6e04a18dc947207bdcf1ae99a2c2ef337875 -->
# Référencer les enfants d'un composant avec des requêtes

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Lisez-le d'abord si vous découvrez Angular.

Un composant peut définir des **requêtes** qui trouvent des éléments enfants et lisent des valeurs depuis leurs injecteurs.

Les développeurs utilisent le plus souvent les requêtes pour récupérer des références vers des composants enfants, des directives, des éléments du DOM, et plus encore.

Toutes les fonctions de requête renvoient des signals qui reflètent les résultats les plus récents. Vous pouvez lire le
résultat en appelant la fonction du signal, y compris dans des [contextes réactifs](guide/signals#reactive-contexts) comme `computed` et `effect`.

Il existe deux catégories de requêtes : les **requêtes de vue** et les **requêtes de contenu.**

## Requêtes de vue {#view-queries}

Les requêtes de vue récupèrent des résultats parmi les éléments de la _vue_ du composant — les éléments définis dans le propre template du composant. Vous pouvez rechercher un résultat unique avec la fonction `viewChild`.

```angular-ts {highlight: [14, 15]}
@Component({
  selector: 'custom-card-header',
  /*...*/
})
export class CustomCardHeader {
  text: string;
}

@Component({
  selector: 'custom-card',
  template: '<custom-card-header>Visit sunny California!</custom-card-header>',
})
export class CustomCard {
  header = viewChild(CustomCardHeader);
  headerText = computed(() => this.header()?.text);
}
```

Dans cet exemple, le composant `CustomCard` recherche un enfant `CustomCardHeader` et utilise le résultat dans un `computed`.

Si la requête ne trouve aucun résultat, sa valeur est `undefined`. Cela peut se produire si l'élément ciblé est masqué par `@if`. Angular maintient le résultat de `viewChild` à jour à mesure que l'état de votre application change.

Vous pouvez aussi rechercher plusieurs résultats avec la fonction `viewChildren`.

```angular-ts {highlight: [17]}
@Component({
  selector: 'custom-card-action',
  /*...*/
})
export class CustomCardAction {
  text: string;
}

@Component({
  selector: 'custom-card',
  template: `
    <custom-card-action>Save</custom-card-action>
    <custom-card-action>Cancel</custom-card-action>
  `,
})
export class CustomCard {
  actions = viewChildren(CustomCardAction);
  actionsTexts = computed(() => this.actions().map((action) => action.text));
}
```

`viewChildren` crée un signal contenant un `Array` des résultats de la requête.

**Les requêtes ne traversent jamais les frontières des composants.** Les requêtes de vue ne peuvent récupérer des résultats que dans le template du composant.

## Requêtes de contenu {#content-queries}

Les requêtes de contenu récupèrent des résultats parmi les éléments du _contenu_ du composant — les éléments imbriqués dans le composant, dans le template où il est utilisé. Vous pouvez rechercher un résultat unique avec la fonction `contentChild`.

```angular-ts {highlight: [14, 15]}
@Component({
  selector: 'custom-toggle',
  /*...*/
})
export class CustomToggle {
  text: string;
}

@Component({
  selector: 'custom-expando',
  /* ... */
})
export class CustomExpando {
  toggle = contentChild(CustomToggle);
  toggleText = computed(() => this.toggle()?.text);
}

@Component({
  /* ... */
  // CustomToggle is used inside CustomExpando as content.
  template: `
    <custom-expando>
      <custom-toggle>Show</custom-toggle>
    </custom-expando>
  `,
})
export class UserProfile {}
```

Si la requête ne trouve aucun résultat, sa valeur est `undefined`. Cela peut se produire si l'élément ciblé est absent ou masqué par `@if`. Angular maintient le résultat de `contentChild` à jour à mesure que l'état de votre application change.

Par défaut, les requêtes `contentChild` parcourent les descendants, tandis que les requêtes `contentChildren` ne trouvent que les enfants _directs_. Consultez [Descendants du contenu](#content-descendants).

Vous pouvez aussi rechercher plusieurs résultats avec la fonction `contentChildren`.

```angular-ts {highlight: [14, 15]}
@Component({
  selector: 'custom-menu-item',
  /*...*/
})
export class CustomMenuItem {
  text: string;
}

@Component({
  selector: 'custom-menu',
  /*...*/
})
export class CustomMenu {
  items = contentChildren(CustomMenuItem);
  itemTexts = computed(() => this.items().map((item) => item.text));
}

@Component({
  selector: 'user-profile',
  template: `
    <custom-menu>
      <custom-menu-item>Cheese</custom-menu-item>
      <custom-menu-item>Tomato</custom-menu-item>
    </custom-menu>
  `,
})
export class UserProfile {}
```

`contentChildren` crée un signal contenant un `Array` des résultats de la requête.

**Les requêtes ne traversent jamais les frontières des composants.** Les requêtes de contenu ne peuvent récupérer des résultats que dans le même template que le composant lui-même.

## Requêtes obligatoires {#required-queries}

Si une requête d'enfant (`viewChild` ou `contentChild`) ne trouve aucun résultat, sa valeur est `undefined`. Cela peut se produire si l'élément ciblé est masqué par une instruction de flux de contrôle comme `@if` ou `@for`. C'est pourquoi les requêtes d'enfants renvoient un signal dont le type de valeur inclut `undefined`.

Dans certains cas, en particulier avec `viewChild`, vous savez avec certitude qu'un enfant donné est toujours disponible. Dans d'autres cas, vous pouvez vouloir imposer strictement la présence d'un enfant donné. Dans ces cas, vous pouvez utiliser une _requête obligatoire_.

```ts
@Component(/* ... */)
export class CustomCard {
  header = viewChild.required(CustomCardHeader);
  body = contentChild.required(CustomCardBody);
}
```

Si une requête obligatoire ne trouve aucun résultat correspondant, Angular signale une erreur. Comme cela garantit qu'un résultat est disponible, les requêtes obligatoires n'incluent pas automatiquement `undefined` dans le type de valeur du signal.

## Localisateurs de requête {#query-locators}

Le premier paramètre de chaque décorateur de requête est son **localisateur**.

La plupart du temps, vous utiliserez un composant ou une directive comme localisateur.

Vous pouvez aussi spécifier un localisateur sous forme de chaîne correspondant à
une [variable de référence de template](guide/templates/variables#template-reference-variables).

```angular-ts
@Component({
  /*...*/
  template: `
    <button #save>Save</button>
    <button #cancel>Cancel</button>
  `,
})
export class ActionBar {
  saveButton = viewChild<ElementRef<HTMLButtonElement>>('save');
}
```

Si plusieurs éléments définissent la même variable de référence de template, la requête récupère le premier élément correspondant.

Angular ne prend pas en charge les sélecteurs CSS comme localisateurs de requête.

### Les requêtes et l'arbre des injecteurs {#queries-and-the-injector-tree}

TIP: Consultez [Injection de dépendances](guide/di) pour en savoir plus sur les providers et l'arbre d'injection d'Angular.

Pour des cas plus avancés, vous pouvez utiliser n'importe quel `ProviderToken` comme localisateur. Cela vous permet de localiser des éléments en fonction des providers des composants et des directives.

```angular-ts
const SUB_ITEM = new InjectionToken<string>('sub-item');

@Component({
  /*...*/
  providers: [{provide: SUB_ITEM, useValue: 'special-item'}],
})
export class SpecialItem {}

@Component(/* ... */)
export class CustomList {
  subItemType = contentChild(SUB_ITEM);
}
```

L'exemple ci-dessus utilise un `InjectionToken` comme localisateur, mais vous pouvez utiliser n'importe quel `ProviderToken` pour localiser des éléments spécifiques.

## Options de requête {#query-options}

Toutes les fonctions de requête acceptent un objet d'options comme second paramètre. Ces options contrôlent la manière dont la requête trouve ses résultats.

### Lire des valeurs spécifiques depuis l'injecteur d'un élément {#reading-specific-values-from-an-elements-injector}

Par défaut, le localisateur de la requête indique à la fois l'élément recherché et la valeur récupérée. Vous pouvez aussi spécifier l'option `read` pour récupérer une autre valeur depuis l'élément trouvé par le localisateur.

```ts
@Component(/* ... */)
export class CustomExpando {
  toggle = contentChild(ExpandoContent, {read: TemplateRef});
}
```

L'exemple ci-dessus localise un élément portant la directive `ExpandoContent` et récupère
le `TemplateRef` associé à cet élément.

Les développeurs utilisent le plus souvent `read` pour récupérer `ElementRef` et `TemplateRef`.

Vous pouvez aussi passer `Injector` à `read`.

```angular-ts
@Component({
  selector: 'custom-table',
  template: `
    <third-party-table #inner>
      <ng-template>
        <ng-container [ngTemplateOutlet]="columns()" [ngTemplateOutletInjector]="innerInjector()" />
      </ng-template>
    </third-party-table>
  `,
})
export class CustomTable {
  columns = contentChild(TemplateRef);
  innerInjector = viewChild('inner', {read: Injector});
}
```

L'exemple ci-dessus récupère l'injecteur de nœud de l'élément `third-party-table`, c'est-à-dire
l'injecteur tel qu'il est vu depuis la position de cet élément dans l'arbre. Le transmettre à `NgTemplateOutlet` via
`ngTemplateOutletInjector` permet aux directives du template projeté d'injecter des valeurs que le
composant tiers fournit.

### Descendants du contenu {#content-descendants}

Par défaut, les requêtes `contentChildren` ne trouvent que les enfants _directs_ du composant et ne parcourent pas les descendants.
Les requêtes `contentChild` parcourent, elles, les descendants par défaut.

```angular-ts {highlight: [13, 14, 15, 16, 17]}
@Component({
  selector: 'custom-expando',
  /*...*/
})
export class CustomExpando {
  toggle = contentChildren(CustomToggle); // none found
  // toggle = contentChild(CustomToggle); // found
}

@Component({
  selector: 'user-profile',
  template: `
    <custom-expando>
      <some-other-component>
        <custom-toggle>Show</custom-toggle>
      </some-other-component>
    </custom-expando>
  `,
})
export class UserProfile {}
```

Dans l'exemple ci-dessus, `CustomExpando` ne peut pas trouver `<custom-toggle>` avec `contentChildren`, car ce n'est pas un enfant direct de `<custom-expando>`. En définissant `descendants: true`, vous configurez la requête pour qu'elle parcoure tous les descendants dans le même template. Les requêtes ne pénètrent toutefois _jamais_ dans les composants pour parcourir des éléments d'autres templates.

Les requêtes de vue n'ont pas cette option, car elles parcourent _toujours_ les descendants.

## Requêtes basées sur des décorateurs {#decorator-based-queries}

TIP: Bien que l'équipe Angular recommande d'utiliser les fonctions de requête basées sur les signals pour les nouveaux projets, les
API de requête d'origine basées sur des décorateurs restent entièrement prises en charge.

Vous pouvez aussi déclarer des requêtes en ajoutant le décorateur correspondant à une propriété. Les requêtes basées sur des décorateurs se comportent comme les requêtes basées sur les signals, sauf dans les cas décrits ci-dessous.

### Requêtes de vue {#decorator-view-queries}

Vous pouvez rechercher un résultat unique avec le décorateur `@ViewChild`.

```angular-ts {highlight: [14, 16, 17, 18]}
@Component({
  selector: 'custom-card-header',
  /*...*/
})
export class CustomCardHeader {
  text: string;
}

@Component({
  selector: 'custom-card',
  template: '<custom-card-header>Visit sunny California!</custom-card-header>',
})
export class CustomCard implements AfterViewInit {
  @ViewChild(CustomCardHeader) header: CustomCardHeader;

  ngAfterViewInit() {
    console.log(this.header.text);
  }
}
```

Dans cet exemple, le composant `CustomCard` recherche un enfant `CustomCardHeader` et accède au résultat dans `ngAfterViewInit`.

Angular maintient le résultat de `@ViewChild` à jour à mesure que l'état de votre application change.

**Les résultats des requêtes de vue deviennent disponibles dans la méthode de cycle de vie `ngAfterViewInit`**. Avant ce moment, la valeur est `undefined`. Consultez la section [Cycle de vie](guide/components/lifecycle) pour en savoir plus sur le cycle de vie des composants.

Vous pouvez aussi rechercher plusieurs résultats avec le décorateur `@ViewChildren`.

```angular-ts {highlight: [17, 19, 20, 21, 22, 23]}
@Component({
  selector: 'custom-card-action',
  /*...*/
})
export class CustomCardAction {
  text: string;
}

@Component({
  selector: 'custom-card',
  template: `
    <custom-card-action>Save</custom-card-action>
    <custom-card-action>Cancel</custom-card-action>
  `,
})
export class CustomCard implements AfterViewInit {
  @ViewChildren(CustomCardAction) actions: QueryList<CustomCardAction>;

  ngAfterViewInit() {
    this.actions.forEach((action) => {
      console.log(action.text);
    });
  }
}
```

`@ViewChildren` crée un objet `QueryList` qui contient les résultats de la requête. Vous pouvez vous abonner aux changements des résultats de la requête au fil du temps via la propriété `changes`.

### Requêtes de contenu {#decorator-content-queries}

Vous pouvez rechercher un résultat unique avec le décorateur `@ContentChild`.

```angular-ts {highlight: [14, 16, 17, 18]}
@Component({
  selector: 'custom-toggle',
  /*...*/
})
export class CustomToggle {
  text: string;
}

@Component({
  selector: 'custom-expando',
  /*...*/
})
export class CustomExpando implements AfterContentInit {
  @ContentChild(CustomToggle) toggle: CustomToggle;

  ngAfterContentInit() {
    console.log(this.toggle.text);
  }
}

@Component({
  selector: 'user-profile',
  template: `
    <custom-expando>
      <custom-toggle>Show</custom-toggle>
    </custom-expando>
  `,
})
export class UserProfile {}
```

Dans cet exemple, le composant `CustomExpando` recherche un enfant `CustomToggle` et accède au résultat dans `ngAfterContentInit`.

Angular maintient le résultat de `@ContentChild` à jour à mesure que l'état de votre application change.

**Les résultats des requêtes de contenu deviennent disponibles dans la méthode de cycle de vie `ngAfterContentInit`**. Avant ce moment, la valeur est `undefined`. Consultez la section [Cycle de vie](guide/components/lifecycle) pour en savoir plus sur le cycle de vie des composants.

Vous pouvez aussi rechercher plusieurs résultats avec le décorateur `@ContentChildren`.

```angular-ts {highlight: [14, 16, 17, 18, 19, 20]}
@Component({
  selector: 'custom-menu-item',
  /*...*/
})
export class CustomMenuItem {
  text: string;
}

@Component({
  selector: 'custom-menu',
  /*...*/
})
export class CustomMenu implements AfterContentInit {
  @ContentChildren(CustomMenuItem) items: QueryList<CustomMenuItem>;

  ngAfterContentInit() {
    this.items.forEach((item) => {
      console.log(item.text);
    });
  }
}

@Component({
  selector: 'user-profile',
  template: `
    <custom-menu>
      <custom-menu-item>Cheese</custom-menu-item>
      <custom-menu-item>Tomato</custom-menu-item>
    </custom-menu>
  `,
})
export class UserProfile {}
```

`@ContentChildren` crée un objet `QueryList` qui contient les résultats de la requête. Vous pouvez vous abonner aux changements des résultats de la requête au fil du temps via la propriété `changes`.

### Options des requêtes basées sur des décorateurs {#decorator-based-query-options}

Tous les décorateurs de requête acceptent un objet d'options comme second paramètre. Ces options fonctionnent de la même manière que pour les requêtes basées sur les signals, sauf dans les cas décrits ci-dessous.

### Requêtes statiques {#static-queries}

Les décorateurs `@ViewChild` et `@ContentChild` acceptent l'option `static`.

```angular-ts
@Component({
  selector: 'custom-card',
  template: '<custom-card-header>Visit sunny California!</custom-card-header>',
})
export class CustomCard implements OnInit {
  @ViewChild(CustomCardHeader, {static: true}) header: CustomCardHeader;

  ngOnInit() {
    console.log(this.header.text);
  }
}
```

En définissant `static: true`, vous garantissez à Angular que la cible de cette requête est _toujours_ présente et n'est pas rendue de façon conditionnelle. Le résultat devient ainsi disponible plus tôt, dans la méthode de cycle de vie `ngOnInit`.

Les résultats des requêtes statiques ne sont pas mis à jour après l'initialisation.

L'option `static` n'est pas disponible pour les requêtes `@ViewChildren` et `@ContentChildren`.

### Utiliser QueryList {#using-querylist}

`@ViewChildren` et `@ContentChildren` fournissent tous deux un objet `QueryList` qui contient une liste de résultats.

`QueryList` propose plusieurs API pratiques pour manipuler les résultats comme un tableau, telles que `map`, `reduce` et `forEach`. Vous pouvez obtenir un tableau des résultats actuels en appelant `toArray`.

Vous pouvez vous abonner à la propriété `changes` pour effectuer une action chaque fois que les résultats changent.

## Pièges courants avec les requêtes {#common-query-pitfalls}

Lorsque vous utilisez des requêtes, certains pièges courants peuvent rendre votre code plus difficile à comprendre et à maintenir.

Conservez toujours une source de vérité unique pour l'état partagé entre plusieurs composants. Cela évite les situations où un état dupliqué dans différents composants se désynchronise.

Évitez d'écrire directement un état dans des composants enfants. Ce pattern peut produire du code fragile, difficile à comprendre et sujet aux erreurs [ExpressionChangedAfterItHasBeenChecked](errors/NG0100).

N'écrivez jamais directement un état dans des composants parents ou ancêtres. Ce pattern peut produire du code fragile, difficile à comprendre et sujet aux erreurs [ExpressionChangedAfterItHasBeenChecked](errors/NG0100).
