<!-- source: ed0f8e65958c6e08f86e4428593fc195d73a259f -->
# Récupération de données avec les resources

IMPORTANT: Les resources de route sont en [developer preview](reference/releases#developer-preview).

Le router d'Angular s'intègre aux signals d'Angular grâce à l'option de configuration de route `resources`. Vous pouvez ainsi récupérer des données de manière réactive à l'aide des API `Resource`.

## Pourquoi utiliser les resources de route ? {#why-use-route-resources}

Les resources de route offrent plusieurs avantages par rapport aux [resolvers de données](/guide/routing/data-resolvers) traditionnels :

- **Exécution parallèle** : les resources de route de toutes les routes correspondantes se chargent simultanément au lieu d'une route à la fois.
- **Chargement de données non bloquant** : utilisez `nonBlocking()` pour activer la route immédiatement et afficher des skeletons de chargement ou des états d'interface pendant que les données se chargent en arrière-plan.
- **Rechargement sans nouvelle navigation** : appelez `.reload()` sur des resources individuelles ou mettez à jour des paramètres sous forme de signals pour rafraîchir les données sans réexécuter les guards ni refaire la correspondance des routes.
- **Récupération de données réactive** : les resources s'intègrent directement aux signals d'Angular, se réévaluent automatiquement lorsque leurs dépendances de signals changent et exposent des signals de statut réactifs comme `isLoading()` et `error()`.

## Activer les resources de route {#enabling-route-resources}

Pour activer les resources de route, fournissez `withRouterResources()` à la configuration de votre router :

```ts
import {provideRouter, withComponentInputBinding, withRouterResources} from '@angular/router';
import {bootstrapApplication} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideRouter(routes, withComponentInputBinding(), withRouterResources())],
});
```

TIP: Activez `withComponentInputBinding()` pour que le router puisse lier les resources résolues directement aux inputs du composant.

## Définir des resources de route {#defining-route-resources}

Vous définissez des resources sur une route avec la fonction `resources`. Cette fonction s'exécute dans un contexte d'injection : vous pouvez donc utiliser `inject()` pour accéder directement à des services, à des clients d'API ou à des stores au sein de la définition de la route.

```angular-ts
import {Component, inject, input, resource} from '@angular/core';
import {Routes} from '@angular/router';
import {UserService} from './user.service';

const routes: Routes = [
  {
    path: 'user/:id',
    component: UserProfile,
    resources: (ctx) => {
      const userService = inject(UserService);
      return {
        user: resource({
          params: () => ctx.params()['id'],
          loader: ({params: id}) => userService.getUser(id),
        }),
      };
    },
  },
];

@Component({
  template: `<p>User: {{ user().name }}</p>`,
})
export class UserProfile {
  // The router binds only the value for blocking resources.
  user = input.required<User>();
}
```

### L'objet `ResourceContext` {#the-resourcecontext-object}

La fonction `resources` reçoit un `ResourceContext` qui donne accès aux signals réactifs de la route : `params`, `queryParams`, `fragment` et `data`.

### Implémentations de resource prises en charge {#supported-resource-implementations}

La fonction `resources` peut renvoyer n'importe quelle implémentation de `Resource` d'Angular, comme `resource()`, `rxResource()` ou une resource personnalisée.

```ts
import {Routes} from '@angular/router';
import {rxResource} from '@angular/core/rxjs-interop';

const routes: Routes = [
  {
    path: 'user/:id',
    component: UserProfile,
    resources: (ctx) => ({
      user: rxResource({
        params: () => ctx.params()['id'],
        stream: ({params: id}) => fetchUserObservable(id),
      }),
    }),
  },
];
```

NOTE: `rxResource` utilise la propriété `stream` au lieu de `loader` pour accepter une fonction qui renvoie un observable.

La fonction `resources` peut aussi être `async` et renvoyer une `Promise` si vous devez effectuer une initialisation asynchrone ou des imports dynamiques avant de configurer les resources :

```ts
resources: async (ctx) => {
  const injector = inject(Injector);
  const {fetchUserData} = await import('./user-api');
  return {
    user: resource({
      params: () => ctx.params()['id'],
      loader: ({params: id}) => fetchUserData(id),
      injector,
    }),
  };
},
```

## Suivi fin des changements avec les signals {#fine-grained-change-tracking-with-signals}

Une resource suit les signals que lit sa fonction `params`. Lisez exactement la valeur dont vous avez besoin pour que la resource ne récupère à nouveau les données que lorsque cette valeur change :

```ts
resources: (ctx) => ({
  products: resource({
    // Tracks only the 'category' query parameter
    params: () => ctx.queryParams()['category'],
    loader: ({params: category}) => fetchProducts(category),
  }),
}),
```

Une navigation qui modifie un query parameter sans rapport, comme `?sort=desc` ou `?page=2`, laisse `category` inchangé : la resource ne récupère donc pas à nouveau les données.

TIP: Lisez des propriétés précises, comme `ctx.params()['id']`, au lieu de renvoyer un objet de paramètres entier, comme `ctx.params()`. Le router crée un nouvel objet à chaque navigation : renvoyer l'objet entier relance donc la récupération de la resource même lorsque les valeurs individuelles n'ont pas changé.

## Exécution parallèle {#parallel-execution}

Les resolvers de données s'exécutent séquentiellement, de la route parente à la route enfant. Si le resolver d'une route parente prend 200 ms et celui d'une route enfant 300 ms, la navigation est bloquée pendant 500 ms.

Les resources de route de toute la hiérarchie de routes correspondante s'exécutent simultanément : la même navigation se termine donc en 300 ms, le temps de la resource la plus lente.

## Resources bloquantes et non bloquantes {#blocking-and-non-blocking-resources}

Par défaut, chaque resource renvoyée par `resources` est bloquante : le router attend que les données soient entièrement chargées avant d'activer la route et le composant.

Pour une resource bloquante, le router lie la valeur résolue à l'input du composant : le type de l'input est donc `T` et non `Resource<T>`. Le composant n'observe jamais d'état `loading`, car le router bloque la navigation jusqu'au chargement de la resource, et il n'observe jamais d'état `error`, car le router annule la navigation lorsque la resource échoue.

Pour gérer plutôt les états de chargement dans l'interface, enveloppez la resource dans `nonBlocking()`. Le router active alors le composant immédiatement et lie l'objet `Resource<T>` complet à l'input du composant, ce qui vous donne accès à `isLoading()`, `error()` et aux autres signals de la resource.

```angular-ts
import {Component, input, Resource, resource} from '@angular/core';
import {Routes, nonBlocking} from '@angular/router';

const routes: Routes = [
  {
    path: 'reports',
    component: Reports,
    resources: () => ({
      reportData: nonBlocking(
        resource({
          loader: () => fetchHeavyReportData(),
        }),
      ),
    }),
  },
];

@Component({
  template: `
    @if (reportData().isLoading()) {
      <p>Loading...</p>
    } @else if (reportData().error()) {
      <p>Error loading report.</p>
    } @else if (reportData().hasValue()) {
      <report-view [data]="reportData().value()" />
    }
  `,
})
export class Reports {
  reportData = input.required<Resource<ReportData>>();
}
```

NOTE: Si une resource bloquante échoue, le router annule la navigation et émet un événement `NavigationError`. Une resource enveloppée dans `nonBlocking()` termine la navigation et expose l'échec via son signal `error()`.

### Rediriger depuis une resource {#redirecting-from-a-resource}

Si une resource bloquante doit rediriger l'utilisateur (par exemple, si un élément est introuvable), levez une `RedirectCommand` dans le loader de la resource. Le router annule la navigation en cours et redirige vers l'URL indiquée :

```ts
import {inject, resource} from '@angular/core';
import {RedirectCommand, Router, Routes} from '@angular/router';

const routes: Routes = [
  {
    path: 'user/:id',
    component: UserProfile,
    resources: (ctx) => {
      const router = inject(Router);

      return {
        user: resource({
          params: () => ctx.params()['id'],
          loader: async ({params: id}) => {
            const user = await fetchUser(id);
            if (!user) {
              throw new RedirectCommand(router.parseUrl('/not-found'));
            }
            return user;
          },
        }),
      };
    },
  },
];
```

## Recharger les resources sans nouvelle navigation {#reloading-resources-without-renavigation}

Avec les resolvers de données, récupérer à nouveau les données nécessite une navigation de route (par exemple, une navigation avec `onSameUrlNavigation: 'reload'`), qui refait la correspondance des routes et réexécute les guards et les resolvers.

Les resources de route permettent de rafraîchir les données sur place de deux façons :

1. **Rechargement programmatique** : appelez `.reload()` sur l'instance de `Resource`.
2. **Rechargement réactif** : mettez à jour un signal que lit la fonction `params` de la resource, comme un filtre de l'application ou un signal d'état, ce qui réexécute le loader.

Comme le router ne lie que la valeur d'une resource bloquante à l'input d'un composant, lisez l'instance de `Resource` depuis `ActivatedRoute` ou `ActivatedRouteSnapshot` lorsque vous devez appeler `.reload()` ou inspecter les signals de statut :

```angular-ts
import {Component, inject, input} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  template: `
    <p>User: {{ user().name }}</p>
    <button (click)="refreshUser()">Refresh</button>
  `,
})
export class UserProfile {
  user = input.required<User>();
  private userResource = inject(ActivatedRoute).resources?.['user'];

  refreshUser() {
    // Reloads only this specific resource without renavigating the route
    this.userResource?.reload();
  }
}
```

## États transitoires pendant les navigations en attente {#transitional-states-during-pending-navigations}

Tant qu'une navigation est en attente, le router fige les resources qu'il expose sur `ActivatedRoute`, ce qui masque les états intermédiaires `loading` et `reloading`.

Si vous naviguez de `/user/1` vers `/user/2` et que le router réutilise le composant `UserProfile`, le composant continue d'afficher les données de `/user/1` jusqu'à ce que `/user/2` soit résolu. Le router libère alors les resources et l'interface passe directement aux nouvelles données, sans flash de chargement.

Le router expose ces resources en lecture seule, même lorsque la fonction `resources` renvoie une resource modifiable comme `resource()`. Vous pouvez lire les signals de la resource et appeler `reload()`, mais pas `set()` ni `update()`. Un appel à `reload()` pendant une navigation active ou pendant la récupération après un rollback renvoie `false`, afin de ne pas interrompre le suivi des transitions par le router.

### Récupération après un rollback lors d'une annulation {#rollback-recovery-on-cancellation}

Si une navigation est annulée (par exemple, par une guard), le router rétablit l'arbre d'état dans son état précédent. Ce retour en arrière peut faire revenir les dépendances de signals de la resource, comme les paramètres de route, à leurs valeurs précédentes.

Comme les paramètres sont revenus à leurs anciennes valeurs, la resource peut déclencher automatiquement un nouveau chargement pour récupérer les données correspondant aux anciens paramètres. Pour éviter l'affichage furtif d'un état de chargement pour des données déjà visibles, le router conserve dans l'interface le snapshot précédent de la resource jusqu'à ce que celle-ci se soit stabilisée dans l'état rétabli.

TIP: Transmettez l'`abortSignal` fourni par le loader de la resource à vos appels asynchrones (comme `fetch`). Lorsque le router annule le changement de paramètres ou qu'une navigation en remplace une autre, la requête en attente est proprement interrompue : `loader: ({params: id, abortSignal}) => fetchUser(id, {signal: abortSignal})`.
