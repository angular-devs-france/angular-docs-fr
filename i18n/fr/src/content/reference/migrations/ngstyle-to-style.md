<!-- source: edef2c71812207a731c7751dd0983b7f00377244 -->
# Migration de NgStyle vers les style bindings

Ce schematic migre les utilisations de la directive NgStyle vers des style bindings dans votre application.
Il ne migre que les utilisations considérées comme sûres à migrer.

Lancez le schematic avec la commande suivante :

```bash
ng generate @angular/core:ngstyle-to-style
```

Un objet avec une seule clé devient un binding vers cette unique propriété de style.

#### Avant {#before}

```html
<div [ngStyle]="{'background-color': 'red'}"></div>
```

#### Après {#after}

```html
<div [style.background-color]="'red'"></div>
```

Un objet avec plusieurs clés devient un unique binding `[style]`.

#### Avant {#multiple-keys-before}

```html
<div [ngStyle]="{'color': 'blue', 'font-weight': 'bold'}"></div>
```

#### Après {#multiple-keys-after}

```html
<div [style]="{'color': 'blue', 'font-weight': 'bold'}"></div>
```

## Options de configuration {#configuration-options}

La migration propose quelques options pour l'ajuster finement à vos besoins.

### `--best-effort-mode`

Par défaut, la migration évite de migrer les utilisations de `NgStyle` liées à des références d'objets.
Quand l'option `--best-effort-mode` est activée, les instances de `ngStyle` liées à des références d'objets sont aussi migrées.
Cette migration peut ne pas être sûre, par exemple si l'objet lié est muté.

```html
<div [ngStyle]="styleObject"></div>
```

devient

```html
<div [style]="styleObject"></div>
```
