<!-- source: 48057fe1e04f752c76b5fb64568b19f20117a99d -->
# Naviguer vers des routes

La directive RouterLink est l'approche déclarative d'Angular pour la navigation. Elle vous permet d'utiliser des éléments d'ancre standard (`<a>`) qui s'intègrent parfaitement au système de routing d'Angular.

## Comment utiliser RouterLink {#how-to-use-routerlink}

Au lieu d'utiliser des éléments d'ancre `<a>` classiques avec un attribut `href`, ajoutez une directive RouterLink avec le chemin approprié afin de tirer parti du routing d'Angular.

```angular-ts
import {RouterLink} from '@angular/router';
import {Component} from '@angular/core';

@Component({
  template: `
    <nav>
      <a routerLink="/user-profile">User profile</a>
      <a routerLink="/settings">Settings</a>
    </nav>
  `,
  imports: [RouterLink],
  ...
})
export class App {}
```

### Utiliser des liens absolus ou relatifs {#using-absolute-or-relative-links}

Dans le routing d'Angular, les **URL relatives** vous permettent de définir des chemins de navigation relatifs à l'emplacement de la route courante. Elles s'opposent aux **URL absolues**, qui contiennent le chemin complet avec le protocole (par exemple, `http://`) et le **domaine racine** (par exemple, `google.com`).

```angular-html
<!-- Absolute URL -->
<a href="https://www.angular.dev/essentials">Angular Essentials Guide</a>

<!-- Relative URL -->
<a href="/essentials">Angular Essentials Guide</a>
```

Dans cet exemple, le premier exemple contient le chemin complet avec le protocole (c'est-à-dire `https://`) et le domaine racine (c'est-à-dire `angular.dev`) explicitement définis pour la page des Fondamentaux. À l'inverse, le second exemple suppose que l'utilisateur se trouve déjà sur le bon domaine racine pour `/essentials`.

De manière générale, les URL relatives sont préférables, car elles sont plus faciles à maintenir d'une application à l'autre : elles n'ont pas besoin de connaître leur position absolue dans la hiérarchie de routing.

### Fonctionnement des URL relatives {#how-relative-urls-work}

Le routing d'Angular propose deux syntaxes pour définir des URL relatives : les chaînes de caractères et les tableaux.

```angular-html
<!-- Navigates user to /dashboard -->
<a routerLink="dashboard">Dashboard</a>
<a [routerLink]="['dashboard']">Dashboard</a>
```

HELPFUL: Passer une chaîne de caractères est la manière la plus courante de définir des URL relatives.

Lorsque vous devez définir des paramètres dynamiques dans une URL relative, utilisez la syntaxe en tableau :

```angular-html
<a [routerLink]="['user', currentUserId]">Current User</a>
```

De plus, le routing d'Angular vous permet d'indiquer si le chemin doit être relatif à l'URL courante ou au domaine racine, selon que le chemin relatif est préfixé ou non par une barre oblique (`/`).

Par exemple, si l'utilisateur se trouve sur `example.com/settings`, voici comment définir différents chemins relatifs selon les scénarios :

```angular-html
<!-- Navigates to /settings/notifications -->
<a routerLink="notifications">Notifications</a>
<a routerLink="/settings/notifications">Notifications</a>

<!-- Navigates to /team/:teamId/user/:userId -->
<a routerLink="/team/123/user/456">User 456</a>
<a [routerLink]="['/team', teamId, 'user', userId]">Current User</a>
```

## Navigation programmatique vers des routes {#programmatic-navigation-to-routes}

Alors que `RouterLink` gère la navigation déclarative dans les templates, Angular fournit une navigation programmatique pour les scénarios où vous devez naviguer en fonction de la logique, des actions de l'utilisateur ou de l'état de l'application. En injectant `Router`, vous pouvez naviguer dynamiquement vers des routes, passer des paramètres et contrôler le comportement de la navigation dans votre code TypeScript.

### `router.navigate()`

Vous pouvez utiliser la méthode `router.navigate()` pour naviguer de manière programmatique entre les routes en indiquant un tableau de chemin d'URL.

```angular-ts
import {Router} from '@angular/router';
import {Component, inject} from '@angular/core';

@Component({
  selector: 'app-dashboard',
  template: ` <button (click)="navigateToProfile()">View Profile</button> `,
})
export class AppDashboard {
  private router = inject(Router);

  navigateToProfile() {
    // Standard navigation
    this.router.navigate(['/profile']);

    // With route parameters
    this.router.navigate(['/users', userId]);

    // With query parameters
    this.router.navigate(['/search'], {
      queryParams: {category: 'books', sort: 'price'},
    });

    // With matrix parameters
    this.router.navigate(['/products', {featured: true, onSale: true}]);
  }
}
```

