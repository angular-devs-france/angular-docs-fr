<!-- source: 711cb25bff7a38fcb03f5919c29c8be42a75b997 -->
# Chargement différé avec `@defer`

Les vues différables, aussi appelées blocs `@defer`, réduisent la taille du bundle initial de votre application en différant le chargement du code qui n'est pas strictement nécessaire au rendu initial d'une page. Cela se traduit souvent par un chargement initial plus rapide et une amélioration des Core Web Vitals (CWV), principalement le Largest Contentful Paint (LCP) et le Time to First Byte (TTFB).

Pour utiliser cette fonctionnalité, vous pouvez envelopper de manière déclarative une section de votre template dans un bloc @defer :

```angular-html
@defer {
  <large-component />
}
```

Le code de tous les composants, directives et pipes présents dans le bloc `@defer` est placé dans un fichier JavaScript séparé et chargé uniquement lorsque c'est nécessaire, après le rendu du reste du template.

Les vues différables prennent en charge divers déclencheurs, des options de préchargement et des sous-blocs pour gérer les états de placeholder, de chargement et d'erreur.

## Quelles dépendances sont différées ? {#which-dependencies-are-deferred}

Les composants, les directives, les pipes et les styles CSS des composants peuvent être différés lors du chargement d'une application.

Pour que les dépendances d'un bloc `@defer` soient différées, elles doivent remplir deux conditions :

1. **Elles doivent être standalone.** Les dépendances non standalone ne peuvent pas être différées et sont toujours chargées immédiatement, même si elles se trouvent dans des blocs `@defer`.
1. **Elles ne peuvent pas être référencées en dehors des blocs `@defer` dans le même fichier.** Si elles sont référencées en dehors du bloc `@defer` ou dans des queries ViewChild, les dépendances seront chargées immédiatement.

Les dépendances _transitives_ des composants, directives et pipes utilisés dans le bloc `@defer` n'ont pas besoin d'être strictement standalone : les dépendances transitives peuvent toujours être déclarées dans un `NgModule` et participer au chargement différé.

Le compilateur d'Angular produit une instruction d'[import dynamique](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) pour chaque composant, directive et pipe utilisé dans le bloc `@defer`. Le contenu principal du bloc est rendu une fois que tous les imports sont résolus. Angular ne garantit aucun ordre particulier pour ces imports.

## Gérer les différentes étapes du chargement différé {#how-to-manage-different-stages-of-deferred-loading}

Les blocs `@defer` disposent de plusieurs sous-blocs qui vous permettent de gérer élégamment les différentes étapes du processus de chargement différé.

### `@defer`

