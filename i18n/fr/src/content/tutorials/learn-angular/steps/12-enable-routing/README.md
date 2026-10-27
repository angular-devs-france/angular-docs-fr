<!-- source: 8052c51c5d79179f23422fa44927a6d42445f283 -->
# Présentation du routing

Dans la plupart des applications, il arrive un moment où l'application a besoin de plus d'une seule page. Quand ce moment arrive inévitablement, le routing devient un élément important des performances perçues par les utilisateurs.

NOTE: Pour en savoir plus, consultez [le routing dans le guide approfondi](/guide/routing).

Dans cette activité, vous allez apprendre à installer et configurer votre application pour utiliser Angular Router.

<hr>

<docs-workflow>

<docs-step title="Créer un fichier app.routes.ts {#create-an-approutests-file}">

Dans `app.routes.ts`, apportez les modifications suivantes :

1. Importez `Routes` depuis le package `@angular/router`.
2. Exportez une constante nommée `routes` de type `Routes`, et affectez-lui la valeur `[]`.

```ts
import {Routes} from '@angular/router';

export const routes: Routes = [];
```

</docs-step>

<docs-step title="Ajouter le routing au provider {#add-routing-to-provider}">

Dans `app.config.ts`, configurez l'application pour Angular Router en suivant ces étapes :

1. Importez la fonction `provideRouter` depuis `@angular/router`.
1. Importez `routes` depuis `./app.routes.ts`.
1. Appelez la fonction `provideRouter` en lui passant `routes` en argument, dans le tableau `providers`.

```ts {highlight:[2,3,6]}
import {ApplicationConfig} from '@angular/core';
import {provideRouter} from '@angular/router';
import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};
```

</docs-step>

<docs-step title="Importer `RouterOutlet` dans le composant {#import-routeroutlet-in-the-component}">

Enfin, pour que votre application soit prête à utiliser Angular Router, vous devez lui indiquer où le router doit afficher le contenu souhaité. Pour cela, utilisez la directive `RouterOutlet` de `@angular/router`.

Mettez à jour le template de `App` en ajoutant `<router-outlet />`

```angular-ts {highlight:[9]}
@Component({
...
template: `
    <nav>
      <a href="/">Home</a>
      |
      <a href="/user">User</a>
    </nav>
    <router-outlet />
  `,
imports: [RouterOutlet],
})
export class App {}
```

</docs-step>

</docs-workflow>

Votre application est maintenant configurée pour utiliser Angular Router. Beau travail ! 🙌

Gardez votre élan pour découvrir l'étape suivante : définir les routes de notre application.
