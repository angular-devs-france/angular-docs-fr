<!-- source: 6d64637494deea86467501da78b2e392109c2e56 -->
<docs-decorative-header title="Signals" imgSrc="adev/src/assets/images/signals.svg"> <!-- markdownlint-disable-line -->
Créer et gérer des données dynamiques.
</docs-decorative-header>

Dans Angular, vous utilisez des _signals_ pour créer et gérer l'état. Un signal est une enveloppe légère autour d'une valeur.

Utilisez la fonction `signal` pour créer un signal qui contient un état local :

```typescript
import {signal} from '@angular/core';

// Create a signal with the `signal` function.
const firstName = signal('Morgan');

// Read a signal value by calling it— signals are functions.
console.log(firstName());

// Change the value of this signal by calling its `set` method with a new value.
firstName.set('Jaime');

// You can also use the `update` method to change the value
// based on the previous value.
firstName.update((name) => name.toUpperCase());
```

Angular suit l'endroit où les signals sont lus et le moment où ils sont mis à jour. Le framework utilise ces informations pour effectuer du travail supplémentaire, comme mettre à jour le DOM avec le nouvel état. Cette capacité à réagir aux changements de valeur des signals au fil du temps s'appelle la _réactivité_.

## Expressions computed {#computed-expressions}

Un `computed` est un signal qui produit sa valeur à partir d'autres signals.

```typescript
import {signal, computed} from '@angular/core';

const firstName = signal('Morgan');
const firstNameCapitalized = computed(() => firstName().toUpperCase());

console.log(firstNameCapitalized()); // MORGAN
```

Un signal `computed` est en lecture seule ; il n'a pas de méthode `set` ni `update`. À la place, la valeur du signal `computed` change automatiquement quand l'un des signals qu'il lit change :

```typescript
import {signal, computed} from '@angular/core';

const firstName = signal('Morgan');
const firstNameCapitalized = computed(() => firstName().toUpperCase());
console.log(firstNameCapitalized()); // MORGAN

firstName.set('Jaime');
console.log(firstNameCapitalized()); // JAIME
```

## Utiliser les signals dans les composants {#using-signals-in-components}

Utilisez `signal` et `computed` dans vos composants pour créer et gérer l'état :

```ts
@Component({
  /* ... */
})
export class UserProfile {
  isTrial = signal(false);
  isTrialExpired = signal(false);
  showTrialDuration = computed(() => this.isTrial() && !this.isTrialExpired());

  activateTrial() {
    this.isTrial.set(true);
  }
}
```

TIP: Vous voulez en savoir plus sur les Angular Signals ? Consultez le [guide approfondi des signals](guide/signals) pour tous les détails.

## Étape suivante {#next-step}

Maintenant que vous savez déclarer et gérer des données dynamiques, il est temps d'apprendre à utiliser ces données dans les templates.

<docs-pill-row>
  <docs-pill title="Des interfaces dynamiques avec les templates" href="essentials/templates" />
  <docs-pill title="Guide approfondi des signals" href="guide/signals" />
</docs-pill-row>
