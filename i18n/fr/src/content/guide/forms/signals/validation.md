<!-- source: c094c0ea6188cf5821b992732ad0a65fbcfb0fd6 -->
# Validation

Les formulaires ont besoin de validation pour garantir que les utilisateurs fournissent des données correctes et complètes avant la soumission. Sans validation, vous devriez gérer les problèmes de qualité des données côté serveur, offrir une mauvaise expérience utilisateur avec des messages d'erreur peu clairs et vérifier manuellement chaque contrainte.

Les Signal Forms proposent une approche de validation basée sur les schémas. Les règles de validation sont liées aux champs via une fonction de schéma, s'exécutent automatiquement quand les valeurs changent et exposent les erreurs via les signals d'état des champs. Cela permet une validation réactive qui se met à jour au fil des interactions de l'utilisateur avec le formulaire.

<docs-code-multifile preview hideCode path="adev/src/content/examples/signal-forms/src/login-validation-complete/app/app.ts">
  <docs-code header="app.ts" path="adev/src/content/examples/signal-forms/src/login-validation-complete/app/app.ts"/>
  <docs-code header="app.html" path="adev/src/content/examples/signal-forms/src/login-validation-complete/app/app.html"/>
  <docs-code header="app.css" path="adev/src/content/examples/signal-forms/src/login-validation-complete/app/app.css"/>
</docs-code-multifile>

## Bases de la validation {#validation-basics}

Dans les Signal Forms, la validation est définie par une fonction de schéma passée comme second argument à `form()`.

### La fonction de schéma {#the-schema-function}

La fonction de schéma reçoit un objet `SchemaPathTree` qui vous permet de définir vos règles de validation :

<docs-code
  header="app.ts"
  path="adev/src/content/examples/signal-forms/src/login-validation-complete/app/app.ts"
  visibleLines="[29,30,31,32,33,34]"
  highlight="[30,31,33]"
/>

La fonction de schéma s'exécute une seule fois lors de l'initialisation du formulaire. Les règles de validation sont liées aux champs via le paramètre de schema path (comme `schemaPath.email`, `schemaPath.password`), et la validation s'exécute automatiquement chaque fois que les valeurs des champs changent.

NOTE: Le paramètre du callback de schéma (`schemaPath` dans ces exemples) est un objet `SchemaPathTree` qui fournit les chemins vers tous les champs de votre formulaire. Vous pouvez nommer ce paramètre comme vous le souhaitez.

### Fonctionnement de la validation {#how-validation-works}

Dans les Signal Forms, la validation suit ce schéma :

1. **Définir les règles de validation dans le schéma** - Liez les règles de validation aux champs dans la fonction de schéma
2. **Exécution automatique** - Les règles de validation s'exécutent quand les valeurs des champs changent
3. **Propagation des erreurs** - Les erreurs de validation sont exposées via les signals d'état des champs
4. **Mises à jour réactives** - L'interface se met à jour automatiquement quand l'état de validation change

La validation s'exécute à chaque changement de valeur pour les champs interactifs. Les champs masqués et désactivés ne sont pas validés : leurs règles de validation sont ignorées jusqu'à ce que le champ redevienne interactif.

### Moment de la validation {#validation-timing}

Les règles de validation s'exécutent dans cet ordre :

1. **Validation synchrone** - Toutes les règles de validation synchrones s'exécutent quand la valeur change
2. **Validation asynchrone** - Les règles de validation asynchrones ne s'exécutent qu'une fois toutes les règles synchrones satisfaites
3. **Mise à jour de l'état du champ** - Les signals `valid()`, `invalid()`, `errors()` et `pending()` se mettent à jour

Les règles de validation synchrones (comme `required()`, `email()`) se terminent immédiatement. Les règles de validation asynchrones (comme `validateHttp()`) peuvent prendre du temps et passent le signal `pending()` à `true` pendant leur exécution.

Toutes les règles de validation s'exécutent à chaque changement : la validation ne s'interrompt pas après la première erreur. Si un champ a à la fois les règles de validation `required()` et `email()`, les deux s'exécutent, et les deux peuvent produire des erreurs simultanément.