`router.navigate()` prend en charge les scénarios de routing simples comme complexes : vous pouvez passer des paramètres de route, des [query parameters](/guide/routing/read-route-state#query-parameters) et contrôler le comportement de la navigation.

Vous pouvez aussi construire des chemins de navigation dynamiques relatifs à l'emplacement de votre composant dans l'arbre de routes, à l'aide de l'option `relativeTo`.

```angular-ts
import {Router, ActivatedRoute} from '@angular/router';
import {Component, inject} from '@angular/core';

@Component({
  selector: 'app-user-detail',
  template: `
    <button (click)="navigateToEdit()">Edit User</button>
    <button (click)="navigateToParent()">Back to List</button>
  `,
})
export class UserDetail {
  private route = inject(ActivatedRoute);
  private router = inject(Router);

  // Navigate to a sibling route
  navigateToEdit() {
    // From: /users/123
    // To:   /users/123/edit
    this.router.navigate(['edit'], {relativeTo: this.route});
  }

  // Navigate to parent
  navigateToParent() {
    // From: /users/123
    // To:   /users
    this.router.navigate(['..'], {relativeTo: this.route});
  }

  navigateToList() {
    // Angular resolves the commands array as a single navigation path relative to the current route.
    // From: /users/123
    // Result: /users/list
    this.router.navigate(['..', 'list'], {relativeTo: this.route});
  }
}
```

Pour remonter de plusieurs niveaux, tous les segments `..` doivent se trouver dans le **premier élément** du tableau de commandes. Le router n'analyse `..` que dans la première chaîne de commande : les éléments suivants du tableau sont traités comme des segments de chemin littéraux.

```angular-ts {prefer}
// From: /team/123/users/456
// Result: /team/123/settings
this.router.navigate(['../../settings'], {relativeTo: this.route});
```

Lorsque vous utilisez `relativeTo`, ne préfixez jamais la première commande par `/`. Un `/` initial rend la navigation absolue et ignore complètement `relativeTo`.

```angular-ts {prefer}
// From: /team/123/users/456
// Result: /team/123/users/456/edit
this.router.navigate(['edit'], {relativeTo: this.route});
```

```angular-ts {avoid}
// From: /team/123/users/456
// Leading '/' causes absolute navigation — relativeTo is ignored
// Result: /edit
this.router.navigate(['/edit'], {relativeTo: this.route});
```

### `router.navigateByUrl()`

La méthode `router.navigateByUrl()` offre un moyen direct de naviguer de manière programmatique à l'aide de chaînes de chemin d'URL plutôt que de segments en tableau. Cette méthode est idéale lorsque vous disposez d'un chemin d'URL complet et devez effectuer une navigation absolue, en particulier avec des URL fournies de l'extérieur ou dans des scénarios de liens profonds (deep linking).

```ts
// Standard route navigation
router.navigateByUrl('/products');

// Navigate to nested route
router.navigateByUrl('/products/featured');

// Complete URL with parameters and fragment
router.navigateByUrl('/products/123?view=details#reviews');

// Navigate with query parameters
router.navigateByUrl('/search?category=books&sortBy=price');

// With matrix parameters
router.navigateByUrl('/sales-awesome;isOffer=true;showModal=false');
```

Si vous devez remplacer l'URL courante dans l'historique, `navigateByUrl` accepte aussi un objet de configuration doté d'une option `replaceUrl`.

```ts
// Replace current URL in history
router.navigateByUrl('/checkout', {
  replaceUrl: true,
});
```

### Afficher une URL différente dans la barre d'adresse {#display-a-different-url-in-the-address-bar}

Vous pouvez passer une option browserUrl à navigateByUrl pour afficher dans la barre d'adresse du navigateur une URL différente de celle utilisée pour la correspondance des routes.

C'est utile lorsque vous voulez rediriger un utilisateur vers une autre route, comme une page d'erreur, sans changer l'URL qu'il a initialement tenté de visiter.

```ts
router.navigateByUrl('/not-found', {browserUrl: '/products/missing-item'});
```

Angular navigue vers la route `/not-found` et l'affiche, mais la barre d'adresse du navigateur indique `/products/missing-item`.

NOTE: `browserUrl` n'affecte que ce qui apparaît dans la barre d'adresse du navigateur.

## Personnaliser l'URL du navigateur avec RouterLink {#customizing-the-browser-url-with-routerlink}

La directive `RouterLink` prend aussi en charge un input `browserUrl`, qui vous permet de contrôler l'URL affichée dans la barre d'adresse du navigateur lors d'un clic sur un lien, indépendamment de la route vers laquelle Angular navigue.

```angular-html
<!-- Navigates to /dashboard, but the address bar shows /home -->
<a [routerLink]="['/dashboard']" [browserUrl]="'/home'">Go to Dashboard</a>
```

Vous pouvez aussi lier un `UrlTree` pour des cas d'usage plus dynamiques :

```angular-ts
import {Component, inject} from '@angular/core';
import {Router, RouterLink, UrlTree} from '@angular/router';

@Component({
  template: `
    <a [routerLink]="['/products', product.id]" [browserUrl]="displayUrl">
      {{ product.name }}
    </a>
  `,
  imports: [RouterLink],
})
export class ProductList {
  private router = inject(Router);

  product = {id: 42, name: 'Widget'};

  // Create a UrlTree to display in the address bar
  displayUrl: UrlTree = this.router.createUrlTree(['/products', 'widget']);
}
```

## Prochaines étapes {#next-steps}

Découvrez comment [lire l'état de la route](/guide/routing/read-route-state) pour créer des composants réactifs et sensibles au contexte.
