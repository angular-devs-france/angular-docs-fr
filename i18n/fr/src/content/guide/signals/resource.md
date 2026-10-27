<!-- source: ae644cc0916da5775462786f0edeba8acdd7dc52 -->
# Réactivité asynchrone avec les ressources

Toutes les API de signals sont synchrones : `signal`, `computed`, `input`, etc. Pourtant, les applications doivent souvent gérer des données disponibles de manière asynchrone. Une `Resource` vous permet d'intégrer des données asynchrones dans le code de votre application basé sur les signals, tout en vous permettant d'accéder à ces données de manière synchrone.

Vous pouvez utiliser une `Resource` pour effectuer n'importe quel type d'opération asynchrone, mais le cas d'usage le plus courant d'une `Resource` est la récupération de données depuis un serveur. L'exemple suivant crée une ressource pour récupérer des données utilisateur.

Le moyen le plus simple de créer une `Resource` est la fonction `resource`.

```typescript
import {computed, resource, Signal} from '@angular/core';

const userId: Signal<string> = getUserId();

const userResource = resource({
  // Define a reactive computation.
  // The params value recomputes whenever any read signals change.
  params: () => ({id: userId()}),

  // Define an async loader that retrieves data.
  // The resource calls this function every time the `params` value changes.
  loader: ({params}) => fetchUser(params),
});

// Create a computed signal based on the result of the resource's loader function.
const firstName = computed(() => {
  if (userResource.hasValue()) {
    // `hasValue` serves 2 purposes:
    // - It acts as type guard to strip `undefined` from the type
    // - It protects against reading a throwing `value` when the resource is in error state
    return userResource.value().firstName;
  }

  // fallback in case the resource value is `undefined` or if the resource is in error state
  return undefined;
});
```

La fonction `resource` accepte un objet `ResourceOptions` avec deux propriétés principales : `params` et `loader`.

La propriété `params` définit un calcul réactif qui produit une valeur de paramètre. Chaque fois que les signals lus dans ce calcul changent, la ressource produit une nouvelle valeur de paramètre, comme le fait `computed`.

