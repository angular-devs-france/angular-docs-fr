<!-- source: 11c25911b4eadd96a81c89ef93170e28cff44e21 -->
# Opérations asynchrones

Certaines validations nécessitent des données provenant de sources externes, comme des API backend ou des services tiers. Les Signal Forms fournissent deux fonctions pour la validation asynchrone : `validateHttp()` pour la validation basée sur HTTP et `validateAsync()` pour une validation personnalisée basée sur une resource.

## Quand utiliser la validation asynchrone {#when-to-use-async-validation}

Utilisez la validation asynchrone lorsque votre logique de validation nécessite des données externes. Voici quelques exemples courants :

- **Vérifications d'unicité** - Vérifier que des noms d'utilisateur ou des e-mails n'existent pas déjà
- **Recherches en base de données** - Comparer des valeurs à des données côté serveur
- **Validation via une API externe** - Valider des adresses, des numéros fiscaux ou d'autres données avec des services tiers
- **Règles métier côté serveur** - Appliquer des règles de validation que seul le serveur peut vérifier

N'utilisez pas la validation asynchrone pour des vérifications que vous pouvez effectuer de manière synchrone sur le client. Utilisez des règles de validation synchrones comme `pattern()`, `email()` ou `validate()` pour la validation de format et les règles statiques.

## Fonctionnement de la validation asynchrone {#how-async-validation-works}

La validation asynchrone ne s'exécute qu'une fois que toute la validation synchrone a réussi. Pendant l'exécution de la validation, le signal `pending()` du champ renvoie `true`. La validation peut cibler des erreurs sur des champs précis, et les requêtes en attente sont annulées automatiquement lorsque les valeurs des champs changent.

Voici un exemple qui vérifie la disponibilité d'un nom d'utilisateur :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, validateHttp, FormField} from '@angular/forms/signals';

@Component({
  selector: 'app-registration',
  imports: [FormField],
  template: `
    <form>
      <label>
        Username:
        <input [formField]="registrationForm.username" />
      </label>

      @if (registrationForm.username().pending()) {
        <span class="checking">Checking availability...</span>
      }
      @if (registrationForm.username().invalid()) {
        @for (error of registrationForm.username().errors(); track $index) {
          <span class="error">{{ error.message }}</span>
        }
      }
    </form>
  `,
})
export class Registration {
  registrationModel = signal({username: ''});

  registrationForm = form(this.registrationModel, (schemaPath) => {
    validateHttp(schemaPath.username, {
      request: ({value}) => {
        const username = value();
        return username ? `/api/users/check?username=${username}` : undefined;
      },
      onSuccess: (response: {available: boolean}) => {
        return response.available
          ? null
          : {
              kind: 'usernameTaken',
              message: 'Username is already taken',
            };
      },
      onError: (error) => {
        console.error('Validation request failed:', error);
        return {
          kind: 'serverError',
          message: 'Could not verify username availability',
        };
      },
    });
  });
}
```

Le flux de validation se déroule ainsi :

1. L'utilisateur saisit une valeur
2. Les règles de validation synchrones s'exécutent en premier
3. Si la validation synchrone échoue, la validation asynchrone ne s'exécute pas
4. Si la validation synchrone réussit, la validation asynchrone démarre et `pending()` passe à `true`
5. La requête se termine et `pending()` passe à `false`
6. Les erreurs sont mises à jour en fonction de la réponse

## Validation HTTP avec validateHttp() {#http-validation-with-validatehttp}

La fonction `validateHttp()` fournit la forme la plus courante de validation asynchrone. Utilisez-la lorsque vous devez valider auprès d'une API REST ou de n'importe quel endpoint HTTP.

### Fonction de requête {#request-function}

La fonction `request` renvoie soit une URL sous forme de chaîne, soit un objet `HttpResourceRequest`. Renvoyez `undefined` pour ignorer la validation :

```ts
import {Component, signal} from '@angular/core';
import {form, validateHttp, FormField} from '@angular/forms/signals';

