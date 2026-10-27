<!-- source: 33334d0c71ddfa638d88bd91133822d9e0633850 -->
# Pipes

Les pipes sont des fonctions qui servent à transformer des données dans les templates. En général, les pipes sont des fonctions « pures », sans effets de bord. Angular propose de nombreux pipes intégrés utiles que vous pouvez importer et utiliser dans vos composants. Vous pouvez aussi créer un pipe personnalisé.

NOTE: Pour en savoir plus, consultez [les pipes dans le guide approfondi](/guide/templates/pipes).

Dans cette activité, vous allez importer un pipe et l'utiliser dans le template.

<hr>

Pour utiliser un pipe dans un template, incluez-le dans une expression interpolée. Regardez cet exemple :

```angular-ts {highlight:[1,5,6]}
import {UpperCasePipe} from '@angular/common';

@Component({
  ...
  template: `{{ loudMessage | uppercase }}`,
  imports: [UpperCasePipe],
})
export class App {
  loudMessage = 'we think you are doing great!'
}
```

À votre tour d'essayer :

<docs-workflow>

<docs-step title="Importer le pipe `LowerCase` {#import-the-lowercase-pipe}">
Commencez par mettre à jour `app.ts` en ajoutant, au niveau du fichier, l'import de `LowerCasePipe` depuis `@angular/common`.

```ts
import {LowerCasePipe} from '@angular/common';
```

</docs-step>

<docs-step title="Ajouter le pipe aux imports du template {#add-the-pipe-to-the-template-imports}">
Ensuite, mettez à jour les `imports` du décorateur `@Component()` pour y inclure une référence à `LowerCasePipe`.

```ts {highlight:[3]}
@Component({
  ...
  imports: [LowerCasePipe]
})
```

</docs-step>

<docs-step title="Ajouter le pipe au template {#add-the-pipe-to-the-template}">
Enfin, dans `app.ts`, mettez à jour le template pour y inclure le pipe `lowercase` :

```angular-html
template: `{{ username | lowercase }}`
```

</docs-step>

</docs-workflow>

Les pipes peuvent aussi accepter des paramètres, qui permettent de configurer leur sortie. Vous en saurez plus dans la prochaine activité.

P.-S. : vous vous en sortez très bien ⭐️
