<!-- source: 4f904a0747ab35cabcd88bde4de0c74356be2ded -->
# Migration vers la fonction `inject`

La fonction [`inject`](/api/core/inject) d'Angular offre des types plus précis et une meilleure compatibilité avec les décorateurs standards que l'injection par constructeur.

Ce schematic convertit l'injection par constructeur de vos classes pour utiliser la fonction [`inject`](/api/core/inject) à la place.

Lancez le schematic avec la commande suivante :

```shell
ng generate @angular/core:inject
```

#### Avant {#before}

```typescript
import {Component, Inject, Optional} from '@angular/core';
import {MyService} from './service';
import {DI_TOKEN} from './token';

@Component({template: ''})
export class MyComp {
  constructor(
    private service: MyService,
    @Inject(DI_TOKEN) @Optional() readonly token: string,
  ) {}
}
```

#### Après {#after}

```typescript
import {Component, inject} from '@angular/core';
import {MyService} from './service';
import {DI_TOKEN} from './token';

@Component({template: ''})
export class MyComp {
  private service = inject(MyService);
  readonly token = inject(DI_TOKEN, {optional: true});
}
```

## Options de la migration {#migration-options}

La migration propose plusieurs options pour personnaliser son résultat.

### `path`

Détermine quel sous-chemin de votre projet doit être migré. Passez `.` ou laissez vide pour
migrer tout le répertoire.

### `migrateAbstractClasses`

Angular ne vérifie pas que les paramètres des classes abstraites sont injectables. La
migration ne peut donc pas les migrer de façon fiable vers [`inject`](/api/core/inject) sans risquer de casser du code : c'est pourquoi elle est
désactivée par défaut pour ces classes. Activez cette option si vous voulez que les classes abstraites soient migrées, mais notez
que vous devrez peut-être **corriger certaines régressions à la main**.

### `backwardsCompatibleConstructors`

Par défaut, la migration essaie de nettoyer le code autant que possible, ce qui inclut la suppression des
paramètres du constructeur, voire du constructeur entier s'il ne contient aucun code.
Dans certains cas, cela peut provoquer des erreurs de compilation quand des classes avec des décorateurs Angular héritent
d'autres classes avec des décorateurs Angular. Si vous activez cette option, la migration génère une
signature de constructeur supplémentaire pour préserver la rétrocompatibilité, au prix de davantage de code.

#### Avant {#backwards-compatible-constructors-before}

```typescript
import {Component} from '@angular/core';
import {MyService} from './service';

@Component({template: ''})
export class MyComp {
  constructor(private service: MyService) {}
}
```

#### Après {#backwards-compatible-constructors-after}

```typescript
import {Component, inject} from '@angular/core';
import {MyService} from './service';

@Component({template: ''})
export class MyComp {
  private service = inject(MyService);

  /** Inserted by Angular inject() migration for backwards compatibility */
  constructor(...args: unknown[]);

  constructor() {}
}
```

### `nonNullableOptional`

Si l'injection échoue pour un paramètre avec le décorateur `@Optional`, Angular renvoie `null`, ce qui
signifie que le type réel de tout paramètre `@Optional` est `| null`. Cependant, comme les décorateurs
ne peuvent pas influencer leurs types, beaucoup de code existant a un type incorrect. Le type est
corrigé dans `inject()`, ce qui peut faire apparaître de nouvelles erreurs de compilation. Si vous activez cette option,
la migration ajoute une assertion non nulle après l'appel à `inject()` pour correspondre à l'ancien type,
au risque de masquer des erreurs de type.

NOTE: les assertions non nulles ne sont pas ajoutées aux paramètres déjà typés comme nullables,
car le code qui en dépend tient probablement déjà compte de leur nullabilité.

#### Avant {#non-nullable-optional-before}

```typescript
import {Component, Inject, Optional} from '@angular/core';
import {TOKEN_ONE, TOKEN_TWO} from './token';

@Component({template: ''})
export class MyComp {
  constructor(
    @Inject(TOKEN_ONE) @Optional() private tokenOne: number,
    @Inject(TOKEN_TWO) @Optional() private tokenTwo: string | null,
  ) {}
}
```

#### Après {#non-nullable-optional-after}

```typescript
import {Component, inject} from '@angular/core';
import {TOKEN_ONE, TOKEN_TWO} from './token';

@Component({template: ''})
export class MyComp {
  // Note the `!` at the end.
  private tokenOne = inject(TOKEN_ONE, {optional: true})!;

  // Does not have `!` at the end, because the type was already nullable.
  private tokenTwo = inject(TOKEN_TWO, {optional: true});
}
```
