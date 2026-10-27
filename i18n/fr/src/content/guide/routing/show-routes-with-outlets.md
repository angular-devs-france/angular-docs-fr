<!-- source: b0e18aa25c15068a2bfb42bb2b033f3bd6134075 -->
# Afficher les routes avec des outlets

La directive `RouterOutlet` est un placeholder qui marque l'emplacement où le router doit afficher le composant correspondant à l'URL courante.

```html
<app-header />
<!-- Angular inserts your route content here -->
<router-outlet />
<app-footer />
```

```ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class App {}
```

Dans cet exemple, si une application définit les routes suivantes :

```ts
import {Routes} from '@angular/router';
import {Home} from './home';
import {Products} from './products';

const routes: Routes = [
  {
    path: '',
    component: Home,
    title: 'Home Page',
  },
  {
    path: 'products',
    component: Products,
    title: 'Our Products',
  },
];
```

Quand un utilisateur visite `/products`, Angular affiche ceci :

```angular-html
<app-header />
<app-products />
<app-footer />
```

Si l'utilisateur revient à la page d'accueil, Angular affiche alors :

```angular-html
<app-header />
<app-home />
<app-footer />
```

Lors de l'affichage d'une route, l'élément `<router-outlet>` reste présent dans le DOM comme point de référence pour les navigations futures. Angular insère le contenu routé juste après l'élément outlet, en tant qu'élément frère.

```angular-html
<!-- Contents of the component's template -->
<app-header />
<router-outlet />
<app-footer />
```

```angular-html
<!-- Content rendered on the page when the user visits /admin -->
<app-header />
<router-outlet />
<app-admin-page />
<app-footer />
```

## Imbriquer des routes avec des routes enfants {#nesting-routes-with-child-routes}

À mesure que votre application gagne en complexité, vous voudrez peut-être créer des routes relatives à un autre composant que votre composant racine. Cela vous permet de créer des expériences où seule une partie de l'application change quand l'URL change, au lieu de donner aux utilisateurs l'impression que la page entière est rechargée.

Ces routes imbriquées sont appelées routes enfants. Cela signifie que vous ajoutez un second `<router-outlet>` à votre application, en plus du `<router-outlet>` de AppComponent.

Dans cet exemple, le composant `Settings` affiche le panneau voulu en fonction de ce que l'utilisateur sélectionne. Une particularité des routes enfants est que le composant possède souvent ses propres `<nav>` et `<router-outlet>`.

```angular-html
<h1>Settings</h1>
<nav>
  <ul>
    <li><a routerLink="profile">Profile</a></li>
    <li><a routerLink="security">Security</a></li>
  </ul>
</nav>
<router-outlet />
```

Une route enfant est une route comme les autres : elle a besoin à la fois d'un `path` et d'un `component`. La seule différence est que vous placez les routes enfants dans un tableau children au sein de la route parente.

```ts
const routes: Routes = [
  {
    path: 'settings',
    component: Settings, // this is the component with the <router-outlet> in the template
    children: [
      {
        path: 'profile', // child route path
        component: Profile, // child route component that the router renders
      },
      {
        path: 'security',
        component: Security, // another child route component that the router renders
      },
    ],
  },
];
```

Une fois les `routes` et le `<router-outlet>` correctement configurés, votre application utilise des routes imbriquées !

## Routes secondaires avec des outlets nommés {#secondary-routes-with-named-outlets}

Une page peut avoir plusieurs outlets : vous pouvez attribuer un nom à chaque outlet pour indiquer quel contenu appartient à quel outlet.

```angular-html
<app-header />
<router-outlet />
<router-outlet name="read-more" />
<router-outlet name="additional-actions" />
<app-footer />
```

Chaque outlet doit avoir un nom unique. Le nom ne peut pas être défini ni modifié dynamiquement. Par défaut, le nom est `'primary'`.

Angular fait correspondre le nom de l'outlet à la propriété `outlet` définie sur chaque route :

```ts
{
  path: 'user/:id',
  component: UserDetails,
  outlet: 'additional-actions'
}
```

## Événements de cycle de vie de l'outlet {#outlet-lifecycle-events}

Un router outlet peut émettre quatre événements de cycle de vie :

| Événement    | Description                                                              |
| ------------ | ------------------------------------------------------------------------ |
| `activate`   | Quand un nouveau composant est instancié                                     |
| `deactivate` | Quand un composant est détruit                                            |
| `attach`     | Quand la `RouteReuseStrategy` demande à l'outlet d'attacher le sous-arbre |
| `detach`     | Quand la `RouteReuseStrategy` demande à l'outlet de détacher le sous-arbre |

Vous pouvez ajouter des event listeners avec la syntaxe standard d'event binding :

```angular-html
<router-outlet
  (activate)="onActivate($event)"
  (deactivate)="onDeactivate($event)"
  (attach)="onAttach($event)"
  (detach)="onDetach($event)"
/>
```

Consultez la [documentation d'API de RouterOutlet](/api/router/RouterOutlet?tab=api) pour en savoir plus.

## Transmettre des données contextuelles aux composants routés {#passing-contextual-data-to-routed-components}

Transmettre des données contextuelles aux composants routés nécessite souvent un état global ou des configurations de route complexes. Pour simplifier cela, chaque `RouterOutlet` accepte un input `routerOutletData`. Les composants routés et leurs enfants peuvent lire ces données sous forme de signal grâce au jeton d'injection `ROUTER_OUTLET_DATA`, ce qui permet une configuration propre à chaque outlet sans modifier les définitions de routes.

```angular-ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-dashboard',
  imports: [RouterOutlet],
  template: `
    <h2>Dashboard</h2>
    <router-outlet [routerOutletData]="{layout: 'sidebar'}" />
  `,
})
export class Dashboard {}
```

Le composant routé peut injecter les données fournies par l'outlet avec `ROUTER_OUTLET_DATA` :

```angular-ts
import {Component, inject, Signal} from '@angular/core';
import {ROUTER_OUTLET_DATA} from '@angular/router';

@Component({
  selector: 'app-stats',
  template: `<p>Stats view (layout: {{ outletData().layout }})</p>`,
})
export class Stats {
  outletData = inject(ROUTER_OUTLET_DATA) as Signal<{layout: string}>;
}
```

Quand Angular active `Stats` dans cet outlet, ce composant reçoit `{ layout: 'sidebar' }` comme données injectées.

NOTE: Quand l'input `routerOutletData` n'est pas défini, la valeur du signal injecté est `undefined` par défaut.

---

## Prochaines étapes {#next-steps}

Découvrez comment [naviguer vers des routes](/guide/routing/navigate-to-routes) avec le router Angular.
