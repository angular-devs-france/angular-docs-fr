<!-- source: 9ff769ab17f2c013227f9dc24258b01d348d1446 -->
# Gestion de l'état des champs

L'état des champs des Signal Forms vous permet de réagir aux interactions de l'utilisateur en fournissant des signals réactifs pour le statut de validation (comme `valid`, `invalid`, `errors`), le suivi des interactions (comme `touched`, `dirty`) et la disponibilité (comme `disabled`, `hidden`).

## Comprendre l'état des champs {#understanding-field-state}

Lorsque vous créez un formulaire avec la fonction [`form()`](api/forms/signals/form), elle renvoie un **arbre de champs** (field tree) : une structure d'objets qui reproduit votre model de formulaire. Chaque champ de l'arbre est accessible avec la notation pointée (comme [`form.email`](api/forms/signals/FieldTree)).

### Accéder à l'état d'un champ {#accessing-field-state}

Lorsque vous appelez un champ de l'arbre comme une fonction (comme [`form.email()`](api/forms/signals/FieldState)), il renvoie un objet `FieldState` contenant des signals réactifs qui suivent l'état de validation, d'interaction et de disponibilité du champ. Par exemple, le signal `invalid()` vous indique si le champ comporte des erreurs de validation :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, required, email} from '@angular/forms/signals';

@Component({
  selector: 'app-registration',
  imports: [FormField],
  template: `
    <input type="email" [formField]="registrationForm.email" />

    @if (registrationForm.email().invalid()) {
      <p class="error">Email has validation errors:</p>
      <ul>
        @for (error of registrationForm.email().errors(); track error) {
          <li>{{ error.message }}</li>
        }
      </ul>
    }
  `,
})
export class Registration {
  registrationModel = signal({
    email: '',
    password: '',
  });

