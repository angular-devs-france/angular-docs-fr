<!-- source: 57111ac55055150ea280e095f41b838c03c74139 -->
# Host elements des composants

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Commencez par celui-ci si vous découvrez Angular.

Angular crée une instance d'un composant pour chaque élément HTML qui correspond au selector
du composant. L'élément du DOM qui correspond au selector d'un composant est le **host element** de ce composant.
Le contenu du template d'un composant est rendu à l'intérieur de son host element.

```angular-ts
// Component source
@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Your profile photo" />`,
})
export class ProfilePhoto {}
```

```angular-html
<!-- Using the component -->
<h3>Your profile photo</h3>
<profile-photo />
<button>Upload a new profile photo</button>
```

```angular-html
<!-- Rendered DOM -->
<h3>Your profile photo</h3>
<profile-photo>
  <img src="profile-photo.jpg" alt="Your profile photo" />
</profile-photo>
<button>Upload a new profile photo</button>
```

Dans l'exemple ci-dessus, `<profile-photo>` est le host element du composant `ProfilePhoto`.

## Lier des valeurs au host element {#binding-to-the-host-element}

Un composant peut lier des propriétés, des attributs, des styles et des événements à son host element. Cela se comporte
exactement comme les bindings sur les éléments à l'intérieur du template du composant, mais se définit avec
la propriété `host` du décorateur `@Component` :

```angular-ts
@Component({
  ...,
  host: {
    'role': 'slider',
    '[attr.aria-valuenow]': 'value',
    '[class.active]': 'isActive()',
    '[style.background]' : `hasError() ? 'red' : 'green'`,
    '[tabIndex]': 'disabled ? -1 : 0',
    '(keydown)': 'updateValue($event)',
  },
})
export class CustomSlider {
  value: number = 0;
  disabled: boolean = false;
  isActive = signal(false);
  hasError = signal(false);
  updateValue(event: KeyboardEvent) { /* ... */ }

  /* ... */
}
```

NOTE: Les noms de cibles globales utilisables pour préfixer un nom d'événement sont `document:`, `window:` et `body:`.

NOTE: Les noms de touches comme `'(keydown.enter)'` sont comparés à `KeyboardEvent.key`, qui dépend de
la disposition du clavier et de la langue de saisie de l'utilisateur. Pour cibler une touche physique quelle que soit la disposition, utilisez
plutôt le modificateur `code`, par exemple `'(keydown.code.Enter)'`. Consultez
[Utiliser les modificateurs de touche](guide/templates/event-listeners#using-key-modifiers) pour plus de détails.

## Les décorateurs `@HostBinding` et `@HostListener` {#the-hostbinding-and-hostlistener-decorators}

Vous pouvez aussi lier des valeurs au host element en appliquant les décorateurs `@HostBinding` et `@HostListener`
aux membres de la classe.

`@HostBinding` vous permet de lier des propriétés et des attributs du host à des propriétés et à des getters :

```ts
@Component({/* ... */})
export class CustomSlider {
  @HostBinding('attr.aria-valuenow')
  value: number = 0;

  @HostBinding('tabIndex')
  get tabIndex() {
    return this.disabled ? -1 : 0;
  }

  /* ... */
}
```

`@HostListener` vous permet de lier des event listeners au host element. Le décorateur accepte un nom
d'événement et un tableau facultatif d'arguments :

```ts
export class CustomSlider {
  @HostListener('keydown', ['$event'])
  updateValue(event: KeyboardEvent) {
    /* ... */
  }
}
```

<docs-callout critical title="Préférez la propriété `host` aux décorateurs">
  **Préférez toujours la propriété `host` à `@HostBinding` et `@HostListener`.** Ces
décorateurs existent uniquement pour la rétrocompatibilité.
</docs-callout>

## Conflits de bindings {#binding-collisions}

Quand vous utilisez un composant dans un template, vous pouvez appliquer des bindings à l'élément de ce composant. Le composant (ou les directives qui lui sont appliquées) peut aussi définir des host bindings pour les mêmes propriétés, attributs, classes ou styles.

La façon dont Angular gère ces conflits dépend du type de binding :

### Class bindings et style bindings {#class-and-style-bindings}

Les class bindings et les style bindings ont un ordre de priorité défini. Les bindings définis dans le template l'emportent sur les host bindings définis sur les directives ou les composants :

1. **Les bindings du template** (par exemple `[class.active]` ou `[style.color]`)
2. **Les valeurs statiques du template** (par exemple `class="..."` ou `style="..."` dans le template)
3. **Les host bindings des directives**
4. **Les host bindings des composants**
5. **Les valeurs statiques du host** (par exemple `host: {class: '...'}`)

Si la valeur d'un binding est `undefined`, Angular se rabat sur le niveau suivant. Une valeur `null` retire la classe ou le style sans se rabattre.

### Conflits de propriétés et d'attributs {#property-and-attribute-collisions}

Pour les propriétés et attributs qui ne concernent pas le style (comme `id`, `role` ou `[attr.aria-*]`) :

- **Attributs statiques :** si le template et le composant déclarent tous deux un attribut statique (par exemple `<my-cmp id="parent">` et `host: {id: 'child'}`), l'attribut défini dans le template l'emporte.
- **Statique et dynamique :** si une valeur est statique et l'autre dynamique, la valeur dynamique l'emporte.
- **Bindings dynamiques :** évitez de lier la même propriété ou le même attribut du DOM à la fois dans le template et via des host bindings. Angular ne réconcilie pas les property bindings en double ; des bindings dynamiques qui se chevauchent peuvent provoquer des écrasements inattendus selon l'ordre d'exécution de la détection de changements.

## Styliser avec les propriétés personnalisées CSS {#styling-with-css-custom-properties}

Les développeurs s'appuient souvent sur les [propriétés personnalisées CSS](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables/Using_CSS_custom_properties) pour permettre une configuration flexible des styles de leurs composants.
Vous pouvez définir ces propriétés personnalisées sur un host element avec un [style binding](guide/templates/binding#css-style-properties).

```angular-ts
@Component({
  /* ... */
  host: {
    '[style.--my-background]': 'color()',
  },
})
export class MyComponent {
  color = signal('lightgreen');
}
```

Dans cet exemple, la propriété personnalisée CSS `--my-background` est liée au signal `color`. La valeur de la propriété personnalisée se met automatiquement à jour chaque fois que le signal `color` change. Cela affecte le composant courant et tous ses enfants qui s'appuient sur cette propriété personnalisée.

### Définir des propriétés personnalisées sur les composants enfants {#setting-custom-properties-on-children-components}

Il est également possible de définir des propriétés personnalisées CSS sur le host element des composants enfants avec un [style binding](guide/templates/binding#css-style-properties).

```angular-ts
@Component({
  selector: 'my-component',
  template: `<my-child [style.--my-background]="color()" />`,
})
export class MyComponent {
  color = signal('lightgreen');
}
```

## Injecter les attributs du host element {#injecting-host-element-attributes}

Les composants et les directives peuvent lire les attributs statiques de leur host element en utilisant `HostAttributeToken` avec la fonction [`inject`](api/core/inject).

```ts
import { Component, HostAttributeToken, inject } from '@angular/core';

@Component({
  selector: 'app-button',
  ...,
})
export class Button {
  variation = inject(new HostAttributeToken('variation'));
}
```

```angular-html
<app-button variation="primary">Click me</app-button>
```

HELPFUL: `HostAttributeToken` lève une erreur si l'attribut est absent, sauf si l'injection est marquée comme facultative.
