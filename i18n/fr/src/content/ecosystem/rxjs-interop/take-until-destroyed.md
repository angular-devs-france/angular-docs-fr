<!-- source: 00179aa5f5dc1f80e1440d35a05193b0505b07e8 -->
# Se désabonner avec `takeUntilDestroyed`

TIP: Ce guide suppose que vous connaissez le [cycle de vie des composants et des directives](guide/components/lifecycle).

L'opérateur `takeUntilDestroyed`, issu de `@angular/core/rxjs-interop`, offre un moyen concis et fiable de se désabonner automatiquement d'un observable lorsqu'un composant ou une directive est détruit. Cela évite les fuites de mémoire courantes liées aux abonnements RxJS. Il fonctionne de manière similaire à l'opérateur RxJS [`takeUntil`](https://rxjs.dev/api/operators/takeUntil), mais sans nécessiter de Subject séparé.

```typescript
import {Component, inject} from '@angular/core';
import {takeUntilDestroyed} from '@angular/core/rxjs-interop';
import {Observable} from 'rxjs';
import {NotificationDispatcher, CustomPopupShower} from './some-shared-project-code';

@Component(/* ... */)
export class UserProfile {
  private dispatcher = inject(NotificationDispatcher);
  private popup = inject(CustomPopupShower);

  constructor() {
    // This subscription the 'notifications' Observable is automatically
    // unsubscribed when the 'UserProfile' component is destroyed.
    const messages: Observable<string> = this.dispatcher.notifications;
    messages.pipe(takeUntilDestroyed()).subscribe((message) => {
      this.popup.show(message);
    });
  }
}
```

L'opérateur `takeUntilDestroyed` accepte un unique argument optionnel [`DestroyRef`](/api/core/DestroyRef). L'opérateur utilise `DestroyRef` pour savoir quand le composant ou la directive a été détruit. Vous pouvez omettre cet argument lorsque vous appelez `takeUntilDestroyed` dans un [contexte d'injection](/guide/di/dependency-injection-context), généralement le constructeur d'un composant ou d'une directive. Fournissez toujours un `DestroyRef` si votre code est susceptible d'appeler `takeUntilDestroyed` en dehors d'un contexte d'injection.

```typescript
@Component(/* ... */)
export class UserProfile {
  private dispatcher = inject(NotificationDispatcher);
  private popup = inject(CustomPopupShower);
  private destroyRef = inject(DestroyRef);

  startListeningToNotifications() {
    // Always pass a `DestroyRef` if you call `takeUntilDestroyed` outside
    // of an injection context.
    const messages: Observable<string> = this.dispatcher.notifications;
    messages.pipe(takeUntilDestroyed(this.destroyRef)).subscribe((message) => {
      this.popup.show(message);
    });
  }
}
```
