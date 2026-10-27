<!-- source: 2b31a1328164b9b09e768b7fad771b99dda24896 -->
# Créer des correspondances de route personnalisées

Le router d'Angular prend en charge une stratégie de correspondance puissante que vous pouvez utiliser pour aider les utilisateurs à naviguer dans votre application.
Cette stratégie de correspondance prend en charge les routes statiques, les routes variables avec paramètres, les routes wildcard, etc.
Vous pouvez aussi construire votre propre correspondance de motifs personnalisée pour les situations où les URL sont plus complexes.

Dans ce tutoriel, vous allez construire un route matcher personnalisé à l'aide de `UrlMatcher` d'Angular.
Ce matcher recherche un identifiant X (anciennement Twitter) dans l'URL.

## Objectifs {#objectives}

Implémenter `UrlMatcher` d'Angular pour créer un route matcher personnalisé.

## Créer une application d'exemple {#create-a-sample-application}

À l'aide de l'Angular CLI, créez une nouvelle application, _angular-custom-route-match_.
En plus du squelette d'application Angular par défaut, vous allez aussi créer un composant _profile_.

1. Créez un nouveau projet Angular, _angular-custom-route-match_.

   ```shell
   ng new angular-custom-route-match
   ```

   Lorsque la question `Would you like to add Angular routing?` s'affiche, sélectionnez `Y`.

   Lorsque la question `Which stylesheet format would you like to use?` s'affiche, sélectionnez `CSS`.

   Au bout de quelques instants, un nouveau projet, `angular-custom-route-match`, est prêt.

1. Depuis votre terminal, placez-vous dans le répertoire `angular-custom-route-match`.
1. Créez un composant, _profile_.

   ```shell
   ng generate component profile
   ```

1. Dans votre éditeur de code, ouvrez le fichier `profile.html` et remplacez le contenu placeholder par le HTML suivant.

   <docs-code header="profile.html" path="adev/src/content/examples/routing-with-urlmatcher/src/app/profile/profile.html"/>

1. Dans votre éditeur de code, ouvrez le fichier `app.html` et remplacez le contenu placeholder par le HTML suivant.

   <docs-code header="app.html" path="adev/src/content/examples/routing-with-urlmatcher/src/app/app.html"/>

## Configurer les routes de votre application {#configure-your-routes-for-your-application}

Une fois le squelette de votre application en place, vous devez ensuite ajouter les fonctionnalités de routing au fichier `app.config.ts`.
Dans le cadre de ce processus, vous allez créer un URL matcher personnalisé qui recherche un identifiant X dans l'URL.
Cet identifiant se reconnaît au symbole `@` qui le précède.

1. Dans votre éditeur de code, ouvrez votre fichier `app.config.ts`.
1. Ajoutez une instruction `import` pour `provideRouter` et `withComponentInputBinding` d'Angular, ainsi que pour les routes de l'application.

   ```ts
   import {provideRouter, withComponentInputBinding} from '@angular/router';

   import {routes} from './app.routes';
   ```

1. Dans le tableau des providers, ajoutez une instruction `provideRouter(routes, withComponentInputBinding())`.

1. Définissez le route matcher personnalisé en ajoutant le code suivant aux routes de l'application.

   <docs-code header="app.routes.ts" path="adev/src/content/examples/routing-with-urlmatcher/src/app/app.routes.ts" region="matcher"/>

Ce matcher personnalisé est une fonction qui effectue les tâches suivantes :

- Le matcher vérifie que le tableau ne contient qu'un seul segment
- Le matcher utilise une expression régulière pour vérifier que le format du nom d'utilisateur correspond
- En cas de correspondance, la fonction renvoie l'URL entière, en définissant un paramètre de route `username` comme sous-chaîne du chemin
- En l'absence de correspondance, la fonction renvoie null et le router continue de chercher d'autres routes qui correspondent à l'URL

HELPFUL: Un URL matcher personnalisé se comporte comme n'importe quelle autre définition de route. Définissez des routes enfants ou des routes chargées en lazy loading comme vous le feriez pour n'importe quelle autre route.

## Lire les paramètres de route {#reading-the-route-parameters}

Une fois le matcher personnalisé en place, vous pouvez maintenant lier le paramètre de route dans le composant `profile`.

Dans votre éditeur de code, ouvrez votre fichier `profile.ts` et créez un `input` correspondant au paramètre `username`.
Nous avons ajouté la fonctionnalité `withComponentInputBinding` plus tôt
dans `provideRouter`. Elle permet au `Router` de lier des informations directement aux composants des routes.

```ts
username = input.required<string>();
```

## Tester votre URL matcher personnalisé {#test-your-custom-url-matcher}

Une fois votre code en place, vous pouvez maintenant tester votre URL matcher personnalisé.

1. Dans une fenêtre de terminal, exécutez la commande `ng serve`.

   ```shell
   ng serve
   ```

1. Ouvrez un navigateur à l'adresse `http://localhost:4200`.

   Vous devriez voir une page web unique, composée d'une phrase qui indique `Navigate to my profile`.

1. Cliquez sur le lien **my profile**.

   Une nouvelle phrase, `Hello, Angular!`, apparaît sur la page.

## Prochaines étapes {#next-steps}

La correspondance de motifs avec le router d'Angular vous offre une grande flexibilité lorsque votre application comporte des URL dynamiques.
Pour en savoir plus sur le router d'Angular, consultez les sujets suivants :

<docs-pill-row>
  <docs-pill href="guide/routing/common-router-tasks" title="Routing et navigation dans l'application"/>
  <docs-pill href="api/router/Router" title="API du Router"/>
</docs-pill-row>

HELPFUL: Ce contenu s'inspire de [Custom Route Matching with the Angular Router](https://medium.com/@brandontroberts/custom-route-matching-with-the-angular-router-fbdd48665483), par [Brandon Roberts](https://x.com/brandontroberts).
