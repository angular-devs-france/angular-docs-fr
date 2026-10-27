<!-- source: 9e1c24eeb530a37a8c20d5d4073ac4a7ab40c47f -->
# Créer des fragments de template avec ng-template

Inspiré de l'[élément natif `<template>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template), l'élément `<ng-template>` vous permet de déclarer un **fragment de template** – une section de contenu que vous pouvez afficher de manière dynamique ou programmatique.

## Créer un fragment de template {#creating-a-template-fragment}

Vous pouvez créer un fragment de template dans n'importe quel template de composant avec l'élément `<ng-template>` :

```angular-html
<p>This is a normal element</p>

<ng-template>
  <p>This is a template fragment</p>
</ng-template>
```

Lors de l'affichage du code ci-dessus, le contenu de l'élément `<ng-template>` n'est pas affiché sur la page. À la place, vous pouvez obtenir une référence au fragment de template et écrire du code pour l'afficher dynamiquement.

### Contexte de binding des fragments {#binding-context-for-fragments}

Les fragments de template peuvent contenir des bindings avec des expressions dynamiques :

```angular-ts
@Component({
  /* ... */,
  template: `<ng-template>You've selected {{count}} items.</ng-template>`,
})
export class ItemCounter {
  count: number = 0;
}
```

Les expressions et les instructions d'un fragment de template sont évaluées par rapport au composant dans lequel le fragment est déclaré, quel que soit l'endroit où le fragment est affiché.

## Obtenir une référence à un fragment de template {#getting-a-reference-to-a-template-fragment}

Vous pouvez obtenir une référence à un fragment de template de trois manières :

- En déclarant une [variable de référence de template](/guide/templates/variables#template-reference-variables) sur l'élément `<ng-template>`
- En interrogeant le fragment avec [une query de composant ou de directive](/guide/components/queries)
- En injectant le fragment dans une directive appliquée directement à un élément `<ng-template>`.

Dans les trois cas, le fragment est représenté par un objet [TemplateRef](/api/core/TemplateRef).

### Référencer un fragment de template avec une variable de référence de template {#referencing-a-template-fragment-with-a-template-reference-variable}

Vous pouvez ajouter une variable de référence de template à un élément `<ng-template>` pour référencer ce fragment de template dans d'autres parties du même fichier de template :

```angular-html
<p>This is a normal element</p>

<ng-template #myFragment>
  <p>This is a template fragment</p>
</ng-template>
```

Vous pouvez ensuite référencer ce fragment n'importe où ailleurs dans le template via la variable `myFragment`.

### Référencer un fragment de template avec des queries {#referencing-a-template-fragment-with-queries}

Vous pouvez obtenir une référence à un fragment de template avec n'importe quelle [API de query de composant ou de directive](/guide/components/queries).

Vous pouvez interroger directement l'objet `TemplateRef` avec une query `viewChild`.

```angular-ts
@Component({
  /* ... */,
  template: `
    <p>This is a normal element</p>

    <ng-template>
      <p>This is a template fragment</p>
    </ng-template>
  `,
})
export class ComponentWithFragment {
  templateRef = viewChild<TemplateRef<unknown>>(TemplateRef);
}
```

Vous pouvez ensuite référencer ce fragment dans le code de votre composant ou dans le template du composant, comme n'importe quel autre membre de classe.

Si un template contient plusieurs fragments, vous pouvez attribuer un nom à chaque fragment en ajoutant une variable de référence de template à chaque élément `<ng-template>`, puis interroger les fragments à partir de ce nom :

```angular-ts
@Component({
  /* ... */,
  template: `
    <p>This is a normal element</p>

    <ng-template #fragmentOne>
      <p>This is one template fragment</p>
    </ng-template>

    <ng-template #fragmentTwo>
      <p>This is another template fragment</p>
    </ng-template>
  `,
})
export class ComponentWithFragment {
    fragmentOne = viewChild<TemplateRef<unknown>>('fragmentOne');
    fragmentTwo = viewChild<TemplateRef<unknown>>('fragmentTwo');
}
```

Là encore, vous pouvez ensuite référencer ces fragments dans le code de votre composant ou dans le template du composant, comme n'importe quels autres membres de classe.

### Injecter un fragment de template {#injecting-a-template-fragment}

Une directive peut injecter un `TemplateRef` si cette directive est appliquée directement à un élément `<ng-template>` :

```angular-ts
@Directive({
  selector: '[myDirective]',
})
export class MyDirective {
  private fragment = inject(TemplateRef);
}
```

```angular-html
<ng-template myDirective>
  <p>This is one template fragment</p>
