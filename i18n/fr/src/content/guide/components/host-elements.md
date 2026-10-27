<!-- source: 0493a09e3fa04b4a25d88c340105bd842dc192be -->
# Éléments hôtes des composants

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Commencez par celui-ci si vous découvrez Angular.

Angular crée une instance d'un composant pour chaque élément HTML qui correspond au sélecteur
du composant. L'élément du DOM qui correspond au sélecteur d'un composant est l'**élément hôte** de ce composant.
Le contenu du template d'un composant est rendu à l'intérieur de son élément hôte.

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

Dans l'exemple ci-dessus, `<profile-photo>` est l'élément hôte du composant `ProfilePhoto`.

## Lier des valeurs à l'élément hôte {#binding-to-the-host-element}

Un composant peut lier des propriétés, des attributs, des styles et des événements à son élément hôte. Cela se comporte
exactement comme les liaisons sur les éléments à l'intérieur du template du composant, mais se définit avec
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

Vous pouvez aussi lier des valeurs à l'élément hôte en appliquant les décorateurs `@HostBinding` et `@HostListener`
aux membres de la classe.

`@HostBinding` vous permet de lier des propriétés et des attributs de l'hôte à des propriétés et à des getters :

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

`@HostListener` vous permet de lier des écouteurs d'événements à l'élément hôte. Le décorateur accepte un nom
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

## Conflits de liaisons {#binding-collisions}

Quand vous utilisez un composant dans un template, vous pouvez ajouter des liaisons à l'élément de cette instance du composant.
Le composant peut _aussi_ définir des liaisons d'hôte pour les mêmes propriétés ou attributs.

```angular-ts
@Component({
  ...,
  host: {
    'role': 'presentation',
    '[id]': 'id',
  }
})
export class ProfilePhoto { /* ... */ }
```

```angular-html
<profile-photo role="group" [id]="otherId" />
```

Dans ce genre de cas, les règles suivantes déterminent quelle valeur l'emporte :

- Si les deux valeurs sont statiques, la liaison de l'instance l'emporte.
- Si une valeur est statique et l'autre dynamique, la valeur dynamique l'emporte.
- Si les deux valeurs sont dynamiques, la liaison d'hôte du composant l'emporte.

## Styliser avec les propriétés personnalisées CSS {#styling-with-css-custom-properties}

Les développeurs s'appuient souvent sur les [propriétés personnalisées CSS](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascading_variables/Using_CSS_custom_properties) pour permettre une configuration flexible des styles de leurs composants.
Vous pouvez définir ces propriétés personnalisées sur un élément hôte avec une [liaison de style](guide/templates/binding#css-style-properties).

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

Il est également possible de définir des propriétés personnalisées CSS sur l'élément hôte des composants enfants avec une [liaison de style](guide/templates/binding#css-style-properties).

```angular-ts
@Component({
  selector: 'my-component',
  template: `<my-child [style.--my-background]="color()" />`,
})
export class MyComponent {
  color = signal('lightgreen');
}
```

## Injecter les attributs de l'élément hôte {#injecting-host-element-attributes}

Les composants et les directives peuvent lire les attributs statiques de leur élément hôte en utilisant `HostAttributeToken` avec la fonction [`inject`](api/core/inject).

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
