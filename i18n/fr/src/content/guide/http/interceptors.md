<!-- source: cbf599173cc4a7a0b1fdbed2b131c3d84d6e5bea -->
# Intercepteurs

`HttpClient` prend en charge une forme de middleware appelée _intercepteurs_.

TLDR: Les intercepteurs sont des middlewares qui permettent d'extraire des requêtes individuelles des patterns courants comme les nouvelles tentatives, la mise en cache, la journalisation et l'authentification.

`HttpClient` prend en charge deux types d'intercepteurs : fonctionnels et basés sur la DI. Nous recommandons d'utiliser des intercepteurs fonctionnels, car leur comportement est plus prévisible, en particulier dans les configurations complexes. Les exemples de ce guide utilisent des intercepteurs fonctionnels, et nous traitons les [intercepteurs basés sur la DI](#di-based-interceptors) dans une section dédiée à la fin.

## Intercepteurs {#interceptors}

Les intercepteurs sont généralement des fonctions que vous pouvez exécuter pour chaque requête et qui disposent de larges capacités pour agir sur le contenu et le déroulement global des requêtes et des réponses. Vous pouvez installer plusieurs intercepteurs, qui forment une chaîne d'intercepteurs dans laquelle chaque intercepteur traite la requête ou la réponse avant de la transmettre à l'intercepteur suivant de la chaîne.

Vous pouvez utiliser des intercepteurs pour mettre en œuvre divers patterns courants, par exemple :

- Ajouter des en-têtes d'authentification aux requêtes sortantes vers une API donnée.
- Relancer les requêtes en échec avec un backoff exponentiel.
- Mettre en cache les réponses pendant une certaine durée, ou jusqu'à leur invalidation par des mutations.
- Personnaliser l'analyse (parsing) des réponses.
- Mesurer les temps de réponse du serveur et les journaliser.
- Piloter des éléments d'interface comme un indicateur de chargement pendant les opérations réseau.
- Regrouper et traiter par lots les requêtes effectuées dans un certain intervalle de temps.
- Faire échouer automatiquement les requêtes après un délai ou un timeout configurable.
- Interroger régulièrement le serveur (polling) et rafraîchir les résultats.

## Définir un intercepteur {#defining-an-interceptor}

Dans sa forme de base, un intercepteur est une fonction qui reçoit la `HttpRequest` sortante et une fonction `next` représentant l'étape de traitement suivante dans la chaîne d'intercepteurs.

Par exemple, ce `loggingInterceptor` journalise l'URL de la requête sortante avec `console.log` avant de transmettre la requête :

```ts
export function loggingInterceptor(
  req: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  console.log(req.url);
  return next(req);
}
```

Pour que cet intercepteur intercepte réellement les requêtes, vous devez configurer `HttpClient` pour qu'il l'utilise.

## Configurer les intercepteurs {#configuring-interceptors}

Vous déclarez l'ensemble des intercepteurs à utiliser lors de la configuration de `HttpClient` via l'injection de dépendances, à l'aide de la fonctionnalité `withInterceptors` :

```ts
bootstrapApplication(App, {
  providers: [provideHttpClient(withInterceptors([loggingInterceptor, cachingInterceptor]))],
});
```

Les intercepteurs que vous configurez sont chaînés dans l'ordre où vous les avez listés dans les providers. Dans l'exemple ci-dessus, le `loggingInterceptor` traiterait la requête puis la transmettrait au `cachingInterceptor`.

### Intercepter les événements de réponse {#intercepting-response-events}

Un intercepteur peut transformer le flux `Observable` de `HttpEvent` renvoyé par `next` afin d'accéder à la réponse ou de la manipuler. Comme ce flux inclut tous les événements de réponse, il peut être nécessaire d'inspecter le `.type` de chaque événement pour identifier l'objet de réponse final.

```ts
export function loggingInterceptor(
  req: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  return next(req).pipe(
    tap((event) => {
      if (event.type === HttpEventType.Response) {
        console.log(req.url, 'returned a response with status', event.status);
      }
    }),
  );
}
```

TIP: Les intercepteurs associent naturellement les réponses à leurs requêtes sortantes, car ils transforment le flux de réponse dans une closure qui capture l'objet de la requête.

## Modifier les requêtes {#modifying-requests}

La plupart des aspects des instances de `HttpRequest` et `HttpResponse` sont _immuables_, et les intercepteurs ne peuvent pas les modifier directement. À la place, les intercepteurs appliquent des mutations en clonant ces objets avec l'opération `.clone()`, et en précisant quelles propriétés doivent être modifiées dans la nouvelle instance. Cela peut impliquer d'effectuer des mises à jour immuables sur la valeur elle-même (comme `HttpHeaders` ou `HttpParams`).

Par exemple, pour ajouter un en-tête à une requête :

```ts
const reqWithHeader = req.clone({
  headers: req.headers.set('X-New-Header', 'new header value'),
});
```

Cette immuabilité permet à la plupart des intercepteurs d'être idempotents si la même `HttpRequest` est soumise plusieurs fois à la chaîne d'intercepteurs. Cela peut se produire pour plusieurs raisons, notamment quand une requête est relancée après un échec.

CRITICAL: Le corps d'une requête ou d'une réponse n'est **pas** protégé contre les mutations profondes. Si un intercepteur doit modifier le corps, veillez à gérer son exécution multiple sur la même requête.

## Injection de dépendances dans les intercepteurs {#dependency-injection-in-interceptors}

Les intercepteurs s'exécutent dans le _contexte d'injection_ de l'injector qui les a enregistrés, et peuvent utiliser l'API [`inject`](/api/core/inject) d'Angular pour récupérer des dépendances.

Par exemple, supposons qu'une application dispose d'un service appelé `AuthService`, qui crée des jetons d'authentification pour les requêtes sortantes. Un intercepteur peut injecter et utiliser ce service :

```ts
export function authInterceptor(req: HttpRequest<unknown>, next: HttpHandlerFn) {
  // Inject the current `AuthService` and use it to get an authentication token:
  const authToken = inject(AuthService).getAuthToken();

  // Clone the request to add the authentication header.
  const newReq = req.clone({
    headers: req.headers.append('X-Authentication-Token', authToken),
  });
  return next(newReq);
}
```

## Métadonnées de requête et de réponse {#request-and-response-metadata}

Il est souvent utile d'inclure dans une requête des informations qui ne sont pas envoyées au backend, mais qui sont spécifiquement destinées aux intercepteurs. Les `HttpRequest` disposent d'un objet `.context` qui stocke ce type de métadonnées sous la forme d'une instance de `HttpContext`. Cet objet fonctionne comme une map typée, avec des clés de type `HttpContextToken`.

Pour illustrer le fonctionnement de ce système, utilisons des métadonnées pour contrôler si un intercepteur de mise en cache est activé pour une requête donnée.

### Définir des jetons de contexte {#defining-context-tokens}

Pour stocker, dans la map `.context` d'une requête, le fait que l'intercepteur de mise en cache doive ou non mettre en cache cette requête, définissez un nouveau `HttpContextToken` qui servira de clé :

```ts
export const CACHING_ENABLED = new HttpContextToken<boolean>(() => true);
```

La fonction fournie crée la valeur par défaut du jeton pour les requêtes qui ne lui ont pas explicitement attribué de valeur. L'utilisation d'une fonction garantit que, si la valeur du jeton est un objet ou un tableau, chaque requête obtient sa propre instance.

### Lire le jeton dans un intercepteur {#reading-the-token-in-an-interceptor}

Un intercepteur peut ensuite lire le jeton et choisir d'appliquer ou non une logique de mise en cache en fonction de sa valeur :

```ts
export function cachingInterceptor(req: HttpRequest<unknown>, next: HttpHandlerFn): Observable<HttpEvent<unknown>> {
  if (req.context.get(CACHING_ENABLED)) {
    // apply caching logic
    return ...;
  } else {
    // caching has been disabled for this request
    return next(req);
  }
}
```

### Définir des jetons de contexte lors d'une requête {#setting-context-tokens-when-making-a-request}

Lorsque vous effectuez une requête via l'API `HttpClient`, vous pouvez fournir des valeurs pour des `HttpContextToken` :

```ts
const data$ = http.get('/sensitive/data', {
  context: new HttpContext().set(CACHING_ENABLED, false),
});
```

Les intercepteurs peuvent lire ces valeurs depuis le `HttpContext` de la requête.

### Le contexte de requête est mutable {#the-request-context-is-mutable}

Contrairement aux autres propriétés des `HttpRequest`, le `HttpContext` associé est _mutable_. Si un intercepteur modifie le contexte d'une requête qui est ensuite relancée, ce même intercepteur observera la mutation du contexte lors de sa nouvelle exécution. C'est utile pour transmettre un état d'une tentative à l'autre si nécessaire.

## Réponses synthétiques {#synthetic-responses}

La plupart des intercepteurs se contentent d'invoquer le handler `next` tout en transformant la requête ou la réponse, mais ce n'est pas strictement obligatoire. Cette section présente plusieurs façons dont un intercepteur peut intégrer un comportement plus avancé.

Les intercepteurs ne sont pas tenus d'invoquer `next`. Ils peuvent à la place choisir de construire des réponses par un autre mécanisme, par exemple à partir d'un cache ou en envoyant la requête par un mécanisme alternatif.

Il est possible de construire une réponse avec le constructeur `HttpResponse` :

```ts
const resp = new HttpResponse({
  body: 'response body',
});
```

## Exploiter les informations de redirection {#working-with-redirect-information}

Lorsque `HttpClient` utilise le backend fetch, les réponses incluent une propriété `redirected` qui indique si la réponse résulte d'une redirection. Cette propriété est alignée sur la spécification native de l'API Fetch et peut être utile dans les intercepteurs pour gérer les scénarios de redirection.

Un intercepteur peut accéder aux informations de redirection et agir en conséquence :

```ts
export function redirectTrackingInterceptor(
  req: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  return next(req).pipe(
    tap((event) => {
      if (event.type === HttpEventType.Response && event.redirected) {
        console.log('Request to', req.url, 'was redirected to', event.url);
        // Handle redirect logic - maybe update analytics, security checks, etc.
      }
    }),
  );
}
```

Vous pouvez aussi utiliser les informations de redirection pour mettre en œuvre une logique conditionnelle dans vos intercepteurs :

```ts
export function authRedirectInterceptor(
  req: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  return next(req).pipe(
    tap((event) => {
      if (event.type === HttpEventType.Response && event.redirected) {
        // Check if we were redirected to a login page
        if (event.url?.includes('/login')) {
          // Handle authentication redirect
          handleAuthRedirect();
        }
      }
    }),
  );
}
```

## Exploiter les types de réponse {#working-with-response-types}

Lorsque `HttpClient` utilise le backend fetch, les réponses incluent une propriété `responseType` qui indique comment le navigateur a traité la réponse en fonction des politiques CORS et du mode de la requête. Cette propriété est alignée sur la spécification native de l'API Fetch et fournit des informations précieuses pour déboguer les problèmes CORS et comprendre l'accessibilité des réponses.

La propriété `responseType` de la réponse peut prendre les valeurs suivantes :

- `'basic'` - Réponse de même origine avec tous les en-têtes accessibles
- `'cors'` - Réponse cross-origin avec des en-têtes CORS correctement configurés
- `'opaque'` - Réponse cross-origin sans CORS, les en-têtes et le corps peuvent être limités
- `'opaqueredirect'` - Réponse à une requête redirigée en mode no-cors
- `'error'` - Une erreur réseau s'est produite

Un intercepteur peut utiliser les informations de type de réponse pour le débogage CORS et la gestion des erreurs :

```ts
export function responseTypeInterceptor(
  req: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  return next(req).pipe(
    tap((event) => {
      if (event.type === HttpEventType.Response) {
        // Handle different response types appropriately
        switch (event.responseType) {
          case 'opaque':
            // Limited access to response data
            console.warn('Limited response data due to CORS policy');
            break;
          case 'cors':
          case 'basic':
            // Full access to response data
            break;
          case 'error':
            // Handle network errors
            console.error('Network error in response');
            break;
        }
      }
    }),
  );
}
```

## Intercepteurs basés sur la DI {#di-based-interceptors}

`HttpClient` prend aussi en charge des intercepteurs définis comme des classes injectables et configurés via le système de DI. Les capacités des intercepteurs basés sur la DI sont identiques à celles des intercepteurs fonctionnels, mais le mécanisme de configuration diffère.

Un intercepteur basé sur la DI est une classe injectable qui implémente l'interface `HttpInterceptor` :

```ts
@Injectable()
export class LoggingInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, handler: HttpHandler): Observable<HttpEvent<any>> {
    console.log('Request URL: ' + req.url);
    return handler.handle(req);
  }
}
```

Les intercepteurs basés sur la DI se configurent via un multi provider d'injection de dépendances :

```ts
bootstrapApplication(App, {
  providers: [
    provideHttpClient(
      // DI-based interceptors must be explicitly enabled.
      withInterceptorsFromDi(),
    ),

    {provide: HTTP_INTERCEPTORS, useClass: LoggingInterceptor, multi: true},
  ],
});
```

Les intercepteurs basés sur la DI s'exécutent dans l'ordre d'enregistrement de leurs providers. Dans une application dotée d'une configuration de DI étendue et hiérarchique, cet ordre peut être très difficile à prévoir.
