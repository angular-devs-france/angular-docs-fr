<!-- source: bf8a245989945f34d4bf04ea80064e9b226ce663 -->
# Ajouter la communication HTTP à votre application

Ce tutoriel montre comment intégrer HTTP et une API dans votre application.

Jusqu'à présent, votre application lisait les données depuis un tableau statique dans un service Angular. L'étape suivante consiste à utiliser un serveur JSON avec lequel votre application communiquera via HTTP. La requête HTTP simulera l'expérience de travailler avec des données provenant d'un serveur.

<docs-video src="https://www.youtube.com/embed/5K10oYJ5Y-E?si=TiuNKx_teR9baO7k"/>

IMPORTANT: Nous vous recommandons d'utiliser votre environnement local pour cette étape du tutoriel.

## Ce que vous allez apprendre {#what-youll-learn}

Votre application utilisera les données d'un serveur JSON

<docs-workflow>

<docs-step title="Configurer le serveur JSON {#configure-the-json-server}">
JSON Server est un outil open source utilisé pour créer des API REST simulées. Vous l'utiliserez pour servir les données de logements actuellement stockées dans le service de logements.

1. Installez `json-server` depuis npm à l'aide de la commande suivante.

   ```bash
   npm install -g json-server
   ```

1. Dans le répertoire racine de votre projet, créez un fichier nommé `db.json`. C'est là que vous stockerez les données pour `json-server`.

1. Ouvrez `db.json` et copiez-y le code suivant

   ```json
   {
     "locations": [
       {
         "id": 0,
         "name": "Acme Fresh Start Housing",
         "city": "Chicago",
         "state": "IL",
         "photo": "https://angular.dev/assets/images/tutorials/common/bernard-hermant-CLKGGwIBTaY-unsplash.jpg",
         "availableUnits": 4,
         "wifi": true,
         "laundry": true
       },
       {
         "id": 1,
         "name": "A113 Transitional Housing",
         "city": "Santa Monica",
         "state": "CA",
         "photo": "https://angular.dev/assets/images/tutorials/common/brandon-griggs-wR11KBaB86U-unsplash.jpg",
         "availableUnits": 0,
         "wifi": false,
         "laundry": true
       },
       {
         "id": 2,
         "name": "Warm Beds Housing Support",
         "city": "Juneau",
         "state": "AK",
         "photo": "https://angular.dev/assets/images/tutorials/common/i-do-nothing-but-love-lAyXdl1-Wmc-unsplash.jpg",
         "availableUnits": 1,
         "wifi": false,
         "laundry": false
       },
       {
         "id": 3,
         "name": "Homesteady Housing",
         "city": "Chicago",
         "state": "IL",
         "photo": "https://angular.dev/assets/images/tutorials/common/ian-macdonald-W8z6aiwfi1E-unsplash.jpg",
         "availableUnits": 1,
         "wifi": true,
         "laundry": false
       },
       {
         "id": 4,
         "name": "Happy Homes Group",
         "city": "Gary",
         "state": "IN",
         "photo": "https://angular.dev/assets/images/tutorials/common/krzysztof-hepner-978RAXoXnH4-unsplash.jpg",
         "availableUnits": 1,
         "wifi": true,
         "laundry": false
       },
       {
         "id": 5,
         "name": "Hopeful Apartment Group",
         "city": "Oakland",
         "state": "CA",
         "photo": "https://angular.dev/assets/images/tutorials/common/r-architecture-JvQ0Q5IkeMM-unsplash.jpg",
         "availableUnits": 2,
         "wifi": true,
         "laundry": true
       },
       {
         "id": 6,
         "name": "Seriously Safe Towns",
         "city": "Oakland",
         "state": "CA",
         "photo": "https://angular.dev/assets/images/tutorials/common/phil-hearing-IYfp2Ixe9nM-unsplash.jpg",
         "availableUnits": 5,
         "wifi": true,
         "laundry": true
       },
       {
         "id": 7,
         "name": "Hopeful Housing Solutions",
         "city": "Oakland",
         "state": "CA",
         "photo": "https://angular.dev/assets/images/tutorials/common/r-architecture-GGupkreKwxA-unsplash.jpg",
         "availableUnits": 2,
         "wifi": true,
         "laundry": true
       },
       {
         "id": 8,
         "name": "Seriously Safe Towns",
         "city": "Oakland",
         "state": "CA",
         "photo": "https://angular.dev/assets/images/tutorials/common/saru-robert-9rP3mxf8qWI-unsplash.jpg",
         "availableUnits": 10,
         "wifi": false,
         "laundry": false
       },
       {
         "id": 9,
         "name": "Capital Safe Towns",
         "city": "Portland",
         "state": "OR",
         "photo": "https://angular.dev/assets/images/tutorials/common/webaliser-_TPTXZd9mOo-unsplash.jpg",
         "availableUnits": 6,
         "wifi": true,
         "laundry": true
       }
     ]
   }
   ```

1. Enregistrez ce fichier.

1. Il est temps de tester votre configuration. Depuis la ligne de commande, à la racine de votre projet, exécutez les commandes suivantes.

   ```bash
   json-server --watch db.json
   ```