@Component({
  selector: 'app-registration',
  imports: [FormField],
  template: `...`,
})
export class Registration {
  registrationModel = signal({username: ''});

  // Cache usernames that passed validation
  private validatedUsernames = new Set<string>();

  registrationForm = form(this.registrationModel, (schemaPath) => {
    validateHttp(schemaPath.username, {
      request: ({value}) => {
        const username = value();
        // Skip HTTP request if already validated
        if (this.validatedUsernames.has(username)) return undefined;

        return `/api/users/check?username=${username}`;
      },
      onSuccess: (response: {available: boolean}, {value}) => {
        if (response.available) {
          // Cache successful validations
          this.validatedUsernames.add(value());
          return null;
        }
        return {
          kind: 'usernameTaken',
          message: 'Username is already taken',
        };
      },
      onError: () => ({
        kind: 'serverError',
        message: 'Could not verify username',
      }),
    });
  });
}
```

Pour des requêtes POST ou des en-têtes personnalisés, renvoyez un objet `HttpResourceRequest` :

```ts
request: ({value}) => ({
  url: '/api/validate',
  method: 'POST',
  body: {username: value()},
}) // prettier-ignore
```

### Gestionnaires de succès et d'erreur {#success-and-error-handlers}

La fonction `onSuccess` reçoit la réponse HTTP et renvoie des erreurs de validation, ou `undefined` pour les valeurs valides :

```ts
onSuccess: (response: { valid: boolean; message?: string }) => {
  if (response.valid) return undefined;

  return {
    kind: 'invalid',
    message: response.message || 'Validation failed',
  };
} // prettier-ignore
```

Renvoyez plusieurs erreurs si nécessaire :

```ts
onSuccess: (response: { usernameTaken: boolean; profanity: boolean }) => {
  const errors = [];
  if (response.usernameTaken) {
    errors.push({
      kind: 'usernameTaken',
      message: 'Username taken',
    });
  }
  if (response.profanity) {
    errors.push({
      kind: 'profanity',
      message: 'Username contains inappropriate content',
    });
  }
  return errors.length > 0 ? errors : undefined;
} // prettier-ignore
```

Le type de `onSuccess` peut être spécifié soit directement dans le paramètre, soit avec la propriété `parse` des `options` de `validateHttp`

```ts
onSuccess: (response: { usernameTaken: boolean; profanity: boolean }) => {
  // ...
} // prettier-ignore

// or

options: {
  parse: (response) => response as {usernameTaken: boolean; profanity: boolean};
}
onSuccess: (response) => {
  // ...
} // prettier-ignore
```

La fonction `onError` gère les échecs de requête comme les erreurs réseau ou les erreurs HTTP :

```ts
onError: (error) => {
  console.error('Validation request failed:', error);
  return {
    kind: 'serverError',
    message: 'Could not verify. Please try again later.',
  };
} // prettier-ignore
```

### Options HTTP {#http-options}

Personnalisez la requête HTTP en renvoyant un objet `HttpResourceRequest` depuis la fonction `request` :

```ts
import {HttpHeaders} from '@angular/common/http';
import {validateHttp} from '@angular/forms/signals';

