<!-- source: 9766071b2099bd4e934dba5fbe1a6c92349ac1b8 -->
# Appliquer un debounce aux signals avec `debounced`

IMPORTANT: `debounced` est [expérimental](reference/releases#experimental). Vous pouvez l'essayer dès maintenant, mais il peut changer avant d'être stable.

Utilisez `debounced` pour retarder la réaction à la valeur d'un signal jusqu'à ce qu'elle cesse de changer. Il renvoie une `Resource` dont la valeur reflète la valeur « debouncée » du signal source.

```angular-ts
import {Component, debounced, resource, signal} from '@angular/core';

@Component({
  template: `
    <input (input)="query.set($event.target.value)" />

    @if (results.isLoading()) {
      <p>Searching…</p>
    }
    @for (item of results.value(); track item.id) {
      <li>{{ item.name }}</li>
    }
  `,
})
export class Search {
  query = signal('');

  debouncedQuery = debounced(this.query, 300);

  results = resource({
    params: () => this.debouncedQuery.value(),
    loader: ({params}) => fetchResults(params),
  });
}
```

`debounced` prend le signal source et une durée d'attente en millisecondes. La `value()` de la resource renvoyée contient toujours la dernière valeur stabilisée, et `status()` vous indique si une nouvelle valeur est encore en attente.

## Statut pendant le debounce {#status-during-debounce}

Pendant le décompte du minuteur de debounce, `status()` vaut `'loading'` et `value()` renvoie la valeur résolue précédemment. Quand le minuteur expire, la resource passe à l'état `'resolved'`. Si le signal source lève une erreur, la resource passe immédiatement à l'état `'error'` ; aucun minuteur ne s'exécute.

Consultez [Statut d'une resource](/guide/signals/resource#resource-status) pour la liste complète des statuts et le comportement de `value()` pour chacun d'eux.

## Fonction d'attente personnalisée {#custom-wait-function}

Au lieu d'une durée en millisecondes, vous pouvez passer une fonction qui renvoie une `Promise<void>`. La resource se résout quand la promise se résout. Si le signal source change avant que la promise soit réglée, Angular abandonne la promise précédente et en démarre une nouvelle.

```ts
debouncedQuery = debounced(query, (value, lastSnapshot) => {
  // Retry immediately after an error rather than making the user wait again.
  if (lastSnapshot.status === 'error') return;
  // Short queries get a longer delay—the user is likely still typing.
  const ms = value.length < 3 ? 500 : 200;
  return new Promise<void>((resolve) => setTimeout(resolve, ms));
});
```

Consultez le type `DebounceTimer` dans la référence de l'API pour plus de détails.

## Égalité {#equality}

Par défaut, `debounced` utilise `Object.is` pour comparer les valeurs.

Fournissez une fonction d'égalité personnalisée avec l'option `equal` quand la vérification d'identité par défaut est trop stricte :

```ts
debouncedFilter = debounced(filter, 200, {
  equal: (a, b) => a.category === b.category && a.minPrice === b.minPrice,
});
```

## Contexte d'injection {#injection-context}

`debounced` doit être appelé dans un [contexte d'injection](guide/di/dependency-injection-context). Angular détruit automatiquement la resource « debouncée » et annule tout minuteur en attente quand l'injector est détruit.

Pour utiliser `debounced` en dehors d'un contexte d'injection, passez un `Injector` explicite via les options :

```ts
@Service()
export class SearchService {
  private injector = inject(Injector);

  createDebouncedQuery(query: Signal<string>): Resource<string> {
    return debounced(query, 300, {injector: this.injector});
  }
}
```
