<!-- source: f9e059b7ef5511708daafcfa5a15fc029187b56e -->
# Migrer des formulaires existants vers les Signal Forms

Ce guide présente des stratégies pour migrer des bases de code existantes vers les Signal Forms, en mettant l'accent sur l'interopérabilité avec
les reactive forms existants.

## Migration descendante avec `compatForm` {#top-down-migration-using-compatform}

Vous pouvez parfois vouloir utiliser des instances réactives `FormControl` existantes dans un formulaire Signal Forms. C'est utile pour
les contrôles qui impliquent :

- Une logique asynchrone complexe.
- Des opérateurs RxJS élaborés qui n'ont pas encore été portés.
- Une intégration avec des libraries tierces existantes.

### Intégrer un `FormControl` dans un formulaire à base de signals {#integrating-a-formcontrol-into-a-signal-form}

Prenons un `passwordControl` existant qui utilise un `enterprisePasswordValidator` spécialisé. Au lieu de réécrire le
validateur, vous pouvez relier le contrôle à votre état basé sur des signals.

Vous pouvez le faire avec `compatForm` :

```typescript
import {signal} from '@angular/core';
import {FormControl, Validators} from '@angular/forms';
import {compatForm} from '@angular/forms/signals/compat';

// 1. Existing control with a specialized validator
const passwordControl = new FormControl('', {
  validators: [Validators.required, enterprisePasswordValidator()],
  nonNullable: true,
});

// 2. Wrap it inside your form state signal
const user = signal({
  email: '',
  password: passwordControl, // Nest the existing control directly
});

// 3. Create the form
const f = compatForm(user);

// Access values via the signal tree
console.log(f.email().value()); // Current email value
console.log(f.password().value()); // Current value of passwordControl

// Reactive state is proxied automatically
const isPasswordValid = f.password().valid();
const passwordErrors = f.password().errors(); // Returns CompatValidationError if the existing validator fails
```

Dans le template, utilisez la syntaxe réactive standard en créant un binding vers le contrôle sous-jacent :

```angular-html
<form novalidate>
  <div>
    <label>
      Email:
      <input [formField]="f.email" />
    </label>
  </div>

  <div>
    <label>
      Password:
      <input [formField]="f.password" type="password" />
    </label>

    @if (f.password().touched() && f.password().invalid()) {
      <div class="error-list">
        @for (error of f.password().errors(); track error) {
          <p>{{ errorMessage(error) }}</p>
        }
      </div>
    }
  </div>
</form>
```

Les erreurs issues des validateurs existants sont des `CompatValidationError`. Elles ne définissent pas `message` ; l'objet d'erreur renvoyé par le validateur est disponible dans `context` à la place :

```typescript
import {ValidationError} from '@angular/forms/signals';
import {CompatValidationError} from '@angular/forms/signals/compat';

errorMessage(error: ValidationError) {
  if (error instanceof CompatValidationError) {
    return (error.context as {message?: string} | null)?.message ?? error.kind;
  }
  return error.message ?? error.kind;
}
```

<docs-code-multifile preview path="adev/src/content/examples/signal-forms/src/compat-form-control-integration/app/app.ts">
  <docs-code header="app.ts" path="adev/src/content/examples/signal-forms/src/compat-form-control-integration/app/app.ts"/>
  <docs-code header="app.html" path="adev/src/content/examples/signal-forms/src/compat-form-control-integration/app/app.html"/>
</docs-code-multifile>

### Intégrer un `FormGroup` dans un formulaire à base de signals {#integrating-a-formgroup-into-a-signal-form}

Vous pouvez aussi envelopper un `FormGroup` entier. C'est courant lorsqu'une sous-section réutilisable d'un formulaire, comme un
**bloc d'adresse**, est encore gérée par des reactive forms existants.

