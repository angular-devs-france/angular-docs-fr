<!-- source: 0a9050dc4824c2d15018e03e586a3a3df09b2621 -->
<docs-decorative-header title="Directives" imgSrc="adev/src/assets/images/directives.svg"> <!-- markdownlint-disable-line -->
Les directives ajoutent du comportement aux éléments et aux composants de vos applications Angular.
</docs-decorative-header>

Une directive peut modifier l'apparence d'un élément, son comportement ou sa place dans le DOM. Angular fournit plusieurs directives intégrées, et vous pouvez écrire les vôtres.

## Quand utiliser une directive {#when-to-use-a-directive}

Les directives sont les plus efficaces quand elles encapsulent un comportement **réutilisable** que vous voulez appliquer à un élément ou à un composant existant.

Exemples courants :

- Appliquer la même apparence ou le même comportement à de nombreux éléments, comme l'autofocus ou une infobulle.
- Lire ou modifier le DOM, les attributs ou les classes du host element.
- Ajouter un comportement à un composant dont vous n'êtes pas propriétaire sans modifier son code source.

Si vous devez rendre votre propre balisage ou gérer une partie de l'interface avec son propre template, tournez-vous vers un [composant](guide/components), une directive spécialisée dotée de son propre template.

## Un exemple rapide {#a-quick-example}

Supposons que vous vouliez que des éléments soient mis en surbrillance quand l'utilisateur les survole avec la souris, en passant leur couleur d'arrière-plan au jaune. Plutôt que de répéter la même logique de gestion d'événements sur chaque élément, vous pouvez regrouper ce comportement dans une directive et l'appliquer partout où vous en avez besoin.

La directive `appHighlight` suivante définit la couleur d'arrière-plan du host element quand la souris entre et l'efface quand la souris sort :

```ts
import {Directive, signal} from '@angular/core';

@Directive({
  selector: '[appHighlight]',
  host: {
    '(mouseenter)': 'isHovered.set(true)',
    '(mouseleave)': 'isHovered.set(false)',
    '[style.background-color]': 'isHovered() ? "yellow" : null',
  },
})
export class HighlightDirective {
  protected isHovered = signal(false);
}
```

Les métadonnées `host` écoutent les événements de la souris pour mettre à jour le signal `isHovered`, et lient le style `background-color` du host element à la valeur du signal.

Appliquez la directive en ajoutant son selector comme attribut sur un élément :

```angular-html
<p appHighlight>Highlight me!</p>
```

Chaque élément qui porte l'attribut `appHighlight` reçoit le même comportement au survol, avec une logique définie en un seul endroit.

## Types de directives {#types-of-directives}

Angular possède trois principaux types de directives :

| Type de directive                                               | Détails                                                                           |
| :-------------------------------------------------------------- | :-------------------------------------------------------------------------------- |
| [Composants](guide/components)                                  | Définissent une interface réutilisable avec leur propre template.                 |
| [Directives d'attribute](guide/directives/attribute-directives) | Modifient l'apparence ou le comportement d'un élément, d'un composant ou d'une autre directive. |
| [Directives structurelles](guide/directives/structural-directives) | Modifient la structure du DOM en ajoutant et en retirant des éléments du DOM.  |

## Et ensuite ? {#whats-next}

Apprenez-en plus sur chaque type de directive dans les guides suivants.

<docs-pill-row>
  <docs-pill href="guide/directives/attribute-directives" title="Directives d'attribute"/>
  <docs-pill href="guide/directives/structural-directives" title="Directives structurelles"/>
  <docs-pill href="guide/directives/directive-composition-api" title="API de composition de directives"/>
</docs-pill-row>
