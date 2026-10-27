<!-- source: 3bc46ba97fe8dc1619843fd142c2d53ab7602514 -->
# Tester les requêtes

Comme pour toute dépendance externe, vous devez mocker le backend HTTP afin que vos tests puissent simuler l'interaction avec un serveur distant. La library `@angular/common/http/testing` fournit des outils pour capturer les requêtes effectuées par l'application, faire des assertions sur celles-ci et mocker les réponses pour émuler le comportement de votre backend.

La library de test est conçue pour un pattern dans lequel l'application exécute du code et effectue d'abord des requêtes. Le test vérifie ensuite que certaines requêtes ont été effectuées ou non, fait des assertions sur ces requêtes, et fournit enfin des réponses en « vidant » (flush) chaque requête attendue.

Enfin, les tests peuvent vérifier que l'application n'a effectué aucune requête inattendue.

## Configuration pour les tests {#setup-for-testing}

Pour commencer à tester l'utilisation de `HttpClient`, configurez `TestBed` et incluez `provideHttpClientTesting()` dans la configuration de votre test. `HttpClient` est fourni par l'environnement de test d'Angular, et `provideHttpClientTesting()` le configure pour utiliser un backend de test au lieu du vrai réseau. Il fournit aussi `HttpTestingController`, que vous utiliserez pour interagir avec le backend de test, définir des attentes sur les requêtes effectuées et renvoyer (flush) des réponses à ces requêtes. `HttpTestingController` peut être injecté depuis `TestBed` une fois configuré.

```ts
TestBed.configureTestingModule({
  providers: [
    // ... other test providers
    provideHttpClientTesting(),
  ],
});

const httpTesting = TestBed.inject(HttpTestingController);
```

Désormais, lorsque vos tests effectuent des requêtes, celles-ci atteignent le backend de test au lieu du backend habituel. Vous pouvez utiliser `httpTesting` pour faire des assertions sur ces requêtes.

### Configurer `HttpClient` dans les tests {#configuring-httpclient-in-tests}

Si un test doit configurer des fonctionnalités de `HttpClient`, comme des intercepteurs, incluez `provideHttpClient(...)` avant `provideHttpClientTesting()`.
IMPORTANT: Pensez à fournir `provideHttpClient()` **avant** `provideHttpClientTesting()`, car `provideHttpClientTesting()` écrase certaines parties de `provideHttpClient()`. Procéder dans l'ordre inverse risque de casser vos tests.

```ts
TestBed.configureTestingModule({
  providers: [provideHttpClient(withInterceptors([authInterceptor])), provideHttpClientTesting()],
});
```

## Attendre des requêtes et y répondre {#expecting-and-answering-requests}

Par exemple, vous pouvez écrire un test qui s'attend à ce qu'une requête GET soit effectuée et qui fournit un mock de réponse :

```ts
TestBed.configureTestingModule({
  providers: [ConfigService, provideHttpClientTesting()],
});

const httpTesting = TestBed.inject(HttpTestingController);

// Load `ConfigService` and request the current configuration.
const service = TestBed.inject(ConfigService);
const config$ = service.getConfig<Config>();

// `firstValueFrom` subscribes to the `Observable`, which makes the HTTP request,
// and creates a `Promise` of the response.
const configPromise = firstValueFrom(config$);

// At this point, the request is pending, and we can assert it was made
// via the `HttpTestingController`:
const req = httpTesting.expectOne('/api/config', 'Request to load the configuration');

// We can assert various properties of the request if desired.
expect(req.request.method).toBe('GET');

// Flushing the request causes it to complete, delivering the result.
req.flush(DEFAULT_CONFIG);

// We can then assert that the response was successfully delivered by the `ConfigService`:
expect(await configPromise).toEqual(DEFAULT_CONFIG);

// Finally, we can assert that no other requests were made.
httpTesting.verify();
```

NOTE: `expectOne` échoue si le test a effectué plus d'une requête correspondant aux critères donnés.

Au lieu de faire une assertion sur `req.method`, vous pouvez utiliser une forme étendue de `expectOne` qui vérifie aussi la méthode de la requête :

```ts
const req = httpTesting.expectOne(
  {
    method: 'GET',
    url: '/api/config',
  },
  'Request to load the configuration',
);
```

HELPFUL: Les API d'attente comparent l'URL complète des requêtes, query parameters compris.

La dernière étape, qui vérifie qu'aucune requête ne reste en attente, est suffisamment courante pour que vous la déplaciez dans une étape `afterEach()` :

