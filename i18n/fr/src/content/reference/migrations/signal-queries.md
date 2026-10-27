<!-- source: fdad1a9e4d999b19db6f1ee54c22a31e5cb525ed -->
# Migration vers les queries signal

Angular a introduit des API améliorées pour les queries, considérées comme
prêtes pour la production depuis la v19.
Pour en savoir plus sur les queries signal et leurs avantages, consultez le [guide dédié](guide/components/queries).

Pour accompagner les équipes existantes qui souhaitent utiliser les queries signal, l'équipe Angular
fournit une migration automatique qui convertit les champs de query existants basés sur des décorateurs vers la nouvelle API.

Lancez le schematic avec la commande suivante :

```bash
ng generate @angular/core:signal-queries-migration
```

La migration est aussi disponible sous forme d'[action de refactoring](https://code.visualstudio.com/docs/typescript/typescript-refactoring#_refactoring) dans VSCode.
Installez la dernière version de l'extension VSCode et cliquez par exemple sur un champ `@ViewChild`.
Plus de détails dans la section [ci-dessous](#vscode-extension).

## Que change la migration ? {#what-does-the-migration-change}

1. Les membres de classe `@ViewChild()`, `@ViewChildren`, `@ContentChild` et `@ContentChildren`
   sont remplacés par leurs équivalents signal.
2. Les références aux queries migrées dans votre application sont mises à jour pour appeler le signal.
   - Cela inclut les références dans les templates, les host bindings ou le code TypeScript.

**Avant**

```angular-ts
import {Component, ContentChild, ElementRef} from '@angular/core';

@Component({
  template: `Has ref: {{ ref ? 'Yes' : 'No' }}`,
})
export class MyComponent {
  @ContentChild('someRef') ref: ElementRef | undefined = undefined;

  someMethod(): void {
    if (this.ref) {
      this.ref.nativeElement;
    }
  }
}
```

**Après**

```angular-ts
import {Component, contentChild, ElementRef} from '@angular/core';

@Component({
  template: `Has ref: {{ ref() ? 'Yes' : 'No' }}`,
})
export class MyComponent {
  readonly ref = contentChild<ElementRef>('someRef');

  someMethod(): void {
    const ref = this.ref();
    if (ref) {
      ref.nativeElement;
    }
  }
}
```

## Options de configuration {#configuration-options}

La migration propose quelques options pour l'ajuster finement à vos besoins.

### `--path`

Par défaut, la migration met à jour tout votre workspace Angular CLI.
Vous pouvez limiter la migration à un sous-répertoire spécifique avec cette option.

### `--best-effort-mode`

Par défaut, la migration ignore les queries qui ne peuvent pas être migrées de façon sûre.
La migration essaie de refactoriser le code de la manière la plus sûre possible.

Quand l'option `--best-effort-mode` est activée, la migration essaie activement
de migrer le plus de code possible, même si cela peut casser votre build.

### `--insert-todos`

Quand cette option est activée, la migration ajoute des TODO aux queries qui n'ont pas pu être migrées.
Les TODO expliquent pourquoi les queries ont été ignorées. Par exemple :

```ts
// TODO: Skipped for migration because:
//  Your application code writes to the query. This prevents migration.
@ViewChild('ref') ref?: ElementRef;
```

### `--analysis-dir`

Dans les grands projets, vous pouvez utiliser cette option pour réduire le nombre de fichiers analysés.
Par défaut, la migration analyse tout le workspace, quelle que soit l'option `--path`, afin
de mettre à jour toutes les références concernées par la migration d'une déclaration de query.

Avec cette option, vous pouvez limiter l'analyse à un sous-dossier. Notez que toutes les
références en dehors de ce répertoire sont alors ignorées sans avertissement, ce qui peut casser votre build.

## Extension VSCode {#vscode-extension}

![Capture d'écran de l'extension VSCode lors d'un clic sur un champ `@ViewChild`](assets/images/migrations/signal-queries-vscode.png 'Capture de la fenêtre VSCode lors du clic sur un champ `@ViewChild`.')

La migration est disponible sous forme d'[action de refactoring](https://code.visualstudio.com/docs/typescript/typescript-refactoring#_refactoring) dans VSCode.

Pour utiliser la migration via VSCode, installez la dernière version de l'extension VSCode et cliquez :

- sur un champ `@ViewChild`, `@ViewChildren`, `@ContentChild` ou `@ContentChildren`,
- ou sur une directive ou un composant.

Attendez ensuite que le bouton de refactoring VSCode en forme d'ampoule jaune apparaisse.
Ce bouton vous permet alors de sélectionner la migration vers les queries signal.