validateHttp(schemaPath.field, {
  request: ({value}) => ({
    url: `/api/validate?value=${value()}`,
    headers: new HttpHeaders({
      Authorization: 'Bearer token',
    }),
    timeout: 5000,
  }),
  onSuccess: (response: {valid: boolean}) =>
    response.valid
      ? null
      : {
          kind: 'invalid',
          message: 'Invalid value',
        },
  onError: () => ({
    kind: 'requestFailed',
    message: 'Unable to reach server to validate.',
  }),
});
```

TIP: Consultez la [documentation de l'API httpResource](api/common/http/httpResource) pour toutes les options disponibles.

## Validation asynchrone personnalisée avec validateAsync() {#custom-async-validation-with-validateasync}

La plupart des applications devraient utiliser `validateHttp()` pour la validation asynchrone. Cette fonction gère les requêtes HTTP avec une configuration minimale et couvre la majorité des cas d'usage.

`validateAsync()` est une API de plus bas niveau qui expose directement la primitive resource d'Angular. Elle offre un contrôle complet, mais demande plus de code et une bonne connaissance de l'API resource d'Angular.

N'envisagez `validateAsync()` que lorsque `validateHttp()` ne répond pas à vos besoins. Voici quelques exemples :

- **Validation hors HTTP** - Connexions WebSocket, recherches IndexedDB ou calculs dans un Web Worker
- **Stratégies de mise en cache personnalisées** - Mise en cache propre à l'application, au-delà d'une simple mémoïsation
- **Logique de nouvelle tentative complexe** - Stratégies de backoff personnalisées ou nouvelles tentatives conditionnelles
- **Accès direct à la resource** - Lorsque vous avez besoin du cycle de vie complet de la resource

### Créer une règle de validation personnalisée {#creating-a-custom-validation-rule}

La fonction `validateAsync()` nécessite quatre propriétés : `params`, `factory`, `onSuccess` et `onError`. La fonction `params` renvoie les paramètres de votre resource, tandis que `factory` crée la resource :

```ts
import {Component, inject, signal, resource, Signal} from '@angular/core';
import {form, validateAsync, FormField} from '@angular/forms/signals';
import {UsernameValidator} from './username-validator';

@Component({
  selector: 'app-registration',
  imports: [FormField],
  template: `...`,
})
export class Registration {
  registrationModel = signal({username: ''});

  private usernameValidator = inject(UsernameValidator);
  private cache = new Map<string, {available: boolean}>();

  // Custom resource factory with caching
  createUsernameResource = (usernameSignal: Signal<string | undefined>) => {
    return resource({
      params: () => usernameSignal(),
      loader: async ({params: username}) => {
        if (!username) return undefined;

        // Check cache first
        const cached = this.cache.get(username);
        if (cached !== undefined) return cached;

        // Use injected service for validation
        const result = await this.usernameValidator.checkAvailability(username);

        // Cache result
        this.cache.set(username, result);
        return result;
      },
    });
  };

  registrationForm = form(this.registrationModel, (schemaPath) => {
    validateAsync(schemaPath.username, {
      params: ({value}) => {
        const username = value();
        return username.length >= 3 ? username : undefined!;
      },
      factory: this.createUsernameResource,
      onSuccess: (result) => {
        return result?.available
          ? null
          : {
              kind: 'usernameTaken',
              message: 'Username taken',
            };
      },
      onError: (error) => {
        console.error('Validation failed:', error);
        return {
          kind: 'serverError',
          message: 'Could not verify username',
        };
      },
    });
  });
}
```

La fonction `params` s'exécute à chaque changement de valeur. Renvoyez `undefined` pour ignorer la validation. La fonction `factory` s'exécute une seule fois lors de la mise en place et reçoit les paramètres sous forme de signal. La resource se met à jour automatiquement lorsque les paramètres changent.

### Utiliser des services basés sur des observables {#using-observable-based-services}

Si votre application dispose de services existants qui renvoient un observable, utilisez `rxResource` depuis `@angular/core/rxjs-interop` :

```ts
import {Component, inject, signal, Signal} from '@angular/core';
import {rxResource} from '@angular/core/rxjs-interop';
import {form, validateAsync, FormField} from '@angular/forms/signals';
import {UsernameService} from './username-service';

@Component({
  selector: 'app-registration',
  imports: [FormField],
  template: `...`,
})
export class Registration {
  registrationModel = signal({username: ''});

  private usernameService = inject(UsernameService);

  private createUsernameResource = (usernameSignal: Signal<string | undefined>) => {
    return rxResource({
      params: () => usernameSignal(),
      stream: ({params: username}) => this.usernameService.checkUsername(username),
    });
  };