### Validation HTML native {#native-html-validation}

Les Signal Forms **n'utilisent pas** la [validation par contraintes](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Constraint_validation) intégrée au navigateur pour exécuter les règles de validation. C'est un choix de conception : les Signal Forms permettent à n'importe quel composant d'agir comme un contrôle de formulaire, et pas seulement aux éléments de formulaire natifs.

À la place, vos règles de validation s'exécutent entièrement dans Angular, et l'état de validation est exposé via les signals d'état des champs comme `valid()`, `invalid()` et `errors()` plutôt que via l'état de validité natif de l'élément. La [directive `FormRoot`](guide/forms/signals/form-submission) définit aussi l'attribut `novalidate` sur l'élément de formulaire pour que le navigateur ne bloque pas la soumission.

Lorsqu'un champ est lié à un élément de formulaire natif, certaines règles de validation intégrées reportent leur contrainte sur l'attribut natif correspondant : `required()`, `min()`, `max()`, `minLength()` et `maxLength()` définissent les attributs `required`, `min`, `max`, `minlength` et `maxlength` quand l'élément les prend en charge. `pattern()` fait exception : elle ne définit pas l'attribut natif `pattern`. Les Signal Forms définissent ces attributs pour contrôler le comportement de saisie et améliorer l'accessibilité, **pas pour déclencher la validation native**.

