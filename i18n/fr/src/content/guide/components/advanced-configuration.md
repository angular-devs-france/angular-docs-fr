<!-- source: f452e2a1482e4e9241fbeebdb35996aa638bbaac -->
# Configuration avancée des composants

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Commencez par celui-ci si vous découvrez Angular.

## ChangeDetectionStrategy {#changedetectionstrategy}

Le décorateur `@Component` accepte une option `changeDetection` qui contrôle le **mode de détection
de changements** du composant. Il existe deux modes de détection de changements.

**`ChangeDetectionStrategy.Eager`/`Default`** est un mode facultatif. Dans ce mode,
Angular vérifie si le DOM du composant doit être mis à jour chaque fois qu'une activité a pu se produire
n'importe où dans l'application. Les activités qui déclenchent cette vérification incluent les interactions de l'utilisateur, les réponses réseau,
les timers, et plus encore.

**`ChangeDetectionStrategy.OnPush`** est la stratégie par défaut (depuis la v22). Ce mode réduit la quantité de vérifications qu'Angular
doit effectuer. Dans ce mode, le framework vérifie si le DOM d'un composant doit être mis à jour uniquement quand :

- Un input du composant change à la suite d'une liaison dans un template, ou
- Un écouteur d'événements de ce composant s'exécute
- Le composant est explicitement marqué pour vérification, via `ChangeDetectorRef.markForCheck` ou quelque chose qui l'encapsule, comme `AsyncPipe`.

De plus, quand un composant OnPush est vérifié, Angular vérifie _aussi_ tous ses composants
ancêtres, en remontant l'arbre de l'application.

## PreserveWhitespaces {#preservewhitespaces}

Par défaut, Angular supprime et réduit les espaces superflus dans les templates, le plus souvent ceux
des retours à la ligne et de l'indentation. Vous pouvez modifier ce réglage en définissant explicitement `preserveWhitespaces` à
`true` dans les métadonnées d'un composant.

## Schémas d'éléments personnalisés {#custom-element-schemas}

Par défaut, Angular lève une erreur quand il rencontre un élément HTML inconnu. Vous pouvez
désactiver ce comportement pour un composant en incluant `CUSTOM_ELEMENTS_SCHEMA` dans la propriété `schemas`
des métadonnées de votre composant.

```angular-ts
import {Component, CUSTOM_ELEMENTS_SCHEMA} from '@angular/core';

@Component({
  ...,
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
  template: '<some-unknown-component />'
})
export class ComponentWithCustomElements { }
```

Angular fournit aussi `NO_ERRORS_SCHEMA`, qui autorise n'importe quel élément et n'importe quelle propriété.
