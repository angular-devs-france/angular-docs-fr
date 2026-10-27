<!-- source: 16f8323fc28955de509c0b4b5471fdf450e77063 -->
# Erreurs non gérées dans Angular

Pendant l'exécution de votre application Angular, une partie de votre code peut lever une erreur. Si elles ne sont pas gérées, ces erreurs peuvent entraîner un comportement inattendu et une interface qui ne répond plus. Ce guide explique comment Angular traite les erreurs qui ne sont pas explicitement interceptées par le code de votre application. Pour écrire votre propre logique de gestion des erreurs au sein de votre application, consultez les bonnes pratiques de gestion des erreurs en JavaScript et en Angular.

Un principe fondamental de la stratégie de gestion des erreurs d'Angular est que les erreurs doivent, autant que possible, remonter aux développeurs à l'endroit de l'appel. Cette approche garantit que le code qui a initié une opération dispose du contexte nécessaire pour comprendre l'erreur, la gérer de façon appropriée et décider de l'état adéquat de l'application. En rendant les erreurs visibles à leur origine, les développeurs peuvent mettre en place une gestion des erreurs propre à l'opération qui a échoué, avec accès aux informations utiles pour récupérer ou fournir un retour informatif à l'utilisateur final. Cela permet aussi d'éviter le « code smell » de l'« erreur trop générale », où les erreurs sont signalées sans contexte suffisant pour en comprendre la cause.

Par exemple, considérez un composant qui récupère les données d'un utilisateur depuis une API. Le code chargé de l'appel à l'API doit inclure une gestion des erreurs (par exemple avec un bloc `try...catch` ou l'opérateur `catchError` de RxJS) pour gérer les éventuels problèmes réseau ou les erreurs renvoyées par l'API. Cela permet au composant d'afficher un message d'erreur compréhensible ou de relancer la requête, plutôt que de laisser l'erreur se propager sans être gérée.

## Les erreurs non gérées sont signalées à l'`ErrorHandler` {#unhandled-errors-are-reported-to-the-errorhandler}

Angular signale les erreurs non gérées à l'`ErrorHandler` racine de l'application. Si vous fournissez un `ErrorHandler` personnalisé, fournissez-le dans votre `ApplicationConfig` lors de l'appel à `bootstrapApplication`.

Quand vous créez une application Angular, vous écrivez souvent du code appelé automatiquement _par_ le framework. Par exemple, Angular se charge d'appeler le constructeur et les méthodes de cycle de vie d'un composant quand ce composant apparaît dans un template. Quand le framework exécute votre code, vous n'avez aucun endroit raisonnable où ajouter un bloc `try` pour gérer proprement les erreurs. Dans ce type de situation, Angular intercepte les erreurs et les envoie à l'`ErrorHandler`.

Angular n'intercepte _pas_ les erreurs au sein des API appelées directement par votre code. Par exemple, si vous avez un service dont une méthode lève une erreur et que vous appelez cette méthode dans votre composant, Angular n'interceptera pas automatiquement cette erreur. C'est à vous de la gérer avec des mécanismes comme `try...catch`.

Angular intercepte les erreurs _asynchrones_ issues des promises ou des observables de l'utilisateur uniquement quand :

- il existe un contrat explicite selon lequel Angular attend le résultat de l'opération asynchrone et l'utilise, et
- les erreurs ne sont pas exposées dans la valeur renvoyée ou dans l'état.

Par exemple, `AsyncPipe` et `PendingTasks.run` transmettent les erreurs à l'`ErrorHandler`, alors que `resource` expose l'erreur dans les propriétés `status` et `error`.

Les erreurs qu'Angular signale à l'`ErrorHandler` sont des erreurs _inattendues_. Ces erreurs peuvent être irrécupérables ou indiquer que l'état de l'application est corrompu. Les applications doivent, autant que possible, gérer les erreurs avec des blocs `try` ou des opérateurs de gestion des erreurs appropriés (comme `catchError` dans RxJS) là où l'erreur se produit, plutôt que de compter sur l'`ErrorHandler`, qui est le plus souvent, et à juste titre, utilisé uniquement comme mécanisme pour remonter les erreurs potentiellement fatales à l'infrastructure de suivi des erreurs et de journalisation.

```ts
export class GlobalErrorHandler implements ErrorHandler {
  private readonly analyticsService = inject(AnalyticsService);
  private readonly router = inject(Router);

  handleError(error: any) {
    const url = this.router.url;
    const errorMessage = error?.message ?? 'unknown';

    this.analyticsService.trackEvent({
      eventName: 'exception',
      description: `Screen: ${url} | ${errorMessage}`,
    });

    console.error(GlobalErrorHandler.name, {error});
  }
}
```