  registrationForm = form(this.registrationModel, (schemaPath) => {
    validateAsync(schemaPath.username, {
      params: ({value}) => value(),
      factory: this.createUsernameResource,
      onSuccess: (result) =>
        result?.available ? null : {kind: 'usernameTaken', message: 'Username taken'},
      onError: () => ({
        kind: 'serverError',
        message: 'Could not verify username',
      }),
    });
  });
}
```

La fonction `rxResource` fonctionne directement avec les observables et gère automatiquement le nettoyage de l'abonnement lorsque la valeur du champ change.

## Debounce {#debouncing}

La règle `debounce` retarde le moment où la saisie de l'utilisateur est validée dans le model de formulaire. Vous pouvez la voir comme une règle qui retient les valeurs jusqu'à ce que l'utilisateur fasse une pause dans sa saisie. C'est utile lorsque le comportement en aval ne doit pas réagir à chaque frappe, par exemple pour des calculs dérivés coûteux, une validation qui affiche des erreurs en plein milieu d'un mot, ou des filtres de recherche réappliqués à chaque caractère.

Ajoutez la règle `debounce` dans un schéma pour retarder la propagation des changements de l'interface d'un champ vers le model de formulaire. Dans sa forme la plus simple, `debounce(path, ms)` retient chaque changement de l'interface pendant le nombre de millisecondes indiqué avant de l'écrire dans le model. Un nouveau changement dans cet intervalle réinitialise le minuteur.

L'exemple suivant applique `debounce` et `validateHttp` au champ du nom d'utilisateur afin de retarder la vérification de disponibilité dans un formulaire d'inscription jusqu'à ce que l'utilisateur fasse une pause dans sa saisie :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, debounce, validateHttp, FormField} from '@angular/forms/signals';

@Component({
  selector: 'app-registration',
  imports: [FormField],
  template: `
    <label>
      Username:
      <input [formField]="registrationForm.username" />
    </label>

    @if (registrationForm.username().pending()) {
      <span class="checking">Checking availability...</span>
    }
  `,
})
export class Registration {
  registrationModel = signal({username: ''});

  registrationForm = form(this.registrationModel, (schemaPath) => {
    // Hold UI updates for 300 ms before writing to the model
    debounce(schemaPath.username, 300);

    // Runs against the debounced model value, not every keystroke
    validateHttp(schemaPath.username, {
      request: ({value}) => {
        const username = value();
        // Skip the request for blank values
        return username ? `/api/users/check?username=${username}` : undefined;
      },
      onSuccess: (response: {available: boolean}) =>
        response.available ? null : {kind: 'usernameTaken', message: 'Username is already taken'},
      onError: () => ({
        kind: 'serverError',
        message: 'Could not verify username availability',
      }),
    });
  });
}
```

Avec un debounce de 300 ms, le model n'est mis à jour et validé qu'une fois que l'utilisateur a interrompu sa saisie plus longtemps que la durée configurée. Par exemple, taper « signal forms » d'une traite déclenche une seule requête de validation au lieu de douze.

### Le passage à l'état touché valide le model {#touch-flushes-the-model}

Quelle que soit la durée du debounce, le framework écrit immédiatement la `controlValue()` du champ dans le model lorsque le champ devient touché. Les champs natifs deviennent touchés à la perte du focus (blur) : un utilisateur qui termine sa saisie et passe au champ suivant avec Tab n'a donc pas à attendre l'expiration du minuteur de debounce. Les contrôles personnalisés peuvent marquer le champ comme touché en réponse à n'importe quel événement de leur choix.

Dans le cas typique, cela compte pour la soumission du formulaire. Lorsque l'utilisateur clique sur un bouton de soumission, le champ qui a le focus le perd, ce qui marque ce champ comme touché et applique son debounce en attente avant que le gestionnaire de soumission ne s'exécute.

### Valider uniquement au blur {#commit-only-on-blur}

