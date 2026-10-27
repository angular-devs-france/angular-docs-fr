<!-- source: 10b1b09ddb2892d437606cbe9d6aaee05372461d -->
# Définir des routes

Les routes sont les briques de base de la navigation dans une application Angular.

## Que sont les routes ? {#what-are-routes}

Dans Angular, une **route** est un objet qui définit quel composant doit s'afficher pour un chemin ou un motif d'URL donné, ainsi que des options de configuration supplémentaires sur ce qui se passe quand un utilisateur navigue vers cette URL.

Voici un exemple simple de route :

```ts
import {AdminPage} from './app-admin';

const adminPage = {
  path: 'admin',
  component: AdminPage,
};
```

Avec cette route, quand un utilisateur visite le chemin `/admin`, l'application affiche le composant `AdminPage`.

### Gérer les routes dans votre application {#managing-routes-in-your-application}

La plupart des projets définissent les routes dans un fichier séparé dont le nom contient `routes`.

Une collection de routes ressemble à ceci :

```ts
import {Routes} from '@angular/router';
import {HomePage} from './home-page';
import {AdminPage} from './admin-page';

export const routes: Routes = [
  {
    path: '',
    component: HomePage,
  },
  {
    path: 'admin',
    component: AdminPage,
  },
];
```

TIP: Si vous avez généré un projet avec Angular CLI, vos routes sont définies dans `src/app/app.routes.ts`.

### Ajouter le router à votre application {#adding-the-router-to-your-application}

Lorsque vous effectuez le bootstrap d'une application Angular sans Angular CLI, vous pouvez passer un objet de configuration qui contient un tableau `providers`.

Dans le tableau `providers`, vous pouvez ajouter le router Angular à votre application en ajoutant un appel à la fonction `provideRouter` avec vos routes.

```ts
import {ApplicationConfig} from '@angular/core';
import {provideRouter} from '@angular/router';

import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    // ...
  ],
};
```

## Chemins d'URL des routes {#route-url-paths}

### Chemins d'URL statiques {#static-url-paths}

Les chemins d'URL statiques désignent des routes dont les chemins sont prédéfinis et ne changent pas en fonction de paramètres dynamiques. Ce sont des routes qui correspondent exactement à une chaîne `path` et dont le résultat est fixe.

En voici quelques exemples :

- "/admin"
- "/blog"
- "/settings/account"

### Définir des chemins d'URL avec des paramètres de route {#define-url-paths-with-route-parameters}

Les URL paramétrées vous permettent de définir des chemins dynamiques qui font correspondre plusieurs URL à un même composant, tout en affichant dynamiquement des données en fonction des paramètres de l'URL.

Vous pouvez définir ce type de motif en ajoutant des paramètres à la chaîne `path` de votre route et en préfixant chaque paramètre par le caractère deux-points (`:`).

