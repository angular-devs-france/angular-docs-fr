<!-- source: 0893e571b74f59f9818defac99ded0f9051ce43a -->
# Intégrer la page de détails dans l'application

Cette leçon du tutoriel montre comment connecter la page de détails à votre application.

<docs-video src="https://www.youtube.com/embed/-jRxG84AzCI?si=CbqIpmRpwp5ZZDnu&amp;start=345"/>

IMPORTANT: Nous vous recommandons d'utiliser votre environnement local pour apprendre le routing.

## Ce que vous allez apprendre {#what-youll-learn}

À la fin de cette leçon, votre application prendra en charge le routing vers la page de détails.

## Aperçu conceptuel du routing avec des paramètres de route {#conceptual-preview-of-routing-with-route-parameters}

Chaque logement possède des détails spécifiques qui doivent être affichés lorsqu'un utilisateur navigue vers la page de détails de cet élément. Pour y parvenir, vous devrez utiliser des paramètres de route.

Les paramètres de route vous permettent d'inclure des informations dynamiques dans l'URL de votre route. Pour identifier le logement sur lequel un utilisateur a cliqué, vous utiliserez la propriété `id` du type `HousingLocation`.

<docs-workflow>

<docs-step title="Utiliser `routerLink` pour la navigation dynamique {#using-routerlink-for-dynamic-navigation}">
Dans la leçon 10, vous avez ajouté une seconde route dans `src/app/routes.ts`, qui inclut un segment particulier identifiant le paramètre de route, `id` :

```
'details/:id'
```

Ici, `:id` est dynamique et changera selon la manière dont la route est demandée par le code.

1.  Dans `src/app/housing-location/housing-location.ts`, ajoutez une balise d'ancre à l'élément `section` et incluez la directive `routerLink` :

    <docs-code language="angular-ts" header="Ajouter une ancre avec une directive routerLink dans housing-location.ts" path="adev/src/content/tutorials/first-app/steps/12-forms/src/app/housing-location/housing-location.ts" visibleLines="[18]"/>

    La directive `routerLink` permet au router d'Angular de créer des liens dynamiques dans l'application. La valeur affectée à `routerLink` est un tableau de deux entrées : la partie statique du chemin et les données dynamiques.

    Pour que `routerLink` fonctionne dans le template, ajoutez un import au niveau du fichier de `RouterLink` depuis '@angular/router', puis modifiez le tableau `imports` du composant pour y inclure `RouterLink`.

1.  À ce stade, vous pouvez vérifier que le routing fonctionne dans votre application. Dans le navigateur, actualisez la page d'accueil et cliquez sur le bouton « Learn More » d'un logement.

      <img alt="page de détails affichant le texte 'details works!'" src="assets/images/tutorials/first-app/homes-app-lesson-11-step-1.png">

</docs-step>

<docs-step title="Récupérer les paramètres de route {#get-route-parameters}">
Dans cette étape, vous allez récupérer le paramètre de route dans `Details`. Actuellement, l'application affiche `details works!`. Vous allez ensuite modifier le code pour afficher la valeur `id` transmise via les paramètres de route.

1.  Dans `src/app/details/details.ts`, modifiez le template pour importer les fonctions, classes et services dont vous aurez besoin dans `Details` :

      <docs-code header="Modifier les imports au niveau du fichier" path="adev/src/content/tutorials/first-app/steps/12-forms/src/app/details/details.ts" visibleLines="[1,4]"/>

1.  Modifiez la propriété `template` du décorateur `@Component` pour afficher la valeur `housingLocationId` :

         ```angular-ts
         template: `<p>details works! {{ housingLocationId }}</p>`,
         ```

1.  Modifiez le corps de la classe `Details` avec le code suivant :

         ```ts
         export class Details {
            route: ActivatedRoute = inject(ActivatedRoute);
            housingLocationId = -1;
            constructor() {
            this.housingLocationId = Number(this.route.snapshot.params['id']);
            }
         }
         ```

    Ce code donne à `Details` accès à la fonctionnalité `ActivatedRoute` du router, qui vous permet d'accéder aux données de la route courante. Dans le `constructor`, le code convertit le paramètre `id` récupéré depuis la route d'une chaîne en nombre.

