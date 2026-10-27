<!-- source: 7b9170c93d04983018f7d1d4f6b39cb8e0659731 -->
<docs-decorative-header title="Angular Signals" imgSrc="adev/src/assets/images/signals.svg"> <!-- markdownlint-disable-line -->
Angular Signals est un système qui suit de manière granulaire comment et où votre état est utilisé dans toute une application, ce qui permet au framework d'optimiser les mises à jour du rendu.
</docs-decorative-header>

TIP: Consultez les [Fondamentaux](essentials/signals) d'Angular avant de vous plonger dans ce guide complet.

## Que sont les signals ? {#what-are-signals}

Un **signal** est une enveloppe autour d'une valeur qui notifie les consommateurs intéressés quand cette valeur change. Les signals peuvent contenir n'importe quelle valeur, des types primitifs aux structures de données complexes.

Vous lisez la valeur d'un signal en appelant sa fonction getter, ce qui permet à Angular de suivre où le signal est utilisé.

Les signals peuvent être soit _modifiables_, soit _en lecture seule_.

### Signals modifiables {#writable-signals}

Les signals modifiables fournissent une API pour mettre à jour directement leur valeur. Vous créez des signals modifiables en appelant la fonction `signal` avec la valeur initiale du signal :

```ts
const count = signal(0);

// Signals are getter functions - calling them reads their value.
console.log('The count is: ' + count());
```

Pour changer la valeur d'un signal modifiable, utilisez soit `.set()` pour la définir directement :

```ts
count.set(3);
```

soit l'opération `.update()` pour calculer une nouvelle valeur à partir de la précédente :

```ts
// Increment the count by 1.
count.update((value) => value + 1);
```

Les signals modifiables ont le type `WritableSignal`.

#### Convertir des signals modifiables en lecture seule {#converting-writable-signals-to-readonly}

`WritableSignal` fournit une méthode `asReadonly()` qui renvoie une version en lecture seule du signal. C'est utile quand vous voulez exposer la valeur d'un signal aux consommateurs sans leur permettre de la modifier directement :

```ts
@Service()
export class CounterState {
  // Private writable state
  private readonly _count = signal(0);

  readonly count = this._count.asReadonly(); // public readonly

  increment() {
    this._count.update((v) => v + 1);
  }
}

@Component({/* ... */})
export class AwesomeCounter {
  state = inject(CounterState);

  count = this.state.count; // can read but not modify

  increment() {
    this.state.increment();
  }
}
```

Le signal en lecture seule reflète tous les changements apportés au signal modifiable d'origine, mais ne peut pas être modifié avec les méthodes `set()` ou `update()`.

IMPORTANT: Les signals en lecture seule ne disposent **d'aucun** mécanisme intégré qui empêcherait la mutation profonde de leur valeur.

### Signals computed {#computed-signals}

Les **signals computed** sont des signals en lecture seule qui dérivent leur valeur d'autres signals. Vous définissez des signals computed avec la fonction `computed` en spécifiant une dérivation :

```typescript
const count: WritableSignal<number> = signal(0);
const doubleCount: Signal<number> = computed(() => count() * 2);
```

Le signal `doubleCount` dépend du signal `count`. Chaque fois que `count` est mis à jour, Angular sait que `doubleCount` doit lui aussi être mis à jour.

#### Les signals computed sont à la fois évalués paresseusement et mémoïsés {#computed-signals-are-both-lazily-evaluated-and-memoized}

La fonction de dérivation de `doubleCount` ne s'exécute pas pour calculer sa valeur avant la première lecture de `doubleCount`. La valeur calculée est ensuite mise en cache, et si vous lisez à nouveau `doubleCount`, il renvoie la valeur en cache sans la recalculer.

Si vous modifiez ensuite `count`, Angular sait que la valeur en cache de `doubleCount` n'est plus valide, et la prochaine fois que vous lirez `doubleCount`, sa nouvelle valeur sera calculée.

Par conséquent, vous pouvez effectuer sans risque des dérivations coûteuses en calcul dans les signals computed, comme le filtrage de tableaux.

#### Les signals computed ne sont pas des signals modifiables {#computed-signals-are-not-writable-signals}

Vous ne pouvez pas assigner directement des valeurs à un signal computed. Autrement dit,

```ts
doubleCount.set(3);
```

produit une erreur de compilation, car `doubleCount` n'est pas un `WritableSignal`.

#### Les dépendances des signals computed sont dynamiques {#computed-signal-dependencies-are-dynamic}

