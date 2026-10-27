<!-- source: 7709afb7412cad5799ef0fc4cb110c4b13e36d27 -->
# Personnaliser le comportement des routes

Angular Router fournit de puissants points d'extension qui vous permettent de personnaliser le comportement des routes dans votre application. Si le comportement de routing par défaut convient à la plupart des applications, des besoins spécifiques exigent souvent des implémentations personnalisées pour optimiser les performances, traiter les URL de façon particulière ou gérer une logique de routing complexe.

La personnalisation des routes peut devenir utile lorsque votre application a besoin de :

- **Préserver l'état des composants** d'une navigation à l'autre pour éviter de récupérer à nouveau les données
- **Charger les modules en lazy loading de façon stratégique** selon le comportement de l'utilisateur ou les conditions réseau
- **Intégrer des URL externes** ou gérer des routes Angular aux côtés de systèmes existants
- **Faire correspondre les routes dynamiquement** selon des conditions d'exécution qui vont au-delà de simples
  motifs de chemin

NOTE: Avant d'implémenter des stratégies personnalisées, assurez-vous que le comportement par défaut du router ne répond pas à vos besoins. Le routing par défaut d'Angular est optimisé pour les cas d'usage courants et offre le meilleur équilibre entre performances et simplicité. Personnaliser les stratégies de routing peut complexifier le code et avoir un impact sur la consommation mémoire si ce n'est pas géré avec soin.

## Options de configuration du router {#router-configuration-options}

`withRouterConfig` ou `RouterModule.forRoot` permet de fournir des `RouterConfigOptions` supplémentaires pour ajuster le comportement du router.

### Gérer les navigations annulées {#handle-canceled-navigations}

`canceledNavigationResolution` contrôle la façon dont le router restaure l'historique du navigateur lorsqu'une navigation est annulée. La valeur par défaut est `'replace'`, qui revient à l'URL d'avant la navigation avec `location.replaceState`. En pratique, cela signifie que chaque fois que la barre d'adresse a déjà été mise à jour pour la navigation, par exemple avec les boutons précédent ou suivant du navigateur, l'entrée d'historique est écrasée par le « retour en arrière » si la navigation échoue ou est rejetée par une guard.
Passer à `'computed'` garde l'index d'historique en cours synchronisé avec la navigation Angular : annuler une navigation déclenchée par le bouton précédent déclenche alors une navigation vers l'avant (et inversement) pour revenir à la page d'origine.

Ce réglage est surtout utile lorsque votre application utilise `urlUpdateStrategy: 'eager'` ou lorsque des guards annulent fréquemment des navigations popstate initiées par le navigateur.

```ts
provideRouter(routes, withRouterConfig({canceledNavigationResolution: 'computed'}));
```

### Réagir aux navigations vers la même URL {#react-to-same-url-navigations}

