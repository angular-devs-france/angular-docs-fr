<!-- source: 7e12c4e8027f0510b83cdf2499ba5d9c56b7aa94 -->
# Utiliser RouterLink pour la navigation

Dans l'état actuel de l'application, toute la page se recharge lorsque l'on clique sur un lien interne à l'application. Si cela peut sembler anodin dans une petite application, cela peut nuire aux performances de pages plus volumineuses, au contenu plus riche, pour lesquelles les utilisateurs doivent retélécharger les ressources et relancer les calculs.

NOTE: Pour en savoir plus, consultez [l'ajout de routes à votre application dans le guide approfondi](/guide/routing/define-routes#adding-the-router-to-your-application).

Dans cette activité, vous allez apprendre à tirer parti de la directive `RouterLink` pour exploiter au mieux le router d'Angular.

<hr>

<docs-workflow>

<docs-step title="Importer la directive `RouterLink` {#import-routerlink-directive}">

Dans `app.ts`, ajoutez l'import de la directive `RouterLink` à l'instruction d'import existante depuis `@angular/router`, et ajoutez-la au tableau `imports` du décorateur de votre composant.

```ts
...
import { RouterLink, RouterOutlet } from '@angular/router';

@Component({
  imports: [RouterLink, RouterOutlet],
  ...
})
```

</docs-step>

<docs-step title="Ajouter un `routerLink` au template {#add-a-routerlink-to-template}">

Pour utiliser la directive `RouterLink`, remplacez les attributs `href` par `routerLink`. Mettez à jour le template avec ce changement.

```angular-ts
import { RouterLink, RouterOutlet } from '@angular/router';

@Component({
  ...
  template: `
    ...
    <a routerLink="/">Home</a>
    <a routerLink="/user">User</a>
    ...
  `,
  imports: [RouterLink, RouterOutlet],
})
```

</docs-step>

</docs-workflow>

Désormais, lorsque vous cliquez sur les liens de navigation, vous ne devriez plus voir de clignotement : seul le contenu de la page elle-même (c'est-à-dire le `router-outlet`) change 🎉

Bravo, vous avez découvert le routing avec Angular. Ce n'est qu'un aperçu de l'API `Router` ; pour aller plus loin, consultez [Le routing dans Angular](guide/routing).