IMPORTANT: Les paramètres sont distincts des informations de la [query string](https://en.wikipedia.org/wiki/Query_string) de l'URL.
Pour en savoir plus, consultez [les query parameters dans Angular dans ce guide](/guide/routing/read-route-state#query-parameters).

L'exemple suivant affiche un composant de profil utilisateur en fonction de l'identifiant d'utilisateur passé dans l'URL.

```ts
import {Routes} from '@angular/router';
import {UserProfile} from './user-profile/user-profile';

const routes: Routes = [{path: 'user/:id', component: UserProfile}];
```

Dans cet exemple, des URL comme `/user/leeroy` et `/user/jenkins` affichent le composant `UserProfile`. Ce composant peut ensuite lire le paramètre `id` et l'utiliser pour effectuer d'autres traitements, comme récupérer des données. Consultez le [guide sur la lecture de l'état de la route](/guide/routing/read-route-state) pour en savoir plus sur la lecture des paramètres de route.

Un nom de paramètre de route valide doit commencer par une lettre (a-z, A-Z) et ne peut contenir que :

- des lettres (a-z, A-Z)
- des chiffres (0-9)
- le tiret bas (\_)
- le trait d'union (-)

Vous pouvez aussi définir des chemins avec plusieurs paramètres :

```ts
import {Routes} from '@angular/router';
import {UserProfile} from './user-profile';
import {SocialMediaFeed} from './social-media-feed';

const routes: Routes = [
  {path: 'user/:id/:social-media', component: SocialMediaFeed},
  {path: 'user/:id', component: UserProfile},
];
```

Avec ce nouveau chemin, les utilisateurs peuvent visiter `/user/leeroy/youtube` et `/user/leeroy/bluesky` et voir les fils de réseaux sociaux correspondants, en fonction du paramètre, pour l'utilisateur leeroy.

Consultez [Lire l'état de la route](/guide/routing/read-route-state) pour en savoir plus sur la lecture des paramètres de route.

### Routes wildcard {#wildcards}

Quand vous devez intercepter toutes les routes d'un chemin donné, la solution est une route wildcard, définie avec le double astérisque (`**`).

Un exemple courant consiste à définir un composant « Page introuvable ».

```ts
import {Routes} from '@angular/router';
import {Home} from './home/home';
import {UserProfile} from './user-profile';
import {NotFound} from './not-found';

const routes: Routes = [
  {path: 'home', component: Home},
  {path: 'user/:id', component: UserProfile},
  {path: '**', component: NotFound},
];
```

Avec ce tableau de routes, l'application affiche le composant `NotFound` quand l'utilisateur visite un chemin autre que `home` et `user/:id`.

TIP: Les routes wildcard sont généralement placées à la fin d'un tableau de routes.

## Comment Angular fait correspondre les URL {#how-angular-matches-urls}

Quand vous définissez des routes, l'ordre est important, car Angular applique une stratégie où la première correspondance l'emporte. Autrement dit, dès qu'Angular fait correspondre une URL au `path` d'une route, il arrête d'examiner les routes suivantes. Placez donc toujours les routes les plus spécifiques avant les moins spécifiques.

L'exemple suivant montre des routes définies de la plus spécifique à la moins spécifique :

```ts
const routes: Routes = [
  {path: '', component: Home}, // Empty path
  {path: 'users/new', component: NewUser}, // Static, most specific
  {path: 'users/:id', component: UserDetail}, // Dynamic
  {path: 'users', component: Users}, // Static, less specific
  {path: '**', component: NotFound}, // Wildcard - always last
];
```

Si un utilisateur visite `/users/new`, le router Angular passe par les étapes suivantes :

1. Il vérifie `''` : pas de correspondance
1. Il vérifie `users/new` : correspondance ! Il s'arrête là
1. Il n'atteint jamais `users/:id`, même si cette route pourrait correspondre
1. Il n'atteint jamais `users`
1. Il n'atteint jamais `**`

## Redirections {#redirects}

Vous pouvez définir une route qui redirige vers une autre route au lieu d'afficher un composant :

```ts
import {Routes} from '@angular/router';
import {Blog} from './home/blog';

const routes: Routes = [
  {
    path: 'articles',
    redirectTo: '/blog',
  },
  {
    path: 'blog',
    component: Blog,
  },
];
```

Si vous modifiez ou supprimez une route, certains utilisateurs peuvent encore cliquer sur des liens ou des favoris obsolètes pointant vers cette route. Vous pouvez ajouter une redirection pour diriger ces utilisateurs vers une route alternative appropriée plutôt que vers une page « introuvable ».

## Titres de page {#page-titles}

Vous pouvez associer un **titre** à chaque route. Angular met automatiquement à jour le [titre de la page](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/title) quand une route s'active. Définissez toujours des titres de page appropriés pour votre application, car ils sont nécessaires pour offrir une expérience accessible.

```ts
import {Routes} from '@angular/router';
import {Home} from './home';
import {About} from './about';
import {Products} from './products';

const routes: Routes = [
  {
    path: '',
    component: Home,
    title: 'Home Page',
  },
  {
    path: 'about',
    component: About,
    title: 'About Us',
  },
];
```

La propriété `title` de la page peut être définie dynamiquement avec une fonction resolver à l'aide de [`ResolveFn`](/api/router/ResolveFn).

```ts
const titleResolver: ResolveFn<string> = (route) => route.queryParams['id'];
const routes: Routes = [
  // ...
  {
    path: 'products',
    component: Products,
    title: titleResolver,
  },
];
```

Les titres de route peuvent aussi être définis via un service qui étend la classe abstraite [`TitleStrategy`](/api/router/TitleStrategy). Par défaut, Angular utilise la [`DefaultTitleStrategy`](/api/router/DefaultTitleStrategy).

### Utiliser TitleStrategy pour les titres de page {#using-titlestrategy-for-page-titles}

Pour les scénarios avancés où vous avez besoin de contrôler de façon centralisée la composition du titre du document, implémentez une `TitleStrategy`.

`TitleStrategy` est un jeton que vous pouvez fournir pour remplacer la stratégie de titre utilisée par défaut par Angular. Vous pouvez fournir une `TitleStrategy` personnalisée pour mettre en œuvre des conventions comme l'ajout d'un suffixe propre à l'application, la mise en forme des titres à partir d'un fil d'Ariane ou la génération dynamique des titres à partir des données de la route.

```ts
import {inject, Injectable} from '@angular/core';
import {Title} from '@angular/platform-browser';
import {TitleStrategy, RouterStateSnapshot} from '@angular/router';

@Injectable()
export class AppTitleStrategy extends TitleStrategy {
  private readonly title = inject(Title);

  updateTitle(snapshot: RouterStateSnapshot): void {
    // PageTitle is equal to the "Title" of a route if it's set
    // If its not set it will use the "title" given in index.html
    const pageTitle = this.buildTitle(snapshot) || this.title.getTitle();
    this.title.setTitle(`MyAwesomeApp - ${pageTitle}`);
  }
}
```

Pour utiliser la stratégie personnalisée, fournissez-la avec le jeton `TitleStrategy` au niveau de l'application :

```ts
import {provideRouter, TitleStrategy} from '@angular/router';
import {AppTitleStrategy} from './app-title.strategy';

export const appConfig = {
  providers: [provideRouter(routes), {provide: TitleStrategy, useClass: AppTitleStrategy}],
};
```

## Providers au niveau de la route pour l'injection de dépendances {#route-level-providers-for-dependency-injection}

Chaque route possède une propriété `providers` qui vous permet de fournir des dépendances au contenu de cette route via l'[injection de dépendances](/guide/di).

C'est utile par exemple dans les applications qui proposent des services différents selon que l'utilisateur est administrateur ou non.

```ts
export const ROUTES: Route[] = [
  {
    path: 'admin',
    providers: [AdminService, {provide: ADMIN_API_KEY, useValue: '12345'}],
    children: [
      {path: 'users', component: AdminUsers},
      {path: 'teams', component: AdminTeams},
    ],
  },
  // ... other application routes that don't
  //     have access to ADMIN_API_KEY or AdminService.
];
```

Dans cet exemple de code, le chemin `admin` contient une propriété de données protégée, `ADMIN_API_KEY`, qui n'est accessible qu'aux enfants de sa section. Par conséquent, aucun autre chemin ne peut accéder aux données fournies via `ADMIN_API_KEY`.

Consultez le [guide de l'injection de dépendances](/guide/di) pour en savoir plus sur les providers et l'injection dans Angular.

## Associer des données aux routes {#associating-data-with-routes}

Les données de route vous permettent d'attacher des informations supplémentaires aux routes. Vous pouvez configurer le comportement des composants en fonction de ces données.

Il existe deux façons de travailler avec les données de route : les données statiques, qui restent constantes, et les données dynamiques, qui peuvent changer selon les conditions d'exécution.

### Données statiques {#static-data}

Vous pouvez associer des données statiques arbitraires à une route via la propriété `data`, afin de centraliser par exemple des métadonnées propres à la route (suivi analytics, permissions, etc.) :

```ts
import {Routes} from '@angular/router';
import {Home} from './home';
import {About} from './about';
import {Products} from './products';

const routes: Routes = [
  {
    path: 'about',
    component: About,
    data: {analyticsId: '456'},
  },
  {
    path: '',
    component: Home,
    data: {analyticsId: '123'},
  },
];
```

Dans cet exemple de code, la page d'accueil et la page « À propos » sont configurées avec un `analyticsId` spécifique, qui serait ensuite utilisé dans leurs composants respectifs pour le suivi analytics des pages.

Vous pouvez lire ces données statiques en injectant l'`ActivatedRoute`. Consultez [Lire l'état de la route](/guide/routing/read-route-state) pour en savoir plus.

### Données dynamiques avec les resources et les resolvers {#dynamic-data-with-resources-and-resolvers}

Quand vous devez récupérer des données pour une route, le router Angular prend en charge les resources de route réactives ainsi que les resolvers de données :

- [Récupérer des données avec les resources](/guide/routing/data-fetching-with-resources) : récupérez des données de façon réactive avec les API `Resource` des signals d'Angular.
- [Resolvers de données de route](/guide/routing/data-resolvers) : récupérez des données avant l'activation de la route à l'aide de fonctions resolver.

## Routes imbriquées {#nested-routes}

Les routes imbriquées, aussi appelées routes enfants, sont une technique courante pour gérer des routes de navigation plus complexes, où un composant possède une sous-vue qui change en fonction de l'URL.

<img alt="Diagramme illustrant les routes imbriquées" src="assets/images/guide/router/nested-routing-diagram.svg">

Vous pouvez ajouter des routes enfants à n'importe quelle définition de route avec la propriété `children` :

```ts
const routes: Routes = [
  {
    path: 'product/:id',
    component: Product,
    children: [
      {
        path: 'info',
        component: ProductInfo,
      },
      {
        path: 'reviews',
        component: ProductReviews,
      },
    ],
  },
];
```

L'exemple ci-dessus définit une route pour une page produit qui permet à l'utilisateur d'afficher soit les informations du produit, soit les avis, en fonction de l'URL.

La propriété `children` accepte un tableau d'objets `Route`.

Pour afficher les routes enfants, le composant parent (`Product` dans l'exemple ci-dessus) contient son propre `<router-outlet>`.

```angular-html
<!-- Product -->
<article>
  <h1>Product {{ id }}</h1>
  <router-outlet />
</article>
```

Après avoir ajouté des routes enfants à la configuration et un `<router-outlet>` au composant, la navigation entre les URL qui correspondent aux routes enfants met à jour uniquement l'outlet imbriqué.

## Étapes suivantes {#next-steps}

<docs-pill-row>
  <docs-pill href="/guide/routing/loading-strategies" title="Stratégies de chargement des routes"/>
  <docs-pill href="/guide/routing/show-routes-with-outlets" title="Afficher le contenu de vos routes avec les outlets"/>
</docs-pill-row>
