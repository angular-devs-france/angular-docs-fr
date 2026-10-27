<!-- source: 457babf1c7bf200ecbdfa6c2ea28fd1eb1ebfa57 -->
# Effectuer des requêtes HTTP

`HttpClient` dispose de méthodes correspondant aux différents verbes HTTP utilisés pour effectuer des requêtes, aussi bien pour charger des données que pour appliquer des mutations sur le serveur. Chaque méthode renvoie un [`Observable` RxJS](https://rxjs.dev/guide/observable) qui, lorsqu'on s'y abonne, envoie la requête puis émet les résultats quand le serveur répond.

NOTE: Les `Observable`s créés par `HttpClient` peuvent faire l'objet d'un nombre quelconque d'abonnements et effectuent une nouvelle requête au backend pour chaque abonnement.

Un objet d'options passé à la méthode de requête permet d'ajuster diverses propriétés de la requête ainsi que le type de réponse renvoyé.

## Récupérer des données JSON {#fetching-json-data}

Récupérer des données depuis un backend nécessite souvent d'effectuer une requête GET avec la méthode [`HttpClient.get()`](api/common/http/HttpClient#get). Cette méthode prend deux arguments : l'URL de l'endpoint (une chaîne) depuis lequel récupérer les données, et un objet d'_options facultatif_ pour configurer la requête.

Par exemple, pour récupérer des données de configuration depuis une API hypothétique avec la méthode `HttpClient.get()` :

```ts
http.get<Config>('/api/config').subscribe((config) => {
  // process the configuration.
});
```

Remarquez l'argument de type générique, qui indique que les données renvoyées par le serveur seront de type `Config`. Cet argument est facultatif ; si vous l'omettez, les données renvoyées seront de type `Object`.

TIP: Lorsque vous manipulez des données de structure incertaine, susceptibles de contenir des valeurs `undefined` ou `null`, envisagez d'utiliser le type `unknown` plutôt que `Object` comme type de réponse.

CRITICAL: Le type générique des méthodes de requête est une **assertion** de type sur les données renvoyées par le serveur. `HttpClient` ne vérifie pas que les données réellement renvoyées correspondent à ce type.

## Récupérer d'autres types de données {#fetching-other-types-of-data}

Par défaut, `HttpClient` suppose que les serveurs renvoient des données JSON. Lorsque vous interagissez avec une API non JSON, vous pouvez indiquer à `HttpClient` le type de réponse attendu au moment d'effectuer la requête. Cela se fait avec l'option `responseType`.

| **Valeur de `responseType`** | **Type de réponse renvoyé**                                                                                                                |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `'json'` (par défaut)       | Données JSON du type générique donné                                                                                                       |
| `'text'`                 | Données de type chaîne                                                                                                                               |
| `'arraybuffer'`          | [`ArrayBuffer`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) contenant les octets bruts de la réponse |
| `'blob'`                 | Instance de [`Blob`](https://developer.mozilla.org/docs/Web/API/Blob)                                                                        |

Par exemple, vous pouvez demander à `HttpClient` de télécharger les octets bruts d'une image `.jpeg` dans un `ArrayBuffer` :

```ts
http.get('/images/dog.jpg', {responseType: 'arraybuffer'}).subscribe((buffer) => {
  console.log('The image is ' + buffer.byteLength + ' bytes large');
});
```

<docs-callout important title="Valeur littérale pour `responseType`">
Comme la valeur de `responseType` influe sur le type renvoyé par `HttpClient`, elle doit avoir un type littéral et non un type `string`.

C'est automatiquement le cas si l'objet d'options passé à la méthode de requête est un objet littéral, mais si vous extrayez les options de la requête dans une variable ou une méthode utilitaire, vous devrez peut-être le spécifier explicitement comme littéral, par exemple `responseType: 'text' as const`.
</docs-callout>

## Modifier l'état du serveur {#mutating-server-state}

Les API serveur qui effectuent des mutations nécessitent souvent des requêtes POST avec un corps de requête qui spécifie le nouvel état ou la modification à apporter.

La méthode [`HttpClient.post()`](api/common/http/HttpClient#post) se comporte de manière similaire à `get()` et accepte un argument `body` supplémentaire avant ses options :

```ts
http.post<Config>('/api/config', newConfig).subscribe((config) => {
  console.log('Updated config:', config);
});
```

De nombreux types de valeurs différents peuvent être fournis comme `body` de la requête, et `HttpClient` les sérialise en conséquence :

| **Type de `body`**                                                                                                               | **Sérialisé en**                                    |
| ----------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| chaîne                                                                                                                        | Texte brut                                           |
| nombre, booléen, tableau ou objet simple                                                                                       | JSON                                                 |
| [`ArrayBuffer`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)                       | données brutes du buffer                             |
| [`Blob`](https://developer.mozilla.org/docs/Web/API/Blob)                                                                     | données brutes avec le type de contenu du `Blob`              |
| [`FormData`](https://developer.mozilla.org/docs/Web/API/FormData)                                                             | données encodées en `multipart/form-data`                   |
| [`HttpParams`](api/common/http/HttpParams) ou [`URLSearchParams`](https://developer.mozilla.org/docs/Web/API/URLSearchParams) | chaîne au format `application/x-www-form-urlencoded` |

IMPORTANT: N'oubliez pas d'appeler `.subscribe()` sur les `Observable`s des requêtes de mutation pour que la requête soit réellement envoyée.

## Définir les paramètres d'URL {#setting-url-parameters}

Spécifiez les paramètres de requête à inclure dans l'URL de la requête avec l'option `params`.

Passer un objet littéral est la façon la plus simple de configurer les paramètres d'URL :

```ts
http
  .get('/api/config', {
    params: {filter: 'all'},
  })
  .subscribe((config) => {
    // ...
  });
```

Vous pouvez aussi passer une instance de `HttpParams` si vous avez besoin de plus de contrôle sur la construction ou la sérialisation des paramètres.

IMPORTANT: Les instances de `HttpParams` sont _immuables_ et ne peuvent pas être modifiées directement. Les méthodes de mutation comme `append()` renvoient plutôt une nouvelle instance de `HttpParams` à laquelle la mutation est appliquée.

```ts
const baseParams = new HttpParams().set('filter', 'all');

http
  .get('/api/config', {
    params: baseParams.set('details', 'enabled'),
  })
  .subscribe((config) => {
    // ...
  });
```

Vous pouvez instancier `HttpParams` avec un `HttpParameterCodec` personnalisé qui détermine comment `HttpClient` encode les paramètres dans l'URL.

### Encodage personnalisé des paramètres {#custom-parameter-encoding}

Par défaut, `HttpParams` utilise le [`HttpUrlEncodingCodec`](api/common/http/HttpUrlEncodingCodec) intégré pour encoder et décoder les clés et les valeurs des paramètres.

Vous pouvez fournir votre propre implémentation de [`HttpParameterCodec`](api/common/http/HttpParameterCodec) pour personnaliser la façon dont l'encodage et le décodage sont appliqués.

```ts
import {HttpClient, HttpParams, HttpParameterCodec} from '@angular/common/http';
import {inject} from '@angular/core';

export class CustomHttpParamEncoder implements HttpParameterCodec {
  encodeKey(key: string): string {
    return encodeURIComponent(key);
  }

  encodeValue(value: string): string {
    return encodeURIComponent(value);
  }

  decodeKey(key: string): string {
    return decodeURIComponent(key);
  }

  decodeValue(value: string): string {
    return decodeURIComponent(value);
  }
}

export class ApiService {
  private http = inject(HttpClient);

  search() {
    const params = new HttpParams({
      encoder: new CustomHttpParamEncoder(),
    })
      .set('email', 'dev+alerts@example.com')
      .set('q', 'a & b? c/d = e');

    return this.http.get('/api/items', {params});
  }
}
```

## Définir les en-têtes de requête {#setting-request-headers}

Spécifiez les en-têtes de requête à inclure dans la requête avec l'option `headers`.

Passer un objet littéral est la façon la plus simple de configurer les en-têtes de requête :

```ts
http
  .get('/api/config', {
    headers: {
      'X-Debug-Level': 'verbose',
    },
  })
  .subscribe((config) => {
    // ...
  });
```

Vous pouvez aussi passer une instance de `HttpHeaders` si vous avez besoin de plus de contrôle sur la construction des en-têtes.

IMPORTANT: Les instances de `HttpHeaders` sont _immuables_ et ne peuvent pas être modifiées directement. Les méthodes de mutation comme `append()` renvoient plutôt une nouvelle instance de `HttpHeaders` à laquelle la mutation est appliquée.

```ts
const baseHeaders = new HttpHeaders().set('X-Debug-Level', 'minimal');

http
  .get<Config>('/api/config', {
    headers: baseHeaders.set('X-Debug-Level', 'verbose'),
  })
  .subscribe((config) => {
    // ...
  });
```

## Interagir avec les événements de réponse du serveur {#interacting-with-the-server-response-events}

Par commodité, `HttpClient` renvoie par défaut un `Observable` des données renvoyées par le serveur (le corps de la réponse). Il est parfois souhaitable d'examiner la réponse elle-même, par exemple pour récupérer certains en-têtes de réponse.

Pour accéder à la réponse complète, définissez l'option `observe` à `'response'` :

```ts
http.get<Config>('/api/config', {observe: 'response'}).subscribe((res) => {
  console.log('Response status:', res.status);
  console.log('Body:', res.body);
});
```

<docs-callout important title="Valeur littérale pour `observe`">
Comme la valeur de `observe` influe sur le type renvoyé par `HttpClient`, elle doit avoir un type littéral et non un type `string`.

C'est automatiquement le cas si l'objet d'options passé à la méthode de requête est un objet littéral, mais si vous extrayez les options de la requête dans une variable ou une méthode utilitaire, vous devrez peut-être le spécifier explicitement comme littéral, par exemple `observe: 'response' as const`.
</docs-callout>

## Recevoir les événements de progression bruts {#receiving-raw-progress-events}

En plus du corps ou de l'objet de la réponse, `HttpClient` peut aussi renvoyer un flux d'_événements_ bruts correspondant à des moments précis du cycle de vie de la requête. Ces événements indiquent notamment quand la requête est envoyée, quand l'en-tête de la réponse est renvoyé et quand le corps est complet. Ils peuvent aussi inclure des _événements de progression_ qui rendent compte de l'état de l'envoi et du téléchargement pour les corps de requête ou de réponse volumineux.

Les événements de progression sont désactivés par défaut (car ils ont un coût en performances), mais peuvent être activés avec les options `reportUploadProgress` et `reportDownloadProgress`.

NOTE: Le backend fetch par défaut de `HttpClient` ne prend pas en charge les événements de progression d'_envoi_ (upload) et lève une erreur si vous définissez `reportUploadProgress`. Si votre application a besoin des événements de progression d'envoi, configurez `HttpClient` avec `withXhr()` dans `provideHttpClient(...)`.

Pour observer le flux d'événements, définissez l'option `observe` à `'events'` :

```ts
http
  .get('/api/download', {
    reportDownloadProgress: true,
    observe: 'events',
  })
  .subscribe((event) => {
    switch (event.type) {
      case HttpEventType.DownloadProgress:
        console.log('Downloaded ' + event.loaded + ' out of ' + event.total + ' bytes');
        break;
      case HttpEventType.Response:
        console.log('Finished downloading!');
        break;
    }
  });
```

<docs-callout important title="Valeur littérale pour `observe`">
Comme la valeur de `observe` influe sur le type renvoyé par `HttpClient`, elle doit avoir un type littéral et non un type `string`.

C'est automatiquement le cas si l'objet d'options passé à la méthode de requête est un objet littéral, mais si vous extrayez les options de la requête dans une variable ou une méthode utilitaire, vous devrez peut-être le spécifier explicitement comme littéral, par exemple `observe: 'events' as const`.
</docs-callout>

Chaque `HttpEvent` émis dans le flux d'événements possède un `type` qui indique ce que l'événement représente :

| **Valeur de `type`**                 | **Signification de l'événement**                                                                  |
| -------------------------------- | ---------------------------------------------------------------------------------- |
| `HttpEventType.Sent`             | La requête a été envoyée au serveur                                      |
| `HttpEventType.UploadProgress`   | Un `HttpUploadProgressEvent` qui rend compte de la progression de l'envoi du corps de la requête      |
| `HttpEventType.ResponseHeader`   | L'en-tête de la réponse a été reçu, y compris le statut et les en-têtes           |
| `HttpEventType.DownloadProgress` | Un `HttpDownloadProgressEvent` qui rend compte de la progression du téléchargement du corps de la réponse |
| `HttpEventType.Response`         | La réponse complète a été reçue, y compris son corps                 |
| `HttpEventType.User`             | Un événement personnalisé provenant d'un intercepteur HTTP.                                           |

## Gérer l'échec d'une requête {#handling-request-failure}

Une requête HTTP peut échouer de trois façons :

- Une erreur réseau ou de connexion peut empêcher la requête d'atteindre le serveur backend.
- Une requête n'a pas répondu à temps alors que l'option de timeout était définie.
- Le backend peut recevoir la requête mais échouer à la traiter, et renvoyer une réponse d'erreur.

`HttpClient` capture tous ces types d'erreurs dans une `HttpErrorResponse` qu'il renvoie via le canal d'erreur de l'`Observable`. Les erreurs réseau et de timeout ont un code de `status` égal à `0`. Pour les timeouts, `error` est une `DOMException` nommée `TimeoutError` ; pour les erreurs réseau, c'est l'erreur levée par `fetch` (ou un [`ProgressEvent`](https://developer.mozilla.org/docs/Web/API/ProgressEvent) avec `withXhr()`). Les erreurs du backend ont le code de `status` d'échec renvoyé par le backend, et la réponse d'erreur comme `error`. Inspectez la réponse pour identifier la cause de l'erreur et l'action appropriée pour la gérer.

La [library RxJS](https://rxjs.dev/) propose plusieurs opérateurs qui peuvent être utiles pour la gestion des erreurs.

Vous pouvez utiliser l'opérateur `catchError` pour transformer une réponse d'erreur en une valeur pour l'interface. Cette valeur peut indiquer à l'interface d'afficher une page ou une valeur d'erreur, et capturer la cause de l'erreur si nécessaire.

Des erreurs passagères, comme des interruptions réseau, peuvent parfois faire échouer une requête de manière inattendue ; il suffit alors de relancer la requête pour qu'elle réussisse. RxJS fournit plusieurs opérateurs de _retry_ qui se réabonnent automatiquement à un `Observable` en échec sous certaines conditions. Par exemple, l'opérateur `retry()` tente automatiquement de se réabonner un nombre de fois donné.

### Timeouts {#timeouts}

Pour définir un timeout sur une requête, vous pouvez définir l'option `timeout` à un nombre de millisecondes, avec les autres options de la requête. Si la requête au backend ne se termine pas dans le délai spécifié, elle est annulée et une erreur est émise.

NOTE: Le timeout ne s'applique qu'à la requête HTTP au backend elle-même. Ce n'est pas un timeout pour toute la chaîne de traitement de la requête. Cette option n'est donc pas affectée par les délais introduits par les intercepteurs.

```ts
http
  .get('/api/config', {
    timeout: 3000,
  })
  .subscribe({
    next: (config) => {
      console.log('Config fetched successfully:', config);
    },
    error: (err) => {
      // If the request times out, an error will have been emitted.
    },
  });
```

## Options fetch avancées {#advanced-fetch-options}

Le `HttpClient` d'Angular prend en charge des options avancées de l'API fetch qui peuvent améliorer les performances et l'expérience utilisateur. Ces options sont disponibles avec le backend fetch, qui est celui par défaut.

### Options fetch {#fetch-options}

Les options suivantes offrent un contrôle fin du comportement des requêtes avec le backend fetch.

#### Connexions keep-alive {#keep-alive-connections}

L'option `keepalive` permet à une requête de survivre à la page qui l'a initiée. C'est particulièrement utile pour les requêtes d'analytics ou de journalisation qui doivent aboutir même si l'utilisateur quitte la page.

```ts
http
  .post('/api/analytics', analyticsData, {
    keepalive: true,
  })
  .subscribe();
```

#### Contrôle de la mise en cache HTTP {#http-caching-control}

L'option `cache` contrôle la façon dont la requête interagit avec le cache HTTP du navigateur, ce qui peut améliorer sensiblement les performances des requêtes répétées.

```ts
//  Use cached response regardless of freshness
http
  .get('/api/config', {
    cache: 'force-cache',
  })
  .subscribe((config) => {
    // ...
  });

// Always fetch from network, bypass cache
http
  .get('/api/live-data', {
    cache: 'no-cache',
  })
  .subscribe((data) => {
    // ...
  });

// Use cached response only, fail if not in cache
http
  .get('/api/static-data', {
    cache: 'only-if-cached',
  })
  .subscribe((data) => {
    // ...
  });
```

#### Priorité des requêtes pour les Core Web Vitals {#request-priority-for-core-web-vitals}

L'option `priority` vous permet d'indiquer l'importance relative d'une requête, ce qui aide les navigateurs à optimiser le chargement des ressources pour de meilleurs scores Core Web Vitals.

```ts
// High priority for critical resources
http
  .get('/api/user-profile', {
    priority: 'high',
  })
  .subscribe((profile) => {
    // ...
  });

// Low priority for non-critical resources
http
  .get('/api/recommendations', {
    priority: 'low',
  })
  .subscribe((recommendations) => {
    // ...
  });

// Auto priority (default) lets the browser decide
http
  .get('/api/settings', {
    priority: 'auto',
  })
  .subscribe((settings) => {
    // ...
  });
```

Valeurs de `priority` disponibles :

- `'high'` : priorité haute, chargée tôt (par exemple, données utilisateur critiques, contenu au-dessus de la ligne de flottaison)
- `'low'` : priorité basse, chargée quand les ressources sont disponibles (par exemple, analytics, données préchargées)
- `'auto'` : le navigateur détermine la priorité selon le contexte de la requête (par défaut)

TIP: Utilisez `priority: 'high'` pour les requêtes qui affectent le Largest Contentful Paint (LCP) et `priority: 'low'` pour celles qui n'ont pas d'impact sur l'expérience utilisateur initiale.

#### Mode de requête {#request-mode}

L'option `mode` contrôle la façon dont la requête gère les requêtes cross-origin et détermine le type de réponse.

```ts
// Same-origin requests only
http
  .get('/api/local-data', {
    mode: 'same-origin',
  })
  .subscribe((data) => {
    // ...
  });

// CORS-enabled cross-origin requests
http
  .get('https://api.external.com/data', {
    mode: 'cors',
  })
  .subscribe((data) => {
    // ...
  });

// No-CORS mode for simple cross-origin requests
http
  .get('https://external-api.com/public-data', {
    mode: 'no-cors',
  })
  .subscribe((data) => {
    // ...
  });
```

Valeurs de `mode` disponibles :

- `'same-origin'` : n'autorise que les requêtes same-origin, échoue pour les requêtes cross-origin
- `'cors'` : autorise les requêtes cross-origin avec CORS (par défaut)
- `'no-cors'` : autorise les requêtes cross-origin simples sans CORS ; la réponse est opaque

TIP: Dans le navigateur, utilisez `mode: 'same-origin'` pour les requêtes sensibles qui ne doivent jamais partir en cross-origin.

IMPORTANT: Pendant le SSR sur Node.js, `HttpClient` utilise l'[implémentation de Fetch de Node.js basée sur Undici](https://nodejs.org/api/globals.html#fetch). [Undici n'applique pas les vérifications CORS des navigateurs](https://undici.nodejs.org/#cors) : `mode: 'same-origin'` ne restreint donc pas les requêtes côté serveur. Validez les URL influencées par l'utilisateur à l'aide d'une liste d'autorisation.

#### Gestion des redirections {#redirect-handling}

L'option `redirect` indique comment gérer les réponses de redirection du serveur.

```ts
// Follow redirects automatically (default behavior)
http
  .get('/api/resource', {
    redirect: 'follow',
  })
  .subscribe((data) => {
    // ...
  });

// Prevent automatic redirects
http
  .get('/api/resource', {
    redirect: 'manual',
  })
  .subscribe((response) => {
    // Handle redirect manually
  });

// Treat redirects as errors
http
  .get('/api/resource', {
    redirect: 'error',
  })
  .subscribe({
    next: (data) => {
      // Success response
    },
    error: (err) => {
      // Redirect responses will trigger this error handler
    },
  });
```

Valeurs de `redirect` disponibles :

- `'follow'` : suit automatiquement les redirections (par défaut)
- `'error'` : traite les redirections comme des erreurs
- `'manual'` : ne suit pas automatiquement les redirections et renvoie la réponse de redirection

TIP: Utilisez `redirect: 'manual'` lorsque vous devez gérer les redirections avec une logique personnalisée.

#### Gestion des identifiants {#credentials-handling}

L'option `credentials` contrôle si les cookies, les en-têtes d'autorisation et les autres identifiants sont envoyés avec les requêtes cross-origin. C'est particulièrement important dans les scénarios d'authentification.

```ts
// Include credentials for cross-origin requests
http
  .get('https://api.example.com/protected-data', {
    credentials: 'include',
  })
  .subscribe((data) => {
    // ...
  });

// Never send credentials (default for cross-origin)
http
  .get('https://api.example.com/public-data', {
    credentials: 'omit',
  })
  .subscribe((data) => {
    // ...
  });

// Send credentials only for same-origin requests
http
  .get('/api/user-data', {
    credentials: 'same-origin',
  })
  .subscribe((data) => {
    // ...
  });

// withCredentials overrides credentials setting
http
  .get('https://api.example.com/data', {
    credentials: 'omit', // This will be ignored
    withCredentials: true, // This forces credentials: 'include'
  })
  .subscribe((data) => {
    // Request will include credentials despite credentials: 'omit'
  });

// Legacy approach (still supported)
http
  .get('https://api.example.com/data', {
    withCredentials: true,
  })
  .subscribe((data) => {
    // Equivalent to credentials: 'include'
  });
```

IMPORTANT: L'option `withCredentials` a la priorité sur l'option `credentials`. Si les deux sont spécifiées, `withCredentials: true` produit toujours `credentials: 'include'`, quelle que soit la valeur explicite de `credentials`.

Valeurs de `credentials` disponibles :

- `'omit'` : n'envoie jamais d'identifiants
- `'same-origin'` : n'envoie les identifiants que pour les requêtes same-origin (par défaut)
- `'include'` : envoie toujours les identifiants, même pour les requêtes cross-origin

TIP: Utilisez `credentials: 'include'` lorsque vous devez envoyer des cookies ou des en-têtes d'authentification à un autre domaine qui prend en charge CORS. Évitez de mélanger les options `credentials` et `withCredentials` pour ne pas créer de confusion.

IMPORTANT: Pendant le SSR sur Node.js, `credentials: 'include'` ne transmet pas automatiquement les cookies de la requête entrante du navigateur. L'option `credentials` ne supprime pas les en-têtes `Cookie` ou `Authorization` que vous ajoutez explicitement. [Undici autorise certains en-têtes que les navigateurs interdisent](https://undici.nodejs.org/#forbidden-and-safelisted-header-names) : ne transmettez donc les en-têtes d'identifiants qu'à des origines de confiance.

#### Referrer {#referrer}

L'option `referrer` vous permet de contrôler les informations de referrer envoyées avec la requête. C'est important pour des raisons de confidentialité et de sécurité.

```ts
// Send a specific referrer URL
http
  .get('/api/data', {
    referrer: 'https://example.com/page',
  })
  .subscribe((data) => {
    // ...
  });

// Use the current page as referrer (default behavior)
http
  .get('/api/analytics', {
    referrer: 'about:client',
  })
  .subscribe((data) => {
    // ...
  });
```

L'option `referrer` accepte :

- Une chaîne d'URL valide : définit l'URL de referrer précise à envoyer
- Une chaîne vide `''` : n'envoie aucune information de referrer
- `'about:client'` : utilise le referrer par défaut (l'URL de la page courante)

TIP: Utilisez `referrer: ''` pour les requêtes sensibles lorsque vous ne voulez pas divulguer l'URL de la page d'origine.

#### Referrer policy {#referrer-policy}

L'option `referrerPolicy` contrôle la quantité d'informations de referrer (l'URL de la page qui effectue la requête) envoyée avec une requête HTTP. Ce paramètre influe à la fois sur la confidentialité et sur les analytics, et vous permet de trouver un équilibre entre visibilité des données et sécurité.

```ts
// Send no referrer information regardless of the current page
http
  .get('/api/data', {
    referrerPolicy: 'no-referrer',
  })
  .subscribe();

// Send origin only (e.g. https://example.com)
http
  .get('/api/analytics', {
    referrerPolicy: 'origin',
  })
  .subscribe();
```

L'option `referrerPolicy` accepte :

- `'no-referrer'` N'envoie jamais l'en-tête `Referer`.
- `'no-referrer-when-downgrade'` Envoie le referrer pour les requêtes same-origin et sécurisées (HTTPS→HTTPS), mais l'omet lors d'une navigation d'une origine sécurisée vers une origine moins sécurisée (HTTPS→HTTP).
- `'origin'` N'envoie que l'origine (schéma, host, port) du referrer, sans le chemin ni la query string.
- `'origin-when-cross-origin'` Envoie l'URL complète pour les requêtes same-origin, mais uniquement l'origine pour les requêtes cross-origin.
- `'same-origin'` Envoie l'URL complète pour les requêtes same-origin et aucun referrer pour les requêtes cross-origin.
- `'strict-origin'` N'envoie que l'origine, et seulement si le niveau de sécurité du protocole n'est pas dégradé (par exemple, HTTPS→HTTPS). Omet le referrer en cas de dégradation.
- `'strict-origin-when-cross-origin'` Comportement par défaut des navigateurs. Envoie l'URL complète pour les requêtes same-origin, l'origine pour les requêtes cross-origin sans dégradation, et omet le referrer en cas de dégradation.
- `'unsafe-url'` Envoie toujours l'URL complète (y compris le chemin et la query string). Cela peut exposer des données sensibles et doit être utilisé avec prudence.

TIP: Préférez des valeurs prudentes comme `'no-referrer'`, `'origin'` ou `'strict-origin-when-cross-origin'` pour les requêtes sensibles en matière de confidentialité.

#### Intégrité {#integrity}

L'option `integrity` vous permet de vérifier que la réponse n'a pas été altérée en fournissant un hash cryptographique du contenu attendu. C'est particulièrement utile pour charger des scripts ou d'autres ressources depuis des CDN.

```ts
// Verify response integrity with SHA-256 hash
http
  .get('/api/script.js', {
    integrity: 'sha256-ABC123...',
    responseType: 'text',
  })
  .subscribe((script) => {
    // Script content is verified against the hash
  });
```

IMPORTANT: L'option `integrity` exige une correspondance exacte entre le contenu de la réponse et le hash fourni. Si le contenu ne correspond pas, la requête échoue avec une erreur réseau.

CRITICAL: Pendant le SSR, l'implémentation de Fetch lit l'intégralité du corps de la réponse pour vérifier `integrity` avant de renvoyer une réponse, comme l'exige le [Fetch Standard](https://fetch.spec.whatwg.org/#concept-main-fetch). Angular n'applique [`maxResponseBodySize`](/guide/ssr#configuring-the-response-body-size-limit) qu'après que Fetch a renvoyé une réponse : cette limite ne restreint donc pas les données mises en mémoire tampon pendant la vérification d'intégrité.

TIP: Utilisez l'intégrité des sous-ressources (subresource integrity) lorsque vous chargez des ressources critiques depuis des sources externes, pour garantir qu'elles n'ont pas été modifiées. Générez les hashs avec des outils comme `openssl`.

## Les `Observable`s HTTP {#http-observables}

Chaque méthode de requête de `HttpClient` construit et renvoie un `Observable` du type de réponse demandé. Il est important de comprendre le fonctionnement de ces `Observable`s quand on utilise `HttpClient`.

`HttpClient` produit ce que RxJS appelle des `Observable`s « froids » (cold) : aucune requête n'a réellement lieu tant qu'on ne s'est pas abonné à l'`Observable`. Ce n'est qu'à ce moment-là que la requête est réellement envoyée au serveur. S'abonner plusieurs fois au même `Observable` déclenche plusieurs requêtes au backend. Chaque abonnement est indépendant.

TIP: Vous pouvez voir les `Observable`s de `HttpClient` comme des _plans_ des véritables requêtes au serveur.

Une fois l'abonnement effectué, se désabonner annule la requête en cours. C'est très utile si l'abonnement à l'`Observable` passe par le pipe `async`, car la requête est automatiquement annulée si l'utilisateur quitte la page courante. De plus, si vous utilisez l'`Observable` avec un combinateur RxJS comme `switchMap`, cette annulation nettoie les requêtes obsolètes.

Une fois la réponse renvoyée, les `Observable`s de `HttpClient` se terminent généralement (même si des intercepteurs peuvent influer sur ce comportement).

Grâce à cette terminaison automatique, il n'y a généralement aucun risque de fuite mémoire si les abonnements à `HttpClient` ne sont pas nettoyés. Toutefois, comme pour toute opération asynchrone, nous vous recommandons fortement de nettoyer les abonnements lorsque le composant qui les utilise est détruit ; sinon, le callback d'abonnement risque de s'exécuter et de rencontrer des erreurs en tentant d'interagir avec le composant détruit.

TIP: Utiliser le pipe `async` ou l'opération `toSignal` pour s'abonner aux `Observable`s garantit que les abonnements sont correctement libérés.

## Bonnes pratiques {#best-practices}

Même si `HttpClient` peut être injecté et utilisé directement depuis les composants, nous vous recommandons généralement de créer des services réutilisables et injectables qui isolent et encapsulent la logique d'accès aux données. Par exemple, ce `UserService` encapsule la logique de requête des données d'un utilisateur à partir de son identifiant :

```ts
@Service()
export class UserService {
  private http = inject(HttpClient);

  getUser(id: string): Observable<User> {
    return this.http.get<User>(`/api/user/${id}`);
  }
}
```

Dans un composant, vous pouvez combiner `@if` avec le pipe `async` pour n'afficher l'interface des données qu'une fois leur chargement terminé :

```angular-ts
import {AsyncPipe} from '@angular/common';
import {Component, input, inject, effect} from '@angular/core';
import {Observable} from 'rxjs';

@Component({
  imports: [AsyncPipe],
  template: `
    @if (user$ | async; as user) {
      <p>Name: {{ user.name }}</p>
      <p>Biography: {{ user.biography }}</p>
    }
  `,
})
export class UserProfile {
  userId = input.required<string>();
  user$!: Observable<User>;

  private userService = inject(UserService);

  constructor() {
    effect(() => {
      this.user$ = this.userService.getUser(this.userId());
    });
  }
}
```
