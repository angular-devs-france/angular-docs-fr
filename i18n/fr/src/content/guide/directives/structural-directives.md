<!-- source: 13ce47f57f88bde057177ee438ec3b19ca55f2e8 -->
# Directives structurelles

Les directives structurelles sont des directives appliquées à un élément `<ng-template>` qui rendent le contenu de cet `<ng-template>` de manière conditionnelle ou répétée.

Pour le rendu conditionnel et répété du quotidien, utilisez les [blocs de control flow](guide/templates/control-flow) intégrés d'Angular (`@if`, `@for` et `@switch`). Écrivez une directive structurelle quand vous avez besoin d'un comportement de rendu réutilisable que le control flow ne couvre pas, comme conditionner l'affichage d'un contenu à une vérification de permission ou fournir à un template des données issues d'une source externe.

## Exemple de cas d'usage {#example-use-case}

Ce guide utilise comme fil conducteur une directive appelée `SelectDirective`. La directive récupère des données depuis une source de données donnée et rend son template quand ces données sont disponibles. Elle tient son nom du mot-clé SQL `SELECT` et correspond à un attribute selector, `[select]`.

`SelectDirective` possède un input nommé `selectFrom`, qui désigne la source de données à utiliser. Le préfixe `select` de cet input est important pour la [syntaxe abrégée](#structural-directive-shorthand). La directive instancie son `<ng-template>` avec un contexte de template qui fournit les données sélectionnées.

Utiliser cette directive directement sur un `<ng-template>` ressemble à ceci :

```angular-html
<ng-template select let-data [selectFrom]="source">
  <p>The data is: {{ data }}</p>
</ng-template>
```

La directive structurelle peut attendre que les données soient disponibles, puis rendre son `<ng-template>`.

HELPFUL: L'élément `<ng-template>` d'Angular définit un template qui ne rend rien par défaut. Si vous enveloppez des éléments dans un `<ng-template>` sans appliquer de directive structurelle, ces éléments ne sont pas rendus.

Pour plus d'informations, consultez la documentation de l'[API ng-template](api/core/ng-template).

## Syntaxe abrégée des directives structurelles {#structural-directive-shorthand}

Angular prend en charge une syntaxe abrégée pour les directives structurelles, qui évite d'écrire explicitement un élément `<ng-template>`.

Les directives structurelles peuvent être appliquées directement sur un élément en préfixant l'attribute selector de la directive par un astérisque (`*`), comme `*select`. Angular transforme l'astérisque placé devant une directive structurelle en un `<ng-template>` qui héberge la directive et entoure l'élément et ses descendants.

Vous pouvez l'utiliser avec `SelectDirective` de la manière suivante :

```angular-html
<p *select="let data; from: source">The data is: {{ data }}</p>
```

Cet exemple montre la flexibilité de la syntaxe abrégée des directives structurelles, parfois appelée _microsyntaxe_.

Utilisée de cette façon, seuls la directive structurelle et ses bindings sont appliqués au `<ng-template>`. Les autres attributs ou bindings de la balise `<p>` restent inchangés. Par exemple, ces deux formes sont équivalentes :

```angular-html
<!-- Shorthand syntax: -->
<p class="data-view" *select="let data; from: source">The data is: {{ data }}</p>

<!-- Long-form syntax: -->
<ng-template select let-data [selectFrom]="source">
  <p class="data-view">The data is: {{ data }}</p>
</ng-template>
```

La syntaxe abrégée est développée selon un ensemble de conventions. Une [grammaire](#structural-directive-syntax-reference) plus complète est définie plus bas, mais dans l'exemple ci-dessus, cette transformation s'explique ainsi :

La première partie de l'expression `*select` est `let data`, qui déclare une variable de template `data`. Comme aucune affectation ne suit, la variable de template est liée à la propriété `$implicit` du contexte de template.

Le deuxième élément de syntaxe est une paire clé-expression, `from source`. `from` est une clé de binding et `source` une expression de template ordinaire. Les clés de binding sont associées à des propriétés en les convertissant en PascalCase et en les préfixant du selector de la directive structurelle. La clé `from` est associée à `selectFrom`, qui est ensuite liée à l'expression `source`. C'est pourquoi beaucoup de directives structurelles ont des inputs tous préfixés par le selector de la directive structurelle.

## Une seule directive structurelle par élément {#one-structural-directive-per-element}

Vous ne pouvez appliquer qu'une seule directive structurelle par élément avec la syntaxe abrégée. En effet, il n'existe qu'un seul élément `<ng-template>` dans lequel cette directive est déployée. Plusieurs directives nécessiteraient plusieurs `<ng-template>` imbriqués, et on ne saurait pas quelle directive doit passer en premier. `<ng-container>` permet de créer des couches d'enveloppe quand plusieurs directives structurelles doivent être appliquées autour du même élément du DOM ou du même composant, ce qui permet à l'utilisateur de définir la structure imbriquée.

## Créer une directive structurelle {#creating-a-structural-directive}

Une directive structurelle est une classe de directive qui injecte deux dépendances :

- [`TemplateRef`](api/core/TemplateRef) donne à la directive accès au contenu du `<ng-template>` auquel elle est appliquée.
- [`ViewContainerRef`](api/core/ViewContainerRef) représente l'emplacement dans le DOM où la directive peut rendre ce template.

La directive contrôle le rendu en créant, ou non, des vues intégrées à partir du template dans le view container. La `SelectDirective` complète ressemble à ceci :

```ts
import {Directive, TemplateRef, ViewContainerRef, inject, input} from '@angular/core';

export interface DataSource<T> {
  load(): Promise<T>;
}

@Directive({
  selector: '[select]',
})
export class SelectDirective {
  private templateRef = inject(TemplateRef);
  private viewContainerRef = inject(ViewContainerRef);

  selectFrom = input.required<DataSource<unknown>>();

  async ngOnInit() {
    const data = await this.selectFrom().load();
    this.viewContainerRef.createEmbeddedView(this.templateRef, {
      // Create the embedded view with a context object that contains
      // the data via the key `$implicit`.
      $implicit: data,
    });
  }
}
```

L'input `selectFrom` désigne la source de données que lit la directive. Il utilise [`input.required()`](guide/components/inputs#required-inputs), car la directive ne peut rien faire d'utile sans source de données.

Quand Angular initialise la directive, celle-ci charge les données puis rend le template en appelant `createEmbeddedView()`. Le second argument est l'_objet de contexte_ du template : les valeurs auxquelles le template peut se lier avec des déclarations `let`. Affecter les données à la clé `$implicit` en fait la valeur par défaut que reçoit `let-data` (ou `let data` en syntaxe abrégée).

NOTE: Cet exemple rend son template une seule fois, à l'initialisation de la directive. Il ne refait pas le rendu quand la source de données liée change.

Une fois la directive fonctionnelle, envisagez d'[ajouter la prise en charge de la vérification de types du template](#typing-the-directives-context).

HELPFUL: La commande CLI [`ng generate directive`](tools/cli/schematics) génère une directive accompagnée de son fichier de test.

## Référence de la syntaxe des directives structurelles {#structural-directive-syntax-reference}

Quand vous écrivez vos propres directives structurelles, utilisez la syntaxe suivante :

```ts {hideCopy}
_: prefix = "( :let | :expression ) (';' | ',')? ( :let | :as | :keyExp )_";
```

Les motifs suivants décrivent chaque partie de la grammaire des directives structurelles :

```ts
as = :export "as" :local ";"?
keyExp = :key ":"? :expression ("as" :local)? ";"?
let = "let" :local "=" :export ";"?
```

| Mot-clé      | Détails                                            |
| :----------- | :------------------------------------------------- |
| `prefix`     | Clé d'attribut HTML                                |
| `key`        | Clé d'attribut HTML                                |
| `local`      | Nom de variable locale utilisé dans le template     |
| `export`     | Valeur exportée par la directive sous un nom donné |
| `expression` | Expression Angular standard                        |

### Comment Angular traduit la syntaxe abrégée {#how-angular-translates-shorthand}

Angular traduit la syntaxe abrégée des directives structurelles en syntaxe de binding normale de la manière suivante :

| Syntaxe abrégée                 | Traduction                                                      |
| :------------------------------ | :-------------------------------------------------------------- |
| `prefix` et `expression` seule  | `[prefix]="expression"`                                         |
| `keyExp`                        | `[prefixKey]="expression"` (le `prefix` est ajouté à la `key`)  |
| `let local`                     | `let-local="export"`                                            |

### Exemples de syntaxe abrégée {#shorthand-examples}

Le tableau suivant présente des exemples de syntaxe abrégée :

| Syntaxe abrégée                                                       | Interprétation de la syntaxe par Angular                                                                      |
| :-------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------ |
| `*myDir="let item of [1,2,3]"`                                        | `<ng-template myDir let-item [myDirOf]="[1, 2, 3]">`                                                          |
| `*myDir="let item of [1,2,3] as items; trackBy: myTrack; index as i"` | `<ng-template myDir let-item [myDirOf]="[1,2,3]" let-items="myDirOf" [myDirTrackBy]="myTrack" let-i="index">` |
| `*ngComponentOutlet="componentClass"`                                 | `<ng-template [ngComponentOutlet]="componentClass">`                                                          |
| `*ngComponentOutlet="componentClass; inputs: myInputs"`               | `<ng-template [ngComponentOutlet]="componentClass" [ngComponentOutletInputs]="myInputs">`                     |
| `*myDir="exp as value"`                                               | `<ng-template [myDir]="exp" let-value="myDir">`                                                               |

## Améliorer la vérification de types des templates pour les directives personnalisées {#improving-template-type-checking-for-custom-directives}

Vous pouvez améliorer la vérification de types des templates pour les directives personnalisées en ajoutant des guards de template à la définition de votre directive.
Ces guards aident le vérificateur de types des templates d'Angular à détecter les erreurs dans le template à la compilation, ce qui peut éviter des erreurs à l'exécution.
Deux types de guards sont possibles :

- `ngTemplateGuard_(input)` permet de contrôler comment une expression d'input doit être affinée en fonction du type d'un input précis.
- `ngTemplateContextGuard` sert à déterminer le type de l'objet de contexte du template, en fonction du type de la directive elle-même.

Cette section fournit des exemples des deux types de guards.
Pour plus d'informations, consultez [Vérification de types des templates](tools/cli/template-typecheck 'Template type-checking guide').

### Affinage de type avec les guards de template {#type-narrowing-with-template-guards}

Une directive structurelle dans un template contrôle si ce template est rendu à l'exécution. Certaines directives structurelles veulent effectuer un affinage de type en fonction du type de l'expression d'input.

Deux affinages sont possibles avec les guards d'input :

- Affiner l'expression d'input à partir d'une fonction d'assertion de type TypeScript.
- Affiner l'expression d'input en fonction de son caractère truthy.

Pour affiner l'expression d'input en définissant une fonction d'assertion de type :

```ts
// This directive only renders its template if the actor is a user.
// You want to assert that within the template, the type of the `actor`
// expression is narrowed to `User`.
@Directive(...)
class ActorIsUser {
  actor = input<User | Robot>();

  static ngTemplateGuard_actor(dir: ActorIsUser, expr: User | Robot): expr is User {
    // The return statement is unnecessary in practice, but included to
    // prevent TypeScript errors.
    return true;
  }
}
```

La vérification de types se comporte dans le template comme si `ngTemplateGuard_actor` avait été vérifié sur l'expression liée à l'input.

Certaines directives ne rendent leurs templates que lorsqu'un input est truthy. Il n'est pas possible de capturer toute la sémantique du caractère truthy dans une fonction d'assertion de type ; à la place, un type littéral `'binding'` peut indiquer au vérificateur de types des templates que l'expression du binding elle-même doit servir de guard :

```ts
@Directive(...)
class CustomIf {
  condition = input.required<boolean>();

  static ngTemplateGuard_condition: 'binding';
}
```

Le vérificateur de types des templates se comporte comme si l'expression liée à `condition` avait été vérifiée comme truthy dans le template.

### Typer le contexte de la directive {#typing-the-directives-context}

Si votre directive structurelle fournit un contexte au template instancié, vous pouvez le typer correctement dans le template en fournissant une fonction d'assertion de type statique `ngTemplateContextGuard`. Cette fonction peut utiliser le type de la directive pour en déduire le type du contexte, ce qui est utile quand le type de la directive est générique.

Pour la `SelectDirective` décrite plus haut, vous pouvez implémenter un `ngTemplateContextGuard` pour spécifier correctement le type des données, même si la source de données est générique.

```ts
// Declare an interface for the template context:
export interface SelectTemplateContext<T> {
  $implicit: T;
}

@Directive(...)
export class SelectDirective<T> {
  // The directive's generic type `T` will be inferred from the `DataSource` type
  // passed to the input.
  selectFrom = input.required<DataSource<T>>();

  // Narrow the type of the context using the generic type of the directive.
  static ngTemplateContextGuard<T>(dir: SelectDirective<T>, ctx: any): ctx is SelectTemplateContext<T> {
    // As before the guard body is not used at runtime, and included only to avoid
    // TypeScript errors.
    return true;
  }
}
```

## Et ensuite ? {#whats-next}

<docs-pill-row>
  <docs-pill href="guide/directives/directive-composition-api" title="API de composition de directives"/>
  <docs-pill href="guide/templates/ng-template" title="ng-template"/>
  <docs-pill href="guide/templates/control-flow" title="Control flow"/>
</docs-pill-row>
