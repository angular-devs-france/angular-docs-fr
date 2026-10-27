<!-- source: 663bac8a22a486da16c808c7498102bd66e5d993 -->
# Migration vers la fonction output

Angular a introduit en v17.3 une API améliorée pour les outputs, considérée comme
prête pour la production depuis la v19. Cette API imite l'API `input()` mais ne repose pas sur les signals.
Pour en savoir plus sur la fonction output pour les événements personnalisés et ses avantages, consultez le [guide dédié](guide/components/outputs).

Pour accompagner les projets existants qui souhaitent utiliser la fonction output, l'équipe Angular
fournit une migration automatique qui convertit les événements personnalisés `@Output` vers la nouvelle API `output()`.

Lancez le schematic avec la commande suivante :

```bash
ng generate @angular/core:output-migration
```

## Que change la migration ? {#what-does-the-migration-change}

1. Les membres de classe `@Output()` sont remplacés par leur équivalent `output()`.
2. Les imports dans le fichier des composants ou des directives, au niveau du module TypeScript, sont aussi mis à jour.
3. Les appels d'API comme `event.next()`, dont l'usage est déconseillé, sont migrés vers `event.emit()` et les appels à `event.complete()` sont supprimés.

**Avant**

```typescript
import {Component, Output, EventEmitter} from '@angular/core';

@Component({
  template: `<button (click)="someMethod('test')">emit</button>`,
})
export class MyComponent {
  @Output() someChange = new EventEmitter<string>();

  someMethod(value: string): void {
    this.someChange.emit(value);
  }
}
```

**Après**

```typescript
import {Component, output} from '@angular/core';

@Component({
  template: `<button (click)="someMethod('test')">emit</button>`,
})
export class MyComponent {
  readonly someChange = output<string>();

  someMethod(value: string): void {
    this.someChange.emit(value);
  }
}
```

## Options de configuration {#configuration-options}

La migration propose quelques options pour l'ajuster finement à vos besoins.

### `--path`

Si cette option n'est pas précisée, la migration vous demande un chemin et met à jour tout votre workspace Angular CLI.
Vous pouvez limiter la migration à un sous-répertoire spécifique avec cette option.

### `--analysis-dir`

Dans les grands projets, vous pouvez utiliser cette option pour réduire le nombre de fichiers analysés.
Par défaut, la migration analyse tout le workspace, quelle que soit l'option `--path`, afin
de mettre à jour toutes les références concernées par une migration de `@Output()`.

Avec cette option, vous pouvez limiter l'analyse à un sous-dossier. Notez que toutes les
références en dehors de ce répertoire sont alors ignorées sans avertissement, ce qui peut casser votre build.

Utilisez ces options comme ci-dessous :

```bash
ng generate @angular/core:output-migration --path src/app/sub-folder
```

## Exceptions {#exceptions}

Dans certains cas, la migration ne modifie pas le code.
L'une de ces exceptions est le cas où l'événement est utilisé avec une méthode `pipe()` en dehors d'un fichier de test.
Le code suivant ne sera pas migré :

```typescript
export class MyDialogComponent {
  @Output() close = new EventEmitter<void>();
  doSome(): void {
    this.close.complete();
  }
  otherThing(): void {
    this.close.pipe();
  }
}
```

Dans un fichier de test, la migration réécrit l'utilisation au lieu d'ignorer l'output : `this.close.pipe(…)` devient donc `outputToObservable(this.close).pipe(…)`.

NOTE: La migration reconnaît un fichier de test en cherchant un import dont le chemin contient `jasmine` ou `catalyst`. Elle ne reconnaît pas un fichier qui importe uniquement Jest ou Vitest : elle y ignore donc l'output au lieu de le réécrire.