  registrationForm = form(this.registrationModel, (schemaPath) => {
    required(schemaPath.email, {message: 'Email is required'});
    email(schemaPath.email, {message: 'Enter a valid email address'});
  });
}
```

Dans cet exemple, le template vérifie `registrationForm.email().invalid()` pour déterminer s'il faut afficher un message d'erreur.

### Signals d'état des champs {#field-state-signals}

Le signal le plus utilisé est `value()`, un `WritableSignal` qui donne accès à la valeur courante du champ :

```ts
const emailValue = registrationForm.email().value();
console.log(emailValue); // Current email string
```

Au-delà de `value()`, l'état du champ comprend des signals pour la validation, le suivi des interactions et le contrôle de la disponibilité :

| Catégorie                               | Signal       | Description                                                                       |
| --------------------------------------- | ------------ | --------------------------------------------------------------------------------- |
| **[Validation](#validation-state)**     | `valid()`    | Le champ respecte toutes les règles de validation et n'a aucun validateur en attente |
|                                         | `invalid()`  | Le champ comporte des erreurs de validation                                       |
|                                         | `errors()`   | Tableau d'objets d'erreur de validation                                           |
|                                         | `pending()`  | Validation asynchrone en cours                                                    |
| **[Interaction](#interaction-state)**   | `touched()`  | L'utilisateur a donné puis retiré le focus au champ (s'il est interactif)         |
|                                         | `dirty()`    | L'utilisateur a modifié le champ (s'il est interactif), même si la valeur correspond à l'état initial |
| **[Disponibilité](#availability-state)** | `disabled()` | Le champ est désactivé et n'affecte pas l'état du formulaire parent              |
|                                         | `hidden()`   | Indique que le champ doit être masqué ; la visibilité dans le template se contrôle avec `@if` |
|                                         | `readonly()` | Le champ est en lecture seule et n'affecte pas l'état du formulaire parent        |

Ces signals vous permettent de créer des expériences de formulaire réactives qui s'adaptent au comportement de l'utilisateur. Les sections suivantes détaillent chaque catégorie.

## État de validation {#validation-state}

Les signals d'état de validation vous indiquent si un champ est valide et quelles erreurs il contient.

NOTE: Ce guide se concentre sur l'**utilisation** de l'état de validation dans vos templates et votre logique (comme la lecture de `valid()`, `invalid()`, `errors()` pour afficher un retour). Pour savoir comment **définir** des règles de validation et créer des validateurs personnalisés, consultez le [guide de validation](guide/forms/signals/validation).

### Vérifier la validité {#checking-validity}

Utilisez `valid()` et `invalid()` pour vérifier le statut de validation :

```angular-ts
@Component({
  template: `
    <input type="email" [formField]="loginForm.email" />

    @if (loginForm.email().invalid()) {
      <p class="error">Email is invalid</p>
    }
    @if (loginForm.email().valid()) {
      <p class="success">Email looks good</p>
    }
  `,
})
export class Login {
  loginModel = signal({email: '', password: ''});
  loginForm = form(this.loginModel);
}
```

| Signal      | Renvoie `true` quand                                            |
| ----------- | --------------------------------------------------------------- |
| `valid()`   | Le champ respecte toutes les règles de validation et n'a aucun validateur en attente |
| `invalid()` | Le champ comporte des erreurs de validation                     |

Pour vérifier la validité dans le code, utilisez `invalid()` plutôt que `!valid()` si vous voulez distinguer « a des erreurs » de « validation en attente ». En effet, `valid()` et `invalid()` peuvent tous deux valoir `false` en même temps lorsqu'une validation asynchrone est en attente : le champ n'est pas encore valide puisque la validation n'est pas terminée, et il n'est pas non plus invalide puisqu'aucune erreur n'a encore été trouvée.

### Lire les erreurs de validation {#reading-validation-errors}

Accédez au tableau des erreurs de validation avec `errors()`, ou à une erreur précise avec `getError()`. Chaque objet d'erreur contient :

| Propriété   | Description                                                     |
| ----------- | --------------------------------------------------------------- |
| `kind`      | La règle de validation qui a échoué (comme « required » ou « email ») |
| `message`   | Message d'erreur lisible, facultatif                            |
| `fieldTree` | Référence au `FieldTree` où l'erreur s'est produite             |

NOTE: La propriété `message` est facultative. Les validateurs peuvent fournir des messages d'erreur personnalisés, mais si aucun n'est précisé, vous devrez peut-être associer les valeurs `kind` des erreurs à vos propres messages.

Voici un exemple qui affiche dans votre template les erreurs à partir d'un tableau d'erreurs de validation :

```angular-ts
@Component({
  template: `
    <input type="email" [formField]="loginForm.email" />

    @if (loginForm.email().errors().length > 0) {
      <div class="errors">
        @for (error of loginForm.email().errors(); track error) {
          <p>{{ error.message }}</p>
        }
      </div>
    }
  `
})
```

Cette approche parcourt toutes les erreurs d'un champ et affiche chaque message d'erreur à l'utilisateur.

Voici un exemple qui affiche une erreur dans votre template avec `getError()` :

```angular-ts
@Component({
  template: `
    <input type="email" [formField]="loginForm.email" />

    @if (loginForm.email().getError('required'); as error) {
      <div class="errors">
        <p>{{ error.message }}</p>
      </div>
    }
  `
})
```

`getError()` accepte n'importe quelle chaîne comme `kind` d'erreur, et reconnaît les types d'erreur des validations intégrées (par exemple « required », « email », « minLength »).

### Validation en attente {#pending-validation}

Le signal `pending()` indique qu'une validation asynchrone est en cours :

```angular-ts
@Component({
  template: `
    <input type="email" [formField]="signupForm.email" />

    @if (signupForm.email().pending()) {
      <p>Checking if email is available...</p>
    }

    @if (signupForm.email().invalid() && !signupForm.email().pending()) {
      <p>Email is already taken</p>
    }
  `
})
```

Ce signal vous permet d'afficher des états de chargement pendant l'exécution de la validation asynchrone.

## État d'interaction {#interaction-state}

L'état d'interaction indique si l'utilisateur a interagi avec les champs, ce qui permet des patterns comme « afficher les erreurs seulement après que l'utilisateur a touché un champ ».

### État touché {#touched-state}

Le signal `touched()` indique si l'utilisateur a donné puis retiré le focus à un champ, ou si le champ a été marqué comme touché par programmation. Seuls les champs interactifs peuvent devenir touchés ; les champs masqués, désactivés et en lecture seule ne deviennent pas touchés à la suite d'interactions de l'utilisateur ni de `markAsTouched()`.

Lorsqu'une action au niveau d'une section doit révéler les erreurs de validation de cette section, appelez `markAsTouched()` sur le champ de la section. La valeur par défaut de `skipDescendants` est `false` : l'appel marque donc comme touchés le champ de la section et chacun de ses champs descendants.

Par exemple, un parcours de paiement peut valider la section de livraison avant de permettre à l'utilisateur de passer à l'étape suivante :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, required} from '@angular/forms/signals';

@Component({
  selector: 'app-checkout-shipping',
  imports: [FormField],
  template: `
    <label>
      Name
      <input [formField]="checkoutForm.shipping.name" />
    </label>
    @if (checkoutForm.shipping.name().touched() && checkoutForm.shipping.name().invalid()) {
      <p>{{ checkoutForm.shipping.name().errors()[0].message }}</p>
    }

    <label>
      Address
      <input [formField]="checkoutForm.shipping.address" />
    </label>
    @if (checkoutForm.shipping.address().touched() && checkoutForm.shipping.address().invalid()) {
      <p>{{ checkoutForm.shipping.address().errors()[0].message }}</p>
    }

    <button type="button" (click)="continueToPayment()">Continue</button>

    @if (showPayment() && checkoutForm.shipping().valid()) {
      <p>Ready for payment.</p>
    }
  `,
})
export class CheckoutShipping {
  checkoutModel = signal({
    shipping: {
      name: '',
      address: '',
    },
  });

  showPayment = signal(false);

  checkoutForm = form(this.checkoutModel, (schemaPath) => {
    required(schemaPath.shipping.name, {message: 'Enter a name'});
    required(schemaPath.shipping.address, {message: 'Enter an address'});
  });

  continueToPayment() {
    this.checkoutForm.shipping().markAsTouched();

    if (this.checkoutForm.shipping().invalid()) {
      return;
    }

    this.showPayment.set(true);
  }
}
```