1. Dans votre navigateur web, accédez à `http://localhost:3000/locations` et vérifiez que la réponse contient les données stockées dans `db.json`.

Si vous rencontrez des difficultés avec votre configuration, vous trouverez plus de détails dans la [documentation officielle](https://www.npmjs.com/package/json-server).
</docs-step>

<docs-step title="Modifier le service pour utiliser un serveur web au lieu d'un tableau local {#update-service-to-use-web-server-instead-of-local-array}">
La source de données est configurée ; l'étape suivante consiste à modifier votre application web pour s'y connecter et utiliser les données.

1.  Dans `src/app/housing.service.ts`, apportez les modifications suivantes :

1.  Modifiez le code pour supprimer la propriété `housingLocationList` et le tableau contenant les données, ainsi que la propriété `baseUrl`.

1.  Ajoutez une propriété de type chaîne nommée `url` et affectez-lui la valeur `'http://localhost:3000/locations'`

    <docs-code header="Ajouter la propriété url dans housing.service.ts" path="adev/src/content/tutorials/first-app/steps/14-http/src-final/app/housing.service.ts" visibleLines="[6]"/>

    Ce code provoquera des erreurs dans le reste du fichier, car celui-ci dépend de la propriété `housingLocationList`. Nous allons ensuite modifier les méthodes du service.

1.  Modifiez la fonction `getAllHousingLocations` pour qu'elle appelle le serveur web que vous avez configuré.

     <docs-code header="Modifier la méthode getAllHousingLocations dans housing.service.ts" path="adev/src/content/tutorials/first-app/steps/14-http/src-final/app/housing.service.ts" visibleLines="[8,11]"/>

    Le code utilise maintenant du code asynchrone pour effectuer une requête **GET** via HTTP.

    HELPFUL: Dans cet exemple, le code utilise `fetch`. Pour des cas d'usage plus avancés, envisagez d'utiliser le `HttpClient` fourni par Angular.

1.  Modifiez la fonction `getHousingLocationById` pour qu'elle appelle le serveur web que vous avez configuré.

    HELPFUL: Remarquez que l'URL de `fetch` ajoute maintenant l'`id` comme segment de chemin (`${this.url}/${id}`) pour demander le logement dont la propriété `id` a la valeur correspondante.

     <docs-code header="Modifier la méthode getHousingLocationById dans housing.service.ts" path="adev/src/content/tutorials/first-app/steps/14-http/src-final/app/housing.service.ts" visibleLines="[13,17]"/>

1.  Une fois toutes les modifications terminées, votre service modifié doit correspondre au code suivant.

     <docs-code header="Version finale de housing.service.ts" path="adev/src/content/tutorials/first-app/steps/14-http/src-final/app/housing.service.ts" visibleLines="[1,23]" />

</docs-step>

<docs-step title="Modifier les composants pour utiliser des appels asynchrones au service de logements {#update-the-components-to-use-asynchronous-calls-to-the-housing-service}">
Le serveur lit maintenant les données à partir de la requête HTTP, mais les composants qui dépendent du service comportent désormais des erreurs, car ils ont été programmés pour utiliser la version synchrone du service.

1.  Dans `src/app/home/home.ts`, modifiez le `constructor` pour utiliser la nouvelle version asynchrone de la méthode `getAllHousingLocations`. Comme nous n'avons pas utilisé de signals pour notre état, vous devez notifier Angular qu'un changement nécessitant une synchronisation s'est produit. Appelez `this.changeDetectorRef.markForCheck()` pour cela.

      <docs-code header="Modifier le constructor dans home.ts" path="adev/src/content/tutorials/first-app/steps/14-http/src-final/app/home/home.ts" visibleLines="[30,38]"/>

1.  Dans `src/app/details/details.ts`, modifiez le `constructor` pour utiliser la nouvelle version asynchrone de la méthode `getHousingLocationById`. Comme précédemment, vous devez aussi appeler `this.changeDetectorRef.markForCheck()` pour notifier Angular des changements.

      <docs-code header="Modifier le constructor dans details.ts" path="adev/src/content/tutorials/first-app/steps/14-http/src-final/app/details/details.ts" visibleLines="[60,66]"/>

1.  Enregistrez votre code.

1.  Ouvrez l'application dans le navigateur et vérifiez qu'elle s'exécute sans erreur.
    </docs-step>

</docs-workflow>

NOTE: Cette leçon s'appuie sur l'API `fetch` du navigateur. Pour la prise en charge des intercepteurs, consultez la [documentation du client HTTP](/guide/http)

SUMMARY: Dans cette leçon, vous avez modifié votre application pour utiliser un serveur web local (`json-server`) et des méthodes de service asynchrones pour récupérer des données.

Félicitations ! Vous avez terminé ce tutoriel avec succès et vous êtes prêt à poursuivre votre parcours en construisant des applications Angular encore plus complexes.

Si vous souhaitez en apprendre davantage, envisagez de suivre d'autres [tutoriels](tutorials) et [guides](overview) Angular destinés aux développeurs.
