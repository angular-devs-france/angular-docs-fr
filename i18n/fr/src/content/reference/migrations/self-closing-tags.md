<!-- source: a6fdf8e0fa0363bd13c80a6dccf5056140aada73 -->
# Migration vers les balises auto-fermantes

Les balises auto-fermantes sont prises en charge dans les templates Angular depuis la [v16](https://blog.angular.dev/angular-v16-is-here-4d7a28ec680d#7065).

Ce schematic migre les templates de votre application pour utiliser des balises auto-fermantes.

Lancez le schematic avec la commande suivante :

```shell
ng generate @angular/core:self-closing-tag
```

#### Avant {#before}

```angular-html
<hello-world></hello-world>
```

#### Après {#after}

```angular-html
<hello-world />
```
