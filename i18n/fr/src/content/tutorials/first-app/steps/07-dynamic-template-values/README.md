<!-- source: e09c85148816fc67f0b24bf697c0a400f61aa827 -->
# Ajouter une interpolation au template d'un composant

Cette leçon du tutoriel montre comment ajouter une interpolation aux templates Angular afin d'afficher des données dynamiques dans un template.

<docs-video src="https://www.youtube.com/embed/eM3zi_n7lNs?si=IFAly3Ss8dwqFx8N&amp;start=338"/>

## Ce que vous allez apprendre {#what-youll-learn}

- Votre application affichera des valeurs interpolées dans le template de `HousingLocation`.
- Votre application affichera les données d'un logement dans le navigateur.

## Aperçu conceptuel de l'interpolation {#conceptual-preview-of-interpolation}

Dans cette étape, vous allez afficher dans un template, à l'aide de l'interpolation, des valeurs lues depuis des propriétés `input`.

Avec `{{ expression }}` dans les templates Angular, vous pouvez afficher les valeurs de propriétés, d'`inputs` et d'expressions JavaScript valides.

Pour une explication plus approfondie, consultez le guide [Afficher du texte dynamique avec l'interpolation de texte](guide/templates/binding#render-dynamic-text-with-text-interpolation).

<docs-workflow>

<docs-step title="Mettre à jour le template de `HousingLocation` pour inclure des valeurs interpolées {#update-housinglocation-template-to-include-interpolated-values}">
Cette étape ajoute une nouvelle structure HTML et des valeurs interpolées dans le template de `HousingLocation`.

Dans l'éditeur de code :

1.  Allez dans `src/app/housing-location/housing-location.ts`
1.  Dans la propriété template du décorateur `@Component`, remplacez le balisage HTML existant par le code suivant :

    <docs-code language="angular-ts"  header="Mettre à jour le template de HousingLocation dans housing-location.ts" path="adev/src/content/tutorials/first-app/steps/08-ngFor/src/app/housing-location/housing-location.ts" visibleLines="[6,17]"/>

Dans ce template mis à jour, vous avez utilisé un property binding pour lier `housingLocation.photo` à l'attribut `src`. L'attribut `alt` utilise l'interpolation pour donner plus de contexte au texte alternatif de l'image.

Vous utilisez l'interpolation pour inclure les valeurs `name`, `city` et `state` de la propriété `housingLocation`.

</docs-step>

<docs-step title="Vérifier que les modifications s'affichent dans le navigateur {#confirm-the-changes-render-in-the-browser}">
1.  Enregistrez toutes les modifications.
1.  Ouvrez le navigateur et vérifiez que l'application affiche les données d'exemple : photo, ville et état.
    <img alt="fenêtre de navigateur de homes-app affichant le logo, le champ de texte de filtre, le bouton de recherche et la carte d'interface du logement" src="assets/images/tutorials/first-app/homes-app-lesson-07-step-2.png">
</docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez ajouté une nouvelle structure HTML et utilisé la syntaxe de template d'Angular pour afficher des valeurs dans le template de `HousingLocation`.

Vous disposez désormais de deux compétences importantes :

- transmettre des données aux composants
- interpoler des valeurs dans un template

Grâce à ces compétences, votre application peut désormais partager des données et afficher des valeurs dynamiques dans le navigateur. Beau travail jusqu'ici.

Pour en savoir plus sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="guide/templates" title="Syntaxe de template"/>
  <docs-pill href="guide/templates/binding#render-dynamic-text-with-text-interpolation" title="Afficher du texte dynamique avec l'interpolation de texte"/>
</docs-pill-row>