Lorsque `continueToPayment()` appelle `markAsTouched()` sur `checkoutForm.shipping()`, il utilise le comportement par défaut `skipDescendants: false`. Angular marque `shipping`, `shipping.name` et `shipping.address` comme touchés : les messages d'erreur conditionnés par `touched() && invalid()` sur les champs enfants deviennent donc visibles avant la soumission du formulaire complet.

NOTE: Passez `{skipDescendants: true}` uniquement lorsque le champ qui reçoit l'appel doit devenir touché sans modifier l'état touché de ses descendants.

### État modifié (dirty) {#dirty-state}

Les formulaires doivent souvent détecter si les données ont réellement changé, par exemple pour avertir l'utilisateur de modifications non enregistrées ou pour n'activer un bouton d'enregistrement que lorsque c'est nécessaire. Le signal `dirty()` indique si l'utilisateur a modifié le champ.

Le signal `dirty()` passe à `true` lorsque l'utilisateur modifie la valeur d'un champ interactif, et reste à `true` même si la valeur est ramenée à sa valeur initiale :

```angular-ts
@Component({
  template: `
    <form novalidate>
      <input [formField]="profileForm.name" />
      <input [formField]="profileForm.bio" />

      @if (profileForm().dirty()) {
        <p class="warning">You have unsaved changes</p>
      }
    </form>
  `,
})
export class Profile {
  profileModel = signal({name: 'Alice', bio: 'Developer'});
  profileForm = form(this.profileModel);
}
```

Utilisez `dirty()` pour les avertissements de « modifications non enregistrées » ou pour n'activer les boutons d'enregistrement que lorsque les données ont changé.

### Touché ou modifié (dirty) {#touched-vs-dirty}

Ces signals suivent différents types d'état d'interaction :

| Signal      | Quand il passe à true                                                                                                           |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `touched()` | L'utilisateur a donné puis retiré le focus à un champ interactif, ou le champ a été marqué comme touché par programmation        |
| `dirty()`   | L'utilisateur a modifié un champ interactif (même s'il n'en a jamais retiré le focus, et même si la valeur courante correspond à la valeur initiale) |

Un champ peut se trouver dans différentes combinaisons :

| État                   | Scénario                                                  |
| ---------------------- | --------------------------------------------------------- |
| Touché mais non modifié | L'utilisateur a donné puis retiré le focus au champ sans rien modifier |
| Touché et modifié      | L'utilisateur a donné le focus au champ, modifié la valeur, puis retiré le focus |

NOTE: Les champs masqués, désactivés et en lecture seule ne sont pas interactifs : ils ne deviennent ni touchés ni modifiés à la suite d'interactions de l'utilisateur.

## État de disponibilité {#availability-state}

Les signals d'état de disponibilité déterminent si les champs sont interactifs, modifiables ou visibles. Les champs désactivés, masqués et en lecture seule ne sont pas interactifs. Ils n'influencent pas le fait que leur formulaire parent soit valide, touché ou modifié.

