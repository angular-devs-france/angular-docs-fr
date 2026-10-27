<!-- source: 46e4a1dd68f8df887858a35056097d57f6ae221e -->
# Gestion des événements

La gestion des événements rend possibles les fonctionnalités interactives des applications web. En tant que développeur, elle vous permet de réagir aux actions de l'utilisateur, comme les clics sur un bouton, les soumissions de formulaire et plus encore.

NOTE: Pour en savoir plus, consultez [la gestion des interactions utilisateur dans le guide des Fondamentaux](/essentials/templates#handling-user-interaction).

Dans cette activité, vous allez apprendre à ajouter un event handler.

<hr />

Dans Angular, vous vous liez aux événements avec la syntaxe des parenthèses `()`. Sur un élément donné, entourez de parenthèses l'événement auquel vous voulez vous lier et définissez un event handler. Prenons cet exemple de `button` :

```angular-ts
@Component({
  ...
  template: `<button (click)="greet()">`
})
export class App {
  greet() {
    console.log('Hello, there 👋');
  }
}
```

Dans cet exemple, la fonction `greet()` s'exécute à chaque clic sur le bouton. Notez que la syntaxe `greet()` inclut les parenthèses finales.

À vous d'essayer :

<docs-workflow>

<docs-step title="Ajouter un event handler {#add-an-event-handler}">
Dans la classe `App`, complétez le corps de la méthode existante `showSecretMessage()` avec le code suivant :

```ts
this.message = 'Way to go 🚀';
```

</docs-step>

<docs-step title="Lier l'événement dans le template {#bind-to-the-template-event}">
Mettez à jour le code du template dans `app.ts` pour lier l'événement `mouseover` de l'élément `section`.

<!-- prettier-ignore -->
```angular-html
<section (mouseover)="showSecretMessage()">
```

</docs-step>

</docs-workflow>

En quelques étapes dans le code, vous avez créé votre premier event handler dans Angular. Vous semblez vous débrouiller plutôt bien : continuez comme ça.
