<!-- source: 3a3bac481c7c34b131b840087cfe5e908b80fe74 -->
# Périmètres d'erreur avec `@boundary`

IMPORTANT: `@boundary` est en [developer preview](reference/releases#developer-preview).

Les templates Angular prennent en charge les périmètres d'erreur (error boundaries) pour gérer proprement les erreurs d'exécution qui surviennent pendant l'affichage et la détection de changements.

Les périmètres d'erreur empêchent la défaillance d'un seul composant de faire planter toute l'application et permettent d'afficher une interface de repli à l'utilisateur.

## Intercepter les erreurs avec `@boundary` et `@error` {#catching-errors-with-boundary-and-error}

Le bloc `@boundary` englobe une section de votre template. Si un composant ou une directive à l'intérieur de ce périmètre lève une erreur pendant l'initialisation ou la détection de changements, le framework intercepte l'erreur et affiche le bloc `@error` à la place.

```angular-html
@boundary {
  <app-risky-component />
} @error {
  <p>Something went wrong!</p>
}
```

## Accéder à l'objet d'erreur {#accessing-the-error-object}

Vous pouvez accéder à l'erreur interceptée via la variable implicite `$error` :

```angular-html
@boundary {
  <app-risky-component />
} @error {
  <p>Error occurred: {{ $error.message }}</p>
}
```

## Réinitialiser le périmètre {#resetting-the-boundary}

Vous pouvez tenter d'afficher à nouveau le contenu du `@boundary` en appelant la fonction implicite `$reset` dans le bloc `@error`. Lorsqu'elle est appelée, elle réinitialise l'état du périmètre et tente d'afficher à nouveau le contenu d'origine.

```angular-html
@boundary {
  <app-flaky-component />
} @error {
  <p>Loading failed.</p>
  <button (click)="$reset()">Try again</button>
}
```

## Gestion conditionnelle des erreurs avec `when` {#conditional-error-handling-with-when}

Vous pouvez utiliser des clauses `when` pour gérer de manière conditionnelle des types d'erreurs précis, ce qui vous permet de fournir différentes interfaces de repli. Angular évalue cette condition lorsqu'il intercepte une erreur.

```angular-html
@boundary {
  <app-chart-dashboard />
} @error (let err; reset = $reset; when isRenderError(err)) {
  <p>Network issue. Check your connection.</p>
  <button (click)="reset()">Retry</button>
} @error {
  <p>An unexpected error occurred: {{ $error.message }}</p>
}
```

Ordonnez vos blocs `@error` du plus spécifique au moins spécifique, car Angular évalue les clauses `when` dans l'ordre et utilise la première qui s'évalue à vrai. Un dernier bloc `@error` sans clause `when` sert de repli pour toutes les autres erreurs.

## Intégration avec le gestionnaire d'erreurs global {#global-error-handler-integration}

Lorsqu'un périmètre intercepte une erreur, Angular peut tout de même notifier l'`ErrorHandler` global. Vous pouvez implémenter le hook optionnel `onViewError` dans votre `ErrorHandler` personnalisé pour journaliser ces erreurs interceptées dans votre service de suivi des erreurs.

```ts
@Injectable()
export class MyErrorHandler implements ErrorHandler {
  handleError(error: any): void {
    // Handle uncaught errors
  }

  onViewError(error: Error, details: ErrorDetails): void {
    // Handle errors caught by a @boundary
    console.warn('Caught by boundary:', details.boundary);
    myErrorTrackingService.log(error);
  }
}
```

IMPORTANT: Si un bloc `@error` lève lui-même une erreur, celle-ci se propage au `@boundary` englobant suivant, ou Angular la traite comme une erreur d'application non gérée.

## Projection de contenu {#content-projection}

Si votre composant utilise la [projection de contenu](guide/components/content-projection), englober `<ng-content>` dans un `@boundary` n'intercepte pas les erreurs du contenu projeté. Ce contenu appartient à la vue qui le déclare, et non à la vue du composant qui le reçoit.

Par exemple, un composant conteneur avec le template suivant n'intercepte pas les erreurs des composants qui y sont projetés :

```angular-html {avoid}
@boundary {
  <ng-content />
} @error {
  <p>Something went wrong!</p>
}
```

Pour intercepter ces erreurs, englobez le composant conteneur et son contenu projeté dans un `@boundary` dans le template parent :

```angular-html {prefer}
@boundary {
  <app-wrapper>
    <app-risky-component />
  </app-wrapper>
} @error {
  <p>Something went wrong!</p>
}
```

## Vues dynamiques et gestion programmatique des erreurs {#dynamic-views-and-programmatic-error-handling}

La gestion des erreurs ne se limite pas à la syntaxe de template. Si vous créez des composants ou des vues intégrées dynamiquement, vous pouvez utiliser l'option `onError` pour gérer les erreurs. Consultez la section [Gérer les erreurs de rendu](guide/components/programmatic-rendering#handling-rendering-errors) du guide sur le rendu de composants par programmation pour plus d'informations.
