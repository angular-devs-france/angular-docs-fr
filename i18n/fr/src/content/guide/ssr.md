<!-- source: c30e340bb51f1248b71f988952a016ecb81ca260 -->
# Rendu serveur et hybride

Angular livre par défaut toutes les applications en rendu côté client (CSR). Si cette approche fournit une charge initiale légère, elle introduit des compromis : temps de chargement plus lents, métriques de performance dégradées et besoins en ressources plus élevés, puisque l'appareil de l'utilisateur effectue la plupart des calculs. C'est pourquoi de nombreuses applications obtiennent des gains de performances significatifs en intégrant le rendu côté serveur (SSR) dans une stratégie de rendu hybride.

## Qu'est-ce que le rendu hybride ? {#what-is-hybrid-rendering}

Le rendu hybride permet aux développeurs de tirer parti des avantages du rendu côté serveur (SSR), du prérendu (aussi appelé « génération de site statique » ou SSG) et du rendu côté client (CSR) pour optimiser votre application Angular. Il vous donne un contrôle fin sur la façon dont les différentes parties de votre application sont rendues, afin d'offrir à vos utilisateurs la meilleure expérience possible.

## Mettre en place le rendu hybride {#setting-up-hybrid-rendering}

Vous pouvez créer un **nouveau** projet avec rendu hybride en utilisant le flag de rendu côté serveur (c'est-à-dire `--ssr`) avec la commande `ng new` d'Angular CLI :

```shell
ng new --ssr
```

Vous pouvez aussi activer le rendu hybride en ajoutant le rendu côté serveur à un projet existant avec la commande `ng add` :

```shell
ng add @angular/ssr
```

NOTE: Par défaut, Angular prérend toute votre application et génère un fichier serveur. Pour désactiver ce comportement et créer une application entièrement statique, définissez `outputMode` à `static`. Pour activer le SSR, mettez à jour les routes serveur pour qu'elles utilisent `RenderMode.Server`. Pour plus de détails, consultez [`Server routing`](#server-routing) et [`Generate a fully static application`](#generate-a-fully-static-application).

## Routing côté serveur {#server-routing}

### Configurer les routes serveur {#configuring-server-routes}

Vous pouvez créer une configuration de routes serveur en déclarant un tableau d'objets [`ServerRoute`](api/ssr/ServerRoute 'API reference'). Cette configuration se trouve généralement dans un fichier nommé `app.routes.server.ts`.

```typescript {header: "app.routes.server.ts"}
import {RenderMode, ServerRoute} from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: '', // This renders the "/" route on the client (CSR)
    renderMode: RenderMode.Client,
  },
  {
    path: 'about', // This page is static, so we prerender it (SSG)
    renderMode: RenderMode.Prerender,
  },
  {
    path: 'profile', // This page requires user-specific data, so we use SSR
    renderMode: RenderMode.Server,
  },
  {
    path: '**', // All other routes will be rendered on the server (SSR)
    renderMode: RenderMode.Server,
  },
];
```

Vous pouvez ajouter cette configuration à votre application avec [`provideServerRendering`](api/ssr/provideServerRendering 'API reference') en utilisant la fonction [`withRoutes`](api/ssr/withRoutes 'API reference') :

```typescript {header: "app.config.server.ts"}
import {provideServerRendering, withRoutes} from '@angular/ssr';
import {ApplicationConfig} from '@angular/core';
import {serverRoutes} from './app.routes.server';

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering(withRoutes(serverRoutes)),
    // ... other providers ...
  ],
};
```

Lorsque vous utilisez le [pattern App shell](ecosystem/service-workers/app-shell), vous devez indiquer le composant à utiliser comme app shell pour les routes rendues côté client. Pour cela, utilisez la fonctionnalité [`withAppShell`](api/ssr/withAppShell 'API reference') :

```typescript
import {provideServerRendering, withRoutes, withAppShell} from '@angular/ssr';
import {ApplicationConfig} from '@angular/core';
import {AppShell} from './app-shell';

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering(withRoutes(serverRoutes), withAppShell(AppShell)),
    // ... other providers ...
  ],
};
```

### Modes de rendu {#rendering-modes}

La configuration du routing côté serveur vous permet d'indiquer comment chaque route de votre application doit être rendue en définissant un [`RenderMode`](api/ssr/RenderMode 'API reference') :

| Mode de rendu       | Description                                                                                                 |
| ------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Serveur (SSR)**   | Rend l'application sur le serveur pour chaque requête, en envoyant au navigateur une page HTML entièrement remplie. |
| **Client (CSR)**    | Rend l'application dans le navigateur. C'est le comportement par défaut d'Angular.                               |
| **Prérendu (SSG)**  | Prérend l'application au moment du build, en générant des fichiers HTML statiques pour chaque route.                      |

#### Choisir un mode de rendu {#choosing-a-rendering-mode}

Chaque mode de rendu présente des avantages et des inconvénients différents. Vous pouvez choisir les modes de rendu en fonction des besoins spécifiques de votre application.

##### Rendu côté client (CSR) {#client-side-rendering-csr}

Le rendu côté client offre le modèle de développement le plus simple, puisque vous pouvez écrire du code en supposant qu'il s'exécute toujours dans un navigateur web. Cela vous permet d'utiliser un large éventail de libraries côté client qui supposent, elles aussi, s'exécuter dans un navigateur.

Le rendu côté client offre généralement de moins bonnes performances que les autres modes de rendu, car il doit télécharger, analyser et exécuter le JavaScript de votre page avant que l'utilisateur puisse voir le moindre contenu rendu. Si votre page récupère d'autres données depuis le serveur pendant le rendu, les utilisateurs doivent aussi attendre ces requêtes supplémentaires avant de pouvoir consulter le contenu complet.

Si votre page est indexée par des robots d'indexation, le rendu côté client peut nuire au référencement (SEO), car les robots d'indexation limitent la quantité de JavaScript qu'ils exécutent lorsqu'ils indexent une page.

Avec le rendu côté client, le serveur n'a aucun travail à effectuer pour rendre une page, hormis servir des ressources JavaScript statiques. Vous pouvez tenir compte de ce facteur si le coût du serveur vous préoccupe.

Les applications qui proposent des expériences installables et hors ligne grâce aux [service workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) peuvent s'appuyer sur le rendu côté client sans avoir besoin de communiquer avec un serveur.

##### Rendu côté serveur (SSR) {#server-side-rendering-ssr}

Le rendu côté serveur offre des chargements de page plus rapides que le rendu côté client. Au lieu d'attendre le téléchargement et l'exécution du JavaScript, le serveur rend directement un document HTML dès qu'il reçoit une requête du navigateur. L'utilisateur ne subit que la latence nécessaire au serveur pour récupérer les données et rendre la page demandée. Ce mode élimine aussi le besoin de requêtes réseau supplémentaires depuis le navigateur, puisque votre code peut récupérer les données pendant le rendu sur le serveur.

Le rendu côté serveur offre généralement un excellent référencement (SEO), car les robots d'indexation reçoivent un document HTML entièrement rendu.

Le rendu côté serveur vous oblige à écrire du code qui ne dépend pas strictement des API du navigateur, et restreint votre choix parmi les libraries JavaScript qui supposent s'exécuter dans un navigateur.

Avec le rendu côté serveur, votre serveur exécute Angular pour produire une réponse HTML à chaque requête, ce qui peut augmenter les coûts d'hébergement du serveur.

##### Prérendu au moment du build {#build-time-prerendering}

Le prérendu offre des chargements de page plus rapides que le rendu côté client et que le rendu côté serveur. Comme le prérendu crée les documents HTML _au moment du build_, le serveur peut répondre directement aux requêtes avec le document HTML statique, sans aucun travail supplémentaire.

Le prérendu exige que toutes les informations nécessaires au rendu d'une page soient disponibles _au moment du build_. Les pages prérendues ne peuvent donc inclure aucune donnée propre à l'utilisateur qui charge la page. Le prérendu est surtout utile pour les pages identiques pour tous les utilisateurs de votre application.

Comme le prérendu a lieu au moment du build, il peut allonger sensiblement vos builds de production. Utiliser [`getPrerenderParams`](api/ssr/ServerRoutePrerenderWithParams#getPrerenderParams 'API reference') pour produire un grand nombre de documents HTML peut affecter la taille totale des fichiers de vos déploiements, et donc ralentir ces derniers.

Le prérendu offre généralement un excellent référencement (SEO), car les robots d'indexation reçoivent un document HTML entièrement rendu.

Le prérendu vous oblige à écrire du code qui ne dépend pas strictement des API du navigateur, et restreint votre choix parmi les libraries JavaScript qui supposent s'exécuter dans un navigateur.

Le prérendu n'entraîne qu'un surcoût infime par requête serveur, puisque votre serveur répond avec des documents HTML statiques. Les fichiers statiques sont aussi facilement mis en cache par les Content Delivery Networks (CDN), les navigateurs et les couches de cache intermédiaires, pour des chargements de page suivants encore plus rapides. Les sites entièrement statiques peuvent aussi être déployés uniquement via un CDN ou un serveur de fichiers statiques, ce qui évite de maintenir un runtime serveur personnalisé pour votre application. Cela améliore la scalabilité en déchargeant le serveur web de l'application, ce qui est particulièrement intéressant pour les applications à fort trafic.

NOTE: Avec le service worker d'Angular, la première requête est rendue par le serveur, mais toutes les requêtes suivantes sont gérées par le service worker et rendues côté client.

### Définir les en-têtes et les codes de statut {#setting-headers-and-status-codes}

Vous pouvez définir des en-têtes et des codes de statut personnalisés pour des routes serveur individuelles grâce aux propriétés `headers` et `status` de la configuration `ServerRoute`.

```typescript {header: "app.routes.server.ts"}
import {RenderMode, ServerRoute} from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: 'profile',
    renderMode: RenderMode.Server,
    headers: {
      'X-My-Custom-Header': 'some-value',
    },
    status: 201,
  },
  // ... other routes
];
```

### Redirections {#redirects}

Angular gère différemment, côté serveur, les redirections indiquées par la propriété [`redirectTo`](api/router/Route#redirectTo 'API reference') dans les configurations de routes.

**Rendu côté serveur (SSR)**
Les redirections sont effectuées à l'aide de redirections HTTP standard (par exemple 301, 302) pendant le processus de rendu côté serveur.

**Prérendu (SSG)**
Les redirections sont implémentées comme des « soft redirects » à l'aide de balises [`<meta http-equiv="refresh">`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta#refresh) dans le HTML prérendu.

### Personnaliser le prérendu au moment du build (SSG) {#customizing-build-time-prerendering-ssg}

Lorsque vous utilisez [`RenderMode.Prerender`](api/ssr/RenderMode#Prerender 'API reference'), vous pouvez indiquer plusieurs options de configuration pour personnaliser le processus de prérendu et de diffusion.

#### Routes paramétrées {#parameterized-routes}

Pour chaque route avec [`RenderMode.Prerender`](api/ssr/RenderMode#Prerender 'API reference'), vous pouvez indiquer une fonction [`getPrerenderParams`](api/ssr/ServerRoutePrerenderWithParams#getPrerenderParams 'API reference'). Cette fonction vous permet de contrôler quels paramètres produisent des documents prérendus distincts.

La fonction [`getPrerenderParams`](api/ssr/ServerRoutePrerenderWithParams#getPrerenderParams 'API reference') renvoie une `Promise` qui se résout en un tableau d'objets. Chaque objet est une map clé-valeur qui associe le nom d'un paramètre de route à sa valeur. Par exemple, si vous définissez une route comme `post/:id`, `getPrerenderParams ` pourrait renvoyer le tableau `[{id: '123'}, {id: '456'}]`, et ainsi rendre des documents distincts pour `post/123` et `post/456`.

Le corps de [`getPrerenderParams`](api/ssr/ServerRoutePrerenderWithParams#getPrerenderParams 'API reference') peut utiliser la fonction [`inject`](api/core/inject 'API reference') d'Angular pour injecter des dépendances et effectuer tout traitement permettant de déterminer quelles routes prérendre. Cela inclut généralement des requêtes pour récupérer les données nécessaires à la construction du tableau de valeurs de paramètres.

Vous pouvez aussi utiliser cette fonction avec des routes wildcard (par exemple `/**`) : le nom du paramètre sera alors `"**"` et la valeur renvoyée sera les segments du chemin, comme `foo/bar`. Vous pouvez les combiner avec d'autres paramètres (par exemple `/post/:id/**`) pour gérer des configurations de routes plus complexes.

```ts {header: "app.routes.server.ts"}
import {RenderMode, ServerRoute} from '@angular/ssr';
import {inject} from '@angular/core';

export const serverRoutes: ServerRoute[] = [
  {
    path: 'post/:id',
    renderMode: RenderMode.Prerender,
    async getPrerenderParams() {
      const dataService = inject(PostService);
      const ids = await dataService.getIds(); // Assuming this returns ['1', '2', '3']

      return ids.map((id) => ({id})); // Generates paths like: /post/1, /post/2, /post/3
    },
  },
  {
    path: 'post/:id/**',
    renderMode: RenderMode.Prerender,
    async getPrerenderParams() {
      return [
        {id: '1', '**': 'foo/3'},
        {id: '2', '**': 'bar/4'},
      ]; // Generates paths like: /post/1/foo/3, /post/2/bar/4
    },
  },
];
```

Comme [`getPrerenderParams`](api/ssr/ServerRoutePrerenderWithParams#getPrerenderParams 'API reference') s'applique exclusivement à [`RenderMode.Prerender`](api/ssr/RenderMode#Prerender 'API reference'), cette fonction s'exécute toujours _au moment du build_. `getPrerenderParams` ne doit dépendre d'aucune API propre au navigateur ou au serveur pour obtenir ses données.

IMPORTANT: Lorsque vous utilisez [`inject`](api/core/inject 'API reference') dans `getPrerenderParams`, n'oubliez pas que `inject` doit être utilisé de manière synchrone. Il ne peut pas être appelé dans des callbacks asynchrones ni après une instruction `await`. Pour plus d'informations, consultez `runInInjectionContext`.

#### Stratégies de repli {#fallback-strategies}

Lorsque vous utilisez le mode [`RenderMode.Prerender`](api/ssr/RenderMode#Prerender 'API reference'), vous pouvez indiquer une stratégie de repli pour gérer les requêtes vers des chemins qui n'ont pas été prérendus.

Les stratégies de repli disponibles sont :

- **Server :** se replie sur le rendu côté serveur. C'est le comportement **par défaut** si aucune propriété `fallback` n'est indiquée.
- **Client :** se replie sur le rendu côté client.
- **None :** aucun repli. Angular ne gère pas les requêtes vers des chemins qui ne sont pas prérendus.

```ts {header: "app.routes.server.ts"}
import {RenderMode, PrerenderFallback, ServerRoute} from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: 'post/:id',
    renderMode: RenderMode.Prerender,
    fallback: PrerenderFallback.Client, // Fallback to CSR if not prerendered
    async getPrerenderParams() {
      // This function returns an array of objects representing prerendered posts at the paths:
      // `/post/1`, `/post/2`, and `/post/3`.
      // The path `/post/4` will utilize the fallback behavior if it's requested.
      return [{id: '1'}, {id: '2'}, {id: '3'}];
    },
  },
];
```

## Écrire des composants compatibles avec le serveur {#authoring-server-compatible-components}

Certaines API et capacités courantes du navigateur peuvent ne pas être disponibles sur le serveur. Les applications ne peuvent pas utiliser les objets globaux propres au navigateur comme `window`, `document`, `navigator` ou `location`, ni certaines propriétés de `HTMLElement`.

En général, le code qui dépend de symboles propres au navigateur ne doit être exécuté que dans le navigateur, et non sur le serveur. Vous pouvez l'imposer grâce aux lifecycle hooks `afterEveryRender` et `afterNextRender`. Ceux-ci ne sont exécutés que dans le navigateur et sont ignorés sur le serveur.

```angular-ts
import {Component, ElementRef, viewChild, afterNextRender} from '@angular/core';

@Component({
  selector: 'my-cmp',
  template: `<span #content>{{ ... }}</span>`,
})
export class MyComponent {
  contentRef = viewChild.required<ElementRef>('content');

  constructor() {
    afterNextRender(() => {
      // Safe to check `scrollHeight` because this will only run in the browser, not the server.
      console.log('content height: ' + this.contentRef().nativeElement.scrollHeight);
    });
  }
}
```

NOTE: Préférez les [providers propres à chaque plateforme](guide/ssr#providing-platform-specific-implementations) aux vérifications à l'exécution avec `isPlatformBrowser` ou `isPlatformServer`.

IMPORTANT: Évitez d'utiliser `isPlatformBrowser` dans les templates avec `@if` ou d'autres conditions pour rendre un contenu différent sur le serveur et sur le client. Cela provoque des incohérences d'hydratation et des décalages de mise en page (layout shifts), ce qui dégrade l'expérience utilisateur et les [Core Web Vitals](https://web.dev/learn-core-web-vitals/). Utilisez plutôt `afterNextRender` pour l'initialisation propre au navigateur et gardez un contenu rendu identique sur toutes les plateformes.

## Définir les providers sur le serveur {#setting-providers-on-the-server}

Côté serveur, les valeurs des providers de premier niveau sont définies une seule fois, lorsque le code de l'application est initialement analysé et évalué.
Cela signifie que les providers configurés avec `useValue` conservent leur valeur d'une requête à l'autre, jusqu'au redémarrage de l'application serveur.

Si vous voulez générer une nouvelle valeur pour chaque requête, utilisez un provider de fabrique avec `useFactory`. La fonction de fabrique s'exécute pour chaque requête entrante, ce qui garantit qu'une nouvelle valeur est créée et associée au jeton à chaque fois.

## Fournir des implémentations propres à chaque plateforme {#providing-platform-specific-implementations}

Lorsque votre application a besoin d'un comportement différent dans le navigateur et sur le serveur, fournissez des implémentations de service distinctes pour chaque plateforme. Cette approche centralise la logique propre à chaque plateforme dans des services dédiés.

```ts
export abstract class AnalyticsService {
  abstract trackEvent(name: string): void;
}
```

Créez l'implémentation pour le navigateur :

```ts
@Injectable()
export class BrowserAnalyticsService implements AnalyticsService {
  trackEvent(name: string): void {
    // Sends the event to the browser-based third-party analytics provider
  }
}
```

Créez l'implémentation pour le serveur :

```ts
@Injectable()
export class ServerAnalyticsService implements AnalyticsService {
  trackEvent(name: string): void {
    // Records the event on the server
  }
}
```

Enregistrez l'implémentation pour le navigateur dans la configuration principale de votre application :

```ts {header: "app.config.ts"}
export const appConfig: ApplicationConfig = {
  providers: [{provide: AnalyticsService, useClass: BrowserAnalyticsService}],
};
```

Remplacez-la par l'implémentation serveur dans votre configuration serveur :

```ts {header: "app.config.server.ts"}
const serverConfig: ApplicationConfig = {
  providers: [{provide: AnalyticsService, useClass: ServerAnalyticsService}],
};
```

Injectez et utilisez le service dans vos composants :

```ts
@Component(/* ... */)
export class Checkout {
  private analytics = inject(AnalyticsService);

  onAction() {
    this.analytics.trackEvent('action');
  }
}
```

## Accéder à Document via la DI {#accessing-document-via-di}

Lorsque vous travaillez avec le rendu côté serveur, évitez de référencer directement des variables globales propres au navigateur comme `document`. Utilisez plutôt le jeton [`DOCUMENT`](api/core/DOCUMENT) pour accéder à l'objet document de manière indépendante de la plateforme.

```ts
import {inject, DOCUMENT, Service} from '@angular/core';

@Service()
export class CanonicalLinkService {
  private readonly document = inject(DOCUMENT);

  // During server rendering, inject a <link rel="canonical"> tag
  // so the generated HTML includes the correct canonical URL
  setCanonical(href: string): void {
    const link = this.document.createElement('link');
    link.rel = 'canonical';
    link.href = href;
    this.document.head.appendChild(link);
  }
}
```

HELPFUL: Pour gérer les balises meta, Angular fournit le service `Meta`.

## Accéder à la requête et à la réponse via la DI {#accessing-request-and-response-via-di}

Le package `@angular/core` fournit plusieurs jetons pour interagir avec l'environnement de rendu côté serveur. Ces jetons vous donnent accès à des informations et à des objets essentiels dans votre application Angular pendant le SSR.

- **[`REQUEST`](api/core/REQUEST 'API reference') :** donne accès à l'objet de la requête courante, de type [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) de la Web API. Cela vous permet d'accéder aux en-têtes, aux cookies et aux autres informations de la requête.
- **[`RESPONSE_INIT`](api/core/RESPONSE_INIT 'API reference') :** donne accès aux options d'initialisation de la réponse, de type [`ResponseInit`](https://developer.mozilla.org/en-US/docs/Web/API/Response/Response#parameters) de la Web API. Cela vous permet de définir dynamiquement les en-têtes et le code de statut de la réponse. Utilisez ce jeton pour définir les en-têtes ou les codes de statut qui doivent être déterminés à l'exécution.
- **[`REQUEST_CONTEXT`](api/core/REQUEST_CONTEXT 'API reference') :** donne accès à un contexte supplémentaire lié à la requête courante. Ce contexte peut être passé comme second paramètre de la fonction [`handle`](api/ssr/AngularAppEngine#handle 'API reference'). On l'utilise généralement pour fournir des informations supplémentaires liées à la requête qui ne font pas partie de la Web API standard.

```angular-ts
import {inject, REQUEST, Component} from '@angular/core';

@Component({
  selector: 'app-my-component',
  template: `<h1>My Component</h1>`,
})
export class MyComponent {
  constructor() {
    const request = inject(REQUEST);
    console.log(request?.url);
  }
}
```

<!-- UL is used below as otherwise the list will not be include as part of the note. -->
<!-- prettier-ignore-start -->

IMPORTANT: Les jetons ci-dessus valent `null` dans les cas suivants :<ul class="docs-list">
  <li>Pendant les processus de build.</li>
  <li>Lorsque l'application est rendue dans le navigateur (CSR).</li>
  <li>Lors de la génération de site statique (SSG).</li>
  <li>Pendant l'extraction des routes en développement (au moment de la requête).</li>
</ul>

<!-- prettier-ignore-end -->

## Générer une application entièrement statique {#generate-a-fully-static-application}

Par défaut, Angular prérend toute votre application et génère un fichier serveur pour gérer les requêtes. Cela permet à votre application de servir du contenu prérendu aux utilisateurs. Cependant, si vous préférez un site entièrement statique sans serveur, vous pouvez désactiver ce comportement en définissant `outputMode` à `static` dans votre fichier de configuration `angular.json`.

Lorsque `outputMode` vaut `static`, Angular génère des fichiers HTML prérendus pour chaque route au moment du build, mais il ne génère pas de fichier serveur et ne nécessite pas de serveur Node.js pour servir l'application. C'est utile pour un déploiement chez des hébergeurs statiques, lorsqu'aucun serveur backend n'est nécessaire.

Pour cela, mettez à jour votre fichier `angular.json` comme suit :

```json
{
  "projects": {
    "your-app": {
      "architect": {
        "build": {
          "options": {
            "outputMode": "static"
          }
        }
      }
    }
  }
}
```

## Mettre en cache les données avec HttpClient {#caching-data-when-using-httpclient}

`HttpClient` met en cache les requêtes réseau sortantes lorsqu'il s'exécute sur le serveur. Ces informations sont sérialisées et transférées au navigateur dans le HTML initial envoyé par le serveur. Dans le navigateur, `HttpClient` vérifie s'il dispose de données dans le cache et, si c'est le cas, les réutilise au lieu d'effectuer une nouvelle requête HTTP pendant le rendu initial de l'application. `HttpClient` cesse d'utiliser le cache dès qu'une application devient [stable](api/core/ApplicationRef#isStable) dans le navigateur.

### Configurer la taille maximale du corps de réponse {#configuring-the-response-body-size-limit}

Lorsque `HttpClient` utilise le backend fetch par défaut pendant le rendu côté serveur, Angular limite chaque corps de réponse à 1 Mo. Cette limite empêche le serveur de mettre en mémoire tampon des réponses anormalement volumineuses pendant le rendu. Si une réponse dépasse la limite configurée, la requête échoue avec l'erreur [NG02825](errors/NG02825).

Si votre application doit récupérer des réponses plus volumineuses pendant le rendu serveur, définissez `maxResponseBodySize` dans les options de `provideServerRendering` :

```ts
import {provideServerRendering, withRoutes} from '@angular/ssr';
import {ApplicationConfig} from '@angular/core';
import {serverRoutes} from './app.routes.server';

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering(
      {
        maxResponseBodySize: 5 * 1024 * 1024, // 5MB
      },
      withRoutes(serverRoutes),
    ),
  ],
};
```

`maxResponseBodySize` s'exprime en octets et s'applique globalement aux requêtes `HttpClient` côté serveur qui utilisent le backend fetch.

IMPORTANT: Gardez cette limite aussi basse que votre application le permet. L'augmenter permet aux requêtes côté serveur de mettre en mémoire tampon des corps de réponse plus volumineux, ce qui peut accroître la consommation mémoire et le risque de déni de service. Déplacez de préférence les téléchargements volumineux en dehors du rendu serveur.

CRITICAL: Pendant le SSR, l'implémentation de Fetch lit l'intégralité du corps de la réponse pour vérifier `integrity` avant de renvoyer une réponse, comme l'exige le [Fetch Standard](https://fetch.spec.whatwg.org/#concept-main-fetch). Angular n'applique [`maxResponseBodySize`](/guide/ssr#configuring-the-response-body-size-limit) qu'une fois que Fetch a renvoyé une réponse ; cette limite ne contraint donc pas les données mises en mémoire tampon pendant la vérification d'intégrité.

### Configurer les options de mise en cache {#configuring-the-caching-options}

Vous pouvez personnaliser la façon dont Angular met en cache les réponses HTTP pendant le rendu côté serveur (SSR) et les réutilise pendant l'hydratation en configurant `HttpTransferCacheOptions`.  
Cette configuration est fournie globalement avec `withHttpTransferCacheOptions` dans `provideClientHydration()`.

Par défaut, `HttpClient` met en cache toutes les requêtes `HEAD` et `GET` qui ne contiennent pas d'en-tête `Authorization`, `Proxy-Authorization` ou `Cookie` et qui ne sont pas envoyées avec `withCredentials` ou avec des modes `credentials` de la Fetch API susceptibles d'envoyer des identifiants. Angular ignore aussi le transfer cache lorsqu'une requête ou une réponse contient des directives `Cache-Control` qui interdisent la mise en cache (`no-store`, `no-cache` ou `private`), ou lorsque l'option `cache` de la Fetch API vaut `no-store` ou `no-cache`. Les réponses qui portent un en-tête `Set-Cookie` sont également ignorées. Vous pouvez remplacer les paramètres de filtrage des requêtes avec `withHttpTransferCacheOptions` dans la configuration de l'hydratation.

```ts
import {
  bootstrapApplication,
  provideClientHydration,
  withHttpTransferCacheOptions,
} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [
    provideClientHydration(
      withHttpTransferCacheOptions({
        includeHeaders: ['ETag', 'Cache-Control'],
        filter: (req) => !req.url.includes('/api/profile'),
        includePostRequests: true,
        includeRequestsWithAuthHeaders: false,
      }),
    ),
  ],
});
```

### `includeHeaders` {#includeheaders}

Indique quels en-têtes de la réponse du serveur doivent être inclus dans les entrées mises en cache.  
Aucun en-tête n'est inclus par défaut.

```ts
withHttpTransferCacheOptions({
  includeHeaders: ['ETag', 'Cache-Control'],
});
```

IMPORTANT: Évitez d'inclure des en-têtes sensibles comme les jetons d'authentification. Ils peuvent faire fuiter des données propres à un utilisateur d'une requête à l'autre.

Inclure `Cache-Control` dans `includeHeaders` rend seulement cet en-tête disponible dans la réponse hydratée. Angular évalue déjà automatiquement les en-têtes `Cache-Control` pour décider si une requête ou une réponse est éligible au transfer cache.

### `includePostRequests` {#includepostrequests}

Par défaut, seules les requêtes `GET` et `HEAD` sont mises en cache.  
Vous pouvez activer la mise en cache des requêtes `POST` lorsqu'elles servent d'opérations de lecture, comme les requêtes GraphQL.

```ts
withHttpTransferCacheOptions({
  includePostRequests: true,
});
```

N'utilisez cette option que lorsque les requêtes `POST` sont **idempotentes** et peuvent être réutilisées sans risque entre le rendu serveur et le rendu client.

### `includeRequestsWithAuthHeaders` {#includerequestswithauthheaders}

Détermine si les requêtes contenant les en-têtes `Authorization`, `Proxy‑Authorization` ou `Cookie` sont éligibles à la mise en cache.  
Par défaut, elles sont exclues pour éviter de mettre en cache des réponses propres à un utilisateur.

```ts
withHttpTransferCacheOptions({
  includeRequestsWithAuthHeaders: true,
});
```

Activez cette option uniquement lorsque les en-têtes d'authentification n'affectent **pas** le contenu de la réponse (par exemple, des jetons publics pour des API d'analytics).

### `includeRequestsWithCredentials` {#includerequestswithcredentials}

Détermine si les requêtes envoyées avec `withCredentials` ou avec les modes `credentials` de la Fetch API (`include` ou `same-origin`) sont éligibles à la mise en cache.  
Par défaut, elles sont exclues pour éviter de mettre en cache des réponses propres à un utilisateur.

```ts
withHttpTransferCacheOptions({
  includeRequestsWithCredentials: true,
});
```

Activez cette option uniquement lorsque les requêtes avec identifiants renvoient des réponses qui peuvent être mises en cache sans risque.

### `includeNonCacheableRequests` {#includenoncacheablerequests}

Détermine si les requêtes et les réponses contenant des directives `Cache-Control` qui interdisent la mise en cache (`no-store`, `no-cache` ou `private`), les réponses avec un en-tête `Set-Cookie`, ou les requêtes utilisant les options `cache` de la Fetch API (`no-store` ou `no-cache`) sont éligibles à la mise en cache.  
Par défaut, elles sont exclues afin de respecter les directives de mise en cache HTTP.

```ts
withHttpTransferCacheOptions({
  includeNonCacheableRequests: true,
});
```

Activez cette option uniquement lorsque vous devez contourner les restrictions de cache-control pour le transfer cache.

### Surcharges par requête {#perrequest-overrides}

Vous pouvez surcharger le comportement de mise en cache d'une requête précise avec l'option de requête `transferCache`.

```ts
// Include specific headers for this request
http.get('/api/profile', {transferCache: {includeHeaders: ['CustomHeader']}});
```

### Désactiver la mise en cache {#disabling-caching}

Vous pouvez désactiver la mise en cache HTTP des requêtes envoyées depuis le serveur, soit globalement, soit individuellement.

#### Globalement {#globally}

Pour désactiver la mise en cache de toutes les requêtes de votre application, utilisez la fonctionnalité `withNoHttpTransferCache` :

```ts
import {
  bootstrapApplication,
  provideClientHydration,
  withNoHttpTransferCache,
} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideClientHydration(withNoHttpTransferCache())],
});
```

#### Filtrage {#filtering}

Vous pouvez aussi désactiver sélectivement la mise en cache de certaines requêtes avec l'option [`filter`](api/common/http/HttpTransferCacheOptions) de `withHttpTransferCacheOptions`. Par exemple, vous pouvez désactiver la mise en cache pour un endpoint d'API précis :

```ts
import {
  bootstrapApplication,
  provideClientHydration,
  withHttpTransferCacheOptions,
} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [
    provideClientHydration(
      withHttpTransferCacheOptions({
        filter: (req) => !req.url.includes('/api/sensitive-data'),
      }),
    ),
  ],
});
```

Utilisez cette option pour exclure les endpoints qui renvoient des données dynamiques ou propres à un utilisateur (par exemple `/api/profile`).

#### Par requête {#per-request}

Pour désactiver la mise en cache d'une requête individuelle, vous pouvez indiquer l'option [`transferCache`](api/common/http/HttpRequest#transferCache) dans une `HttpRequest`.

```ts
httpClient.get('/api/sensitive-data', {transferCache: false});
```

`HttpTransferCache` ne met pas en cache les requêtes ou les réponses qui refusent explicitement la mise en cache. Angular ignore les entrées du transfer cache lorsqu'une requête contient un en-tête `Cache-Control` avec `no-store`, `no-cache` ou `private`, ou lorsque la requête utilise l'option `cache` de la Fetch API définie à `no-store` ou `no-cache`. Les réponses avec `Cache-Control: no-store`, `Cache-Control: no-cache` ou `Cache-Control: private` ne sont pas non plus stockées dans le transfer cache. Les réponses qui contiennent un en-tête `Set-Cookie` ne sont pas stockées non plus, car elles portent généralement un état propre à l'utilisateur.

NOTE: Si votre application utilise des origines HTTP différentes pour effectuer les appels d'API sur le serveur et sur le client, le jeton `HTTP_TRANSFER_CACHE_ORIGIN_MAP` vous permet d'établir une correspondance entre ces origines, afin que la fonctionnalité `HttpTransferCache` reconnaisse ces requêtes comme identiques et réutilise, pendant l'hydratation côté client, les données mises en cache sur le serveur.

## Configurer un serveur {#configuring-a-server}

### Node.js {#nodejs}

`@angular/ssr/node` étend `@angular/ssr` spécifiquement pour les environnements Node.js. Il fournit des API qui facilitent la mise en œuvre du rendu côté serveur dans votre application Node.js. Pour la liste complète des fonctions et des exemples d'utilisation, consultez la [référence de l'API `@angular/ssr/node`](api/ssr/node/AngularNodeAppEngine).

```ts {header: "server.ts"}
import {
  AngularNodeAppEngine,
  createNodeRequestHandler,
  writeResponseToNodeResponse,
} from '@angular/ssr/node';
import express from 'express';

