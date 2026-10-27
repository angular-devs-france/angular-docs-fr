<!-- source: fe1ed0e4d6da61a20edb179339ed0fc63dcb906d -->
# Ajouter un formulaire à votre application Angular

Cette leçon du tutoriel montre comment ajouter à une application Angular un formulaire qui collecte des données utilisateur.
Cette leçon part d'une application Angular fonctionnelle et montre comment y ajouter un formulaire.

Les données collectées par le formulaire sont envoyées uniquement au service de l'application, qui les écrit dans la console du navigateur.
L'utilisation d'une API REST pour envoyer et recevoir les données du formulaire n'est pas traitée dans cette leçon.

<docs-video src="https://www.youtube.com/embed/kWbk-dOJaNQ?si=FYMXGdUiT-qh321h"/>

IMPORTANT: Nous vous recommandons d'utiliser votre environnement local pour cette étape du tutoriel.

## Ce que vous allez apprendre {#what-youll-learn}

- Votre application dispose d'un formulaire dans lequel les utilisateurs peuvent saisir des données envoyées au service de votre application.
- Le service écrit les données du formulaire dans le journal de la console du navigateur.

<docs-workflow>

<docs-step title="Ajouter une méthode pour envoyer les données du formulaire {#add-a-method-to-send-form-data}">
Cette étape ajoute au service de votre application une méthode qui reçoit les données du formulaire à envoyer vers leur destination.
Dans cet exemple, la méthode écrit les données du formulaire dans le journal de la console du navigateur.

Dans le panneau **Edit** de votre IDE :

1.  Dans `src/app/housing.service.ts`, dans la classe `HousingService`, collez cette méthode à la fin de la définition de la classe.

       <docs-code header="Méthode de soumission dans src/app/housing.service.ts" path="adev/src/content/tutorials/first-app/steps/13-search/src/app/housing.service.ts" visibleLines="[118,122]"/>

1.  Vérifiez que l'application se compile sans erreur.
    Corrigez les éventuelles erreurs avant de passer à l'étape suivante.
    </docs-step>

<docs-step title="Ajouter les fonctions du formulaire à la page de détails {#add-the-form-functions-to-the-details-page}">
Cette étape ajoute à la page de détails le code qui gère les interactions avec le formulaire.

Dans le panneau **Edit** de votre IDE, dans `src/app/details/details.ts` :

1.  Après les instructions `import` en haut du fichier, ajoutez le code suivant pour importer les classes de formulaire d'Angular.

      <docs-code header="Imports de formulaires dans src/app/details/details.ts" path="adev/src/content/tutorials/first-app/steps/13-search/src/app/details/details.ts" visibleLines="[5]"/>

1.  Dans les métadonnées du décorateur de `Details`, modifiez la propriété `imports` avec le code suivant :

      <docs-code header="Directive imports dans src/app/details/details.ts" path="adev/src/content/tutorials/first-app/steps/13-search/src/app/details/details.ts" visibleLines="[9]"/>

1.  Dans la classe `Details`, avant la méthode `constructor()`, ajoutez le code suivant pour créer l'objet du formulaire.

      <docs-code header="Directive template dans src/app/details/details.ts" path="adev/src/content/tutorials/first-app/steps/13-search/src/app/details/details.ts" visibleLines="[52,56]"/>

    Dans Angular, `FormGroup` et `FormControl` sont des types qui vous permettent de construire des formulaires. Le type `FormControl` peut fournir une valeur par défaut et définir la forme des données du formulaire. Dans cet exemple, `firstName` est une `string` et sa valeur par défaut est une chaîne vide.

1.  Dans la classe `Details`, après la méthode `constructor()`, ajoutez le code suivant pour gérer le clic sur **Apply now**.

      <docs-code header="Directive template dans src/app/details/details.ts" path="adev/src/content/tutorials/first-app/steps/13-search/src/app/details/details.ts" visibleLines="[62,68]"/>

    Ce bouton n'existe pas encore : vous l'ajouterez à l'étape suivante. Dans le code ci-dessus, les `FormControl` peuvent renvoyer `null`. Ce code utilise l'opérateur de nullish coalescing pour utiliser une chaîne vide par défaut si la valeur est `null`.

1.  Vérifiez que l'application se compile sans erreur.
    Corrigez les éventuelles erreurs avant de passer à l'étape suivante.
    </docs-step>

<docs-step title="Ajouter le balisage du formulaire à la page de détails {#add-the-forms-markup-to-the-details-page}">
Cette étape ajoute à la page de détails le balisage qui affiche le formulaire.

Dans le panneau **Edit** de votre IDE, dans `src/app/details/details.ts` :

1. Dans les métadonnées du décorateur de `Details`, modifiez le HTML du `template` pour qu'il corresponde au code suivant afin d'ajouter le balisage du formulaire.

   <docs-code language="angular-ts" header="Directive template dans src/app/details/details.ts" path="adev/src/content/tutorials/first-app/steps/13-search/src/app/details/details.ts" visibleLines="[10,45]"/>

   Le template inclut maintenant un event handler `(submit)="submitApplication()"`. Angular utilise une syntaxe avec des parenthèses autour du nom de l'événement pour définir des événements dans le code du template. Le code à droite du signe égal est le code qui doit être exécuté lorsque cet événement est déclenché. Vous pouvez vous lier aux événements du navigateur et à des événements personnalisés.

1. Vérifiez que l'application se compile sans erreur.
   Corrigez les éventuelles erreurs avant de passer à l'étape suivante.

   <img alt="page de détails avec un formulaire de candidature pour habiter à cette adresse" src="assets/images/tutorials/first-app/homes-app-lesson-12-step-3.png">

</docs-step>

<docs-step title="Tester le nouveau formulaire de votre application {#test-your-apps-new-form}">
Cette étape teste le nouveau formulaire pour vérifier que, lorsque les données du formulaire sont soumises à l'application, elles apparaissent dans le journal de la console.

1. Dans le panneau **Terminal** de votre IDE, exécutez `ng serve` s'il n'est pas déjà en cours d'exécution.
1. Dans votre navigateur, ouvrez votre application à l'adresse `http://localhost:4200`.
1. Faites un clic droit sur l'application dans le navigateur et, dans le menu contextuel, choisissez **Inspect**.
1. Dans la fenêtre des outils de développement, choisissez l'onglet **Console**.
   Assurez-vous que la fenêtre des outils de développement reste visible pour les étapes suivantes
1. Dans votre application :
   1. Sélectionnez un logement et cliquez sur **Learn more** pour voir les détails du logement.
   1. Sur la page de détails du logement, faites défiler jusqu'en bas pour trouver le nouveau formulaire.
   1. Saisissez des données dans les champs du formulaire : n'importe quelles données conviennent.
   1. Choisissez **Apply now** pour soumettre les données.
1. Dans la fenêtre des outils de développement, consultez la sortie du journal pour retrouver les données de votre formulaire.
   </docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez modifié votre application pour ajouter un formulaire à l'aide de la fonctionnalité de formulaires d'Angular, et connecté les données saisies dans le formulaire à un composant à l'aide d'un event handler.

Pour plus d'informations sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="guide/forms" title="Les formulaires dans Angular"/>
  <docs-pill href="guide/templates/event-listeners" title="Gestion des événements"/>
</docs-pill-row>
