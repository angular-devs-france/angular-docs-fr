<!-- source: 62a6523d5e41e13999f85e50c8b160115f301cb7 -->
# Two-way binding

Le **two-way binding** est un raccourci pour lier simultanément une valeur à un élément, tout en donnant à cet élément la possibilité de propager les modifications en retour via ce binding.

## Syntaxe {#syntax}

La syntaxe du two-way binding est une combinaison de crochets et de parenthèses, `[()]`. Elle combine la syntaxe du property binding, `[]`, et celle de l'event binding, `()`. La communauté Angular appelle familièrement cette syntaxe « banana-in-a-box ».

## Two-way binding avec les contrôles de formulaire {#two-way-binding-with-form-controls}

Les développeurs utilisent couramment le two-way binding pour garder les données d'un composant synchronisées avec un contrôle de formulaire pendant que l'utilisateur interagit avec ce contrôle. Par exemple, quand un utilisateur remplit un champ de saisie de texte, l'état du composant doit être mis à jour.

L'exemple suivant met à jour dynamiquement l'attribut `firstName` sur la page :

```angular-ts
import {Component} from '@angular/core';
import {FormsModule} from '@angular/forms';

@Component({
  imports: [FormsModule],
  template: `
    <main>
      <h2>Hello {{ firstName }}!</h2>
      <input type="text" [(ngModel)]="firstName" />
    </main>
  `,
})
export class App {
  firstName = 'Ada';
}
```

Pour utiliser le two-way binding avec des contrôles de formulaire natifs, vous devez :

1. Importer le `FormsModule` depuis `@angular/forms`
1. Utiliser la directive `ngModel` avec la syntaxe de two-way binding (par exemple, `[(ngModel)]`)
1. Lui assigner l'état que vous voulez qu'elle mette à jour (par exemple, `firstName`)

Une fois cette configuration en place, Angular s'assure que toute modification dans le champ de saisie de texte se reflète correctement dans l'état du composant !

Pour en savoir plus sur [`NgModel`](/api/forms/NgModel), consultez la documentation officielle.

## Two-way binding entre composants {#two-way-binding-between-components}

Mettre en place un two-way binding entre un composant parent et un composant enfant demande plus de configuration que pour les éléments de formulaire.

Voici un exemple où `App` est responsable de définir l'état initial du compteur, tandis que la logique de mise à jour et de rendu de l'interface du compteur se trouve principalement dans son enfant `Counter`.

```angular-ts {header: 'app.ts'}
import {Component} from '@angular/core';
import {Counter} from './counter';

@Component({
  selector: 'app-root',
  imports: [Counter],
  template: `
    <main>
      <h1>Counter: {{ initialCount }}</h1>
      <app-counter [(count)]="initialCount"></app-counter>
    </main>
  `,
})
export class App {
  initialCount = 18;
}
```

```angular-ts {header: 'counter.ts'}
import {Component, model} from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <button (click)="updateCount(-1)">-</button>
    <span>{{ count() }}</span>
    <button (click)="updateCount(+1)">+</button>
  `,
})
export class Counter {
  count = model<number>(0);

  updateCount(amount: number): void {
    this.count.update((currentCount) => currentCount + amount);
  }
}
```

### Activer le two-way binding entre composants {#enabling-two-way-binding-between-components}

Si l'on réduit l'exemple ci-dessus à l'essentiel, chaque two-way binding entre composants nécessite ce qui suit :

Le composant enfant doit contenir une propriété `model`.

Voici un exemple simplifié :

```angular-ts {header: 'counter.ts'}
import {Component, model} from '@angular/core';

@Component({
  /* Omitted for brevity */
})
export class Counter {
  count = model<number>(0);

  updateCount(amount: number): void {
    this.count.update((currentCount) => currentCount + amount);
  }
}
```

Le composant parent doit :

1. Entourer le nom de la propriété `model` avec la syntaxe de two-way binding.
1. Assigner une propriété ou un signal à la propriété `model`.

Voici un exemple simplifié :

```angular-ts {header: 'app.ts'}
import {Component} from '@angular/core';
import {Counter} from './counter';

@Component({
  selector: 'app-root',
  imports: [Counter],
  template: `
    <main>
      <app-counter [(count)]="initialCount"></app-counter>
    </main>
  `,
})
export class App {
  initialCount = 18;
}
```
