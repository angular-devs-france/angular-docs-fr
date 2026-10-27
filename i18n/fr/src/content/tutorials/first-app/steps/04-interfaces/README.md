<!-- source: 680f13658ca3c2d82ab84ec12f9001eab286e117 -->
# Créer une interface

Cette leçon du tutoriel montre comment créer une interface et l'inclure dans un composant de votre application.

<docs-video src="https://www.youtube.com/embed/eM3zi_n7lNs?si=YkFSeUeV8Ixtz8pm"/>

## Ce que vous allez apprendre {#what-youll-learn}

- Votre application dispose d'une nouvelle interface qu'elle peut utiliser comme type de données.
- Votre application dispose d'une instance de cette nouvelle interface avec des données d'exemple.

## Aperçu conceptuel des interfaces {#conceptual-preview-of-interfaces}

Les [interfaces](https://www.typescriptlang.org/docs/handbook/interfaces.html) sont des types de données personnalisés pour votre application.

Angular utilise TypeScript pour tirer parti d'un environnement de programmation fortement typé.
Une vérification de types stricte réduit le risque qu'un élément de votre application envoie des données mal formatées à un autre.
Ces erreurs d'incompatibilité de types sont détectées par le compilateur TypeScript, et beaucoup peuvent aussi être détectées dans votre IDE.

Dans cette leçon, vous allez créer une interface pour définir les propriétés qui représentent les données d'un logement.

<docs-workflow>

<docs-step title="Créer une nouvelle interface Angular {#create-a-new-angular-interface}">
Cette étape crée une nouvelle interface dans votre application.

Dans le panneau **Terminal** de votre IDE :

1. Dans le répertoire de votre projet, allez dans le répertoire `first-app`.
1. Dans le répertoire `first-app`, exécutez cette commande pour créer la nouvelle interface.

   ```shell

   ng generate interface housinglocation

   ```

1. Exécutez `ng serve` pour compiler l'application et la servir sur `http://localhost:4200`.
1. Dans un navigateur, ouvrez `http://localhost:4200` pour voir votre application.
1. Vérifiez que l'application se compile sans erreur.
   Corrigez les éventuelles erreurs avant de passer à l'étape suivante.
   </docs-step>

<docs-step title="Ajouter des propriétés à la nouvelle interface {#add-properties-to-the-new-interface}">
Cette étape ajoute à l'interface les propriétés dont votre application a besoin pour représenter un logement.

1.  Dans le panneau **Terminal** de votre IDE, lancez la commande `ng serve` si elle ne tourne pas déjà, pour compiler l'application et la servir sur `http://localhost:4200`.
1.  Dans le panneau **Edit** de votre IDE, ouvrez le fichier `src/app/housinglocation.ts`.
1.  Dans `housinglocation.ts`, remplacez le contenu par défaut par le code suivant pour que votre nouvelle interface corresponde à cet exemple.

      <docs-code header="Mettre à jour src/app/housinglocation.ts pour correspondre à ce code" path="adev/src/content/tutorials/first-app/steps/05-inputs/src/app/housinglocation.ts" visibleLines="[1,10]" />

1.  Enregistrez vos modifications et vérifiez que l'application n'affiche aucune erreur. Corrigez les éventuelles erreurs avant de passer à l'étape suivante.

À ce stade, vous avez défini une interface qui représente les données d'un logement, dont un `id`, un `name` et des informations de localisation.
</docs-step>

<docs-step title="Créer un logement de test pour votre application {#create-a-test-house-for-your-app}">
Vous avez une interface, mais vous ne l'utilisez pas encore.

Dans cette étape, vous créez une instance de l'interface et lui affectez des données d'exemple.
Ces données d'exemple n'apparaissent pas encore dans votre application.
Il reste quelques leçons à suivre avant que cela n'arrive.

1.  Dans le panneau **Terminal** de votre IDE, exécutez la commande `ng serve` si elle ne tourne pas déjà, pour compiler l'application et la servir sur `http://localhost:4200`.
1.  Dans le panneau **Edit** de votre IDE, ouvrez `src/app/home/home.ts`.
1.  Dans `src/app/home/home.ts`, ajoutez cette instruction d'import après les instructions `import` existantes afin que `Home` puisse utiliser la nouvelle interface.

      <docs-code language="angular-ts" header="Importer Home dans src/app/home/home.ts" path="adev/src/content/tutorials/first-app/steps/05-inputs/src/app/home/home.ts" visibleLines="[3]"/>

1.  Dans `src/app/home/home.ts`, remplacez la définition vide `export class Home {}` par ce code pour créer une instance unique de la nouvelle interface dans le composant.

      <docs-code language="angular-ts" header="Ajouter des données d'exemple dans src/app/home/home.ts" path="adev/src/content/tutorials/first-app/steps/05-inputs/src/app/home/home.ts" visibleLines="[21,34]"/>

1.  Vérifiez que votre fichier `home.ts` correspond à cet exemple.

      <docs-code language="angular-ts" header="src/app/home/home.ts" path="adev/src/content/tutorials/first-app/steps/05-inputs/src/app/home/home.ts" visibleLines="[1,34]" />

    En ajoutant la propriété `housingLocation` de type `HousingLocation` à la classe `Home`, nous pouvons vérifier que les données correspondent à la description de l'interface. Si les données ne respectaient pas la description de l'interface, l'IDE disposerait de suffisamment d'informations pour nous signaler des erreurs utiles.

1.  Enregistrez vos modifications et vérifiez que l'application ne présente aucune erreur. Ouvrez le navigateur et vérifiez que votre application affiche toujours le message _housing-location works!_

      <img alt="fenêtre de navigateur de homes-app affichant le logo, le champ de texte de filtre, le bouton de recherche et le message 'housing-location works!'" src="assets/images/tutorials/first-app/homes-app-lesson-03-step-2.png">

1.  Corrigez les éventuelles erreurs avant de passer à l'étape suivante.
    </docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez créé une interface qui définit un nouveau type de données pour votre application.
Ce nouveau type de données vous permet de préciser où des données `HousingLocation` sont requises.
Il permet aussi à votre IDE et au compilateur TypeScript de garantir que des données `HousingLocation` sont utilisées là où elles sont requises.

Pour en savoir plus sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="cli/generate/interface" title="ng generate interface"/>
  <docs-pill href="cli/generate" title="ng generate"/>
</docs-pill-row>