`onSameUrlNavigation` configure ce qui doit se passer lorsque l'utilisateur demande à naviguer vers l'URL courante. La valeur par défaut `'ignore'` ne fait rien, tandis que `'reload'` indique au router de traiter l'URL dans son pipeline de navigation au lieu de l'ignorer. La réexécution de chaque guard et de chaque resolver est régie par [`runGuardsAndResolvers`](api/router/RunGuardsAndResolvers), et la réutilisation des composants est déterminée par la [stratégie de réutilisation des routes](#route-reuse-strategy).

C'est utile lorsque vous voulez que des clics répétés sur un filtre de liste, un élément de navigation latérale ou un bouton d'actualisation déclenchent une nouvelle récupération de données alors que l'URL ne change pas.

```ts
provideRouter(routes, withRouterConfig({onSameUrlNavigation: 'reload'}));
```

Vous pouvez aussi contrôler ce comportement navigation par navigation plutôt que globalement. Cela vous permet de conserver la valeur par défaut `'ignore'` tout en activant le rechargement de façon sélective pour des cas d'usage précis :

```ts
router.navigate(['/some-path'], {onSameUrlNavigation: 'reload'});
```

TIP: Si vous utilisez des [resources de route](/guide/routing/data-fetching-with-resources#reloading-resources-without-renavigation), vous pouvez actualiser les données à la demande en appelant `.reload()` sur la resource ou en mettant à jour les signals lus dans `params`, sans avoir à configurer `onSameUrlNavigation` ni à déclencher une navigation du router.

### Contrôler l'héritage des paramètres {#control-parameter-inheritance}

`paramsInheritanceStrategy` définit comment les paramètres et les données de route sont transmis depuis les routes parentes.

Par défaut (`'always'`), les routes enfants héritent automatiquement des paramètres, des données de route et des valeurs résolues de leurs routes parentes. Les paramètres matriciels, les données de route et les valeurs résolues sont ainsi disponibles plus bas dans l'arbre de routes, ce qui est pratique lorsque vous partagez des identifiants contextuels entre plusieurs zones fonctionnelles, par exemple :

```text {hideCopy}
/org/:orgId/projects/:projectId/customers/:customerId
```

```ts
export const routes: Routes = [
  {
    path: 'org/:orgId',
    component: Organization,
    children: [
      {
        path: 'projects/:projectId',
        component: Project,
        children: [
          {
            path: 'customers/:customerId',
            component: Customer,
          },
        ],
      },
    ],
  },
];
```

```ts
@Component({/* ... */})
export class Customer {
  private route = inject(ActivatedRoute);

  // All parent parameters are available directly
  orgId = this.route.snapshot.params['orgId'];
  projectId = this.route.snapshot.params['projectId'];
  customerId = this.route.snapshot.params['customerId'];
}
```

Pour restaurer l'ancien comportement, définissez `paramsInheritanceStrategy` à `'emptyOnly'`. Avec `'emptyOnly'`, les routes enfants n'héritent des paramètres que lorsque leur chemin est vide ou que la route parente ne déclare pas de composant :

```ts
provideRouter(routes, withRouterConfig({paramsInheritanceStrategy: 'emptyOnly'}));
```

Dans ce cas, le composant `Customer` doit lire les paramètres parents depuis ses routes ancêtres :

```ts
@Component({/* ... */})
export class Customer {
  private route = inject(ActivatedRoute);

  orgId = this.route.parent?.parent?.snapshot.params['orgId'];
  projectId = this.route.parent?.snapshot.params['projectId'];
  customerId = this.route.snapshot.params['customerId'];
}
```

### Décider quand l'URL est mise à jour {#decide-when-the-url-updates}

`urlUpdateStrategy` détermine quand Angular écrit dans la barre d'adresse du navigateur. La valeur par défaut `'deferred'` attend qu'une navigation réussisse avant de modifier l'URL. Utilisez `'eager'` pour mettre à jour l'URL dès le début de la navigation. Les mises à jour immédiates permettent plus facilement d'exposer l'URL tentée si la navigation échoue à cause de guards ou d'erreurs, mais peuvent afficher brièvement une URL en cours de navigation si vos guards s'exécutent longtemps.

Envisagez cette option lorsque votre pipeline d'analytics doit voir la route tentée, même si des guards la bloquent.

```ts
provideRouter(routes, withRouterConfig({urlUpdateStrategy: 'eager'}));
```

### Choisir le traitement par défaut des query parameters {#choose-default-query-parameter-handling}

`defaultQueryParamsHandling` définit le comportement de repli de `Router.createUrlTree` lorsque l'appel ne précise pas `queryParamsHandling`. `'replace'` est la valeur par défaut et remplace la query string existante. `'merge'` combine les valeurs fournies avec les valeurs courantes, et `'preserve'` conserve les query parameters existants, sauf si vous en fournissez explicitement de nouveaux.

```ts
provideRouter(routes, withRouterConfig({defaultQueryParamsHandling: 'merge'}));
```

C'est particulièrement utile pour les pages de recherche et de filtrage, afin de conserver automatiquement les filtres existants lorsque des paramètres supplémentaires sont fournis.

### Configurer le traitement de la barre oblique finale {#configure-trailing-slash-handling}

Par défaut, le service `Location` supprime la barre oblique finale des URL à la lecture.

Vous pouvez configurer le service `Location` pour forcer une barre oblique finale sur toutes les URL écrites dans le navigateur en fournissant `TrailingSlashPathLocationStrategy` dans votre application.

```ts
import {LocationStrategy, TrailingSlashPathLocationStrategy} from '@angular/common';
import {bootstrapApplication} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [{provide: LocationStrategy, useClass: TrailingSlashPathLocationStrategy}],
});
```

Vous pouvez aussi forcer le service `Location` à ne jamais ajouter de barre oblique finale aux URL écrites dans le navigateur en fournissant `NoTrailingSlashPathLocationStrategy` dans votre application.

```ts
import {LocationStrategy, NoTrailingSlashPathLocationStrategy} from '@angular/common';
import {bootstrapApplication} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [{provide: LocationStrategy, useClass: NoTrailingSlashPathLocationStrategy}],
});
```

Ces stratégies n'affectent que l'URL écrite dans le navigateur.
`Location.path()` et `Location.normalize()` continueront de supprimer la barre oblique finale à la lecture de l'URL.

Angular Router expose quatre grands axes de personnalisation :

  <docs-pill-row>
    <docs-pill href="#route-reuse-strategy" title="Stratégie de réutilisation des routes"/>
    <docs-pill href="#preloading-strategy" title="Stratégie de préchargement"/>
    <docs-pill href="#url-handling-strategy" title="Stratégie de traitement des URL"/>
    <docs-pill href="#custom-route-matchers" title="Matchers de route personnalisés"/>
  </docs-pill-row>

## Stratégie de réutilisation des routes {#route-reuse-strategy}

La stratégie de réutilisation des routes contrôle si Angular détruit et recrée les composants pendant la navigation ou les préserve pour les réutiliser. Par défaut, Angular détruit les instances de composant lorsque l'on quitte une route et crée de nouvelles instances lorsque l'on y revient.

### Quand implémenter la réutilisation des routes {#when-to-implement-route-reuse}

Les stratégies de réutilisation des routes personnalisées profitent aux applications qui ont besoin de :

- **Préserver l'état des formulaires** : conserver les formulaires partiellement remplis lorsque les utilisateurs quittent la page puis y reviennent
- **Conserver des données coûteuses** : éviter de récupérer à nouveau de gros jeux de données ou de refaire des calculs complexes
- **Maintenir la position de défilement** : préserver les positions de défilement dans les longues listes ou les implémentations de défilement infini
- **Créer des interfaces à onglets** : conserver l'état des composants lors du passage d'un onglet à l'autre

### Créer une stratégie de réutilisation des routes personnalisée {#creating-a-custom-route-reuse-strategy}

La classe `RouteReuseStrategy` d'Angular vous permet de personnaliser le comportement de navigation grâce au concept de « detached route handles » (références de routes détachées).

Les « detached route handles » sont la façon dont Angular stocke les instances de composant et toute leur hiérarchie de vues. Lorsqu'une route est détachée, Angular préserve en mémoire l'instance du composant, ses composants enfants et tout l'état associé. Cet état préservé peut ensuite être rattaché lorsque l'on revient sur la route.

La classe `RouteReuseStrategy` fournit les méthodes suivantes, qui contrôlent le cycle de vie des composants de route :

| Méthode | Description |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| [`shouldDetach`](api/router/RouteReuseStrategy#shouldDetach) | Détermine si une route doit être stockée pour être réutilisée plus tard lorsque l'on la quitte |
| [`store`](api/router/RouteReuseStrategy#store) | Stocke le detached route handle lorsque `shouldDetach` renvoie true |
| [`shouldAttach`](api/router/RouteReuseStrategy#shouldAttach) | Détermine si une route stockée doit être rattachée lorsque l'on y navigue |
| [`retrieve`](api/router/RouteReuseStrategy#retrieve) | Renvoie le route handle stocké précédemment pour le rattacher |
| [`shouldReuseRoute`](api/router/RouteReuseStrategy#shouldReuseRoute) | Détermine si le router doit réutiliser l'instance de route courante au lieu de la détruire pendant la navigation |
| [`shouldDestroyInjector`](api/router/RouteReuseStrategy#shouldDestroyInjector) | Détermine si le router doit détruire l'injector d'une route détachée lorsqu'elle n'est plus stockée |

L'exemple suivant montre une stratégie de réutilisation des routes personnalisée qui préserve de façon sélective l'état des composants en fonction des métadonnées de la route :

```ts
import {
  RouteReuseStrategy,
  Route,
  ActivatedRouteSnapshot,
  DetachedRouteHandle,
} from '@angular/router';
import {Injectable} from '@angular/core';

@Injectable()
export class CustomRouteReuseStrategy implements RouteReuseStrategy {
  private handlers = new Map<Route | null, DetachedRouteHandle>();

  shouldDetach(route: ActivatedRouteSnapshot): boolean {
    // Determines if a route should be stored for later reuse
    return route.data['reuse'] === true;
  }

  store(route: ActivatedRouteSnapshot, handle: DetachedRouteHandle | null): void {
    // Stores the detached route handle when shouldDetach returns true
    if (handle && route.data['reuse'] === true) {
      const key = this.getRouteKey(route);
      this.handlers.set(key, handle);
    }
  }

  shouldAttach(route: ActivatedRouteSnapshot): boolean {
    // Checks if a stored route should be reattached
    const key = this.getRouteKey(route);
    return route.data['reuse'] === true && this.handlers.has(key);
  }

  retrieve(route: ActivatedRouteSnapshot): DetachedRouteHandle | null {
    // Returns the stored route handle for reattachment
    const key = this.getRouteKey(route);
    return route.data['reuse'] === true ? (this.handlers.get(key) ?? null) : null;
  }

  shouldReuseRoute(future: ActivatedRouteSnapshot, curr: ActivatedRouteSnapshot): boolean {
    // Determines if the router should reuse the current route instance
    return future.routeConfig === curr.routeConfig;
  }

  private getRouteKey(route: ActivatedRouteSnapshot): Route | null {
    return route.routeConfig;
  }
}
```

### Détruire manuellement les detached route handles {#manually-destroying-detached-route-handles}

Lorsque vous implémentez une `RouteReuseStrategy` personnalisée, vous pouvez avoir besoin de détruire manuellement un `DetachedRouteHandle` si vous décidez de l'abandonner sans le rattacher. Par exemple, si votre stratégie limite la taille du cache ou fait expirer les handles après un certain temps, vous devez vous assurer que le composant et son état sont correctement détruits pour éviter les fuites mémoire.

Comme `DetachedRouteHandle` est un type opaque, vous ne pouvez pas appeler directement une méthode de destruction dessus. Utilisez plutôt la fonction `destroyDetachedRouteHandle` fournie par le router.

```ts
import {destroyDetachedRouteHandle} from '@angular/router';

// ... inside your strategy
if (this.handles.size > MAX_CACHE_SIZE) {
  const handle = this.handles.get(oldestKey);
  if (handle) {
    destroyDetachedRouteHandle(handle);
    this.handles.delete(oldestKey);
  }
}
```

NOTE: Évitez d'utiliser le chemin de la route comme clé lorsque des guards `canMatch` sont impliquées, car cela peut produire des entrées en double.

### Nettoyage automatique des injectors de route inutilisés {#automatic-cleanup-of-unused-route-injectors}

Par défaut, Angular ne détruit pas les injectors des routes détachées, même s'ils ne sont plus stockés par la `RouteReuseStrategy`. C'est principalement parce que ce niveau de gestion de la mémoire n'est pas nécessaire pour la plupart des applications.

Pour activer le nettoyage automatique des injectors de route inutilisés, vous pouvez utiliser la fonctionnalité `withAutoCleanupInjectors` dans la configuration de votre router. Après chaque navigation, cette fonctionnalité vérifie quelles routes sont actuellement stockées par la stratégie et détruit les injectors de toutes les routes détachées qui ne sont pas stockées par la stratégie de réutilisation.

```ts
import {provideRouter, withAutoCleanupInjectors} from '@angular/router';
import {ApplicationConfig} from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes, withAutoCleanupInjectors())],
};
```

Si vous ne fournissez pas de `RouteReuseStrategy` personnalisée, ou si votre stratégie personnalisée étend `BaseRouteReuseStrategy`, les injectors seront désormais détruits lorsque la route est inactive.

#### Nettoyage avec une `RouteReuseStrategy` personnalisée {#cleanup-with-a-custom-routereusestrategy}

Si votre application utilise une `RouteReuseStrategy` personnalisée _et_ que cette stratégie n'étend pas `BaseRouteReuseStrategy`, vous devez implémenter `shouldDestroyInjector` pour indiquer au router quelles routes doivent voir leurs injectors détruits :

```ts
@Injectable()
export class CustomRouteReuseStrategy implements RouteReuseStrategy {
  // ... other methods

  shouldDestroyInjector(route: Route): boolean {
    return !route.data['retainInjector'];
  }
}
```

Si votre stratégie stocke un jour un `DetachedRouteHandle`, vous devrez aussi en informer le router pour qu'il ne détruise pas les injectors dont ce handle détaché a besoin :

```ts
@Injectable()
export class CustomRouteReuseStrategy implements RouteReuseStrategy {
  private readonly handles = new Map<Route, DetachedRouteHandle>();

  store(route: ActivatedRouteSnapshot, handle: DetachedRouteHandle | null) {
    if (handle) {
      this.handles.set(route.routeConfig!, handle);
    }
  }

  retrieveStoredRouteHandles(): DetachedRouteHandle[] {
    return Array.from(this.handles.values());
  }

  // ... other methods
}
```

### Configurer une route pour utiliser une stratégie de réutilisation personnalisée {#configuring-a-route-to-use-a-custom-route-reuse-strategy}

Les routes peuvent activer la réutilisation via les métadonnées de leur configuration. Cette approche sépare la logique de réutilisation du code des composants, ce qui permet d'ajuster facilement le comportement sans modifier les composants :

```ts
export const routes: Routes = [
  {
    path: 'products',
    component: ProductList,
    data: {reuse: true}, // Component state persists across navigations
  },
  {
    path: 'products/:id',
    component: ProductDetail,
    // No reuse flag - component recreates on each navigation
  },
  {
    path: 'search',
    component: Search,
    data: {reuse: true}, // Preserves search results and filter state
  },
];
```

Vous pouvez aussi configurer une stratégie de réutilisation des routes personnalisée au niveau de l'application grâce au système d'injection de dépendances d'Angular. Dans ce cas, Angular crée une seule instance de la stratégie, qui gère toutes les décisions de réutilisation des routes dans l'application :

```ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    {provide: RouteReuseStrategy, useClass: CustomRouteReuseStrategy},
  ],
};
```

## Stratégie de préchargement {#preloading-strategy}

Les stratégies de préchargement déterminent quand Angular charge en arrière-plan les modules de route chargés en lazy loading. Si le lazy loading améliore le temps de chargement initial en reportant le téléchargement des modules, les utilisateurs subissent tout de même un délai lors de leur première navigation vers une route en lazy loading. Les stratégies de préchargement suppriment ce délai en chargeant les modules avant que les utilisateurs ne les demandent.

### Stratégies de préchargement intégrées {#built-in-preloading-strategies}

Angular fournit deux stratégies de préchargement prêtes à l'emploi :

| Stratégie | Description |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| [`NoPreloading`](api/router/NoPreloading) | La stratégie par défaut, qui désactive tout préchargement. Autrement dit, les modules ne se chargent que lorsque les utilisateurs y naviguent |
| [`PreloadAllModules`](api/router/PreloadAllModules) | Charge tous les modules en lazy loading immédiatement après la navigation initiale |

La stratégie `PreloadAllModules` peut être configurée comme suit :

```ts
import {ApplicationConfig} from '@angular/core';
import {provideRouter, withPreloading, PreloadAllModules} from '@angular/router';
import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes, withPreloading(PreloadAllModules))],
};
```

La stratégie `PreloadAllModules` fonctionne bien pour les applications petites à moyennes, où télécharger tous les modules n'a pas d'impact notable sur les performances. Les applications plus grandes, avec de nombreux modules fonctionnels, peuvent toutefois tirer parti d'un préchargement plus sélectif.

### Créer une stratégie de préchargement personnalisée {#creating-a-custom-preloading-strategy}

Les stratégies de préchargement personnalisées implémentent l'interface `PreloadingStrategy`, qui exige une seule méthode `preload`. Cette méthode reçoit la configuration de la route et une fonction qui déclenche le chargement effectif du module. La stratégie renvoie un observable qui émet lorsque le préchargement est terminé, ou un observable vide pour ignorer le préchargement :

```ts
import {Injectable} from '@angular/core';
import {PreloadingStrategy, Route} from '@angular/router';
import {Observable, of, timer} from 'rxjs';
import {mergeMap} from 'rxjs/operators';

@Injectable()
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // Only preload routes marked with data: { preload: true }
    if (route.data?.['preload']) {
      return load();
    }
    return of(null);
  }
}
```

Cette stratégie sélective vérifie les métadonnées de la route pour déterminer le comportement de préchargement. Les routes peuvent activer le préchargement via leur configuration :

```ts
import {Routes} from '@angular/router';

export const routes: Routes = [
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.routes'),
    data: {preload: true}, // Preload immediately after initial navigation
  },
  {
    path: 'reports',
    loadChildren: () => import('./reports/reports.routes'),
    data: {preload: false}, // Only load when user navigates to reports
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes'),
    // No preload flag - won't be preloaded
  },
];
```

### Performances et préchargement {#performance-considerations-for-preloading}

Le préchargement a un impact à la fois sur l'utilisation du réseau et sur la consommation mémoire. Chaque module préchargé consomme de la bande passante et augmente l'empreinte mémoire de l'application. Les utilisateurs mobiles avec une connexion facturée à l'usage peuvent préférer un préchargement minimal, tandis que les utilisateurs sur ordinateur avec un réseau rapide peuvent supporter des stratégies de préchargement agressives.

Le moment du préchargement compte aussi. Un préchargement immédiat après le chargement initial peut entrer en concurrence avec d'autres ressources critiques, comme les images ou les appels d'API. Les stratégies doivent tenir compte du comportement de l'application après le chargement et se coordonner avec les autres tâches d'arrière-plan pour éviter de dégrader les performances.

Les limites de ressources du navigateur influencent aussi le comportement du préchargement. Les navigateurs limitent le nombre de connexions HTTP simultanées : un préchargement agressif peut donc se retrouver en file d'attente derrière d'autres requêtes. Les service workers peuvent aider en offrant un contrôle fin de la mise en cache et des requêtes réseau, en complément de la stratégie de préchargement.

## Stratégie de traitement des URL {#url-handling-strategy}

Les stratégies de traitement des URL déterminent quelles URL le router d'Angular traite et lesquelles il ignore. Par défaut, Angular tente de gérer tous les événements de navigation au sein de l'application, mais les applications réelles doivent souvent coexister avec d'autres systèmes, gérer des liens externes ou s'intégrer à des applications existantes qui gèrent leurs propres routes.

La classe `UrlHandlingStrategy` vous donne le contrôle de cette frontière entre les routes gérées par Angular et les URL externes. Cela devient essentiel lorsque vous migrez progressivement des applications vers Angular, ou lorsque des applications Angular doivent partager l'espace d'URL avec d'autres frameworks.

### Implémenter une stratégie de traitement des URL personnalisée {#implementing-a-custom-url-handling-strategy}

Les stratégies de traitement des URL personnalisées étendent la classe `UrlHandlingStrategy` et implémentent trois méthodes. La méthode `shouldProcessUrl` détermine si Angular doit traiter une URL donnée, `extract` renvoie la partie de l'URL qu'Angular doit traiter, et `merge` combine le fragment d'URL avec le reste de l'URL :

```ts
import {Injectable} from '@angular/core';
import {UrlHandlingStrategy, UrlTree} from '@angular/router';

@Injectable()
export class CustomUrlHandlingStrategy implements UrlHandlingStrategy {
  shouldProcessUrl(url: UrlTree): boolean {
    // Only handle URLs that start with /app or /admin
    return url.toString().startsWith('/app') || url.toString().startsWith('/admin');
  }

  extract(url: UrlTree): UrlTree {
    // Return the URL unchanged if we should process it
    return url;
  }

  merge(newUrlPart: UrlTree, rawUrl: UrlTree): UrlTree {
    // Combine the URL fragment with the rest of the URL
    return newUrlPart;
  }
}
```

Cette stratégie crée des frontières claires dans l'espace d'URL. Angular gère les chemins `/app` et `/admin` et ignore tout le reste. Ce pattern fonctionne bien pour migrer des applications existantes, où Angular contrôle certaines sections tandis que l'ancien système gère les autres.

### Configurer une stratégie de traitement des URL personnalisée {#configuring-a-custom-url-handling-strategy}

Vous pouvez enregistrer une stratégie personnalisée via le système d'injection de dépendances d'Angular :

```ts
import {ApplicationConfig} from '@angular/core';
import {provideRouter, UrlHandlingStrategy} from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    {provide: UrlHandlingStrategy, useClass: CustomUrlHandlingStrategy},
  ],
};
```

## Matchers de route personnalisés {#custom-route-matchers}

Par défaut, le router d'Angular parcourt les routes dans l'ordre où elles sont définies et tente de faire correspondre le chemin de l'URL au motif de chemin de chaque route. Il prend en charge les segments statiques, les segments paramétrés (`:id`) et les wildcards (`**`). La première route qui correspond l'emporte, et le router arrête sa recherche.

Lorsque les applications ont besoin d'une logique de correspondance plus élaborée, basée sur des conditions d'exécution, des motifs d'URL complexes ou d'autres règles personnalisées, les matchers personnalisés offrent cette flexibilité sans sacrifier la simplicité des routes standards.

Le router évalue les matchers personnalisés pendant la phase de correspondance des routes, avant la correspondance des chemins. Lorsqu'un matcher renvoie une correspondance réussie, il peut aussi extraire des paramètres de l'URL et les mettre à la disposition du composant activé, exactement comme les paramètres de route standards.

### Créer un matcher personnalisé {#creating-a-custom-matcher}

Un matcher personnalisé est une fonction qui reçoit des segments d'URL et renvoie soit un résultat de correspondance avec les segments consommés et les paramètres, soit null pour indiquer qu'il n'y a pas de correspondance. La fonction de matcher s'exécute avant qu'Angular n'évalue la propriété path de la route :

```ts
import {Route, UrlSegment, UrlSegmentGroup, UrlMatchResult} from '@angular/router';

export function customMatcher(
  segments: UrlSegment[],
  group: UrlSegmentGroup,
  route: Route,
): UrlMatchResult | null {
  // Matching logic here
  if (matchSuccessful) {
    return {
      consumed: segments,
      posParams: {
        paramName: new UrlSegment('paramValue', {}),
      },
    };
  }
  return null;
}
```

### Implémenter un routing basé sur les versions {#implementing-version-based-routing}

Prenons un site de documentation d'API qui doit router selon le numéro de version présent dans l'URL. Les différentes versions peuvent avoir des structures de composants ou des ensembles de fonctionnalités différents :

```ts
import {Routes, UrlSegment, UrlMatchResult} from '@angular/router';

export function versionMatcher(segments: UrlSegment[]): UrlMatchResult | null {
  // Match patterns like /v1/docs, /v2.1/docs, /v3.0.1/docs
  if (segments.length >= 2 && segments[0].path.match(/^v\d+(\.\d+)*$/)) {
    return {
      consumed: segments.slice(0, 2), // Consume version and 'docs'
      posParams: {
        version: segments[0], // Make version available as a parameter
        section: segments[1], // Make section available too
      },
    };
  }
  return null;
}

// Route configuration
export const routes: Routes = [
  {
    matcher: versionMatcher,
    component: Documentation,
  },
  {
    path: 'latest/docs',
    redirectTo: 'v3/docs',
  },
];
```

Le composant reçoit les paramètres extraits via les inputs de route :

```angular-ts
import {Component, input, inject, resource} from '@angular/core';

@Component({
  selector: 'app-documentation',
  template: `
    @if (documentation.isLoading()) {
      <div>Loading documentation...</div>
    } @else if (documentation.error()) {
      <div>Error loading documentation</div>
    } @else if (documentation.value(); as docs) {
      <article>{{ docs.content }}</article>
    }
  `,
})
export class Documentation {
  // Route parameters are automatically bound to signal inputs
  version = input.required<string>(); // Receives the version parameter
  section = input.required<string>(); // Receives the section parameter

  private docsService = inject(DocumentationService);

  // Resource automatically loads documentation when version or section changes
  documentation = resource({
    params: () => {
      if (!this.version() || !this.section()) return;

      return {
        version: this.version(),
        section: this.section(),
      };
    },
    loader: ({params}) => {
      return this.docsService.loadDocumentation(params.version, params.section);
    },
  });
}
```

### Routing tenant compte de la locale {#locale-aware-routing}

Les applications internationales encodent souvent la locale dans les URL. Un matcher personnalisé peut extraire les codes de locale et router vers les composants appropriés, tout en rendant la locale disponible sous forme de paramètre :

```ts
// Supported locales
const locales = ['en', 'es', 'fr', 'de', 'ja', 'zh'];

export function localeMatcher(segments: UrlSegment[]): UrlMatchResult | null {
  if (segments.length > 0) {
    const potentialLocale = segments[0].path;

    if (locales.includes(potentialLocale)) {
      // This is a locale prefix, consume it and continue matching
      return {
        consumed: [segments[0]],
        posParams: {
          locale: segments[0],
        },
      };
    } else {
      // No locale prefix, use default locale
      return {
        consumed: [], // Don't consume any segments
        posParams: {
          locale: new UrlSegment('en', {}),
        },
      };
    }
  }

  return null;
}
```

### Correspondance selon une logique métier complexe {#complex-business-logic-matching}

Les matchers personnalisés excellent dans l'implémentation de règles métier qu'il serait malaisé d'exprimer avec des motifs de chemin. Prenons un site d'e-commerce où les URL des produits suivent des motifs différents selon le type de produit :

```ts
export function productMatcher(segments: UrlSegment[]): UrlMatchResult | null {
  if (segments.length === 0) return null;

  const firstSegment = segments[0].path;

  // Books: /isbn-1234567890
  if (firstSegment.startsWith('isbn-')) {
    return {
      consumed: [segments[0]],
      posParams: {
        productType: new UrlSegment('book', {}),
        identifier: new UrlSegment(firstSegment.substring(5), {}),
      },
    };
  }

  // Electronics: /sku/ABC123
  if (firstSegment === 'sku' && segments.length > 1) {
    return {
      consumed: segments.slice(0, 2),
      posParams: {
        productType: new UrlSegment('electronics', {}),
        identifier: segments[1],
      },
    };
  }

  // Clothing: /style/BRAND/ITEM
  if (firstSegment === 'style' && segments.length > 2) {
    return {
      consumed: segments.slice(0, 3),
      posParams: {
        productType: new UrlSegment('clothing', {}),
        brand: segments[1],
        identifier: segments[2],
      },
    };
  }

  return null;
}
```

### Performances des matchers personnalisés {#performance-considerations-for-custom-matchers}

Les matchers personnalisés s'exécutent à chaque tentative de navigation jusqu'à ce qu'une correspondance soit trouvée. Une logique de correspondance complexe peut donc affecter les performances de navigation, en particulier dans les applications comptant de nombreuses routes. Gardez vos matchers ciblés et efficaces :

- Sortez au plus tôt lorsqu'une correspondance est impossible
- Évitez les opérations coûteuses comme les appels d'API ou les expressions régulières complexes
- Envisagez de mettre en cache les résultats pour les motifs d'URL répétés

Si les matchers personnalisés répondent avec élégance à des besoins de routing complexes, en abuser peut rendre la configuration des routes plus difficile à comprendre et à maintenir. Réservez-les aux scénarios où la correspondance de chemin standard est réellement insuffisante.
