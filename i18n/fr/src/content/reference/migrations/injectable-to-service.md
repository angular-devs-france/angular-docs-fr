<!-- source: 12a6e1b6c9bb6cfbfe93f7bdf678a2ffb9e7ada0 -->
# Migration de `@Injectable` vers `@Service`

Ce schematic convertit les classes `@Injectable` éligibles vers le décorateur [`@Service`](/api/core/Service), un raccourci ergonomique pour `@Injectable({providedIn: 'root'})`.
Il est disponible depuis la v22.1 et ne migre que les classes considérées comme sûres à migrer.

Lancez le schematic avec la commande suivante :

```bash
ng generate @angular/core:service
```

Une classe déclarée avec `providedIn: 'root'` devient un `@Service` sans options.

#### Avant {#before}

```ts
import {Injectable} from '@angular/core';

@Injectable({providedIn: 'root'})
export class MyService {}
```

#### Après {#after}

```ts
import {Service} from '@angular/core';

@Service()
export class MyService {}
```

Une classe déclarée sans options n'est pas fournie automatiquement : la migration conserve donc ce comportement avec `autoProvided: false`.
C'est toujours à vous de l'ajouter à un tableau `providers`, comme auparavant.

#### Avant {#no-options-before}

```ts
import {Injectable} from '@angular/core';

@Injectable()
export class MyService {}
```

#### Après {#no-options-after}

```ts
import {Service} from '@angular/core';

@Service({autoProvided: false})
export class MyService {}
```

Pour plus de détails, consultez [Désactiver la fourniture automatique](guide/di/creating-and-using-services#opting-out-of-automatic-provisioning).

## Options de configuration {#configuration-options}

### `--path`

Par défaut, la migration met à jour tout votre workspace Angular CLI, fichiers de test compris.
Vous pouvez limiter la migration à un sous-répertoire spécifique avec cette option.

```bash
ng generate @angular/core:service --path src/app/sub-component
```

## Limitations {#limitations}

Pour éviter d'introduire des régressions dans votre application, le schematic ignore une classe quand :

- La classe, ou l'ancêtre le plus proche dans votre propre code source qui définit un constructeur, injecte des dépendances via son constructeur.
- Le décorateur `@Injectable` reçoit une option autre que `providedIn`.
- L'option `providedIn` a une valeur autre que `'root'`.

Quand tous les `@Injectable` d'un fichier sont migrés, la migration supprime l'import de `Injectable`.
Un fichier qui contient à la fois des classes migrées et des classes ignorées conserve les deux imports.

Si la migration indique qu'elle n'a trouvé aucun fichier à migrer, elle n'a modifié aucun fichier.
Soit elle n'a trouvé aucune classe `@Injectable` qu'elle pouvait convertir, soit le code est déjà migré.
L'injection par constructeur est la raison la plus fréquente d'une classe ignorée : vous pouvez d'abord lancer la [migration vers la fonction `inject`](reference/migrations/inject-function), puis relancer celle-ci.
