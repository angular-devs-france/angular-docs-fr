<!-- source: ffe1949cca5319a017ae9d8b2d3c2d74fc3f58f6 -->
# Récupération de données réactive avec `httpResource`

`httpResource` est un wrapper réactif autour de `HttpClient` qui vous fournit le statut de la requête et la réponse sous forme de signals. Vous pouvez donc utiliser ces signals avec `computed`, `effect`, `linkedSignal` ou toute autre API réactive. Comme il est construit sur `HttpClient`, `httpResource` prend en charge les mêmes fonctionnalités, comme les intercepteurs.

Pour en savoir plus sur le pattern `resource` d'Angular, consultez [Réactivité asynchrone avec `resource`](/guide/signals/resource).

## `Using httpResource` {#using-httpresource}

TIP: `httpResource` utilise le `HttpClient` disponible globalement. N'utilisez `provideHttpClient(...)` que si vous devez configurer des fonctionnalités HTTP, comme des intercepteurs ou des options XSRF. Consultez [Configurer HttpClient](/guide/http/setup) pour plus de détails.

Vous pouvez définir une resource HTTP en renvoyant une URL :

```ts
userId = input.required<string>();

user = httpResource(() => `/api/user/${userId()}`); // A reactive function as argument
```

`httpResource` est réactif : chaque fois qu'un des signals dont il dépend change (comme `userId`), la resource émet une nouvelle requête HTTP.
Si une requête est déjà en attente, la resource l'annule avant d'en émettre une nouvelle.

HELPFUL: `httpResource` diffère de `HttpClient` car il lance la requête _immédiatement_. À l'inverse, `HttpClient` ne lance les requêtes qu'au moment de l'abonnement à l'`Observable` renvoyé.

Pour des requêtes plus avancées, vous pouvez définir un objet de requête semblable à celui accepté par `HttpClient`.
Chaque propriété de l'objet de requête qui doit être réactive doit être composée à partir d'un signal.

```ts
user = httpResource(() => ({
  url: `/api/user/${userId()}`,
  method: 'GET',
  headers: {
    'X-Special': 'true',
  },
  params: {
    'fast': 'yes',
  },
  reportProgress: true,
  transferCache: true,
  keepalive: true,
  mode: 'cors',
  redirect: 'error',
  priority: 'high',
  cache: 'force-cache',
  credentials: 'include',
  referrer: 'no-referrer',
  integrity: 'sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GhEXAMPLEKEY=',
  referrerPolicy: 'no-referrer',
}));
```

TIP: Évitez d'utiliser `httpResource` pour des _mutations_ comme `POST` ou `PUT`. Préférez plutôt utiliser directement les API sous-jacentes de `HttpClient`.

Les signals de la `httpResource` peuvent être utilisés dans le template pour contrôler quels éléments afficher.

```angular-html
@if (user.hasValue()) {
  <user-details [user]="user.value()" />
} @else if (user.error()) {
  <div>Could not load user information</div>
} @else if (user.isLoading()) {
  <div>Loading user info...</div>
}
```

HELPFUL: Lire le signal `value` d'une `resource` en état d'erreur lève une exception à l'exécution. Il est recommandé de protéger les lectures de `value` avec `hasValue()`.

### Types de réponse {#response-types}

Par défaut, `httpResource` renvoie et analyse la réponse en JSON. Vous pouvez toutefois spécifier un autre type de retour grâce à des fonctions supplémentaires de `httpResource` :

```ts
httpResource.text(() => ({ … })); // returns a string in value()

httpResource.blob(() => ({ … })); // returns a Blob object in value()

httpResource.arrayBuffer(() => ({ … })); // returns an ArrayBuffer in value()
```

### Suivre la progression du téléchargement {#tracking-download-progress}

Définissez `reportProgress: true` dans l'objet de requête pour suivre la progression du téléchargement de la réponse. Le dernier `HttpProgressEvent` est disponible via le signal `progress` de la resource :

```ts
file = httpResource.blob(() => ({
  url: `/api/files/${fileId()}`,
  reportProgress: true,
}));

downloaded = computed(() => this.file.progress()?.loaded ?? 0);
```

NOTE: Contrairement à `HttpClient`, où `reportProgress` est déprécié au profit de `reportUploadProgress` et `reportDownloadProgress`, `httpResource` conserve l'option unique `reportProgress`. Elle n'active que les événements de progression du téléchargement.

## Analyse et validation des réponses {#response-parsing-and-validation}

Lorsque vous récupérez des données, vous pouvez vouloir valider les réponses par rapport à un schéma prédéfini, souvent à l'aide de libraries open source populaires comme [Zod](https://zod.dev) ou [Valibot](https://valibot.dev). Vous pouvez intégrer ce type de library de validation à `httpResource` en spécifiant une option `parse`. Le type de retour de la fonction `parse` détermine le type de la `value` de la resource.

L'exemple suivant utilise Zod pour analyser et valider la réponse de l'[API StarWars](https://swapi.info/). La resource est alors typée comme le type de sortie de l'analyse de Zod.

```ts
const starWarsPersonSchema = z.object({
  name: z.string(),
  height: z.number({coerce: true}),
  edited: z.string().datetime(),
  films: z.array(z.string()),
});

export class CharacterViewer {
  id = signal(1);

  swPersonResource = httpResource(() => `https://swapi.info/api/people/${this.id()}`, {
    parse: starWarsPersonSchema.parse,
  });
}
```

## Tester une httpResource {#testing-an-httpresource}

Comme `httpResource` est un wrapper autour de `HttpClient`, vous pouvez tester `httpResource` avec exactement les mêmes API que `HttpClient`. Consultez [Tester HttpClient](/guide/http/testing) pour plus de détails.

L'exemple suivant montre un test unitaire pour du code qui utilise `httpResource`.

```ts
TestBed.configureTestingModule({
  providers: [provideHttpClientTesting()],
});

const id = signal(0);
const mockBackend = TestBed.inject(HttpTestingController);
const response = httpResource(() => `/data/${id()}`, {injector: TestBed.inject(Injector)});
TestBed.tick(); // Triggers the effect
const firstRequest = mockBackend.expectOne('/data/0');
firstRequest.flush(0);

// Ensures the values are propagated to the httpResource
await TestBed.inject(ApplicationRef).whenStable();

expect(response.value()).toEqual(0);
```
