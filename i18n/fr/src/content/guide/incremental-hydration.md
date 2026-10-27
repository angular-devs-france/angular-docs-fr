<!-- source: e4ac873f9efb8d8444b76082b39d8043a73fe638 -->
# Hydratation incrémentale

L'**hydratation incrémentale** est un type avancé d'[hydratation](guide/hydration) qui peut laisser des sections de votre application déshydratées et déclencher _progressivement_ l'hydratation de ces sections lorsqu'elles deviennent nécessaires.

## Pourquoi utiliser l'hydratation incrémentale ? {#why-use-incremental-hydration}

L'hydratation incrémentale est une amélioration des performances qui s'appuie sur l'hydratation complète de l'application. Elle peut produire des bundles initiaux plus petits tout en offrant à l'utilisateur final une expérience comparable à celle d'une hydratation complète de l'application. Des bundles plus petits améliorent les temps de chargement initiaux, en réduisant le [First Input Delay (FID)](https://web.dev/fid) et le [Cumulative Layout Shift (CLS)](https://web.dev/cls).

L'hydratation incrémentale vous permet aussi d'utiliser des vues différables (`@defer`) pour du contenu qui ne pouvait pas être différé auparavant. En particulier, vous pouvez désormais utiliser des vues différables pour du contenu situé au-dessus de la ligne de flottaison. Avant l'hydratation incrémentale, placer un bloc `@defer` au-dessus de la ligne de flottaison entraînait le rendu du contenu du placeholder, ensuite remplacé par le contenu du template principal du bloc `@defer`. Cela provoquait un décalage de mise en page. Avec l'hydratation incrémentale, le template principal du bloc `@defer` est rendu sans décalage de mise en page lors de l'hydratation.

## Comment activer l'hydratation incrémentale dans Angular ? {#how-do-you-enable-incremental-hydration-in-angular}

