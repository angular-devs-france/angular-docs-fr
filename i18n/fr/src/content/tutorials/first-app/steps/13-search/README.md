<!-- source: c2c4dc6cc21f718607222908b68042742200f22c -->
# Ajouter la fonctionnalité de recherche à votre application

Cette leçon du tutoriel montre comment ajouter une fonctionnalité de recherche à votre application Angular.

L'application permettra aux utilisateurs de rechercher parmi les données fournies par votre application et de n'afficher que les résultats correspondant au terme saisi.

<docs-video src="https://www.youtube.com/embed/5K10oYJ5Y-E?si=TiuNKx_teR9baO7k&amp;start=457"/>

IMPORTANT: Nous vous recommandons d'utiliser votre environnement local pour cette étape du tutoriel.

## Ce que vous allez apprendre {#what-youll-learn}

- Votre application utilisera les données d'un formulaire pour rechercher les logements correspondants
- Votre application n'affichera que les logements correspondants

<docs-workflow>

<docs-step title="Modifier les propriétés du composant home {#update-the-home-component-properties}">
Dans cette étape, vous allez modifier la classe `Home` pour stocker des données dans une nouvelle propriété de type tableau que vous utiliserez pour le filtrage.

1. Dans `src/app/home/home.ts`, ajoutez à la classe une nouvelle propriété nommée `filteredLocationList`.

   <docs-code header="Ajouter la propriété filteredLocationList dans home.ts" path="adev/src/content/tutorials/first-app/steps/14-http/src/app/home/home.ts" visibleLines="[28]"/>

   `filteredLocationList` contient les valeurs qui correspondent aux critères de recherche saisis par l'utilisateur.

1. Par défaut, au chargement de la page, `filteredLocationList` doit contenir l'ensemble des logements. Modifiez le `constructor` de `Home` pour définir cette valeur.

   <docs-code header="Définir la valeur de filteredLocationList" path="adev/src/content/tutorials/first-app/steps/14-http/src/app/home/home.ts" visibleLines="[29,32]"/>

</docs-step>

<docs-step title="Modifier le template du composant home {#update-the-home-component-template}">
`Home` contient déjà un champ de saisie que vous utiliserez pour capturer la saisie de l'utilisateur. Ce texte servira à filtrer les résultats.

1. Modifiez le template de `Home` pour inclure une variable de template nommée `#filter` dans l'élément `input`.

   <docs-code language="angular-ts" header="Ajouter une variable de template à l'élément HTML input dans home.ts" path="adev/src/content/tutorials/first-app/steps/14-http/src/app/home/home.ts" visibleLines="[12]"/>
   Cet exemple utilise une [variable de référence de template](guide/templates) pour accéder à l'élément `input` et à sa valeur.

1. Ensuite, modifiez le template du composant pour attacher un event handler au bouton « Search ».

   <docs-code language="angular-ts" header="Lier l'événement click du bouton à une méthode dans home.ts" path="adev/src/content/tutorials/first-app/steps/14-http/src/app/home/home.ts" visibleLines="[13]"/>

   En vous liant à l'événement `click` de l'élément `button`, vous pouvez appeler la fonction `filterResults`. L'argument de la fonction est la propriété `value` de la variable de template `filter`, c'est-à-dire la propriété `.value` de l'élément HTML `input`.

1. La dernière modification du template concerne la directive `@for`. Modifiez le `@for` pour qu'il itère sur les valeurs du tableau `filteredLocationList`.

   <docs-code language="angular-ts" header="Modifier la directive de template @for dans home.ts" path="adev/src/content/tutorials/first-app/steps/14-http/src/app/home/home.ts" visibleLines="[17,19]"/>

</docs-step>

<docs-step title="Implémenter la fonction de l'event handler {#implement-the-event-handler-function}">
Le template a été modifié pour lier la fonction `filterResults` à l'événement `click`. Votre tâche consiste maintenant à implémenter la fonction `filterResults` dans la classe `Home`.

1.  Modifiez la classe `Home` pour inclure l'implémentation de la fonction `filterResults`.

    <docs-code header="Ajouter l'implémentation de la fonction filterResults" path="adev/src/content/tutorials/first-app/steps/14-http/src/app/home/home.ts" visibleLines="[35,44]"/>

    Cette fonction utilise la fonction `filter` de `Array` pour comparer la valeur du paramètre `text` à la propriété `housingLocation.city`. À titre d'exercice ludique, vous pouvez modifier cette fonction pour qu'elle compare n'importe quelle propriété ou plusieurs propriétés.

1.  Enregistrez votre code.

1.  Actualisez le navigateur et vérifiez que vous pouvez rechercher les logements par ville lorsque vous cliquez sur le bouton « Search » après avoir saisi du texte.

       <img alt="résultats de recherche filtrés selon la saisie de l'utilisateur" src="assets/images/tutorials/first-app/homes-app-lesson-13-step-3.png">

    </docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez modifié votre application pour utiliser des variables de template afin d'interagir avec les valeurs du template, et ajouté une fonctionnalité de recherche à l'aide de l'event binding et de fonctions de tableau.

Pour plus d'informations sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="guide/templates" title="Variables de template"/>
  <docs-pill href="guide/templates/event-listeners" title="Gestion des événements"/>
</docs-pill-row>