### Champs désactivés {#disabled-fields}

Le signal `disabled()` indique si un champ accepte les saisies de l'utilisateur. Les champs désactivés apparaissent dans l'interface, mais l'utilisateur ne peut pas interagir avec eux.

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, disabled} from '@angular/forms/signals';

@Component({
  selector: 'app-order',
  imports: [FormField],
  // TIP: The `[formField]` directive automatically binds the `disabled` attribute based
  // on the field's `disabled()` state, so you don't need to manually add `[disabled]="field().disabled()"`
  template: `
    <input [formField]="orderForm.couponCode" />

    @if (orderForm.couponCode().disabled()) {
      <p class="info">Coupon code is only available for orders over $50</p>
    }
  `,
})
export class Order {
  orderModel = signal({
    total: 25,
    couponCode: '',
  });

  orderForm = form(this.orderModel, (schemaPath) => {
    disabled(schemaPath.couponCode, {when: ({valueOf}) => valueOf(schemaPath.total) < 50});
  });
}
```

Dans cet exemple, nous utilisons `valueOf(schemaPath.total)` pour lire la valeur du champ `total` et déterminer si `couponCode` doit être désactivé.

NOTE: Le paramètre du callback de schéma (`schemaPath` dans ces exemples) est un objet `SchemaPathTree` qui fournit les chemins vers tous les champs de votre formulaire. Vous pouvez nommer ce paramètre comme vous le souhaitez.

Lorsque vous définissez des règles comme `disabled()`, `hidden()` ou `readonly()`, la fonction `when` reçoit un objet `FieldContext`, généralement déstructuré (comme `({valueOf})`). Deux méthodes couramment utilisées dans les règles de validation sont :

- `valueOf(schemaPath.otherField)` - Lit la valeur d'un autre champ du formulaire
- `value()` - Un signal contenant la valeur du champ auquel la règle s'applique

Les champs désactivés ne contribuent pas à l'état de validation du formulaire parent. Même si un champ désactivé était invalide, le formulaire parent peut quand même être valide. L'état `disabled()` affecte l'interactivité et la validation, mais ne modifie pas la valeur du champ.

### Champs masqués {#hidden-fields}

Le signal `hidden()` indique si un champ est masqué selon une condition. Utilisez `hidden()` avec `@if` pour afficher ou masquer des champs selon des conditions :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, hidden} from '@angular/forms/signals';

@Component({
  selector: 'app-profile',
  imports: [FormField],
  template: `
    <label>
      <input type="checkbox" [formField]="profileForm.isPublic" />
      Make profile public
    </label>

    @if (!profileForm.publicUrl().hidden()) {
      <label>
        Public URL
        <input [formField]="profileForm.publicUrl" />
      </label>
    }
  `,
})
export class Profile {
  profileModel = signal({
    isPublic: false,
    publicUrl: '',
  });

  profileForm = form(this.profileModel, (schemaPath) => {
    hidden(schemaPath.publicUrl, {when: ({valueOf}) => !valueOf(schemaPath.isPublic)});
  });
}
```

Les champs masqués ne participent pas à la validation. Si un champ obligatoire est masqué, il n'empêche pas la soumission du formulaire. L'état `hidden()` affecte la disponibilité et la validation, mais ne modifie pas la valeur du champ.

### Champs en lecture seule {#readonly-fields}

Le signal `readonly()` indique si un champ est en lecture seule. Les champs en lecture seule affichent leur valeur, mais l'utilisateur ne peut pas les modifier :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, readonly} from '@angular/forms/signals';

@Component({
  selector: 'app-account',
  imports: [FormField],
  template: `
    <label>
      Username (cannot be changed)
      <input [formField]="accountForm.username" />
    </label>

    <label>
      Email
      <input [formField]="accountForm.email" />
    </label>
  `,
})
export class Account {
  accountModel = signal({
    username: 'johndoe',
    email: 'john@example.com',
  });