NOTE: Il existe un cas où les Signal Forms lisent l'état de validité natif : lorsque le navigateur ne parvient pas à analyser la valeur d'un champ natif (par exemple, une date partiellement saisie), les Signal Forms la signalent comme une erreur d'analyse. Même dans ce cas, l'erreur remonte via le signal `errors()` du champ comme toute autre erreur de validation. Consultez [Transformation de valeur](guide/forms/signals/custom-controls#value-transformation) pour plus de détails.

IMPORTANT: Ne vous fiez pas aux fonctionnalités de validité natives comme les pseudo-classes CSS `:valid` et `:invalid`, la propriété `validity` de l'élément ou `validationMessage`. Le navigateur peut signaler certains champs comme invalides en tant qu'effet de bord des attributs reportés (par exemple, un champ numérique dont la valeur est inférieure à son `min`), mais ce comportement varie selon la règle de validation et le type de champ, et ce n'est pas un moyen pris en charge d'observer l'état de validation.

Pour styliser les contrôles selon l'état de validation, liez des classes aux signals d'état des champs (voir [Gestion de l'état des champs](guide/forms/signals/field-state-management#conditional-error-display)) ou configurez les classes de statut automatiques (voir [Classes de statut automatiques](guide/forms/signals/migration#automatic-status-classes)).

## Règles de validation intégrées {#built-in-validation-rules}

Les Signal Forms fournissent des règles de validation pour les scénarios de validation courants. Toutes les règles de validation intégrées acceptent un objet d'options pour personnaliser les messages d'erreur et ajouter une logique conditionnelle.

### required() {#required}

La règle de validation `required()` garantit qu'un champ a une valeur :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, required} from '@angular/forms/signals';

@Component({
  selector: 'app-registration',
  imports: [FormField],
  template: `
    <form novalidate>
      <label>
        Username
        <input [formField]="registrationForm.username" />
      </label>

      <label>
        Email
        <input type="email" [formField]="registrationForm.email" />
      </label>

      <button type="submit">Register</button>
    </form>
  `,
})
export class RegistrationComponent {
  registrationModel = signal({
    username: '',
    email: '',
  });

  registrationForm = form(this.registrationModel, (schemaPath) => {
    required(schemaPath.username, {message: 'Username is required'});
    required(schemaPath.email, {message: 'Email is required'});
  });
}
```

Un champ est considéré comme « vide » lorsque sa valeur fait partie des suivantes, et comme non vide pour toute autre
valeur, y compris `0` et le tableau vide `[]` :

| Condition                    | Exemple     |
| ------------------------ | ----------- |
| La valeur est `null`         | `null`      |
| La valeur est `undefined`    | `undefined` |
| La valeur est une chaîne vide | `''`        |
| La valeur est `false`        | `false`     |
| La valeur est `NaN`          | `NaN`       |

Les deux derniers méritent une explication :

- `false` est vide pour suivre la sémantique native de `required` sur `<input type="checkbox">`, où
  une case non cochée échoue à la validation.
- `NaN` est vide parce qu'il résulte généralement d'une erreur d'analyse et n'est pas un nombre valide.

Pour les exigences conditionnelles, utilisez l'option `when` :

```ts
registrationForm = form(this.registrationModel, (schemaPath) => {
  required(schemaPath.promoCode, {
    message: 'Promo code is required for discounts',
    when: ({valueOf}) => valueOf(schemaPath.applyDiscount),
  });
});
```

La règle de validation ne s'exécute que lorsque la fonction `when` renvoie `true`.

NOTE: `required` considère un tableau vide comme présent (valide) ; utilisez donc [`minLength()`](#minlength-and-maxlength) pour imposer un nombre minimum d'éléments dans un tableau.

### email() {#email}

La règle de validation `email()` vérifie que le format de l'adresse e-mail est valide :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, email} from '@angular/forms/signals';

@Component({
  selector: 'app-contact',
  imports: [FormField],
  template: `
    <form novalidate>
      <label>
        Your Email
        <input type="email" [formField]="contactForm.email" />
      </label>
    </form>
  `,
})
export class ContactComponent {
  contactModel = signal({email: ''});

  contactForm = form(this.contactModel, (schemaPath) => {
    email(schemaPath.email, {message: 'Please enter a valid email address'});
  });
}
```

La règle de validation `email()` utilise une expression régulière standard de format d'e-mail. Elle accepte des adresses comme `user@example.com` mais rejette les adresses mal formées comme `user@` ou `@example.com`.

### min() et max() {#min-and-max}

Les règles de validation `min()` et `max()` fonctionnent avec des valeurs numériques :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, min, max} from '@angular/forms/signals';

@Component({
  selector: 'app-age-form',
  imports: [FormField],
  template: `
    <form novalidate>
      <label>
        Age
        <input type="number" [formField]="ageForm.age" />
      </label>

      <label>
        Rating (1-5)
        <input type="number" [formField]="ageForm.rating" />
      </label>
    </form>
  `,
})
export class AgeFormComponent {
  ageModel = signal({
    age: 0,
    rating: 0,
  });

  ageForm = form(this.ageModel, (schemaPath) => {
    min(schemaPath.age, 18, {message: 'You must be at least 18 years old'});
    max(schemaPath.age, 120, {message: 'Please enter a valid age'});

    min(schemaPath.rating, 1, {message: 'Rating must be at least 1'});
    max(schemaPath.rating, 5, {message: 'Rating cannot exceed 5'});
  });
}
```

Vous pouvez utiliser des valeurs calculées (computed) pour des contraintes dynamiques :

```ts
ageForm = form(this.ageModel, (schemaPath) => {
  min(schemaPath.participants, () => this.minimumRequired(), {
    message: 'Not enough participants',
  });
});
```

### minDate() et maxDate() {#mindate-and-maxdate}

Les règles de validation `minDate()` et `maxDate()` fonctionnent avec des valeurs `Date`, comme un champ lié à `<input type="date">`. Elles définissent aussi les attributs `min` et `max` de ce champ.

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, minDate, maxDate} from '@angular/forms/signals';

@Component({
  selector: 'app-booking-form',
  imports: [FormField],
  template: `
    <form novalidate>
      <label>
        Check-in
        <input type="date" [formField]="bookingForm.checkIn" />
      </label>
    </form>
  `,
})
export class BookingFormComponent {
  bookingModel = signal<{checkIn: Date | null}>({checkIn: null});

  bookingForm = form(this.bookingModel, (schemaPath) => {
    minDate(schemaPath.checkIn, new Date('2026-06-01'), {message: 'Bookings open on June 1'});
    maxDate(schemaPath.checkIn, new Date('2026-08-31'), {message: 'Bookings close on August 31'});
  });
}
```

IMPORTANT: Un `<input type="date">` lit et écrit des valeurs `Date` à minuit UTC, quel que soit le fuseau horaire de l'utilisateur, car c'est ainsi que fonctionne `valueAsDate` dans le navigateur. Construisez les limites de la même manière, par exemple avec `new Date('2026-06-01')` ou `new Date(Date.UTC(2026, 5, 1))`. Une `Date` créée à partir de l'heure locale, comme `new Date(2026, 5, 1)` ou `new Date()`, est décalée du décalage UTC de l'utilisateur. Selon le fuseau horaire, l'attribut `min` ou `max` peut alors être décalé d'un jour, et la date limite elle-même peut être rejetée.

Pour exiger une date qui ne soit pas antérieure à aujourd'hui, utilisez la date calendaire du jour dans le fuseau horaire de l'utilisateur, exprimée à minuit UTC :

```ts
const now = new Date();
const today = new Date(Date.UTC(now.getFullYear(), now.getMonth(), now.getDate()));

minDate(schemaPath.checkIn, today, {message: 'Check-in cannot be in the past'});
```

Passer directement `new Date()` peut rejeter la date du jour, car la valeur choisie dans le champ est minuit UTC, ce qui est antérieur à l'heure actuelle. Cela se produit en UTC et dans les fuseaux horaires en retard sur UTC.

### minLength() et maxLength() {#minlength-and-maxlength}

Les règles de validation `minLength()` et `maxLength()` fonctionnent avec les chaînes et les tableaux :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, minLength, maxLength} from '@angular/forms/signals';

@Component({
  selector: 'app-password-form',
  imports: [FormField],
  template: `
    <form novalidate>
      <label>
        Password
        <input type="password" [formField]="passwordForm.password" />
      </label>

      <label>
        Bio
        <textarea [formField]="passwordForm.bio"></textarea>
      </label>
    </form>
  `,
})
export class PasswordFormComponent {
  passwordModel = signal({
    password: '',
    bio: '',
  });

  passwordForm = form(this.passwordModel, (schemaPath) => {
    minLength(schemaPath.password, 8, {message: 'Password must be at least 8 characters'});
    maxLength(schemaPath.password, 100, {message: 'Password is too long'});

    maxLength(schemaPath.bio, 500, {message: 'Bio cannot exceed 500 characters'});
  });
}
```

Pour les chaînes, la « longueur » désigne le nombre de caractères. Pour les tableaux, la « longueur » désigne le nombre d'éléments.

### pattern() {#pattern}

La règle de validation `pattern()` valide la valeur par rapport à une expression régulière :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, pattern} from '@angular/forms/signals';

@Component({
  selector: 'app-phone-form',
  imports: [FormField],
  template: `
    <form novalidate>
      <label>
        Phone Number
        <input [formField]="phoneForm.phone" placeholder="555-123-4567" />
      </label>

      <label>
        Postal Code
        <input [formField]="phoneForm.postalCode" placeholder="12345" />
      </label>
    </form>
  `,
})
export class PhoneFormComponent {
  phoneModel = signal({
    phone: '',
    postalCode: '',
  });

  phoneForm = form(this.phoneModel, (schemaPath) => {
    pattern(schemaPath.phone, /^\d{3}-\d{3}-\d{4}$/, {
      message: 'Phone must be in format: 555-123-4567',
    });

    pattern(schemaPath.postalCode, /^\d{5}$/, {
      message: 'Postal code must be 5 digits',
    });
  });
}
```

Patterns courants :

| Type de pattern       | Expression régulière    | Exemple      |
| ---------------- | ----------------------- | ------------ |
| Téléphone             | `/^\d{3}-\d{3}-\d{4}$/` | 555-123-4567 |
| Code postal (US)      | `/^\d{5}$/`             | 12345        |
| Alphanumérique        | `/^[a-zA-Z0-9]+$/`      | abc123       |
| Compatible URL        | `/^[a-zA-Z0-9_-]+$/`    | my-url_123   |

## Validation des éléments de tableau {#validation-of-array-items}

Les formulaires peuvent contenir des tableaux d'objets imbriqués (par exemple, une liste d'articles de commande). Pour appliquer des règles de validation à chaque élément d'un tableau, utilisez `applyEach()` dans votre fonction de schéma. `applyEach()` parcourt le chemin du tableau et fournit un chemin pour chaque élément, auquel vous pouvez appliquer des validateurs comme pour les champs de premier niveau.

