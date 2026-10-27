<!-- source: f1980a4063ab74d87ab5688f1586792f59085ec9 -->
# Dériver un état avec des linked signals

Maintenant que vous avez appris [à dériver un état avec des signals computed](/tutorials/signals/2-deriving-state-with-computed-signals), vous avez créé un signal computed `notificationsEnabled` qui suivait automatiquement le statut de l'utilisateur. Mais que se passe-t-il si les utilisateurs veulent désactiver manuellement les notifications, même lorsqu'ils sont en ligne ? C'est là qu'interviennent les linked signals.

Les linked signals sont des signals modifiables qui maintiennent une connexion réactive avec leurs signals sources. Ils sont parfaits pour créer un état qui suit normalement un calcul mais peut être remplacé si nécessaire.

Dans cette activité, vous allez apprendre en quoi `linkedSignal()` diffère de `computed()` en transformant le signal computed `notificationsEnabled` du système de statut utilisateur précédent en un linked signal modifiable.

<hr />

<docs-workflow>

<docs-step title="Importer la fonction linkedSignal {#import-linkedsignal-function}">
Ajoutez `linkedSignal` à vos imports existants.

```ts
// Add linkedSignal to existing imports
import {Component, signal, computed, linkedSignal, ChangeDetectionStrategy} from '@angular/core';
```

</docs-step>

<docs-step title="Convertir computed en linkedSignal avec la même expression {#convert-computed-to-linkedsignal-with-the-same-expression}">
Remplacez le signal computed `notificationsEnabled` par un linkedSignal utilisant exactement la même expression :

```ts
// Previously (from lesson 2):
// notificationsEnabled = computed(() => this.userStatus() === 'online');

// Now with linkedSignal - same expression, but writable:
notificationsEnabled = linkedSignal(() => this.userStatus() === 'online');
```

L'expression est identique, mais linkedSignal crée un signal modifiable. Il se met toujours à jour automatiquement quand `userStatus` change, mais vous pouvez aussi le définir manuellement.
</docs-step>

<docs-step title="Ajouter une méthode pour basculer manuellement les notifications {#add-a-method-to-manually-toggle-notifications}">
Ajoutez une méthode pour montrer qu'il est possible d'écrire directement dans les linked signals :

```ts
toggleNotifications() {
  // This works with linkedSignal but would error with computed!
  this.notificationsEnabled.set(!this.notificationsEnabled());
}
```

C'est la différence essentielle : les signals computed sont en lecture seule, alors que les linked signals peuvent être mis à jour manuellement tout en conservant leur connexion réactive.
</docs-step>

<docs-step title="Mettre à jour le template pour contrôler manuellement les notifications {#update-the-template-to-add-manual-notification-control}">
Mettez à jour votre template pour ajouter un bouton de bascule pour les notifications :

```angular-html
<div class="status-info">
  <div class="notifications">
    <strong>Notifications:</strong>
    @if (notificationsEnabled()) {
      Enabled
    } @else {
      Disabled
    }
    <button (click)="toggleNotifications()" class="override-btn">
      @if (notificationsEnabled()) {
        Disable
      } @else {
        Enable
      }
    </button>
  </div>
  <!-- existing message and working-hours divs remain -->
</div>
```

</docs-step>

<docs-step title="Observer le comportement réactif {#observe-the-reactive-behavior}">
Testez maintenant le comportement :

1. Changez le statut de l'utilisateur : remarquez que `notificationsEnabled` se met à jour automatiquement
2. Basculez manuellement les notifications : cela remplace la valeur calculée
3. Changez à nouveau le statut : le linked signal se resynchronise avec son calcul

Cela montre que les linked signals conservent leur connexion réactive même après avoir été définis manuellement !
</docs-step>

</docs-workflow>

Excellent ! Vous avez appris les différences essentielles entre les signals computed et les linked signals :

- **Signals computed** : en lecture seule, toujours dérivés d'autres signals
- **Linked signals** : modifiables, peuvent être à la fois dérivés ET mis à jour manuellement
- **Utilisez computed quand** : la valeur doit toujours être calculée
- **Utilisez linkedSignal quand** : vous avez besoin d'un calcul par défaut qui peut être remplacé

Dans la prochaine leçon, vous apprendrez [à gérer des données asynchrones avec des signals](/tutorials/signals/4-managing-async-data-with-signals) !
