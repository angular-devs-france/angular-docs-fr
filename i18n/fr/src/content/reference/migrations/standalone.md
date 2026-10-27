<!-- source: 0db21adbc4d453d44c4ba522225038345478032d -->
# Migrer un projet Angular existant vers standalone

Les **composants standalone** offrent une façon simplifiée de construire des applications Angular. Les composants, directives et pipes standalone visent à simplifier l'expérience de développement en réduisant le besoin de `NgModule`. Les applications existantes peuvent, si elles le souhaitent, adopter progressivement le nouveau style standalone sans aucun changement incompatible.

<docs-video src="https://www.youtube.com/embed/x5PZwb4XurU" title="Bien démarrer avec les composants standalone"/>

Ce schematic aide à transformer les composants, directives et pipes des projets existants en standalone. Le schematic vise à transformer automatiquement le plus de code possible, mais l'auteur du projet peut devoir faire quelques corrections manuelles.

Lancez le schematic avec la commande suivante :

```shell
ng generate @angular/core:standalone
```

## Avant la mise à jour {#before-updating}

Avant d'utiliser le schematic, assurez-vous que le projet :

1. Utilise Angular 15.2.0 ou une version ultérieure.
2. Compile sans aucune erreur de compilation.
3. Est sur une branche Git propre et que tout le travail est sauvegardé.

## Options du schematic {#schematic-options}