Certains champs ne devraient pas du tout se mettre à jour pendant la saisie, mais uniquement une fois que l'utilisateur a fini d'entrer une valeur. Par exemple, si vous avez un filtre de recherche réappliqué à chaque changement, ou un formulaire qui déclenche un état dérivé coûteux, il est souvent préférable que le model attende que l'utilisateur ait fini de taper.

Dans ces scénarios, passez `'blur'` au lieu d'une durée pour différer toutes les mises à jour jusqu'à ce que le champ devienne touché :

```ts
form(this.registrationModel, (schemaPath) => {
  debounce(schemaPath.username, 'blur');
});
```

Avec `'blur'`, le model conserve sa valeur précédente pendant que l'utilisateur tape. La validation synchrone et asynchrone, les signals dérivés et toutes les règles réactives qui lisent le champ voient la valeur précédente jusqu'à ce que le champ devienne touché. Cela se produit généralement lorsque l'utilisateur quitte un champ natif (blur), ou lorsqu'un contrôle personnalisé signale lui-même le passage à l'état touché.

### Logique de temporisation personnalisée {#custom-timing-logic}

Pour une logique de temporisation qu'une durée ou `'blur'` ne peut pas exprimer, passez une fonction `Debouncer`. La fonction reçoit le contexte du champ et un [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal), et renvoie une `Promise<void>` qui se résout lorsque le model doit être mis à jour :

```ts
import {debounce, type Debouncer, form} from '@angular/forms/signals';

const shorterWhenLonger: Debouncer<string> = ({value}, abortSignal) => {
  // Shorter queries get a longer delay since the user is likely still typing.
  const ms = value().length < 3 ? 500 : 200;
  return new Promise((resolve) => {
    const timeoutId = setTimeout(resolve, ms);
    // Abort fires when this field is touched or its value changes, so the pending timer is cleared
    abortSignal.addEventListener(
      'abort',
      () => {
        clearTimeout(timeoutId);
        resolve();
      },
      {once: true},
    );
  });
};

const registrationForm = form(registrationModel, (schemaPath) => {
  debounce(schemaPath.username, shorterWhenLonger);
});
```

L'`abortSignal` se déclenche lorsque le champ devient touché, ou lorsque sa valeur change avant que le debounce ne se résolve. Résolvez la promise lors de l'abandon afin que votre debouncer libère les minuteurs en attente. Le framework écrit la valeur en attente dans le model lors du passage à l'état touché, et l'abandonne lorsqu'une valeur plus récente arrive. Consultez la [référence de l'API `debounce`](api/forms/signals/debounce) pour la signature complète de `Debouncer`.

### Appliquer un debounce à un seul validateur asynchrone {#debouncing-a-single-async-validator}

La règle `debounce` retient toutes les réactions au champ, de la validation synchrone aux signals dérivés en passant par la validation asynchrone. Il arrive pourtant que vous vouliez l'inverse : des validateurs synchrones peu coûteux comme `required` ou `email` qui s'exécutent immédiatement pour un retour instantané, tandis que seul l'appel asynchrone coûteux attend que l'utilisateur se stabilise. `validateHttp()` et `validateAsync()` acceptent tous deux leur propre [option `debounce`](api/forms/signals/validateAsync) qui limite uniquement ce validateur :

```ts
form(this.registrationModel, (schemaPath) => {
  validateHttp(schemaPath.username, {
    // Throttles only this HTTP call
    debounce: 300,
    request: ({value}) => {
      const username = value();
      // Skip the request for blank values
      return username ? `/api/users/check?username=${username}` : undefined;
    },
    onSuccess: (response: {available: boolean}) =>
      response.available ? null : {kind: 'usernameTaken', message: 'Username is already taken'},
    onError: () => ({
      kind: 'serverError',
      message: 'Could not verify username availability',
    }),
  });
});
```

