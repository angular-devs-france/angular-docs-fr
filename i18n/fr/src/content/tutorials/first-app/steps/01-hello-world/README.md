<!-- source: 22752d550080583e24093d34349bc56b75354667 -->
# Hello world

Cette première leçon sert de point de départ : chaque leçon de ce tutoriel y ajoute de nouvelles fonctionnalités pour construire une application Angular complète. Dans cette leçon, nous allons modifier l'application pour qu'elle affiche le célèbre texte « Hello World ».

<docs-video src="https://www.youtube.com/embed/UnOwDuliqZA?si=uML-cDRbrxmYdD_9"/>

## Ce que vous allez apprendre {#what-youll-learn}

Si votre application s'exécute correctement dans le navigateur et affiche le contenu sans erreur, cela confirme que votre environnement de développement et votre IDE sont correctement configurés pour commencer à créer une application Angular.

NOTE: Si vous travaillez avec l'éditeur intégré, passez directement à l'[étape quatre](#create-hello-world).
Dans le Playground du navigateur, vous n'avez pas besoin de lancer `ng serve` pour exécuter l'application. Les autres commandes, comme `ng generate`, peuvent être lancées dans la console à votre droite.

<docs-workflow>

<docs-step title="Télécharger l'application par défaut {#download-the-default-app}">
Commencez par cliquer sur l'icône « Download » dans le panneau en haut à droite de l'éditeur de code. Cela télécharge un fichier `.zip` contenant le code source de ce tutoriel. Ouvrez-le dans votre Terminal et votre IDE locaux, puis passez au test de l'application par défaut.

À n'importe quelle étape du tutoriel, vous pouvez cliquer sur cette icône pour télécharger le code source de l'étape et repartir de là.
</docs-step>

<docs-step title="Tester l'application par défaut {#test-the-default-app}">
Dans cette étape, après avoir téléchargé l'application de départ, vous compilez l'application Angular par défaut.
Cela confirme que votre environnement de développement dispose de ce qu'il faut pour poursuivre le tutoriel.

Dans le panneau **Terminal** de votre IDE :

1. Dans le répertoire de votre projet, allez dans le répertoire `first-app`.
1. Exécutez cette commande pour installer les dépendances nécessaires à l'exécution de l'application.

   ```shell
   npm install
   ```

1. Exécutez cette commande pour compiler et servir l'application par défaut.

   ```shell
   ng serve
   ```

   L'application doit se compiler sans erreur.

1. Dans un navigateur web sur votre ordinateur de développement, ouvrez `http://localhost:4200`.
1. Vérifiez que le site web par défaut s'affiche dans le navigateur.
1. Vous pouvez laisser `ng serve` tourner pendant que vous réalisez les étapes suivantes.
   </docs-step>

<docs-step title="Passer en revue les fichiers du projet {#review-the-files-in-the-project}">
Dans cette étape, vous découvrez les fichiers qui composent une application Angular par défaut.

Dans le panneau **Explorer** de votre IDE :

1. Dans le répertoire de votre projet, allez dans le répertoire `first-app`.
1. Ouvrez le répertoire `src` pour voir ces fichiers.
   1. Dans l'explorateur de fichiers, trouvez les fichiers de l'application Angular (`/src`).
      1. `index.html` est le template HTML de plus haut niveau de l'application.
      1. `styles.css` est la feuille de style de plus haut niveau de l'application.
      1. `main.ts` est l'endroit où l'application démarre.
      1. `favicon.ico` est l'icône de l'application, comme sur n'importe quel site web.
   1. Dans l'explorateur de fichiers, trouvez les fichiers de composant de l'application Angular (`/app`).
      1. `app.ts` est le fichier source qui décrit le composant `app-root`.
         C'est le composant Angular de plus haut niveau de l'application. Un composant est la brique de base d'une application Angular.
         La description du composant comprend son code, son template HTML et ses styles, qui peuvent être décrits dans ce fichier ou dans des fichiers séparés.

         Dans cette application, les styles sont dans un fichier séparé, tandis que le code et le template HTML du composant sont dans ce fichier.

      1. `app.css` est la feuille de style de ce composant.
      1. Les nouveaux composants sont ajoutés dans ce répertoire.

   1. Dans l'explorateur de fichiers, trouvez le répertoire des images (`/public`), qui contient les images utilisées par l'application.
   1. Dans l'explorateur de fichiers, trouvez les fichiers et répertoires dont une application Angular a besoin pour se compiler et s'exécuter, mais avec lesquels vous n'interagissez normalement pas.
      1. `.angular` contient les fichiers nécessaires à la compilation de l'application Angular.
      1. `.e2e` contient les fichiers utilisés pour tester l'application.
      1. `.node_modules` contient les packages node.js utilisés par l'application.
      1. `angular.json` décrit l'application Angular aux outils de build.
      1. `package.json` est utilisé par `npm` (le node package manager) pour exécuter l'application finale.
      1. `tsconfig.*` sont les fichiers qui décrivent la configuration de l'application au compilateur TypeScript.

Une fois que vous avez passé en revue les fichiers qui composent un projet d'application Angular, passez à l'étape suivante.
</docs-step>

<docs-step title="Créer `Hello World` {#create-hello-world}">
Dans cette étape, vous modifiez les fichiers du projet Angular pour changer le contenu affiché.

Dans votre IDE :

1. Ouvrez `first-app/src/index.html`.
   NOTE: Cette étape et la suivante concernent uniquement votre environnement local !

1. Dans `index.html`, remplacez l'élément `<title>` par ce code pour modifier le titre de l'application.

   <docs-code header="Remplacer dans src/index.html" path="adev/src/content/tutorials/first-app/steps/02-Home/src/index.html" visibleLines="[5]"/>

   Enregistrez ensuite les modifications que vous venez d'apporter à `index.html`.

1. Ouvrez ensuite `first-app/src/app/app.ts`.
1. Dans `app.ts`, dans la définition `@Component`, remplacez la ligne `template` par ce code pour changer le texte du composant de l'application.

   <docs-code language="angular-ts" header="Remplacer dans src/app/app.ts" path="adev/src/content/tutorials/first-app/steps/02-Home/src/app/app.ts" visibleLines="[6]"/>

1. Dans `app.ts`, dans la définition de la classe `App`, remplacez la ligne `title` par ce code pour changer le titre du composant.

   <docs-code header="Remplacer dans src/app/app.ts" path="adev/src/content/tutorials/first-app/steps/02-Home/src/app/app.ts" visibleLines="[10]"/>

   Enregistrez ensuite les modifications apportées à `app.ts`.

1. Si vous avez arrêté la commande `ng serve` de l'étape 1, exécutez à nouveau `ng serve` dans la fenêtre **Terminal** de votre IDE.
1. Ouvrez votre navigateur, allez sur `localhost:4200` et vérifiez que l'application se compile sans erreur et affiche _Homes_ dans le titre et _Hello world_ dans le corps de votre application :
   <img alt="fenêtre de navigateur affichant une page avec le texte 'Hello World'" src="assets/images/tutorials/first-app/homes-app-lesson-01-browser.png">
   </docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez modifié une application Angular par défaut pour qu'elle affiche _Hello world_.
Ce faisant, vous avez découvert la commande `ng serve`, qui sert votre application en local pour la tester.

Pour en savoir plus sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="guide/components" title="Les composants Angular"/>
  <docs-pill href="tools/cli" title="Créer des applications avec l'Angular CLI"/>
</docs-pill-row>