const app = express();
const angularApp = new AngularNodeAppEngine();

app.use((req, res, next) => {
  angularApp
    .handle(req)
    .then((response) => {
      if (response) {
        writeResponseToNodeResponse(response, res);
      } else {
        next(); // Pass control to the next middleware
      }
    })
    .catch(next);
});

/**
 * The request handler used by the Angular CLI (dev-server and during build).
 */
export const reqHandler = createNodeRequestHandler(app);
```

### Hors Node.js {#non-nodejs}

`@angular/ssr` fournit les API essentielles pour effectuer le rendu côté serveur de votre application Angular sur des plateformes autres que Node.js. Il s'appuie sur les objets standard [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) et [`Response`](https://developer.mozilla.org/en-US/docs/Web/API/Response) de la Web API, ce qui vous permet d'intégrer le SSR d'Angular dans divers environnements serveur. Pour des informations détaillées et des exemples, consultez la [référence de l'API `@angular/ssr`](api/ssr/AngularAppEngine).

```ts {header: "server.ts"}
import {AngularAppEngine, createRequestHandler} from '@angular/ssr';

const angularApp = new AngularAppEngine();

/**
 * This is a request handler used by the Angular CLI (dev-server and during build).
 */
export const reqHandler = createRequestHandler(async (req: Request) => {
  const res: Response | null = await angularApp.handle(req);

  // ...
});
```

## Sécurité {#security}

Pour des informations détaillées sur la prévention de la falsification de requête côté serveur (SSRF) et la configuration des hôtes autorisés, consultez le guide [Sécurité côté serveur](best-practices/security#preventing-server-side-request-forgery-ssrf).