</ng-template>
```

Vous pouvez ensuite référencer ce fragment dans le code de votre directive, comme n'importe quel autre membre de classe.

## Afficher un fragment de template {#rendering-a-template-fragment}

Une fois que vous avez une référence à l'objet `TemplateRef` d'un fragment de template, vous pouvez afficher le fragment de deux manières : dans votre template avec la directive `NgTemplateOutlet`, ou dans votre code TypeScript avec `ViewContainerRef`.

### Utiliser `NgTemplateOutlet` {#using-ngtemplateoutlet}

La directive `NgTemplateOutlet` de `@angular/common` accepte un `TemplateRef` et affiche le fragment comme **frère** de l'élément qui porte l'outlet. En général, utilisez `NgTemplateOutlet` sur un [élément `<ng-container>`](/guide/templates/ng-container).

Commencez par importer `NgTemplateOutlet` :

```typescript
import {NgTemplateOutlet} from '@angular/common';
```

L'exemple suivant déclare un fragment de template et affiche ce fragment dans un élément `<ng-container>` avec `NgTemplateOutlet` :

```angular-html
<p>This is a normal element</p>

<ng-template #myFragment>
  <p>This is a fragment</p>
</ng-template>

<ng-container *ngTemplateOutlet="myFragment"></ng-container>
```

Cet exemple produit le DOM suivant :

```angular-html
<p>This is a normal element</p>
<p>This is a fragment</p>
```

### Utiliser `ViewContainerRef` {#using-viewcontainerref}

Un **view container** est un nœud de l'arbre de composants d'Angular qui peut contenir du contenu. Tout composant ou toute directive peut injecter `ViewContainerRef` pour obtenir une référence au view container correspondant à l'emplacement de ce composant ou de cette directive dans le DOM.

Vous pouvez utiliser la méthode `createEmbeddedView` de `ViewContainerRef` pour afficher dynamiquement un fragment de template. Lorsque vous affichez un fragment avec un `ViewContainerRef`, Angular l'ajoute au DOM comme frère suivant du composant ou de la directive qui a injecté le `ViewContainerRef`.

L'exemple suivant montre un composant qui accepte une référence à un fragment de template comme input et qui affiche ce fragment dans le DOM lors d'un clic sur un bouton.

```angular-ts
@Component({
  /* ... */,
  selector: 'component-with-fragment',
  template: `
    <h2>Component with a fragment</h2>
    <ng-template #myFragment>
      <p>This is the fragment</p>
    </ng-template>
    <my-outlet [fragment]="myFragment" />
  `,
})
export class ComponentWithFragment { }

@Component({
  /* ... */,
  selector: 'my-outlet',
  template: `<button (click)="showFragment()">Show</button>`,
})
export class MyOutlet {
  private viewContainer = inject(ViewContainerRef);
  fragment = input<TemplateRef<unknown> | undefined>();

  showFragment() {
    if (this.fragment()) {
      this.viewContainer.createEmbeddedView(this.fragment());
    }
  }
}
```

Dans l'exemple ci-dessus, un clic sur le bouton « Show » produit le résultat suivant :

```angular-html
<component-with-fragment>
  <h2>Component with a fragment</h2>
  <my-outlet>
    <button>Show</button>
  </my-outlet>
  <p>This is the fragment</p>
</component-with-fragment>
```

## Passer des paramètres lors de l'affichage d'un fragment de template {#passing-parameters-when-rendering-a-template-fragment}

Lorsque vous déclarez un fragment de template avec `<ng-template>`, vous pouvez en plus déclarer les paramètres acceptés par le fragment. Lorsque vous affichez un fragment, vous pouvez éventuellement passer un objet `context` correspondant à ces paramètres. Vous pouvez utiliser les données de cet objet de contexte dans les expressions de binding et les instructions, en plus des données du composant dans lequel le fragment est déclaré.

Chaque paramètre s'écrit sous la forme d'un attribut préfixé par `let-`, dont la valeur correspond à un nom de propriété de l'objet de contexte :

```angular-html
<ng-template let-pizzaTopping="topping">
  <p>You selected: {{ pizzaTopping }}</p>
</ng-template>
```

### Utiliser `NgTemplateOutlet` {#using-ngtemplateoutlet-with-parameters}

Vous pouvez lier un objet de contexte à l'input `ngTemplateOutletContext` :

```angular-html
<ng-template #myFragment let-pizzaTopping="topping">
  <p>You selected: {{ pizzaTopping }}</p>
