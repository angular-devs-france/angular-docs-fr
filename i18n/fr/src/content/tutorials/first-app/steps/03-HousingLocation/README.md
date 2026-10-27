<!-- source: ac8d748b9f416ddb8efad2aec92d0fee28626e3c -->
# Créer le composant HousingLocation de l'application

Cette leçon du tutoriel montre comment ajouter le composant `HousingLocation` à votre application Angular.

<docs-video src="https://www.youtube.com/embed/R0nRX8jD2D0?si=U4ONEbPvtptdUHTt&amp;start=440"/>

## Ce que vous allez apprendre {#what-youll-learn}

- Votre application dispose d'un nouveau composant, `HousingLocation`, qui affiche un message confirmant que le composant a été ajouté à votre application.

<docs-workflow>

<docs-step title="Créer le `HousingLocation` {#create-the-housinglocation}">
Dans cette étape, vous créez un nouveau composant pour votre application.

Dans le panneau **Terminal** de votre IDE :

1. Dans le répertoire de votre projet, allez dans le répertoire `first-app`.

1. Exécutez cette commande pour créer un nouveau `HousingLocation`

   ```shell
   ng generate component housingLocation
   ```

1. Exécutez cette commande pour compiler et servir votre application.

   ```shell
   ng serve
   ```

   NOTE: Cette étape concerne uniquement votre environnement local !

1. Ouvrez un navigateur et allez sur `http://localhost:4200` pour trouver l'application.
1. Vérifiez que l'application se compile sans erreur.

   HELPFUL: Le rendu doit être identique à celui de la leçon précédente car, même si vous avez ajouté un nouveau composant, vous ne l'avez encore inclus dans aucun template de l'application.

1. Laissez `ng serve` tourner pendant que vous réalisez les étapes suivantes.
   </docs-step>

<docs-step title="Ajouter le nouveau composant à la mise en page de votre application {#add-the-new-component-to-your-apps-layout}">
Dans cette étape, vous ajoutez le nouveau composant, `HousingLocation`, au `Home` de votre application, afin qu'il s'affiche dans la mise en page de votre application.

Dans le panneau **Edit** de votre IDE :

1.  Ouvrez `home.ts` dans l'éditeur.
1.  Dans `home.ts`, importez `HousingLocation` en ajoutant cette ligne aux imports du fichier.

      <docs-code header="Importer HousingLocation dans src/app/home/home.ts" path="adev/src/content/tutorials/first-app/steps/04-interfaces/src/app/home/home.ts" visibleLines="[2]"/>

1.  Mettez ensuite à jour la propriété `imports` des métadonnées `@Component` en ajoutant `HousingLocation` au tableau.

      <docs-code  header="Ajouter HousingLocation au tableau imports dans src/app/home/home.ts" path="adev/src/content/tutorials/first-app/steps/04-interfaces/src/app/home/home.ts" visibleLines="[6]"/>

1.  Le composant est maintenant prêt à être utilisé dans le template de `Home`. Mettez à jour la propriété `template` des métadonnées `@Component` pour inclure une référence à la balise `<app-housing-location>`.

      <docs-code language="angular-ts" header="Ajouter housing location au template du composant dans src/app/home/home.ts" path="adev/src/content/tutorials/first-app/steps/04-interfaces/src/app/home/home.ts" visibleLines="[7,17]"/>

</docs-step>

<docs-step title="Ajouter les styles du composant {#add-the-styles-for-the-component}">
Dans cette étape, vous copiez dans votre application les styles déjà écrits pour `HousingLocation`, afin que l'application s'affiche correctement.

1. Ouvrez `src/app/housing-location/housing-location.css` et collez-y les styles ci-dessous :

   NOTE: Dans le navigateur, ils peuvent être placés dans `src/app/housing-location/housing-location.ts`, dans le tableau `styles`.

   <docs-code header="Ajouter les styles CSS du composant housing location dans src/app/housing-location/housing-location.css" path="adev/src/content/tutorials/first-app/steps/04-interfaces/src/app/housing-location/housing-location.css"/>

1. Enregistrez votre code, revenez au navigateur et vérifiez que l'application se compile sans erreur. Le message _housing-location works!_ doit s'afficher à l'écran. Corrigez les éventuelles erreurs avant de passer à l'étape suivante.

   <img alt="fenêtre de navigateur de homes-app affichant le logo, le champ de texte de filtre, le bouton de recherche et le message 'housing-location works!'" src="assets/images/tutorials/first-app/homes-app-lesson-03-step-2.png">

</docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez créé un nouveau composant pour votre application et l'avez ajouté à la mise en page de l'application.
