<!-- source: 75f6165f7353505615bc641be282226864586c72 -->
# Interopérabilité RxJS avec les outputs de composants et de directives

TIP: Ce guide suppose que vous connaissez les [outputs de composants et de directives](guide/components/outputs).

Le package `@angular/core/rxjs-interop` propose deux API liées aux outputs de composants et de directives.

## Créer un output à partir d'un observable RxJS {#creating-an-output-based-on-an-rxjs-observable}

La fonction `outputFromObservable` vous permet de créer un output de composant ou de directive qui émet en fonction d'un observable RxJS :

```ts {highlight:[9]}
import {Directive} from '@angular/core';
import {outputFromObservable} from '@angular/core/rxjs-interop';
import {Observable} from 'rxjs';

@Directive(/* ... */)
class Draggable {
  pointerMoves$: Observable<PointerMovements> = listenToPointerMoves();

  // Whenever `pointerMoves$` emits, the `pointerMove` event fires.
  pointerMove = outputFromObservable(this.pointerMoves$);
}
```

La fonction `outputFromObservable` a une signification particulière pour le compilateur Angular. **Vous ne pouvez appeler `outputFromObservable` que dans les initialiseurs de propriétés de composants et de directives.**

Lorsque vous vous abonnez (`subscribe`) à l'output, Angular transmet automatiquement l'abonnement à l'observable sous-jacent. Angular cesse de transmettre les valeurs lorsque le composant ou la directive est détruit.

HELPFUL: Envisagez d'utiliser directement `output()` si vous pouvez émettre les valeurs de manière impérative.

## Créer un observable RxJS à partir d'un output de composant ou de directive {#creating-an-rxjs-observable-from-a-component-or-directive-output}

La fonction `outputToObservable` vous permet de créer un observable RxJS à partir d'un output de composant.

```ts {highlight:[11]}
import {outputToObservable} from '@angular/core/rxjs-interop';
import {Component, output} from '@angular/core';

@Component(/*...*/)
class CustomSlider {
    valueChange = output<number>();
}

// Instance reference to `CustomSlider`.
const slider: CustomSlider = createSlider();

outputToObservable(slider.valueChange) // Observable<number>
    .pipe(...)
    .subscribe(...);
```

HELPFUL: Envisagez d'utiliser directement la méthode `subscribe` de `OutputRef` si elle répond à vos besoins.
