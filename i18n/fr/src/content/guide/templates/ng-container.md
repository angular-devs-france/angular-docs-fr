<!-- source: fefbea4216b322943b0da2756215b8fb4a0b5aca -->
# Regrouper des éléments avec ng-container

`<ng-container>` est un élément spécial d'Angular qui regroupe plusieurs éléments ou marque un emplacement dans un template sans afficher de véritable élément dans le DOM.

```angular-html
<!-- Component template -->
<section>
  <ng-container>
    <h3>User bio</h3>
    <p>Here's some info about the user</p>
  </ng-container>
</section>
```

```angular-html
<!-- Rendered DOM -->
<section>
  <h3>User bio</h3>
  <p>Here's some info about the user</p>
</section>
```

Vous pouvez appliquer des directives à `<ng-container>` pour ajouter des comportements ou de la configuration à une partie de votre template.

Angular ignore tous les attribute bindings et tous les event listeners appliqués à `<ng-container>`, y compris ceux appliqués via une directive.

## Utiliser `<ng-container>` pour afficher du contenu dynamique {#using-ng-container-to-display-dynamic-contents}

`<ng-container>` peut servir de placeholder pour afficher du contenu dynamique.

### Afficher des composants {#rendering-components}

Vous pouvez utiliser la directive intégrée `NgComponentOutlet` d'Angular pour afficher dynamiquement un composant à l'emplacement du `<ng-container>`.

```angular-ts
@Component({
  template: `
    <h2>Your profile</h2>
    <ng-container [ngComponentOutlet]="profileComponent()" />
  `,
})
export class UserProfile {
  isAdmin = input(false);
  profileComponent = computed(() => (this.isAdmin() ? AdminProfile : BasicUserProfile));
}
```

Dans l'exemple ci-dessus, la directive `NgComponentOutlet` affiche dynamiquement `AdminProfile` ou `BasicUserProfile` à l'emplacement de l'élément `<ng-container>`.

### Afficher des fragments de template {#rendering-template-fragments}

Vous pouvez utiliser la directive intégrée `NgTemplateOutlet` d'Angular pour afficher dynamiquement un fragment de template à l'emplacement du `<ng-container>`.

```angular-ts
@Component({
  template: `
    <h2>Your profile</h2>
    <ng-container [ngTemplateOutlet]="profileTemplate()" />

    <ng-template #admin>This is the admin profile</ng-template>
    <ng-template #basic>This is the basic profile</ng-template>
  `,
})
export class UserProfile {
  isAdmin = input(false);
  adminTemplate = viewChild('admin', {read: TemplateRef});
  basicTemplate = viewChild('basic', {read: TemplateRef});
  profileTemplate = computed(() => (this.isAdmin() ? this.adminTemplate() : this.basicTemplate()));
}
```

Dans l'exemple ci-dessus, la directive `ngTemplateOutlet` affiche dynamiquement l'un des deux fragments de template à l'emplacement de l'élément `<ng-container>`.

Pour plus d'informations sur `NgTemplateOutlet`, consultez la [page de documentation de l'API NgTemplateOutlet](/api/common/NgTemplateOutlet).

## Utiliser `<ng-container>` avec des directives structurelles {#using-ng-container-with-structural-directives}

Vous pouvez aussi appliquer des directives structurelles aux éléments `<ng-container>`. Les exemples courants sont `ngIf` et `ngFor`.

```angular-html
<ng-container *ngIf="permissions == 'admin'">
  <h1>Admin Dashboard</h1>
  <admin-infographic />
</ng-container>

<ng-container *ngFor="let item of items; index as i; trackBy: trackByFn">
  <h2>{{ item.title }}</h2>
  <p>{{ item.description }}</p>
</ng-container>
```

## Utiliser `<ng-container>` pour l'injection {#using-ng-container-for-injection}

Consultez le [guide de l'injection de dépendances](/guide/di) pour plus d'informations sur le système d'injection de dépendances d'Angular.

Lorsque vous appliquez une directive à `<ng-container>`, les éléments descendants peuvent injecter la directive ou tout ce que la directive fournit. Utilisez cette technique lorsque vous voulez fournir une valeur de manière déclarative à une partie précise de votre template.

```angular-ts
@Directive({
  selector: '[theme]',
})
export class Theme {
  // Create an input that accepts 'light' or 'dark`, defaulting to 'light'.
  mode = input<'light' | 'dark'>('light');
}
```

```angular-html
<ng-container theme="dark">
  <profile-pic />
  <user-bio />
</ng-container>
```

Dans l'exemple ci-dessus, les composants `ProfilePic` et `UserBio` peuvent injecter la directive `Theme` et appliquer des styles en fonction de son `mode`.