Le model est toujours mis à jour à chaque frappe, et les autres règles attachées au champ réagissent toujours immédiatement. Seule la requête HTTP est soumise au debounce : chaque changement attend 300 ms sans activité avant de se déclencher, si bien qu'une requête n'est envoyée qu'une fois que l'utilisateur a fait une pause dans sa saisie.

Choisissez entre les deux niveaux selon la portée :

| Option                                                        | Quand l'utiliser                                                                                                                    |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| Règle `debounce()`                                            | La validation synchrone, l'état dérivé et la soumission doivent tous attendre que le champ soit validé. Le champ entier ne doit pas réagir pendant la saisie. |
| `validateHttp({ debounce })` ou `validateAsync({ debounce })` | Les validateurs synchrones peu coûteux doivent donner un retour immédiat, mais les appels asynchrones coûteux doivent attendre que l'utilisateur fasse une pause. |

Les deux options acceptent une durée en millisecondes. Leurs callbacks de temporisation personnalisée diffèrent : la règle au niveau du formulaire prend un `Debouncer`, et l'option au niveau du validateur prend un `DebounceTimer` de `@angular/core`. Les deux signatures ne sont pas interchangeables.

## Composer des resources dans la validation asynchrone avec une fabrique {#composing-resources-in-async-validation-with-a-factory}

L'[option `debounce`](api/forms/signals/validateAsync) intégrée couvre la limitation des appels, mais `validateAsync()` expose un point de composition plus profond : la fonction `factory`. La fabrique reçoit les paramètres sous forme de signal et renvoie une resource. Entre ces deux points, vous êtes libre de composer tout ce dont vous avez besoin.

Dans sa forme la plus simple, une fabrique encapsule une seule resource. Une vérification de disponibilité du nom d'utilisateur peut être définie comme une méthode de la classe du composant, puis branchée dans `validateAsync` par référence :

```ts
export class Registration {
  registrationModel = signal({username: ''});
  private usernameValidator = inject(UsernameValidator);

  // Factory function
  checkUsernameAvailable = (username: Signal<string | undefined>) =>
    resource({
      params: () => username(),
      loader: async ({params: name}) => this.usernameValidator.checkAvailability(name),
    });

  registrationForm = form(this.registrationModel, (schemaPath) => {
    validateAsync(schemaPath.username, {
      params: ({value}) => {
        const username = value();
        // Skip validation for short usernames
        return username.length >= 3 ? username : undefined!;
      },
      debounce: 300,
      // Reference to the factory defined above
      factory: this.checkUsernameAvailable,
      onSuccess: (result) =>
        result?.available ? null : {kind: 'usernameTaken', message: 'Username taken'},
      onError: () => ({kind: 'serverError', message: 'Could not verify'}),
    });
  });
}
```

Le callback `params` renvoie `undefined` pour les noms d'utilisateur courts, signalant ainsi que la validation doit être ignorée. Avec `debounce: 300`, la resource attend que l'utilisateur fasse une pause de 300 ms dans sa saisie avant d'agir sur chaque changement. Elle exécute ensuite le loader pour les noms d'utilisateur valides et reste inactive une fois que la valeur debouncée se stabilise à `undefined`.

### Combiner le debounce avec une logique supplémentaire {#combining-debounce-with-additional-logic}

Lorsque vous avez besoin d'une logique qui va au-delà d'un simple debounce basé sur une durée, utilisez une fabrique personnalisée pour combiner le debounce avec cette logique. Un cas courant est la mise en cache des réponses validées. Par exemple, une fois que le serveur a confirmé un nom d'utilisateur, vous n'avez pas besoin de lui redemander lors des frappes suivantes qui reviennent à la même valeur.

