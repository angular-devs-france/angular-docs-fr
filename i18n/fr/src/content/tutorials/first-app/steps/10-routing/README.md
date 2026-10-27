<!-- source: db18b91ed28f2a6c0af0a5f1c6398a645c3e3a47 -->
# Ajouter des routes à l'application

Cette leçon du tutoriel montre comment ajouter des routes à votre application.

<docs-video src="https://www.youtube.com/embed/r5DEBMuStPw?si=H6Bx6nLJoMLaMxkx" />

IMPORTANT: Nous vous recommandons d'utiliser votre environnement local pour apprendre le routing.

## Ce que vous allez apprendre {#what-youll-learn}

À la fin de cette leçon, votre application prendra en charge le routing.

## Aperçu conceptuel du routing {#conceptual-preview-of-routing}

Ce tutoriel présente le routing dans Angular. Le routing est la capacité de naviguer d'un composant de l'application à un autre. Dans les [Single Page Applications (SPA)](guide/routing), seules certaines parties de la page sont mises à jour pour représenter la vue demandée par l'utilisateur.

Le [router Angular](guide/routing) permet aux utilisateurs de déclarer des routes et d'indiquer quel composant doit être affiché à l'écran si cette route est demandée par l'application.

Dans cette leçon, vous allez activer le routing dans votre application pour naviguer vers la page de détails.

<docs-workflow>

<docs-step title="Créer un composant de détails par défaut {#create-a-default-details-component-}">
1. Depuis le terminal, saisissez la commande suivante pour créer le `Details` :

    ```shell
    ng generate component details
    ```

    Ce composant représentera la page de détails qui fournit plus d'informations sur un logement donné.

</docs-step>

<docs-step title="Ajouter le routing à l'application {#add-routing-to-the-application}">
1.  Dans le répertoire `src/app`, créez un fichier nommé `routes.ts`. C'est dans ce fichier que nous définirons les routes de l'application.

2.  Dans `main.ts`, apportez les modifications suivantes pour activer le routing dans l'application :
    1.  Importez le fichier de routes et la fonction `provideRouter` :

          <docs-code header="Importer les éléments de routing dans src/main.ts" path="adev/src/content/tutorials/first-app/steps/11-details-page/src/main.ts" visibleLines="[3,4]"/>

    1.  Modifiez l'appel à `bootstrapApplication` pour inclure la configuration du routing :

          <docs-code header="Ajouter la configuration du router dans src/main.ts" path="adev/src/content/tutorials/first-app/steps/11-details-page/src/main.ts" visibleLines="[6,8]"/>

3.  Dans `src/app/app.ts`, modifiez le composant pour qu'il utilise le routing :
    1.  Ajoutez des imports au niveau du fichier pour les directives du router `RouterOutlet` et `RouterLink` :

          <docs-code language="angular-ts" header="Importer les directives du router dans src/app/app.ts" path="adev/src/content/tutorials/first-app/steps/11-details-page/src/app/app.ts" visibleLines="[3]"/>

    1.  Ajoutez `RouterOutlet` et `RouterLink` aux imports des métadonnées de `@Component`

          <docs-code language="angular-ts" header="Ajouter les directives du router aux imports du composant dans src/app/app.ts" path="adev/src/content/tutorials/first-app/steps/11-details-page/src/app/app.ts" visibleLines="[6]"/>

    1.  Dans la propriété `template`, remplacez la balise `<app-home />` par la directive `<router-outlet>` et ajoutez un lien de retour vers la page d'accueil. Votre code doit correspondre à celui-ci :

          <docs-code language="angular-ts" header="Ajouter router-outlet dans src/app/app.ts" path="adev/src/content/tutorials/first-app/steps/11-details-page/src/app/app.ts" visibleLines="[7,18]"/>

</docs-step>

<docs-step title="Ajouter une route vers le nouveau composant {#add-route-to-new-component}">
À l'étape précédente, vous avez supprimé la référence au composant `<app-home>` dans le template. Dans cette étape, vous allez ajouter une nouvelle route vers ce composant.

1. Dans `routes.ts`, apportez les modifications suivantes pour créer une route.
   1. Ajoutez des imports au niveau du fichier pour `Home`, `Details` et le type `Routes` que vous utiliserez dans les définitions de routes.

      <docs-code header="Importer les composants et Routes" path="adev/src/content/tutorials/first-app/steps/11-details-page/src/app/routes.ts" visibleLines="[1,3]"/>

   1. Définissez une variable nommée `routeConfig` de type `Routes` et définissez deux routes pour l'application :
      <docs-code header="Ajouter des routes à l'application" path="adev/src/content/tutorials/first-app/steps/11-details-page/src/app/routes.ts" visibleLines="[5,18]"/>

      Les entrées du tableau `routeConfig` représentent les routes de l'application. La première entrée navigue vers `Home` chaque fois que l'URL correspond à `''`. La seconde entrée utilise un format particulier qui sera abordé dans une prochaine leçon.

1. Enregistrez toutes les modifications et vérifiez que l'application fonctionne dans le navigateur. L'application doit toujours afficher la liste des logements.
   </docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez activé le routing dans votre application et défini de nouvelles routes. Votre application peut maintenant prendre en charge la navigation entre les vues. Dans la prochaine leçon, vous apprendrez à naviguer vers la page « details » d'un logement donné.

Vous progressez bien avec votre application, bravo.

Pour plus d'informations sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="guide/routing" title="Le routing dans Angular"/>
  <docs-pill href="guide/routing/common-router-tasks" title="Tâches courantes de routing"/>
</docs-pill-row>
