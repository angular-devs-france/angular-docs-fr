<!-- source: 36f8585f5d7958062b048129fa7c1409b76f1635 -->
# Ajouter la soumission du formulaire

Pour finir, voyons comment gérer la soumission du formulaire. Vous allez apprendre à utiliser la fonction `submit()` pour exécuter des opérations asynchrones quand le formulaire est valide, et à désactiver le bouton de soumission quand le formulaire contient des erreurs.

Dans cette activité, vous allez apprendre à :

- Importer la fonction `submit()`
- Créer une méthode de gestion de la soumission
- Utiliser `submit()` pour n'exécuter la logique que si le formulaire est valide
- Désactiver le bouton de soumission en fonction de l'état du formulaire

Terminons le formulaire !

<hr />

<docs-workflow>

<docs-step title="Importer la fonction submit {#import-the-submit-function}">
Importez la fonction `submit` depuis `@angular/forms/signals` :

```ts
import {form, FormField, required, email, submit} from '@angular/forms/signals';
```

</docs-step>

<docs-step title="Ajouter la méthode onSubmit {#add-the-onsubmit-method}">
Dans la classe de votre composant, ajoutez une méthode `onSubmit()` qui gère la soumission du formulaire :

```ts
onSubmit(event: Event) {
  event.preventDefault();
  submit(this.loginForm, async () => {
    const credentials = this.loginModel();
    console.log('Logging in with:', credentials);
    // Add your login logic here
  });
}
```

La fonction `submit()` n'exécute votre callback asynchrone que si le formulaire est valide. Elle gère aussi automatiquement l'état de soumission du formulaire.
</docs-step>

<docs-step title="Lier le gestionnaire de soumission au formulaire {#bind-the-submit-handler-to-the-form}">
Dans votre template, liez la méthode `onSubmit()` à l'événement submit du formulaire :

```html
<form (submit)="onSubmit($event)"></form>
```

</docs-step>

<docs-step title="Désactiver le bouton quand le formulaire est invalide {#disable-the-button-when-form-is-invalid}">
Modifiez le bouton de soumission pour qu'il soit désactivé quand le formulaire est invalide :

```html
<button type="submit" [disabled]="loginForm().invalid()">Log in</button>
```

Cela empêche la soumission quand le formulaire contient des erreurs de validation.
</docs-step>

</docs-workflow>

Félicitations ! Vous avez créé un formulaire de connexion complet avec les Signal Forms.

Prêt à revoir ce que vous avez appris et à explorer des sujets avancés ? Passez aux [prochaines étapes](/tutorials/signal-forms/6-next-steps) !
