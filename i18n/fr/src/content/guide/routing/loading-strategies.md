<!-- source: dfdd13c7348141f6c3029fd3c1b35f765da3dc6c -->
# Stratégies de chargement des routes

Comprendre comment et quand les routes et les composants se chargent dans le routing Angular est essentiel pour construire des applications web réactives. Angular propose deux stratégies principales pour contrôler le comportement de chargement :

1. **Chargement immédiat (eager)** : les routes et les composants sont chargés immédiatement
2. **Lazy loading** : les routes et les composants ne sont chargés qu'en cas de besoin

Chaque approche offre des avantages distincts selon les scénarios.

## Composants chargés immédiatement {#eagerly-loaded-components}

Lorsque vous définissez une route avec la propriété [`component`](api/router/Route#component), le composant référencé est chargé immédiatement, dans le même bundle JavaScript que la configuration de la route.

```ts
import {Routes} from '@angular/router';
import {HomePage} from './components/home/home-page';
import {LoginPage} from './components/auth/login-page';

export const routes: Routes = [
  // HomePage and LoginPage are both directly referenced in this config,
  // so their code is eagerly included in the same JavaScript bundle as this file.
  {
    path: '',
    component: HomePage,
  },
  {
    path: 'login',
    component: LoginPage,
  },
];
```

Charger immédiatement les composants de route de cette manière signifie que le navigateur doit télécharger et analyser tout le JavaScript de ces composants lors du chargement initial de la page, mais les composants sont disponibles pour Angular immédiatement.

Si inclure davantage de JavaScript dans le chargement initial de la page ralentit ce chargement initial, cela peut rendre les transitions plus fluides lorsque l'utilisateur navigue dans l'application.

## Composants et routes en lazy loading {#lazily-loaded-components-and-routes}

Vous pouvez utiliser la propriété [`loadComponent`](api/router/Route#loadComponent) pour charger en lazy loading le JavaScript d'un composant au moment où sa route devient active. La propriété [`loadChildren`](api/router/Route#loadChildren) charge en lazy loading les routes enfants pendant la correspondance des routes.

```ts
import {Routes} from '@angular/router';

export const routes: Routes = [
  {
    path: 'login',
    loadComponent: () => import('./components/auth/login-page'),
  },
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component'),
    loadChildren: () => import('./admin/admin.routes'),
  },
];
```

Les propriétés [`loadComponent`](/api/router/Route#loadComponent) et [`loadChildren`](/api/router/Route#loadChildren) acceptent une fonction loader qui renvoie une promise résolue respectivement en un composant Angular ou en un ensemble de routes. Dans la plupart des cas, cette fonction utilise l'[API standard d'import dynamique de JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import). Vous pouvez toutefois utiliser n'importe quelle fonction loader asynchrone.

Si le fichier chargé en lazy loading utilise un export `default`, vous pouvez renvoyer directement la promise de `import()` sans appel `.then` supplémentaire pour sélectionner la classe exportée.

Le lazy loading des routes peut améliorer considérablement la vitesse de chargement de votre application Angular en retirant de grandes portions de JavaScript du bundle initial. Ces portions de code sont compilées dans des « chunks » JavaScript séparés que le router ne demande que lorsque l'utilisateur visite la route correspondante.

## Lazy loading dans le contexte d'injection {#injection-context-lazy-loading}

Le router exécute [`loadComponent`](/api/router/Route#loadComponent) et [`loadChildren`](/api/router/Route#loadChildren) dans le **contexte d'injection de la route courante**, ce qui vous permet d'appeler [`inject`](/api/core/inject) dans ces fonctions loader pour accéder aux providers déclarés sur cette route, hérités des routes parentes par l'injection de dépendances hiérarchique, ou disponibles globalement. Cela permet un lazy loading qui tient compte du contexte.

```ts
import {Routes} from '@angular/router';
import {inject} from '@angular/core';
import {FeatureFlags} from './feature-flags';

export const routes: Routes = [
  {
    path: 'dashboard',
    // Runs inside the route's injection context
    loadComponent: () => {
      const flags = inject(FeatureFlags);
      return flags.isPremium
        ? import('./dashboard/premium-dashboard')
        : import('./dashboard/basic-dashboard');
    },
  },
];
```

## Faut-il utiliser une route chargée immédiatement ou en lazy loading ? {#should-i-use-an-eager-or-a-lazy-route}

De nombreux facteurs entrent en compte pour décider si une route doit être chargée immédiatement ou en lazy loading.

En général, le chargement immédiat est recommandé pour la ou les pages d'accueil principales, tandis que les autres pages sont chargées en lazy loading.

NOTE: Si les routes en lazy loading offrent l'avantage immédiat de réduire la quantité de données initiales demandées par l'utilisateur, elles ajoutent des requêtes de données ultérieures qui peuvent être indésirables. C'est particulièrement vrai avec un lazy loading imbriqué sur plusieurs niveaux, qui peut avoir un impact important sur les performances.

## Étapes suivantes {#next-steps}

Découvrez comment [afficher le contenu de vos routes avec les outlets](/guide/routing/show-routes-with-outlets).
