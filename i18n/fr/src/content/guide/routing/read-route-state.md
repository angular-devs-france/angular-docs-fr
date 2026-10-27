<!-- source: b23421fbd8b4b444442216954ea422658f1e1690 -->
# Lire l'état de la route

Le router d'Angular vous permet de lire et d'utiliser les informations associées à une route pour créer des composants réactifs et conscients de leur contexte.

## Obtenir des informations sur la route courante avec ActivatedRoute {#get-information-about-the-current-route-with-activatedroute}

`ActivatedRoute` est un service de `@angular/router` qui fournit toutes les informations associées à la route courante.

```angular-ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-product',
})
export class Product {
  private activatedRoute = inject(ActivatedRoute);

  constructor() {
    console.log(this.activatedRoute);
  }
}
```

`ActivatedRoute` peut fournir différentes informations sur la route. Voici quelques propriétés courantes :

| Propriété     | Détails                                                                                                                           |
| :------------ | :-------------------------------------------------------------------------------------------------------------------------------- |
| `url`         | Un `Observable` des chemins de la route, représentés sous forme de tableau d'objets `UrlSegment` pour chaque partie du chemin de la route. |
| `data`        | Un `Observable` qui contient l'objet `data` fourni pour la route. Contient aussi les valeurs résolues par la resolve guard. |
| `params`      | Un `Observable` qui contient les paramètres obligatoires et facultatifs propres à la route.                                         |
| `queryParams` | Un `Observable` qui contient les query parameters disponibles pour toutes les routes.                                                       |
| `resources`   | Un enregistrement facultatif d'instances de `Resource` définies sur la route (quand `withRouterResources` est activé).                          |