### `TestBed` relance les erreurs par défaut {#testbed-rethrows-errors-by-default}

Dans de nombreux cas, l'`ErrorHandler` peut se contenter de journaliser les erreurs et laisser l'application continuer à s'exécuter. Dans les tests, en revanche, vous voulez presque toujours faire remonter ces erreurs. Le `TestBed` d'Angular relance les erreurs inattendues pour garantir que les erreurs interceptées par le framework ne puissent pas être manquées ou ignorées par inadvertance. Dans de rares cas, un test peut chercher précisément à vérifier que les erreurs ne rendent pas l'application inactive et ne la font pas planter. Dans ces situations, vous pouvez [configurer `TestBed` pour _ne pas_ relancer les erreurs de l'application](api/core/testing/TestModuleMetadata#rethrowApplicationErrors) avec `TestBed.configureTestingModule({rethrowApplicationErrors: false})`.

### Erreurs levées pendant le démarrage de votre application {#errors-thrown-while-your-app-is-still-starting-up}

Il existe un bref moment pendant lequel Angular ne peut pas encore envoyer les erreurs à votre `ErrorHandler` : tant qu'il est encore en train de créer le module racine ou le composant racine de votre application. Angular a besoin que cette instance racine existe pour pouvoir trouver l'`ErrorHandler` que vous avez fourni, donc une erreur levée avant ce moment n'a nulle part où aller. Elle se comporte comme une erreur non interceptée classique au lieu d'être signalée via l'`ErrorHandler`.

C'est avec [Angular Elements](guide/elements) que vous risquez le plus de rencontrer ce cas. Si vous définissez un custom element alors que sa balise est déjà présente sur la page, le navigateur le met à niveau immédiatement, en exécutant le constructeur et le `ngOnInit` du composant pendant cette phase de démarrage précoce. Toute erreur levée à ce moment peut être perdue.

Si cela vous arrive, déplacez le code qui lève l'erreur pour qu'il s'exécute une fois le démarrage de votre application terminé, par exemple :

- Encapsulez-le dans `setTimeout(() => /* your code */)` pour qu'il s'exécute dans son propre tour plutôt que pendant le démarrage.
- Fournissez-le en tant qu'`APP_BOOTSTRAP_LISTENER` au lieu de l'exécuter depuis un constructeur.
- Définissez vos custom elements dans `ngDoBootstrap` plutôt que dans le constructeur de votre module racine.

Activer `provideBrowserGlobalErrorListeners()` (voir ci-dessous) peut aussi aider à intercepter ces erreurs, puisqu'elles atteignent tout de même le navigateur en tant qu'erreurs non interceptées.

## Listeners d'erreurs globaux {#global-error-listeners}

Les erreurs qui ne sont interceptées ni par le code de l'application ni par l'instance d'application du framework peuvent atteindre la portée globale. Les erreurs qui atteignent la portée globale peuvent avoir des conséquences imprévues si elles ne sont pas prises en compte. Dans les environnements hors navigateur, elles peuvent faire planter le processus. Dans le navigateur, ces erreurs peuvent ne pas être signalées et les visiteurs du site peuvent voir les erreurs dans la console du navigateur. Angular fournit des listeners globaux pour ces deux environnements afin de traiter ces problèmes.

### Rendu côté client {#client-side-rendering}

Ajouter [`provideBrowserGlobalErrorListeners()`](/api/core/provideBrowserGlobalErrorListeners) à l'[ApplicationConfig](guide/di/defining-dependency-providers#application-bootstrap) ajoute les listeners `'error'` et `'unhandledrejection'` à la fenêtre du navigateur et transmet ces erreurs à l'`ErrorHandler`. L'Angular CLI génère par défaut les nouvelles applications avec ce provider. L'équipe Angular recommande de gérer ces erreurs globales dans la plupart des applications, soit avec les listeners intégrés du framework, soit avec vos propres listeners personnalisés. Si vous fournissez des listeners personnalisés, vous pouvez retirer `provideBrowserGlobalErrorListeners`.

### Rendu côté serveur et rendu hybride {#server-side-and-hybrid-rendering}

Quand vous utilisez [Angular avec le SSR](guide/ssr), Angular ajoute automatiquement les listeners `'unhandledRejection'` et `'uncaughtException'` au processus serveur. Ces handlers empêchent le serveur de planter et journalisent à la place les erreurs capturées dans la console.

IMPORTANT: Si l'application utilise Zone.js, seul le handler `'unhandledRejection'` est ajouté. Quand Zone.js est présent, les erreurs survenant dans la zone de l'application sont déjà transmises à l'`ErrorHandler` de l'application et n'atteignent pas le processus serveur.