</ng-template>

<ng-container [ngTemplateOutlet]="myFragment" [ngTemplateOutletContext]="{topping: 'onion'}" />
```

### Utiliser `ViewContainerRef` {#using-viewcontainerref-with-parameters}

Vous pouvez passer un objet de contexte comme second argument de `createEmbeddedView` :

```ts
this.viewContainer.createEmbeddedView(this.myFragment, {topping: 'onion'});
```

## Fournir des injectors aux fragments de template {#providing-injectors-to-template-fragments}

Lorsque vous affichez un fragment de template, son contexte d'injector provient de **l'emplacement de la déclaration du template**, et non de l'endroit où il est affiché. Vous pouvez modifier ce comportement en fournissant un injector personnalisé.

### Utiliser `NgTemplateOutlet` {#using-ngtemplateoutlet-with-injectors}

Vous pouvez passer un `Injector` personnalisé à l'input `ngTemplateOutletInjector` :

```angular-ts
export const THEME_DATA = new InjectionToken<string>('THEME_DATA', {
  factory: () => 'light',
});

@Component({
  selector: 'themed-panel',
  template: `<div [class]="theme">...</div>`,
})
export class ThemedPanel {
  theme = inject(THEME_DATA);
}

@Component({
  selector: 'root',
  imports: [NgTemplateOutlet, ThemedPanel],
  template: `
    <ng-template #myFragment>
      <themed-panel />
    </ng-template>
    <ng-container *ngTemplateOutlet="myFragment; injector: customInjector" />
  `,
})
export class Root {
  customInjector = Injector.create({
    providers: [{provide: THEME_DATA, useValue: 'dark'}],
  });
}
```

#### Hériter de l'injector de l'outlet {#inheriting-the-outlets-injector}

Vous pouvez définir `ngTemplateOutletInjector` sur la chaîne `'outlet'` pour que la vue intégrée hérite de l'injector de l'emplacement de l'outlet dans le DOM, plutôt que de l'endroit où le template a été déclaré.

```angular-html
<ng-template #node let-items>
  <item-component>
    @for (child of items; track $index) {
      <ng-container
        *ngTemplateOutlet="node; context: {$implicit: child.children}; injector: 'outlet'"
      />
    }
  </item-component>
</ng-template>

<ng-container *ngTemplateOutlet="node; context: {$implicit: topLevelItems}" />
```

Chaque affichage récursif du template `node` hérite de l'injector de l'`<item-component>` englobant, ce qui permet à chaque niveau imbriqué d'accéder aux providers rattachés à son composant parent.

NOTE: C'est utile pour construire des structures récursives, ou dans toute situation où le template affiché a besoin d'accéder aux providers de l'arbre de composants à l'emplacement de l'outlet.

### Utiliser `ViewContainerRef` {#using-viewcontainerref-with-injectors}

Vous pouvez passer un injector personnalisé dans l'objet d'options de `createEmbeddedView` :

```ts
this.viewContainer.createEmbeddedView(this.myFragment, context, {
  injector: myCustomInjector,
});
```

## Directives structurelles {#structural-directives}

Une **directive structurelle** est une directive qui :

- Injecte `TemplateRef`
- Injecte `ViewContainerRef` et affiche de manière programmatique le `TemplateRef` injecté

Angular prend en charge une syntaxe simplifiée spéciale pour les directives structurelles. Si vous appliquez la directive à un élément en préfixant le selector de la directive par un astérisque (`*`), Angular interprète l'élément entier et tout son contenu comme un fragment de template :

```angular-html
<section *myDirective>
  <p>This is a fragment</p>
</section>
```

Cela équivaut à :

```angular-html
<ng-template myDirective>
  <section>
    <p>This is a fragment</p>
  </section>
</ng-template>
```

Les développeurs utilisent généralement les directives structurelles pour afficher des fragments de manière conditionnelle ou pour afficher des fragments plusieurs fois.

Pour plus de détails, consultez [Directives structurelles](/guide/directives/structural-directives).

## Ressources supplémentaires {#additional-resources}

Pour des exemples d'utilisation de `ng-template` dans d'autres libraries, consultez :

- [Les onglets d'Angular Material](https://material.angular.dev/components/tabs/overview) - rien n'est affiché dans le DOM tant que l'onglet n'est pas activé
- [Le tableau d'Angular Material](https://material.angular.dev/components/table/overview) - permet aux développeurs de définir différentes manières d'afficher les données