```ts
afterEach(() => {
  // Verify that none of the tests make any extra HTTP requests.
  TestBed.inject(HttpTestingController).verify();
});
```

## Gérer plusieurs requêtes à la fois {#handling-more-than-one-request-at-once}

Si vous devez répondre à des requêtes en double dans votre test, utilisez l'API `match()` au lieu de `expectOne()`. Elle prend les mêmes arguments mais renvoie un tableau des requêtes correspondantes. Une fois renvoyées, ces requêtes sont exclues des correspondances futures, et c'est à vous de les vider (flush) et de les vérifier.

```ts
const allGetRequests = httpTesting.match({method: 'GET'});
for (const req of allGetRequests) {
  // Handle responding to each request.
}
```

## Correspondance avancée {#advanced-matching}

Toutes les fonctions de correspondance acceptent une fonction prédicat pour une logique de correspondance personnalisée :

```ts
// Look for one request that has a request body.
const requestsWithBody = httpTesting.expectOne((req) => req.body !== null);
```

La fonction `expectNone` vérifie qu'aucune requête ne correspond aux critères donnés.

```ts
// Assert that no mutation requests have been issued.
httpTesting.expectNone((req) => req.method !== 'GET');
```

## Tester la gestion des erreurs {#testing-error-handling}

Vous devriez tester les réactions de votre application quand des requêtes HTTP échouent.

### Erreurs du backend {#backend-errors}

Pour tester la gestion des erreurs du backend (quand le serveur renvoie un code de statut d'échec), videz (flush) les requêtes avec une réponse d'erreur qui émule ce que votre backend renverrait en cas d'échec d'une requête.

```ts
const req = httpTesting.expectOne('/api/config');
req.flush('Failed!', {status: 500, statusText: 'Internal Server Error'});

// Assert that the application successfully handled the backend error.
```

### Erreurs réseau {#network-errors}

Les requêtes peuvent aussi échouer à cause d'erreurs réseau, qui se manifestent sous forme d'erreurs `ProgressEvent`. Celles-ci peuvent être transmises avec la méthode `error()` :

```ts
const req = httpTesting.expectOne('/api/config');
req.error(new ProgressEvent('network error!'));

// Assert that the application successfully handled the network error.
```

## Tester un intercepteur {#testing-an-interceptor}

Vous devriez tester que vos intercepteurs fonctionnent dans les conditions souhaitées.

Par exemple, une application peut devoir ajouter à chaque requête sortante un jeton d'authentification généré par un service.
Ce comportement peut être assuré à l'aide d'un intercepteur :

```ts
export function authInterceptor(
  request: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  const authService = inject(AuthService);

  const clonedRequest = request.clone({
    headers: request.headers.append('X-Authentication-Token', authService.getAuthToken()),
  });
  return next(clonedRequest);
}
```

La configuration de `TestBed` pour cet intercepteur doit s'appuyer sur la fonctionnalité `withInterceptors`.

```ts
TestBed.configureTestingModule({
  providers: [
    AuthService,
    // Testing one interceptor at a time is recommended.
    provideHttpClient(withInterceptors([authInterceptor])),
    provideHttpClientTesting(),
  ],
});
```

Le `HttpTestingController` peut récupérer l'instance de la requête, que vous pouvez ensuite inspecter pour vous assurer que la requête a bien été modifiée.

```ts
const service = TestBed.inject(AuthService);
const http = TestBed.inject(HttpClient);

// Make an HTTP request.
const response = firstValueFrom(http.get('/api/config'));

const req = httpTesting.expectOne('/api/config');

expect(req.request.headers.get('X-Authentication-Token')).toEqual(service.getAuthToken());
```

Un intercepteur similaire pourrait être implémenté avec des intercepteurs basés sur des classes :

```ts
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  private authService = inject(AuthService);

  intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    const clonedRequest = request.clone({
      headers: request.headers.append('X-Authentication-Token', this.authService.getAuthToken()),
    });
    return next.handle(clonedRequest);
  }
}
```

Pour le tester, la configuration de `TestBed` doit plutôt être la suivante :

```ts
TestBed.configureTestingModule({
  providers: [
    AuthService,
    provideHttpClient(withInterceptorsFromDi()),
    provideHttpClientTesting(),
    // We rely on the HTTP_INTERCEPTORS token to register the AuthInterceptor as an HttpInterceptor
    {provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true},
  ],
});
```
