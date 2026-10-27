<!-- source: f7f96e338b72d80fcc0b94ba7f38067fa408fb60 -->
# Interopérabilité RxJS avec les signals Angular

Le package `@angular/core/rxjs-interop` propose des API qui vous aident à intégrer RxJS et les signals Angular.

## Créer un signal à partir d'un observable RxJS avec `toSignal` {#create-a-signal-from-an-rxjs-observable-with-tosignal}

Utilisez la fonction `toSignal` pour créer un signal qui suit la valeur d'un observable. Elle se comporte de manière similaire au pipe `async` dans les templates, mais elle est plus flexible et peut être utilisée n'importe où dans une application.

```angular-ts
import {Component} from '@angular/core';
import {AsyncPipe} from '@angular/common';
import {interval} from 'rxjs';
import {toSignal} from '@angular/core/rxjs-interop';

@Component({
  template: `{{ counter() }}`,
})
export class Ticker {
  counterObservable = interval(1000);

  // Get a `Signal` representing the `counterObservable`'s value.
  counter = toSignal(this.counterObservable, {initialValue: 0});
}
```

Comme le pipe `async`, `toSignal` s'abonne immédiatement à l'observable, ce qui peut déclencher des effets de bord. L'abonnement créé par `toSignal` se désabonne automatiquement de l'observable donné lorsque le composant ou le service qui appelle `toSignal` est détruit.

IMPORTANT: `toSignal` crée un abonnement. Évitez de l'appeler plusieurs fois pour le même observable, et réutilisez plutôt le signal qu'il renvoie.

### Contexte d'injection {#injection-context}

Par défaut, `toSignal` doit s'exécuter dans un [contexte d'injection](guide/di/dependency-injection-context), par exemple pendant la construction d'un composant ou d'un service. Si aucun contexte d'injection n'est disponible, vous pouvez spécifier manuellement l'`Injector` à utiliser à la place.

### Valeurs initiales {#initial-values}

Les observables ne produisent pas forcément de valeur de manière synchrone lors de l'abonnement, mais les signals exigent toujours une valeur courante. Il existe plusieurs façons de gérer cette valeur « initiale » des signals créés par `toSignal`.

#### L'option `initialValue` {#the-initialvalue-option}

Comme dans l'exemple ci-dessus, vous pouvez spécifier une option `initialValue` avec la valeur que le signal doit renvoyer avant la première émission de l'observable.

#### Valeurs initiales `undefined` {#undefined-initial-values}

Si vous ne fournissez pas d'`initialValue`, le signal obtenu renvoie `undefined` jusqu'à ce que l'observable émette. Ce comportement est similaire à celui du pipe `async`, qui renvoie `null`.

#### L'option `requireSync` {#the-requiresync-option}

Certains observables émettent de manière synchrone de façon garantie, comme `BehaviorSubject`. Dans ce cas, vous pouvez spécifier l'option `requireSync: true`.

Lorsque `requireSync` vaut `true`, `toSignal` impose que l'observable émette de manière synchrone lors de l'abonnement. Cela garantit que le signal a toujours une valeur, et qu'aucun type `undefined` ni valeur initiale n'est nécessaire.

### `manualCleanup` {#manualcleanup}

Par défaut, `toSignal` se désabonne automatiquement de l'observable lorsque le composant ou le service qui le crée est détruit.

Pour modifier ce comportement, vous pouvez passer l'option `manualCleanup`. Vous pouvez utiliser ce paramètre pour les observables qui se terminent naturellement d'eux-mêmes.

#### Comparaison d'égalité personnalisée {#custom-equality-comparison}

Certains observables peuvent émettre des valeurs **égales** bien qu'elles diffèrent par leur référence ou par un détail mineur. L'option `equal` vous permet de définir une **fonction d'égalité personnalisée** pour déterminer quand deux valeurs consécutives doivent être considérées comme identiques.

Lorsque deux valeurs émises sont considérées comme égales, le signal obtenu **n'est pas mis à jour**. Cela évite les calculs redondants, les mises à jour du DOM ou la réexécution inutile des effects.

```ts
import {Component} from '@angular/core';
import {toSignal} from '@angular/core/rxjs-interop';
import {interval, map} from 'rxjs';

@Component(/* ... */)
export class EqualExample {
  temperature$ = interval(1000).pipe(
    map(() => ({temperature: Math.floor(Math.random() * 3) + 20})), // 20, 21, or 22 randomly
  );

  // Only update if the temperature changes
  temperature = toSignal(this.temperature$, {
    initialValue: {temperature: 20},
    equal: (prev, curr) => prev.temperature === curr.temperature,
  });
}
```

