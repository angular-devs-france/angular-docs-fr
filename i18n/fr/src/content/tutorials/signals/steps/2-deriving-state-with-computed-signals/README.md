<!-- source: f581a713e024e34b45229db4db0263734e810b06 -->
# Dériver un état avec des signals computed

Maintenant que vous avez appris [à créer et mettre à jour des signals](/tutorials/signals/1-creating-your-first-signal), découvrons les signals computed. Les signals computed sont des valeurs dérivées qui se mettent à jour automatiquement quand leurs dépendances changent. Ils sont parfaits pour créer des calculs réactifs basés sur d'autres signals.

Dans cette activité, vous allez apprendre à utiliser la fonction `computed()` pour créer un état dérivé qui se met à jour automatiquement quand les signals sous-jacents changent.

Enrichissons notre système de statut utilisateur en ajoutant des valeurs computed qui dérivent des informations de notre signal de statut utilisateur. Le code de départ inclut désormais trois options de statut : `'online'`, `'away'` et `'offline'`.

<hr />

<docs-workflow>

<docs-step title="Importer la fonction computed {#import-computed-function}">
Ajoutez `computed` à vos imports existants.

```ts
// Add computed to existing imports
import {Component, signal, computed, ChangeDetectionStrategy} from '@angular/core';
```

</docs-step>

<docs-step title="Créer un signal computed pour les notifications {#create-a-computed-signal-for-notifications}">
Ajoutez un signal computed qui détermine si les notifications doivent être activées en fonction du statut de l'utilisateur.

```ts
notificationsEnabled = computed(() => this.userStatus() === 'online');
```

Ce signal computed se recalcule automatiquement chaque fois que le signal `userStatus` change. Remarquez que nous appelons `this.userStatus()` dans la fonction computed pour lire la valeur du signal.
</docs-step>

<docs-step title="Créer un signal computed pour un message descriptif {#create-a-computed-signal-for-a-descriptive-message}">
Ajoutez un signal computed qui crée un message descriptif en fonction du statut de l'utilisateur.

```ts
statusMessage = computed(() => {
  const status = this.userStatus();
  switch (status) {
    case 'online':
      return 'Available for meetings and messages';
    case 'away':
      return 'Temporarily away, will respond soon';
    case 'offline':
      return 'Not available, check back later';
    default:
      return 'Status unknown';
  }
});
```

Cela montre comment les signals computed peuvent gérer une logique plus complexe avec des instructions switch et des transformations de chaînes.
</docs-step>

<docs-step title="Créer un signal computed qui calcule la disponibilité pendant les heures de travail {#create-a-computed-signal-that-calculates-working-hours-availability}">
Ajoutez un signal computed qui calcule si l'utilisateur se trouve dans ses heures de travail.

```ts
isWithinWorkingHours = computed(() => {
  const now = new Date();
  const hour = now.getHours();
  const isWeekday = now.getDay() > 0 && now.getDay() < 6;
  return this.userStatus() !== 'offline' && isWeekday && hour >= 9 && hour < 17;
});
```

Cela montre comment les signals computed peuvent effectuer des calculs et combiner plusieurs sources de données. La valeur se met à jour automatiquement quand `userStatus` change.
</docs-step>

<docs-step title="Afficher les valeurs computed dans le template {#display-the-computed-values-in-the-template}">
Le template contient déjà des placeholders qui affichent « Loading... ». Remplacez-les par vos signals computed :

1. Pour les notifications, remplacez `Loading...` par un bloc `@if` :

   ```angular-html
   @if (notificationsEnabled()) {
     Enabled
   } @else {
     Disabled
   }
   ```

1. Pour le message, remplacez `Loading...` par :

   ```angular-html
   {{ statusMessage() }}
   ```

1. Pour les heures de travail, remplacez `Loading...` par un bloc `@if` :

   ```angular-html
   @if (isWithinWorkingHours()) {
     Yes
   } @else {
     No
   }
   ```

Remarquez que les signals computed s'appellent exactement comme les signals classiques, avec des parenthèses !
</docs-step>

</docs-workflow>

Excellent ! Vous savez maintenant créer des signals computed.

Voici quelques points clés à retenir :

- **Les signals computed sont réactifs** : ils se mettent à jour automatiquement quand leurs dépendances changent
- **Ils sont en lecture seule** : vous ne pouvez pas définir directement des valeurs computed, elles sont dérivées d'autres signals
- **Ils peuvent contenir une logique complexe** : utilisez-les pour des calculs, des transformations et de l'état dérivé
- **Ils permettent des calculs performants basés sur un état dynamique** : Angular ne les recalcule que lorsque leurs dépendances changent réellement

Dans la prochaine leçon, vous découvrirez [une autre façon de dériver un état avec les linked signals](/tutorials/signals/3-deriving-state-with-linked-signals) !
