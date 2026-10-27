<!-- source: efe7da9aa9a25e06250e4806dfc08a3fde44782e -->
# Contrôler l'accès aux routes avec des guards

CRITICAL: Ne comptez jamais sur des guards côté client comme seule source de contrôle d'accès. Tout le JavaScript exécuté dans un navigateur web peut être modifié par l'utilisateur de ce navigateur. Appliquez toujours l'autorisation des utilisateurs côté serveur, en plus des éventuelles guards côté client.

Les guards de route sont des fonctions qui contrôlent si un utilisateur peut naviguer vers une route donnée ou la quitter. Elles agissent comme des points de contrôle qui déterminent si un utilisateur peut accéder à des routes spécifiques. L'authentification et le contrôle d'accès sont des exemples courants d'utilisation des guards de route.

## Créer une guard de route {#creating-a-route-guard}

Vous pouvez générer une guard de route avec Angular CLI :

```bash
ng generate guard CUSTOM_NAME
```

Cette commande vous invite à choisir le [type de guard de route](#types-of-route-guards) à utiliser, puis crée le fichier `CUSTOM_NAME-guard.ts` correspondant.

TIP: Vous pouvez aussi créer une guard de route manuellement en créant un fichier TypeScript distinct dans votre projet Angular. Les développeurs ajoutent généralement le suffixe `-guard.ts` au nom du fichier pour le distinguer des autres fichiers.

## Types de retour des guards de route {#route-guard-return-types}

Toutes les guards de route partagent les mêmes types de retour possibles. Cela vous donne de la flexibilité dans la façon de contrôler la navigation :

| Types de retour                 | Description                                                                       |
| ------------------------------- | --------------------------------------------------------------------------------- |
| `boolean`                       | `true` autorise la navigation, `false` la bloque (voir la note sur la guard de route `CanMatch`) |
| `UrlTree` ou `RedirectCommand`  | Redirige vers une autre route au lieu de bloquer                                    |
| `Promise<T>` ou `Observable<T>` | Le router utilise la première valeur émise, puis se désabonne                         |

NOTE: `CanMatch` se comporte différemment : lorsqu'elle renvoie `false`, Angular essaie d'autres routes correspondantes au lieu de bloquer complètement la navigation.

## Types de guards de route {#types-of-route-guards}

Angular fournit quatre types de guards de route, chacun ayant un rôle différent :

<docs-pill-row>
  <docs-pill href="#canactivate" title="CanActivate"/>
  <docs-pill href="#canactivatechild" title="CanActivateChild"/>
  <docs-pill href="#candeactivate" title="CanDeactivate"/>
  <docs-pill href="#canmatch" title="CanMatch"/>
</docs-pill-row>

Toutes les guards ont accès aux [services fournis au niveau de la route](guide/di/defining-dependency-providers#route-providers), ainsi qu'aux informations propres à la route via l'argument `route`.

### CanActivate

La guard `CanActivate` détermine si un utilisateur peut accéder à une route. Elle est le plus souvent utilisée pour l'authentification et l'autorisation.

Elle a accès aux arguments par défaut suivants :

- `route` : `ActivatedRouteSnapshot` - Contient les informations sur la route en cours d'activation
- `state` : `RouterStateSnapshot` - Contient l'état courant du router

Elle peut renvoyer les [types de retour standard des guards](#route-guard-return-types).

```ts
export const authGuard: CanActivateFn = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  const authService = inject(AuthService);
  return authService.isAuthenticated();
};
```

TIP: Si vous devez rediriger l'utilisateur, renvoyez un [`URLTree`](api/router/UrlTree) ou une [`RedirectCommand`](api/router/RedirectCommand). Ne renvoyez **pas** `false` pour ensuite faire naviguer l'utilisateur par programmation avec `navigate`.

Pour plus d'informations, consultez la [documentation d'API de CanActivateFn](api/router/CanActivateFn).

### CanActivateChild

La guard `CanActivateChild` détermine si un utilisateur peut accéder aux routes enfants d'une route parente donnée. C'est utile lorsque vous voulez protéger toute une section de routes imbriquées. Autrement dit, `canActivateChild` s'exécute pour _tous_ les enfants. Si un composant enfant possède lui-même un autre composant enfant, `canActivateChild` s'exécute une fois pour chacun d'eux.

Elle a accès aux arguments par défaut suivants :

- `childRoute` : `ActivatedRouteSnapshot` - Contient les informations sur le snapshot « futur » (c'est-à-dire l'état vers lequel le router tente de naviguer) de la route enfant en cours d'activation
- `state` : `RouterStateSnapshot` - Contient l'état courant du router

Elle peut renvoyer les [types de retour standard des guards](#route-guard-return-types).

```ts
export const adminChildGuard: CanActivateChildFn = (
  childRoute: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  const authService = inject(AuthService);
  return authService.hasRole('admin');
};
```

Pour plus d'informations, consultez la [documentation d'API de CanActivateChildFn](api/router/CanActivateChildFn).

### CanDeactivate

La guard `CanDeactivate` détermine si un utilisateur peut quitter une route. Un scénario courant consiste à empêcher de quitter des formulaires non enregistrés.

Elle a accès aux arguments par défaut suivants :

- `component` : `T` - L'instance du composant en cours de désactivation
- `currentRoute` : `ActivatedRouteSnapshot` - Contient les informations sur la route courante
- `currentState` : `RouterStateSnapshot` - Contient l'état courant du router
- `nextState` : `RouterStateSnapshot` - Contient le prochain état du router vers lequel la navigation se dirige

Elle peut renvoyer les [types de retour standard des guards](#route-guard-return-types).

```ts
export const unsavedChangesGuard: CanDeactivateFn<Form> = (
  component: Form,
  currentRoute: ActivatedRouteSnapshot,
  currentState: RouterStateSnapshot,
  nextState: RouterStateSnapshot,
) => {
  return component.hasUnsavedChanges()
    ? confirm('You have unsaved changes. Are you sure you want to leave?')
    : true;
};
```

Pour plus d'informations, consultez la [documentation d'API de CanDeactivateFn](api/router/CanDeactivateFn).

### CanMatch

La guard `CanMatch` détermine si une route peut être retenue lors de la correspondance des chemins. Contrairement aux autres guards, un refus ne bloque pas entièrement la navigation : le router essaie les autres routes correspondantes. Cela peut être utile pour les feature flags, les tests A/B ou le chargement conditionnel de routes.

Elle a accès aux arguments par défaut suivants :

- `route` : `Route` - La configuration de route en cours d'évaluation
- `segments` : `UrlSegment[]` - Les segments d'URL qui n'ont pas été consommés par les évaluations des routes parentes précédentes
- `currentSnapshot: PartialMatchRouteSnapshot` - Le snapshot de route courant jusqu'à ce stade du processus de correspondance

Elle peut renvoyer les [types de retour standard des guards](#route-guard-return-types), mais lorsqu'elle renvoie `false`, Angular essaie d'autres routes correspondantes au lieu de bloquer complètement la navigation.

```ts
export const featureToggleGuard: CanMatchFn = (
  route: Route,
  segments: UrlSegment[],
  currentSnapshot: PartialMatchRouteSnapshot,
) => {
  const featureService = inject(FeatureService);
  return featureService.isFeatureEnabled('newDashboard');
};
```

Elle permet aussi d'utiliser des composants différents pour le même chemin.

```ts {header: "routes.ts"}
const routes: Routes = [
  {
    path: 'dashboard',
    component: AdminDashboard,
    canMatch: [adminGuard],
  },
  {
    path: 'dashboard',
    component: UserDashboard,
    canMatch: [userGuard],
  },
];
```

Dans cet exemple, quand l'utilisateur visite `/dashboard`, la première route dont la guard est satisfaite est utilisée.

Pour plus d'informations, consultez la [documentation d'API de CanMatchFn](api/router/CanMatchFn).

## Appliquer des guards aux routes {#applying-guards-to-routes}

Une fois vos guards de route créées, vous devez les configurer dans vos définitions de routes.

Les guards sont spécifiées sous forme de tableaux dans la configuration de route afin de vous permettre d'appliquer plusieurs guards à une même route. Le router démarre les guards dans l'ordre où elles apparaissent dans le tableau, mais il n'attend pas qu'une guard ait terminé pour démarrer la suivante. Les guards qui renvoient une `Promise` ou un `Observable` s'exécutent donc en parallèle.

L'ordre du tableau détermine quel résultat est prioritaire. La navigation ne continue que si chaque guard renvoie `true`. Sinon, le router utilise le résultat de la première guard du tableau qui n'a pas renvoyé `true`, même si une guard suivante a terminé plus tôt.

NOTE: Comme les guards asynchrones s'exécutent en parallèle, une guard ne doit pas dépendre d'un travail effectué par une guard précédente du même tableau, comme le chargement de l'utilisateur courant. Placez ce travail dans un service utilisé par les deux guards, ou combinez les vérifications dans une seule guard.

```ts
import {Routes} from '@angular/router';
import {authGuard} from './guards/auth.guard';
import {adminGuard} from './guards/admin.guard';
import {canDeactivateGuard} from './guards/can-deactivate.guard';
import {featureToggleGuard} from './guards/feature-toggle.guard';

const routes: Routes = [
  // Basic CanActivate - requires authentication
  {
    path: 'dashboard',
    component: Dashboard,
    canActivate: [authGuard],
  },

  // Multiple CanActivate guards - requires authentication AND admin role
  {
    path: 'admin',
    component: Admin,
    canActivate: [authGuard, adminGuard],
  },

  // CanActivate + CanDeactivate - protected route with unsaved changes check
  {
    path: 'profile',
    component: Profile,
    canActivate: [authGuard],
    canDeactivate: [canDeactivateGuard],
  },

  // CanActivateChild - protects all child routes
  {
    path: 'users', // /user - NOT protected
    canActivateChild: [authGuard],
    children: [
      // /users/list - PROTECTED
      {path: 'list', component: UserList},
      // /users/detail/:id - PROTECTED
      {path: 'detail/:id', component: UserDetail},
    ],
  },

  // CanMatch - conditionally matches route based on feature flag
  {
    path: 'beta-feature',
    component: BetaFeature,
    canMatch: [featureToggleGuard],
  },

  // Fallback route if beta feature is disabled
  {
    path: 'beta-feature',
    component: ComingSoon,
  },
];
```