Seuls les signals réellement lus pendant la dérivation sont suivis. Par exemple, dans ce `computed`, le signal `count` n'est lu que si le signal `showCount` vaut true :

```ts
const showCount = signal(false);
const count = signal(0);
const conditionalCount = computed(() => {
  if (showCount()) {
    return `The count is ${count()}.`;
  } else {
    return 'Nothing to see here!';
  }
});
```

Quand vous lisez `conditionalCount`, si `showCount` vaut `false`, le message `'Nothing to see here!'` est renvoyé _sans_ lire le signal `count`. Cela signifie que si vous mettez à jour `count` plus tard, cela ne provoquera _pas_ de nouveau calcul de `conditionalCount`.

Si vous définissez `showCount` à `true` puis lisez à nouveau `conditionalCount`, la dérivation se réexécute et prend la branche où `showCount` vaut `true`, renvoyant le message qui affiche la valeur de `count`. Modifier `count` invalidera alors la valeur en cache de `conditionalCount`.

Notez que des dépendances peuvent être retirées pendant une dérivation, tout comme elles peuvent être ajoutées. Si vous redéfinissez plus tard `showCount` à `false`, alors `count` ne sera plus considéré comme une dépendance de `conditionalCount`.

## Contextes réactifs {#reactive-contexts}

Un **contexte réactif** est un état d'exécution dans lequel Angular surveille les lectures de signals pour établir une dépendance. Le code qui lit le signal est le _consommateur_, et le signal lu est le _producteur_.

Angular entre automatiquement dans un contexte réactif lors des opérations suivantes :

- L'exécution d'un callback `effect` ou `afterRenderEffect`.
- L'évaluation d'un signal `computed`.
- L'évaluation d'un `linkedSignal`.
- L'évaluation de la fonction params d'une `resource`.
- Le rendu du template d'un composant (y compris les bindings dans la [propriété host](guide/components/host-elements#binding-to-the-host-element)).

Pendant ces opérations, Angular crée une connexion _active_. Si un signal suivi change, Angular réexécutera _à terme_ le consommateur.

### Vérifier le contexte réactif {#asserts-the-reactive-context}

Angular fournit la fonction utilitaire `assertNotInReactiveContext` pour vérifier que du code ne s'exécute pas dans un contexte réactif. Passez une référence à la fonction appelante pour que le message d'erreur pointe vers le bon point d'entrée de l'API si la vérification échoue. Cela produit un message d'erreur plus clair et plus exploitable qu'une erreur générique de contexte réactif.

```ts
import {assertNotInReactiveContext} from '@angular/core';

function subscribeToEvents() {
  assertNotInReactiveContext(subscribeToEvents);
  // Safe to proceed - subscription logic here
}
```

### Lire sans suivre les dépendances {#reading-without-tracking-dependencies}

Dans de rares cas, vous pouvez vouloir exécuter du code susceptible de lire des signals au sein d'une fonction réactive comme `computed` ou `effect` _sans_ créer de dépendance.

Par exemple, supposons que lorsque `currentUser` change, la valeur d'un `counter` doive être journalisée. Vous pourriez créer un `effect` qui lit les deux signals :

```ts
effect(() => {
  console.log(`User set to ${currentUser()} and the counter is ${counter()}`);
});
```

Cet exemple journalise un message quand `currentUser` _ou_ `counter` change. Cependant, si l'effect ne doit s'exécuter que lorsque `currentUser` change, alors la lecture de `counter` n'est qu'accessoire et les changements de `counter` ne devraient pas journaliser de nouveau message.

Vous pouvez empêcher le suivi de la lecture d'un signal en appelant son getter avec `untracked` :

```ts
effect(() => {
  console.log(`User set to ${currentUser()} and the counter is ${untracked(counter)}`);
});
```

`untracked` est aussi utile quand un effect doit invoquer du code externe qui ne doit pas être traité comme une dépendance :

```ts
effect(() => {
  const user = currentUser();
  untracked(() => {
    // If the `loggingService` reads signals, they won't be counted as
    // dependencies of this effect.
    this.loggingService.log(`User set to ${user}`);
  });
});
```

### Contexte réactif et opérations asynchrones {#reactive-context-and-async-operations}

Le contexte réactif n'est actif que pour le code synchrone. Toute lecture de signal qui se produit après une frontière asynchrone ne sera pas suivie comme dépendance.

```ts {avoid}
effect(async () => {
  const data = await fetchUserData();
  // Reactive context is lost here - theme() won't be tracked
  console.log(`User: ${data.name}, Theme: ${theme()}`);
});
```