```ts
import {Component, signal} from '@angular/core';
import {applyEach, FormField, form, min, required, SchemaPathTree} from '@angular/forms/signals';

type Item = {name: string; quantity: number};

interface Order {
  title: string;
  description: string;
  items: Item[];
}

function ItemSchema(item: SchemaPathTree<Item>) {
  required(item.name, {message: 'Item name is required'});
  min(item.quantity, 1, {message: 'Quantity must be at least 1'});
}

@Component(/* ... */)
export class OrderComponent {
  orderModel = signal<Order>({
    title: '',
    description: '',
    items: [{name: '', quantity: 0}],
  });

  orderForm = form(this.orderModel, (schemaPath) => {
    required(schemaPath.title);
    required(schemaPath.description);

    applyEach(schemaPath.items, ItemSchema);
  });
}
```

## Erreurs de validation {#validation-errors}

Lorsque des règles de validation échouent, elles produisent des objets d'erreur qui décrivent le problème. Comprendre la structure des erreurs vous aide à fournir un retour clair aux utilisateurs.

NOTE: Cette section couvre les erreurs produites par les règles de validation. Pour afficher et utiliser les erreurs de validation dans votre interface, consultez le [guide Gestion de l'état des champs](guide/forms/signals/field-state-management).

### Structure des erreurs {#error-structure}

Chaque objet d'erreur de validation contient ces propriétés :

| Propriété | Description                                                                      |
| --------- | ------------------------------------------------------------------------ |
| `kind`    | La règle de validation qui a échoué (par ex. « required », « email », « minLength ») |
| `message` | Message d'erreur lisible, optionnel                                              |

Les règles de validation intégrées définissent automatiquement la propriété `kind`. La propriété `message` est optionnelle : vous pouvez fournir des messages personnalisés via les options de la règle de validation.

### Messages d'erreur personnalisés {#custom-error-messages}

Toutes les règles de validation intégrées acceptent une option `message` pour personnaliser le texte de l'erreur :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, required, minLength} from '@angular/forms/signals';

@Component({
  selector: 'app-signup',
  imports: [FormField],
  template: `
    <form novalidate>
      <label>
        Username
        <input [formField]="signupForm.username" />
      </label>

      <label>
        Password
        <input type="password" [formField]="signupForm.password" />
      </label>
    </form>
  `,
})
export class SignupComponent {
  signupModel = signal({
    username: '',
    password: '',
  });

  signupForm = form(this.signupModel, (schemaPath) => {
    required(schemaPath.username, {
      message: 'Please choose a username',
    });

    required(schemaPath.password, {
      message: 'Password cannot be empty',
    });
    minLength(schemaPath.password, 12, {
      message: 'Password must be at least 12 characters for security',
    });
  });
}
```

Les messages personnalisés doivent être clairs, précis et indiquer aux utilisateurs comment corriger le problème. Au lieu de « Invalid input », utilisez « Password must be at least 12 characters for security ».

### Plusieurs erreurs par champ {#multiple-errors-per-field}

Lorsqu'un champ a plusieurs règles de validation, chacune s'exécute indépendamment et peut produire une erreur :

```ts
signupForm = form(this.signupModel, (schemaPath) => {
  required(schemaPath.email, {message: 'Email is required'});
  email(schemaPath.email, {message: 'Enter a valid email address'});
  minLength(schemaPath.email, 5, {message: 'Email is too short'});
});
```

Si le champ e-mail est vide, seule l'erreur `required()` apparaît. Si l'utilisateur saisit « a@b », les erreurs `email()` et `minLength()` apparaissent toutes les deux. Toutes les règles de validation s'exécutent : la validation ne s'arrête pas après le premier échec.

TIP: Utilisez le pattern `touched() && invalid()` dans vos templates pour éviter que les erreurs n'apparaissent avant que l'utilisateur n'ait interagi avec un champ. Pour des conseils complets sur l'affichage des erreurs de validation, consultez le [guide Gestion de l'état des champs](guide/forms/signals/field-state-management#conditional-error-display).

## Règles de validation personnalisées {#custom-validation-rules}

Si les règles de validation intégrées couvrent les cas courants, vous aurez souvent besoin d'une logique de validation personnalisée pour des règles métier, des formats complexes ou des contraintes propres à votre domaine.

### Utiliser validate() {#using-validate}

La fonction `validate()` crée des règles de validation personnalisées. Elle reçoit une fonction de validation qui accède au contexte du champ et renvoie :

| Valeur renvoyée       | Signification       |
| --------------------- | ---------------- |
| Objet d'erreur        | La valeur est invalide |
| `null` ou `undefined` | La valeur est valide   |

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, validate} from '@angular/forms/signals';

@Component({
  selector: 'app-url-form',
  imports: [FormField],
  template: `
    <form novalidate>
      <label>
        Website URL
        <input [formField]="urlForm.website" />
      </label>
    </form>
  `,
})
export class UrlFormComponent {
  urlModel = signal({website: ''});

  urlForm = form(this.urlModel, (schemaPath) => {
    validate(schemaPath.website, ({value}) => {
      if (!value().startsWith('https://')) {
        return {
          kind: 'https',
          message: 'URL must start with https://',
        };
      }

      return null;
    });
  });
}
```

La fonction de validation reçoit un objet `FieldContext` contenant :

| Propriété       | Type       | Description                                          |
| --------------- | ---------- | ------------------------------------------- |
| `value`         | Signal     | Signal contenant la valeur courante du champ         |
| `state`         | FieldState | La référence à l'état du champ                       |
| `fieldTree`     | FieldTree  | La référence à l'arbre de champs                     |
| `valueOf()`     | Méthode    | Récupère la valeur d'un autre champ par son chemin   |
| `stateOf()`     | Méthode    | Récupère l'état d'un autre champ par son chemin      |
| `fieldTreeOf()` | Méthode    | Récupère l'arbre de champs d'un autre champ par son chemin |
| `pathKeys`      | Signal     | Clés du chemin depuis la racine jusqu'au champ courant |

NOTE: Les champs enfants ont aussi un signal `key`, et les champs d'éléments de tableau ont à la fois les signals `key` et `index`.

Renvoyez un objet d'erreur avec `kind` et `message` lorsque la validation échoue. Renvoyez `null` ou `undefined` lorsque la validation réussit.

### Utiliser validateTree() {#using-validatetree}

La fonction `validateTree()` crée des règles de validation personnalisées qui peuvent cibler plusieurs champs ou fournir une logique de validation complexe pour tout un sous-arbre.

```angular-ts
import {Component, model} from '@angular/core';
import {form, FormField, validateTree} from '@angular/forms/signals';

interface User {
  firstName: string;
  lastName: string;
}

@Component({/* ... */})
export class UserFormComponent {
  readonly userModel = model<User>({
    firstName: '',
    lastName: '',
  });

  userForm = form(this.userModel, (path) => {
    validateTree(path, (ctx) => {
      if (ctx.valueOf(path.firstName).length < 5) {
        return {
          kind: 'minLength5',
          message: 'First name must be at least 5 characters',
          fieldTree: ctx.fieldTree.lastName,
        };
      }

      return null;
    });
  });
}
```

La fonction de validation de `validateTree()` reçoit le même objet `FieldContext` que `validate()`.

### Règles de validation réutilisables {#reusable-validation-rules}

Créez des fonctions de règle de validation réutilisables en enveloppant `validate()` :

```ts
function url(path: SchemaPath<string>, options?: {message?: string}) {
  validate(path, ({value}) => {
    try {
      new URL(value());
      return null;
    } catch {
      return {
        kind: 'url',
        message: options?.message || 'Enter a valid URL',
      };
    }
  });
}

function phoneNumber(path: SchemaPath<string>, options?: {message?: string}) {
  validate(path, ({value}) => {
    const phoneRegex = /^\d{3}-\d{3}-\d{4}$/;

    if (!phoneRegex.test(value())) {
      return {
        kind: 'phoneNumber',
        message: options?.message || 'Phone must be in format: 555-123-4567',
      };
    }

    return null;
  });
}
```

Vous pouvez utiliser les règles de validation personnalisées comme les règles de validation intégrées :

```ts
urlForm = form(this.urlModel, (schemaPath) => {
  url(schemaPath.website, {message: 'Please enter a valid website URL'});
  phoneNumber(schemaPath.phone);
});
```

## Validation inter-champs {#cross-field-validation}

La validation inter-champs compare ou met en relation les valeurs de plusieurs champs.

La confirmation du mot de passe est un scénario courant de validation inter-champs :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, required, minLength, validate} from '@angular/forms/signals';

