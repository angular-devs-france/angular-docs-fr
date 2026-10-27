<!-- source: 244aec81994bfbaf305efb64702396286688f2a9 -->
# Migration de NgClass vers les class bindings

Ce schematic migre les utilisations de la directive NgClass vers des class bindings dans votre application.
Il ne migre que les utilisations considérées comme sûres à migrer.

Lancez le schematic avec la commande suivante :

```bash
ng generate @angular/core:ngclass-to-class
```

#### Avant {#before}

```html
<div [ngClass]="{admin: isAdmin, dense: density === 'high'}"></div>
```

#### Après {#after}

```html
<div [class]="{admin: isAdmin, dense: density === 'high'}"></div>
```

## Options de configuration {#configuration-options}

La migration propose quelques options pour l'ajuster finement à vos besoins.

### `--migrate-space-separated-key`

Par défaut, la migration évite de migrer les utilisations de `NgClass` dont les clés de l'objet littéral contiennent des noms de classes séparés par des espaces.
Quand l'option --migrate-space-separated-key est activée, un binding est créé pour chaque clé individuelle.

```html
<div [ngClass]="{'class1 class2': condition}"></div>
```

devient

```html
<div [class.class1]="condition" [class.class2]="condition"></div>
```