Vous pouvez activer l'hydratation incrémentale pour les applications qui utilisent déjà le rendu côté serveur (SSR) avec hydratation. Suivez d'abord le [guide SSR d'Angular](guide/ssr) pour activer le rendu côté serveur, puis le [guide de l'hydratation d'Angular](guide/hydration) pour activer l'hydratation.

L'hydratation incrémentale est activée par défaut lorsque vous utilisez `provideClientHydration()`.

```ts
import {bootstrapApplication, provideClientHydration} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideClientHydration()],
});
```

NOTE: L'hydratation incrémentale dépend de l'[event replay](guide/hydration#capturing-and-replaying-events) et l'active automatiquement. Si `withEventReplay()` figure déjà dans votre liste, vous pouvez le retirer sans risque.

Pour désactiver l'hydratation incrémentale, utilisez `withNoIncrementalHydration()` :

```ts
import {
  bootstrapApplication,
  provideClientHydration,
  withNoIncrementalHydration,
} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideClientHydration(withNoIncrementalHydration())],
});
```

## Comment fonctionne l'hydratation incrémentale ? {#how-does-incremental-hydration-work}

L'hydratation incrémentale s'appuie sur l'[hydratation](guide/hydration) complète de l'application, les [vues différables](/guide/templates/defer) et l'[event replay](guide/hydration#capturing-and-replaying-events). Avec l'hydratation incrémentale, vous pouvez ajouter aux blocs `@defer` des déclencheurs supplémentaires qui définissent des frontières d'hydratation incrémentale. Ajouter un déclencheur `hydrate` à un bloc defer indique à Angular qu'il doit charger les dépendances de ce bloc defer pendant le rendu côté serveur et rendre le template principal plutôt que le `@placeholder`. Lors du rendu côté client, les dépendances restent différées, et le contenu du bloc defer reste déshydraté jusqu'à ce que son déclencheur `hydrate` se déclenche. Ce déclencheur indique au bloc defer de récupérer ses dépendances et d'hydrater le contenu. Tous les événements du navigateur déclenchés par l'utilisateur avant l'hydratation, plus précisément ceux qui correspondent aux listeners enregistrés dans votre composant, sont mis en file d'attente et rejoués une fois le processus d'hydratation terminé.

## Contrôler l'hydratation du contenu avec des déclencheurs {#controlling-hydration-of-content-with-triggers}

Vous pouvez spécifier des **déclencheurs d'hydratation** qui contrôlent le moment où Angular charge et hydrate le contenu différé. Ce sont des déclencheurs supplémentaires, utilisables en complément des déclencheurs `@defer` habituels.

Chaque bloc `@defer` peut avoir plusieurs déclencheurs d'hydratation, séparés par un point-virgule (`;`). Angular déclenche l'hydratation dès que _l'un_ des déclencheurs se déclenche.

Il existe trois types de déclencheurs d'hydratation : `hydrate on`, `hydrate when` et `hydrate never`.

### `hydrate on`

`hydrate on` spécifie une condition qui détermine quand l'hydratation du bloc `@defer` est déclenchée.

Les déclencheurs disponibles sont les suivants :

| Déclencheur                                         | Description                                                            |
| --------------------------------------------------- | ---------------------------------------------------------------------- |
| [`hydrate on idle`](#hydrate-on-idle)               | Se déclenche lorsque le navigateur est inactif. Accepte un délai d'expiration optionnel. |
| [`hydrate on viewport`](#hydrate-on-viewport)       | Se déclenche lorsque le contenu spécifié entre dans le viewport        |
| [`hydrate on interaction`](#hydrate-on-interaction) | Se déclenche lorsque l'utilisateur interagit avec l'élément spécifié  |
| [`hydrate on hover`](#hydrate-on-hover)             | Se déclenche lorsque la souris survole la zone spécifiée               |
| [`hydrate on immediate`](#hydrate-on-immediate)     | Se déclenche immédiatement après la fin du rendu du contenu non différé |
| [`hydrate on timer`](#hydrate-on-timer)             | Se déclenche après une durée donnée                                    |

#### `hydrate on idle`

Le déclencheur `hydrate on idle` charge les dépendances de la vue différable et hydrate le contenu dès que le navigateur a atteint un état d'inactivité, en s'appuyant sur `requestIdleCallback`.

Vous pouvez éventuellement spécifier un délai d'expiration en millisecondes, transmis à [`requestIdleCallback`](https://developer.mozilla.org/docs/Web/API/Window/requestIdleCallback). Si le navigateur ne planifie pas le callback assez tôt, le travail s'exécutera au plus tard à l'expiration du délai spécifié.

```angular-html
@defer (hydrate on idle) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}

<!-- With a 500ms timeout -->
@defer (hydrate on idle(500)) {
  <large-cmp />
}
```

#### `hydrate on viewport`

Le déclencheur `hydrate on viewport` charge les dépendances de la vue différable et hydrate la partie correspondante de l'application lorsque le contenu spécifié entre dans le viewport, à l'aide de l'
[API Intersection Observer](https://developer.mozilla.org/docs/Web/API/Intersection_Observer_API).

```angular-html
@defer (hydrate on viewport) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

#### `hydrate on interaction`

Le déclencheur `hydrate on interaction` charge les dépendances de la vue différable et hydrate le contenu lorsque l'utilisateur interagit avec l'élément spécifié via des
événements `click` ou `keydown`.

```angular-html
@defer (hydrate on interaction) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

#### `hydrate on hover`

Le déclencheur `hydrate on hover` charge les dépendances de la vue différable et hydrate le contenu lorsque la souris a survolé la zone de déclenchement, via les
événements `mouseover` et `focusin`.

```angular-html
@defer (hydrate on hover) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

#### `hydrate on immediate`

Le déclencheur `hydrate on immediate` charge les dépendances de la vue différable et hydrate immédiatement le contenu. Cela signifie que le bloc différé se charge dès
que tout le reste du contenu non différé a fini son rendu.

```angular-html
@defer (hydrate on immediate) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

#### `hydrate on timer`

Le déclencheur `hydrate on timer` charge les dépendances de la vue différable et hydrate le contenu après une durée spécifiée.

```angular-html
@defer (hydrate on timer(500ms)) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

Le paramètre de durée doit être spécifié en millisecondes (`ms`) ou en secondes (`s`).

### `hydrate when`

Le déclencheur `hydrate when` accepte une expression conditionnelle personnalisée, charge les dépendances de la vue différable et hydrate le contenu lorsque la
condition devient truthy.

```angular-html
@defer (hydrate when condition) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

NOTE: Les conditions `hydrate when` ne se déclenchent que lorsqu'elles portent sur le bloc `@defer` déshydraté le plus haut dans l'arbre. La condition fournie au déclencheur est
définie dans le composant parent, qui doit exister avant de pouvoir être déclenché. Si le bloc parent est déshydraté, Angular ne peut pas encore
résoudre cette expression.

### `hydrate never`

`hydrate never` permet aux utilisateurs d'indiquer que le contenu du bloc defer doit rester déshydraté indéfiniment, devenant de fait du contenu
statique. Notez que cela ne s'applique qu'au rendu initial. Lors d'un rendu côté client ultérieur, un bloc `@defer` avec `hydrate never`
récupérera quand même ses dépendances, car l'hydratation ne s'applique qu'au chargement initial du contenu rendu côté serveur. Dans l'exemple ci-dessous, les rendus
côté client ultérieurs chargeront les dépendances du bloc `@defer` à l'entrée dans le viewport.

```angular-html
@defer (on viewport; hydrate never) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

NOTE: Utiliser `hydrate never` empêche l'hydratation de tout le sous-arbre imbriqué d'un bloc `@defer` donné. Aucun autre déclencheur `hydrate` ne se déclenche pour le contenu imbriqué sous ce bloc.

## Déclencheurs d'hydratation et déclencheurs habituels {#hydrate-triggers-alongside-regular-triggers}

Les déclencheurs d'hydratation sont des déclencheurs supplémentaires, utilisés en complément des déclencheurs habituels d'un bloc `@defer`. L'hydratation est une optimisation du chargement initial, ce qui signifie que les déclencheurs d'hydratation ne s'appliquent qu'à ce chargement initial. Tout rendu côté client ultérieur utilise toujours le déclencheur habituel.

```angular-html
@defer (on idle; hydrate on interaction) {
  <example-cmp />
} @placeholder {
  <div>Example Placeholder</div>
}
```

Dans cet exemple, lors du chargement initial, c'est `hydrate on interaction` qui s'applique. L'hydratation est déclenchée lors d'une interaction avec le composant `<example-cmp />`. Lors de tout chargement de page ultérieur rendu côté client, par exemple lorsqu'un utilisateur clique sur un routerLink qui charge une page contenant ce composant, c'est `on idle` qui s'applique.

## Comment l'hydratation incrémentale fonctionne-t-elle avec des blocs `@defer` imbriqués ? {#how-does-incremental-hydration-work-with-nested-defer-blocks}

Le système de composants et de dépendances d'Angular est hiérarchique, ce qui signifie que l'hydratation d'un composant nécessite que tous ses parents soient aussi hydratés. Ainsi, si l'hydratation est déclenchée pour un bloc `@defer` enfant au sein d'un ensemble de blocs `@defer` déshydratés imbriqués, l'hydratation est déclenchée depuis le bloc `@defer` déshydraté le plus haut jusqu'à l'enfant déclenché, dans cet ordre.

```angular-html
@defer (hydrate on interaction) {
  <parent-block-cmp />
  @defer (hydrate on hover) {
    <child-block-cmp />
  } @placeholder {
    <div>Child placeholder</div>
  }
} @placeholder {
  <div>Parent Placeholder</div>
}
```

Dans l'exemple ci-dessus, survoler le bloc `@defer` imbriqué déclenche l'hydratation. Le bloc `@defer` parent contenant `<parent-block-cmp />` est hydraté en premier, puis le bloc `@defer` enfant contenant `<child-block-cmp />` est hydraté ensuite.

## Contraintes {#constraints}

L'hydratation incrémentale a les mêmes contraintes que l'hydratation complète de l'application, notamment les limites sur la manipulation directe du DOM et l'exigence d'une structure HTML valide. Consultez la section [Contraintes du guide de l'hydratation](guide/hydration#constraints) pour plus de détails.

## Faut-il toujours spécifier des blocs `@placeholder` ? {#do-i-still-need-to-specify-placeholder-blocks}

Oui. Le contenu du bloc `@placeholder` n'est pas utilisé pour l'hydratation incrémentale, mais un `@placeholder` reste nécessaire pour les rendus côté client ultérieurs. Si votre contenu ne se trouvait pas sur la route faisant partie du chargement initial, toute navigation vers la route qui contient votre bloc `@defer` effectue le rendu comme un bloc `@defer` classique. Le `@placeholder` est donc rendu dans ces cas de rendu côté client.
