<!-- source: f382976c7bbdde15b199e482d2da30c22f26fa0a -->
# Error boundaries dans le routing

IMPORTANT: Les error boundaries du router sont en [developer preview](reference/releases#developer-preview).

Lors du développement d'applications web, des erreurs d'exécution peuvent survenir pendant l'initialisation d'un composant, le chargement des données ou le rendu. Le router Angular vous permet d'isoler ces échecs à des routes individuelles grâce aux **error boundaries**, ce qui évite qu'un seul composant défaillant ne fasse tomber toute la coquille de l'application.

NOTE: Ce guide couvre la gestion des erreurs au niveau des routes. Pour capturer les erreurs dans des sections précises du template d'un composant, consultez le [guide sur le bloc `@boundary` des templates](/guide/templates/error-boundaries).

## Que sont les error boundaries du router ? {#what-are-router-error-boundaries}

Les error boundaries du router offrent une gestion déclarative des erreurs au niveau des router outlets. Quand un composant d'une route lève une erreur, comme une resource réseau en échec, une référence nulle inattendue dans un template ou une erreur pendant l'injection de dépendances, le router Angular capture l'erreur et affiche un composant de repli dédié (`errorComponent`) à la place de la vue défaillante.

Les outlets frères, les composants de mise en page parents et les en-têtes de navigation globaux restent entièrement interactifs et intacts.

## Configurer un composant d'erreur {#configure-an-error-component}

Pour afficher une interface de repli personnalisée quand une route échoue, ajoutez la propriété `errorComponent` à la définition de votre route :

```ts {header:"app.routes.ts"}
import {Routes} from '@angular/router';
import {ProductDetails} from './product-details';
import {ProductErrorFallback} from './product-error-fallback';

export const routes: Routes = [
  {
    path: 'products/:id',
    component: ProductDetails,
    errorComponent: ProductErrorFallback,
  },
];
```

Quand vous naviguez vers `/products/123`, si `ProductDetails` lève une erreur pendant sa création ou son rendu, le router Angular instancie `ProductErrorFallback` dans le `<router-outlet>` à la place.

NOTE: `errorComponent` ne prend effet qu'une fois les error boundaries activées. Consultez [Configuration globale des error boundaries](#global-error-boundary-configuration).

NOTE: Contrairement aux composants de route, qui peuvent être chargés en lazy loading avec `loadComponent`, l'`errorComponent` doit être chargé immédiatement dans votre configuration de routes.

---

## Créer un composant de repli en cas d'erreur {#create-an-error-fallback-component}

Un composant de repli en cas d'erreur est un composant Angular standard qui peut recevoir les détails de l'erreur capturée.

### Accéder aux détails de l'erreur avec un input {#access-error-details-with-an-input}

Vous pouvez recevoir l'objet d'erreur capturé en déclarant un input `error` :

```angular-ts {header:"product-error-fallback.ts"}
import {Component, input} from '@angular/core';

@Component({
  selector: 'app-product-error-fallback',
  template: `
    <div class="error-card">
      <h3>Failed to load product</h3>
      <p>{{ error()?.message }}</p>
    </div>
  `,
})
export class ProductErrorFallback {
  readonly error = input<Error>();
}
```

NOTE: Si votre configuration de routes inclut des paramètres, des query parameters ou des données de route nommés `error`, l'`Error` d'exécution capturée a toujours la priorité sur l'input `error` d'un composant d'erreur.

IMPORTANT: Si l'`errorComponent` lève lui-même une erreur pendant sa création ou son rendu, le `RouterOutlet` ne la capture pas une seconde fois. L'exception remonte hors de l'outlet jusqu'au `ErrorHandler` racine (ou à un [bloc `@boundary` de template](/guide/templates/error-boundaries) englobant) pour éviter les boucles d'erreur infinies.

## Liaison des paramètres et des données de route {#route-parameter-and-data-binding}

Avec `withComponentInputBinding()`, le router Angular lie aussi les paramètres de route et les données statiques aux inputs de l'`errorComponent`. Cela vous permet d'afficher des informations contextuelles dans votre interface d'erreur :

```angular-ts {header:"product-error-fallback.ts"}
import {Component, input} from '@angular/core';

@Component({
  selector: 'app-product-error-fallback',
  template: `
    <div class="error-card">
      <h3>Failed to load product #{{ id() }}</h3>
      <p>{{ error()?.message }}</p>
    </div>
  `,
})
export class ProductErrorFallback {
  readonly id = input<string>();
  readonly error = input<Error>();
}
```

## Configuration globale des error boundaries {#global-error-boundary-configuration}

Activez les error boundaries du router pour votre application avec la fonctionnalité `withErrorBoundaries` de `provideRouter`. Sans elle, l'`errorComponent` d'une route est ignoré :

```ts {header:"app.config.ts"}
import {ApplicationConfig, inject} from '@angular/core';
import {provideRouter, withComponentInputBinding, withErrorBoundaries} from '@angular/router';
import {routes} from './app.routes';
import {GlobalErrorFallback} from './global-error-fallback';
import {AnalyticsService} from './analytics.service';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(
      routes,
      withComponentInputBinding(),
      withErrorBoundaries({
        // Default fallback component for routes that do not define their own errorComponent
        defaultErrorComponent: GlobalErrorFallback,

        // Global telemetry hook invoked whenever a route error is caught
        onError: (error, details) => {
          inject(AnalyticsService).logException(error, details);
        },
      }),
    ),
  ],
};
```

### Options {#options}

| Option | Type | Description |
| ----------------------- | ------------------------------------------------ | ----------- |
| `defaultErrorComponent` | `Type<unknown>` | Composant de repli par défaut à afficher quand une route sans `errorComponent` propre lève une erreur. |
| `onError` | `(error: Error, details?: ErrorDetails) => void` | Callback global invoqué chaque fois qu'une erreur est capturée par une error boundary du router. Idéal pour envoyer de la télémétrie à des services de supervision. |

---

## Rediriger avec `RedirectCommand` {#redirecting-with-redirectcommand}

Quand un échec d'authentification ou une condition sur les données impose de naviguer vers une autre page, les composants et les resources de route peuvent lever une `RedirectCommand` :

```angular-ts {header:"user-profile.ts"}
import {Component, inject, resource} from '@angular/core';
import {Router, RedirectCommand} from '@angular/router';

@Component({
  selector: 'app-user-profile',
  template: `...`,
})
export class UserProfile {
  private router = inject(Router);

  userData = resource({
    loader: async () => {
      const response = await fetch('/api/user');
      if (response.status === 401) {
        // Automatically redirects to /login when unauthorized
        throw new RedirectCommand(this.router.parseUrl('/login'));
      }
      return response.json();
    },
  });
}
```

Quand une `RedirectCommand` est levée :

1. `RouterOutlet` capture la commande et lance la navigation vers la destination de redirection indiquée.
2. S'il existe des outlets imbriqués ou des error boundaries de template, Angular déduplique la commande pour que la navigation ne soit déclenchée qu'une seule fois.
3. Le composant d'erreur (s'il est configuré) s'affiche pendant la transition.

