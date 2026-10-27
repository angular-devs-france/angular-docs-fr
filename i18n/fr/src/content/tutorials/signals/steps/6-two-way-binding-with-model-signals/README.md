<!-- source: 6050acb661a17a35b04ee9ff44ec934fec002075 -->
# Two-way binding avec les signals model

Maintenant que vous avez appris [à passer des données aux composants avec les inputs signal](/tutorials/signals/5-component-communication-with-signals), découvrons l'API `model()` d'Angular pour le two-way binding. Les signals model sont parfaits pour les composants d'interface comme les cases à cocher, les curseurs ou les contrôles de formulaire personnalisés, où le composant doit à la fois recevoir une valeur ET la mettre à jour.

Dans cette activité, vous allez créer un composant de case à cocher personnalisé qui gère son propre état tout en gardant le parent synchronisé.

<hr />

<docs-workflow>

<docs-step title="Mettre en place la case à cocher personnalisée avec un signal model {#set-up-the-custom-checkbox-with-model-signal}">
Créez dans le composant `custom-checkbox` un signal model qui peut à la fois recevoir et mettre à jour la valeur du parent.

```ts
// Add imports for model signals
import {Component, model, input, ChangeDetectionStrategy} from '@angular/core';

// Model signal for two-way binding
checked = model.required<boolean>();

// Optional input for label
label = input<string>('');
```

Contrairement aux signals `input()`, qui sont en lecture seule, les signals `model()` peuvent être à la fois lus et modifiés.
</docs-step>

<docs-step title="Créer le template de la case à cocher {#create-the-checkbox-template}">
Construisez le template de la case à cocher, qui réagit aux clics et met à jour son propre model.

```html
<label class="custom-checkbox">
  <input type="checkbox" [checked]="checked()" (change)="toggle()" />
  <span class="checkmark"></span>
  {{ label() }}
</label>
```

Le composant lit son signal model et dispose d'une méthode pour le mettre à jour.
</docs-step>

<docs-step title="Ajouter la méthode de bascule {#add-the-toggle-method}">
Implémentez la méthode de bascule qui met à jour le signal model quand la case est cliquée.

```ts
toggle() {
  // This updates BOTH the component's state AND the parent's model!
  this.checked.set(!this.checked());
}
```

Quand le composant enfant appelle `this.checked.set()`, le changement est automatiquement propagé au parent. C'est la différence essentielle avec les signals `input()`.
</docs-step>

<docs-step title="Mettre en place le two-way binding dans le parent {#set-up-two-way-binding-in-the-parent}">
Commencez par décommenter les propriétés et méthodes des signals model dans `app.ts` :

```ts
// Parent signal models
agreedToTerms = model(false);
enableNotifications = model(true);

// Methods to test two-way binding
toggleTermsFromParent() {
  this.agreedToTerms.set(!this.agreedToTerms());
}

resetAll() {
  this.agreedToTerms.set(false);
  this.enableNotifications.set(false);
}
```

Mettez ensuite à jour le template :

Partie 1. **Décommentez les cases à cocher et ajoutez le two-way binding :**

- Remplacez `___ADD_TWO_WAY_BINDING___` par `[(checked)]="agreedToTerms"` pour la première case à cocher
- Remplacez `___ADD_TWO_WAY_BINDING___` par `[(checked)]="enableNotifications"` pour la seconde

Partie 2. **Remplacez les placeholders `???` par des blocs @if :**

```angular-html
@if (agreedToTerms()) {
  Yes
} @else {
  No
}

@if (enableNotifications()) {
  Yes
} @else {
  No
}
```

Partie 3. **Ajoutez des gestionnaires de clic aux boutons :**

```html
<button (click)="toggleTermsFromParent()">Toggle Terms from Parent</button>
<button (click)="resetAll()">Reset All</button>
```

La syntaxe `[(checked)]` crée un two-way binding : les données descendent vers le composant ET les changements remontent vers le parent via l'émission d'un événement qui référence le signal lui-même, sans appeler _directement_ le getter du signal.
</docs-step>

<docs-step title="Tester le two-way binding {#test-the-two-way-binding}">
Interagissez avec votre application pour voir le two-way binding en action :

1. **Cliquez sur les cases à cocher** : le composant met à jour son propre état et notifie le parent
2. **Cliquez sur « Toggle Terms from Parent »** : les mises à jour du parent se propagent jusqu'au composant
3. **Cliquez sur « Reset All »** : le parent réinitialise les deux models et les composants se mettent à jour automatiquement

Le parent comme l'enfant peuvent mettre à jour l'état partagé, et les deux restent automatiquement synchronisés !
</docs-step>

</docs-workflow>

Parfait ! Vous avez appris comment les signals model permettent le two-way binding :

- **Signals model** : utilisez `model()` et `model.required()` pour les valeurs qui peuvent être à la fois lues et modifiées
- **Two-way binding** : utilisez la syntaxe `[(property)]` pour lier les signals du parent aux models de l'enfant
- **Parfaits pour les composants d'interface** : cases à cocher, contrôles de formulaire et widgets qui doivent gérer leur propre état
- **Synchronisation automatique** : le parent et l'enfant restent synchronisés sans gestion manuelle des événements

**Quand utiliser `model()` plutôt que `input()` :**

- Utilisez `input()` pour les données qui ne font que descendre (données affichées, configuration)
- Utilisez `model()` pour les composants d'interface qui doivent mettre à jour leur propre valeur (contrôles de formulaire, interrupteurs)

Dans la prochaine leçon, vous apprendrez [à utiliser les signals avec les services](/tutorials/signals/7-using-signals-with-services) !
