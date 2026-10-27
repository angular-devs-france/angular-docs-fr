<!-- source: 563cdcd96e6c6bd4e0d1463151d4e49339faa663 -->
# Glisser-déposer

## Vue d'ensemble {#overview}

Cette page décrit les directives de glisser-déposer, qui vous permettent de créer rapidement des interfaces de glisser-déposer avec :

- Le glissement libre
- La création d'une liste d'éléments déplaçables réordonnables
- Le transfert d'éléments déplaçables entre des listes
- Des animations de glissement
- Le verrouillage des éléments déplaçables le long d'un axe ou dans un élément
- L'ajout de poignées de glissement personnalisées
- L'ajout d'aperçus pendant le glissement
- L'ajout d'un placeholder de glissement personnalisé

Pour la référence complète de l'API, consultez la [page de référence de l'API de glisser-déposer du CDK Angular](api#angular_cdk_drag-drop).

## Avant de commencer {#before-you-start}

### Installation du CDK {#cdk-installation}

Le [Component Dev Kit (CDK)](https://material.angular.dev/cdk/categories) est un ensemble de primitives de comportement pour construire des composants. Pour utiliser les directives de glisser-déposer, installez d'abord `@angular/cdk` depuis npm. Vous pouvez le faire depuis votre terminal avec Angular CLI :

```shell
ng add @angular/cdk
```

### Importer le glisser-déposer {#importing-drag-and-drop}

Pour utiliser le glisser-déposer, importez dans votre composant les directives dont vous avez besoin.

```ts
import {Component} from '@angular/core';
import {CdkDrag} from '@angular/cdk/drag-drop';

@Component({
  selector: 'drag-drop-example',
  templateUrl: 'drag-drop-example.html',
  imports: [CdkDrag],
})
export class DragDropExample {}
```

## Créer des éléments déplaçables {#create-draggable-elements}

Vous pouvez rendre n'importe quel élément déplaçable en ajoutant la directive `cdkDrag`. Par défaut, tous les éléments déplaçables prennent en charge le glissement libre.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/overview/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/overview/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/overview/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/overview/app/app.css"/>
</docs-code-multifile>

## Créer une liste d'éléments déplaçables réordonnables {#create-a-list-of-reorderable-draggable-elements}

Ajoutez la directive `cdkDropList` à un élément parent pour regrouper des éléments déplaçables dans une collection réordonnable. Elle définit où les éléments déplaçables peuvent être déposés. Les éléments déplaçables du groupe de la liste de dépôt se réorganisent automatiquement lorsqu'un élément se déplace.

Les directives de glisser-déposer ne mettent pas à jour votre modèle de données. Pour mettre à jour le modèle de données, écoutez l'événement `cdkDropListDropped` (une fois que l'utilisateur a fini de faire glisser) et mettez à jour le modèle de données manuellement.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/sorting/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/sorting/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/sorting/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/sorting/app/app.css"/>
</docs-code-multifile>

Vous pouvez utiliser le jeton d'injection `CDK_DROP_LIST` pour référencer des instances de `cdkDropList`. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di) et l'[API du jeton d'injection de la liste de dépôt](api/cdk/drag-drop/CDK_DROP_LIST).

## Transférer des éléments déplaçables entre des listes {#transfer-draggable-elements-between-lists}

La directive `cdkDropList` permet de transférer des éléments déplaçables entre des listes de dépôt connectées. Il existe deux façons de connecter une ou plusieurs instances de `cdkDropList` entre elles :

- Définir la propriété `cdkDropListConnectedTo` sur une autre liste de dépôt.
- Envelopper les éléments dans un élément portant l'attribut `cdkDropListGroup`.

La directive `cdkDropListConnectedTo` fonctionne aussi bien avec une référence directe à une autre `cdkDropList` qu'en référençant l'id d'un autre conteneur de dépôt.

```html
<!-- This is valid -->
<div cdkDropList #listOne="cdkDropList" [cdkDropListConnectedTo]="[listTwo]"></div>
<div cdkDropList #listTwo="cdkDropList" [cdkDropListConnectedTo]="[listOne]"></div>

<!-- This is valid as well -->
<div cdkDropList id="list-one" [cdkDropListConnectedTo]="['list-two']"></div>
<div cdkDropList id="list-two" [cdkDropListConnectedTo]="['list-one']"></div>
```

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/connected-sorting/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/connected-sorting/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/connected-sorting/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/connected-sorting/app/app.css"/>
</docs-code-multifile>

Utilisez la directive `cdkDropListGroup` si vous avez un nombre inconnu de listes de dépôt connectées, afin d'établir la connexion automatiquement. Toute nouvelle `cdkDropList` ajoutée dans un groupe se connecte automatiquement à toutes les autres listes.

```angular-html
<div cdkDropListGroup>
  <!-- All lists in here will be connected. -->
  @for (list of lists; track list) {
    <div cdkDropList></div>
  }
</div>
```

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/connected-sorting-group/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/connected-sorting-group/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/connected-sorting-group/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/connected-sorting-group/app/app.css"/>
</docs-code-multifile>

Vous pouvez utiliser le jeton d'injection `CDK_DROP_LIST_GROUP` pour référencer des instances de `cdkDropListGroup`. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di) et l'[API du jeton d'injection du groupe de listes de dépôt](api/cdk/drag-drop/CDK_DROP_LIST_GROUP).

### Glissement sélectif {#selective-dragging}

Par défaut, un utilisateur peut déplacer des éléments `cdkDrag` d'un conteneur vers un autre conteneur connecté. Pour contrôler plus finement quels éléments peuvent être déposés dans un conteneur, utilisez `cdkDropListEnterPredicate`. Angular appelle le prédicat chaque fois qu'un élément déplaçable entre dans un nouveau conteneur. Selon que le prédicat renvoie true ou false, l'élément est autorisé ou non à entrer dans le nouveau conteneur.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/enter-predicate/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/enter-predicate/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/enter-predicate/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/enter-predicate/app/app.css"/>
</docs-code-multifile>

## Associer des données {#attach-data}

Vous pouvez associer des données arbitraires à `cdkDrag` et à `cdkDropList` en définissant respectivement `cdkDragData` ou `cdkDropListData`. Vous pouvez vous lier aux événements émis par ces deux directives, qui incluent ces données, ce qui vous permet d'identifier facilement l'origine de l'interaction de glissement ou de dépôt.

```angular-html
@for (list of lists; track list) {
  <div cdkDropList [cdkDropListData]="list" (cdkDropListDropped)="drop($event)">
    @for (item of list; track item) {
      <div cdkDrag [cdkDragData]="item"></div>
    }
  </div>
}
```

## Personnaliser le glissement {#dragging-customizations}

### Personnaliser la poignée de glissement {#customize-drag-handle}

Par défaut, l'utilisateur peut faire glisser l'ensemble de l'élément `cdkDrag` pour le déplacer. Pour qu'il ne puisse le faire qu'au moyen d'un élément servant de poignée, ajoutez la directive `cdkDragHandle` à un élément situé à l'intérieur de `cdkDrag`. Vous pouvez avoir autant d'éléments `cdkDragHandle` que vous le souhaitez.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/custom-handle/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/custom-handle/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/custom-handle/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/custom-handle/app/app.css"/>
</docs-code-multifile>

Vous pouvez utiliser le jeton d'injection `CDK_DRAG_HANDLE` pour référencer des instances de `cdkDragHandle`. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di) et l'[API du jeton d'injection de la poignée de glissement](api/cdk/drag-drop/CDK_DRAG_HANDLE).

### Personnaliser l'aperçu de glissement {#customize-drag-preview}

Un élément d'aperçu devient visible lorsqu'un élément `cdkDrag` est en cours de glissement. Par défaut, l'aperçu est un clone de l'élément d'origine, positionné à côté du curseur de l'utilisateur.

Pour personnaliser l'aperçu, fournissez un template personnalisé via `*cdkDragPreview`. L'aperçu personnalisé n'adopte pas la taille de l'élément d'origine en cours de glissement, car aucune hypothèse n'est faite sur le contenu de l'élément. Pour que l'aperçu de glissement adopte la taille de l'élément, passez true à l'input `matchSize`.

L'élément cloné perd son attribut id afin d'éviter d'avoir plusieurs éléments avec le même id dans la page. Par conséquent, tout CSS qui cible cet id ne sera pas appliqué.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/custom-preview/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/custom-preview/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/custom-preview/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/custom-preview/app/app.css"/>
</docs-code-multifile>

Vous pouvez utiliser le jeton d'injection `CDK_DRAG_PREVIEW` pour référencer des instances de `cdkDragPreview`. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di) et l'[API du jeton d'injection de l'aperçu de glissement](api/cdk/drag-drop/CDK_DRAG_PREVIEW).

### Personnaliser le point d'insertion du glissement {#customize-drag-insertion-point}

Par défaut, Angular insère l'aperçu de `cdkDrag` dans le `<body>` de la page afin d'éviter les problèmes de positionnement et de débordement. Cela peut ne pas être souhaitable dans certains cas, car les styles hérités ne seront pas appliqués à l'aperçu.

Vous pouvez changer l'endroit où Angular insère l'aperçu grâce à l'input `cdkDragPreviewContainer` de `cdkDrag`. Les valeurs possibles sont :

| Valeur                        | Description                                                                            | Avantages                                                                                                                   | Inconvénients                                                                                                                                                             |
| :---------------------------- | :------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `global`                      | Valeur par défaut. Angular insère l'aperçu dans le <body> ou dans la shadow root la plus proche. | L'aperçu n'est pas affecté par `z-index` ni par `overflow: hidden`. Il n'affecte pas non plus les selectors `:nth-child` ni les mises en page flex. | Ne conserve pas les styles hérités.                                                                                                                                       |
| `parent`                      | Angular insère l'aperçu dans le parent de l'élément en cours de glissement.            | L'aperçu hérite des mêmes styles que l'élément glissé.                                                                      | L'aperçu peut être rogné par `overflow: hidden` ou placé sous d'autres éléments à cause de `z-index`. De plus, il peut affecter les selectors `:nth-child` et certaines mises en page flex. |
| `ElementRef` ou `HTMLElement` | Angular insère l'aperçu dans l'élément spécifié.                                       | L'aperçu hérite des styles de l'élément conteneur spécifié.                                                                 | L'aperçu peut être rogné par `overflow: hidden` ou placé sous d'autres éléments à cause de `z-index`. De plus, il peut affecter les selectors `:nth-child` et certaines mises en page flex. |

Vous pouvez aussi modifier le jeton d'injection `CDK_DRAG_CONFIG` pour mettre à jour `previewContainer` dans la configuration si la valeur est `global` ou `parent`. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di), l'[API du jeton d'injection de la configuration de glissement](api/cdk/drag-drop/CDK_DRAG_CONFIG) et l'[API de configuration du glisser-déposer](api/cdk/drag-drop/DragDropConfig).

### Personnaliser le placeholder de glissement {#customize-drag-placeholder}

Pendant qu'un élément `cdkDrag` est en cours de glissement, la directive crée un élément placeholder qui indique où l'élément sera placé une fois déposé. Par défaut, le placeholder est un clone de l'élément en cours de glissement. Vous pouvez remplacer le placeholder par un placeholder personnalisé grâce à la directive `*cdkDragPlaceholder` :

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/custom-placeholder/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/custom-placeholder/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/custom-placeholder/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/custom-placeholder/app/app.css"/>
</docs-code-multifile>

Vous pouvez utiliser le jeton d'injection `CDK_DRAG_PLACEHOLDER` pour référencer des instances de `cdkDragPlaceholder`. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di) et l'[API du jeton d'injection du placeholder de glissement](api/cdk/drag-drop/CDK_DRAG_PLACEHOLDER).

### Personnaliser l'élément racine du glissement {#customize-drag-root-element}

Définissez l'attribut `cdkDragRootElement` s'il existe un élément que vous souhaitez rendre déplaçable mais auquel vous n'avez pas accès directement.

L'attribut accepte un selector et remonte le DOM jusqu'à trouver un élément qui correspond au selector. Si un élément est trouvé, il devient déplaçable. C'est utile par exemple pour rendre une boîte de dialogue déplaçable.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/root-element/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/root-element/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/root-element/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/root-element/app/app.css"/>
</docs-code-multifile>

Vous pouvez aussi modifier le jeton d'injection `CDK_DRAG_CONFIG` pour mettre à jour `rootElementSelector` dans la configuration. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di), l'[API du jeton d'injection de la configuration de glissement](api/cdk/drag-drop/CDK_DRAG_CONFIG) et l'[API de configuration du glisser-déposer](api/cdk/drag-drop/DragDropConfig).

### Définir la position dans le DOM d'un élément déplaçable {#set-dom-position-of-a-draggable-element}

Par défaut, les éléments `cdkDrag` qui ne sont pas dans une `cdkDropList` ne quittent leur position normale dans le DOM que lorsqu'un utilisateur déplace manuellement l'élément. Utilisez l'input `cdkDragFreeDragPosition` pour définir explicitement la position de l'élément. Un cas d'usage courant consiste à restaurer la position d'un élément déplaçable après qu'un utilisateur a quitté la page puis y est revenu.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/free-drag-position/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/free-drag-position/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/free-drag-position/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/free-drag-position/app/app.css"/>
</docs-code-multifile>

### Restreindre le déplacement à l'intérieur d'un élément {#restrict-movement-within-an-element}

Pour empêcher l'utilisateur de faire glisser un élément `cdkDrag` hors d'un autre élément, passez un selector CSS à l'attribut `cdkDragBoundary`. Cet attribut accepte un selector et remonte le DOM jusqu'à trouver un élément qui y correspond. Si une correspondance est trouvée, l'élément devient la limite hors de laquelle l'élément déplaçable ne peut pas être glissé. `cdkDragBoundary` peut aussi être utilisé lorsque `cdkDrag` est placé dans une `cdkDropList`.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/boundary/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/boundary/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/boundary/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/boundary/app/app.css"/>
</docs-code-multifile>

Vous pouvez aussi modifier le jeton d'injection `CDK_DRAG_CONFIG` pour mettre à jour boundaryElement dans la configuration. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di), l'[API du jeton d'injection de la configuration de glissement](api/cdk/drag-drop/CDK_DRAG_CONFIG) et l'[API de configuration du glisser-déposer](api/cdk/drag-drop/DragDropConfig).

### Restreindre le déplacement le long d'un axe {#restrict-movement-along-an-axis}

Par défaut, `cdkDrag` permet un déplacement libre dans toutes les directions. Pour restreindre le glissement à un axe précis, définissez `cdkDragLockAxis` à « x » ou « y » sur `cdkDrag`. Pour restreindre le glissement de plusieurs éléments déplaçables dans une `cdkDropList`, définissez plutôt `cdkDropListLockAxis` sur `cdkDropList`.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/axis-lock/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/axis-lock/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/axis-lock/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/axis-lock/app/app.css"/>
</docs-code-multifile>

Vous pouvez aussi modifier le jeton d'injection `CDK_DRAG_CONFIG` pour mettre à jour `lockAxis` dans la configuration. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di), l'[API du jeton d'injection de la configuration de glissement](api/cdk/drag-drop/CDK_DRAG_CONFIG) et l'[API de configuration du glisser-déposer](api/cdk/drag-drop/DragDropConfig).

### Retarder le glissement {#delay-dragging}

Par défaut, lorsque l'utilisateur appuie avec son pointeur sur un `cdkDrag`, la séquence de glissement commence. Ce comportement peut ne pas être souhaitable, par exemple pour des éléments déplaçables en plein écran sur des appareils tactiles, où l'utilisateur risque de déclencher accidentellement un événement de glissement en faisant défiler la page.

Vous pouvez retarder la séquence de glissement grâce à l'input `cdkDragStartDelay`. L'input attend que l'utilisateur maintienne son pointeur enfoncé pendant le nombre de millisecondes spécifié avant de faire glisser l'élément.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/delay-drag/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/delay-drag/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/delay-drag/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/delay-drag/app/app.css"/>
</docs-code-multifile>

Vous pouvez aussi modifier le jeton d'injection `CDK_DRAG_CONFIG` pour mettre à jour dragStartDelay dans la configuration. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di), l'[API du jeton d'injection de la configuration de glissement](api/cdk/drag-drop/CDK_DRAG_CONFIG) et l'[API de configuration du glisser-déposer](api/cdk/drag-drop/DragDropConfig).

### Désactiver le glissement {#disable-dragging}

Pour désactiver le glissement d'un élément particulier, définissez l'input `cdkDragDisabled` d'un élément `cdkDrag` à true ou false. Vous pouvez désactiver une liste entière avec l'input `cdkDropListDisabled` d'une `cdkDropList`. Il est également possible de désactiver une poignée précise via `cdkDragHandleDisabled` sur `cdkDragHandle`.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/disable-drag/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/disable-drag/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/disable-drag/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/disable-drag/app/app.css"/>
</docs-code-multifile>

Vous pouvez aussi modifier le jeton d'injection `CDK_DRAG_CONFIG` pour mettre à jour `draggingDisabled` dans la configuration. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di), l'[API du jeton d'injection de la configuration de glissement](api/cdk/drag-drop/CDK_DRAG_CONFIG) et l'[API de configuration du glisser-déposer](api/cdk/drag-drop/DragDropConfig).

## Personnaliser le tri {#sorting-customizations}

### Orientation de la liste {#list-orientation}

Par défaut, la directive `cdkDropList` suppose que les listes sont verticales. Vous pouvez changer cela en définissant la propriété `cdkDropListOrientation` à horizontal.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/horizontal-sorting/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/horizontal-sorting/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/horizontal-sorting/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/horizontal-sorting/app/app.css"/>
</docs-code-multifile>

Vous pouvez aussi modifier le jeton d'injection `CDK_DRAG_CONFIG` pour mettre à jour `listOrientation` dans la configuration. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di), l'[API du jeton d'injection de la configuration de glissement](api/cdk/drag-drop/CDK_DRAG_CONFIG) et l'[API de configuration du glisser-déposer](api/cdk/drag-drop/DragDropConfig).

### Retour à la ligne de la liste {#list-wrapping}

Par défaut, `cdkDropList` trie les éléments déplaçables en les déplaçant au moyen d'une transformation CSS. Cela permet d'animer le tri, ce qui offre une meilleure expérience utilisateur. En contrepartie, la liste de dépôt ne fonctionne que dans une seule direction : verticalement ou horizontalement.

Si vous avez une liste triable qui doit passer à la ligne, vous pouvez définir l'attribut `cdkDropListOrientation` à `mixed`. La liste utilise alors une autre stratégie de tri des éléments, qui consiste à les déplacer dans le DOM. En revanche, la liste ne peut plus animer l'action de tri.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/mixed-sorting/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/mixed-sorting/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/mixed-sorting/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/mixed-sorting/app/app.css"/>
</docs-code-multifile>

### Tri sélectif {#selective-sorting}

Par défaut, les éléments `cdkDrag` peuvent être triés à n'importe quelle position dans une `cdkDropList`. Pour changer ce comportement, définissez l'attribut `cdkDropListSortPredicate`, qui reçoit une fonction. La fonction prédicat est appelée chaque fois qu'un élément déplaçable est sur le point d'être déplacé vers un nouvel index de la liste de dépôt. Si le prédicat renvoie true, l'élément est déplacé vers le nouvel index ; sinon, il conserve sa position actuelle.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/sort-predicate/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/sort-predicate/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/sort-predicate/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/sort-predicate/app/app.css"/>
</docs-code-multifile>

### Désactiver le tri {#disable-sorting}

Dans certains cas, des éléments déplaçables peuvent être glissés d'une `cdkDropList` vers une autre, sans que l'utilisateur puisse les trier dans la liste source. Dans ces cas, ajoutez l'attribut `cdkDropListSortingDisabled` pour empêcher le tri des éléments déplaçables d'une `cdkDropList`. L'élément glissé conserve alors sa position initiale dans la liste source s'il n'est pas déposé à une nouvelle position valide.

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/disable-sorting/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/disable-sorting/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/disable-sorting/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/disable-sorting/app/app.css"/>
</docs-code-multifile>

Vous pouvez aussi modifier le jeton d'injection `CDK_DRAG_CONFIG` pour mettre à jour sortingDisabled dans la configuration. Pour plus d'informations, consultez le [guide de l'injection de dépendances](/guide/di), l'[API du jeton d'injection de la configuration de glissement](api/cdk/drag-drop/CDK_DRAG_CONFIG) et l'[API de configuration du glisser-déposer](api/cdk/drag-drop/DragDropConfig).

### Copier des éléments entre des listes {#copying-items-between-lists}

Par défaut, lorsqu'un élément est glissé d'une liste vers une autre, il est retiré de sa liste d'origine. Vous pouvez toutefois configurer les directives pour copier l'élément, en laissant l'élément d'origine dans la liste source.

Pour activer la copie, définissez l'input `cdkDropListHasAnchor`. Il indique à `cdkDropList` de créer un élément « ancre » qui reste dans le conteneur d'origine et ne se déplace pas avec l'élément. Si l'utilisateur ramène l'élément dans le conteneur d'origine, l'ancre est retirée automatiquement. Vous pouvez styliser l'élément ancre en ciblant la classe CSS `.cdk-drag-anchor`.

Combiner `cdkDropListHasAnchor` avec `cdkDropListSortingDisabled` permet de construire une liste depuis laquelle un utilisateur peut copier des éléments sans pouvoir réordonner la liste source (par exemple une liste de produits et un panier d'achat).

<docs-code-multifile preview path="adev/src/content/examples/drag-drop/src/copy-list/app/app.ts">
  <docs-code header="app.html" path="adev/src/content/examples/drag-drop/src/copy-list/app/app.html"/>
  <docs-code header="app.ts" path="adev/src/content/examples/drag-drop/src/copy-list/app/app.ts"/>
  <docs-code header="app.css" path="adev/src/content/examples/drag-drop/src/copy-list/app/app.css"/>
</docs-code-multifile>

## Personnaliser les animations {#customize-animations}

Le glisser-déposer prend en charge les animations pour :

- Le tri d'un élément déplaçable dans une liste
- Le déplacement de l'élément déplaçable depuis la position où l'utilisateur l'a déposé jusqu'à sa position finale dans la liste

Pour configurer vos animations, définissez une transition CSS qui cible la propriété transform. Les classes suivantes peuvent être utilisées pour les animations :

| Nom de la classe CSS | Résultat de l'ajout d'une transition                                                                                                                                                                       |
| :------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| .cdk-drag           | Anime les éléments déplaçables pendant leur tri.                                                                                                                                                           |
| .cdk-drag-animating | Anime l'élément déplaçable depuis la position où il a été déposé jusqu'à sa position finale dans la `cdkDropList`.<br><br>Cette classe CSS n'est appliquée à un élément `cdkDrag` qu'une fois l'action de glissement terminée. |

## Styles {#styling}

Les directives `cdkDrag` et `cdkDropList` n'appliquent que les styles essentiels à leur fonctionnement. Les applications peuvent personnaliser leurs styles en ciblant les classes CSS suivantes.

| Nom de la classe CSS     | Description                                                                                                                                                                                                                                                                                             |
| :----------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| .cdk-drop-list           | Selector des éléments conteneurs `cdkDropList`.                                                                                                                                                                                                                                                         |
| .cdk-drag                | Selector des éléments `cdkDrag`.                                                                                                                                                                                                                                                                        |
| .cdk-drag-disabled       | Selector des éléments `cdkDrag` désactivés.                                                                                                                                                                                                                                                             |
| .cdk-drag-handle         | Selector du host element de `cdkDragHandle`.                                                                                                                                                                                                                                                            |
| .cdk-drag-preview        | Selector de l'élément d'aperçu de glissement. C'est l'élément qui apparaît à côté du curseur lorsqu'un utilisateur fait glisser un élément dans une liste triable.<br><br>Cet élément est identique à l'élément en cours de glissement, sauf s'il est personnalisé avec un template personnalisé via `*cdkDragPreview`. |
| .cdk-drag-placeholder    | Selector de l'élément placeholder de glissement. C'est l'élément affiché à l'endroit où l'élément déplaçable sera déposé une fois l'action de glissement terminée.<br><br>Cet élément est identique à l'élément en cours de tri, sauf s'il est personnalisé avec la directive cdkDragPlaceholder. |
| .cdk-drop-list-dragging  | Selector de l'élément conteneur `cdkDropList` qui contient un élément déplaçable en cours de glissement.                                                                                                                                                                                                 |
| .cdk-drop-list-disabled  | Selector des éléments conteneurs `cdkDropList` désactivés.                                                                                                                                                                                                                                              |
| .cdk-drop-list-receiving | Selector de l'élément conteneur `cdkDropList` qui peut recevoir un élément déplaçable en cours de glissement depuis une liste de dépôt connectée.                                                                                                                                                         |
| .cdk-drag-anchor         | Selector de l'élément ancre créé lorsque `cdkDropListHasAnchor` est activé. Cet élément indique la position d'où l'élément glissé est parti.                                                                                                                                                             |

## Glisser dans un conteneur avec défilement {#dragging-in-a-scrollable-container}

Si vos éléments déplaçables se trouvent dans un conteneur avec défilement (par exemple une `div` avec `overflow: auto`), le défilement automatique ne fonctionne pas tant que le conteneur n'a pas la directive `cdkScrollable`. Sans elle, le CDK ne peut ni détecter ni contrôler le comportement de défilement du conteneur pendant les opérations de glissement.

## Intégrations avec d'autres composants {#integrations-with-other-components}

La fonctionnalité de glisser-déposer du CDK peut être intégrée à différents composants. Les cas d'usage courants incluent des composants `MatTable` triables et des composants `MatTabGroup` triables.
