<!-- source: 7e8f6f3c9be3aa679e0b69d98b4eb735f4db9eea -->
# Contrôles personnalisés

NOTE: Ce guide suppose que vous connaissez déjà les [Fondamentaux des Signal Forms](essentials/signal-forms).

Les contrôles de formulaire natifs du navigateur (comme input, select, textarea) couvrent les cas courants, mais les applications ont souvent besoin de champs de saisie spécialisés. Un sélecteur de date avec une interface de calendrier, un éditeur de texte riche avec une barre d'outils de mise en forme ou un sélecteur de tags avec autocomplétion nécessitent tous des implémentations personnalisées.

Les Signal Forms fonctionnent avec n'importe quel composant qui implémente certaines interfaces. Une **interface de contrôle** définit les propriétés et les signals qui permettent à votre composant de communiquer avec le système de formulaires. Quand votre composant implémente l'une de ces interfaces, la directive `[formField]` connecte automatiquement votre contrôle à l'état du formulaire, à la validation et au binding des données.

HELPFUL: Les contrôles personnalisés des Signal Forms [peuvent être utilisés](guide/forms/signals/migration#custom-controls) avec les Signal Forms, les reactive forms et les template-driven forms, sans aucun code de compatibilité supplémentaire.

## Créer un contrôle personnalisé de base {#creating-a-basic-custom-control}

Commençons par une implémentation minimale, puis ajoutons des fonctionnalités au besoin.

### Contrôle de saisie minimal {#minimal-input-control}

Un champ de saisie personnalisé de base n'a besoin que d'implémenter l'interface `FormValueControl` et de définir le signal model `value` requis.

```angular-ts
import {Component, model} from '@angular/core';
import {FormValueControl} from '@angular/forms/signals';

@Component({
  selector: 'app-basic-input',
  template: `
    <div class="basic-input">
      <input
        type="text"
        [value]="value()"
        (input)="value.set($event.target.value)"
        placeholder="Enter text..."
      />
    </div>
  `,
})
export class BasicInput implements FormValueControl<string> {
  /** The current input value */
  value = model('');
}
```

### Contrôle de case à cocher minimal {#minimal-checkbox-control}

Un contrôle de type case à cocher a besoin de deux choses :

1. Implémenter l'interface `FormCheckboxControl` pour que la directive `FormField` le reconnaisse comme contrôle de formulaire
2. Fournir un signal model `checked`

```angular-ts
import {Component, model, ChangeDetectionStrategy} from '@angular/core';
import {FormCheckboxControl} from '@angular/forms/signals';

@Component({
  selector: 'app-basic-toggle',
  template: `
    <button type="button" [class.active]="checked()" (click)="toggle()">
      <span class="toggle-slider"></span>
    </button>
  `,
})
export class BasicToggle implements FormCheckboxControl {
  /** Whether the toggle is checked */
  checked = model<boolean>(false);

  toggle() {
    this.checked.update((val) => !val);
  }
}
```

### Utiliser votre contrôle personnalisé {#using-your-custom-control}

Une fois votre contrôle créé, vous pouvez l'utiliser partout où vous utiliseriez un champ natif, en lui ajoutant la directive `FormField` :

```angular-ts
import {Component, signal, ChangeDetectionStrategy} from '@angular/core';
import {form, FormField, required} from '@angular/forms/signals';
import {BasicInput} from './basic-input';
import {BasicToggle} from './basic-toggle';

@Component({
  imports: [FormField, BasicInput, BasicToggle],
  template: `
    <form novalidate>
      <label>
        Email
        <app-basic-input [formField]="registrationForm.email" />
      </label>

      <label>
        Accept terms
        <app-basic-toggle [formField]="registrationForm.acceptTerms" />
      </label>

      <button type="submit" [disabled]="registrationForm().invalid()">Register</button>
    </form>
  `,
})
export class Registration {
  registrationModel = signal({
    email: '',
    acceptTerms: false,
  });

  registrationForm = form(this.registrationModel, (schemaPath) => {
    required(schemaPath.email, {message: 'Email is required'});
    required(schemaPath.acceptTerms, {message: 'You must accept the terms'});
  });
}
```

NOTE: Le paramètre du callback de schéma (`schemaPath` dans ces exemples) est un objet `SchemaPathTree` qui fournit les chemins vers tous les champs de votre formulaire. Vous pouvez nommer ce paramètre comme vous le souhaitez.

La directive `[formField]` fonctionne de manière identique pour les contrôles personnalisés et les champs natifs. Les Signal Forms les traitent de la même façon : la validation s'exécute, l'état se met à jour et le binding des données fonctionne automatiquement.

## Comprendre les interfaces de contrôle {#understanding-control-interfaces}

Maintenant que vous avez vu des contrôles personnalisés en action, voyons comment ils s'intègrent aux Signal Forms.

### Interfaces de contrôle {#control-interfaces}

Les composants `BasicInput` et `BasicToggle` que vous avez créés implémentent des interfaces de contrôle spécifiques qui indiquent aux Signal Forms comment interagir avec eux.

#### FormValueControl {#formvaluecontrol}

`FormValueControl` est l'interface destinée à la plupart des types de champs : champs texte, champs numériques, sélecteurs de date, listes déroulantes et tout contrôle qui modifie une seule valeur. Quand votre composant implémente cette interface :

- **Propriété requise** : votre composant doit fournir un signal model `value`
- **Ce que fait la directive FormField** : elle lie la valeur du champ de formulaire au signal `value` de votre contrôle

IMPORTANT: Les contrôles qui implémentent `FormValueControl` ne doivent PAS avoir de propriété `checked`

#### FormCheckboxControl {#formcheckboxcontrol}

`FormCheckboxControl` est l'interface destinée aux contrôles de type case à cocher : bascules, interrupteurs et tout contrôle qui représente un état booléen activé/désactivé. Quand votre composant implémente cette interface :

- **Propriété requise** : votre composant doit fournir un signal model `checked`
- **Ce que fait la directive FormField** : elle lie la valeur du champ de formulaire au signal `checked` de votre contrôle

IMPORTANT: Les contrôles qui implémentent `FormCheckboxControl` ne doivent PAS avoir de propriété `value`

### Propriétés d'état optionnelles {#optional-state-properties}

`FormValueControl` et `FormCheckboxControl` étendent tous deux `FormUiControl`, une interface de base qui fournit des propriétés optionnelles pour s'intégrer à l'état du formulaire.

Toutes les propriétés sont optionnelles. N'implémentez que ce dont votre contrôle a besoin.

#### État d'interaction {#interaction-state}

Suivez les interactions des utilisateurs avec votre contrôle :

| Propriété | Rôle                                             |
| --------- | ------------------------------------------------ |
| `touched` | Indique si l'utilisateur a interagi avec le champ |
| `dirty`   | Indique si la valeur diffère de son état initial |

#### État de validation {#validation-state}

Affichez aux utilisateurs le retour de validation :

| Propriété | Rôle                                    |
| --------- | --------------------------------------- |
| `errors`  | Tableau des erreurs de validation courantes |
| `invalid` | Indique si le champ a des erreurs de validation |
| `pending` | Indique si une validation asynchrone est en cours |

#### État de disponibilité {#availability-state}

Contrôlez si les utilisateurs peuvent interagir avec votre champ :

| Propriété         | Rôle                                                     |
| ----------------- | -------------------------------------------------------- |
| `disabled`        | Indique si le champ est désactivé                        |
| `disabledReasons` | Raisons pour lesquelles le champ est désactivé           |
| `readonly`        | Indique si le champ est en lecture seule (visible mais non modifiable) |
| `hidden`          | Indique si le champ est masqué                           |

NOTE: `disabledReasons` est un tableau d'objets `DisabledReason`. Chaque objet possède une propriété `fieldTree` (référence à l'arbre de champs) et une propriété optionnelle `message`. Accédez au message via `reason.message`.

