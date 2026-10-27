<!-- source: 9162d3520269076769193c56867645c73a482b8c -->
# Convertir l'utilisation de CommonModule en imports standalone

Cette migration aide les projets à supprimer les imports de `CommonModule` dans les composants en ajoutant l'ensemble minimal d'imports de directive et de pipe requis par chaque template (par exemple, `NgIf`, `NgFor`, `AsyncPipe`, etc.).

Lancez le schematic avec la commande suivante :

```shell
ng generate @angular/core:common-to-standalone
```

## Options {#options}

| Option | Détails                                                                                                                       |
| :----- | :---------------------------------------------------------------------------------------------------------------------------- |
| `path` | Le chemin (relatif à la racine du projet) à migrer. Vaut `./` par défaut. Utilisez-le pour migrer progressivement une partie de votre projet. |

## Exemple {#example}

Avant :

```angular-ts
import {Component} from '@angular/core';
import {CommonModule} from '@angular/common';

@Component({
  selector: 'app-example',
  imports: [CommonModule],
  template: `
    <div *ngIf="show">
      {{ data | async | json }}
    </div>
  `,
})
export class Example {
  show = true;
  data = Promise.resolve({message: 'Hello'});
}
```

Après l'exécution de la migration (imports ajoutés au composant, CommonModule supprimé) :

```angular-ts
import {Component} from '@angular/core';
import {AsyncPipe, JsonPipe, NgIf} from '@angular/common';

@Component({
  selector: 'app-example',
  imports: [AsyncPipe, JsonPipe, NgIf],
  template: `
    <div *ngIf="show">
      {{ data | async | json }}
    </div>
  `,
})
export class Example {
  show = true;
  data = Promise.resolve({message: 'Hello'});
}
```
