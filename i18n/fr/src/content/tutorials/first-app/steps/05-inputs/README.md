<!-- source: a3371d3a9e679dd6f036129a9e4be4788f575f3c -->
# Ajouter un paramètre d'input au composant

Cette leçon du tutoriel montre comment créer un `input` de composant et l'utiliser pour transmettre des données à un composant afin de le personnaliser.

<docs-video src="https://www.youtube.com/embed/eM3zi_n7lNs?si=WvRGFSkW_7_zDIFD&amp;start=241"/>

NOTE: Cette vidéo présente une ancienne syntaxe, mais les concepts principaux restent valables.

## Ce que vous allez apprendre {#what-youll-learn}

Le template `HousingLocation` de votre application dispose d'une propriété `HousingLocation` pour recevoir des données en entrée.

## Aperçu conceptuel des inputs {#conceptual-preview-of-inputs}

Les [inputs](api/core/input) permettent aux composants de spécifier des données qui peuvent leur être transmises par un composant parent.

Dans cette leçon, vous allez définir une propriété `input` dans le composant `HousingLocation`, qui vous permet de personnaliser les données affichées dans le composant.

Pour en savoir plus, consultez les guides [Accepter des données avec les propriétés d'input](guide/components/inputs) et [Événements personnalisés avec les outputs](guide/components/outputs).

<docs-workflow>

<docs-step title="Importer la fonction input() {#import-the-input-function}">
Dans l'éditeur de code, importez la fonction utilitaire `input` depuis `@angular/core` et l'interface `HousingLocationInfo` depuis `../housinglocation` dans le composant `HousingLocation`.

<docs-code header="Importer input et HousingLocationInfo dans housing-location.ts" path="adev/src/content/tutorials/first-app/steps/06-property-binding/src/app/housing-location/housing-location.ts" visibleLines="[1,2]"/>

</docs-step>

<docs-step title="Ajouter la propriété d'input {#add-the-input-property}">
Ajoutez une propriété obligatoire nommée `housingLocation` et initialisez-la avec `input.required()`, avec le type `HousingLocationInfo`.

  <docs-code header="Déclarer la propriété d'input dans housing-location.ts" path="adev/src/content/tutorials/first-app/steps/06-property-binding/src/app/housing-location/housing-location.ts" visibleLines="[10]"/>

Vous devez appeler la méthode `required` sur `input` pour indiquer que le composant parent doit fournir une valeur. Dans notre application d'exemple, nous savons que cette valeur sera toujours transmise : c'est voulu. L'appel à `.required()` garantit que le compilateur TypeScript l'impose et traite la propriété comme non nullable lorsque ce composant est utilisé dans un template.

</docs-step>

<docs-step title="Transmettre des données à l'input {#pass-data-to-the-input}">
Envoyez la valeur `housingLocation` du composant `Home` vers la propriété `housingLocation` du composant HousingLocation.

<docs-code language="angular-ts" header="Déclarer la propriété d'input pour HousingLocation dans home.ts" path="adev/src/content/tutorials/first-app/steps/06-property-binding/src/app/home/home.ts" visibleLines="[16]"/>

</docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez créé une nouvelle propriété `input`. Vous avez aussi utilisé la méthode `.required` pour garantir que la valeur du signal est toujours définie.

<docs-pill-row>
  <docs-pill href="guide/components/inputs" title="Accepter des données avec les propriétés d'input"/>
  <docs-pill href="guide/components/outputs" title="Événements personnalisés avec les outputs"/>
</docs-pill-row>
