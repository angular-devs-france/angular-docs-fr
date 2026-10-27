<!-- source: 9754515c033c0c41451e1e5cebc337481f95da6c -->
# Effects

Les signals sont utiles parce qu'ils notifient les consommateurs intéressés quand ils changent. Un **effect** est une opération qui s'exécute chaque fois que la valeur d'un ou plusieurs signals change. Vous pouvez créer un effect avec la fonction `effect` :

```ts
import {effect} from '@angular/core';

effect(() => {
  console.log(`The current count is: ${count()}`);
});
```

Les effects s'exécutent toujours **au moins une fois.** Quand un effect s'exécute, il suit toutes les lectures de valeurs de signals. Chaque fois que l'une de ces valeurs de signals change, l'effect s'exécute à nouveau. Comme les signals computed, les effects suivent leurs dépendances de manière dynamique et ne suivent que les signals lus lors de l'exécution la plus récente.

Les effects s'exécutent toujours de manière **asynchrone**, pendant le processus de détection de changements.

## Cas d'usage des effects {#use-cases-for-effects}

Les effects devraient être la dernière API à laquelle vous avez recours. Préférez toujours `computed()` pour les valeurs dérivées et `linkedSignal()` pour les valeurs qui peuvent être à la fois dérivées et définies manuellement. Si vous vous retrouvez à copier des données d'un signal à un autre avec un effect, c'est le signe que vous devriez remonter votre source de vérité et utiliser `computed()` ou `linkedSignal()` à la place. Les effects sont surtout adaptés pour synchroniser l'état des signals avec des API impératives qui n'utilisent pas les signals.

TIP: Il n'existe aucune situation où un effect est bon, seulement des situations où il est approprié.

- Journaliser des valeurs de signals, que ce soit pour l'analytique ou comme outil de débogage.
- Garder des données synchronisées avec différents types de stockage : `window.localStorage`, session storage, cookies, etc.
- Ajouter un comportement DOM personnalisé qui ne peut pas être exprimé avec la syntaxe de template.
- Effectuer un rendu personnalisé dans un élément `<canvas>`, une library de graphiques ou une autre library d'interface tierce.

<docs-callout critical title="Quand ne pas utiliser les effects">
Évitez d'utiliser les effects pour propager des changements d'état. Cela peut entraîner des erreurs `ExpressionChangedAfterItHasBeenChecked`, des mises à jour circulaires infinies ou des cycles de détection de changements inutiles.

Utilisez plutôt des signals `computed` pour modéliser un état qui dépend d'un autre état.
</docs-callout>

## Contexte d'injection {#injection-context}