Il s'agit du bloc principal, qui définit la section de contenu chargée de manière différée. Il n'est pas rendu initialement : le contenu différé est chargé et rendu dès que le [déclencheur](#controlling-deferred-content-loading-with-triggers) spécifié se produit ou que la condition `when` est remplie.

Par défaut, un bloc `@defer` est déclenché lorsque le navigateur passe à l'état [inactif](/guide/templates/defer#idle).

```angular-html
@defer {
  <large-component />
}
```

### Afficher le contenu du placeholder avec `@placeholder` {#show-placeholder-content-with-placeholder}

Par défaut, les blocs defer n'affichent aucun contenu avant d'être déclenchés.

Le bloc `@placeholder` est un bloc optionnel qui déclare le contenu à afficher avant que le bloc `@defer` ne soit déclenché.

```angular-html
@defer {
  <large-component />
} @placeholder {
  <p>Placeholder content</p>
}
```

Bien qu'il soit optionnel, certains déclencheurs peuvent nécessiter la présence d'un `@placeholder` ou d'une [variable de référence de template](/guide/templates/variables#template-reference-variables) pour fonctionner. Consultez la section [Déclencheurs](#controlling-deferred-content-loading-with-triggers) pour plus de détails.

Angular remplace le contenu du placeholder par le contenu principal une fois le chargement terminé. Vous pouvez utiliser n'importe quel contenu dans la section placeholder, y compris du HTML simple, des composants, des directives et des pipes. Gardez à l'esprit que _les dépendances du bloc placeholder sont chargées immédiatement_.

Le bloc `@placeholder` accepte un paramètre optionnel pour spécifier la durée `minimum` pendant laquelle ce placeholder doit être affiché après le rendu initial de son contenu.

```angular-html
@defer {
  <large-component />
} @placeholder (minimum 500ms) {
  <p>Placeholder content</p>
}
```

Ce paramètre `minimum` s'exprime en millisecondes (ms) ou en secondes (s). Vous pouvez l'utiliser pour éviter un clignotement rapide du contenu du placeholder lorsque les dépendances différées sont récupérées rapidement.

### Afficher un contenu de chargement avec `@loading` {#show-loading-content-with-loading}

Le bloc `@loading` est un bloc optionnel qui vous permet de déclarer le contenu affiché pendant le chargement des dépendances différées. Il remplace le bloc `@placeholder` dès que le chargement est déclenché.

```angular-html
@defer {
  <large-component />
} @loading {
  <img alt="loading..." src="loading.gif" />
} @placeholder {
  <p>Placeholder content</p>
}
```

Ses dépendances sont chargées immédiatement (comme pour `@placeholder`).

Le bloc `@loading` accepte deux paramètres optionnels pour aider à éviter le clignotement rapide du contenu qui peut se produire lorsque les dépendances différées sont récupérées rapidement :

- `minimum` - la durée minimale pendant laquelle ce placeholder doit être affiché
- `after` - le délai à attendre après le début du chargement avant d'afficher le template de chargement

```angular-html
@defer {
  <large-component />
} @loading (after 100ms; minimum 1s) {
  <img alt="loading..." src="loading.gif" />
}
```

Les deux paramètres s'expriment en millisecondes (ms) ou en secondes (s). De plus, les minuteurs des deux paramètres démarrent immédiatement après le déclenchement du chargement.

### Afficher un état d'erreur en cas d'échec du chargement différé avec `@error` {#show-error-state-when-deferred-loading-fails-with-error}

Le bloc `@error` est un bloc optionnel qui s'affiche si le chargement différé échoue. Comme pour `@placeholder` et `@loading`, les dépendances du bloc @error sont chargées immédiatement.

```angular-html
@defer {
  <large-component />
} @error {
  <p>Failed to load large component.</p>
}
```

## Contrôler le chargement du contenu différé avec des déclencheurs {#controlling-deferred-content-loading-with-triggers}

Vous pouvez spécifier des **déclencheurs** qui contrôlent le moment où Angular charge et affiche le contenu différé.

Lorsqu'un bloc `@defer` est déclenché, il remplace le contenu du placeholder par le contenu chargé de manière différée.

Plusieurs déclencheurs d'événements peuvent être définis en les séparant par un point-virgule, `;`, et seront évalués comme des conditions OU.

Il existe deux types de déclencheurs : `on` et `when`.

### `on`

`on` spécifie une condition qui détermine quand le bloc `@defer` est déclenché.

Les déclencheurs disponibles sont les suivants :

| Déclencheur                   | Description                                                            |
| ----------------------------- | ---------------------------------------------------------------------- |
| [`idle`](#idle)               | Se déclenche lorsque le navigateur est inactif. Accepte un délai d'expiration optionnel. |
| [`viewport`](#viewport)       | Se déclenche lorsque le contenu spécifié entre dans le viewport        |
| [`interaction`](#interaction) | Se déclenche lorsque l'utilisateur interagit avec l'élément spécifié  |
| [`hover`](#hover)             | Se déclenche lorsque la souris survole la zone spécifiée               |
| [`immediate`](#immediate)     | Se déclenche immédiatement après la fin du rendu du contenu non différé |
| [`timer`](#timer)             | Se déclenche après une durée donnée                                    |

#### `idle`

Le déclencheur `idle` charge le contenu différé dès que le navigateur a atteint un état d'inactivité, en s'appuyant sur requestIdleCallback. C'est le comportement par défaut d'un bloc defer.

Vous pouvez éventuellement spécifier un délai d'expiration en millisecondes, transmis à [`requestIdleCallback`](https://developer.mozilla.org/docs/Web/API/Window/requestIdleCallback). Si le navigateur ne planifie pas le callback assez tôt, le travail s'exécutera au plus tard à l'expiration du délai spécifié.

```angular-html
<!-- @defer (on idle) -->
@defer {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}

<!-- With a 500ms timeout -->
@defer (on idle(500)) {
  <large-cmp />
}
```

##### Personnaliser le comportement de `idle` {#customizing-idle-behavior}

Vous pouvez personnaliser le déclencheur `idle` en fournissant votre propre implémentation de `IdleService` et en l'enregistrant avec `provideIdleServiceWith` dans les providers de votre application.

```ts
@Service()
class CustomIdleService implements IdleService {
  requestOnIdle(callback: (deadline?: IdleDeadline) => void, options?: IdleRequestOptions): number {
    // Custom idle scheduling logic can be implemented here.
    return requestIdleCallback(callback, options);
  }

  cancelOnIdle(id: number) {
    // Implement custom idle cancellation here.
  }
}

bootstrapApplication(App, {
  providers: [provideIdleServiceWith(CustomIdleService)],
});
```

#### `viewport`

Le déclencheur `viewport` charge le contenu différé lorsque le contenu spécifié entre dans le viewport, à l'aide de l'[API Intersection Observer](https://developer.mozilla.org/docs/Web/API/Intersection_Observer_API). Le contenu observé peut être le contenu du `@placeholder` ou une référence explicite à un élément.

Par défaut, le bloc `@defer` surveille l'entrée du placeholder dans le viewport. Les placeholders utilisés de cette manière doivent avoir un unique élément racine.

```angular-html
@defer (on viewport) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

Vous pouvez aussi spécifier une [variable de référence de template](/guide/templates/variables) dans le même template que le bloc `@defer` comme élément dont l'entrée dans le viewport est surveillée. Cette variable est transmise en paramètre du déclencheur viewport.

```angular-html
<div #greeting>Hello!</div>
@defer (on viewport(greeting)) {
  <greetings-cmp />
}
```

Si vous souhaitez personnaliser les options de l'`IntersectionObserver`, le déclencheur `viewport` accepte un objet littéral. Celui-ci prend en charge toutes les propriétés du second paramètre d'`IntersectionObserver`, à l'exception de `root`. Lorsque vous utilisez la notation objet littéral, vous devez transmettre votre déclencheur via la propriété `trigger`.

```angular-html
<div #greeting>Hello!</div>

<!-- With options and a trigger -->
@defer (on viewport({trigger: greeting, rootMargin: '100px', threshold: 0.5})) {
  <greetings-cmp />
}

<!-- With options and an implied trigger -->
@defer (on viewport({rootMargin: '100px', threshold: 0.5})) {
  <greetings-cmp />
} @placeholder {
  <div>Implied trigger</div>
}
```

#### `interaction`

Le déclencheur `interaction` charge le contenu différé lorsque l'utilisateur interagit avec l'élément spécifié via des événements `click` ou `keydown`.

Par défaut, le placeholder sert d'élément d'interaction. Les placeholders utilisés de cette manière doivent avoir un unique élément racine.

```angular-html
@defer (on interaction) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

Vous pouvez aussi spécifier une [variable de référence de template](/guide/templates/variables) dans le même template que le bloc `@defer` comme élément dont les interactions sont surveillées. Cette variable est transmise en paramètre du déclencheur interaction.

```angular-html
<div #greeting>Hello!</div>
@defer (on interaction(greeting)) {
  <greetings-cmp />
}
```

#### `hover`

Le déclencheur `hover` charge le contenu différé lorsque la souris a survolé la zone de déclenchement, via les événements `mouseover` et `focusin`.

Par défaut, le placeholder sert d'élément d'interaction. Les placeholders utilisés de cette manière doivent avoir un unique élément racine.

```angular-html
@defer (on hover) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

Vous pouvez aussi spécifier une [variable de référence de template](/guide/templates/variables) dans le même template que le bloc `@defer` comme élément survolé. Cette variable est transmise en paramètre du déclencheur hover.

```angular-html
<div #greeting>Hello!</div>
@defer (on hover(greeting)) {
  <greetings-cmp />
}
```

#### `immediate`

Le déclencheur `immediate` charge immédiatement le contenu différé. Cela signifie que le bloc différé se charge dès que tout le reste du contenu non différé a fini son rendu.

```angular-html
@defer (on immediate) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

#### `timer`

Le déclencheur `timer` charge le contenu différé après une durée spécifiée.

```angular-html
@defer (on timer(500ms)) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

Le paramètre de durée doit être spécifié en millisecondes (`ms`) ou en secondes (`s`).

### `when`

Le déclencheur `when` accepte une expression conditionnelle personnalisée et charge le contenu différé lorsque la condition devient truthy.

```angular-html
@defer (when condition) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

Il s'agit d'une opération ponctuelle : le bloc `@defer` ne revient pas au placeholder si la condition repasse à une valeur falsy après être devenue truthy.

## Précharger des données avec `prefetch` {#prefetching-data-with-prefetch}

En plus de spécifier une condition qui détermine quand le contenu différé est affiché, vous pouvez éventuellement spécifier un **déclencheur de préchargement**. Ce déclencheur vous permet de charger le JavaScript associé au bloc `@defer` avant que le contenu différé ne soit affiché.

Le préchargement permet des comportements plus avancés, comme commencer à précharger des ressources avant qu'un utilisateur n'ait réellement vu un bloc defer ou interagi avec lui, alors qu'il pourrait bientôt le faire, ce qui rend les ressources disponibles plus rapidement.

Vous pouvez spécifier un déclencheur de préchargement de la même manière que le déclencheur principal du bloc, mais en le préfixant du mot-clé `prefetch`. Le déclencheur principal du bloc et le déclencheur de préchargement sont séparés par un point-virgule (`;`).

Dans l'exemple ci-dessous, le préchargement démarre lorsque le navigateur devient inactif, et le contenu du bloc n'est rendu qu'une fois que l'utilisateur interagit avec le placeholder.

```angular-html
@defer (on interaction; prefetch on idle) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}

<!-- Prefetching with a 500ms idle timeout -->
@defer (on interaction; prefetch on idle(500)) {
  <large-cmp />
}
```

## Tester les blocs `@defer` {#testing-defer-blocks}

Angular fournit des API TestBed pour simplifier le test des blocs `@defer` et le déclenchement de leurs différents états pendant les tests. Par défaut, les blocs `@defer` se déroulent dans les tests comme ils le feraient dans une application réelle. Si vous voulez parcourir les états manuellement, vous pouvez passer le comportement des blocs defer à `Manual` dans la configuration de TestBed.

```angular-ts
it('should render a defer block in different states', async () => {
  // configures the defer block behavior to start in "paused" state for manual control.
  TestBed.configureTestingModule({deferBlockBehavior: DeferBlockBehavior.Manual});
  @Component({
    // ...
    template: `
      @defer {
        <large-component />
      } @placeholder {
        Placeholder
      } @loading {
        Loading...
      }
    `,
  })
  class ExampleA {}
  // Create component fixture.
  const componentFixture = TestBed.createComponent(ExampleA);
  // Retrieve the list of all defer block fixtures and get the first block.
  const deferBlockFixture = (await componentFixture.getDeferBlocks())[0];
  // Renders placeholder state by default.
  expect(componentFixture.nativeElement.innerHTML).toContain('Placeholder');
  // Render loading state and verify rendered output.
  await deferBlockFixture.render(DeferBlockState.Loading);
  expect(componentFixture.nativeElement.innerHTML).toContain('Loading');
  // Render final state and verify the output.
  await deferBlockFixture.render(DeferBlockState.Complete);
  expect(componentFixture.nativeElement.innerHTML).toContain('large works!');
});
```

## `@defer` fonctionne-t-il avec `NgModule` ? {#does-defer-work-with-ngmodule}

Les blocs `@defer` sont compatibles à la fois avec les composants, directives et pipes standalone et avec ceux basés sur NgModule. Cependant, **seuls les composants, directives et pipes standalone peuvent être différés**. Les dépendances basées sur NgModule ne sont pas différées et sont incluses dans le bundle chargé immédiatement.

## Compatibilité entre les blocs `@defer` et le Hot Module Reload (HMR) {#compatibility-between-defer-blocks-and-hot-module-reload-hmr}

Lorsque le Hot Module Replacement (HMR) est actif, tous les chunks des blocs `@defer` sont récupérés immédiatement, sans tenir compte des déclencheurs configurés. Pour rétablir le comportement standard des déclencheurs, vous devez désactiver le HMR en servant votre application avec l'option `--no-hmr`.

## Comment `@defer` fonctionne-t-il avec le rendu côté serveur (SSR) et la génération de site statique (SSG) ? {#how-does-defer-work-with-server-side-rendering-ssr-and-static-site-generation-ssg}

Par défaut, lors du rendu d'une application sur le serveur (en SSR ou en SSG), les blocs defer rendent toujours leur `@placeholder` (ou rien si aucun placeholder n'est spécifié) et les déclencheurs ne sont pas invoqués. Côté client, le contenu du `@placeholder` est hydraté et les déclencheurs sont activés.

Pour rendre le contenu principal des blocs `@defer` sur le serveur (en SSR comme en SSG), vous pouvez activer [la fonctionnalité d'hydratation incrémentale](/guide/incremental-hydration) et configurer des déclencheurs `hydrate` pour les blocs concernés.

## Fichiers barrel et chunks lazy {#barrel-files-and-lazy-chunks}

Si vous utilisez `@defer` mais ne voyez pas de chunk lazy séparé dans la sortie de votre build, vérifiez la façon dont vous importez le composant différé. L'import via un fichier barrel (`index.ts`) est un coupable fréquent : les bundlers considèrent le barrel comme un module unique et conservent tous ses exports ensemble, si bien que votre composant se retrouve dans le bundle principal, indépendamment de `@defer`.

```typescript {header: "index.ts"}
export {HeavyComponent} from './heavy.component';
export {OtherComponent} from './other.component';
```

```typescript {header: "parent.component.ts"}
import {Component} from '@angular/core';
import {HeavyComponent} from './index'; // pulls in OtherComponent too

@Component({
  imports: [HeavyComponent],
  template: `@defer {
    <heavy-component />
  }`,
})
export class ParentComponent {}
```

La solution est simple : importez directement depuis le fichier propre au composant :

```typescript
import {HeavyComponent} from './heavy.component';
```

Cela suffit pour que le bundler le place dans son propre chunk et le charge en lazy loading lorsque le déclencheur se déclenche.

## Bonnes pratiques pour différer des vues {#best-practices-for-deferring-views}

### Évitez les chargements en cascade avec des blocs `@defer` imbriqués {#avoid-cascading-loads-with-nested-defer-blocks}

Lorsque vous avez des blocs `@defer` imbriqués, ils doivent avoir des déclencheurs différents afin d'éviter de se charger simultanément, ce qui provoquerait des requêtes en cascade et pourrait nuire aux performances de chargement de la page.

### Évitez les décalages de mise en page {#avoid-layout-shifts}

Évitez de différer des composants visibles dans le viewport de l'utilisateur lors du chargement initial. Cela peut nuire aux Core Web Vitals en augmentant le cumulative layout shift (CLS).

Si cela s'avère nécessaire, évitez les déclencheurs `immediate`, `timer`, `viewport` et `when` personnalisés qui provoquent le chargement du contenu pendant le rendu initial de la page.

### Pensez à l'accessibilité {#keep-accessibility-in-mind}

Lorsque vous utilisez des blocs `@defer`, tenez compte de l'impact sur les utilisateurs de technologies d'assistance comme les lecteurs d'écran.
Les lecteurs d'écran qui se positionnent sur une section différée liront d'abord le contenu du placeholder ou de chargement, mais peuvent ne pas annoncer les changements lorsque le contenu différé se charge.

Pour que les changements du contenu différé soient annoncés aux lecteurs d'écran, vous pouvez envelopper votre bloc `@defer` dans un élément doté d'une région live :

```angular-html
<div aria-live="polite" aria-atomic="true">
  @defer (on timer(2000)) {
    <user-profile [user]="currentUser" />
  } @placeholder {
    Loading user profile...
  } @loading {
    Please wait...
  } @error {
    Failed to load profile
  }
</div>
```

Ainsi, les changements sont annoncés à l'utilisateur lors des transitions (placeholder → chargement → contenu/erreur).
