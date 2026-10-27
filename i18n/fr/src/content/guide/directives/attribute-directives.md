<!-- source: ad50cc276f2e629108f065b15f65ca6549cc362a -->
# Directives d'attribute

Les directives d'attribute modifient l'apparence ou le comportement des éléments du DOM et des composants Angular.

## Utiliser les bindings de template pour un comportement ponctuel {#use-template-bindings-for-one-off-behavior}

La syntaxe de template d'Angular couvre déjà la modification des classes, des styles, des propriétés et des événements d'un seul élément :

- Les [bindings de classe et de style](guide/templates/binding#css-class-and-style-property-bindings) ajoutent et retirent des classes CSS et des styles inline.
- Les [property bindings et attribute bindings](guide/templates/binding) définissent des propriétés du DOM et des attributs HTML.
- Les [event listeners](guide/templates/event-listeners) répondent aux interactions de l'utilisateur.

Les directives d'attribute sont utiles quand vous voulez regrouper ce type de comportement dans une unité réutilisable que vous pouvez appliquer à n'importe quel élément ou composant.

## Créer une directive d'attribute {#building-an-attribute-directive}

Une directive d'attribute personnalisée est une classe JavaScript dotée du décorateur `@Directive()`. Le `selector` du décorateur définit l'attribut qui applique la directive. Les crochets en font un attribute selector : la directive correspond donc aux éléments qui portent cet attribut. Par convention, utilisez un préfixe comme `app` pour éviter les collisions de noms :

```ts
import {Directive} from '@angular/core';

@Directive({
  selector: '[appHighlight]',
})
export class HighlightDirective {}
```

HELPFUL: La commande CLI [`ng generate directive`](tools/cli/schematics) génère une directive accompagnée de son fichier de test.

Une directive peut modifier son host de manière déclarative avec des host bindings, ou de manière impérative grâce à une référence au host element. Cet exemple [injecte](guide/di) [`ElementRef`](api/core/ElementRef) et accède à l'élément via sa propriété `nativeElement` pour passer l'arrière-plan en jaune :

<docs-code header="highlight.directive.ts" path="adev/src/content/examples/attribute-directives/src/app/highlight.directive.1.ts"/>

IMPORTANT: Les directives _ne prennent pas en charge_ les namespaces.

```angular-html {avoid}
<p app:Highlight>This is invalid</p>
```

## Appliquer une directive d'attribute {#applying-an-attribute-directive}

Pour appliquer la directive, ajoutez son selector comme attribut sur un élément :

<docs-code header="app.component.html" path="adev/src/content/examples/attribute-directives/src/app/app.component.1.html" region="applied"/>

Angular crée une instance de `HighlightDirective` pour cet élément `<p>`, injecte une référence à l'élément et passe son arrière-plan en jaune.

## Gérer les événements utilisateur {#handling-user-events}

Pour répondre aux interactions de l'utilisateur, liez les événements du host element à des méthodes de gestion via la propriété `host` du décorateur `@Directive()`. La directive suivante met en surbrillance le host element tant que le pointeur le survole et retire la surbrillance quand le pointeur en sort :

<docs-code header="highlight.directive.ts" path="adev/src/content/examples/attribute-directives/src/app/highlight.directive.2.ts"/>

La propriété `host` associe les événements `mouseenter` et `mouseleave` aux méthodes `onMouseEnter()` et `onMouseLeave()`, qui délèguent à un helper `highlight()` chargé de définir la couleur d'arrière-plan du host element. Pour en savoir plus sur les bindings d'événements du host, consultez [lier des valeurs au host element](guide/components/host-elements#binding-to-the-host-element).

## Accepter des valeurs en input {#accepting-input-values}

Comme les composants, les directives acceptent des inputs via la fonction [`input()`](guide/components/inputs). Donnez à un input le même nom que le selector pour qu'un seul binding applique la directive et lui transmette une valeur :

<docs-code header="highlight.directive.ts" path="adev/src/content/examples/attribute-directives/src/app/highlight.directive.3.ts" region="input"/>

Lisez l'input en l'appelant comme un signal, et utilisez une valeur par défaut quand aucune couleur n'est définie :

<docs-code header="highlight.directive.ts" path="adev/src/content/examples/attribute-directives/src/app/highlight.directive.3.ts" region="mouse-enter"/>

Dans le template, liez la valeur au selector. Comme l'input porte le même nom que le selector, `[appHighlight]` applique la directive et définit sa valeur. Ici, `color` est une propriété du composant :

<docs-code header="app.component.html" path="adev/src/content/examples/attribute-directives/src/app/app.component.html" region="color"/>

<docs-code header="app.component.ts" path="adev/src/content/examples/attribute-directives/src/app/app.component.ts" region="class"/>

Une directive peut déclarer plusieurs inputs. La directive suivante ajoute un input `defaultColor`, puis se rabat successivement sur `appHighlight`, `defaultColor` et enfin `red` :

<docs-code header="highlight.directive.ts" path="adev/src/content/examples/attribute-directives/src/app/highlight.directive.ts"/>

Liez les deux inputs sur le même élément. Comme `defaultColor` reçoit une chaîne statique plutôt qu'une expression dynamique, il n'a pas besoin de crochets :

<docs-code header="app.component.html" path="adev/src/content/examples/attribute-directives/src/app/app.component.html" region="defaultColor"/>

## Désactiver le traitement d'Angular avec `NgNonBindable` {#deactivating-angular-processing-with-ngnonbindable}

Pour empêcher l'évaluation des expressions dans le navigateur, ajoutez `ngNonBindable` au host element.
`ngNonBindable` désactive l'interpolation, les directives et le binding dans les templates.

Dans l'exemple suivant, l'expression `{{ 1 + 1 }}` s'affiche telle qu'elle apparaît dans votre éditeur de code, et n'affiche pas `2`.

<docs-code header="app.component.html" path="adev/src/content/examples/attribute-directives/src/app/app.component.html" region="ngNonBindable"/>

Appliquer `ngNonBindable` à un élément arrête le binding pour les éléments enfants de cet élément.
Cependant, `ngNonBindable` laisse les directives fonctionner sur l'élément auquel vous appliquez `ngNonBindable`.
Dans l'exemple suivant, la directive `appHighlight` reste active, mais Angular n'évalue pas l'expression `{{ 1 + 1 }}`.

<docs-code header="app.component.html" path="adev/src/content/examples/attribute-directives/src/app/app.component.html" region="ngNonBindable-with-directive"/>

Si vous appliquez `ngNonBindable` à un élément parent, Angular désactive l'interpolation et tout type de binding, comme le property binding ou l'event binding, pour les enfants de cet élément.

## Et ensuite ? {#whats-next}

<docs-pill-row>
  <docs-pill href="guide/directives/structural-directives" title="Directives structurelles"/>
  <docs-pill href="guide/directives/directive-composition-api" title="API de composition de directives"/>
</docs-pill-row>
