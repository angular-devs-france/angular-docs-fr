<!-- source: 72012936a257dac78a8a0268962cc4edc868950a -->
# Soumission de formulaire

Lorsqu'un utilisateur soumet un formulaire, votre application doit généralement gérer plusieurs aspects à la fois : afficher les erreurs de validation, empêcher les soumissions en double, envoyer les données à un serveur, et bien plus encore. Gérer chacun de ces aspects manuellement peut être fastidieux et source d'erreurs.

Les Signal Forms fournissent une fonction `submit()` qui vous aide à gérer le cycle de vie de la soumission d'un formulaire. Ce guide explique comment l'utiliser.

## Que fait `submit()` ? {#what-does-submit-do}

La fonction `submit()` suit une séquence précise :

1. **Marquer les champs interactifs comme touchés** — Les champs qui n'affichent leurs erreurs qu'après avoir été touchés affichent alors leurs erreurs de validation. Les champs masqués, désactivés et en lecture seule sont ignorés.
1. **Vérifier la validation** — Si une règle de validation a échoué, la soumission s'arrête et la fonction `action` ne s'exécute pas.
1. **Exécuter l'action** — La fonction `action` s'exécute avec la valeur courante du formulaire. Pendant son exécution, `submitting()` renvoie `true`.
1. **Traiter le résultat** — Si l'action renvoie des erreurs, elles sont acheminées vers leurs champs cibles. Si elle ne renvoie rien, la soumission est considérée comme réussie.

La fonction `submit()` renvoie une `Promise<boolean>` qui se résout à `true` lorsque l'action se termine sans erreur, et à `false` lorsque la validation échoue, que l'action renvoie des erreurs ou qu'une soumission est déjà en cours. Si l'action lève une exception, la promise est rejetée avec cette erreur.

## Configurer la soumission du formulaire avec `FormRoot` {#setting-up-form-submission-with-formroot}

La façon la plus courante d'utiliser la fonction `submit()` passe par la directive `FormRoot`.

Lorsqu'elle est liée à un élément `<form>`, la directive `FormRoot` gère automatiquement trois choses :

1. **Définit [`novalidate`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/form#novalidate)** — Désactive la validation native du navigateur pour que les Signal Forms gèrent la validation à sa place
1. **Empêche le comportement par défaut** — Empêche le navigateur de naviguer lors de la soumission du formulaire
1. **Appelle `submit()`** — Déclenche le flux de soumission lorsque l'utilisateur soumet le formulaire, si celui-ci a été créé avec une option `submission`

NOTE: La directive `FormRoot` définit automatiquement l'attribut `novalidate` sur l'élément `form`. Vous n'avez pas besoin de l'ajouter manuellement lorsque vous utilisez `FormRoot`.

`FormRoot` gère l'événement de soumission, mais vous devez encore lui indiquer _quoi faire_ des données du formulaire. Cela nécessite trois choses :

1. Liez votre formulaire à la directive `FormRoot`
1. Passez une option `submission` à la fonction `form()`
1. Définissez, dans l'option `submission`, une fonction `action` qui traite les données soumises

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, FormRoot, required} from '@angular/forms/signals';

@Component({
  selector: 'app-contact',
  imports: [FormField, FormRoot],
  template: `
    <form [formRoot]="contactForm">
      <label>
        Name
        <input [formField]="contactForm.name" />
      </label>

      <label>
        Email
        <input type="email" [formField]="contactForm.email" />
      </label>

      <button type="submit">Send</button>
    </form>
  `,
})
export class Contact {
  contactModel = signal({
    name: '',
    email: '',
  });

  contactForm = form(
    this.contactModel,
    (schemaPath) => {
      required(schemaPath.name);
      required(schemaPath.email);
    },
    {
      submission: {
        action: async (field) => {
          const result = await saveContact(field().value());
          if (result.ok) return;

          return {kind: 'serverError', message: 'Failed to submit form'};
        },
      },
    },
  );
}
```

La fonction `action` ne s'exécute que si aucune règle de validation n'a échoué. Par défaut, les validateurs asynchrones en attente ne bloquent pas la soumission (voir [Contrôler le blocage par la validation](#controlling-validation-gating-with-ignorevalidators) pour plus de détails). L'action reçoit l'arbre de champs et un objet `detail` contenant les arbres de champs `root` et `submitted`, ce qui est utile pour soumettre un sous-formulaire.

Une fois la validation réussie, l'action elle-même peut encore échouer, par exemple à cause d'une erreur réseau ou d'une entrée en double. Dans ces cas, vous pouvez signaler l'échec en renvoyant la ou les erreurs. À l'inverse, pour indiquer un succès, il suffit de renvoyer `null` ou `undefined`, ou d'utiliser un `return` vide.

## Afficher l'état de soumission avec `submitting()` {#showing-submission-state-with-submitting}

Lorsque vous devez savoir si le formulaire est en cours de soumission, les Signal Forms fournissent un signal `submitting()` qui renvoie `true` pendant l'exécution de la fonction `action`. Utilisez-le pour afficher des indicateurs de chargement ou désactiver le bouton de soumission afin d'empêcher les soumissions en double.

```angular-html
<button type="submit" [disabled]="contactForm().submitting()">
  @if (contactForm().submitting()) {
    Sending...
  } @else {
    Send
  }