```ts
export class Registration {
  registrationModel = signal({username: ''});
  private usernameValidator = inject(UsernameValidator);

  registrationForm = form(this.registrationModel, (schemaPath) => {
    validateAsync(schemaPath.username, {
      params: ({value}) => {
        const username = value();
        return username.length >= 3 ? username : undefined;
      },
      factory: (username) => {
        // Core primitive: settles 300 ms after the source stops changing
        const debouncedUsername = debounced(username, 300);
        // Cache lives in the factory's closure and persists for the field's lifetime
        const cache = new Map<string, {available: boolean}>();
        return resource({
          // Read from the debounced signal, not the raw one
          params: () => debouncedUsername.value(),
          loader: async ({params: name}) => {
            const cached = cache.get(name);
            if (cached) return cached;

            const result = await this.usernameValidator.checkAvailability(name);
            cache.set(name, result);
            return result;
          },
        });
      },
      onSuccess: (result) =>
        result?.available ? null : {kind: 'usernameTaken', message: 'Username taken'},
      onError: () => ({
        kind: 'serverError',
        message: 'Could not verify username',
      }),
    });
  });
}
```

Le `cache` réside dans la closure de la fabrique : il persiste donc pendant toute la durée de vie du champ. Lorsque l'utilisateur saisit un nom d'utilisateur que le serveur a déjà vérifié, le loader lit le cache au lieu d'effectuer une nouvelle requête réseau.

## Comprendre l'état en attente {#understanding-pending-state}

Pendant l'exécution de la validation asynchrone, le signal `pending()` du champ renvoie `true`. Pendant ce temps :

- `valid()` renvoie `false`
- `invalid()` renvoie `false`
- `errors()` renvoie un tableau vide
- `submit()` ne l'attend pas ; par défaut, l'`action` s'exécute quand même (voir [`ignoreValidators`](guide/forms/signals/form-submission#controlling-validation-gating-with-ignorevalidators))

Affichez l'état en attente dans votre template pour fournir un retour à l'utilisateur :

```angular-html
<input [formField]="loginForm.username" />

@if (loginForm.username().pending()) {
  <span class="loading">Checking availability...</span>
}

@if (loginForm.username().touched() && loginForm.username().invalid()) {
  @for (error of loginForm.username().errors(); track $index) {
    <span class="error">{{ error.message }}</span>
  }
}
```

Désactivez la soumission du formulaire pendant que la validation est en attente :

```angular-html
<button type="submit" [disabled]="loginForm().pending()">
  @if (loginForm().pending()) {
    Validating...
  } @else {
    Submit
  }
</button>
```

