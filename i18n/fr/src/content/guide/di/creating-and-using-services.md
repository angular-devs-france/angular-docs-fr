<!-- source: 10034ca916653c9aec42067db3be6cbf20fd4422 -->
# Créer et utiliser des services

Les services sont des morceaux de code réutilisables que vous pouvez partager dans votre application Angular. Vous les utilisez généralement pour gérer la récupération de données, la logique métier ou d'autres fonctionnalités auxquelles plusieurs composants doivent accéder.

## Créer un service {#creating-a-service}

Vous pouvez créer un service avec l'[Angular CLI](tools/cli) à l'aide de la commande suivante :

```bash
ng generate service CUSTOM_NAME
```

Cette commande crée un fichier dédié `CUSTOM_NAME.ts` dans votre répertoire `src`.

Vous pouvez aussi créer un service manuellement en ajoutant le décorateur `@Service()` à une classe TypeScript. Cela indique à Angular que vous pouvez utiliser la classe comme une dépendance injectable.

L'exemple suivant définit un service qui permet aux utilisateurs d'ajouter et de récupérer des données :

```ts {header: "src/app/basic-data-store.ts"}
import {Service} from '@angular/core';

@Service()
export class BasicDataStore {
  private data: string[] = [];

  addData(item: string): void {
    this.data.push(item);
  }

  getData(): string[] {
    return [...this.data];
  }
}
```

## Comment les services deviennent disponibles {#how-services-become-available}

Par défaut, les services sont fournis au niveau racine. Quand un service est fourni globalement, Angular garantit trois avantages principaux :

- **Instance singleton :** une seule instance partagée est créée pour toute l'application.
- **Disponibilité globale :** le service est automatiquement accessible partout, sans enregistrement manuel de provider.
- **Tree-shaking :** le service est exclu du bundle de production final si votre code ne l'utilise jamais explicitement.

### Utiliser le décorateur `@Service` ou `@Injectable` {#using-the-service-vs-injectable-decorator}

Le décorateur `@Service` est un raccourci moderne et ergonomique pour la syntaxe traditionnelle `@Injectable({ providedIn: 'root' })`.

Utilisez ce tableau récapitulatif pour choisir le décorateur adapté à votre situation :

| Fonctionnalité / besoin                       | `@Service` | `@Injectable`                           |
| --------------------------------------------- | ---------- | --------------------------------------- |
| **Prise en charge de la fonction `inject()`** | Oui        | Oui                                     |
| **DI par le constructeur**                    | ❌ Non     | Oui                                     |
| **Provider singleton racine implicite**       | Oui        | ❌ Non (nécessite `{providedIn: 'root'}`) |
| **Clés de provider avancées (`useClass`, etc.)** | ❌ Non  | Oui                                     |
| **Fabriques d'initialisation personnalisées** | Oui        | Oui                                     |
| **Portées autres que root (`platform`, etc.)** | ❌ Non    | Oui                                     |

### Remplacer l'implémentation par une fabrique {#replacing-the-implementation-with-a-factory}

Si vous devez contrôler la façon dont le singleton est créé, par exemple pour utiliser une autre implémentation selon l'environnement, passez une fonction `factory`.

La fabrique s'exécute dans un [contexte d'injection](guide/di/dependency-injection-context) : vous pouvez donc y utiliser [`inject()`](api/core/inject) pour lire d'autres dépendances.

Le service `Analytics` suivant ne fait rien en local, pour que les événements ne polluent pas la console pendant le développement. En production, la fabrique lit un jeton `ANALYTICS_ENABLED` et renvoie une sous-classe `GoogleAnalytics` qui transmet les événements au véritable outil de suivi :

```ts {header: "src/app/analytics.ts"}
import {inject, InjectionToken, Service} from '@angular/core';
import {ANALYTICS_ENABLED} from './token';

@Service({
  factory: () => (inject(ANALYTICS_ENABLED) ? new GoogleAnalytics() : new Analytics()),
})
export class Analytics {
  track(event: string, payload?: Record<string, unknown>) {
    // No-op by default.
  }
}

class GoogleAnalytics extends Analytics {
  override track(event: string, payload?: Record<string, unknown>) {
    // Dispatches an analytics event to Google Analytics
  }
}
```

NOTE: L'option `factory` remplace les options `useClass`, `useValue`, `useExisting` et `useFactory` de `@Injectable`. Si vous avez besoin de l'une d'elles, continuez à utiliser `@Injectable`.

### Désactiver la fourniture automatique {#opting-out-of-automatic-provisioning}

Par défaut, `@Service` fournit la classe dans l'injector racine. Si vous voulez la fournir manuellement, par exemple pour limiter sa portée à une route ou à un composant précis, définissez `autoProvided: false` :

```ts {header: "src/app/analytics-logger.ts"}
import {Service} from '@angular/core';

@Service({autoProvided: false})
export class AnalyticsLogger {
  trackEvent(name: string) {
    console.log('event:', name);
  }
}
```

Il vous appartient alors d'ajouter le service à un tableau `providers`, comme avec un simple `@Injectable()` :

### Quand utiliser `@Service` ou `@Injectable` {#when-to-use-service-vs-injectable}

Optez pour `@Service` quand vous créez une nouvelle classe singleton qui utilise `inject()` pour ses dépendances. Continuez à utiliser `@Injectable` quand vous avez besoin de l'un des éléments suivants :

- **L'injection de dépendances par le constructeur.** `@Service` ne prend en charge que la fonction [`inject()`](api/core/inject).
- **Une configuration avancée du provider**, comme `useClass`, `useValue`, `useExisting` ou `useFactory`. `@Service` expose à la place une seule option `factory`.
- **Des portées autres que root**, comme `providedIn: 'platform'`.

## Injecter un service {#injecting-a-service}

Une fois que vous avez créé un service avec `providedIn: 'root'`, vous pouvez l'injecter n'importe où dans votre application avec la fonction `inject()` de `@angular/core`.

### Injecter dans un composant {#injecting-into-a-component}

```angular-ts
import {Component, inject} from '@angular/core';
import {BasicDataStore} from './basic-data-store';

@Component({
  selector: 'app-example',
  template: `
    <div>
      <p>{{ dataStore.getData() }}</p>
      <button (click)="dataStore.addData('More data')">Add more data</button>
    </div>
  `,
})
export class Example {
  dataStore = inject(BasicDataStore);
}
```

### Injecter dans un autre service {#injecting-into-another-service}

```ts
import {inject, Service} from '@angular/core';
import {AdvancedDataStore} from './advanced-data-store';

@Service()
export class BasicDataStore {
  private advancedDataStore = inject(AdvancedDataStore);
  private data: string[] = [];

  addData(item: string): void {
    this.data.push(item);
  }

  getData(): string[] {
    return [...this.data, ...this.advancedDataStore.getData()];
  }
}
```

## Prochaines étapes {#next-steps}

Si `providedIn: 'root'` couvre la plupart des cas d'usage, Angular propose aussi d'autres façons de configurer les services pour des scénarios plus spécialisés :

- **Instances propres à un composant** - quand des composants ont besoin de leurs propres instances isolées d'un service
- **Configuration manuelle** - pour les services qui nécessitent une configuration à l'exécution
- **Providers de fabrique** - pour créer dynamiquement des services selon des conditions d'exécution
- **Providers de valeur** - pour fournir des objets de configuration ou des constantes

Vous pouvez en apprendre davantage sur ces patterns avancés dans le guide suivant : [définir des providers de dépendances](/guide/di/defining-dependency-providers).
