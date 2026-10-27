<!-- source: 8627604e0df8cef344fc724dfdc340508579fd24 -->
# Créer votre première application Angular

Ce tutoriel se compose de leçons qui présentent les concepts Angular à connaître pour commencer à coder avec Angular.

Vous pouvez en suivre autant que vous le souhaitez, et dans l'ordre de votre choix.

HELPFUL: Vous préférez la vidéo ? Nous proposons aussi un [cours YouTube](https://youtube.com/playlist?list=PL1w1q3fL4pmj9k1FrJ3Pe91EPub2_h4jF&si=1q9889ulHp8VZ0e7) complet pour ce tutoriel !

<docs-video src="https://www.youtube.com/embed/xAT0lHYhHMY?si=cKUW_MGn3MesFT7o"/>

## Avant de commencer {#before-you-start}

Pour tirer le meilleur parti de ce tutoriel, passez en revue ces prérequis afin de vous assurer de disposer de tout ce qu'il vous faut pour réussir.

### Votre expérience {#your-experience}

Les leçons de ce tutoriel supposent que vous avez déjà :

1. Créé une page web HTML en modifiant directement le HTML.
1. Programmé le contenu d'un site web en JavaScript.
1. Lu du contenu CSS (Cascading Style Sheets) et compris comment les sélecteurs sont utilisés.
1. Utilisé des instructions en ligne de commande pour effectuer des tâches sur votre ordinateur.

### Votre équipement {#your-equipment}

Ces leçons peuvent être suivies avec une installation locale des outils Angular ou dans notre éditeur intégré. Le développement Angular en local est possible sous Windows, macOS ou Linux.

NOTE: Repérez les alertes comme celle-ci, qui signalent les étapes susceptibles de ne concerner que votre éditeur local.

## Aperçu conceptuel de votre première application Angular {#conceptual-preview-of-your-first-angular-app}

Les leçons de ce tutoriel créent une application Angular qui liste des logements à louer et affiche le détail de chaque logement.
Cette application utilise des fonctionnalités communes à de nombreuses applications Angular.

<img alt="Rendu de la page d'accueil de l'application homes" src="assets/images/tutorials/first-app/homes-app-landing-page.png">

## Environnement de développement local {#local-development-environment}

NOTE: Cette étape concerne uniquement votre environnement local !

Effectuez ces étapes dans un outil en ligne de commande sur l'ordinateur que vous souhaitez utiliser pour ce tutoriel.

<docs-workflow>

<docs-step title="Identifier la version de `node.js` requise par Angular {#identify-the-version-of-nodejs-that-angular-requires}">
Angular nécessite une version LTS active ou LTS de maintenance de Node. Vérifions votre version de `node.js`. Pour en savoir plus sur les versions requises, consultez la propriété engines dans le [fichier package.json](https://unpkg.com/browse/@angular/core@15.1.5/package.json).

Depuis une fenêtre de **Terminal** :

1. Exécutez la commande suivante : `node --version`
1. Vérifiez que le numéro de version affiché répond aux exigences.
   </docs-step>

<docs-step title="Installer la bonne version de `node.js` pour Angular {#install-the-correct-version-of-nodejs-for-angular}">
Si aucune version de `node.js` n'est installée, suivez les [instructions d'installation sur nodejs.org](https://nodejs.org/en/download/)
</docs-step>

<docs-step title="Installer la dernière version d'Angular {#install-the-latest-version-of-angular}">
Une fois `node.js` et `npm` installés, l'étape suivante consiste à installer l'[Angular CLI](tools/cli), qui fournit les outils pour développer efficacement avec Angular.

Depuis une fenêtre de **Terminal**, exécutez la commande suivante : `npm install -g @angular/cli`.
</docs-step>

<docs-step title="Installer un environnement de développement intégré (IDE) {#install-integrated-development-environment-ide}">
Vous êtes libre d'utiliser l'outil de votre choix pour créer des applications avec Angular. Nous recommandons les suivants :

1. [Visual Studio Code](https://code.visualstudio.com/)
2. Étape facultative mais recommandée : vous pouvez améliorer encore votre expérience de développement en installant l'[Angular Language Service](https://marketplace.visualstudio.com/items?itemName=Angular.ng-template)
3. [WebStorm](https://www.jetbrains.com/webstorm/)
   </docs-step>

<docs-step title="Facultatif : configurer votre IDE assisté par IA {#optional-set-up-your-ai-powered-ide}">

Si vous suivez ce tutoriel dans votre IDE assisté par IA préféré, [consultez les règles de prompt et les bonnes pratiques Angular](/ai/develop-with-ai).

</docs-step>

</docs-workflow>

Pour en savoir plus sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="/overview" title="Qu'est-ce qu'Angular ?"/>
  <docs-pill href="/tools/cli/setup-local" title="Configurer l'environnement local et le workspace"/>
  <docs-pill href="/cli" title="Référence de l'Angular CLI"/>
</docs-pill-row>