La propriété `loader` définit un `ResourceLoader` : une fonction asynchrone qui récupère un état. La ressource appelle le loader chaque fois que le calcul `params` produit une nouvelle valeur, en lui passant cette valeur. Consultez la section [Loaders de ressources](#resource-loaders) ci-dessous pour plus de détails.

`Resource` possède un signal `value` qui contient les résultats du loader.

## Loaders de ressources {#resource-loaders}

Quand vous créez une ressource, vous spécifiez un `ResourceLoader`. Ce loader est une fonction asynchrone qui accepte un unique paramètre (un objet `ResourceLoaderParams`) et renvoie une valeur.

L'objet `ResourceLoaderParams` contient trois propriétés : `params`, `previous` et `abortSignal`.

| Propriété     | Description                                                                                                                                      |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `params`      | La valeur du calcul `params` de la ressource.                                                                                                    |
| `previous`    | Un objet avec une propriété `status`, contenant le `ResourceStatus` précédent.                                                                   |
| `abortSignal` | Un [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal). Consultez [Annuler des requêtes](#aborting-requests) ci-dessous pour plus de détails. |

Si le calcul `params` renvoie `undefined`, la fonction loader ne s'exécute pas et le statut de la ressource devient `'idle'`.

### Ressources en streaming {#streaming-resources}

Certaines sources de données asynchrones produisent plusieurs valeurs au fil du temps au lieu de renvoyer un résultat unique. C'est le cas par exemple des WebSockets, des Server-Sent Events (SSE) et des écouteurs `onSnapshot` de Firestore.

Utilisez `stream` pour ces sources de données mises à jour en continu. Contrairement à `loader`, qui se résout une fois pour chaque requête, `stream` renvoie un signal dont la valeur peut continuer à se mettre à jour à mesure que de nouvelles données deviennent disponibles.

Utilisez `loader` pour les opérations asynchrones ponctuelles, comme la récupération de données depuis un endpoint HTTP.

Dans cet exemple, `userUpdates` simule une source externe comme une connexion WebSocket ou SSE.

```typescript
const userUpdates = signal({value: 'Alice'});

const userResource = resource({
  stream: () => userUpdates,
});

// Simulate a new value arriving from the external data source:
userUpdates.set({value: 'Bob'});
```

### Annuler des requêtes {#aborting-requests}

Une ressource annule une opération de chargement en cours si le calcul `params` change pendant que la ressource charge.

Vous pouvez utiliser l'`abortSignal` de `ResourceLoaderParams` pour réagir aux requêtes annulées. Par exemple, la fonction native `fetch` accepte un `AbortSignal` :

```typescript
const userId: Signal<string> = getUserId();

const userResource = resource({
  params: () => ({id: userId()}),
  loader: ({params, abortSignal}): Promise<User> => {
    // fetch cancels any outstanding HTTP requests when the given `AbortSignal`
    // indicates that the request has been aborted.
    return fetch(`users/${params.id}`, {signal: abortSignal}).then((res) => res.json());
  },
});
```

Consultez [`AbortSignal` sur MDN](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) pour plus de détails sur l'annulation de requêtes avec `AbortSignal`.

### Rechargement {#reloading}

Vous pouvez déclencher programmatiquement le `loader` d'une ressource en appelant la méthode `reload`.

```typescript
const userId: Signal<string> = getUserId();

const userResource = resource({
  params: () => ({id: userId()}),
  loader: ({params}) => fetchUser(params),
});

// ...

userResource.reload();
```

## Statut d'une ressource {#resource-status}

L'objet ressource possède plusieurs propriétés signal permettant de lire le statut du loader asynchrone.

| Propriété   | Description                                                                                                                                                                                  |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `value`     | La valeur la plus récente de la ressource, ou la valeur par défaut (`undefined` sauf si `defaultValue` est défini) si aucune valeur n'a été reçue. Lire `value()` lève une erreur quand le statut est `'error'`. |
| `hasValue`  | Indique si la ressource a une valeur.                                                                                                                                                        |
| `error`     | La dernière erreur rencontrée lors de l'exécution du loader de la ressource, ou `undefined` si aucune erreur ne s'est produite.                                                              |
| `isLoading` | Indique si le loader de la ressource est en cours d'exécution.                                                                                                                               |
| `status`    | Le `ResourceStatus` précis de la ressource, comme décrit ci-dessous.                                                                                                                         |

Le signal `status` fournit un `ResourceStatus` précis qui décrit l'état de la ressource à l'aide d'une constante de type chaîne.

| Statut        | `value()`         | Description                                                                  |
| ------------- | :---------------- | ---------------------------------------------------------------------------- |
| `'idle'`      | `undefined`       | La ressource n'a pas de requête valide et le loader ne s'est pas exécuté.    |
| `'error'`     | Lève une erreur   | Le loader a rencontré une erreur.                                            |
| `'loading'`   | `undefined`       | Le loader s'exécute suite au changement de la valeur `params`.               |
| `'reloading'` | Valeur précédente | Le loader s'exécute suite à l'appel de la méthode `reload` de la ressource.  |
| `'resolved'`  | Valeur résolue    | Le loader a terminé.                                                         |
| `'local'`     | Valeur définie localement | La valeur de la ressource a été définie localement via `.set()` ou `.update()` |

Vous pouvez utiliser ces informations de statut pour afficher conditionnellement des éléments d'interface utilisateur, comme des indicateurs de chargement et des messages d'erreur.

## Mettre en cache les données de `resource` avec le SSR {#caching-resource-data-with-ssr}

Quand une application est rendue sur le serveur, un loader de ressource s'exécute une fois pour produire le HTML initial. Pendant l'hydratation, le navigateur exécute normalement à nouveau le même loader.

Pour réutiliser le résultat du serveur, fournissez un `id` pour la ressource. Angular stocke la valeur résolue dans `TransferState` sur le serveur et l'utilise côté client pour initialiser la ressource dans un état `'resolved'`.

```ts
const userId: Signal<string> = getUserId();

const userResource = resource({
  params: () => ({id: userId()}),
  loader: ({params}) => fetchUser(params),
  id: 'user-unique-id',
});
```

La valeur de `id` doit être unique dans votre application et identique sur le serveur et le client, afin qu'Angular puisse faire correspondre l'entrée en cache à la ressource qui l'a demandée.

IMPORTANT: Comme la valeur en cache est sérialisée dans le HTML de la page, évitez de définir `id` sur les ressources qui chargent des données propres à l'utilisateur ayant déclenché le rendu côté serveur, en particulier si le HTML rendu peut être mis en cache ou partagé entre utilisateurs.

## Chaîner des ressources {#chaining-resources}

Parfois, une ressource dépend du résultat d'une autre. Vous pouvez exprimer cette dépendance avec la fonction `chain` disponible dans l'objet de contexte de `params`.

```typescript
import {resource} from '@angular/core';

const userResource = resource({
  params: () => ({id: getUserId()}),
  loader: ({params}) => fetchUser(params),
});

const companyResource = resource({
  params: ({chain}) => chain(userResource)?.companyId,
  loader: ({params: companyId}) => fetchCompany(companyId),
});
```

Ici, `companyResource` dépend du `companyId` de l'utilisateur, qui n'est connu qu'une fois `userResource` chargée. `chain(userResource)` lit la valeur de `userResource` et propage automatiquement son statut à `companyResource` :

- Si `userResource` est **idle**, `companyResource` devient également `idle`.
- Si `userResource` est **loading** ou **reloading**, `companyResource` passe à l'état `loading` et son loader ne s'exécute pas. Notez que pendant `reloading`, `chain` ne renvoie pas la valeur résolue précédemment.
- Si `userResource` est dans un état **error**, `companyResource` passe également à l'état `error`.
- Si `userResource` est **resolved** ou **local**, `chain` renvoie sa valeur actuelle, que `companyResource` utilise alors comme params.

Quand `chain` propage un statut de `userResource` (`idle`, `loading`, `reloading` ou `error`), la fonction params ne continue pas. Quand `userResource` est `resolved` ou `local`, `chain` renvoie sa valeur, qui peut elle-même être `undefined`. L'exemple gère ce cas avec `chain(userResource)?.companyId`, de sorte qu'une valeur `undefined` donne des params `undefined` et que `companyResource` devient `idle`.

NOTE: Passez directement la valeur chaînée comme valeur de params, au lieu de l'envelopper dans un objet. Une valeur de params comme `{companyId: undefined}` reste une valeur définie : le loader s'exécuterait donc avec un `companyId` `undefined` au lieu que la ressource devienne `idle`.

### Chaîner ou lire directement les valeurs d'une ressource {#chaining-vs-reading-resource-values-directly}

Vous pourriez être tenté de lire directement la valeur d'une ressource dans `params` :

```typescript {avoid, header: 'reads value() directly without status propagation'}
const companyResource = resource({
  params: () => {
    const user = userResource.value(); // may be undefined
    return user ? {companyId: user.companyId} : undefined;
  },
  loader: ({params}) => fetchCompany(params.companyId),
});
```

Même si cela fonctionne, renvoyer `undefined` depuis `params` fait passer la ressource à l'état `idle` au lieu de refléter l'état réel de la ressource en amont. Il est préférable d'utiliser `chain`, car cette fonction reflète correctement les états `loading` et `error`.

N'utilisez `chain` que lorsque la ressource en aval effectue son propre travail asynchrone qui dépend de la valeur en amont. Si vous avez seulement besoin de dériver une valeur de manière synchrone à partir d'une ressource, utilisez plutôt `computed`.

## Récupération de données réactive avec `httpResource` {#reactive-data-fetching-with-httpresource}

[`httpResource`](/guide/http/http-resource) est une enveloppe autour de `HttpClient` qui vous fournit le statut de la requête et la réponse sous forme de signals. Elle effectue les requêtes HTTP via la pile HTTP d'Angular, intercepteurs compris.

## Composition de ressources avec des snapshots {#resource-composition-with-snapshots}

Un `ResourceSnapshot` est une représentation structurée de l'état actuel d'une ressource. Chaque ressource possède une propriété `snapshot` qui fournit un signal de son état actuel.

```ts
const userId: Signal<string> = getUserId();

const userResource = resource({
  params: () => ({id: userId()}),
  loader: ({params}) => fetchUser(params),
});

const userSnapshot = userResource.snapshot;
```

Chaque snapshot contient un `status` et soit une `value`, soit une `error`.

### Composer des ressources avec des snapshots {#composing-resources-with-snapshots}

IMPORTANT: `resourceFromSnapshots` est [expérimental](reference/releases#experimental). Vous pouvez l'essayer dès maintenant, mais il pourrait changer avant d'être stable.

Vous pouvez créer de nouvelles ressources à partir de snapshots avec `resourceFromSnapshots`. Cela permet la composition avec des API de signals comme `computed` et `linkedSignal` pour transformer le comportement d'une ressource.

```ts
import {
  Component,
  input,
  linkedSignal,
  resourceFromSnapshots,
  Resource,
  ResourceSnapshot,
} from '@angular/core';
import {httpResource} from '@angular/common/http';

function withPreviousValue<T>(input: Resource<T>): Resource<T> {
  const derived = linkedSignal<ResourceSnapshot<T>, ResourceSnapshot<T>>({
    source: input.snapshot,
    computation: (snap, previous) => {
      if (snap.status === 'loading' && previous && previous.value.status !== 'error') {
        // When the input resource enters loading state, we keep the value
        // from its previous state, if any.
        return {status: 'loading' as const, value: previous.value.value};
      }

      // Otherwise we simply forward the state of the input resource.
      return snap;
    },
  });

  return resourceFromSnapshots(derived);
}

@Component({/*... */})
export class AwesomeProfile {
  userId = input.required<number>();
  user = withPreviousValue(httpResource(() => `/user/${this.userId()}`));
  // When userId changes, user.value() keeps the old user data until the new one loads
}
```
