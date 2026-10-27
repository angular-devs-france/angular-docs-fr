<!-- source: aefe8611e1a3ad7c7b6e02e895378b9cc63da9cd -->
# Créer un pipe personnalisé

Vous pouvez créer des pipes personnalisés dans Angular pour répondre à vos besoins de transformation de données.

NOTE: Pour en savoir plus, consultez [la création de pipes personnalisés dans le guide approfondi](/guide/templates/pipes#creating-custom-pipes).

Dans cette activité, vous allez créer un pipe personnalisé et l'utiliser dans votre template.

<hr>

Un pipe est une classe TypeScript dotée d'un décorateur `@Pipe`. Voici un exemple :

```ts
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({
  name: 'star',
})
export class StarPipe implements PipeTransform {
  transform(value: string): string {
    return `⭐️ ${value} ⭐️`;
  }
}
```

Le `StarPipe` accepte une chaîne de caractères et renvoie cette chaîne entourée d'étoiles. Notez que :

- le nom défini dans la configuration du décorateur `@Pipe` est celui qui sera utilisé dans le template
- c'est dans la fonction `transform` que vous placez votre logique

À votre tour d'essayer : vous allez créer le `ReversePipe` :

<docs-workflow>

<docs-step title="Créer le `ReversePipe` {#create-the-reversepipe}">

Dans `reverse.pipe.ts`, ajoutez le décorateur `@Pipe` à la classe `ReversePipe` et fournissez la configuration suivante :

```ts
@Pipe({
  name: 'reverse'
})
```

</docs-step>

<docs-step title="Implémenter la fonction `transform` {#implement-the-transform-function}">

La classe `ReversePipe` est maintenant un pipe. Mettez à jour la fonction `transform` pour ajouter la logique d'inversion :

```ts {highlight:[3,4,5,6,7,8,9]}
export class ReversePipe implements PipeTransform {
  transform(value: string): string {
    let reverse = '';

    for (let i = value.length - 1; i >= 0; i--) {
      reverse += value[i];
    }

    return reverse;
  }
}
```

</docs-step>

<docs-step title="Utiliser le `ReversePipe` dans le template {#use-the-reversepipe-in-the-template}">
Maintenant que la logique du pipe est implémentée, la dernière étape consiste à l'utiliser dans le template. Dans `app.ts`, incluez le pipe dans le template et ajoutez-le aux imports du composant :

```angular-ts {highlight:[3,4]}
@Component({
  ...
  template: `Reverse Machine: {{ word | reverse }}`,
  imports: [ReversePipe]
})
```

</docs-step>

</docs-workflow>

Et voilà, vous y êtes arrivé. Félicitations, vous avez terminé cette activité. Vous savez maintenant utiliser des pipes, et même implémenter vos propres pipes personnalisés.