  accountForm = form(this.accountModel, (schemaPath) => {
    readonly(schemaPath.username);
  });
}
```

NOTE: La directive `[formField]` lie automatiquement l'attribut `readonly` en fonction de l'état `readonly()` du champ : vous n'avez donc pas besoin d'ajouter manuellement `[readonly]="field().readonly()"`.

Comme les champs désactivés et masqués, les champs en lecture seule ne sont pas interactifs et n'affectent pas l'état du formulaire parent. L'état `readonly()` affecte la possibilité de modification et la validation, mais ne modifie pas la valeur du champ.

### Quand utiliser chaque état {#when-to-use-each}

| État         | À utiliser quand                                                    | Visible par l'utilisateur | Interaction possible | Pris en compte dans la validation |
| ------------ | ------------------------------------------------------------------- | --------------- | ----------------- | ------------------------- |
| `disabled()` | Le champ est temporairement indisponible (par exemple en fonction des valeurs d'autres champs) | Oui             | Non               | Non                       |
| `hidden()`   | Le champ n'est pas pertinent dans le contexte actuel                | Non (avec @if)  | Non               | Non                       |
| `readonly()` | La valeur doit être visible mais non modifiable                     | Oui             | Non               | Non                       |

## État au niveau du formulaire {#form-level-state}

Le formulaire racine est lui aussi un champ de l'arbre de champs. Lorsque vous l'appelez comme une fonction, il renvoie lui aussi un objet `FieldState` qui agrège l'état de tous les champs enfants.

### Accéder à l'état du formulaire {#accessing-form-state}

```angular-ts
@Component({
  template: `
    <form novalidate>
      <input [formField]="loginForm.email" />
      <input [formField]="loginForm.password" />

      <button [disabled]="!loginForm().valid()">Sign In</button>
    </form>
  `,
})
export class Login {
  loginModel = signal({email: '', password: ''});
  loginForm = form(this.loginModel);
}
```

Dans cet exemple, le formulaire n'est valide que lorsque tous les champs enfants sont valides. Cela vous permet d'activer ou de désactiver les boutons de soumission en fonction de la validité globale du formulaire.

### Signals au niveau du formulaire {#form-level-signals}

Comme le formulaire racine est un champ, il possède les mêmes signals (comme `valid()`, `invalid()`, `touched()`, `dirty()`, etc.).

| Signal      | Comportement au niveau du formulaire                           |
| ----------- | -------------------------------------------------------------- |
| `valid()`   | Tous les champs interactifs sont valides et aucun validateur n'est en attente |
| `invalid()` | Au moins un champ interactif comporte des erreurs de validation |
| `pending()` | Au moins un champ interactif a une validation asynchrone en attente |
| `touched()` | Le formulaire, ou au moins un descendant interactif, est touché |
| `dirty()`   | L'utilisateur a modifié au moins un champ interactif           |

### Quand utiliser le niveau formulaire ou le niveau champ {#when-to-use-form-level-vs-field-level}

**Utilisez l'état au niveau du formulaire pour :**

- L'état activé/désactivé du bouton de soumission
- L'état du bouton « Enregistrer »
- Les vérifications de validité globale du formulaire
- Les avertissements de modifications non enregistrées

**Utilisez l'état au niveau du champ pour :**

- Les messages d'erreur propres à chaque champ
- Les styles spécifiques à un champ
- Le retour de validation champ par champ
- La disponibilité conditionnelle des champs

## Propagation de l'état {#state-propagation}

L'état des champs se propage des champs enfants vers le formulaire racine, en passant par les groupes de champs parents.

### Comment l'état des enfants affecte les formulaires parents {#how-child-state-affects-parent-forms}

Lorsqu'un champ enfant devient invalide, son groupe de champs parent devient invalide, tout comme le formulaire racine. Lorsqu'un enfant devient touché ou modifié, le groupe de champs parent et le formulaire racine reflètent ce changement. Cette agrégation vous permet de vérifier la validité à n'importe quel niveau : un champ ou le formulaire entier.

```ts
const userModel = signal({
  profile: {
    firstName: '',
    lastName: '',
  },
  address: {
    street: '',
    city: '',
  },
});

const userForm = form(userModel);

// If firstName is invalid, profile is invalid
userForm.profile.firstName().invalid() === true;
// → userForm.profile().invalid() === true
// → userForm().invalid() === true
```

### Champs masqués, désactivés et en lecture seule {#hidden-disabled-and-readonly-fields}

Les champs masqués, désactivés et en lecture seule ne sont pas interactifs et n'affectent pas l'état du formulaire parent :

```ts
const orderModel = signal({
  customerName: '',
  requiresShipping: false,
  shippingAddress: '',
});