1.  Enregistrez toutes les modifications.

1.  Dans le navigateur, cliquez sur l'un des liens « Learn More » d'un logement et vérifiez que la valeur numérique affichée sur la page correspond à la propriété `id` de ce logement dans les données.
    </docs-step>

<docs-step title="Personnaliser `Details` {#customize-the-details}">
Maintenant que le routing fonctionne correctement dans l'application, c'est le bon moment pour modifier le template de `Details` afin d'afficher les données spécifiques du logement correspondant au paramètre de route.

Pour accéder aux données, vous allez ajouter un appel à `HousingService`.

1. Modifiez le code du template pour qu'il corresponde au code suivant :

   <docs-code language="angular-ts" header="Modifier le template de Details dans src/app/details/details.ts" path="adev/src/content/tutorials/first-app/steps/12-forms/src/app/details/details.ts" visibleLines="[8,29]"/>

   Remarquez que les propriétés de `housingLocation` sont accédées avec l'opérateur de chaînage optionnel `?`. Cela garantit que l'application ne plante pas si la valeur de `housingLocation` est null ou undefined.

1. Modifiez le corps de la classe `Details` pour qu'il corresponde au code suivant :

   <docs-code language="angular-ts" header="Modifier la classe Details dans src/app/details/details.ts" path="adev/src/content/tutorials/first-app/steps/12-forms/src/app/details/details.ts" visibleLines="[32,41]"/>

   Le composant dispose maintenant du code pour afficher les bonnes informations selon le logement sélectionné. Le `constructor` inclut désormais un appel à `HousingService` pour passer le paramètre de route en argument à la fonction `getHousingLocationById` du service.

1. Copiez les styles suivants dans le fichier `src/app/details/details.css` :

   <docs-code header="Ajouter les styles de Details" path="adev/src/content/tutorials/first-app/steps/12-forms/src/app/details/details.css" visibleLines="[1,72]"/>

   et enregistrez vos modifications

1. Dans `Details`, utilisez le fichier `details.css` que vous venez de créer comme source des styles :
   <docs-code language="angular-ts" header="Modifier details.ts pour utiliser le fichier css créé" path="adev/src/content/tutorials/first-app/steps/12-forms/src/app/details/details.ts" visibleLines="[30]"/>

1. Dans le navigateur, actualisez la page et vérifiez que, lorsque vous cliquez sur le lien « Learn More » d'un logement donné, la page de détails affiche les bonnes informations selon les données de l'élément sélectionné.

<img alt="Page de détails présentant les informations du logement" src="assets/images/tutorials/first-app/homes-app-lesson-11-step-3.png">

</docs-step>

<docs-step title="Vérifier la navigation dans `Home` {#check-navigation-in-the-home}">
Dans une leçon précédente, vous avez modifié le template de `App` pour y inclure un `routerLink`. Ce code a permis à votre application de revenir à `Home` chaque fois que l'on clique sur le logo.

1.  Vérifiez que votre code correspond à ce qui suit :

      <docs-code language="angular-ts" header="Vérifier le routerLink dans app.ts" path="adev/src/content/tutorials/first-app/steps/12-forms/src/app/app.ts" visibleLines="[8,19]"/>

    Votre code devrait déjà être à jour, mais vérifiez-le pour en être sûr.

    </docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez ajouté du routing pour afficher les pages de détails.

Vous savez maintenant :

- utiliser des paramètres de route pour transmettre des données à une route
- utiliser la directive `routerLink` pour créer une route à partir de données dynamiques
- utiliser un paramètre de route pour récupérer des données depuis `HousingService` afin d'afficher les informations d'un logement précis.

Excellent travail jusqu'ici.

Pour plus d'informations sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="guide/routing/read-route-state#get-information-about-the-current-route-with-activatedroute" title="Paramètres de route"/>
  <docs-pill href="guide/routing" title="Le routing dans Angular"/>
  <docs-pill href="guide/routing/common-router-tasks" title="Tâches courantes de routing"/>
  <docs-pill href="https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Optional_chaining" title="Opérateur de chaînage optionnel"/>
</docs-pill-row>