#### Contraintes de validation {#validation-constraints}

Recevez du formulaire les valeurs des contraintes de validation :

| Propriété   | Rôle                                                 |
| ----------- | ---------------------------------------------------- |
| `required`  | Indique si le champ est obligatoire                  |
| `min`       | Valeur numérique minimale (`undefined` en l'absence de contrainte) |
| `max`       | Valeur numérique maximale (`undefined` en l'absence de contrainte) |
| `minLength` | Longueur minimale de la chaîne (undefined en l'absence de contrainte) |
| `maxLength` | Longueur maximale de la chaîne (undefined en l'absence de contrainte) |
| `pattern`   | Tableau de motifs d'expressions régulières à respecter |

#### Métadonnées du champ {#field-metadata}

| Propriété | Rôle                                                               |
| -------- | ------------------------------------------------------------------ |
| `name`   | L'attribut name du champ (unique entre les formulaires et les applications) |

La section « [Ajouter des signals d'état](#adding-state-signals) » ci-dessous montre comment implémenter ces propriétés dans vos contrôles.

### Fonctionnement de la directive FormField {#how-the-formfield-directive-works}

La directive `[formField]` détecte l'interface que votre contrôle implémente et lie automatiquement les signals appropriés :

```angular-ts
import {Component, signal, ChangeDetectionStrategy} from '@angular/core';
import {form, FormField, required} from '@angular/forms/signals';
import {CustomInput} from './custom-input';
import {CustomToggle} from './custom-toggle';

@Component({
  selector: 'app-my-form',
  imports: [FormField, CustomInput, CustomToggle],
  template: `
    <form novalidate>
      <app-custom-input [formField]="userForm.username" />
      <app-custom-toggle [formField]="userForm.subscribe" />
    </form>
  `,
})
export class MyForm {
  formModel = signal({
    username: '',
    subscribe: false,
  });

  userForm = form(this.formModel, (schemaPath) => {
    required(schemaPath.username, {message: 'Username is required'});
  });
}
```

TIP: Pour une présentation complète de la création et de la gestion des models de formulaire, consultez le [guide des models de formulaire](guide/forms/signals/models).

Quand vous liez `[formField]="userForm.username"`, la directive FormField :

1. Détecte que votre contrôle implémente `FormValueControl`
2. Accède en interne à `userForm.username().value()` et le lie au signal model `value` de votre contrôle
3. Lie les signals d'état du formulaire (`disabled()`, `errors()`, etc.) aux signals d'input optionnels de votre contrôle
4. Les mises à jour se font automatiquement grâce à la réactivité des signals

## Ajouter des signals d'état {#adding-state-signals}

Les contrôles minimaux présentés ci-dessus fonctionnent, mais ils ne réagissent pas à l'état du formulaire. Vous pouvez ajouter des signals d'input optionnels pour que vos contrôles réagissent à l'état désactivé, affichent les erreurs de validation et suivent les interactions de l'utilisateur.

Voici un exemple complet qui implémente les propriétés d'état courantes :

```angular-ts
import {Component, model, input, output, ChangeDetectionStrategy} from '@angular/core';
import {
  FormValueControl,
  WithOptionalFieldTree,
  ValidationError,
  DisabledReason,
} from '@angular/forms/signals';

@Component({
  selector: 'app-stateful-input',
  template: `
    @if (!hidden()) {
      <div class="input-container">
        <input
          type="text"
          [value]="value()"
          (input)="value.set($event.target.value)"
          [disabled]="disabled()"
          [readonly]="readonly()"
          [class.invalid]="invalid()"
          [attr.aria-invalid]="invalid()"
          (blur)="touch.emit()"
        />

        @if (invalid()) {
          <div class="error-messages" role="alert">
            @for (error of errors(); track error) {
              <span class="error">{{ error.message }}</span>
            }
          </div>
        }

        @if (disabled() && disabledReasons().length > 0) {
          <div class="disabled-reasons">
            @for (reason of disabledReasons(); track reason) {
              <span>{{ reason.message }}</span>
            }
          </div>
        }
      </div>
    }
  `,
})
export class StatefulInput implements FormValueControl<string> {
  // Required
  value = model<string>('');

  // Writable interaction state - control updates these
  touched = input<boolean>(false);
  touch = output<void>();

  // Read-only state - form system manages these
  disabled = input<boolean>(false);
  disabledReasons = input<readonly WithOptionalFieldTree<DisabledReason>[]>([]);
  readonly = input<boolean>(false);
  hidden = input<boolean>(false);
  invalid = input<boolean>(false);
  errors = input<readonly WithOptionalFieldTree<ValidationError>[]>([]);
}
```

Vous pouvez ainsi utiliser le contrôle avec la validation et la gestion d'état :

```angular-ts
import {Component, signal, ChangeDetectionStrategy} from '@angular/core';
import {form, FormField, required, email} from '@angular/forms/signals';
import {StatefulInput} from './stateful-input';

@Component({
  imports: [FormField, StatefulInput],
  template: `
    <form novalidate>
      <label>
        Email
        <app-stateful-input [formField]="loginForm.email" />
      </label>
    </form>
  `,
})
export class Login {
  loginModel = signal({email: ''});

  loginForm = form(this.loginModel, (schemaPath) => {
    required(schemaPath.email, {message: 'Email is required'});
    email(schemaPath.email, {message: 'Enter a valid email address'});
  });
}
```

Quand l'utilisateur saisit un e-mail invalide, la directive FormField met automatiquement à jour `invalid()` et `errors()`. Votre contrôle peut afficher le retour de validation.

### Utiliser `debounce('blur')` {#working-with-debounceblur}

La règle [`debounce('blur')`](api/forms/signals/debounce) retarde la propagation des mises à jour de l'interface vers le model de formulaire jusqu'à ce que le champ perde le focus, au lieu de les appliquer à chaque frappe. Les contrôles natifs signalent automatiquement la perte de focus au formulaire. Un contrôle personnalisé n'y participe que s'il émet son output `touch` en réponse à l'[événement `blur`](https://developer.mozilla.org/en-US/docs/Web/API/Element/blur_event) natif :

```angular-ts
import {Component, model, output} from '@angular/core';
import {FormValueControl} from '@angular/forms/signals';

@Component({
  selector: 'app-custom-input',
  template: `
    <input
      type="text"
      [value]="value()"
      (input)="value.set($event.target.value)"
      (blur)="touch.emit()"
    />
  `,
})
export class CustomInput implements FormValueControl<string> {
  value = model('');
  touch = output<void>();
}
```

Avec l'output `touch` en place, `debounce('blur')` se comporte pour votre contrôle exactement comme pour les champs natifs :

```angular-ts
import {Component, signal} from '@angular/core';
import {debounce, form, FormField} from '@angular/forms/signals';
import {CustomInput} from './custom-input';

@Component({
  selector: 'app-root',
  imports: [CustomInput, FormField],
  template: `<app-custom-input [formField]="userForm.name" />`,
})
export class App {
  userModel = signal({name: ''});

  userForm = form(this.userModel, (schemaPath) => {
    debounce(schemaPath.name, 'blur');
  });
}
```

IMPORTANT: Émettez `touch` sur `blur` (quand le focus quitte le contrôle), pas sur `focus`. Sans l'output `touch`, le champ n'est jamais considéré comme ayant perdu le focus, et `debounce('blur')` n'a donc aucun effet sur votre contrôle.

## Transformation de valeur {#value-transformation}

Les contrôles affichent parfois les valeurs autrement que le model de formulaire ne les stocke : un sélecteur de date peut afficher « 15 janvier 2024 » tout en stockant « 2024-01-15 », ou un champ monétaire peut afficher « 1 234,56 $ » tout en stockant 1234.56.

Utilisez `transformedValue()` (de `@angular/forms/signals`) pour garder la valeur brute affichée dans l'interface synchronisée avec la valeur du model. Elle prend le signal model `value` du contrôle ainsi qu'une fonction `parse` et une fonction `format`, et renvoie un signal modifiable contenant la valeur brute :

- `format` convertit la valeur du model en la valeur brute affichée par le template.
- `parse` reconvertit ce que l'utilisateur a saisi en valeur du model, et peut à la place signaler des erreurs d'analyse.

```angular-ts
import {Component, model} from '@angular/core';
import {FormValueControl, transformedValue} from '@angular/forms/signals';

@Component({
  selector: 'number-input',
  template: `
    <input type="text" [value]="rawValue()" (input)="rawValue.set($event.target.value)" />
  `,
})
export class NumberInput implements FormValueControl<number | null> {
  readonly value = model.required<number | null>();

  protected readonly rawValue = transformedValue(this.value, {
    parse: (val: string): {value: number | null} => ({value: val ? Number(val) : null}),
    format: (val: number | null): string => val?.toString() ?? '',
  });
}
```

Écrire dans le signal renvoyé (`rawValue.set(...)`) exécute `parse` et écrit le résultat dans `value`. Chaque fois que le model change ailleurs (un `reset()`, une règle du schéma ou une autre partie de l'application), `format` s'exécute à nouveau et la valeur brute se met à jour en conséquence.

### Signaler les erreurs d'analyse {#reporting-parse-errors}

Parfois, la valeur brute n'a aucune représentation valide dans le model : une date à moitié saisie, ou des lettres dans un champ numérique. Le `NumberInput` ci-dessus présente ce problème : `Number('abc')` vaut `NaN`, que `parse` écrit sans broncher dans le model.

Renvoyez plutôt `{error}` :

```ts
export class NumberInput implements FormValueControl<number | null> {
  readonly value = model.required<number | null>();

  protected readonly rawValue = transformedValue(this.value, {
    parse: (val) => {
      const parsed = val ? Number(val) : null;

      return Number.isNaN(parsed)
        ? {error: {kind: 'parse', message: `${val} is not a number`}}
        : {value: parsed};
    },
    format: (val) => val?.toString() ?? '',
  });
}
```

Renvoyez à la fois `value` et `error` quand vous voulez mettre à jour le model _et_ signaler un problème.

Quand le contrôle est lié avec `[formField]`, les erreurs d'analyse sont automatiquement remontées au champ : elles apparaissent donc dans le signal `errors()` du champ, aux côtés des erreurs de validation :

```angular-ts
@Component({
  imports: [NumberInput, FormField],
  template: `
    <number-input [formField]="orderForm.amount" />

    @for (error of orderForm.amount().errors(); track $index) {
      <!-- {kind: 'parse', message: '...'} is reported here too -->
      <p class="error">{{ error.message }}</p>
    }
  `,
})
export class Order {
  orderModel = signal<{amount: number | null}>({amount: null});
  orderForm = form(this.orderModel);
}
```

Un champ qui a des erreurs d'analyse est invalide, ce qui bloque la soumission de la même manière qu'une règle de validation non respectée.

HELPFUL: Les Signal Forms utilisent le même mécanisme pour les champs natifs. Quand le navigateur ne parvient pas à analyser une valeur (par exemple, une date partiellement saisie dans `<input type="date">`), elle apparaît comme une erreur `parse` sur le champ. Consultez [Validation HTML native](guide/forms/signals/validation#native-html-validation) pour plus de détails.

### Réinitialisation {#resetting}

Appeler `reset()` sur le champ efface les éventuelles erreurs d'analyse en attente et reformate la valeur brute à partir du model : un contrôle laissé dans un état non analysable retrouve ainsi une valeur d'affichage propre :

```ts
orderForm.amount().reset();
```

## Intégration avec la validation {#validation-integration}

Les contrôles affichent l'état de validation mais n'effectuent pas la validation. La validation a lieu dans le schéma du formulaire : votre contrôle reçoit les signals `invalid()` et `errors()` de la directive FormField et les affiche (comme dans l'exemple StatefulInput ci-dessus).

La directive FormField transmet aussi les valeurs des contraintes de validation comme `required`, `min`, `max`, `minLength`, `maxLength` et `pattern`. Votre contrôle peut s'en servir pour enrichir l'interface :

```ts
export class NumberInput implements FormValueControl<number> {
  value = model<number>(0);

  // Constraint values from schema validation rules
  required = input<boolean>(false);
  min = input<number | undefined>(undefined);
  max = input<number | undefined>(undefined);
}
```

Quand vous ajoutez des règles de validation `min()` et `max()` au schéma, la directive FormField transmet ces valeurs à votre contrôle. Utilisez-les pour appliquer des attributs HTML5 ou afficher des indications sur les contraintes dans votre template.

IMPORTANT: N'implémentez pas de logique de validation dans votre contrôle. Définissez les règles de validation dans le schéma du formulaire et laissez votre contrôle afficher les résultats :

```ts {avoid}
// Avoid: Validation in control
export class BadControl implements FormValueControl<string> {
  value = model<string>('');
  isValid() {
    return this.value().length >= 8;
  } // Don't do this!
}
```

```ts {prefer}
// Good: Validation in schema, control displays results
accountForm = form(this.accountModel, (schemaPath) => {
  minLength(schemaPath.password, 8, {message: 'Password must be at least 8 characters'});
});
```

## Rendre les contrôles réutilisables {#making-controls-reusable}

Un contrôle personnalisé implique souvent des attentes implicites en matière de validation. Un champ e-mail a besoin des règles `required` et `email` dans chaque formulaire qui l'utilise. Plutôt que de compter sur chaque consommateur pour redéclarer ces règles, fournissez un schéma compagnon avec le contrôle et exportez les deux depuis le même module :

```ts {header: 'email-input.ts'}
import {schema, required, email} from '@angular/forms/signals';

export const emailFieldSchema = schema<string>((path) => {
  required(path, {message: 'Email is required'});
  email(path, {message: 'Enter a valid email address'});
});
```

Un consommateur importe le schéma compagnon et l'intègre à son formulaire avec `apply()` :

```ts {header: 'registration.ts'}
import {form, apply} from '@angular/forms/signals';
import {emailFieldSchema} from './email-input';

registrationForm = form(this.registrationModel, (path) => {
  apply(path.email, emailFieldSchema);
});
```

`apply()` fusionne les règles du schéma compagnon dans le formulaire parent au chemin indiqué. Le consommateur peut toujours ajouter d'autres règles au même champ, car `apply()` se combine avec les autres règles au lieu de les remplacer. Consultez le [guide des schémas](guide/forms/signals/schemas) pour une présentation complète de `schema()`, `apply()` et de la composition conditionnelle avec `applyWhen()`.

### Points de conception {#design-considerations}

Le model du consommateur doit initialiser chaque champ avec une valeur définie. Dans les Signal Forms, `undefined` signifie l'absence d'un champ, et non une valeur vide. Pour un contrôle e-mail réutilisable, le consommateur doit donc utiliser `''` comme valeur initiale, et ne pas laisser la propriété indéfinie. Consultez le [guide des models de formulaire](guide/forms/signals/models) pour savoir comment choisir les valeurs initiales.

Par ailleurs, les contrôles ne doivent pas enregistrer leurs propres effects pour gérer l'état. Le système de formulaires gère l'état des champs au moyen d'effects internes. Votre contrôle reçoit donc les mises à jour d'état via des signals d'input. Si un contrôle doit transformer des valeurs, utilisez `transformedValue()` comme indiqué dans la section « [Transformation de valeur](#value-transformation) » plutôt qu'un `effect()`.

## Étapes suivantes {#next-steps}

Ce guide a présenté la création de contrôles personnalisés intégrés aux Signal Forms. D'autres guides explorent d'autres aspects des Signal Forms :

<docs-pill-row>
  <docs-pill href="guide/forms/signals/models" title="Models de formulaire" />
  <docs-pill href="guide/forms/signals/field-state-management" title="Gestion de l'état des champs" />
  <docs-pill href="guide/forms/signals/validation" title="Validation" />
  <!-- <docs-pill href="guide/forms/signals/arrays" title="Working with Arrays" /> -->
</docs-pill-row>
