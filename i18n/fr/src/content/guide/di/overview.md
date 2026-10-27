<!-- source: 7142858bb1f940ccc72195b80ab0be17da46e454 -->
<docs-decorative-header title="L'injection de dépendances dans Angular" imgSrc="adev/src/assets/images/dependency_injection.svg"> <!-- markdownlint-disable-line -->

L'injection de dépendances (DI) est un design pattern qui vous permet d'organiser et de partager du code dans votre application en fournissant des dépendances à une classe au lieu de les créer à l'intérieur de celle-ci.
</docs-decorative-header>

TIP: Consultez les [Fondamentaux](essentials/dependency-injection) d'Angular avant de vous plonger dans ce guide approfondi.

À mesure qu'une application grandit, les développeurs ont souvent besoin de réutiliser et de partager des fonctionnalités entre différentes parties du code. L'[injection de dépendances (DI)](https://en.wikipedia.org/wiki/Dependency_injection) vous y aide en vous permettant de fournir des dépendances à une classe au lieu de les créer directement à l'intérieur de celle-ci. Les différentes parties de l'application deviennent ainsi plus réutilisables et plus faciles à gérer.

L'injection de dépendances est un pattern populaire, car elle permet aux développeurs de relever des défis courants, comme :

- **Une meilleure maintenabilité du code** : l'injection de dépendances favorise une séparation claire des responsabilités, ce qui facilite le refactoring du code et réduit la duplication.
- **La scalabilité** : vous pouvez réutiliser des fonctionnalités modulaires dans différentes parties d'une application, ce qui facilite sa montée en charge.
- **De meilleurs tests** : la DI permet aux tests unitaires d'utiliser des [doublures de test](https://en.wikipedia.org/wiki/Test_double) à la place des implémentations réelles si nécessaire.

## Comment fonctionne l'injection de dépendances dans Angular ? {#how-does-dependency-injection-work-in-angular}

Une dépendance est n'importe quel objet, valeur, fonction ou service dont une classe a besoin pour fonctionner, mais qu'elle ne crée pas elle-même. Vous la fournissez plutôt depuis l'extérieur, ce qui crée une relation claire entre les différentes parties de l'application.

Vous interagissez avec un système d'injection de dépendances de deux façons principales :

- Vous pouvez _fournir_, c'est-à-dire mettre à disposition, des valeurs.
- Vous pouvez _injecter_, c'est-à-dire demander, ces valeurs en tant que dépendances.

Dans ce contexte, les « valeurs » peuvent désigner n'importe quelle valeur JavaScript, y compris des objets, des fonctions ou des instances de classe. Les types de dépendances injectées les plus courants sont :

- **Les valeurs de configuration** : constantes propres à l'environnement, URL d'API, feature flags, etc.
- **Les fabriques** : fonctions qui créent des objets ou des valeurs selon des conditions d'exécution
- **Les services** : classes qui fournissent des fonctionnalités communes, de la logique métier ou de l'état

Les composants et les directives Angular participent automatiquement à la DI : vous pouvez donc y injecter des dépendances et les rendre disponibles pour l'injection.

## Que sont les services ? {#what-are-services}

Un _service_ Angular est une classe TypeScript décorée avec `@Service`, ce qui vous permet d'injecter une instance de la classe en tant que dépendance. Les services sont le moyen le plus courant de partager des données et des fonctionnalités dans une application.

Les types de services les plus courants sont :

- **Les clients de données :** ils masquent les détails des requêtes envoyées à un serveur pour récupérer et modifier des données
- **La gestion d'état :** ils définissent un état partagé entre plusieurs composants ou pages
- **L'authentification et l'autorisation :** ils gèrent l'authentification des utilisateurs, le stockage des jetons et le contrôle d'accès
- **La journalisation et la gestion des erreurs :** ils établissent une API commune pour journaliser ou communiquer les états d'erreur à l'utilisateur
- **La gestion et la diffusion d'événements :** ils gèrent les événements ou les notifications qui ne sont pas associés à un composant spécifique, ou diffusent des événements et des notifications aux composants, selon le [pattern observateur](https://en.wikipedia.org/wiki/Observer_pattern)
- **Les fonctions utilitaires :** ils proposent des fonctions utilitaires réutilisables, comme le formatage de données, la validation ou des calculs

L'exemple suivant déclare un service nommé `AnalyticsLogger` :

```ts
import {Service} from '@angular/core';

@Service()
export class AnalyticsLogger {
  trackEvent(category: string, value: string) {
    console.log('Analytics event logged:', {
      category,
      value,
      timestamp: new Date().toISOString(),
    });
  }
}
```

NOTE: Le décorateur `@Service` rend ce service disponible dans toute votre application en tant que singleton. C'est l'approche recommandée pour la plupart des services.

HELPFUL: Le décorateur [`@Service`](guide/di/creating-and-using-services#using-the-service-vs-injectable-decorator) est un raccourci ergonomique pour `@Injectable({providedIn: 'root'})`.

## Injecter des dépendances avec `inject()` {#injecting-dependencies-with-inject}

Vous pouvez injecter des dépendances avec la fonction `inject()` d'Angular.

Voici l'exemple d'une barre de navigation qui injecte `AnalyticsLogger` et le service `Router` d'Angular pour permettre aux utilisateurs de naviguer vers une autre page tout en suivant l'événement.

```angular-ts
import {Component, inject} from '@angular/core';
import {Router} from '@angular/router';
import {AnalyticsLogger} from './analytics-logger';

@Component({
  selector: 'app-navbar',
  template: `<a href="#" (click)="navigateToDetail($event)">Detail Page</a>`,
})
export class Navbar {
  private router = inject(Router);
  private analytics = inject(AnalyticsLogger);

  navigateToDetail(event: Event) {
    event.preventDefault();
    this.analytics.trackEvent('navigation', '/details');
    this.router.navigate(['/details']);
  }
}
```

### Où peut-on utiliser `inject()` ? {#where-can-inject-be-used}

Vous pouvez injecter des dépendances pendant la construction d'un composant, d'une directive ou d'un service. L'appel à [`inject`](/api/core/inject) peut se trouver soit dans le `constructor`, soit dans l'initialiseur d'un champ. Voici quelques exemples courants :

```ts
@Component(/* ... */)
export class MyComponent {
  // ✅ In class field initializer
  private service = inject(MyService);

  // ✅ In constructor body
  private anotherService: MyService;

  constructor() {
    this.anotherService = inject(MyService);
  }
}
```

```ts
@Directive({...})
export class MyDirective {
  // ✅ In class field initializer
  private element = inject(ElementRef);
}
```

```ts
import {Service, inject} from '@angular/core';
import {HttpClient} from '@angular/common/http';

@Service()
export class MyService {
  // ✅ In a service
  private http = inject(HttpClient);
}
```

```ts
export const authGuard = () => {
  // ✅ In a route guard
  const auth = inject(AuthService);
  return auth.isAuthenticated();
};
```

Angular utilise le terme « contexte d'injection » pour décrire tout endroit de votre code où vous pouvez appeler [`inject`](/api/core/inject). Si la construction des composants, directives et services est le cas le plus courant, consultez les [contextes d'injection](/guide/di/dependency-injection-context) pour plus de détails.

Pour plus d'informations, consultez la [documentation de l'API inject](api/core/inject#usage-notes).

## Prochaines étapes {#next-steps}

Maintenant que vous comprenez les bases de l'injection de dépendances dans Angular, vous êtes prêt à apprendre à créer vos propres services.

Le guide suivant, [Créer et utiliser des services](guide/di/creating-and-using-services), vous montrera :

- Comment créer un service avec l'Angular CLI ou manuellement
- Comment fonctionne le pattern `providedIn: 'root'`
- Comment injecter des services dans des composants et d'autres services

Cela couvre le cas d'usage le plus courant des services dans les applications Angular.
