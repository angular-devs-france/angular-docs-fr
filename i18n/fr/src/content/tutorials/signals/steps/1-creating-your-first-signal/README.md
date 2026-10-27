<!-- source: 542e085ec144b274682eb757727c2e193628fc2c -->
# Créer et mettre à jour votre premier signal

Bienvenue dans le tutoriel sur les signals Angular ! Les [signals](/essentials/signals) sont la primitive réactive d'Angular : ils permettent de gérer l'état et de mettre à jour automatiquement votre interface quand cet état change.

Dans cette activité, vous allez apprendre à :

- Créer votre premier signal avec la fonction `signal()`
- Afficher sa valeur dans un template
- Mettre à jour la valeur du signal avec les méthodes `set()` et `update()`

Construisons un système interactif de statut utilisateur avec des signals !

<hr />

<docs-workflow>

<docs-step title="Importer la fonction signal {#import-the-signal-function}">
Importez la fonction `signal` depuis `@angular/core` en haut du fichier de votre composant.

```ts
import {Component, signal, ChangeDetectionStrategy} from '@angular/core';
```

</docs-step>

<docs-step title="Créer un signal dans votre composant {#create-a-signal-in-your-component}">
Ajoutez à la classe de votre composant un signal `userStatus` initialisé avec la valeur `'offline'`.

```ts
@Component({
  /* Config omitted */
})
export class App {
  userStatus = signal<'online' | 'offline'>('offline');
}
```

</docs-step>

<docs-step title="Afficher la valeur du signal dans le template {#display-the-signal-value-in-the-template}">
Mettez à jour l'indicateur de statut pour afficher le statut actuel de l'utilisateur :
1. Créez un binding du signal sur l'attribut class avec `[class]="userStatus()"`
2. Affichez le texte du statut en remplaçant `???` par `{{ userStatus() }}`

```angular-html
<!-- Update from: -->
<div class="status-indicator offline">
  <span class="status-dot"></span>
  Status: ???
</div>

<!-- To: -->
<div class="status-indicator" [class]="userStatus()">
  <span class="status-dot"></span>
  Status: {{ userStatus() }}
</div>
```

Remarquez que nous appelons le signal `userStatus()` avec des parenthèses pour lire sa valeur.
</docs-step>

<docs-step title="Ajouter des méthodes pour mettre à jour le signal {#add-methods-to-update-the-signal}">
Ajoutez à votre composant des méthodes qui changent le statut de l'utilisateur avec la méthode `set()`.

```ts
goOnline() {
  this.userStatus.set('online');
}

goOffline() {
  this.userStatus.set('offline');
}
```

La méthode `set()` remplace entièrement la valeur du signal par une nouvelle valeur.

</docs-step>

<docs-step title="Brancher les boutons de contrôle {#wire-up-the-control-buttons}">
Les boutons sont déjà présents dans le template. Connectez-les maintenant à vos méthodes en ajoutant :
1. Des gestionnaires de clic avec `(click)`
2. Des états désactivés avec `[disabled]` quand l'utilisateur a déjà ce statut

```html
<!-- Add bindings to the existing buttons: -->
<button (click)="goOnline()" [disabled]="userStatus() === 'online'">Go Online</button>
<button (click)="goOffline()" [disabled]="userStatus() === 'offline'">Go Offline</button>
```

</docs-step>

<docs-step title="Ajouter une méthode de bascule avec update() {#add-a-toggle-method-using-update}">
Ajoutez une méthode `toggleStatus()` qui bascule entre online et offline avec la méthode `update()`.

```ts
toggleStatus() {
  this.userStatus.update(current => current === 'online' ? 'offline' : 'online');
}
```

La méthode `update()` prend une fonction qui reçoit la valeur actuelle et renvoie la nouvelle valeur. C'est utile quand vous devez modifier la valeur existante en fonction de son état actuel.

</docs-step>

<docs-step title="Ajouter le gestionnaire du bouton de bascule {#add-the-toggle-button-handler}">
Le bouton de bascule est déjà présent dans le template. Connectez-le à votre méthode `toggleStatus()` :

```html
<button (click)="toggleStatus()" class="toggle-btn">Toggle Status</button>
```

</docs-step>

</docs-workflow>

Félicitations ! Vous avez créé votre premier signal et appris à le mettre à jour avec les méthodes `set()` et `update()`. La fonction `signal()` crée une valeur réactive qu'Angular suit, et quand vous la mettez à jour, votre interface reflète automatiquement les changements.

Ensuite, vous apprendrez [à dériver un état à partir de signals avec computed](/tutorials/signals/2-deriving-state-with-computed-signals) !