const orderForm = form(orderModel, (schemaPath) => {
  hidden(schemaPath.shippingAddress, {when: ({valueOf}) => !valueOf(schemaPath.requiresShipping)});
});
```

Dans cet exemple, lorsque `shippingAddress` est masqué, il n'affecte pas la validité du formulaire. Ainsi, même si `shippingAddress` est vide et obligatoire, le formulaire peut être valide.

Ce comportement empêche les champs masqués, désactivés ou en lecture seule de bloquer la soumission du formulaire ou d'affecter la validation et les états touché et modifié.

## Utiliser l'état dans les templates {#using-state-in-templates}

Les signals d'état des champs s'intègrent naturellement aux templates Angular et permettent des expériences de formulaire réactives sans gestion manuelle des événements.

### Affichage conditionnel des erreurs {#conditional-error-display}

Affichez les erreurs uniquement après que l'utilisateur a interagi avec un champ :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, email} from '@angular/forms/signals';

@Component({
  selector: 'app-signup',
  imports: [FormField],
  template: `
    <label>
      Email
      <input type="email" [formField]="signupForm.email" />
    </label>

    @if (signupForm.email().touched() && signupForm.email().invalid()) {
      <p class="error">{{ signupForm.email().errors()[0].message }}</p>
    }
  `,
})
export class Signup {
  signupModel = signal({email: '', password: ''});

  signupForm = form(this.signupModel, (schemaPath) => {
    email(schemaPath.email);
  });
}
```

Ce pattern évite d'afficher des erreurs avant que l'utilisateur ait eu l'occasion d'interagir avec le champ. Les erreurs n'apparaissent qu'une fois que l'utilisateur a donné le focus au champ puis l'a quitté.

### Disponibilité conditionnelle des champs {#conditional-field-availability}

Utilisez le signal `hidden()` avec `@if` pour afficher ou masquer des champs de façon conditionnelle :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, hidden} from '@angular/forms/signals';

@Component({
  selector: 'app-order',
  imports: [FormField],
  template: `
    <label>
      <input type="checkbox" [formField]="orderForm.requiresShipping" />
      Requires shipping
    </label>

    @if (!orderForm.shippingAddress().hidden()) {
      <label>
        Shipping Address
        <input [formField]="orderForm.shippingAddress" />
      </label>
    }
  `,
})
export class Order {
  orderModel = signal({
    requiresShipping: false,
    shippingAddress: '',
  });

  orderForm = form(this.orderModel, (schemaPath) => {
    hidden(schemaPath.shippingAddress, {
      when: ({valueOf}) => !valueOf(schemaPath.requiresShipping),
    });
  });
}
```

Les champs masqués ne participent pas à la validation, ce qui permet de soumettre le formulaire même si le champ masqué serait autrement invalide.

### Suivre les valeurs des champs de tableau {#tracking-values-for-array-fields}

Dans les Signal Forms, un bloc `@for` qui parcourt un ensemble de champs doit être suivi par l'identité des champs.

```angular-ts
@Component({
  imports: [FormField],
  template: `
    @for (field of form.emails; track field) {
      <input [formField]="field" />
    }
  `,
})
export class App {
  formModel = signal({emails: ['john.doe@mail.com', 'max.musterman@mail.com']});
  form = form(this.formModel);
}
```

Le système de formulaires suit déjà les valeurs du model dans le tableau et maintient automatiquement une identité stable pour les champs qu'il crée.

Lorsqu'un élément change, il peut représenter une nouvelle entité logique même si certaines de ses propriétés semblent identiques. Le suivi par identité garantit que le framework le traite comme un élément distinct au lieu de réutiliser des éléments d'interface existants. Cela évite que des éléments avec état, comme les champs de formulaire, soient partagés à tort et garde les bindings alignés sur la bonne partie du model.

## Utiliser l'état des champs dans la logique du composant {#using-field-state-in-component-logic}

Les signals d'état des champs fonctionnent avec les primitives réactives d'Angular comme `computed()` et `effect()` pour une logique de formulaire avancée.

### Vérifications de validation avant la soumission {#validation-checks-before-submission}

Vérifiez la validité du formulaire dans les méthodes du composant :

```ts
export class Registration {
  registrationModel = signal({
    username: '',
    email: '',
    password: '',
  });

  registrationForm = form(this.registrationModel);