| Option | Détails                                                                                                                       |
| :----- | :---------------------------------------------------------------------------------------------------------------------------- |
| `mode` | La transformation à effectuer. Consultez [Modes de migration](#migration-modes) ci-dessous pour le détail des options disponibles. |
| `path` | Le chemin à migrer, relatif à la racine du projet. Vous pouvez utiliser cette option pour migrer progressivement des parties de votre projet. |

## Étapes de la migration {#migrations-steps}

Le processus de migration se compose de trois étapes. Vous devrez le lancer plusieurs fois et vérifier manuellement que le projet compile et se comporte comme prévu.

NOTE: Bien que le schematic puisse mettre à jour automatiquement la majeure partie du code, certains cas particuliers nécessitent l'intervention d'un développeur.
Prévoyez d'appliquer des corrections manuelles après chaque étape de la migration. De plus, le nouveau code généré par le schematic peut ne pas respecter les règles de formatage de votre code.

Lancez la migration dans l'ordre ci-dessous, en vérifiant que votre code compile et s'exécute entre chaque étape :

1. Lancez `ng g @angular/core:standalone` et sélectionnez « Convert all components, directives and pipes to standalone »
2. Lancez `ng g @angular/core:standalone` et sélectionnez « Remove unnecessary NgModule classes »
3. Lancez `ng g @angular/core:standalone` et sélectionnez « Bootstrap the application using standalone APIs »
4. Lancez les vérifications de lint et de formatage, corrigez les éventuels échecs et committez le résultat

## Après la migration {#after-the-migration}

Félicitations, votre application a été convertie en standalone 🎉. Voici quelques étapes de suivi facultatives que vous pouvez effectuer maintenant :

- Trouvez et supprimez les déclarations `NgModule` restantes : comme l'[étape « Supprimer les NgModules inutiles »](#remove-unnecessary-ngmodules) ne peut pas supprimer tous les modules automatiquement, vous devrez peut-être supprimer manuellement les déclarations restantes.
- Lancez les tests unitaires du projet et corrigez les éventuels échecs.
- Lancez les outils de formatage de code, si le projet utilise un formatage automatique.
- Lancez les linters de votre projet et corrigez les nouveaux avertissements. Certains linters prennent en charge une option `--fix` qui peut résoudre automatiquement une partie de vos avertissements.

## Modes de migration {#migration-modes}

La migration propose les modes suivants :

1. Convertir les déclarations en standalone.
2. Supprimer les NgModules inutiles.
3. Passer à l'API de bootstrap standalone.
   Lancez ces migrations dans l'ordre indiqué.

### Convertir les déclarations en standalone {#convert-declarations-to-standalone}

Dans ce mode, la migration convertit tous les composants, directives et pipes en standalone en supprimant `standalone: false` et en ajoutant les dépendances à leur tableau `imports`.

HELPFUL: Pendant cette étape, le schematic ignore les NgModules qui font le bootstrap d'un composant, car il s'agit probablement de modules racines utilisés par `bootstrapModule` plutôt que par `bootstrapApplication`, compatible avec standalone. Le schematic convertit automatiquement ces déclarations lors de l'étape [« Passer à l'API de bootstrap standalone »](#switch-to-standalone-bootstrapping-api).

**Avant :**

```typescript {header: "shared.module.ts"}
@NgModule({
  imports: [CommonModule],
  declarations: [Greeter],
  exports: [Greeter],
})
export class SharedModule {}
```

```angular-ts {header: "greeter.ts"}
@Component({
  selector: 'greeter',
  template: '<div *ngIf="showGreeting">Hello</div>',
  standalone: false,
})
export class Greeter {
  showGreeting = true;
}
```

**Après :**

```typescript {header: "shared.module.ts"}
@NgModule({
  imports: [CommonModule, Greeter],
  exports: [Greeter],
})
export class SharedModule {}
```

```angular-ts {header: "greeter.ts"}
@Component({
  selector: 'greeter',
  template: '<div *ngIf="showGreeting">Hello</div>',
  imports: [NgIf],
})
export class Greeter {
  showGreeting = true;
}
```

### Supprimer les NgModules inutiles {#remove-unnecessary-ngmodules}

Une fois toutes les déclarations converties en standalone, de nombreux NgModules peuvent être supprimés sans risque. Cette étape supprime ces déclarations de modules et autant de références correspondantes que possible. Si la migration ne peut pas supprimer une référence automatiquement, elle laisse le commentaire TODO suivant pour que vous puissiez supprimer le NgModule manuellement :

```typescript
/* TODO(standalone-migration): clean up removed NgModule reference manually */
```

La migration considère qu'un module peut être supprimé sans risque si ce module :

- N'a pas de `declarations`.
- N'a pas de `providers`.
- N'a pas de composants `bootstrap`.
- N'a pas d'`imports` qui référencent un symbole `ModuleWithProviders` ou un module qui ne peut pas être supprimé.
- N'a pas de membres de classe. Les constructeurs vides sont ignorés.

**Avant :**

```typescript {header: "importer.module.ts"}
@NgModule({
  imports: [FooComponent, BarPipe],
  exports: [FooComponent, BarPipe],
})
export class ImporterModule {}
```

**Après :**

```typescript {header: "importer.module.ts"}
// Does not exist!
```

### Passer à l'API de bootstrap standalone {#switch-to-standalone-bootstrapping-api}

Cette étape convertit toutes les utilisations de `bootstrapModule` vers la nouvelle API `bootstrapApplication`, basée sur standalone. Elle supprime aussi `standalone: false` du composant racine et supprime le NgModule racine. Si le module racine a des `providers` ou des `imports`, la migration tente de copier autant de cette configuration que possible dans le nouvel appel de bootstrap.

**Avant :**

```typescript {header: "./app/app.module.ts"}
import {NgModule} from '@angular/core';
import {App} from './app';

@NgModule({
  declarations: [App],
  bootstrap: [App],
})
export class AppModule {}
```

```typescript {header: "./app/app.ts"}
@Component({
  selector: 'app',
  template: 'hello',
  standalone: false,
})
export class App {}
```

```typescript {header: "./main.ts"}
import {platformBrowser} from '@angular/platform-browser';
import {AppModule} from './app/app.module';

platformBrowser()
  .bootstrapModule(AppModule)
  .catch((e) => console.error(e));
```

**Après :**

```typescript {header: "./app/app.module.ts"}
// Does not exist!
```

```typescript {header: "./app/app.ts"}
@Component({
  selector: 'app',
  template: 'hello',
})
export class App {}
```

```typescript {header: "./main.ts"}
import {bootstrapApplication} from '@angular/platform-browser';
import {App} from './app/app';

bootstrapApplication(App).catch((e) => console.error(e));
```

## Problèmes courants {#common-problems}

Voici quelques problèmes courants qui peuvent empêcher le schematic de fonctionner correctement :

- Erreurs de compilation : si le projet comporte des erreurs de compilation, Angular ne peut pas l'analyser ni le migrer correctement.
- Fichiers non inclus dans un tsconfig : le schematic détermine les fichiers à migrer en analysant les fichiers `tsconfig.json` de votre projet. Le schematic exclut tous les fichiers qui ne sont couverts par aucun tsconfig.
- Code impossible à analyser statiquement : le schematic utilise l'analyse statique pour comprendre votre code et déterminer où apporter des modifications. La migration peut ignorer les classes dont les métadonnées ne peuvent pas être analysées statiquement au moment du build.

## Limitations {#limitations}

En raison de la taille et de la complexité de la migration, le schematic ne peut pas gérer certains cas :

- Comme les tests unitaires ne sont pas compilés ahead-of-time (AoT), les `imports` ajoutés aux composants dans les tests unitaires peuvent ne pas être entièrement corrects.
- Le schematic s'appuie sur des appels directs aux API Angular. Le schematic ne peut pas reconnaître les wrappers personnalisés autour des API Angular. Par exemple, si vous définissez une fonction personnalisée `customConfigureTestModule` qui encapsule `TestBed.configureTestingModule`, les composants qu'elle déclare risquent de ne pas être reconnus.
