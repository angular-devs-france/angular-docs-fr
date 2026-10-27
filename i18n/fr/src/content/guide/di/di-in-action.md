<!-- source: de0b7f5dceb67ab0e8cc87a25ad40489c6cd982a -->
# La DI en pratique

Ce guide explore d'autres fonctionnalités de l'injection de dépendances (DI) dans Angular.

NOTE: Pour une présentation complète d'InjectionToken et des providers personnalisés, consultez le [guide sur la définition des providers de dépendances](guide/di/defining-dependency-providers#injection-tokens).

## Injecter l'élément DOM du composant {#inject-the-components-dom-element}

Bien que les développeurs l'évitent généralement, certains effets visuels et outils tiers vous obligent à accéder directement au DOM.
Dans ces cas, vous pouvez avoir besoin d'accéder à l'élément DOM d'un composant.

Angular expose l'élément DOM sous-jacent d'un `@Component` ou d'une `@Directive` par injection, grâce au jeton `ElementRef` :

```ts {highlight:[7]}
import {Directive, ElementRef, inject} from '@angular/core';

@Directive({
  selector: '[appHighlight]',
})
export class HighlightDirective {
  private element = inject(ElementRef);

  update() {
    this.element.nativeElement.style.color = 'red';
  }
}
```

## Injecter le nom de balise du host element {#inject-the-host-elements-tag-name}

Pour obtenir le nom de balise d'un host element, injectez-le avec le jeton `HOST_TAG_NAME`.

```ts
import {Directive, HOST_TAG_NAME, inject} from '@angular/core';

@Directive({
  selector: '[roleButton]',
})
export class RoleButtonDirective {
  private tagName = inject(HOST_TAG_NAME);

  onAction() {
    switch (this.tagName) {
      case 'button':
        // Handle button action
        break;
      case 'a':
        // Handle anchor action
        break;
      default:
        // Handle other elements
        break;
    }
  }
}
```

NOTE: Si le host element peut ne pas avoir de nom de balise (par exemple `ng-container` ou `ng-template`), rendez l'injection facultative.

## Résoudre les dépendances circulaires avec une référence anticipée {#resolve-circular-dependencies-with-a-forward-reference}

En TypeScript, l'ordre des déclarations de classes compte.
Vous ne pouvez pas faire directement référence à une classe tant que vous ne l'avez pas définie.

Ce n'est généralement pas un problème, surtout si vous respectez la règle recommandée d'_une classe par fichier_.
Cependant, dans certains cas, les références circulaires sont inévitables.
Par exemple, si la classe 'A' fait référence à la classe 'B' et que la classe 'B' fait référence à la classe 'A', l'une d'elles doit être définie en premier.

La fonction Angular `forwardRef()` crée une référence _indirecte_ qu'Angular peut résoudre plus tard.

Vous rencontrez un problème similaire quand une classe fait _référence à elle-même_.
Par exemple, dans son tableau `providers`.
Le tableau `providers` est une propriété de la fonction décorateur `@Component()`, qui doit apparaître avant la définition de la classe.
Ces références circulaires peuvent être résolues avec `forwardRef`.

```typescript {header: 'app.component.ts', highlight: [4]}
providers: [
  {
    provide: PARENT_MENU_ITEM,
    useExisting: forwardRef(() => MenuItem),
  },
],
```
