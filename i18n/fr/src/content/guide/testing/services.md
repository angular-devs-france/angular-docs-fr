<!-- source: 7af23d220098f7db2df015a5be41d7bf03251b94 -->
# Tester les services

Les services contiennent généralement la logique métier de votre application, sur laquelle s'appuient les composants. Tester les services permet de vérifier que cette logique fonctionne correctement de manière isolée, indépendamment de tout composant ou template.

Ce guide utilise [Vitest](https://vitest.dev/), que les projets Angular CLI incluent par défaut. Pour en savoir plus sur la configuration des tests, consultez le [guide de présentation des tests](guide/testing#set-up-for-testing).

## Tester un service {#testing-a-service}

Prenons un service `Calculator` qui effectue des opérations arithmétiques de base :

```ts { header: 'calculator.ts' }
import {Service} from '@angular/core';

@Service()
export class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }

  subtract(a: number, b: number): number {
    return a - b;
  }
}
```

Pour tester ce service, configurez un `TestBed`, l'utilitaire de test d'Angular qui crée un environnement de test isolé pour chaque test. Il met en place l'injection de dépendances et vous permet de récupérer des instances de services, en simulant la manière dont Angular assemble les éléments dans une vraie application.

```ts { header: 'calculator.spec.ts' }
import {TestBed} from '@angular/core/testing';
import {beforeEach, describe, expect, it} from 'vitest';
import {Calculator} from './calculator';

describe('Calculator', () => {
  let service: Calculator;

  beforeEach(() => {
    // Injects the Calculator service which is available to Angular
    // because the service uses `@Service`
    service = TestBed.inject(Calculator);
  });

  it('adds two numbers', () => {
    expect(service.add(1, 2)).toBe(3);
  });

  it('subtracts two numbers', () => {
    expect(service.subtract(5, 3)).toBe(2);
  });
});
```

Dans l'exemple ci-dessus, le bloc `beforeEach` injecte une nouvelle instance du service avant chaque test. Cela garantit que chaque test s'exécute de manière isolée, sans état hérité des tests précédents.

## Tester des services avec des dépendances {#testing-services-with-dependencies}

La plupart des services dépendent d'autres services pour fonctionner correctement. Par défaut, `TestBed` fournit les vraies implémentations de ces dépendances, ce qui signifie que vos tests exercent les chemins de code réellement utilisés par votre application. Parfois, cependant, une dépendance peut être complexe, lente ou imprévisible. Dans ces cas-là, vous pouvez la remplacer par un substitut contrôlé.

Prenons un service `OrderTotal` qui s'appuie sur un `TaxCalculator` pour calculer le prix final d'une commande :

```ts { header: 'tax-calculator.ts' }
import {Service} from '@angular/core';

@Service()
export class TaxCalculator {
  calculate(subtotal: number): number {
    return subtotal * 0.05;
  }
}
```

```ts { header: 'order-total.ts' }
import {inject, Service} from '@angular/core';
import {TaxCalculator} from './tax-calculator';

@Service()
export class OrderTotal {
  private taxCalculator = inject(TaxCalculator);

  total(subtotal: number): number {
    return subtotal + this.taxCalculator.calculate(subtotal);
  }
}
```

Dans cet exemple, `OrderTotal` utilise `inject()` pour demander `TaxCalculator` au système d'injection de dépendances d'Angular. Par défaut, `TestBed` fournit le vrai `TaxCalculator`, ce qui convient parfaitement à des calculs simples comme celui-ci. En revanche, si `TaxCalculator` impliquait une logique complexe, des requêtes réseau ou des résultats imprévisibles, vous pourriez vouloir le remplacer par un substitut contrôlé.

### Remplacer une dépendance par un stub {#replacing-a-dependency-with-a-stub}

Un stub permet de remplacer une dépendance ou une méthode par une autre qui renvoie des valeurs prévisibles, ce qui peut rendre les résultats des tests plus faciles à vérifier.

Pour tester `OrderTotal` sans dépendre du vrai `TaxCalculator`, vous pouvez fournir un stub dans la configuration du `TestBed`.

```ts { header: 'order-total.spec.ts' }
import {TestBed} from '@angular/core/testing';
import {beforeEach, describe, expect, it, vi, type Mocked} from 'vitest';
import {OrderTotal} from './order-total';
import {TaxCalculator} from './tax-calculator';

// Vitest's `Mocked` utility type ensures the stub is type-safe,
// while `vi.fn()` creates a mock function for each method
const taxCalculatorStub: Mocked<TaxCalculator> = {
  calculate: vi.fn(),
};

describe('OrderTotal', () => {
  let service: OrderTotal;

  beforeEach(() => {
    // `mockReturnValue` sets a controlled return value for the stub
    taxCalculatorStub.calculate.mockReturnValue(5);

    TestBed.configureTestingModule({
      // The `providers` array accepts a provider object where `provide`
      // specifies the dependency to replace and `useValue` defines the stub
      providers: [{provide: TaxCalculator, useValue: taxCalculatorStub}],
    });
    service = TestBed.inject(OrderTotal);
  });

  it('adds tax to the subtotal', () => {
    expect(service.total(100)).toBe(105);
  });
});
```

Avec ce stub, chaque fois que `OrderTotal` demande `TaxCalculator`, le `TestBed` sait qu'il doit utiliser `taxCalculatorStub` à la place. Comme le stub renvoie toujours 5, le test vérifie que `OrderTotal` ajoute correctement le montant de la taxe au sous-total, que le taux de taxe change ou non dans `TaxCalculator`.

### Vérifier les interactions avec des spies {#verifying-interactions-with-spies}

Un stub contrôle ce que renvoie une dépendance, mais vous avez parfois aussi besoin de vérifier qu'un service a appelé sa dépendance avec les bons arguments. Vous pouvez le faire avec des spies, qui enregistrent la manière dont une fonction est appelée. Avec Vitest, cette fonctionnalité est intégrée à `vi.fn()` et vous permet de faire des assertions sur les interactions entre services.

```ts { header: 'order-total.spec.ts' }
import {TestBed} from '@angular/core/testing';
import {afterEach, beforeEach, describe, expect, it, vi, type Mocked} from 'vitest';
import {OrderTotal} from './order-total';
import {TaxCalculator} from './tax-calculator';

const taxCalculatorStub: Mocked<TaxCalculator> = {
  calculate: vi.fn(),
};

describe('OrderTotal', () => {
  let service: OrderTotal;

  beforeEach(() => {
    taxCalculatorStub.calculate.mockReturnValue(5);

    TestBed.configureTestingModule({
      providers: [{provide: TaxCalculator, useValue: taxCalculatorStub}],
    });
    service = TestBed.inject(OrderTotal);
  });

  afterEach(() => {
    taxCalculatorStub.calculate.mockClear();
  });

  it('adds tax to the subtotal', () => {
    expect(service.total(100)).toBe(105);
  });

  // Verify the interaction with a spy
  it('calls the tax calculator', () => {
    service.total(100);
    expect(taxCalculatorStub.calculate).toHaveBeenCalledExactlyOnceWith(100);
  });
});
```

Le nouveau test vérifie que `OrderTotal` a appelé `TaxCalculator.calculate` lors du calcul du total. C'est utile pour vérifier que l'interaction entre les services s'est déroulée correctement.

## Tester les services HTTP {#testing-http-services}

De nombreux services utilisent le `HttpClient` d'Angular pour récupérer des données depuis un serveur. Angular fournit des utilitaires de test dédiés à `HttpClient`, qui vous permettent de contrôler les réponses HTTP sans effectuer de vraies requêtes réseau.

Pour en savoir plus sur le test des services qui utilisent `HttpClient`, consultez le [guide sur les tests HTTP](guide/http/testing).
