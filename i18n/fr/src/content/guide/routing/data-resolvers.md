<!-- source: 831dec66d59fb82829dc541f90e86faa5e2ba204 -->
# Resolvers de données

Les resolvers de données vous permettent de récupérer des données avant de naviguer vers une route, afin que vos composants reçoivent les données dont ils ont besoin avant leur rendu. Cela peut éviter de devoir gérer des états de chargement et améliorer l'expérience utilisateur en préchargeant les données essentielles.

TIP: Si votre application utilise les signals d'Angular, envisagez d'utiliser les [resources de route](/guide/routing/data-fetching-with-resources) pour récupérer des données de façon réactive avec les API `Resource` et une prise en charge intégrée du chargement de données non bloquant.

## Que sont les resolvers de données ? {#what-are-data-resolvers}

Un resolver de données est un service qui implémente la fonction `ResolveFn`. Il s'exécute avant l'activation d'une route et peut récupérer des données depuis des API, des bases de données ou d'autres sources. Les données résolues deviennent accessibles au composant via l'`ActivatedRoute`.

Les resolvers de données ont accès aux [services fournis au niveau de la route](guide/di/defining-dependency-providers#route-providers) ainsi qu'aux informations propres à la route via l'argument `route`.

## Pourquoi utiliser des resolvers de données ? {#why-use-data-resolvers}

Les resolvers de données répondent à des problématiques courantes du routing :

- **Éviter les états vides** : les composants reçoivent les données dès leur chargement
- **Une meilleure expérience utilisateur** : pas d'indicateur de chargement pour les données essentielles
- **La gestion des erreurs** : gérez les erreurs de récupération des données avant la navigation
- **La cohérence des données** : assurez-vous que les données requises sont disponibles avant le rendu, ce qui est important pour le SSR

## Créer un resolver {#creating-a-resolver}

Vous créez un resolver en écrivant une fonction du type `ResolveFn`.

Elle reçoit en paramètres l'`ActivatedRouteSnapshot` et le `RouterStateSnapshot`.

Voici un resolver qui récupère les informations de l'utilisateur avant le rendu d'une route, à l'aide de la fonction [`inject`](api/core/inject) :

```ts
import {inject} from '@angular/core';
import {UserStore, SettingsStore} from './user-store';
import type {ActivatedRouteSnapshot, ResolveFn, RouterStateSnapshot} from '@angular/router';
import type {User, Settings} from './types';

export const userResolver: ResolveFn<User> = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  const userStore = inject(UserStore);
  const userId = route.paramMap.get('id')!;
  return userStore.getUser(userId);
};

export const settingsResolver: ResolveFn<Settings> = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  const settingsStore = inject(SettingsStore);
  const userId = route.paramMap.get('id')!;
  return settingsStore.getUserSettings(userId);
};
```

## Configurer des routes avec des resolvers {#configuring-routes-with-resolvers}

Pour ajouter un ou plusieurs resolvers de données à une route, ajoutez-les sous la clé `resolve` de la configuration de la route. Le type `Routes` définit la structure des configurations de routes :

```ts
import {Routes} from '@angular/router';

export const routes: Routes = [
  {
    path: 'user/:id',
    component: UserDetail,
    resolve: {
      user: userResolver,
      settings: settingsResolver,
    },
  },
];
```

Pour en savoir plus, consultez la [configuration `resolve` dans la documentation de l'API](api/router/Route#resolve).

## Accéder aux données résolues dans les composants {#accessing-resolved-data-in-components}

### Utiliser ActivatedRoute {#using-activatedroute}

Vous pouvez accéder aux données résolues dans un composant en convertissant l'observable `data` de l'`ActivatedRoute` en signal à l'aide de la fonction `toSignal` :

```angular-ts
import {Component, inject, computed} from '@angular/core';
import {ActivatedRoute} from '@angular/router';
import {toSignal} from '@angular/core/rxjs-interop';
import type {User, Settings} from './types';

@Component({
  template: `
    <h1>{{ user().name }}</h1>
    <p>{{ user().email }}</p>
    <div>Theme: {{ settings().theme }}</div>
  `,
})
export class UserDetail {
  private route = inject(ActivatedRoute);
  private data = toSignal(this.route.data, {requireSync: true});
  user = computed(() => this.data().user as User);
  settings = computed(() => this.data().settings as Settings);
}
```

### Utiliser withComponentInputBinding {#using-withcomponentinputbinding}

Une autre façon d'accéder aux données résolues consiste à utiliser `withComponentInputBinding()` lorsque vous configurez votre router avec `provideRouter`. Les données résolues peuvent ainsi être passées directement comme inputs du composant :

```ts
import {bootstrapApplication} from '@angular/platform-browser';
import {provideRouter, withComponentInputBinding} from '@angular/router';
import {routes} from './app.routes';

bootstrapApplication(App, {
  providers: [provideRouter(routes, withComponentInputBinding())],
});
```

Avec cette configuration, vous pouvez définir dans votre composant des inputs qui correspondent aux clés des resolvers, à l'aide de la fonction `input`, et de `input.required` pour les inputs obligatoires :

```angular-ts
import {Component, input} from '@angular/core';
import type {User, Settings} from './types';

@Component({
  template: `
    <h1>{{ user().name }}</h1>
    <p>{{ user().email }}</p>
    <div>Theme: {{ settings().theme }}</div>
  `,
})
export class UserDetail {
  user = input.required<User>();
  settings = input.required<Settings>();
}
```

Cette approche offre une meilleure sûreté de typage et évite d'injecter `ActivatedRoute` uniquement pour accéder aux données résolues.

## Gestion des erreurs dans les resolvers {#error-handling-in-resolvers}

En cas d'échec de navigation, il est important de gérer proprement les erreurs dans vos resolvers de données. Sinon, une `NavigationError` se produira et la navigation vers la route courante échouera, ce qui dégradera l'expérience de vos utilisateurs.

Il existe trois façons principales de gérer les erreurs avec les resolvers de données :

1. Centraliser la gestion des erreurs dans `withNavigationErrorHandler`
2. Gérer les erreurs via un abonnement aux événements du router
3. Gérer les erreurs directement dans le resolver

### Centraliser la gestion des erreurs dans `withNavigationErrorHandler` {#centralize-error-handling-in-withnavigationerrorhandler}

La fonctionnalité [`withNavigationErrorHandler`](api/router/withNavigationErrorHandler) offre un moyen centralisé de gérer toutes les erreurs de navigation, y compris celles provenant de resolvers de données en échec. Cette approche regroupe la logique de gestion des erreurs en un seul endroit et évite de dupliquer le code de gestion des erreurs dans les resolvers.

```ts
import {bootstrapApplication} from '@angular/platform-browser';
import {provideRouter, Router, withNavigationErrorHandler} from '@angular/router';
import {inject} from '@angular/core';
import {routes} from './app.routes';

bootstrapApplication(App, {
  providers: [
    provideRouter(
      routes,
      withNavigationErrorHandler((error) => {
        const router = inject(Router);

        if (error.error?.message) {
          console.error('Navigation error occurred:', error.error.message);
        }

        router.navigate(['/error']);
      }),
    ),
  ],
});
```

Avec cette configuration, vos resolvers peuvent se concentrer sur la récupération des données et laisser le gestionnaire centralisé traiter les cas d'erreur :

```ts
export const userResolver: ResolveFn<User> = (route) => {
  const userStore = inject(UserStore);
  const userId = route.paramMap.get('id')!;
  // No need for explicit error handling - let it bubble up
  return userStore.getUser(userId);
};
```

### Gérer les erreurs via un abonnement aux événements du router {#managing-errors-through-a-subscription-to-router-events}

Vous pouvez aussi gérer les erreurs des resolvers en vous abonnant aux événements du router et en écoutant les événements `NavigationError`. Cette approche vous donne un contrôle plus fin sur la gestion des erreurs et vous permet d'implémenter votre propre logique de reprise après erreur.

```angular-ts
import {Component, inject, signal} from '@angular/core';
import {Router, NavigationError, RouterOutlet} from '@angular/router';
import {toSignal} from '@angular/core/rxjs-interop';
import {map} from 'rxjs';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  template: `
    @if (errorMessage()) {
      <div class="error-banner">
        {{ errorMessage() }}
        <button (click)="retryNavigation()">Retry</button>
      </div>
    }
    <router-outlet />
  `,
})
export class App {
  private router = inject(Router);
  private lastFailedUrl = signal('');

  private navigationErrors = toSignal(
    this.router.events.pipe(
      map((event) => {
        if (event instanceof NavigationError) {
          this.lastFailedUrl.set(event.url);

          if (event.error) {
            console.error('Navigation error', event.error);
          }

          return 'Navigation failed. Please try again.';
        }
        return '';
      }),
    ),
    {initialValue: ''},
  );

  errorMessage = this.navigationErrors;

  retryNavigation() {
    if (this.lastFailedUrl()) {
      this.router.navigateByUrl(this.lastFailedUrl());
    }
  }
}
```

Cette approche est particulièrement utile quand vous devez :

- Implémenter une logique de nouvelle tentative personnalisée pour les navigations en échec
- Afficher des messages d'erreur spécifiques selon le type d'échec
- Suivre les échecs de navigation à des fins d'analytics

### Gérer les erreurs directement dans le resolver {#handling-errors-directly-in-the-resolver}

Voici une version mise à jour du `userResolver` qui journalise l'erreur et renvoie vers la page générique `/users` à l'aide du service `Router` :

```ts
import {inject} from '@angular/core';
import {ResolveFn, RedirectCommand, Router} from '@angular/router';
import {catchError, of, EMPTY} from 'rxjs';
import {UserStore} from './user-store';
import type {User} from './types';

export const userResolver: ResolveFn<User | RedirectCommand> = (route) => {
  const userStore = inject(UserStore);
  const router = inject(Router);
  const userId = route.paramMap.get('id')!;

  return userStore.getUser(userId).pipe(
    catchError((error) => {
      console.error('Failed to load user:', error);
      return of(new RedirectCommand(router.parseUrl('/users')));
    }),
  );
};
```

## Considérations sur le chargement pendant la navigation {#navigation-loading-considerations}

Si les resolvers de données évitent les états de chargement dans les composants, ils introduisent une autre considération d'UX : la navigation est bloquée pendant l'exécution des resolvers. Les utilisateurs peuvent constater un délai entre le clic sur un lien et l'affichage de la nouvelle route, en particulier avec des requêtes réseau lentes.

TIP: Pour éviter de bloquer la navigation et afficher immédiatement les composants avec des skeletons ou des indicateurs de chargement, utilisez des [resources de route non bloquantes](/guide/routing/data-fetching-with-resources#blocking-and-non-blocking-resources).

### Indiquer la progression de la navigation {#providing-navigation-feedback}

Pour améliorer l'expérience utilisateur pendant l'exécution des resolvers, vous pouvez écouter les événements du router et afficher des indicateurs de chargement :

```angular-ts
import {Component, computed, inject} from '@angular/core';
import {Router, RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  template: `
    @if (isNavigating()) {
      <div class="loading-bar">Loading...</div>
    }
    <router-outlet />
  `,
})
export class App {
  private router = inject(Router);
  isNavigating = computed(() => !!this.router.currentNavigation());
}
```

Cette approche garantit que les utilisateurs disposent d'un retour visuel indiquant que la navigation est en cours pendant que les resolvers récupèrent les données.

## Bonnes pratiques {#best-practices}

- **Gardez des resolvers légers** : les resolvers doivent récupérer uniquement les données essentielles, et non tout ce dont la page pourrait avoir besoin
- **Gérez les erreurs** : pensez toujours à gérer proprement les erreurs pour offrir la meilleure expérience possible aux utilisateurs
- **Utilisez la mise en cache** : envisagez de mettre en cache les données résolues pour améliorer les performances
- **Pensez à l'UX de la navigation** : affichez des indicateurs de chargement pendant l'exécution des resolvers, car la navigation est bloquée pendant la récupération des données
- **Définissez des délais d'expiration raisonnables** : évitez les resolvers qui pourraient rester bloqués indéfiniment et bloquer la navigation
- **Sûreté de typage** : utilisez des interfaces TypeScript pour les données résolues

## Lire les données résolues du parent dans les resolvers enfants {#reading-parent-resolved-data-in-child-resolvers}

Les resolvers s'exécutent séquentiellement, du parent vers l'enfant. Quand une route parente définit un resolver, ses données résolues sont accessibles aux resolvers enfants qui s'exécutent ensuite.

NOTE: Comme les resolvers s'exécutent séquentiellement du parent vers l'enfant, chaque niveau d'imbrication s'ajoute au temps d'attente total de la navigation (une cascade de requêtes réseau). Si vos routes enfants ne dépendent pas des données du parent, envisagez les [resources de route](/guide/routing/data-fetching-with-resources), qui s'exécutent en parallèle sur toutes les routes.

```ts
import {inject} from '@angular/core';
import {provideRouter, ActivatedRouteSnapshot} from '@angular/router';
import {userResolver} from './resolvers';
import {UserPosts} from './pages';
import {PostService} from './services';
import type {User} from './types';

provideRouter([
  {
    path: 'users/:id',
    resolve: {user: userResolver}, // user resolver in the parent route
    children: [
      {
        path: 'posts',
        component: UserPosts,
        // route.data.user is available here while this resolver runs
        resolve: {
          posts: (route: ActivatedRouteSnapshot) => {
            const postService = inject(PostService);
            const user = route.parent?.data['user'] as User; // parent data
            const userId = user.id;
            return postService.getPostByUser(userId);
          },
        },
      },
    ],
  },
]);
```

## Étapes suivantes {#next-steps}

<docs-pill-row>
  <docs-pill href="/guide/routing/data-fetching-with-resources" title="Récupérer des données avec les resources"/>
  <docs-pill href="/guide/routing/route-guards" title="Contrôler l'accès aux routes avec des guards"/>
</docs-pill-row>