  async onSubmit() {
    // Wait for any pending async validation
    if (this.registrationForm().pending()) {
      console.log('Waiting for validation...');
      return;
    }

    // Guard against invalid submissions
    if (this.registrationForm().invalid()) {
      console.error('Form is invalid');
      return;
    }

    const data = this.registrationModel();
    await this.api.register(data);
  }
}
```

Cela garantit que seules des données valides et entièrement validées atteignent votre API.

### État dérivé avec computed {#derived-state-with-computed}

Créez des signals computed basés sur l'état des champs pour qu'ils se mettent à jour automatiquement quand l'état sous-jacent des champs change :

```ts
export class Password {
  passwordModel = signal({password: '', confirmPassword: ''});
  passwordForm = form(this.passwordModel);

  // Compute password strength indicator
  passwordStrength = computed(() => {
    const password = this.passwordForm.password().value();
    if (password.length < 8) return 'weak';
    if (password.length < 12) return 'medium';
    return 'strong';
  });

  // Check if all required fields are filled
  allFieldsFilled = computed(() => {
    return (
      this.passwordForm.password().value().length > 0 &&
      this.passwordForm.confirmPassword().value().length > 0
    );
  });
}
```

### Modifier l'état par programmation {#programmatic-state-changes}

Si l'état des champs se met généralement à jour via les interactions de l'utilisateur (saisie, prise et perte du focus), vous devez parfois le contrôler par programmation. Les cas courants incluent la soumission et la réinitialisation des formulaires.

#### Soumission du formulaire {#form-submission}

Les Signal Forms fournissent une directive `FormRoot` qui simplifie la soumission des formulaires. Elle empêche automatiquement le comportement de soumission par défaut du navigateur et définit l'attribut `novalidate` sur l'élément `<form>`.

```angular-ts
import {FormField, FormRoot, form, required, email} from '@angular/forms/signals';
import {Component, signal} from '@angular/core';

@Component({
  imports: [FormRoot, FormField],
  template: `
    <form [formRoot]="registrationForm">
      <input [formField]="registrationForm.username" />
      <input type="email" [formField]="registrationForm.email" />
      <input type="password" [formField]="registrationForm.password" />

      <button type="submit">Register</button>
    </form>
  `,
})
export class Registration {
  registrationModel = signal({username: '', email: '', password: ''});

  registrationForm = form(
    this.registrationModel,
    (schemaPath) => {
      required(schemaPath.username);
      email(schemaPath.email);
      required(schemaPath.password);
    },
    {
      submission: {
        action: async () => this.submitToServer(),
      },
    },
  );

  private submitToServer() {
    // Send data to server
  }
}
```

Lorsque vous utilisez `FormRoot`, soumettre le formulaire appelle automatiquement la fonction `submit()`, qui marque tous les champs comme touchés (ce qui révèle les erreurs de validation) et exécute votre callback `action` si le formulaire est valide.

Vous pouvez aussi soumettre un formulaire manuellement, sans utiliser la directive, en appelant `submit(this.registrationForm)`. Lorsque vous appelez explicitement la fonction `submit` de cette façon, vous pouvez passer un `FormSubmitOptions` pour remplacer la logique de `submission` par défaut du formulaire : `submit(this.registrationForm, {action: () => /* ... */ })`.

#### Réinitialiser les formulaires après la soumission {#resetting-forms-after-submission}

Après avoir soumis un formulaire avec succès, vous pouvez vouloir le ramener à son état initial, en effaçant à la fois l'historique des interactions de l'utilisateur et les valeurs des champs. La méthode `reset()` réinitialise les indicateurs touché et modifié. Vous pouvez aussi passer une valeur facultative à `reset()` pour mettre à jour les données du model :

```ts
export class Contact {
  private readonly INITIAL_MODEL = {name: '', email: '', message: ''};
  contactModel = signal({...this.INITIAL_MODEL});
  contactForm = form(this.contactModel, {
    submission: {
      action: async (f) => {
        await this.api.sendMessage(this.contactModel());
        // Clear interaction state (touched, dirty) and reset to initial values
        f().reset({...this.INITIAL_MODEL});
      },
    },
  });
}
```

Le formulaire est ainsi prêt pour une nouvelle saisie, sans afficher de messages d'erreur obsolètes ni d'indicateurs d'état modifié.

## Appliquer des styles selon l'état de validation {#styling-based-on-validation-state}

Vous pouvez appliquer des styles personnalisés à votre formulaire en liant des classes CSS en fonction de l'état de validation :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, email} from '@angular/forms/signals';

@Component({
  imports: [FormField],
  template: `
    <input
      type="email"
      [formField]="form.email"
      [class.is-invalid]="form.email().touched() && form.email().invalid()"
      [class.is-valid]="form.email().touched() && form.email().valid()"
    />
  `,
  styles: `
    input.is-invalid {
      border: 2px solid red;
      background-color: white;
    }

    input.is-valid {
      border: 2px solid green;
    }
  `,
})
export class StyleExample {
  model = signal({email: ''});

  form = form(this.model, (schemaPath) => {
    email(schemaPath.email);
  });
}
```