@Component({
  selector: 'app-password-change',
  imports: [FormField],
  template: `
    <form novalidate>
      <label>
        New Password
        <input type="password" [formField]="passwordForm.password" />
      </label>

      <label>
        Confirm Password
        <input type="password" [formField]="passwordForm.confirmPassword" />
      </label>

      <button type="submit">Change Password</button>
    </form>
  `,
})
export class PasswordChangeComponent {
  passwordModel = signal({
    password: '',
    confirmPassword: '',
  });

  passwordForm = form(this.passwordModel, (schemaPath) => {
    required(schemaPath.password, {message: 'Password is required'});
    minLength(schemaPath.password, 8, {message: 'Password must be at least 8 characters'});

    required(schemaPath.confirmPassword, {message: 'Please confirm your password'});

    validate(schemaPath.confirmPassword, ({value, valueOf}) => {
      const confirmPassword = value();
      const password = valueOf(schemaPath.password);

      if (confirmPassword !== password) {
        return {
          kind: 'passwordMismatch',
          message: 'Passwords do not match',
        };
      }

      return null;
    });
  });
}
```

La règle de validation de confirmation accède à la valeur du champ de mot de passe avec `valueOf(schemaPath.password)` et la compare à la valeur de confirmation. Cette règle de validation s'exécute de manière réactive : si l'un des deux mots de passe change, la validation se relance automatiquement.

## Validation conditionnelle {#conditional-validation}

Parfois, une règle de validation ne doit s'appliquer que dans certaines conditions, par exemple pour valider une adresse de livraison uniquement quand une commande est expédiée à l'international, ou pour appliquer un ensemble de règles différent à chaque variante d'un champ typé par une union.

Comme les règles de validation se trouvent dans la fonction de schéma, vous les appliquez de manière conditionnelle avec les mêmes fonctions structurelles que celles qui composent les schémas :

- Utilisez [`applyWhen()`](guide/forms/signals/form-logic#conditional-logic-with-applywhen) pour activer un groupe de règles en fonction de l'état réactif du formulaire, y compris des valeurs d'autres champs.
- Utilisez [`applyWhenValue()`](guide/forms/signals/schemas#type-narrowing-with-applywhenvalue) pour appliquer des règles en fonction de la valeur du champ lui-même. Lorsque le prédicat est un type guard, les règles sont typées selon la valeur affinée, ce qui en fait la méthode recommandée pour valider les unions discriminées et les autres types à variantes.

Pour des exemples complets, notamment avec des schémas réutilisables et des unions discriminées, consultez le [guide Schémas et composabilité des schémas](guide/forms/signals/schemas).

## Validation asynchrone {#async-validation}

La validation asynchrone gère les validations qui nécessitent des sources de données externes, comme vérifier la disponibilité d'un nom d'utilisateur sur un serveur ou valider auprès d'une API.

### Utiliser validateHttp() {#using-validatehttp}

La fonction `validateHttp()` effectue une validation basée sur HTTP :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, required, validateHttp} from '@angular/forms/signals';

@Component({
  selector: 'app-username-form',
  imports: [FormField],
  template: `
    <form novalidate>
      <label>
        Username
        <input [formField]="usernameForm.username" />

        @if (usernameForm.username().pending()) {
          <span class="checking">Checking availability...</span>
        }
      </label>
    </form>
  `,
})
export class UsernameFormComponent {
  usernameModel = signal({username: ''});

  usernameForm = form(this.usernameModel, (schemaPath) => {
    required(schemaPath.username, {message: 'Username is required'});

    validateHttp(schemaPath.username, {
      request: ({value}) => `/api/check-username?username=${value()}`,
      onSuccess: (response: any) => {
        if (response.taken) {
          return {
            kind: 'usernameTaken',
            message: 'Username is already taken',
          };
        }
        return null;
      },
      onError: (error) => ({
        kind: 'networkError',
        message: 'Could not verify username availability',
      }),
    });
  });
}
```

