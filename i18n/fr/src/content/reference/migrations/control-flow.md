<!-- source: c55e0ca974911f1dc9b3ec15c47e924068051eaf -->
# Migration vers la syntaxe du control flow

La [syntaxe du control flow](guide/templates/control-flow) est disponible depuis Angular v17. La nouvelle syntaxe est intégrée au template : vous n'avez donc plus besoin d'importer `CommonModule`.

Ce schematic migre tout le code existant de votre application vers la nouvelle syntaxe du control flow.

Lancez le schematic avec la commande suivante :

```shell
ng generate @angular/core:control-flow
```

## Options de configuration {#configuration-options}

La migration propose quelques options pour l'ajuster finement à vos besoins.

### `--path`

Par défaut, la migration met à jour tout votre workspace Angular CLI.
Vous pouvez limiter la migration à un sous-répertoire spécifique avec cette option.

### `--format`

Par défaut, la migration reformate les templates qu'elle modifie.
Désactivez cette option pour conserver la mise en forme existante de vos templates.

## Changements incompatibles {#breaking-changes}

### Réutilisation des vues avec `@for` {#for-view-reuse}

Si une propriété utilisée dans l'expression `track` d'un bloc `@for` change mais que la référence de l'objet reste la même (modification en place), Angular met à jour les bindings de la vue (y compris les inputs des composants) au lieu de détruire et recréer l'élément.

Cela diffère de `*ngFor`, qui remonterait l'élément (le détruirait puis le recréerait) dans une situation similaire si la fonction `trackBy` renvoyait une valeur différente.
