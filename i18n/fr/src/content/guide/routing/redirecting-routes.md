<!-- source: 797b4a5596c84224731815cb0518f74ea7f30c85 -->
# Rediriger des routes

Les redirections de routes vous permettent de faire naviguer automatiquement les utilisateurs d'une route vers une autre. C'est comparable à la réexpédition du courrier, où le courrier destiné à une adresse est envoyé à une autre adresse. C'est utile pour gérer d'anciennes URL, mettre en place des routes par défaut ou gérer le contrôle d'accès.

## Configurer des redirections {#how-to-configure-redirects}

Vous pouvez définir des redirections dans votre configuration de routes avec la propriété `redirectTo`. Cette propriété accepte une chaîne de caractères.

```ts
import {Routes} from '@angular/router';

const routes: Routes = [
  // Simple redirect
  {path: 'marketing', redirectTo: 'newsletter'},

  // Redirect with path parameters
  {path: 'legacy-user/:id', redirectTo: 'users/:id'},

  // Redirect any other URLs that don’t match
  // (also known as a "wildcard" redirect)
  {path: '**', redirectTo: '/login'},
];
```

Cet exemple contient trois redirections :

1. Quand un utilisateur visite le chemin `/marketing`, il est redirigé vers `/newsletter`.
2. Quand un utilisateur visite n'importe quel chemin `/legacy-user/:id`, il est dirigé vers le chemin `/users/:id` correspondant.
3. Quand un utilisateur visite un chemin qui n'est pas défini dans le router, il est redirigé vers la page de connexion grâce à la définition du chemin wildcard `**`.

## Comprendre `pathMatch` {#understanding-pathmatch}

La propriété `pathMatch` des routes permet aux développeurs de contrôler la façon dont Angular associe une URL aux routes.

`pathMatch` accepte deux valeurs :

| Valeur     | Description                                      |
| ---------- | -------------------------------------------- |
| `'full'`   | Le chemin complet de l'URL doit correspondre exactement |
| `'prefix'` | Seul le début de l'URL doit correspondre |

Par défaut, toutes les redirections utilisent la stratégie `prefix`.

### `pathMatch: 'prefix'` {#pathmatch-prefix}

`pathMatch: 'prefix'` est la stratégie par défaut, idéale quand vous voulez que le router d'Angular fasse correspondre toutes les routes qui suivent lors du déclenchement d'une redirection.

```ts
export const routes: Routes = [
  // This redirect route is equivalent to…
  {path: 'news', redirectTo: 'blog'},

  // This explicitly defined route redirect pathMatch
  {path: 'news', redirectTo: 'blog', pathMatch: 'prefix'},
];
```

Dans cet exemple, toutes les routes préfixées par `news` sont redirigées vers leurs équivalents `/blog`. Voici quelques exemples de redirections lorsque les utilisateurs visitent l'ancien préfixe `news` :

- `/news` redirige vers `/blog`
- `/news/article` redirige vers `/blog/article`
- `/news/article/:id` redirige vers `/blog/article/:id`

### `pathMatch: 'full'` {#pathmatch-full}

À l'inverse, `pathMatch: 'full'` est utile quand vous voulez que le router d'Angular ne redirige qu'un chemin précis.

```ts
export const routes: Routes = [{path: '', redirectTo: '/dashboard', pathMatch: 'full'}];
```

Dans cet exemple, chaque fois que l'utilisateur visite l'URL racine (c.-à-d. `''`), le router le redirige vers la page `'/dashboard'`.

Les pages suivantes (par ex. `/login`, `/about`, `/product/id`, etc.) sont ignorées et ne déclenchent pas de redirection.

TIP: Soyez prudent lorsque vous configurez une redirection sur la page racine (c.-à-d. `"/"` ou `""`). Si vous ne définissez pas `pathMatch: 'full'`, le router redirigera toutes les URL.

Pour mieux l'illustrer, si l'exemple `news` de la section précédente utilisait plutôt `pathMatch: 'full'` :

```ts
export const routes: Routes = [{path: 'news', redirectTo: '/blog', pathMatch: 'full'}];
```

Cela signifie que :

1. Seul le chemin `/news` sera redirigé vers `/blog`.
2. Les segments suivants, comme `/news/articles` ou `/news/articles/1`, ne seraient pas redirigés avec le nouveau préfixe `/blog`.

## Redirections conditionnelles {#conditional-redirects}

La propriété `redirectTo` peut aussi accepter une fonction afin d'ajouter de la logique à la façon dont les utilisateurs sont redirigés.

La [fonction](api/router/RedirectFunction) n'a accès qu'à une partie des données de [`ActivatedRouteSnapshot`](api/router/ActivatedRouteSnapshot), car certaines données ne sont pas connues avec précision pendant la phase de correspondance des routes. Par exemple : les titres résolus, les composants chargés en lazy loading, etc.

Elle renvoie généralement une chaîne de caractères ou un [`URLTree`](api/router/UrlTree), mais elle peut aussi renvoyer un observable ou une promise.

Voici un exemple où l'utilisateur est redirigé vers un menu différent selon l'heure de la journée :

```ts
import {Routes} from '@angular/router';
import {Menu} from './menu';

export const routes: Routes = [
  {
    path: 'restaurant/:location/menu',
    pathMatch: 'full',
    redirectTo: (activatedRouteSnapshot) => {
      const location = activatedRouteSnapshot.params['location'];
      const currentHour = new Date().getHours();

      // Check if user requested a specific meal via query parameter
      if (activatedRouteSnapshot.queryParams['meal']) {
        return `/restaurant/${location}/menu/${activatedRouteSnapshot.queryParams['meal']}`;
      }

      // Auto-redirect based on time of day
      if (currentHour >= 5 && currentHour < 11) {
        return `/restaurant/${location}/menu/breakfast`;
      } else if (currentHour >= 11 && currentHour < 17) {
        return `/restaurant/${location}/menu/lunch`;
      } else {
        return `/restaurant/${location}/menu/dinner`;
      }
    },
  },

  // Destination routes
  {path: 'restaurant/:location/menu/breakfast', component: Menu},
  {path: 'restaurant/:location/menu/lunch', component: Menu},
  {path: 'restaurant/:location/menu/dinner', component: Menu},

  // Default redirect
  {path: '', redirectTo: '/restaurant/downtown/menu', pathMatch: 'full'},
];
```

Pour en savoir plus, consultez [la documentation de l'API de RedirectFunction](api/router/RedirectFunction).

## Étapes suivantes {#next-steps}

Pour plus d'informations sur la propriété `redirectTo`, consultez la [documentation de l'API](api/router/Route#redirectTo).
