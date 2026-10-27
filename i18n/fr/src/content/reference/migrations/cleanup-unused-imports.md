<!-- source: 1f710afe578ee7e7bf7a771d29191afd273ee773 -->
# Nettoyer les imports inutilisés

Depuis la version 19, Angular signale quand le tableau `imports` d'un composant contient des symboles qui ne sont pas utilisés dans son template.

Ce schematic nettoie tous les imports inutilisés du projet.

Lancez le schematic avec la commande suivante :

```shell
ng generate @angular/core:cleanup-unused-imports
```

#### Avant {#before}

```angular-ts
import {Component} from '@angular/core';
import {UnusedDirective} from './unused';

@Component({
  template: 'Hello',
  imports: [UnusedDirective],
})
export class MyComp {}
```

#### Après {#after}

```angular-ts
import {Component} from '@angular/core';

@Component({
  template: 'Hello',
  imports: [],
})
export class MyComp {}
```
