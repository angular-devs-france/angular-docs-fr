<!-- source: 86a3b711562312752f27cc04bb2a9e2cd06c4d30 -->
# Créer le composant Home

Cette leçon du tutoriel montre comment créer un nouveau [composant](guide/components) pour votre application Angular.

<docs-video src="https://www.youtube.com/embed/R0nRX8jD2D0?si=OMVaw71EIa44yIOJ"/>

## Ce que vous allez apprendre {#what-youll-learn}

Votre application dispose d'un nouveau composant : `Home`.

## Aperçu conceptuel des composants Angular {#conceptual-preview-of-angular-components}

Les applications Angular sont construites autour des composants, qui sont les briques de base d'Angular.
Les composants contiennent le code, la mise en page HTML et les informations de style CSS qui assurent le fonctionnement et l'apparence d'un élément de l'application.
Dans Angular, les composants peuvent contenir d'autres composants. Les fonctionnalités et l'apparence d'une application peuvent être réparties et découpées en composants.

Dans Angular, les composants ont des métadonnées qui définissent leurs propriétés.
Lorsque vous créez votre `Home`, vous utilisez ces propriétés :

- `selector` : pour décrire comment Angular fait référence au composant dans les templates.
- `imports` : pour décrire les dépendances du composant.
- `template` : pour décrire le balisage HTML et la mise en page du composant.
- `styleUrls` : pour lister dans un tableau les URL des fichiers CSS utilisés par le composant.

<docs-pill-row>
  <docs-pill href="api/core/Component" title="En savoir plus sur les composants"/>
</docs-pill-row>

<docs-workflow>

<docs-step title="Créer le `Home` {#create-the-home}">
Dans cette étape, vous créez un nouveau composant pour votre application.

Dans le panneau **Terminal** de votre IDE :

1. Dans le répertoire de votre projet, allez dans le répertoire `first-app`.
1. Exécutez cette commande pour créer un nouveau `Home`

   ```shell
   ng generate component home
   ```

1. Exécutez cette commande pour compiler et servir votre application.

   NOTE: Cette étape concerne uniquement votre environnement local !

   ```shell
   ng serve
   ```

1. Ouvrez un navigateur et allez sur `http://localhost:4200` pour trouver l'application.

1. Vérifiez que l'application se compile sans erreur.

   HELPFUL: Le rendu doit être identique à celui de la leçon précédente car, même si vous avez ajouté un nouveau composant, vous ne l'avez encore inclus dans aucun template de l'application.

1. Laissez `ng serve` tourner pendant que vous réalisez les étapes suivantes.
   </docs-step>

<docs-step title="Ajouter le nouveau composant à la mise en page de votre application {#add-the-new-component-to-your-apps-layout}">
Dans cette étape, vous ajoutez le nouveau composant, `Home`, au composant racine de votre application, `App`, afin qu'il s'affiche dans la mise en page de votre application.

Dans le panneau **Edit** de votre IDE :

1.  Ouvrez `app.ts` dans l'éditeur.
1.  Dans `app.ts`, importez `Home` en ajoutant cette ligne aux imports du fichier.

      <docs-code header="Importer Home dans src/app/app.ts" path="adev/src/content/tutorials/first-app/steps/03-HousingLocation/src/app/app.ts" visibleLines="[2]"/>

1.  Dans `app.ts`, dans `@Component`, mettez à jour la propriété tableau `imports` en y ajoutant `Home`.

      <docs-code header="Remplacer dans src/app/app.ts" path="adev/src/content/tutorials/first-app/steps/03-HousingLocation/src/app/app.ts" visibleLines="[6]"/>

1.  Dans `app.ts`, dans `@Component`, mettez à jour la propriété `template` pour inclure le code HTML suivant.

      <docs-code language="angular-ts" header="Remplacer dans src/app/app.ts" path="adev/src/content/tutorials/first-app/steps/03-HousingLocation/src/app/app.ts" visibleLines="[7,16]"/>

1.  Enregistrez vos modifications dans `app.ts`.
1.  Si `ng serve` tourne, l'application doit se mettre à jour.
    Si `ng serve` ne tourne pas, relancez-le.
    Le texte _Hello world_ de votre application doit être remplacé par _home works!_, provenant de `Home`.
1.  Regardez l'application dans le navigateur et vérifiez qu'elle a bien été mise à jour.

  <img alt="fenêtre de navigateur affichant une page avec le texte 'home works!'" src="assets/images/tutorials/first-app/homes-app-lesson-02-step-2.png">

</docs-step>

<docs-step title="Ajouter des fonctionnalités à `Home` {#add-features-to-home}">

Dans cette étape, vous ajoutez des fonctionnalités à `Home`.

À l'étape précédente, vous avez ajouté le `Home` par défaut au template de votre application, si bien que son HTML par défaut est apparu dans l'application.
Dans cette étape, vous ajoutez un filtre de recherche et un bouton qui serviront dans une leçon ultérieure.
Pour l'instant, c'est tout ce que contient `Home`.
Notez que cette étape ajoute seulement les éléments de recherche à la mise en page, sans aucune fonctionnalité pour le moment.

Si vous êtes parti d'un nouveau projet Angular au lieu de télécharger le projet de départ
(ng new) : ajoutez ces styles globaux à `src/styles.css` pour que le bouton de recherche et la bordure du champ de saisie soient visibles :

```css
:root {
  --primary-color: #605dc8;
  --secondary-color: #8b89e6;
  --accent-color: #e8e7fa;
  --shadow-color: #e8e8e8;
}

button.primary {
  padding: 10px;
  border: solid 1px var(--primary-color);
  background: var(--primary-color);
  color: white;
  border-radius: 8px;
}
```

Dans le panneau **Edit** de votre IDE :

1.  Dans le répertoire `first-app`, ouvrez `home.ts` dans l'éditeur.
1.  Dans `home.ts`, dans `@Component`, mettez à jour la propriété `template` avec ce code.

      <docs-code language="angular-ts" header="Remplacer dans src/app/home/home.ts" path="adev/src/content/tutorials/first-app/steps/03-HousingLocation/src/app/home/home.ts" visibleLines="[5,12]"/>

1.  Ouvrez ensuite `home.css` dans l'éditeur et remplacez son contenu par ces styles.

    NOTE: Dans le navigateur, ils peuvent être placés dans `src/app/home/home.ts`, dans le tableau `styles`.

       <docs-code header="Remplacer dans src/app/home/home.css" path="adev/src/content/tutorials/first-app/steps/03-HousingLocation/src/app/home/home.css"/>

1.  Vérifiez que l'application se compile sans erreur. Vous devez trouver le champ de filtre et le bouton dans votre application, avec leurs styles appliqués. Corrigez les éventuelles erreurs avant de passer à l'étape suivante.

   <img alt="fenêtre de navigateur de homes-app affichant le logo, le champ de texte de filtre et le bouton de recherche" src="assets/images/tutorials/first-app/homes-app-lesson-02-step-3.png">
</docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez créé un nouveau composant pour votre application et lui avez ajouté un champ de filtre et un bouton.

Pour en savoir plus sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="cli/generate/component" title="`ng generate component`"/>
  <docs-pill href="api/core/Component" title="Référence de `Component`"/>
  <docs-pill href="guide/components" title="Vue d'ensemble des composants Angular"/>
</docs-pill-row>
