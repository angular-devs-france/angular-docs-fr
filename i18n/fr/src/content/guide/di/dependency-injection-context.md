<!-- source: e9c78053f08d16964c2c1d176e6bfe408ad16dae -->
# Contexte d'injection

Le système d'injection de dépendances (DI) repose sur un contexte d'exécution dans lequel l'injector courant est disponible.

Les injectors ne fonctionnent donc que lorsque vous exécutez du code dans ce contexte.

Vous disposez d'un contexte d'injection dans les situations suivantes :

- Pendant la construction (via le `constructor`) d'une classe instanciée par le système de DI, comme un `@Injectable` ou un `@Component`.
- Dans les initialiseurs de champs de ces classes.
- Dans la fonction de fabrique spécifiée pour `useFactory` d'un `Provider` ou d'un `@Injectable`.
- Dans la fonction `factory` spécifiée pour un `InjectionToken`.
- Dans une stack frame qui s'exécute dans un contexte d'injection.

Savoir quand vous êtes dans un contexte d'injection vous permet d'utiliser la fonction [`inject`](api/core/inject) pour récupérer des dépendances.

NOTE: Pour des exemples simples d'utilisation de `inject()` dans les constructeurs de classe et les initialiseurs de champs, consultez le [guide de présentation](/guide/di#where-can-inject-be-used).

## Stack frame dans le contexte {#stack-frame-in-context}

Certaines API sont conçues pour s'exécuter dans un contexte d'injection. C'est le cas, par exemple, des guards du router. Cela vous permet d'utiliser [`inject`](api/core/inject) dans la fonction de la guard pour accéder à des services.

Voici un exemple pour `CanActivateFn`

```ts {highlight: [3]}
const canActivateTeam: CanActivateFn = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  return inject(PermissionsService).canActivate(inject(UserToken), route.params.id);
};
```

## Exécuter dans un contexte d'injection {#run-within-an-injection-context}

Si vous devez exécuter une fonction dans un contexte d'injection sans vous y trouver déjà, vous pouvez utiliser `runInInjectionContext`.
Cela nécessite d'avoir accès à un injector, comme l'`EnvironmentInjector` :

```ts {highlight: [7], header:"hero.service.ts"}
@Service()
export class HeroService {
  private environmentInjector = inject(EnvironmentInjector);

  someMethod() {
    runInInjectionContext(this.environmentInjector, () => {
      inject(SomeService); // Do what you need with the injected service
    });
  }
}
```

Notez que [`inject`](/api/core/inject) ne renvoie une instance que si l'injector peut résoudre le jeton demandé.

## Vérifier le contexte {#asserts-the-context}

Angular fournit la fonction utilitaire `assertInInjectionContext` pour vérifier que le contexte courant est un contexte d'injection et lever une erreur claire si ce n'est pas le cas. Passez une référence à la fonction appelante pour que le message d'erreur désigne le bon point d'entrée de l'API. Le message obtenu est plus clair et plus exploitable que l'erreur d'injection générique par défaut.

```ts
import {ElementRef, assertInInjectionContext, inject} from '@angular/core';

export function injectNativeElement<T extends Element>(): T {
  assertInInjectionContext(injectNativeElement);
  return inject(ElementRef).nativeElement;
}
```

Vous pouvez ensuite appeler cette fonction utilitaire **depuis un contexte d'injection** (constructeur, initialiseur de champ, fabrique de provider ou code exécuté via `runInInjectionContext`) :

```ts
import {Component, inject} from '@angular/core';
import {injectNativeElement} from './dom-helpers';

@Component({/* … */})
export class PreviewCard {
  readonly hostEl = injectNativeElement<HTMLElement>(); // Field initializer runs in an injection context.

  onAction() {
    const anotherRef = injectNativeElement<HTMLElement>(); // Fails: runs outside an injection context.
  }
}
```

## Utiliser la DI en dehors d'un contexte {#using-di-outside-of-a-context}

Si vous appelez [`inject`](api/core/inject) ou `assertInInjectionContext` en dehors d'un contexte d'injection, Angular lève l'[erreur NG0203](/errors/NG0203).
