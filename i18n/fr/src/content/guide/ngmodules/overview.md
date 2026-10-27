<!-- source: 0f849f716fed62633e344849663981dd254982b7 -->
# NgModules

IMPORTANT: L'équipe Angular recommande d'utiliser des [composants standalone](guide/components) plutôt que `NgModule` pour tout nouveau code. Utilisez ce guide pour comprendre le code existant construit avec `@NgModule`.

Un NgModule est une classe marquée par le décorateur `@NgModule`. Ce décorateur accepte des _métadonnées_ qui indiquent à Angular comment compiler les templates des composants et configurer l'injection de dépendances.

```typescript
import {NgModule} from '@angular/core';

@NgModule({
  // Metadata goes here
})
export class CustomMenuModule {}
```

Un NgModule a deux responsabilités principales :

- Déclarer les composants, directives et pipes qui appartiennent au NgModule
- Ajouter des providers à l'injector pour les composants, directives et pipes qui importent le NgModule

## Déclarations {#declarations}

La propriété `declarations` des métadonnées `@NgModule` déclare les composants, directives et pipes qui appartiennent au NgModule.

```typescript
@NgModule({
  /* ... */
  // CustomMenu and CustomMenuItem are components.
  declarations: [CustomMenu, CustomMenuItem],
})
export class CustomMenuModule {}
```

Dans l'exemple ci-dessus, les composants `CustomMenu` et `CustomMenuItem` appartiennent à `CustomMenuModule`.

La propriété `declarations` accepte également des _tableaux_ de composants, de directives et de pipes. Ces tableaux peuvent eux-mêmes contenir d'autres tableaux.

```typescript
const MENU_COMPONENTS = [CustomMenu, CustomMenuItem];
const WIDGETS = [MENU_COMPONENTS, CustomSlider];

@NgModule({
  /* ... */
  // This NgModule declares all of CustomMenu, CustomMenuItem,
  // CustomSlider, and CustomCheckbox.
  declarations: [WIDGETS, CustomCheckbox],
})
export class CustomMenuModule {}
```

Si Angular découvre des composants, des directives ou des pipes déclarés dans plus d'un NgModule, il signale une erreur.

Tout composant, toute directive et tout pipe doit être explicitement marqué `standalone: false` pour pouvoir être déclaré dans un NgModule.

```typescript
@Component({
  // Mark this component as `standalone: false` so that it can be declared in an NgModule.
  standalone: false,
  /* ... */
})
export class CustomMenu {
  /* ... */
}
```

### imports {#imports}

Les composants déclarés dans un NgModule peuvent dépendre d'autres composants, directives et pipes. Ajoutez ces dépendances à la propriété `imports` des métadonnées `@NgModule`.

```typescript
@NgModule({
  /* ... */
  // CustomMenu and CustomMenuItem depend on the PopupTrigger and SelectorIndicator components.
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],
})
export class CustomMenuModule {}
```

Le tableau `imports` accepte d'autres NgModules, ainsi que des composants, directives et pipes standalone.

### exports {#exports}

Un NgModule peut _exporter_ les composants, directives et pipes qu'il déclare afin qu'ils soient disponibles pour d'autres composants et NgModules.

```typescript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],

  // Make CustomMenu and CustomMenuItem available to
  // components and NgModules that import CustomMenuModule.
  exports: [CustomMenu, CustomMenuItem],
})
export class CustomMenuModule {}
```

La propriété `exports` ne se limite cependant pas aux déclarations. Un NgModule peut aussi exporter tous les autres composants, directives, pipes et NgModules qu'il importe.

```typescript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],

  // Also make PopupTrigger available to any component or NgModule that imports CustomMenuModule.
  exports: [CustomMenu, CustomMenuItem, PopupTrigger],
})
export class CustomMenuModule {}
```

## Providers d'un `NgModule` {#ngmodule-providers}

TIP: Consultez le [guide de l'injection de dépendances](guide/di) pour en savoir plus sur l'injection de dépendances et les providers.

Un `NgModule` peut spécifier des `providers` pour les dépendances injectées. Ces providers sont disponibles pour :

- Tout composant, toute directive ou tout pipe standalone qui importe le NgModule, et
- Les `declarations` et les `providers` de tout _autre_ NgModule qui importe le NgModule.

```typescript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],

  // Provide the OverlayManager service
  providers: [OverlayManager],
  /* ... */
})
export class CustomMenuModule {}

@NgModule({
  imports: [CustomMenuModule],
  declarations: [UserProfile],
  providers: [UserDataClient],
})
export class UserProfileModule {}
```

Dans l'exemple ci-dessus :

- `CustomMenuModule` fournit `OverlayManager`.
- Les composants `CustomMenu` et `CustomMenuItem` peuvent injecter `OverlayManager`, car ils sont déclarés dans `CustomMenuModule`.
- `UserProfile` peut injecter `OverlayManager`, car son NgModule importe `CustomMenuModule`.
- `UserDataClient` peut injecter `OverlayManager`, car son NgModule importe `CustomMenuModule`.

### Le pattern `forRoot` et `forChild` {#the-forroot-and-forchild-pattern}

Certains NgModules définissent une méthode statique `forRoot` qui accepte une configuration et renvoie un tableau de providers. Le nom « `forRoot` » est une convention qui indique que ces providers sont destinés à être ajoutés exclusivement à la _racine_ de votre application lors du bootstrap.

Tous les providers inclus de cette manière sont chargés immédiatement, ce qui augmente la taille du bundle JavaScript du chargement initial de la page.

```typescript
bootstrapApplication(MyApplicationRoot, {
  providers: [CustomMenuModule.forRoot(/* some config */)],
});
```

De même, certains NgModules peuvent définir une méthode statique `forChild` qui indique que les providers sont destinés à être ajoutés à des composants situés dans la hiérarchie de votre application.

```typescript
@Component({
  /* ... */
  providers: [CustomMenuModule.forChild(/* some config */)],
})
export class UserProfile {
  /* ... */
}
```

## Bootstrap d'une application {#bootstrapping-an-application}

IMPORTANT: L'équipe Angular recommande d'utiliser [bootstrapApplication](api/platform-browser/bootstrapApplication) plutôt que `bootstrapModule` pour tout nouveau code. Utilisez ce guide pour comprendre les applications existantes dont le bootstrap utilise `@NgModule`.

Le décorateur `@NgModule` accepte un tableau `bootstrap` facultatif qui peut contenir un ou plusieurs composants.

Vous pouvez utiliser la méthode [`bootstrapModule`](/api/core/PlatformRef#bootstrapModule) de [`platformBrowser`](api/platform-browser/platformBrowser) ou de [`platformServer`](api/platform-server/platformServer) pour démarrer une application Angular. À l'exécution, cette fonction repère les éléments de la page dont un selector CSS correspond au ou aux composants listés, et rend ces composants sur la page.

```typescript
import {platformBrowser} from '@angular/platform-browser';
import {NgModule} from '@angular/core';

@NgModule({
  bootstrap: [MyApplication],
})
export class MyApplicationModule {}

platformBrowser().bootstrapModule(MyApplicationModule);
```

Les composants listés dans `bootstrap` sont automatiquement inclus dans les déclarations du NgModule.

Quand vous effectuez le bootstrap d'une application à partir d'un NgModule, les `providers` collectés de ce module et tous les `providers` de ses `imports` sont chargés immédiatement et disponibles à l'injection dans toute l'application.
