<!-- source: 911633123101e6f8ff506d08e70c1b3a55e913f8 -->
# Ajouter de la validation à votre formulaire

Ajouter de la validation à votre formulaire est essentiel pour garantir que les utilisateurs saisissent des données valides. Les Signal Forms utilisent des validateurs dans une fonction de schéma que vous passez à la fonction `form()`.

Dans cette activité, vous allez apprendre à :

- Importer les validateurs intégrés
- Définir une fonction de schéma pour votre formulaire
- Appliquer des validateurs à des champs précis avec des messages d'erreur personnalisés

Ajoutons de la validation !

<hr />

<docs-workflow>

<docs-step title="Importer les validateurs {#import-the-validators}">
Importez les validateurs `required` et `email` depuis `@angular/forms/signals` :

```ts
import {form, FormField, required, email} from '@angular/forms/signals';
```

</docs-step>

<docs-step title="Ajouter une fonction de schéma à votre formulaire {#add-a-schema-function-to-your-form}">
Modifiez votre appel à `form()` pour inclure une fonction de schéma en second paramètre. La fonction de schéma reçoit un paramètre `fieldPath` qui vous permet d'accéder à chaque champ :

```ts
loginForm = form(this.loginModel, (fieldPath) => {
  // Validators will go here
});
```

</docs-step>

<docs-step title="Ajouter de la validation au champ email {#add-validation-to-the-email-field}">
Dans la fonction de schéma, ajoutez la validation du champ email. Utilisez les deux validateurs `required()` et `email()` :

```ts
loginForm = form(this.loginModel, (fieldPath) => {
  required(fieldPath.email, {message: 'Email is required'});
  email(fieldPath.email, {message: 'Enter a valid email address'});
});
```

L'option `message` fournit des messages d'erreur personnalisés aux utilisateurs.
</docs-step>

<docs-step title="Ajouter de la validation au champ password {#add-validation-to-the-password-field}">
Ajoutez la validation du champ password avec le validateur `required()` :

```ts
loginForm = form(this.loginModel, (fieldPath) => {
  required(fieldPath.email, {message: 'Email is required'});
  email(fieldPath.email, {message: 'Enter a valid email address'});
  required(fieldPath.password, {message: 'Password is required'});
});
```

</docs-step>

</docs-workflow>

Parfait ! Vous avez ajouté de la validation à votre formulaire. Les validateurs s'exécutent automatiquement à mesure que les utilisateurs interagissent avec le formulaire. Quand la validation échoue, l'état du champ reflète les erreurs.

Ensuite, vous allez apprendre [à afficher les erreurs de validation dans le template](/tutorials/signal-forms/4-display-errors) !
