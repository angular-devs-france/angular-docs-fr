<!-- source: c70729cc03479aa96b2a1e475840ba769aaa9c5f -->
# Les services Angular

Cette leçon du tutoriel montre comment créer un service Angular et utiliser l'injection de dépendances pour l'inclure dans votre application.

<docs-video src="https://www.youtube.com/embed/-jRxG84AzCI?si=rieGfJawp9xJ00Sz"/>

## Ce que vous allez apprendre {#what-youll-learn}

Votre application dispose d'un service qui lui fournit ses données.
À la fin de cette leçon, le service lit des données locales et statiques.
Dans une leçon ultérieure, vous modifierez le service pour qu'il récupère les données depuis un service web.

## Aperçu conceptuel des services {#conceptual-preview-of-services}

Ce tutoriel présente les services Angular et l'injection de dépendances.

### Les services Angular {#angular-services}

Les _services Angular_ vous permettent de séparer les données et les fonctions de l'application Angular qui peuvent être utilisées par plusieurs composants de votre application.
Pour être utilisé par plusieurs composants, un service doit être rendu _injectable_.
Les services injectables utilisés par un composant deviennent des dépendances de ce composant.
Le composant dépend de ces services et ne peut pas fonctionner sans eux.

### L'injection de dépendances {#dependency-injection}

L'_injection de dépendances_ est le mécanisme qui gère les dépendances des composants d'une application et les services que d'autres composants peuvent utiliser.

<docs-workflow>

<docs-step title="Créer un nouveau service pour votre application {#create-a-new-service-for-your-app}">
Cette étape crée un service injectable pour votre application.

Dans le panneau **Terminal** de votre IDE :

1. Dans le répertoire de votre projet, placez-vous dans le répertoire `first-app`.
1. Dans le répertoire `first-app`, exécutez cette commande pour créer le nouveau service.

   ```shell
   ng generate service housing --skip-tests
   ```

1. Exécutez `ng serve` pour compiler l'application et la servir sur `http://localhost:4200`.
1. Vérifiez que l'application se compile sans erreur.
   Corrigez les éventuelles erreurs avant de passer à l'étape suivante.
   </docs-step>

<docs-step title="Ajouter des données statiques au nouveau service {#add-static-data-to-the-new-service}">
Cette étape ajoute quelques données d'exemple à votre nouveau service.
Dans une leçon ultérieure, vous remplacerez les données statiques par une interface web pour récupérer les données comme vous le feriez dans une vraie application.
Pour l'instant, le nouveau service de votre application utilise les données créées jusqu'ici localement dans `Home`.

Dans le panneau **Edit** de votre IDE :

1. Dans `src/app/home/home.ts`, copiez depuis `Home` la variable `baseUrl` et la variable `housingLocationList` avec son tableau de valeurs.
1. Dans `src/app/housing.service.ts` :
   1. Dans la classe `HousingService`, collez les variables que vous avez copiées depuis `Home` à l'étape précédente.
   1. Dans la classe `HousingService`, collez ces fonctions après les données que vous venez de copier.
      Ces fonctions permettent aux dépendances d'accéder aux données du service.

      <docs-code header="Fonctions du service dans src/app/housing.service.ts" path="adev/src/content/tutorials/first-app/steps/10-routing/src/app/housing.service.ts" visibleLines="[110,116]"/>

      Vous aurez besoin de ces fonctions dans une prochaine leçon. Pour l'instant, il suffit de comprendre que ces fonctions renvoient soit un `HousingLocation` précis selon son id, soit la liste complète.

   1. Ajoutez un import au niveau du fichier pour `HousingLocation`.

      <docs-code header="Importer le type HousingLocation dans  src/app/housing.service.ts" path="adev/src/content/tutorials/first-app/steps/10-routing/src/app/housing.service.ts" visibleLines="[2]"/>

1. Vérifiez que l'application se compile sans erreur.
   Corrigez les éventuelles erreurs avant de passer à l'étape suivante.
   </docs-step>

<docs-step title="Injecter le nouveau service dans `Home` {#inject-the-new-service-into-home}">
Cette étape injecte le nouveau service dans le `Home` de votre application pour qu'il puisse lire les données de l'application depuis un service.
Dans une leçon ultérieure, vous remplacerez les données statiques par une source de données réelle pour récupérer les données comme vous le feriez dans une vraie application.

Dans le panneau **Edit** de votre IDE, dans `src/app/home/home.ts` :

1.  En haut de `src/app/home/home.ts`, ajoutez `inject` aux éléments importés depuis `@angular/core`. Cela importera la fonction `inject` dans la classe `Home`.

      <docs-code language="angular-ts" header="Modification de src/app/home/home.ts" path="adev/src/content/tutorials/first-app/steps/10-routing/src/app/home/home.ts" visibleLines="[1]"/>

1.  Ajoutez un nouvel import au niveau du fichier pour `HousingService` :

      <docs-code language="angular-ts" header="Ajouter l'import dans src/app/home/home.ts" path="adev/src/content/tutorials/first-app/steps/10-routing/src/app/home/home.ts" visibleLines="[4]"/>

1.  Dans `Home`, supprimez la variable `baseUrl`, supprimez les entrées du tableau `housingLocationList` et affectez à `housingLocationList` un tableau vide (`[]`). Dans quelques étapes, vous modifierez le code pour récupérer les données depuis `HousingService`.

1.  Dans `Home`, ajoutez le code suivant pour injecter le nouveau service et initialiser les données de l'application. Le `constructor` est la première fonction exécutée à la création de ce composant. Le code du `constructor` affectera à `housingLocationList` la valeur renvoyée par l'appel à `getAllHousingLocations`.

      <docs-code language="angular-ts" header="Initialiser les données depuis le service dans src/app/home/home.ts" path="adev/src/content/tutorials/first-app/steps/10-routing/src/app/home/home.ts" visibleLines="[23,30]"/>

1.  Enregistrez les modifications de `src/app/home/home.ts` et vérifiez que votre application se compile sans erreur.
    Corrigez les éventuelles erreurs avant de passer à l'étape suivante.
    </docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez ajouté un service Angular à votre application et l'avez injecté dans la classe `Home`.
Cela isole la manière dont votre application obtient ses données.
Pour l'instant, le nouveau service obtient ses données à partir d'un tableau de données statique.
Dans une leçon ultérieure, vous refactoriserez le service pour qu'il obtienne ses données depuis un endpoint d'API.

Pour plus d'informations sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="guide/di/creating-and-using-services" title="Créer un service injectable"/>
  <docs-pill href="guide/di" title="L'injection de dépendances dans Angular"/>
  <docs-pill href="cli/generate/service" title="ng generate service"/>
  <docs-pill href="cli/generate" title="ng generate"/>
</docs-pill-row>