NOTE: Si une `RedirectCommand` est levée et qu'aucun `errorComponent` n'est configuré, le router lance la navigation et avale volontairement l'erreur pour éviter qu'elle ne soit journalisée comme un plantage par l'`ErrorHandler` global.

## Portée des error boundaries et erreurs de navigation {#error-boundary-scope-vs-navigation-errors}

Il est utile de comprendre la distinction entre les **erreurs de la couche vue** et les **erreurs du pipeline de navigation** :

- **Erreurs de la couche vue (error boundaries)** : elles surviennent dans les constructeurs de composants, l'injection de dépendances, les lifecycle hooks (`ngOnInit`), les expressions de template et les loaders de `resource()` des composants. Elles sont capturées par `RouterOutlet`, qui affiche l'`errorComponent`.
- **Erreurs du pipeline de navigation (`NavigationError`)** : elles surviennent avant l'activation de la route, par exemple quand une guard `canActivate` / `canMatch` ou une fonction de données `resolve` échoue. Elles interrompent la transition de navigation avant d'atteindre l'outlet et peuvent être personnalisées avec `withNavigationErrorHandler`.

NOTE: Quand une error boundary affiche un `errorComponent`, les guards `canDeactivate` configurées pour cette route s'exécutent lors de la navigation vers une autre page. La guard reçoit l'instance active de l'`errorComponent` comme paramètre `component`. Si votre guard accède à des propriétés ou méthodes propres au composant, vérifiez le type du composant ou gérez correctement l'instance du composant d'erreur.

## Prochaines étapes {#next-steps}

<docs-pill-row>
  <docs-pill href="/guide/templates/error-boundaries" title="Error boundaries de template"/>
  <docs-pill href="/guide/routing/show-routes-with-outlets" title="Afficher le contenu de vos routes avec les outlets"/>
  <docs-pill href="/guide/routing/redirecting-routes" title="Rediriger des routes"/>
  <docs-pill href="/guide/routing/data-fetching-with-resources" title="Récupérer des données avec les resources"/>
</docs-pill-row>
