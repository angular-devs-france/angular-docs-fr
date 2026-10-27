<!-- source: a05fc21a1d12ebc4de5b48acf8d31b254af20c93 -->
# Configurer `HttpClient`

`HttpClient` est disponible par défaut pour l'injection à partir d'Angular v21.

## Fournir `HttpClient` via l'injection de dépendances {#providing-httpclient-through-dependency-injection}

Vous pouvez utiliser la fonction utilitaire `provideHttpClient` pour configurer l'ensemble de fonctionnalités HTTP par défaut ou ajouter des fonctionnalités dans les `providers` de l'application, dans `app.config.ts`.

```ts
export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient(/* add features here, such as withInterceptors(...) */)],
};
```

Si votre application utilise plutôt un bootstrap basé sur les NgModules, vous pouvez inclure `provideHttpClient` dans les providers du NgModule de votre application pour configurer l'ensemble de fonctionnalités HTTP par défaut ou ajouter des fonctionnalités :

```ts
@NgModule({
  providers: [provideHttpClient(/* add features here, such as withInterceptors(...) */)],
  // ... other application configuration
})
export class AppModule {}
```

Vous pouvez ensuite injecter le service `HttpClient` en tant que dépendance de vos composants, services ou autres classes :

```ts
@Service()
export class ConfigService {
  private http = inject(HttpClient);
  // This service can now make HTTP requests via `this.http`.
}
```

## Configurer les fonctionnalités de `HttpClient` {#configuring-features-of-httpclient}

`provideHttpClient` accepte une liste de configurations de fonctionnalités optionnelles pour activer ou configurer différents aspects du client. Cette section détaille ces fonctionnalités optionnelles et leur utilisation.

### `withXhr` {#withxhr}

```ts
export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient(withXhr())],
};
```

Par défaut, `HttpClient` utilise l'API [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API) pour effectuer les requêtes. La fonctionnalité `withXhr` fait plutôt utiliser au client l'API [`XMLHttpRequest`](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest).

`fetch` est une API plus moderne, disponible dans quelques environnements où `XMLHttpRequest` n'est pas pris en charge. Elle présente toutefois quelques limitations, comme l'absence d'événements de progression pour l'envoi (upload).

<docs-callout critical title="N'utilisez pas `withXhr` dans les environnements de rendu côté serveur (SSR)">

La prise en charge de XHR côté serveur est **dépréciée** et doit être supprimée dans Angular 23. La library `xhr2` sous-jacente ne gère pas les redirections de manière sûre : elle peut transmettre les en-têtes `Authorization` lors de redirections cross-origin et est exposée à des dénis de service (DoS) via des boucles de redirection. Pour les applications SSR, utilisez plutôt le backend `fetch` par défaut.

</docs-callout>

### `withInterceptors(...)` {#withinterceptors}

`withInterceptors` configure l'ensemble des fonctions intercepteurs qui traiteront les requêtes effectuées via `HttpClient`. Consultez le [guide des intercepteurs](guide/http/interceptors) pour plus d'informations.

### `withInterceptorsFromDi()` {#withinterceptorsfromdi}

`withInterceptorsFromDi` inclut l'ancien style d'intercepteurs à base de classes dans la configuration de `HttpClient`. Consultez le [guide des intercepteurs](guide/http/interceptors) pour plus d'informations.

HELPFUL: Les intercepteurs fonctionnels (via `withInterceptors`) ont un ordre d'exécution plus prévisible et nous les recommandons plutôt que les intercepteurs basés sur la DI.

### `withRequestsMadeViaParent()` {#withrequestsmadeviaparent}

Par défaut, lorsque vous configurez `HttpClient` avec `provideHttpClient` dans un injector donné, cette configuration remplace toute configuration de `HttpClient` éventuellement présente dans l'injector parent.

Lorsque vous ajoutez `withRequestsMadeViaParent()`, `HttpClient` est au contraire configuré pour transmettre les requêtes à l'instance de `HttpClient` de l'injector parent, une fois qu'elles sont passées par les intercepteurs configurés à ce niveau. C'est utile si vous voulez _ajouter_ des intercepteurs dans un injector enfant tout en faisant passer la requête par les intercepteurs de l'injector parent.

CRITICAL: Vous devez configurer une instance de `HttpClient` au-dessus de l'injector courant ; sinon, cette option n'est pas valide et vous obtiendrez une erreur à l'exécution en essayant de l'utiliser.

### `withJsonpSupport()` {#withjsonpsupport}

<docs-callout critical title="JSONP est déprécié">

`withJsonpSupport`, `HttpClientJsonpModule` et la méthode `.jsonp()` de `HttpClient` sont dépréciés, car JSONP peut provoquer des vulnérabilités de cross-site scripting (XSS). Utilisez plutôt des requêtes HTTP standard avec [CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS).

</docs-callout>

Inclure `withJsonpSupport` active la méthode `.jsonp()` de `HttpClient`, qui effectue une requête GET via la [convention JSONP](https://en.wikipedia.org/wiki/JSONP) pour charger des données entre domaines.

HELPFUL: Préférez utiliser [CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS) plutôt que JSONP pour effectuer des requêtes entre domaines lorsque c'est possible.

### `withXsrfConfiguration(...)` {#withxsrfconfiguration}

Inclure cette option permet de personnaliser la fonctionnalité de sécurité XSRF intégrée à `HttpClient`. Consultez le [guide de sécurité](best-practices/security) pour plus d'informations.

### `withNoXsrfProtection()` {#withnoxsrfprotection}

Inclure cette option désactive la fonctionnalité de sécurité XSRF intégrée à `HttpClient`. Consultez le [guide de sécurité](best-practices/security) pour plus d'informations.

## Configuration basée sur `HttpClientModule` {#httpclientmodule-based-configuration}

Certaines applications configurent `HttpClient` avec l'ancienne API basée sur les NgModules.

Ce tableau liste les NgModules disponibles dans `@angular/common/http` et leur correspondance avec les fonctions de configuration de provider ci-dessus.

| **NgModule**                            | Équivalent `provideHttpClient()`                         |
| --------------------------------------- | -------------------------------------------------------- |
| `HttpClientModule`                      | `provideHttpClient(withInterceptorsFromDi(), withXhr())` |
| `HttpClientJsonpModule` (déprécié)      | `withJsonpSupport()` (déprécié)                          |
| `HttpClientXsrfModule.withOptions(...)` | `withXsrfConfiguration(...)`                             |
| `HttpClientXsrfModule.disable()`        | `withNoXsrfProtection()`                                 |

NOTE: `HttpClientModule` et `HttpClientXsrfModule` sont également dépréciés. Utilisez plutôt `provideHttpClient` avec les fonctionnalités correspondantes.

<docs-callout important title="Soyez prudent en utilisant HttpClientModule dans plusieurs injectors">
Lorsque `HttpClientModule` est présent dans plusieurs injectors, le comportement des intercepteurs est mal défini et dépend des options exactes et de l'ordre des providers et des imports.

Préférez `provideHttpClient` pour les configurations à plusieurs injectors, car son comportement est plus stable. Consultez la fonctionnalité `withRequestsMadeViaParent` ci-dessus.
</docs-callout>