```typescript
import {signal} from '@angular/core';
import {FormGroup, FormControl, Validators} from '@angular/forms';
import {compatForm} from '@angular/forms/signals/compat';
import {required} from '@angular/forms/signals';

// 1. An existing address group with its own validation logic
const addressGroup = new FormGroup({
  street: new FormControl('123 Angular Way', Validators.required),
  city: new FormControl('Mountain View', Validators.required),
  zip: new FormControl('94043', Validators.required),
});

// 2. Include it in the state like it's a value
const checkoutModel = signal({
  customerName: 'Pirojok the Cat',
  shippingAddress: addressGroup,
});

const f = compatForm(checkoutModel, (p) => {
  required(p.customerName);
});
```

Le champ `shippingAddress` agit comme une branche dans votre arbre Signal Forms. Vous pouvez lier ces contrôles imbriqués dans votre
template en accédant aux contrôles existants sous-jacents via `.control()` :

```angular-html
<form novalidate>
  <h3>Shipping Details</h3>

  <div>
    <label>
      Customer Name:
      <input [formField]="f.customerName" />
    </label>

    @if (f.customerName().touched() && f.customerName().invalid()) {
      <div class="error-list">
        <p>Customer name is required.</p>
      </div>
    }
  </div>

  <fieldset>
    <legend>Address</legend>

    @let street = f.shippingAddress().control().controls.street;
    <div>
      <label>
        Street:
        <input [formControl]="street" />
      </label>
      @if (street.touched && street.invalid) {
        <div class="error-list">
          <p>Street is required</p>
        </div>
      }
    </div>

    @let city = f.shippingAddress().control().controls.city;
    <div>
      <label>
        City:
        <input [formControl]="city" />
      </label>
      @if (city.touched && city.invalid) {
        <div class="error-list">
          <p>City is required</p>
        </div>
      }
    </div>

    @let zip = f.shippingAddress().control().controls.zip;
    <div>
      <label>
        Zip Code:
        <input [formControl]="zip" />
      </label>
      @if (zip.touched && zip.invalid) {
        <div class="error-list">
          <p>Zip Code is required</p>
        </div>
      }
    </div>
  </fieldset>
</form>
```

<docs-code-multifile preview path="adev/src/content/examples/signal-forms/src/compat-form-group-integration/app/app.ts">
  <docs-code header="app.ts" path="adev/src/content/examples/signal-forms/src/compat-form-group-integration/app/app.ts"/>
  <docs-code header="app.html" path="adev/src/content/examples/signal-forms/src/compat-form-group-integration/app/app.html"/>
</docs-code-multifile>

### Accéder aux valeurs {#accessing-values}

Si `compatForm` relaie l'accès à la valeur au niveau du `FormControl`, la valeur complète du formulaire conserve le contrôle :

```typescript
const passwordControl = new FormControl('password' /** ... */);

const user = signal({
  email: '',
  password: passwordControl, // Nest the existing control directly
});

const form = compatForm(user);
form.password().value(); // 'password'
form().value(); // { email: '', password: FormControl}
```

Si vous avez besoin de la valeur complète du formulaire, vous devez la construire manuellement :

```typescript
const formValue = computed(() => ({
  email: form.email().value(),
  password: form.password().value(),
})); // {email: '', password: ''}
```

## Migration ascendante {#bottom-up-migration}

### Intégrer un formulaire Signal Forms dans un `FormGroup` {#integrating-a-signal-form-into-a-formgroup}

Vous pouvez utiliser `SignalFormControl` pour exposer un formulaire à base de signals comme un `FormControl` standard. C'est utile lorsque vous voulez
migrer les nœuds feuilles d'un formulaire vers les signals tout en conservant la structure du `FormGroup` parent.

```typescript
import {Component, signal} from '@angular/core';
import {ReactiveFormsModule, FormGroup} from '@angular/forms';
import {SignalFormControl} from '@angular/forms/signals/compat';
import {required} from '@angular/forms/signals';

@Component({
  // ...
  imports: [ReactiveFormsModule],
})
export class UserProfile {
  // 1. Create a SignalFormControl, use signal form rules.
  emailControl = new SignalFormControl('', (p) => {
    required(p, {message: 'Email is required'});
  });

  // 2. Use it in an existing FormGroup
  form = new FormGroup({
    email: this.emailControl,
  });
}
```

Le `SignalFormControl` synchronise les valeurs dans les deux sens entre le système **Signal Forms** et le système **Reactive Forms** :