Pour garantir que toutes les lectures de signals sont suivies, lisez les signals avant le `await`. Cela inclut le fait de les passer en arguments à la fonction attendue, puisque les arguments sont évalués de manière synchrone :

```ts {prefer}
effect(async () => {
  const currentTheme = theme(); // Read before await
  const data = await fetchUserData();
  console.log(`User: ${data.name}, Theme: ${currentTheme}`);
});
```

```ts {prefer}
effect(async () => {
  // Also works: signal is read before await (as function argument)
  await renderContent(docContent());
});
```

## Dérivations avancées {#advanced-derivations}

Alors que `computed` gère les dérivations simples en lecture seule, vous pourriez avoir besoin d'un état modifiable qui dépend d'autres signals.
Pour plus d'informations, consultez le guide [État dépendant avec linkedSignal](/guide/signals/linked-signal).

Toutes les API de signals sont synchrones : `signal`, `computed`, `input`, etc. Cependant, les applications doivent souvent gérer des données disponibles de manière asynchrone. Une `Resource` vous permet d'intégrer des données asynchrones dans le code de votre application basé sur les signals tout en vous permettant d'accéder à ses données de manière synchrone. Pour plus d'informations, consultez le guide [Réactivité asynchrone avec les resources](/guide/signals/resource).

## Exécuter des effets de bord sur des API non réactives {#executing-side-effects-on-non-reactive-apis}

Les dérivations synchrones ou asynchrones sont recommandées quand nous voulons réagir aux changements d'état. Cependant, cela ne couvre pas tous les cas d'usage possibles, et vous vous trouverez parfois dans une situation où vous devez réagir aux changements de signals sur des API non réactives. Utilisez `effect` ou `afterRenderEffect` pour ces cas d'usage spécifiques. Pour plus d'informations, consultez le guide [Effets de bord pour les API non réactives](/guide/signals/effect).

## Lire des signals dans les composants `OnPush` {#reading-signals-in-onpush-components}

Quand vous lisez un signal dans le template d'un composant `OnPush`, Angular suit le signal comme une dépendance de ce composant. Quand la valeur de ce signal change, Angular [marque](api/core/ChangeDetectorRef#markForCheck) automatiquement le composant pour garantir qu'il soit mis à jour lors de la prochaine exécution de la détection de changements. Consultez le guide [Ignorer des sous-arbres de composants](best-practices/skipping-subtrees) pour plus d'informations sur les composants `OnPush`.

## Sujets avancés {#advanced-topics}

### Fonctions d'égalité des signals {#signal-equality-functions}

Lors de la création d'un signal, vous pouvez éventuellement fournir une fonction d'égalité, qui servira à vérifier si la nouvelle valeur est réellement différente de la précédente.

```ts
import {signal} from '@angular/core';
import isEqual from 'lodash/isEqual';

const data = signal(['test'], {equal: isEqual});

// Even though this is a different array instance, the deep equality
// function will consider the values to be equal, and the signal won't
// trigger any updates.
data.set(['test']);
```

Les fonctions d'égalité peuvent être fournies aussi bien aux signals modifiables qu'aux signals computed.

HELPFUL: Par défaut, les signals utilisent l'égalité référentielle (comparaison avec [`Object.is()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/is)).

### Vérifier le type des signals {#type-checking-signals}

Vous pouvez utiliser `isSignal` pour vérifier si une valeur est un `Signal` :

```ts
const count = signal(0);
const doubled = computed(() => count() * 2);

isSignal(count); // true
isSignal(doubled); // true
isSignal(42); // false
```

Pour vérifier spécifiquement si un signal est modifiable, utilisez `isWritableSignal` :

```ts
const count = signal(0);
const doubled = computed(() => count() * 2);

isWritableSignal(count); // true
isWritableSignal(doubled); // false
```

## Utiliser les signals avec RxJS {#using-signals-with-rxjs}

Consultez [Interopérabilité RxJS avec les signals Angular](ecosystem/rxjs-interop) pour plus de détails sur l'interopérabilité entre les signals et RxJS.

## Déboguer les signals {#debugging-signals}

Angular DevTools peut visualiser le graphe de signals d'un composant ou d'une directive, en montrant comment ses signals, computeds, effects et autres nœuds réactifs dépendent les uns des autres.
Consultez [Déboguer le graphe de signals](tools/devtools/signals) pour apprendre à ouvrir, lire et inspecter le graphe.
