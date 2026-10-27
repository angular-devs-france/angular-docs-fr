<!-- source: c8e8b7fd0597d785db0ae3c2a8330e7958d7875e -->
# Cycle de vie et événements du router

Le router d'Angular fournit un ensemble complet de lifecycle hooks et d'événements qui vous permettent de réagir aux changements de navigation et d'exécuter une logique personnalisée pendant le processus de routing.

## Événements courants du router {#common-router-events}

Le router d'Angular émet des événements de navigation auxquels vous pouvez vous abonner pour suivre le cycle de vie de la navigation. Ces événements sont disponibles via l'observable `Router.events`. Cette section présente les événements courants du cycle de vie du routing pour le suivi de la navigation et des erreurs (dans l'ordre chronologique).

| Événements | Description |
| --------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| [`NavigationStart`](api/router/NavigationStart) | Se produit au début de la navigation et contient l'URL demandée. |
| [`RoutesRecognized`](api/router/RoutesRecognized) | Se produit une fois que le router a déterminé quelle route correspond à l'URL et contient les informations d'état de la route. |
| [`GuardsCheckStart`](api/router/GuardsCheckStart) | Démarre la phase des guards de route. Le router évalue les guards de route comme `canActivate` et `canDeactivate`. |
| [`GuardsCheckEnd`](api/router/GuardsCheckEnd) | Signale la fin de l'évaluation des guards. Contient le résultat (autorisé/refusé). |
| [`ResolveStart`](api/router/ResolveStart) | Démarre la phase de résolution des données. Les resolvers et les resources de route commencent à récupérer les données. |
| [`ResolveEnd`](api/router/ResolveEnd) | La résolution des données est terminée. Toutes les données requises et les resources bloquantes deviennent disponibles. |
| [`NavigationEnd`](api/router/NavigationEnd) | Événement final lorsque la navigation se termine avec succès. Le router met à jour l'URL. |
| [`NavigationSkipped`](api/router/NavigationSkipped) | Se produit lorsque le router ignore la navigation (par exemple, navigation vers la même URL). |

Voici les événements d'erreur courants :

| Événement | Description |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| [`NavigationCancel`](api/router/NavigationCancel) | Se produit lorsque le router annule la navigation, souvent parce qu'une guard a renvoyé false. |
| [`NavigationError`](api/router/NavigationError) | Se produit lorsque la navigation échoue, par exemple à cause de routes invalides, d'erreurs de resolver ou de resources bloquantes rejetées. |

Pour la liste de tous les événements du cycle de vie, consultez le [tableau complet de ce guide](#all-router-events).

## Comment s'abonner aux événements du router {#how-to-subscribe-to-router-events}

Pour exécuter du code lors d'événements précis du cycle de vie de la navigation, abonnez-vous à `router.events` et vérifiez l'instance de l'événement :

```ts
// Example of subscribing to router events
import {Component, inject, signal, effect} from '@angular/core';
import {Event, Router, NavigationStart, NavigationEnd} from '@angular/router';
import {takeUntilDestroyed} from '@angular/core/rxjs-interop';

@Component(/* ... */)
export class RouterEvents {
  private readonly router = inject(Router);

  constructor() {
    // Subscribe to router events and react to events
    this.router.events.pipe(takeUntilDestroyed()).subscribe((event: Event) => {
      if (event instanceof NavigationStart) {
        // Navigation starting
        console.log('Navigation starting:', event.url);
      }
      if (event instanceof NavigationEnd) {
        // Navigation completed
        console.log('Navigation completed:', event.url);
      }
    });
  }
}
```

NOTE: Le type [`Event`](api/router/Event) de `@angular/router` porte le même nom que le type global [`Event`](https://developer.mozilla.org/en-US/docs/Web/API/Event) habituel, mais il est différent du type [`RouterEvent`](api/router/RouterEvent).

## Comment déboguer les événements de routing {#how-to-debug-routing-events}

Déboguer des problèmes de navigation du router peut s'avérer difficile sans visibilité sur la séquence des événements. Angular fournit une fonctionnalité de débogage intégrée qui journalise tous les événements du router dans la console, ce qui vous aide à comprendre le déroulement de la navigation et à identifier où se produisent les problèmes.

Lorsque vous devez inspecter une séquence d'événements du router, vous pouvez activer la journalisation des événements de navigation internes à des fins de débogage. Pour cela, passez une option de configuration (`withDebugTracing()`) qui active une journalisation détaillée de tous les événements de routing dans la console.

```ts
import {provideRouter, Routes, withDebugTracing} from '@angular/router';
import {bootstrapApplication} from '@angular/platform-browser';

const appRoutes: Routes = [];
bootstrapApplication(App, {
  providers: [provideRouter(appRoutes, withDebugTracing())],
});
```

Pour en savoir plus, consultez la documentation officielle de [`withDebugTracing`](api/router/withDebugTracing).

## Cas d'usage courants {#common-use-cases}

Les événements du router permettent de nombreuses fonctionnalités pratiques dans les applications réelles. Voici quelques patterns courants utilisés avec les événements du router.

### Indicateurs de chargement {#loading-indicators}

Affichez des indicateurs de chargement pendant la navigation :

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

### Suivi analytique {#analytics-tracking}

Suivez les pages vues à des fins d'analyse :

```ts
import {takeUntilDestroyed} from '@angular/core/rxjs-interop';
import {inject, DestroyRef, Service} from '@angular/core';
import {Router, NavigationEnd} from '@angular/router';

@Service()
export class AnalyticsService {
  private router = inject(Router);
  private destroyRef = inject(DestroyRef);

  startTracking() {
    this.router.events.pipe(takeUntilDestroyed(this.destroyRef)).subscribe((event) => {
      // Track page views when URL changes
      if (event instanceof NavigationEnd) {
        // Send page view to analytics
        this.analytics.trackPageView(event.url);
      }
    });
  }

  private analytics = {
    trackPageView: (url: string) => {
      console.log('Page view tracked:', url);
    },
  };
}
```

### Gestion des erreurs {#error-handling}

Gérez les erreurs de navigation proprement et informez l'utilisateur :

```angular-ts
import {Component, inject, signal} from '@angular/core';
import {
  Router,
  NavigationStart,
  NavigationError,
  NavigationCancel,
  NavigationCancellationCode,
} from '@angular/router';
import {takeUntilDestroyed} from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-error-handler',
  template: `
    @if (errorMessage()) {
      <div class="error-banner">
        {{ errorMessage() }}
        <button (click)="dismissError()">Dismiss</button>
      </div>
    }
  `,
})
export class ErrorHandler {
  private router = inject(Router);
  readonly errorMessage = signal('');

  constructor() {
    this.router.events.pipe(takeUntilDestroyed()).subscribe((event) => {
      if (event instanceof NavigationStart) {
        this.errorMessage.set('');
      } else if (event instanceof NavigationError) {
        console.error('Navigation error:', event.error);
        this.errorMessage.set('Failed to load page. Please try again.');
      } else if (event instanceof NavigationCancel) {
        console.warn('Navigation cancelled:', event.reason);
        if (event.code === NavigationCancellationCode.GuardRejected) {
          this.errorMessage.set('Access denied. Please check your permissions.');
        }
      }
    });
  }

  dismissError() {
    this.errorMessage.set('');
  }
}
```

## Tous les événements du router {#all-router-events}

Pour référence, voici la liste complète de tous les événements du router disponibles dans Angular. Ces événements sont regroupés par catégorie et listés dans l'ordre où ils se produisent généralement pendant la navigation.

### Événements de navigation {#navigation-events}

Ces événements suivent le processus de navigation principal, du démarrage jusqu'à la reconnaissance des routes, à la vérification des guards et à la résolution des données. Ils donnent de la visibilité sur chaque phase du cycle de vie de la navigation.

| Événement | Description |
| --------------------------------------------------------- | --------------------------------------------------------------- |
| [`NavigationStart`](api/router/NavigationStart) | Se produit au démarrage de la navigation |
| [`RouteConfigLoadStart`](api/router/RouteConfigLoadStart) | Se produit avant le chargement en lazy loading d'une configuration de route |
| [`RouteConfigLoadEnd`](api/router/RouteConfigLoadEnd) | Se produit après le chargement d'une configuration de route en lazy loading |
| [`RoutesRecognized`](api/router/RoutesRecognized) | Se produit lorsque le router analyse l'URL et reconnaît les routes |
| [`GuardsCheckStart`](api/router/GuardsCheckStart) | Se produit au début de la phase des guards |
| [`GuardsCheckEnd`](api/router/GuardsCheckEnd) | Se produit à la fin de la phase des guards |
| [`ResolveStart`](api/router/ResolveStart) | Se produit au début de la phase de résolution |
| [`ResolveEnd`](api/router/ResolveEnd) | Se produit à la fin de la phase de résolution |

### Événements d'activation {#activation-events}

Ces événements se produisent pendant la phase d'activation, lorsque les composants des routes sont instanciés et initialisés. Les événements d'activation sont déclenchés pour chaque route de l'arbre de routes, y compris les routes parentes et enfants.

| Événement | Description |
| --------------------------------------------------------- | --------------------------------------------- |
| [`ActivationStart`](api/router/ActivationStart) | Se produit au début de l'activation d'une route |
| [`ChildActivationStart`](api/router/ChildActivationStart) | Se produit au début de l'activation d'une route enfant |
| [`ActivationEnd`](api/router/ActivationEnd) | Se produit à la fin de l'activation d'une route |
| [`ChildActivationEnd`](api/router/ChildActivationEnd) | Se produit à la fin de l'activation d'une route enfant |

### Événements de fin de navigation {#navigation-completion-events}

Ces événements représentent l'issue finale d'une tentative de navigation. Chaque navigation se termine par exactement un de ces événements, qui indique si elle a réussi, a été annulée, a échoué ou a été ignorée.

| Événement | Description |
| --------------------------------------------------- | ------------------------------------------------------------------- |
| [`NavigationEnd`](api/router/NavigationEnd) | Se produit lorsque la navigation se termine avec succès |
| [`NavigationCancel`](api/router/NavigationCancel) | Se produit lorsque le router annule la navigation |
| [`NavigationError`](api/router/NavigationError) | Se produit lorsque la navigation échoue à cause d'une erreur inattendue |
| [`NavigationSkipped`](api/router/NavigationSkipped) | Se produit lorsque le router ignore la navigation (par exemple, navigation vers la même URL) |

### Autres événements {#other-events}

Il existe un événement supplémentaire qui se produit en dehors du cycle de vie principal de la navigation, mais qui fait tout de même partie du système d'événements du router.

| Événement | Description |
| ----------------------------- | ----------------------- |
| [`Scroll`](api/router/Scroll) | Se produit pendant le défilement |

## Prochaines étapes {#next-steps}

Découvrez-en plus sur les [guards de route](/guide/routing/route-guards), la [récupération de données avec les resources](/guide/routing/data-fetching-with-resources) et les [tâches courantes du router](/guide/routing/common-router-tasks).
