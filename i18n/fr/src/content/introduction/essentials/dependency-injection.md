<!-- source: 5142cf688e3dc5f168950d07fd2bb36c0e0c89fb -->
<docs-decorative-header title="Injection de dépendances" imgSrc="adev/src/assets/images/dependency_injection.svg"> <!-- markdownlint-disable-line -->
Réutilisez du code et contrôlez les comportements dans toute votre application et vos tests.
</docs-decorative-header>

Quand vous devez partager de la logique entre composants, Angular s'appuie sur le patron de conception d'[injection de dépendances](guide/di), qui vous permet de créer un « service » pour injecter du code dans les composants tout en le gérant depuis une source de vérité unique.

## Que sont les services ? {#what-are-services}

Les services sont des morceaux de code réutilisables qui peuvent être injectés.

Comme pour la définition d'un composant, un service se compose des éléments suivants :

- Un **décorateur TypeScript** qui déclare la classe comme un service Angular via `@Service` et vous permet de définir un service accessible partout dans votre application.
- Une **classe TypeScript** qui définit le code qui sera accessible quand le service est injecté

Voici un exemple d'un service `Calculator`.

```angular-ts
import {Service} from '@angular/core';

@Service()
export class Calculator {
  add(x: number, y: number) {
    return x + y;
  }
}
```

## Comment utiliser un service {#how-to-use-a-service}

Quand vous voulez utiliser un service dans un composant, vous devez :

1. Importer le service
2. Déclarer un champ de classe dans lequel le service est injecté. Affectez à ce champ le résultat de l'appel à la fonction intégrée [`inject`](/api/core/inject), qui crée le service

Voici à quoi cela peut ressembler dans le composant `Receipt` :

```angular-ts
import {Component, inject} from '@angular/core';
import {Calculator} from './calculator';

@Component({
  selector: 'app-receipt',
  template: `<h1>The total is {{ totalCost }}</h1>`,
})
export class Receipt {
  private calculator = inject(Calculator);
  totalCost = this.calculator.add(50, 25);
}
```

Dans cet exemple, `Calculator` est utilisé en appelant la fonction Angular [`inject`](/api/core/inject) et en lui passant le service.

## Étape suivante {#next-step}

<docs-pill-row>
  <docs-pill title="Après les fondamentaux" href="essentials/next-steps" />
  <docs-pill title="Guide approfondi de l'injection de dépendances" href="guide/di" />
</docs-pill-row>
