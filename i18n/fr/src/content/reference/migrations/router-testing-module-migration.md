<!-- source: 96e35e1c02afbdb7d1bf02c93a2c6e8ae02c739c -->
# Migration de RouterTestingModule

Ce schematic migre les utilisations de `RouterTestingModule` dans les tests vers `RouterModule`.

Quand un test importe `SpyLocation` depuis `@angular/common/testing` et utilise la propriété `urlChanges`, le schematic ajoute aussi `provideLocationMocks()` pour préserver le comportement d'origine.

Lancez le schematic avec :

```shell
ng generate @angular/core:router-testing-module-migration
```

## Options {#options}

| Option | Détails                                                                                                                       |
| :----- | :---------------------------------------------------------------------------------------------------------------------------- |
| `path` | Le chemin (relatif à la racine du projet) à migrer. Vaut `./` par défaut. Utilisez-le pour migrer progressivement une partie de votre projet. |

## Exemples {#examples}

### Préserver les options du router {#preserve-router-options}

Avant :

```ts
import {RouterTestingModule} from '@angular/router/testing';
import {TestBed} from '@angular/core/testing';
import {SpyLocation} from '@angular/common/testing';

describe('test', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [RouterTestingModule.withRoutes(routes, {initialNavigation: 'enabledBlocking'})],
    });
  });
});
```

Après :

```ts
import {RouterModule} from '@angular/router';
import {TestBed} from '@angular/core/testing';
import {SpyLocation} from '@angular/common/testing';

describe('test', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [RouterModule.forRoot(routes, {initialNavigation: 'enabledBlocking'})],
    });
  });
});
```

### Ajouter provideLocationMocks quand `SpyLocation` est importé et que `urlChanges` est utilisé {#add-providelocationmocks-when-spylocation-is-imported-and-urlchanges-is-used}

Avant :

```ts
import {RouterTestingModule} from '@angular/router/testing';
import {TestBed} from '@angular/core/testing';
import {SpyLocation} from '@angular/common/testing';

describe('test', () => {
  let spy: SpyLocation;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [RouterTestingModule],
    });
    spy = TestBed.inject(SpyLocation);
  });

  it('Awesome test', () => {
    expect(spy.urlChanges).toBeDefined();
  });
});
```

Après :

```ts
import {RouterModule} from '@angular/router';
import {TestBed} from '@angular/core/testing';
import {SpyLocation, provideLocationMocks} from '@angular/common/testing';

describe('test', () => {
  let spy: SpyLocation;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [RouterModule],
      providers: [provideLocationMocks()],
    });
    spy = TestBed.inject(SpyLocation);
  });

  it('Awesome test', () => {
    expect(spy.urlChanges).toBeDefined();
  });
});
```
