<!-- source: 202af3e35bf7c06645bf999f4dfa4f62aa29e5b1 -->
# Projection de contenu avec ng-content

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Lisez-le d'abord si vous découvrez Angular.

Vous avez souvent besoin de créer des composants qui servent de conteneurs pour différents types de contenu. Par
exemple, vous pouvez vouloir créer un composant de carte personnalisé :

```angular-ts
@Component({
  selector: 'custom-card',
  template: '<div class="card-shadow"> <!-- card content goes here --> </div>',
})
export class CustomCard {
  /* ... */
}
```

**Vous pouvez utiliser l'élément `<ng-content>` comme placeholder pour indiquer où le contenu doit être placé** :

```angular-ts
@Component({
  selector: 'custom-card',
  template: '<div class="card-shadow"> <ng-content/> </div>',
})
export class CustomCard {
  /* ... */
}
```

TIP: `<ng-content>` fonctionne de manière similaire
à [l'élément natif `<slot>`](https://developer.mozilla.org/docs/Web/HTML/Element/slot),
mais avec certaines fonctionnalités propres à Angular.

Quand vous utilisez un composant avec `<ng-content>`, tous les enfants du host element du composant sont
rendus, ou **projetés**, à l'emplacement de ce `<ng-content>` :

```angular-ts
// Component source
@Component({
  selector: 'custom-card',
  template: `
    <div class="card-shadow">
      <ng-content />
    </div>
  `,
})
export class CustomCard {
  /* ... */
}
```

```angular-html
<!-- Using the component -->
<custom-card>
  <p>This is the projected content</p>
</custom-card>
```

```angular-html
<!-- The rendered DOM -->
<custom-card>
  <div class="card-shadow">
    <p>This is the projected content</p>
  </div>
</custom-card>
```

Angular appelle **contenu** d'un composant tous les enfants qui lui sont passés de cette façon. Il
se distingue de la **vue** du composant, qui désigne les éléments définis dans le template
du composant.

**L'élément `<ng-content>` n'est ni un composant ni un élément du DOM**. Il s'agit plutôt d'un
placeholder spécial qui indique à Angular où rendre le contenu. Le compilateur d'Angular traite
tous les éléments `<ng-content>` au moment du build. Vous ne pouvez pas insérer, supprimer ni modifier `<ng-content>` à
l'exécution. Vous ne pouvez pas ajouter de directives, de styles ni d'attributs arbitraires à `<ng-content>`.

IMPORTANT: Vous ne devez pas inclure `<ng-content>` de manière conditionnelle avec `@if`, `@for` ou `@switch`. Angular
instancie et crée toujours les nœuds du DOM pour le contenu rendu dans un placeholder `<ng-content>`, même si
ce placeholder `<ng-content>` est masqué. Pour le rendu conditionnel du contenu d'un composant,
consultez [Fragments de template](api/core/ng-template).

## Plusieurs placeholders de contenu {#multiple-content-placeholders}

Angular permet de projeter plusieurs éléments différents dans différents placeholders `<ng-content>`
en fonction d'un selector CSS. En reprenant l'exemple de carte ci-dessus, vous pouvez créer deux placeholders pour
le titre et le corps de la carte grâce à l'attribut `select` :

```angular-ts
@Component({
  selector: 'card-title',
  template: `<ng-content>card-title</ng-content>`,
})
export class CardTitle {}

@Component({
  selector: 'card-body',
  template: `<ng-content>card-body</ng-content>`,
})
export class CardBody {}
```

```angular-ts
<!-- Component template -->
@Component({
  selector: 'custom-card',
  template: `
  <div class="card-shadow">
    <ng-content select="card-title" />
    <div class="card-divider"></div>
    <ng-content select="card-body" />
  </div>
  `,
})
export class CustomCard {}
```

```angular-ts
<!-- Using the component -->
@Component({
  selector: 'app-root',
  imports: [CustomCard, CardTitle, CardBody],
  template: `
    <custom-card>
      <card-title>Hello</card-title>
      <card-body>Welcome to the example</card-body>
    </custom-card>
`,
})
export class App {}
```

```angular-html
<!-- Rendered DOM -->
<custom-card>
  <div class="card-shadow">
    <card-title>Hello</card-title>
    <div class="card-divider"></div>
    <card-body>Welcome to the example</card-body>
  </div>
</custom-card>
```

Le placeholder `<ng-content>` prend en charge les mêmes selectors CSS
que les [selectors de composant](guide/components/selectors).

Si vous incluez un ou plusieurs placeholders `<ng-content>` avec un attribut `select` et
un placeholder `<ng-content>` sans attribut `select`, ce dernier capture tous les éléments qui
ne correspondent à aucun attribut `select` :

```angular-html
<!-- Component template -->
<div class="card-shadow">
  <ng-content select="card-title" />
  <div class="card-divider"></div>
  <!-- capture anything except "card-title" -->
  <ng-content />
</div>
```

```angular-html
<!-- Using the component -->
<custom-card>
  <card-title>Hello</card-title>
  <img src="..." />
  <p>Welcome to the example</p>
</custom-card>
```

```angular-html
<!-- Rendered DOM -->
<custom-card>
  <div class="card-shadow">
    <card-title>Hello</card-title>
    <div class="card-divider"></div>
    <img src="..." />
    <p>Welcome to the example</p>
  </div>
</custom-card>
```

Si un composant n'inclut pas de placeholder `<ng-content>` sans attribut `select`, les
éléments qui ne correspondent à aucun des placeholders du composant ne sont pas rendus dans le DOM.

## Contenu de repli {#fallback-content}

Angular peut afficher un _contenu de repli_ pour le placeholder `<ng-content>` d'un composant si ce composant n'a aucun contenu enfant correspondant. Vous pouvez définir un contenu de repli en ajoutant du contenu enfant directement dans l'élément `<ng-content>`.

```angular-html
<!-- Component template -->
<div class="card-shadow">
  <ng-content select="card-title">Default Title</ng-content>
  <div class="card-divider"></div>
  <ng-content select="card-body">Default Body</ng-content>
</div>
```

```angular-html
<!-- Using the component -->
<custom-card>
  <card-title>Hello</card-title>
  <!-- No card-body provided -->
</custom-card>
```

```angular-html
<!-- Rendered DOM -->
<custom-card>
  <div class="card-shadow">
    <card-title>Hello</card-title>
    <div class="card-divider"></div>
    Default Body
  </div>
</custom-card>
```

## Définir un alias de contenu pour la projection {#aliasing-content-for-projection}

Angular prend en charge un attribut spécial, `ngProjectAs`, qui vous permet de définir un selector CSS sur
n'importe quel élément. Chaque fois qu'un élément portant `ngProjectAs` est comparé à un placeholder
`<ng-content>`, Angular utilise la valeur de `ngProjectAs` au lieu de l'identité de l'élément :

```angular-html
<!-- Component template -->
<div class="card-shadow">
  <ng-content select="card-title" />
  <div class="card-divider"></div>
  <ng-content />
</div>
```

```angular-html
<!-- Using the component -->
<custom-card>
  <h3 ngProjectAs="card-title">Hello</h3>

  <p>Welcome to the example</p>
</custom-card>
```

```angular-html
<!-- Rendered DOM -->
<custom-card>
  <div class="card-shadow">
    <h3>Hello</h3>
    <div class="card-divider"></div>
    <p>Welcome to the example</p>
  </div>
</custom-card>
```

`ngProjectAs` ne prend en charge que des valeurs statiques et ne peut pas être lié à des expressions dynamiques.

## Mises en garde {#caveats}

### Le contenu projeté vit dans la vue du parent {#projected-content-lives-in-the-parents-view}

Même si le contenu projeté est _rendu_ à l'intérieur du composant qui le reçoit, il appartient toujours au composant qui l'a déclaré. Angular le suit comme une partie de la vue du parent, ce qui a quelques effets de bord qu'il est bon de connaître.

**Détection de changements :** le contenu projeté est vérifié quand le _parent_ exécute la détection de changements. Si le composant qui le reçoit utilise `OnPush`, Angular peut ignorer la vérification du template de ce composant, mais il n'ignorera pas le contenu projeté, car celui-ci appartient au parent.

```angular-html
<!-- Parent template (default change detection) -->
<onpush-wrapper>
  <!-- Still checked on every parent cycle, OnPush doesn't help here -->
  <expensive-component />
</onpush-wrapper>
```

**Injection de dépendances :** le contenu projeté obtient ses dépendances depuis l'injector du parent, et non depuis les `viewProviders` du composant qui le reçoit. Consultez [Providers et viewProviders](guide/di/hierarchical-dependency-injection) pour plus de détails.

### Certains composants de library ne prennent pas en charge les enfants projetés {#some-library-components-dont-support-projected-children}

Certains composants (menus, onglets, listes) utilisent `ContentChildren` pour trouver leurs enfants et mettre en place des comportements comme la navigation au clavier, la gestion du focus ou les attributs ARIA. Ils sont écrits en partant du principe qu'ils possèdent directement leurs enfants, si bien que projeter du contenu externe à l'intérieur a tendance à provoquer des dysfonctionnements subtils.

Par exemple, envelopper des éléments `<mat-menu-item>` dans une couche supplémentaire et les projeter dans `<mat-menu>` peut casser silencieusement la navigation au clavier et la prise en charge des lecteurs d'écran. La query trouve toujours les éléments, mais la configuration interne qui les rend interactifs peut ne pas fonctionner correctement quand les éléments proviennent d'un contexte de vue différent.

Si un composant de library gère le comportement de ses enfants, consultez sa documentation avant de recourir à la projection de contenu : elle n'est peut-être pas prise en charge.