Vérifier à la fois `touched()` et l'état de validation garantit que les styles n'apparaissent qu'après que l'utilisateur a interagi avec le champ.

## Donner le focus au contrôle de formulaire lié à un champ {#focus-a-form-control-bound-to-a-form-field}

Les Signal Forms d'Angular fournissent sur l'état du champ une méthode `focusBoundControl()` qui vous permet de déplacer par programmation le [focus](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) vers le contrôle de formulaire associé à un champ donné.

Un cas d'usage courant consiste à améliorer l'accessibilité lors de la soumission d'un formulaire : lorsque le formulaire est invalide, affichez les messages d'erreur et déplacez automatiquement le focus vers le premier champ invalide pour guider l'utilisateur dans sa correction.

### Utilisation de base {#basic-usage}

Soit un formulaire d'inscription :

```ts
@Component({/* ... */})
export class Registration {
  registrationModel = signal({username: '', email: '', password: ''});
  registrationForm = form(this.registrationModel, (schemaPath) => {
    required(schemaPath.username);
    email(schemaPath.email);
    required(schemaPath.password);
  });
}
```

Pour déplacer le focus vers le contrôle lié au champ `email` :

```ts
registrationForm.email().focusBoundControl();
```

### Empêcher le défilement {#preventing-scroll}

Si le contrôle ciblé est en dehors du viewport et que vous voulez lui donner le focus sans déclencher de défilement, vous pouvez définir l'option [preventScroll](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus#preventscroll) à `true` lors de l'appel à la méthode `focusBoundControl()`.

```ts
registrationForm.email().focusBoundControl({preventScroll: true});
```

### Donner le focus au premier champ invalide lors de la soumission {#focusing-the-first-invalid-field-on-submission}

Utilisez `errorSummary()` pour trouver le premier champ invalide et lui donner le focus lorsque l'utilisateur soumet le formulaire avec des erreurs :

```ts
onSubmit() {
  const firstError = this.registrationForm().errorSummary()[0];
  if (firstError?.fieldTree) {
    firstError.fieldTree().focusBoundControl();
  } else {
    // proceed with submission
  }
}
```

### Contrôles personnalisés {#custom-controls}

Par défaut, appeler `focusBoundControl()` sur un contrôle personnalisé n'a aucun effet, car un contrôle personnalisé peut contenir plusieurs champs natifs. Par exemple, un sélecteur de date peut contenir des champs distincts pour le jour, le mois et l'année. Angular ne peut donc pas déterminer quel élément doit recevoir le focus ni quelle action effectuer.

Pour prendre en charge le focus par programmation dans un contrôle personnalisé, implémentez une méthode `focus()`. Lorsque `focusBoundControl()` est appelée sur l'état du champ associé à un contrôle personnalisé, Angular appelle la méthode `focus()` du contrôle si elle existe.

Prenons un champ de mot de passe personnalisé :

```html
<div class="password-block">
  <input type="password" #passwordCtrl [value]="value()" (input)="value.set($event.target.value)" />
</div>
```

```ts
@Component({/* ... */})
export class PasswordInput implements FormValueControl<string> {
  readonly value = model<string>('');
  readonly passwordCtrl = viewChild.required<ElementRef<HTMLInputElement>>('passwordCtrl');

  // Called automatically when focusBoundControl() is invoked
  // on the field state associated with this custom control
  focus(): void {
    this.passwordCtrl().nativeElement.focus();
  }
}
```

## Prochaines étapes {#next-steps}

Ce guide a couvert la gestion des statuts de validation et de disponibilité, le suivi des interactions et la propagation de l'état des champs. D'autres guides explorent d'autres aspects des Signal Forms :

<docs-pill-row>
  <docs-pill href="guide/forms/signals/models" title="Models de formulaire" />
  <docs-pill href="guide/forms/signals/validation" title="Validation" />
  <docs-pill href="guide/forms/signals/custom-controls" title="Contrôles personnalisés" />
  <!-- <docs-pill href="guide/forms/signals/arrays" title="Working with Arrays" /> -->
</docs-pill-row>