La règle de validation `validateHttp()` :

1. Appelle l'URL ou la requête renvoyée par la fonction `request`
2. Convertit la réponse réussie en erreur de validation ou en `null` avec `onSuccess`
3. Gère les échecs de requête (erreurs réseau, erreurs HTTP) avec `onError`
4. Passe `pending()` à `true` pendant que la requête est en cours
5. Ne s'exécute qu'une fois toutes les règles de validation synchrones satisfaites

### État en attente {#pending-state}

Pendant l'exécution de la validation asynchrone, le signal `pending()` du champ renvoie `true`. Utilisez-le pour afficher des indicateurs de chargement :

```angular-html
@if (form.username().pending()) {
  <span class="spinner">Checking...</span>
}
```

Le signal `valid()` renvoie `false` tant que la validation est en attente, même s'il n'y a encore aucune erreur. Le signal `invalid()` ne renvoie `true` que s'il existe des erreurs.

## Intégration avec les libraries de validation par schéma {#integration-with-schema-validation-libraries}

Les Signal Forms prennent en charge nativement les libraries conformes à [Standard Schema](https://standardschema.dev/), comme [Zod](https://zod.dev/) ou [Valibot](https://valibot.dev/). L'intégration est fournie par la fonction `validateStandardSchema`. Cela vous permet d'utiliser des schémas existants tout en conservant les avantages de la validation réactive des Signal Forms.

```ts
import {form, validateStandardSchema} from '@angular/forms/signals';
import {signal} from '@angular/core';
import * as z from 'zod';

// Define your schema
const userSchema = z.object({
  email: z.email(),
  password: z.string().min(8),
});

// Use with Signal Forms
const userForm = form(signal({email: '', password: ''}), (schemaPath) => {
  validateStandardSchema(schemaPath, userSchema);
});
```

### Schémas dynamiques {#dynamic-schemas}

Vous pouvez passer un signal au lieu d'un schéma statique pour que le schéma de validation se mette à jour automatiquement quand ses dépendances changent.

```angular-ts
import {Component, computed, signal} from '@angular/core';
import {form, FormField, validateStandardSchema} from '@angular/forms/signals';
import z from 'zod';

@Component({/* ... */})
export class DynamicSchema {
  model = signal({document: '', type: 'dni'});

  // Schema reacts automatically to type changes
  schema = computed(() =>
    z.object({
      document:
        this.model().type === 'dni'
          ? z.string().length(8, 'DNI must be 8 digits')
          : z.string().min(12, 'Passport must be at least 12 characters'),
    }),
  );

  f = form(this.model, (p) => validateStandardSchema(p, () => this.schema()));
}
```

## Prochaines étapes {#next-steps}

Ce guide a couvert la création et l'application de règles de validation. D'autres guides explorent d'autres aspects des Signal Forms :

<docs-pill-row>
  <docs-pill href="guide/forms/signals/field-state-management" title="Gestion de l'état des champs" />
  <docs-pill href="guide/forms/signals/models" title="Models de formulaire" />
  <docs-pill href="guide/forms/signals/form-logic" title="Ajouter de la logique de formulaire" />
  <docs-pill href="guide/forms/signals/schemas" title="Schémas et composabilité des schémas" />
</docs-pill-row>