Par défaut, vous ne pouvez créer un `effect()` que dans un [contexte d'injection](guide/di/dependency-injection-context) (où vous avez accès à la fonction `inject`). Le moyen le plus simple de satisfaire cette exigence est d'appeler `effect` dans le `constructor` d'un composant, d'une directive ou d'un service :

```ts
@Component(/* ... */)
export class EffectiveCounter {
  readonly count = signal(0);

  constructor() {
    // Register a new effect.
    effect(() => {
      console.log(`The count is: ${this.count()}`);
    });
  }
}
```

Pour créer un effect en dehors du constructeur, vous pouvez passer un `Injector` à `effect` via ses options :

```ts
@Component(/* ... */)
export class EffectiveCounter {
  readonly count = signal(0);
  private injector = inject(Injector);

  initializeLogging(): void {
    effect(
      () => {
        console.log(`The count is: ${this.count()}`);
      },
      {injector: this.injector},
    );
  }
}
```

## Exécution des effects {#execution-of-effects}

Angular définit implicitement deux comportements pour ses effects selon le contexte dans lequel ils ont été créés.

Un « View Effect » est un `effect` créé dans le contexte de l'instanciation d'un composant. Cela inclut les effects créés par des services liés aux injectors de composants.<br>
Un « Root Effect » est créé dans le contexte de l'instanciation d'un service fourni à la racine.

L'exécution de ces deux types d'`effect` est liée au processus de détection de changements.

- Les « View effects » sont exécutés _avant_ que leur composant correspondant soit vérifié par le processus de détection de changements.
- Les « Root effects » sont exécutés avant que tous les composants soient vérifiés par le processus de détection de changements.

Dans les deux cas, si au moins une des dépendances de l'effect a changé pendant son exécution, l'effect se réexécute avant de poursuivre le processus de détection de changements.

## Détruire les effects {#destroying-effects}

Quand un composant ou une directive est détruit, Angular nettoie automatiquement tous les effects associés.

Un `effect` peut être créé dans deux contextes différents, qui déterminent le moment où il est détruit :

- Un « View effect » est détruit quand le composant est détruit.
- Un « Root effect » est détruit quand l'application est détruite.

Les effects renvoient un `EffectRef`. Vous pouvez utiliser la méthode `destroy` de cette référence pour supprimer manuellement un effect. Vous pouvez combiner cela avec l'option `manualCleanup` lors de la création d'un effect pour désactiver le nettoyage automatique. Veillez à bien détruire ces effects quand ils ne sont plus nécessaires.

## Fonctions de nettoyage des effects {#effect-cleanup-functions}

Quand un composant ou une directive est détruit, Angular nettoie automatiquement tous les effects associés.
Les effects peuvent lancer des opérations de longue durée, que vous devriez annuler si l'effect est détruit ou s'exécute à nouveau avant la fin de la première opération. Quand vous créez un effect, votre fonction peut éventuellement accepter une fonction `onCleanup` comme premier paramètre. Cette fonction `onCleanup` vous permet d'enregistrer un callback invoqué avant le début de la prochaine exécution de l'effect, ou quand l'effect est détruit.

```ts
effect((onCleanup) => {
  const user = currentUser();

  const timer = setTimeout(() => {
    console.log(`1 second ago, the user became ${user}`);
  }, 1000);

  onCleanup(() => {
    clearTimeout(timer);
  });
});
```

## Effets de bord sur les éléments du DOM {#side-effects-on-dom-elements}

La fonction `effect` est un outil polyvalent pour exécuter du code en réaction aux changements de signals. Cependant, elle s'exécute _avant_ qu'Angular mette à jour le DOM. Dans certaines situations, vous pouvez avoir besoin d'inspecter ou de modifier manuellement le DOM, ou d'intégrer une library tierce qui nécessite un accès direct au DOM.

Pour ces situations, vous pouvez utiliser `afterRenderEffect`. Elle fonctionne comme `effect`, mais s'exécute après qu'Angular a terminé le rendu et appliqué ses changements au DOM.

```ts
@Component(/* ... */)
export class MyFancyChart {
  chartData = input.required<ChartData>();
  canvas = viewChild.required<ElementRef<HTMLCanvasElement>>('canvas');
  chart: ChartInstance;

  constructor() {
    // Run a single time to create the chart instance
    afterNextRender({
      write: () => {
        this.chart = initializeChart(this.canvas().nativeElement, this.chartData());
      },
    });

    // Re-run after DOM has been updated whenever `chartData` changes
    afterRenderEffect(() => {
      this.chart.updateData(this.chartData());
    });
  }
}
```

Dans cet exemple, `afterRenderEffect` est utilisée pour mettre à jour un graphique créé par une library tierce.

TIP: Vous n'avez souvent pas besoin de `afterRenderEffect` pour détecter les changements du DOM. Des API comme `ResizeObserver`, `MutationObserver` et `IntersectionObserver` sont préférables à `effect` ou `afterRenderEffect` lorsque c'est possible.

### Phases de rendu {#render-phases}

Accéder au DOM et le modifier peut avoir un impact sur les performances de votre application, par exemple en déclenchant trop de [reflows](https://developer.mozilla.org/en-US/docs/Glossary/Reflow) inutiles.

Pour optimiser ces opérations, `afterRenderEffect` propose quatre phases pour regrouper les callbacks et les exécuter dans un ordre optimisé.

Les phases sont :

| Phase            | Description                                                                                                                                                                                        |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `earlyRead`      | Utilisez cette phase pour lire le DOM avant un callback d'écriture ultérieur, par exemple pour effectuer une mise en page personnalisée que le navigateur ne prend pas en charge nativement. Préférez la phase de lecture si la lecture peut attendre. |
| `write`          | Utilisez cette phase pour écrire dans le DOM. Ne lisez **jamais** le DOM dans cette phase.                                                                                                                     |
| `mixedReadWrite` | Utilisez cette phase pour lire et écrire dans le DOM simultanément. N'utilisez jamais cette phase s'il est possible de répartir le travail entre les autres phases.                                         |
| `read`           | Utilisez cette phase pour lire le DOM. N'écrivez **jamais** dans le DOM dans cette phase.                                                                                                                     |

L'utilisation de ces phases aide à éviter le layout thrashing et garantit que vos opérations sur le DOM sont effectuées de manière sûre et efficace.

Vous pouvez spécifier les phases en passant à `afterRenderEffect` un objet contenant un callback pour chaque phase :

```ts
afterRenderEffect({
  earlyRead: (cleanupFn) => {
    /* ... */
  },
  write: (previousPhaseValue, cleanupFn) => {
    /* ... */
  },
  mixedReadWrite: (previousPhaseValue, cleanupFn) => {
    /* ... */
  },
  read: (previousPhaseValue, cleanupFn) => {
    /* ... */
  },
});
```

CRITICAL: Si vous ne spécifiez pas la phase, `afterRenderEffect` exécute les callbacks pendant la phase `mixedReadWrite`. Cela peut dégrader les performances de l'application en provoquant des reflows supplémentaires du DOM.

#### Exécution des phases {#phase-executions}

Le callback de la phase `earlyRead` ne reçoit que la fonction de nettoyage. Chaque phase suivante reçoit, sous forme de Signal, la valeur de retour du callback de la phase précédente. Vous pouvez vous en servir pour coordonner le travail entre les phases.

Les effects s'exécutent dans l'ordre de phases suivant :

1. `earlyRead`
2. `write`
3. `mixedReadWrite`
4. `read`

Si l'une des phases modifie la valeur d'un signal suivi par `afterRenderEffect`, les phases concernées s'exécutent à nouveau.

#### Nettoyage {#cleanup}

Chaque phase fournit une fonction de nettoyage (callback) en argument. Les callbacks de nettoyage sont exécutés quand l'`afterRenderEffect` est détruit ou avant la réexécution des effects de phase.

### Mises en garde concernant le rendu côté serveur {#server-side-rendering-caveats}

`afterRenderEffect`, comme `afterNextRender`/`afterEveryRender`, ne s'exécute que côté client.

NOTE: Rien ne garantit que les composants soient [hydratés](/guide/hydration) avant l'exécution du callback. Soyez prudent quand vous lisez ou écrivez directement dans le DOM et la mise en page.