TIP: Consultez le [guide de gestion de l'état des champs](guide/forms/signals/field-state-management) pour d'autres patterns utilisant les signals `pending()`, `valid()` et `invalid()`.

### Ordre d'exécution de la validation {#validation-execution-order}

La validation asynchrone ne s'exécute qu'une fois la validation synchrone réussie. Cela évite des requêtes serveur inutiles pour des saisies invalides :

```ts
import {form, required, minLength, validateHttp} from '@angular/forms/signals';
import {model} from '@angular/core';

form(model, (schemaPath) => {
  // 1. These synchronous validation rules run first
  required(schemaPath.username);
  minLength(schemaPath.username, 3);

  // 2. This async validation rule only runs if synchronous validation passes
  validateHttp(schemaPath.username, {
    request: ({value}) => `/api/check?username=${value()}`,
    onSuccess: (result: {valid: boolean}) =>
      result.valid
        ? null
        : {
            kind: 'usernameTaken',
            message: 'Username taken',
          },
    onError: () => ({
      kind: 'serverError',
      message: 'Validation failed',
    }),
  });
});
```

Cet ordre d'exécution améliore les performances en réduisant la charge serveur et en détectant instantanément les erreurs de format.

### Annulation des requêtes {#request-cancellation}

Lorsque la valeur d'un champ change, les Signal Forms annulent automatiquement toute requête de validation asynchrone en attente pour ce champ. Cela évite les race conditions et garantit que la validation reflète toujours la valeur actuelle. Vous n'avez pas besoin d'implémenter vous-même la logique d'annulation.

## Bonnes pratiques {#best-practices}

### Combiner avec la validation synchrone {#combine-with-synchronous-validation}

Validez toujours le format avant d'effectuer des requêtes asynchrones. Cela détecte les erreurs instantanément et évite des requêtes serveur inutiles :

```ts
import {form, required, email, validateHttp} from '@angular/forms/signals';
import {model} from '@angular/core';

form(model, (schemaPath) => {
  // Validate format first
  required(schemaPath.email);
  email(schemaPath.email);

  // Then check availability
  validateHttp(schemaPath.email, {
    request: ({value}) => `/api/emails/check?email=${value()}`,
    onSuccess: (result: {available: boolean}) =>
      result.available
        ? null
        : {
            kind: 'emailInUse',
            message: 'Email already in use',
          },
    onError: () => ({
      kind: 'serverError',
      message: 'Could not verify email',
    }),
  });
});
```

### Ignorer la validation lorsque c'est pertinent {#skip-validation-when-appropriate}

Renvoyez `undefined` depuis la fonction `request` pour ignorer la validation. Utilisez cette technique pour éviter de valider des champs vides ou des valeurs qui ne respectent pas les exigences minimales :

```ts
import {validateHttp} from '@angular/forms/signals';

validateHttp(schemaPath.username, {
  request: ({value}) => {
    const username = value();
    // Skip validation for empty or short usernames
    if (!username || username.length < 3) return undefined;

    return `/api/users/check?username=${username}`;
  },
  onSuccess: (result: {valid: boolean}) =>
    result.valid
      ? null
      : {
          kind: 'usernameTaken',
          message: 'Username taken',
        },
  onError: () => ({
    kind: 'serverError',
    message: 'Validation failed',
  }),
});
```

### Gérer les erreurs avec élégance {#handle-errors-gracefully}

Fournissez des messages d'erreur clairs et compréhensibles pour l'utilisateur. Journalisez les détails techniques pour le débogage, mais affichez des messages simples aux utilisateurs :

```ts
import {validateHttp} from '@angular/forms/signals';

validateHttp(schemaPath.field, {
  request: ({value}) => `/api/validate?field=${value()}`,
  onSuccess: (result: {valid: boolean; message?: string}) => {
    if (result.valid) return null;
    // Use server message when available
    return {
      kind: 'serverError',
      message: result.message || 'Validation failed',
    };
  },
  onError: (error) => {
    // Log for debugging
    console.error('Validation request failed:', error);

    // Show user-friendly message
    return {
      kind: 'serverError',
      message: 'Unable to validate. Please try again later.',
    };
  },
});
```

### Afficher un retour clair {#show-clear-feedback}

Utilisez le signal `pending()` pour indiquer qu'une validation est en cours. Cela aide les utilisateurs à comprendre les délais et améliore les performances perçues :

```angular-html
@if (field().pending()) {
  <span class="checking">
    <span class="spinner"></span>
    Checking...
  </span>
}
@if (field().valid() && !field().pending()) {
  <span class="success">Available</span>
}
@if (field().invalid()) {
  <span class="error">{{ field().errors()[0]?.message }}</span>
}
```

## Prochaines étapes {#next-steps}

Ce guide a présenté la validation asynchrone avec `validateHttp()` et `validateAsync()`. Des guides connexes explorent d'autres aspects des Signal Forms :

<docs-pill-row>
  <docs-pill href="guide/forms/signals/validation" title="Validation"/>
  <docs-pill href="guide/forms/signals/field-state-management" title="Gestion de l'état des champs"/>
</docs-pill-row>

Pour une documentation détaillée de l'API, consultez :

- [`validateHttp()`](api/forms/signals/validateHttp) - Validation asynchrone basée sur HTTP
- [`validateAsync()`](api/forms/signals/validateAsync) - Validation asynchrone personnalisée basée sur une resource
- [`httpResource()`](api/common/http/httpResource) - L'API de resource HTTP d'Angular
- [`resource()`](api/core/resource) - La primitive resource d'Angular