### Erreur et complétion {#error-and-completion}

Si un observable utilisé dans `toSignal` produit une erreur, cette erreur est levée lors de la lecture du signal.

Si un observable utilisé dans `toSignal` se termine, le signal continue de renvoyer la dernière valeur émise avant la complétion.

## Créer un observable RxJS à partir d'un signal avec `toObservable` {#create-an-rxjs-observable-from-a-signal-with-toobservable}

Utilisez l'utilitaire `toObservable` pour créer un `Observable` qui suit la valeur d'un signal. La valeur du signal est surveillée par un `effect` qui émet la valeur vers l'observable lorsqu'elle change.

```ts
import {Component, signal, Signal, inject} from '@angular/core';
import {toObservable} from '@angular/core/rxjs-interop';
import {switchMap} from 'rxjs';

@Component(/* ... */)
export class SearchResults {
  query: Signal<string> = inject(QueryService).query;
  query$ = toObservable(this.query);

  results$ = this.query$.pipe(switchMap((query) => this.http.get('/search?q=' + query)));
}
```

Lorsque le signal `query` change, l'observable `query$` émet la dernière recherche et déclenche une nouvelle requête HTTP.

### Contexte d'injection {#injection-context-to-observable}

Par défaut, `toObservable` doit s'exécuter dans un [contexte d'injection](guide/di/dependency-injection-context), par exemple pendant la construction d'un composant ou d'un service. Si aucun contexte d'injection n'est disponible, vous pouvez spécifier manuellement l'`Injector` à utiliser à la place.

### Timing de `toObservable` {#timing-of-toobservable}

`toObservable` utilise un effect pour suivre la valeur du signal dans un `ReplaySubject`. Lors de l'abonnement, la première valeur (si elle est disponible) peut être émise de manière synchrone, et toutes les valeurs suivantes seront asynchrones.

Contrairement aux observables, les signals ne fournissent jamais de notification synchrone des changements. Même si vous mettez à jour la valeur d'un signal plusieurs fois, `toObservable` n'émettra la valeur qu'une fois le signal stabilisé.

```ts
const obs$ = toObservable(mySignal);
obs$.subscribe((value) => console.log(value));

mySignal.set(1);
mySignal.set(2);
mySignal.set(3);
```

Ici, seule la dernière valeur (3) sera affichée dans la console.

## Utiliser `rxResource` pour les données asynchrones {#using-rxresource-for-async-data}

La [fonction `resource`](/guide/signals/resource) d'Angular vous permet d'intégrer des données asynchrones dans le code de votre application basé sur les signals. En s'appuyant sur ce pattern, `rxResource` vous permet de définir une resource dont la source de données est définie sous la forme d'un `Observable` RxJS. Au lieu d'accepter une fonction `loader`, `rxResource` accepte une fonction `stream` qui renvoie un `Observable` RxJS.

```typescript
import {Component, inject, input} from '@angular/core';
import {rxResource} from '@angular/core/rxjs-interop';

@Component(/* ... */)
export class UserProfile {
  // This component relies on a service that exposes data through an RxJS Observable.
  private userData = inject(MyUserDataClient);

  protected userId = input<string>();

  private userResource = rxResource({
    params: () => ({userId: this.userId()}),

    // The `stream` property expects a factory function that returns
    // a data stream as an RxJS Observable.
    stream: ({params}) => this.userData.load(params.userId),
  });
}
```

La propriété `stream` accepte une fonction de fabrique pour un `Observable` RxJS. Cette fonction de fabrique reçoit la valeur `params` de la resource et renvoie un `Observable`. La resource appelle cette fonction de fabrique chaque fois que le calcul de `params` produit une nouvelle valeur. Consultez [Loaders de resources](/guide/signals/resource#resource-loaders) pour plus de détails sur les paramètres transmis à la fonction de fabrique.

Pour tout le reste, `rxResource` se comporte comme `resource` et fournit les mêmes API pour spécifier des paramètres, lire des valeurs, vérifier l'état de chargement et examiner les erreurs.

L'`Observable` renvoyé par `stream` doit toujours émettre une valeur ou une erreur avant de se terminer ; consultez [`NG0991`](/errors/NG0991) pour savoir ce qui se passe dans le cas contraire et comment l'éviter.
