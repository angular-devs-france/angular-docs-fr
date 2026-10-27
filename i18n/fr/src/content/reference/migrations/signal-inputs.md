<!-- source: 3dd3f8d7d644de2915c4e687933b69140bdc7caf -->
# Migration vers les inputs signal

Angular a introduit une API améliorée pour les inputs, considérée comme
prête pour la production depuis la v19.
Pour en savoir plus sur les inputs signal et leurs avantages, consultez le [guide dédié](guide/components/inputs).

Pour accompagner les équipes existantes qui souhaitent utiliser les inputs signal, l'équipe Angular
fournit une migration automatique qui convertit les champs `@Input` vers la nouvelle API `input()`.

Lancez le schematic avec la commande suivante :

```bash
ng generate @angular/core:signal-input-migration
```

La migration est aussi disponible sous forme d'[action de refactoring](https://code.visualstudio.com/docs/typescript/typescript-refactoring#_refactoring) dans VSCode.
Installez la dernière version de l'extension VSCode et cliquez sur un champ `@Input`.
Plus de détails dans la section [ci-dessous](#vscode-extension).

## Que change la migration ? {#what-does-the-migration-change}

1. Les membres de classe `@Input()` sont remplacés par leur équivalent signal `input()`.
2. Les références aux inputs migrés sont mises à jour pour appeler le signal.
   - Cela inclut les références dans les templates, les host bindings ou le code TypeScript.

**Avant**

```angular-ts
import {Component, Input} from '@angular/core';

@Component({
  template: `Name: {{ name ?? '' }}`,
})
export class MyComponent {
  @Input() name: string | undefined = undefined;

  someMethod(): number {
    if (this.name) {
      return this.name.length;
    }
    return -1;
  }
}
```

**Après**

```angular-ts {highlight: [[4],[7], [10,12]]}
import {Component, input} from '@angular/core';

@Component({
  template: `Name: {{ name() ?? '' }}`,
})
export class MyComponent {
  readonly name = input<string>();

  someMethod(): number {
    const name = this.name();
    if (name) {
      return name.length;
    }
    return -1;
  }
}
```

## Options de configuration {#configuration-options}

La migration propose quelques options pour l'ajuster finement à vos besoins.

### `--path`

Par défaut, la migration met à jour tout votre workspace Angular CLI.
Vous pouvez limiter la migration à un sous-répertoire spécifique avec cette option.

### `--best-effort-mode`

Par défaut, la migration ignore les inputs qui ne peuvent pas être migrés de façon sûre.
La migration essaie de refactoriser le code de la manière la plus sûre possible.

Quand l'option `--best-effort-mode` est activée, la migration essaie activement
de migrer le plus de code possible, même si cela peut casser votre build.

### `--insert-todos`

Quand cette option est activée, la migration ajoute des TODO aux inputs qui n'ont pas pu être migrés.
Les TODO expliquent pourquoi les inputs ont été ignorés. Par exemple :

```ts
// TODO: Skipped for migration because:
//  Your application code writes to the input. This prevents migration.
@Input() myInput = false;
```

### `--analysis-dir`

Dans les grands projets, vous pouvez utiliser cette option pour réduire le nombre de fichiers analysés.
Par défaut, la migration analyse tout le workspace, quelle que soit l'option `--path`, afin
de mettre à jour toutes les références concernées par une migration de `@Input()`.

Avec cette option, vous pouvez limiter l'analyse à un sous-dossier. Notez que toutes les
références en dehors de ce répertoire sont alors ignorées sans avertissement, ce qui peut casser votre build.

## Extension VSCode {#vscode-extension}

![Capture d'écran de l'extension VSCode lors d'un clic sur un champ `@Input`](assets/images/migrations/signal-inputs-vscode.png 'Capture de la fenêtre VSCode lors du clic sur un champ `@Input`.')

La migration est disponible sous forme d'[action de refactoring](https://code.visualstudio.com/docs/typescript/typescript-refactoring#_refactoring) dans VSCode.

Pour utiliser la migration via VSCode, installez la dernière version de l'extension VSCode et cliquez :

- sur un champ `@Input`,
- ou sur une directive ou un composant.

Attendez ensuite que le bouton de refactoring VSCode en forme d'ampoule jaune apparaisse.
Ce bouton vous permet alors de sélectionner la migration vers les inputs signal.