</button>
```

Dès que la fonction `action` réussit ou renvoie une erreur, le signal `submitting()` revient automatiquement à `false`.

## Gérer les erreurs de soumission {#managing-submission-errors}

### Erreurs serveur {#server-errors}

Lorsque votre fonction `action` communique avec un serveur, celui-ci peut renvoyer des erreurs qui doivent apparaître sur des champs précis. Renvoyez ces erreurs depuis l'`action` pour les acheminer vers leurs champs cibles.

#### Erreurs sur le champ soumis {#errors-on-the-submitted-field}

Par défaut, les erreurs renvoyées par l'`action` sont attribuées au champ soumis (l'arbre de champs que vous avez passé à `submit()`) :

```ts
action: async (field) => {
  const result = await saveContact(field().value());
  if (result.ok) return;

  return {kind: 'serverError', message: 'Failed to submit form'};
};
```

#### Erreurs sur des champs spécifiques {#errors-on-specific-fields}

Pour acheminer une erreur vers un champ spécifique, ajoutez une propriété `fieldTree` qui pointe vers ce champ :

```ts
action: async (field) => {
  const result = await saveContact(field().value());
  if (result.ok) return;

  return {kind: 'taken', message: result.message, fieldTree: field.email};
};
```

#### Erreurs multiples {#multiple-errors}

Pour signaler des erreurs sur plusieurs champs, renvoyez un tableau :

```ts
action: async (field) => {
  const result = await registerUser(field().value());
  if (result.ok) return;

  return result.errors.map((err: {field: string; message: string}) => ({
    kind: 'serverError',
    message: err.message,
    fieldTree: field[err.field as keyof typeof field],
  }));
};
```

### Effacement automatique des erreurs de soumission {#auto-clearing-submission-errors}

Les erreurs de soumission s'effacent automatiquement lorsque l'utilisateur modifie le champ. Si l'`action` renvoie une erreur sur le champ email, cette erreur disparaît dès que l'utilisateur modifie la valeur de l'email.

Ce comportement diffère de celui des erreurs de validation, qui sont recalculées de manière réactive. Les règles de validation s'exécutent à nouveau à chaque modification et peuvent produire la même erreur. Les erreurs de soumission sont des résultats ponctuels renvoyés par le serveur : une fois effacées, elles ne réapparaissent pas, sauf si le formulaire est soumis à nouveau.

TIP: Les erreurs de soumission apparaissent aux côtés des erreurs de validation dans le signal `errors()` du champ. Pour savoir comment afficher les erreurs dans votre template, consultez le [guide de gestion de l'état des champs](guide/forms/signals/field-state-management).

## Gérer les soumissions invalides avec `onInvalid` {#handling-invalid-submissions-with-oninvalid}

Lorsque la validation échoue, la fonction `action` ne s'exécute pas. Si vous devez réagir à une tentative de soumission échouée (par exemple faire défiler la page jusqu'à la première erreur, afficher un toast ou donner le focus à un champ invalide), utilisez le callback `onInvalid`.

```ts
contactForm = form(
  this.contactModel,
  (schemaPath) => {
    required(schemaPath.name);
    required(schemaPath.email);
  },
  {
    submission: {
      action: async (field) => {
        await saveContact(field().value());
      },
      onInvalid: (field) => {
        const firstError = field().errorSummary()[0];
        firstError?.fieldTree().focusBoundControl();
      },
    },
  },
);
```

Le callback `onInvalid` reçoit les mêmes paramètres `(field, detail)` que `action`. Il s'exécute après que tous les champs interactifs ont été marqués comme touchés : les erreurs de validation sont donc déjà visibles dans l'interface lorsqu'il s'exécute.

## Contrôler le blocage par la validation avec `ignoreValidators` {#controlling-validation-gating-with-ignorevalidators}

Par défaut, `submit()` ignore les validateurs en attente. Si aucun validateur n'a échoué, l'action s'exécute même si certains validateurs asynchrones sont encore en cours. L'option `ignoreValidators` vous permet de contrôler ce comportement.

| Valeur      | Comportement                                                                                                  |
| ----------- | ------------------------------------------------------------------------ |
| `'pending'` | Soumettre si aucun validateur n'a échoué, même si certains sont en attente (par défaut)                        |
| `'none'`    | Soumettre uniquement si tous les validateurs réussissent : les validateurs en attente bloquent la soumission |
| `'all'`     | Toujours soumettre, quel que soit l'état de validation                                                        |

```ts
contactForm = form(
  this.contactModel,
  (schemaPath) => {
    required(schemaPath.name);
    required(schemaPath.email);
  },
  {
    submission: {
      action: async (field) => {
        await saveContact(field().value());
      },
      ignoreValidators: 'none',
    },
  },
);
```

Utilisez `'none'` lorsque votre formulaire comporte des validateurs asynchrones (comme la vérification de la disponibilité d'un nom d'utilisateur) et que toute la validation doit être terminée avant la soumission. Utilisez `'all'` pour les scénarios d'enregistrement de brouillon, dans lesquels vous voulez persister les données quel que soit l'état de validation.

## Soumission manuelle avec `submit()` {#manual-submission-with-submit}

La directive `FormRoot` est la façon la plus courante de déclencher la soumission, mais vous pouvez aussi appeler `submit()` directement. C'est utile pour les assistants en plusieurs étapes, l'enregistrement automatique ou pour déclencher la soumission depuis l'extérieur de l'élément de formulaire.

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, required, submit} from '@angular/forms/signals';

@Component({
  selector: 'app-contact',
  imports: [FormField],
  template: `
    <label>
      Name
      <input [formField]="contactForm.name" />
    </label>

    <label>
      Email
      <input type="email" [formField]="contactForm.email" />
    </label>

    <button (click)="onSave()">Save</button>
  `,
})
export class Contact {
  contactModel = signal({
    name: '',
    email: '',
  });

  contactForm = form(this.contactModel, (schemaPath) => {
    required(schemaPath.name);
    required(schemaPath.email);
  });

  async onSave() {
    // When calling `submit()` directly, you pass the action as the second argument
    // instead of configuring it in `FormOptions`.
    const success = await submit(this.contactForm, async (field) => {
      const result = await saveContact(field().value());
      if (result.ok) return;

      return {kind: 'serverError', message: 'Failed to save'};
    });

    if (success) {
      // Handle success — navigate, show confirmation, etc.
    }
  }
}
```