- **Signal -> Reactive** : mettre à jour la valeur via les Signal Forms met immédiatement à jour le contrôle des reactive forms.

```typescript
// Signal Forms update
this.emailControl.fieldTree().value.set('new@example.com');

// Reactive Forms reflects the change
console.log(this.form.value); // {email: 'new@example.com'}
```

- **Reactive -> Signal** : mettre à jour la valeur via le `FormGroup` parent met à jour l'état des Signal Forms.

```typescript
// Reactive Forms update
this.form.patchValue({email: 'other@example.com'});

// Signal Forms reflects the change
console.log(this.emailControl.fieldTree().value()); // 'other@example.com'
```

### Lier un `SignalFormControl` {#binding-signalformcontrol}

Pour utiliser `SignalFormControl` dans un `FormGroup`, passez-le comme contrôle et liez-le dans le template avec `.fieldTree` :

```typescript
readonly emailControl = new SignalFormControl('', (p) => { required(p); });

readonly form = new FormGroup({
  name: new FormControl('Alice'),
  email: this.emailControl,
});
```

```angular-html {prefer}
<form [formGroup]="form">
  <!-- Standard control -->
  <input formControlName="name" />

  <!-- Signal control -->
  <input [formField]="emailControl.fieldTree" />
</form>
```

```angular-html {avoid}
<!-- Avoid: Using formControlName or [formControl] for SignalFormControl -->
<input formControlName="email" />
<input [formControl]="emailControl" />
```

### Pourquoi `SignalFormControl` prend une valeur plutôt qu'un signal {#why-signalformcontrol-takes-a-value-instead-of-a-signal}

Avec les Signal Forms standard, vous créez un formulaire en passant un signal : `form(mySignal)`.

En revanche, `SignalFormControl` prend une **valeur brute** (comme une chaîne ou un objet) comme premier argument :

```typescript
// Takes a raw value, not a signal
const userControl = new SignalFormControl({
  email: 'pirojok@example.com',
});
```

`SignalFormControl` crée le signal en interne pour intercepter les écritures et déclencher les **mises à jour synchrones** attendues par les reactive forms.

Vous pouvez toujours accéder au signal interne via `.sourceValue` :

```typescript
const value = userControl.sourceValue();
```

### Désactiver/activer le contrôle {#disabling-enabling-control}

Les API impératives qui modifient l'état activé/désactivé (comme `enable()`, `disable()`) ne sont volontairement pas prises en charge
dans `SignalFormControl`. En effet, l'état du contrôle doit être dérivé de l'état des signals et des règles.

Toute tentative d'appeler disable/enable lève une erreur.

```typescript {avoid}
import {signal, effect} from '@angular/core';
import {SignalFormControl} from '@angular/forms/signals/compat';

export class UserProfile {
  readonly emailControl = new SignalFormControl('');

  readonly isLoading = signal(false);

  constructor() {
    // This will throw an error
    effect(() => {
      if (this.isLoading()) {
        this.emailControl.disable();
      } else {
        this.emailControl.enable();
      }
    });
  }
}
```

Utilisez plutôt la règle disabled :

```typescript {prefer}
import {signal} from '@angular/core';
import {SignalFormControl} from '@angular/forms/signals/compat';
import {disabled} from '@angular/forms/signals';

export class UserProfile {
  readonly isLoading = signal(false);

  readonly emailControl = new SignalFormControl('', (p) => {
    // The control becomes disabled whenever isLoading is true
    disabled(p, {when: () => this.isLoading()});
  });

  async saveData() {
    this.isLoading.set(true);
    // ... perform save ...
    this.isLoading.set(false);
  }
}
```

### Manipulation dynamique {#dynamic-manipulation}

Les API impératives d'ajout ou de suppression de validateurs (comme `addValidators()`, `removeValidators()`, `setValidators()`) ne sont
volontairement pas prises en charge dans `SignalFormControl`.

Toute tentative d'appeler ces méthodes lève une erreur.

