<!-- source: f5b4e95e986967340426e99dad205f0066516656 -->
# Afficher les erreurs de validation

Maintenant que vous savez valider le formulaire, il est important d'afficher les erreurs de validation aux utilisateurs.

Dans cette activité, vous allez apprendre à :

- Accéder à l'état des champs grâce aux signals de validation
- Utiliser `@if` pour afficher les erreurs de manière conditionnelle
- Parcourir les erreurs avec `@for`
- Afficher les erreurs uniquement après une interaction de l'utilisateur

Affichons les retours de validation !

<hr />

<docs-workflow>

<docs-step title="Afficher les erreurs du champ email {#add-error-display-for-email-field}">
Sous l'input email, ajoutez un affichage conditionnel des erreurs. Les erreurs ne s'afficheront que lorsque le champ est à la fois invalide et touché :

```angular-html
<label>
  Email
  <input type="email" [formField]="loginForm.email" />
</label>
@if (loginForm.email().invalid() && loginForm.email().touched()) {
  <div class="error">
    @for (error of loginForm.email().errors(); track error.kind) {
      <span>{{ error.message }}</span>
    }
  </div>
}
```

L'appel `loginForm.email()` accède au signal d'état du champ. La méthode `invalid()` renvoie `true` quand la validation échoue, `touched()` renvoie `true` une fois que l'utilisateur a interagi avec le champ, et `errors()` fournit un tableau des erreurs de validation avec leurs messages personnalisés.
</docs-step>

<docs-step title="Afficher les erreurs du champ password {#add-error-display-for-password-field}">
Sous l'input password, ajoutez le même pattern pour les erreurs du mot de passe :

```angular-html
<label>
  Password
  <input type="password" [formField]="loginForm.password" />
</label>
@if (loginForm.password().invalid() && loginForm.password().touched()) {
  <div class="error">
    @for (error of loginForm.password().errors(); track error.kind) {
      <span>{{ error.message }}</span>
    }
  </div>
}
```

</docs-step>

</docs-workflow>

Excellent ! Vous avez ajouté l'affichage des erreurs à votre formulaire. Les erreurs n'apparaissent qu'après une interaction de l'utilisateur avec un champ : elles apportent un retour utile sans être intrusives.

Ensuite, vous allez apprendre [à gérer la soumission du formulaire](/tutorials/signal-forms/5-add-submission) !