## Gérer les effets de bord {#handling-side-effects}

La fonction `submit()` renvoie une `Promise<boolean>` : `true` lorsque l'action se termine sans erreur, `false` lorsque la validation échoue, que l'action renvoie des erreurs ou qu'une soumission est déjà en cours. Si l'action lève une exception, la promise est rejetée. Utilisez ce résultat pour déclencher des effets de bord comme une navigation ou des notifications.

```ts
async onSave() {
  const success = await submit(this.contactForm, async (field) => {
    await saveContact(field().value());
  });

  if (success) {
    await this.router.navigate(['/confirmation']);
  }
}
```

Lorsque l'action produit des données dont un effet de bord a besoin, comme un identifiant généré par le serveur, gérez l'effet de bord à l'intérieur de l'action :

```ts
async onSave() {
  await submit(this.contactForm, async (field) => {
    const contact = await createContact(field().value());
    await this.router.navigate(['/confirmation', contact.id]);
  });
}
```

Avec `FormRoot`, les effets de bord se placent eux aussi dans l'`action`, puisque `FormRoot` appelle `submit()` en interne :

```ts
submission: {
  action: async (field) => {
    const result = await saveContact(field().value());
    if (result.ok) {
      await this.router.navigate(['/confirmation']);
      return;
    }

    return {kind: 'serverError', message: 'Failed to submit form'};
  },
}
```

## Soumissions concurrentes {#concurrent-submissions}

Lorsqu'une soumission est en cours, les appels suivants à `submit()` pour le même formulaire ou l'un de ses descendants renvoient immédiatement `false` sans exécuter l'action. Cela évite les soumissions et les effets de bord en double si l'utilisateur déclenche plusieurs fois rapidement la soumission.

## Étapes suivantes {#next-steps}

Ce guide a présenté la soumission des formulaires et la gestion des erreurs de soumission. D'autres guides explorent d'autres aspects des Signal Forms :

<docs-pill-row>
  <docs-pill href="guide/forms/signals/validation" title="Validation" />
  <docs-pill href="guide/forms/signals/field-state-management" title="Gestion de l'état des champs" />
  <docs-pill href="guide/forms/signals/form-logic" title="Ajouter de la logique de formulaire" />
</docs-pill-row>