```typescript {avoid}
export class UserProfile {
  readonly emailControl = new SignalFormControl('');
  readonly isRequired = signal(false);

  toggleRequired() {
    this.isRequired.update((v) => !v);
    // This will throw an error
    if (this.isRequired()) {
      this.emailControl.addValidators(Validators.required);
    } else {
      this.emailControl.removeValidators(Validators.required);
    }
  }
}
```

Utilisez plutôt la règle `applyWhen` pour appliquer des validateurs de manière conditionnelle :

```typescript {prefer}
import {signal} from '@angular/core';
import {SignalFormControl} from '@angular/forms/signals/compat';
import {applyWhen, required} from '@angular/forms/signals';

export class UserProfile {
  readonly isRequired = signal(false);

  readonly emailControl = new SignalFormControl('', (p) => {
    // The control becomes required whenever isRequired is true
    applyWhen(
      p,
      () => this.isRequired(),
      (p) => {
        required(p);
      },
    );
  });
}
```

### Définition manuelle des erreurs {#manual-error-selection}

Les méthodes `setErrors()` et `markAsPending()` ne sont pas prises en charge. Avec les Signal Forms, les erreurs sont dérivées des règles
de validation et du statut de la validation asynchrone. Si vous devez signaler une erreur, faites-le de manière déclarative via une règle de validation
dans le schéma.

## Classes de statut automatiques {#automatic-status-classes}

Les reactive forms et les template-driven forms ajoutent automatiquement des [attributs de classe](/guide/forms/template-driven-forms#track-control-states) (
comme `.ng-valid` ou `.ng-dirty`) pour faciliter le style des états des contrôles. Les Signal Forms ne le font pas.

Si vous voulez conserver ce comportement, vous pouvez fournir le preset `NG_STATUS_CLASSES` :

```typescript
import {provideSignalFormsConfig} from '@angular/forms/signals';
import {NG_STATUS_CLASSES} from '@angular/forms/signals/compat';
import {bootstrapApplication} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [
    provideSignalFormsConfig({
      classes: NG_STATUS_CLASSES,
    }),
  ],
});
```

Vous pouvez aussi fournir votre propre configuration pour appliquer les classes de votre choix selon votre logique personnalisée :

```typescript
import {provideSignalFormsConfig} from '@angular/forms/signals';
import {bootstrapApplication} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [
    provideSignalFormsConfig({
      classes: {
        'ng-valid': ({state}) => state().valid(),
        'ng-invalid': ({state}) => state().invalid(),
        'ng-touched': ({state}) => state().touched(),
        'ng-dirty': ({state}) => state().dirty(),
      },
    }),
  ],
});
```

## Contrôles personnalisés {#custom-controls}

N'importe quel [contrôle personnalisé Signal Forms](guide/forms/signals/custom-controls) peut être
utilisé tel quel avec les reactive forms (et les template-driven forms). Cela vous permet de
migrer des implémentations existantes de `ControlValueAccessor` vers
`FormValueControl`/`FormCheckboxControl` sans casser les usages existants.

IMPORTANT: N'implémentez **pas** à la fois `ControlValueAccessor` et
`FormValueControl`/`FormCheckboxControl` sur le même composant. Implémentez l'un
ou l'autre.

Prenons le contrôle personnalisé suivant :

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

Vous pouvez utiliser ce contrôle personnalisé avec les reactive forms comme vous le feriez avec un champ natif
ou un contrôle personnalisé basé sur `ControlValueAccessor`. Par exemple, prenons ce
composant simple avec un formulaire construit avec les reactive forms.

```angular-ts
import {Component} from '@angular/core';
import {FormGroup, FormControl, ReactiveFormsModule} from '@angular/forms';
import {BasicInput} from './basic-input';

@Component({
  selector: 'app-example',
  template: `
    <form [formGroup]="reactiveFormGroup">
      <app-basic-input formControlName="reactiveControlName" />
    </form>
    <p>Text: {{ reactiveFormGroup.value.reactiveControlName }}</p>
  `,
  imports: [ReactiveFormsModule, BasicInput],
})
export class ExampleComponent {
  readonly reactiveFormGroup = new FormGroup({
    reactiveControlName: new FormControl(''),
  });
}
```

Toute modification du contrôle personnalisé `app-basic-input` est répercutée dans le
`FormControl` réactif.
