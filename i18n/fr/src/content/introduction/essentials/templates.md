<!-- source: 21a8c6ffe54efabe4388fb4276329cd8d15bf92b -->
<docs-decorative-header title="Templates" imgSrc="adev/src/assets/images/templates.svg"> <!-- markdownlint-disable-line -->
Utilisez la syntaxe de template d'Angular pour créer des interfaces utilisateur dynamiques.
</docs-decorative-header>

Les templates de composant ne sont pas du simple HTML statique : ils peuvent utiliser les données de la classe de votre composant et mettre en place des event handlers pour les interactions de l'utilisateur.

## Afficher du texte dynamique {#showing-dynamic-text}

Dans Angular, un _binding_ crée une connexion dynamique entre le template d'un composant et ses données. Cette connexion garantit que les modifications des données du composant mettent automatiquement à jour le template rendu.

Vous pouvez créer un binding pour afficher du texte dynamique dans un template avec des doubles accolades :

```angular-ts
@Component({
  selector: 'user-profile',
  template: `<h1>Profile for {{ userName() }}</h1>`,
})
export class UserProfile {
  userName = signal('pro_programmer_123');
}
```

Quand Angular rend le composant, vous voyez :

```html
<h1>Profile for pro_programmer_123</h1>
```

Angular maintient automatiquement le binding à jour quand la valeur du signal change. En reprenant
l'exemple ci-dessus, si nous mettons à jour la valeur du signal `userName` :

```typescript
this.userName.set('cool_coder_789');
```

La page rendue se met à jour pour refléter la nouvelle valeur :

```html
<h1>Profile for cool_coder_789</h1>
```

## Définir des propriétés et des attributs dynamiques {#setting-dynamic-properties-and-attributes}

Angular permet de lier des valeurs dynamiques à des propriétés du DOM avec des crochets :

```angular-ts
@Component({
  /*...*/
  // Set the `disabled` property of the button based on the value of `isValidUserId`.
  template: `<button [disabled]="!isValidUserId()">Save changes</button>`,
})
export class UserProfile {
  isValidUserId = signal(false);
}
```

Vous pouvez aussi lier des _attributs_ HTML en préfixant le nom de l'attribut par `attr.` :

```angular-html
<!-- Bind the `role` attribute on the `<ul>` element to value of `listRole`. -->
<ul [attr.role]="listRole()"></ul>
```

Angular met automatiquement à jour les propriétés et les attributs du DOM quand la valeur liée change.

## Gérer les interactions de l'utilisateur {#handling-user-interaction}

Angular vous permet d'ajouter des event listeners à un élément de votre template avec des parenthèses :

```angular-ts
@Component({
  /*...*/
  // Add an 'click' event handler that calls the `cancelSubscription` method.
  template: `<button (click)="cancelSubscription()">Cancel subscription</button>`,
})
export class UserProfile {
  /* ... */

  cancelSubscription() {
    /* Your event handling code goes here. */
  }
}
```

Si vous devez passer l'objet [événement](https://developer.mozilla.org/docs/Web/API/Event) à votre event listener, vous pouvez utiliser la variable intégrée `$event` d'Angular dans l'appel de fonction :

```angular-ts
@Component({
  /*...*/
  // Add an 'click' event handler that calls the `cancelSubscription` method.
  template: `<button (click)="cancelSubscription($event)">Cancel subscription</button>`,
})
export class UserProfile {
  /* ... */

  cancelSubscription(event: Event) {
    /* Your event handling code goes here. */
  }
}
```

## Control flow avec `@if` et `@for` {#control-flow-with-if-and-for}

Vous pouvez masquer et afficher des parties d'un template selon une condition avec le bloc `@if` d'Angular :

```angular-html
<h1>User profile</h1>

@if (isAdmin()) {
  <h2>Admin settings</h2>
  <!-- ... -->
}
```

Le bloc `@if` accepte aussi un bloc `@else` facultatif :

```angular-html
<h1>User profile</h1>

@if (isAdmin()) {
  <h2>Admin settings</h2>
  <!-- ... -->
} @else {
  <h2>User settings</h2>
  <!-- ... -->
}
```

Vous pouvez répéter une partie d'un template plusieurs fois avec le bloc `@for` d'Angular :

```angular-html
<h1>User profile</h1>

<ul class="user-badge-list">
  @for (badge of badges(); track badge.id) {
    <li class="user-badge">{{ badge.name }}</li>
  }
</ul>
```

Angular utilise le mot-clé `track`, visible dans l'exemple ci-dessus, pour associer les données aux éléments du DOM créés par `@for`. Consultez [_Pourquoi track est-il important dans les blocs @for ?_](guide/templates/control-flow#why-is-track-in-for-blocks-important) pour plus d'informations.

TIP: Vous voulez en savoir plus sur les templates Angular ? Consultez le [guide approfondi des templates](guide/templates) pour tous les détails.

## Étape suivante {#next-step}

Maintenant que votre application a des données dynamiques et des templates, il est temps d'apprendre à enrichir les templates en masquant ou en affichant certains éléments selon une condition, en itérant sur des éléments, et plus encore.

<docs-pill-row>
  <docs-pill title="Les formulaires avec les signals" href="essentials/signal-forms" />
  <docs-pill title="Guide approfondi des templates" href="guide/templates" />
</docs-pill-row>
