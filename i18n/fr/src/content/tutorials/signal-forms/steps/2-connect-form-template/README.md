<!-- source: 83725ef98073620048d3553944ab28a6a66b458e -->
# Connecter votre formulaire au template

Vous devez maintenant connecter votre formulaire au template à l'aide de la directive `[formField]`. Elle crée un two-way binding des données entre votre model de formulaire et les éléments de saisie.

Dans cette leçon, vous allez apprendre à :

- Importer la directive `FormField`
- Utiliser la directive `[formField]` pour lier les champs du formulaire aux éléments de saisie
- Connecter des champs texte et des cases à cocher à votre formulaire
- Afficher les valeurs des champs du formulaire dans le template

Branchons le template !

<hr />

<docs-workflow>

<docs-step title="Importer la directive FormField {#import-the-formfield-directive}">
Importez la directive `FormField` depuis `@angular/forms/signals` et ajoutez-la au tableau imports de votre composant :

```ts
import { form, FormField } from '@angular/forms/signals';

@Component({
  selector: 'app-root',
  templateUrl: './app.html',
  styleUrl: './app.css',
  imports: [FormField],
})
```

</docs-step>

<docs-step title="Lier le champ email {#bind-the-email-field}">
Dans votre template, ajoutez la directive `[formField]` à l'input email :

```html
<input type="email" [formField]="loginForm.email" />
```

L'expression `loginForm.email` accède au champ email de votre formulaire.
</docs-step>

<docs-step title="Lier le champ password {#bind-the-password-field}">
Ajoutez la directive `[formField]` à l'input password :

```html
<input type="password" [formField]="loginForm.password" />
```

</docs-step>

<docs-step title="Lier la case à cocher {#bind-the-checkbox-field}">
Ajoutez la directive `[formField]` à l'input de type checkbox :

```html
<input type="checkbox" [formField]="loginForm.rememberMe" />
```

</docs-step>

<docs-step title="Afficher les valeurs du formulaire {#display-the-form-values}">
Sous le formulaire se trouve une section de débogage qui affiche les valeurs actuelles du formulaire. Affichez la valeur de chaque champ avec `.value()` :

```angular-html
<p>Email: {{ loginForm.email().value() }}</p>
<p>Password: {{ loginForm.password().value() ? '••••••••' : '(empty)' }}</p>
<p>Remember me: {{ loginForm.rememberMe().value() ? 'Yes' : 'No' }}</p>
```

Les valeurs des champs du formulaire sont des signals : les valeurs affichées se mettent donc à jour automatiquement pendant la saisie.
</docs-step>

</docs-workflow>

Beau travail ! Vous avez connecté votre formulaire au template et affiché les valeurs du formulaire. La directive `[formField]` gère automatiquement le two-way binding des données : pendant la saisie, le signal `loginModel` se met à jour et les valeurs affichées changent immédiatement.

Ensuite, vous allez apprendre [à ajouter de la validation à votre formulaire](/tutorials/signal-forms/3-add-validation) !
