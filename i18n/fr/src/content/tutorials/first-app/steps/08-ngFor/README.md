<!-- source: d998bbe9392bc8095f06fd3de275e97d301d1326 -->
# Utiliser @for pour lister des objets dans un composant

Cette leçon du tutoriel montre comment utiliser le bloc `@for` dans les templates Angular afin d'afficher des données répétées dynamiquement dans un template.

<docs-video src="https://www.youtube.com/embed/eM3zi_n7lNs?si=MIl5NcRxvcLjYt5f&amp;start=477"/>

NOTE: Cette vidéo présente une ancienne syntaxe, mais les concepts principaux restent valables.

## Ce que vous allez apprendre {#what-youll-learn}

- Vous aurez ajouté un jeu de données à l'application
- Votre application affichera une liste d'éléments du nouveau jeu de données à l'aide de `@for`

## Aperçu conceptuel de `@for` {#conceptual-preview-of-for}

Dans Angular, `@for` est un type particulier de [bloc de control flow](/guide/templates/control-flow) utilisé pour répéter dynamiquement des données dans un template. En JavaScript pur, vous utiliseriez une boucle for : `@for` offre une fonctionnalité similaire pour les templates Angular.

Vous pouvez utiliser `@for` pour itérer sur des tableaux et même sur des valeurs asynchrones. Dans cette leçon, vous allez ajouter un nouveau tableau de données sur lequel itérer.

Pour une explication plus approfondie, consultez le guide [Control flow](guide/templates/control-flow#repeat-content-with-the-for-block).

<docs-workflow>

<docs-step title="Ajouter des données de logements au `Home` {#add-housing-data-to-the-home}">

Le `Home` ne contient qu'un seul logement. Dans cette étape, vous allez ajouter un tableau d'entrées `HousingLocation`.

1. Dans `src/app/home/home.ts`, supprimez la propriété `housingLocation` de la classe `Home`.
1. Modifiez la classe `Home` pour qu'elle ait une propriété nommée `housingLocationList`. Modifiez votre code pour qu'il corresponde au code suivant :
   <docs-code language="angular-ts"  header="Ajouter la propriété housingLocationList dans home.ts" path="adev/src/content/tutorials/first-app/steps/09-services/src/app/home/home.ts" visibleLines="[26,127]"/>

   IMPORTANT: Ne supprimez pas le décorateur `@Component`, vous modifierez ce code dans une prochaine étape.

</docs-step>

<docs-step title="Modifier le template de `Home` pour utiliser `@for` {#update-the-home-template-to-use-for}">
L'application dispose maintenant d'un jeu de données que vous pouvez utiliser pour afficher les entrées dans le navigateur à l'aide du bloc `@for`.

1. Remplacez la balise `<app-housing-location>` dans le code du template par ceci :
   <docs-code language="angular-ts"  header="Ajouter @for au template de Home dans home.ts" path="adev/src/content/tutorials/first-app/steps/09-services/src/app/home/home.ts" visibleLines="[15,19]"/>

   Notez que dans le code `[housingLocation] = "housingLocation"`, la valeur `housingLocation` fait désormais référence à la variable utilisée dans le bloc `@for`. Avant cette modification, elle faisait référence à la propriété de la classe `Home`.

1. Enregistrez toutes les modifications.

1. Actualisez le navigateur et vérifiez que l'application affiche maintenant une grille de logements.

<section class="lightbox">
<img alt="fenêtre du navigateur affichant l'application homes-app avec le logo, un champ de saisie de filtre, un bouton de recherche et une grille de cartes de logements" src="assets/images/tutorials/first-app/homes-app-lesson-08-step-2.png">
</section>

</docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez utilisé le bloc `@for` pour répéter dynamiquement des données dans les templates Angular. Vous avez aussi ajouté un nouveau tableau de données à utiliser dans l'application Angular. L'application affiche maintenant dynamiquement une liste de logements dans le navigateur.

L'application prend forme, beau travail.

Pour plus d'informations sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="guide/templates/control-flow" title="Blocs de control flow"/>
  <docs-pill href="guide/templates/control-flow#repeat-content-with-the-for-block" title="Guide de @for"/>
  <docs-pill href="/api/core/@for" title="@for"/>
</docs-pill-row>