Consultez la [documentation de l'API `ActivatedRoute`](/api/router/ActivatedRoute) pour la liste complète de ce à quoi vous pouvez accéder dans la route.

## Comprendre les snapshots de route {#understanding-route-snapshots}

Les navigations entre pages sont des événements dans le temps, et vous pouvez accéder à l'état du router à un instant donné en récupérant un snapshot de la route.

Les snapshots de route contiennent les informations essentielles sur la route, notamment ses paramètres, ses données et ses routes enfants. De plus, les snapshots sont statiques et ne refléteront pas les changements futurs.

Voici un exemple d'accès à un snapshot de route :

```angular-ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component(/* ... */)
export class UserProfile {
  readonly userId: string | null;
  private route = inject(ActivatedRoute);

  constructor() {
    // Example URL: https://www.angular.dev/users/123?role=admin&status=active#contact

    // Access route parameters from snapshot
    this.userId = this.route.snapshot.paramMap.get('id');

    // Access multiple route elements
    const snapshot = this.route.snapshot;
    console.log({
      url: snapshot.url, // Array of UrlSegments matched by this route
      // Route parameters object: {id: '123'}
      params: snapshot.params,
      // Query parameters object: {role: 'admin', status: 'active'}
      queryParams: snapshot.queryParams, // Query parameters
    });
  }
}
```

Consultez la [documentation de l'API `ActivatedRoute`](/api/router/ActivatedRoute) et la [documentation de l'API `ActivatedRouteSnapshot`](/api/router/ActivatedRouteSnapshot) pour la liste complète des propriétés accessibles.

## Lire les paramètres d'une route {#reading-parameters-on-a-route}

Les développeurs peuvent exploiter deux types de paramètres d'une route : les paramètres de route et les query parameters.

### Paramètres de route {#route-parameters}

Les paramètres de route vous permettent de transmettre des données à un composant via l'URL. C'est utile quand vous voulez afficher un contenu précis en fonction d'un identifiant présent dans l'URL, comme un ID d'utilisateur ou de produit.

Vous pouvez [définir des paramètres de route](/guide/routing/define-routes#define-url-paths-with-route-parameters) en préfixant le nom du paramètre par deux-points (`:`).

```angular-ts
import {Routes} from '@angular/router';
import {Product} from './product';

const routes: Routes = [{path: 'product/:id', component: Product}];
```

Vous pouvez accéder aux paramètres en vous abonnant à `route.params`.

```angular-ts
import {Component, inject, signal} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-product-detail',
  template: `<h1>Product Details: {{ productId() }}</h1>`,
})
export class ProductDetail {
  productId = signal('');
  private activatedRoute = inject(ActivatedRoute);

  constructor() {
    // Access route parameters
    this.activatedRoute.params.subscribe((params) => {
      this.productId.set(params['id']);
    });
  }
}
```

### Query parameters {#query-parameters}

Les [query parameters](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) offrent un moyen souple de transmettre des données facultatives via les URL sans modifier la structure de la route. Contrairement aux paramètres de route, les query parameters peuvent persister d'une navigation à l'autre et sont parfaits pour gérer le filtrage, le tri, la pagination et d'autres éléments d'interface avec état.

```angular-ts
// Single parameter structure
// /products?category=electronics
router.navigate(['/products'], {
  queryParams: {category: 'electronics'},
});

// Multiple parameters
// /products?category=electronics&sort=price&page=1
router.navigate(['/products'], {
  queryParams: {
    category: 'electronics',
    sort: 'price',
    page: 1,
  },
});
```

Vous pouvez accéder aux query parameters avec `route.queryParams`.

Voici un exemple de `ProductList` qui met à jour les query parameters influant sur l'affichage d'une liste de produits :

```angular-ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute, Router} from '@angular/router';

@Component({
  selector: 'app-product-list',
  template: `
    <div>
      <select (change)="updateSort($event)">
        <option value="price">Price</option>
        <option value="name">Name</option>
      </select>
      <!-- Products list -->
    </div>
  `,
})
export class ProductList {
  private route = inject(ActivatedRoute);
  private router = inject(Router);

  constructor() {
    // Access query parameters reactively
    this.route.queryParams.subscribe((params) => {
      const sort = params['sort'] || 'price';
      const page = Number(params['page']) || 1;
      this.loadProducts(sort, page);
    });
  }

  updateSort(event: Event) {
    const sort = (event.target as HTMLSelectElement).value;
    // Update URL with new query parameter
    this.router.navigate([], {
      queryParams: {sort},
      queryParamsHandling: 'merge', // Preserve other query parameters
    });
  }
}
```

Dans cet exemple, les utilisateurs peuvent utiliser un élément select pour trier la liste de produits par nom ou par prix. L'event handler de changement associé met à jour les query parameters de l'URL, ce qui déclenche à son tour un événement de changement qui peut lire les query parameters mis à jour et actualiser la liste de produits.

Pour plus d'informations, consultez la [documentation officielle de QueryParamsHandling](/api/router/QueryParamsHandling).

### Paramètres matriciels {#matrix-parameters}

Les paramètres matriciels sont des paramètres facultatifs qui appartiennent à un segment d'URL précis, au lieu de s'appliquer à la route entière. Contrairement aux query parameters, qui apparaissent après un `?` et s'appliquent globalement, les paramètres matriciels utilisent des points-virgules (`;`) et sont limités à des segments de chemin individuels.

Les paramètres matriciels sont utiles quand vous devez transmettre des données auxiliaires à un segment de route précis sans modifier la définition de la route ni son comportement de correspondance. Comme les query parameters, ils n'ont pas besoin d'être définis dans votre configuration de routes.

```ts
// URL format: /path;key=value
// Multiple parameters: /path;key1=value1;key2=value2

// Navigate with matrix parameters
this.router.navigate(['/awesome-products', {view: 'grid', filter: 'new'}]);
// Results in URL: /awesome-products;view=grid;filter=new
```

**Avec ActivatedRoute**

```ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component(/* ... */)
export class AwesomeProducts {
  private route = inject(ActivatedRoute);

  constructor() {
    // Access matrix parameters via params
    this.route.params.subscribe((params) => {
      const view = params['view']; // e.g., 'grid'
      const filter = params['filter']; // e.g., 'new'
    });
  }
}
```

NOTE: Comme alternative à `ActivatedRoute`, les paramètres matriciels sont aussi liés aux inputs du composant lorsque vous utilisez `withComponentInputBinding`.

## Détecter la route active avec RouterLinkActive {#detect-active-current-route-with-routerlinkactive}

Vous pouvez utiliser la directive `RouterLinkActive` pour styliser dynamiquement les éléments de navigation en fonction de la route active. C'est courant dans les éléments de navigation pour indiquer aux utilisateurs quelle est la route active.

```angular-html
<nav>
  <a
    class="button"
    routerLink="/about"
    routerLinkActive="active-button"
    ariaCurrentWhenActive="page"
  >
    About
  </a>
  |
  <a
    class="button"
    routerLink="/settings"
    routerLinkActive="active-button"
    ariaCurrentWhenActive="page"
  >
    Settings
  </a>
</nav>
```

Dans cet exemple, le router d'Angular applique la classe `active-button` au bon lien d'ancre et `ariaCurrentWhenActive` à `page` quand l'URL correspond au `routerLink` associé.

Si vous devez ajouter plusieurs classes à l'élément, vous pouvez utiliser soit une chaîne séparée par des espaces, soit un tableau :

```angular-html
<!-- Space-separated string syntax -->
<a routerLink="/user/bob" routerLinkActive="class1 class2">Bob</a>

<!-- Array syntax -->
<a routerLink="/user/bob" [routerLinkActive]="['class1', 'class2']">Bob</a>
```

Définir `routerLinkActive` seul ne définit pas `aria-current`. Définissez l'input `ariaCurrentWhenActive`, comme dans l'exemple ci-dessus, afin que les utilisateurs malvoyants (qui peuvent ne pas percevoir le style différent appliqué) puissent aussi identifier le lien actif.

### Stratégie de correspondance des routes {#route-matching-strategy}

Par défaut, `RouterLinkActive` considère tout ancêtre dans la route comme une correspondance.

```angular-html
<a [routerLink]="['/user/jane']" routerLinkActive="active-link"> User </a>
<a [routerLink]="['/user/jane/role/admin']" routerLinkActive="active-link"> Role </a>
```

Quand l'utilisateur visite `/user/jane/role/admin`, les deux liens ont la classe `active-link`.

### N'appliquer RouterLinkActive que sur une correspondance exacte de la route {#only-apply-routerlinkactive-on-exact-route-matches}

Si vous voulez appliquer la classe uniquement en cas de correspondance exacte, vous devez fournir à la directive `routerLinkActiveOptions` un objet de configuration contenant la valeur `exact: true`.

```angular-html
<a
  [routerLink]="['/user/jane']"
  routerLinkActive="active-link"
  [routerLinkActiveOptions]="{exact: true}"
>
  User
</a>
<a
  [routerLink]="['/user/jane/role/admin']"
  routerLinkActive="active-link"
  [routerLinkActiveOptions]="{exact: true}"
>
  Role
</a>
```

Si vous voulez contrôler plus précisément la correspondance d'une route, notez que `exact: true` n'est en réalité qu'un raccourci syntaxique pour l'ensemble complet des options de correspondance :

```angular-ts
// `exact: true` is equivalent to
{
  paths: 'exact',
  fragment: 'ignored',
  matrixParams: 'ignored',
  queryParams: 'exact',
}

// `exact: false` is equivalent
{
  paths: 'subset',
  fragment: 'ignored',
  matrixParams: 'ignored',
  queryParams: 'subset',
}
```

Pour plus d'informations, consultez la documentation officielle de [isActiveMatchOptions](/api/router/IsActiveMatchOptions).

### Appliquer RouterLinkActive à un ancêtre {#apply-routerlinkactive-to-an-ancestor}

La directive RouterLinkActive peut aussi être appliquée à un élément ancêtre afin de permettre aux développeurs de styliser les éléments comme ils le souhaitent.

```angular-html
<div routerLinkActive="active-link" [routerLinkActiveOptions]="{exact: true}">
  <a routerLink="/user/jim">Jim</a>
  <a routerLink="/user/bob">Bob</a>
</div>
```

Pour plus d'informations, consultez la [documentation de l'API de RouterLinkActive](/api/router/RouterLinkActive).

## Vérifier si une URL est active {#check-if-a-url-is-active}

La fonction `isActive` renvoie un signal computed qui suit si une URL donnée est actuellement active dans le router. Le signal se met à jour automatiquement quand l'état du router change.

```angular-ts
import {Component, inject} from '@angular/core';
import {isActive, Router} from '@angular/router';

@Component({
  template: `
    <div [class.active]="isSettingsActive()">
      <h2>Settings</h2>
    </div>
  `,
})
export class Panel {
  private router = inject(Router);

  isSettingsActive = isActive('/settings', this.router, {
    paths: 'subset',
    queryParams: 'ignored',
    fragment: 'ignored',
    matrixParams: 'ignored',
  });
}
```
