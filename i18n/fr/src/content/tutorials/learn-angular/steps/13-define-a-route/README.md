<!-- source: 768283ec84489281bdbc0dd400294861b1b3958f -->
# Définir une route

Maintenant que vous avez configuré l'application pour utiliser le router d'Angular, vous devez définir les routes.

NOTE: Pour en savoir plus, consultez [la définition d'une route simple dans le guide approfondi](/guide/routing/define-routes).

Dans cette activité, vous allez apprendre à ajouter et à configurer des routes dans votre application.

<hr>

<docs-workflow>

<docs-step title="Définir une route dans `app.routes.ts` {#define-a-route-in-approutests}">

Votre application comporte deux pages à afficher : (1) la page d'accueil (Home) et (2) la page utilisateur (User).

Pour définir une route, ajoutez au tableau `routes` de `app.routes.ts` un objet route qui contient :

- Le `path` de la route (qui part automatiquement du chemin racine, c'est-à-dire `/`)
- Le `component` que la route doit afficher

```ts
import {Routes} from '@angular/router';
import {Home} from './home/home';

export const routes: Routes = [
  {
    path: '',
    component: Home,
  },
];
```

Le code ci-dessus montre comment ajouter `Home` en tant que route. Implémentez maintenant cette route, ainsi que celle de `User`, dans le Playground.

Utilisez `'user'` comme chemin de `User`.

</docs-step>

<docs-step title="Ajouter un titre à la définition de route {#add-title-to-route-definition}">

En plus de vous permettre de définir les routes, le router d'Angular vous permet aussi de définir le titre de la page à chaque navigation de l'utilisateur, en ajoutant la propriété `title` à chaque route.

Dans `app.routes.ts`, ajoutez la propriété `title` à la route par défaut (`path: ''`) et à la route `user`. Voici un exemple :

```ts {highlight:[7]}
import {Routes} from '@angular/router';
import {Home} from './home/home';

export const routes: Routes = [
  {
    path: '',
    title: 'App Home Page',
    component: Home,
  },
];
```

</docs-step>

</docs-workflow>

Dans cette activité, vous avez appris à définir et à configurer des routes dans votre application Angular. Beau travail. 🙌

Vous avez presque terminé de mettre en place le routing dans votre application, continuez.
